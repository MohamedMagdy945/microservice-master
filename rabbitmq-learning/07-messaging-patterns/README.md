# 07 — Messaging Patterns

> **Goal:** Learn the most important real-world messaging patterns and implement them in .NET.

---

## 1. Work Queues (Competing Consumers)

**Problem:** One producer creates tasks faster than one consumer can handle them.  
**Solution:** Multiple consumers share a single queue — each message goes to exactly one consumer.

```
Producer ──► [work-queue] ──► Consumer 1 (processing task 1)
                          └──► Consumer 2 (processing task 2)
                          └──► Consumer 3 (processing task 3)
```

### Implementation

```csharp
// Producer — sends tasks
for (int i = 1; i <= 10; i++)
{
    var task = new { TaskId = i, Data = $"Task_{i}" };
    producer.Publish("", "work-queue", task);
}

// Consumer — multiple instances run the same code
channel.BasicQos(prefetchSize: 0, prefetchCount: 1, global: false); // fair dispatch

consumer.Received += (sender, ea) =>
{
    var task = Deserialize<WorkTask>(ea.Body.ToArray());
    
    Console.WriteLine($"[Worker] Processing Task {task.TaskId}...");
    Thread.Sleep(task.DurationMs); // simulate work
    Console.WriteLine($"[Worker] Task {task.TaskId} complete.");

    channel.BasicAck(ea.DeliveryTag, multiple: false);
};
```

### Key Setting: Fair Dispatch
```csharp
// Without this, RabbitMQ uses round-robin (slow workers get overwhelmed)
// With this, each worker only gets a new task when they finish the previous one
channel.BasicQos(prefetchSize: 0, prefetchCount: 1, global: false);
```

**Use when:** Image processing, email sending, report generation, any CPU-intensive task.

---

## 2. Publish/Subscribe (Event Broadcasting)

**Problem:** An event needs to notify multiple services simultaneously.  
**Solution:** Use a Fanout exchange — every bound queue gets a copy.

```
OrderService (producer)
       │
       ▼
[Fanout Exchange: "order-events"]
       ├──► [email-queue]    ──► EmailService      (sends confirmation)
       ├──► [inventory-queue]──► InventoryService  (updates stock)
       └──► [analytics-queue]──► AnalyticsService  (records metrics)
```

### Implementation

```csharp
// Producer — publish event once
channel.ExchangeDeclare("order-events", ExchangeType.Fanout, durable: true);
producer.Publish("order-events", routingKey: "", new OrderCreatedEvent { OrderId = 123 });

// Consumer 1 — EmailService
channel.ExchangeDeclare("order-events", ExchangeType.Fanout, durable: true);
var queueName = channel.QueueDeclare("email-queue", durable: true).QueueName;
channel.QueueBind(queueName, "order-events", routingKey: "");

// Consumer 2 — InventoryService (same exchange, different queue)
var invQueue = channel.QueueDeclare("inventory-queue", durable: true).QueueName;
channel.QueueBind(invQueue, "order-events", routingKey: "");
```

> **Tip:** Use temporary queues (`QueueDeclare("")`) for temporary subscribers — RabbitMQ generates a unique name and auto-deletes when the consumer disconnects.

**Use when:** Order created, user registered, payment processed — any event multiple services care about.

---

## 3. Routing (Filtered Delivery)

**Problem:** Different consumers care about different message types.  
**Solution:** Use a Direct or Topic exchange with specific routing keys.

```
"order.created.eu"  ──►[Topic Exchange]──► [eu-orders-queue]   ──► EuropeHandler
"order.created.us"  ──►[Topic Exchange]──► [us-orders-queue]   ──► UsaHandler
"order.cancelled.#" ──►[Topic Exchange]──► [cancels-queue]     ──► CancelHandler
```

### Implementation

```csharp
// Declare topic exchange
channel.ExchangeDeclare("orders", ExchangeType.Topic, durable: true);

// EU orders handler
channel.QueueDeclare("eu-orders", durable: true, exclusive: false, autoDelete: false);
channel.QueueBind("eu-orders", "orders", "order.created.eu");
channel.QueueBind("eu-orders", "orders", "order.updated.eu");

// All cancellations handler
channel.QueueDeclare("all-cancels", durable: true, exclusive: false, autoDelete: false);
channel.QueueBind("all-cancels", "orders", "order.cancelled.#");

// Producer
channel.BasicPublish("orders", "order.created.eu", null, body);
// → Goes to: eu-orders ✅, all-cancels ❌

channel.BasicPublish("orders", "order.cancelled.us", null, body);
// → Goes to: eu-orders ❌, all-cancels ✅
```

**Use when:** Multi-region routing, log level filtering, feature-specific consumers.

---

## 4. Request/Response (RPC Pattern)

**Problem:** You need an answer back from another service.  
**Solution:** Send a message with a `ReplyTo` queue; consumer responds there.

```
Client ──[request]──► [rpc-queue] ──► Server
Client ◄──[response]── [reply-queue] ◄── Server
```

### Implementation

```csharp
// Client (Caller)
public async Task<string> CallRpc(string input)
{
    var replyQueue  = channel.QueueDeclare(exclusive: true); // temp queue for response
    var correlationId = Guid.NewGuid().ToString();
    var tcs = new TaskCompletionSource<string>();

    var consumer = new EventingBasicConsumer(channel);
    consumer.Received += (sender, ea) =>
    {
        if (ea.BasicProperties.CorrelationId == correlationId)
        {
            var response = Encoding.UTF8.GetString(ea.Body.ToArray());
            tcs.TrySetResult(response);
        }
    };
    channel.BasicConsume(replyQueue.QueueName, autoAck: true, consumer);

    // Send request
    var props = channel.CreateBasicProperties();
    props.CorrelationId = correlationId;
    props.ReplyTo       = replyQueue.QueueName; // tell server where to respond

    var body = Encoding.UTF8.GetBytes(input);
    channel.BasicPublish("", "rpc-queue", props, body);

    return await tcs.Task.WaitAsync(TimeSpan.FromSeconds(30));
}

// Server (Responder)
consumer.Received += (sender, ea) =>
{
    var request  = Encoding.UTF8.GetString(ea.Body.ToArray());
    var response = ProcessRequest(request); // do the work

    // Send response back
    var replyProps = channel.CreateBasicProperties();
    replyProps.CorrelationId = ea.BasicProperties.CorrelationId;

    var responseBody = Encoding.UTF8.GetBytes(response);
    channel.BasicPublish("", ea.BasicProperties.ReplyTo, replyProps, responseBody);
    channel.BasicAck(ea.DeliveryTag, false);
};
```

**Use when:** Getting product price, checking inventory, validating data — when you need a synchronous-style call but want decoupling.

---

## 5. Retry Mechanism

**Problem:** Processing fails temporarily (e.g., database down). Retrying immediately overwhelms the system.  
**Solution:** Use a delay queue with exponential backoff.

```
[main-queue] → fails → [retry-queue with TTL] → expires → [main-queue]
                                                             (retry attempt)
```

### Implementation

```csharp
// Setup: main queue + retry queue
channel.ExchangeDeclare("main-exchange",  ExchangeType.Direct, durable: true);
channel.ExchangeDeclare("retry-exchange", ExchangeType.Direct, durable: true);

// Main queue — on rejection, route to retry
channel.QueueDeclare("main-queue", durable: true, exclusive: false, autoDelete: false,
    arguments: new Dictionary<string, object>
    {
        { "x-dead-letter-exchange", "retry-exchange" },  // failed messages go here
        { "x-dead-letter-routing-key", "retry" }
    });

// Retry queue — holds messages for 5 seconds, then sends back to main
channel.QueueDeclare("retry-queue", durable: true, exclusive: false, autoDelete: false,
    arguments: new Dictionary<string, object>
    {
        { "x-message-ttl",           5000 },             // wait 5 seconds
        { "x-dead-letter-exchange",  "main-exchange" },  // return to main after TTL
        { "x-dead-letter-routing-key","main" }
    });

channel.QueueBind("main-queue",  "main-exchange",  "main");
channel.QueueBind("retry-queue", "retry-exchange", "retry");

// Consumer with retry tracking
consumer.Received += (sender, ea) =>
{
    try
    {
        ProcessMessage(ea.Body.ToArray());
        channel.BasicAck(ea.DeliveryTag, false);
    }
    catch
    {
        int retryCount = GetRetryCount(ea.BasicProperties.Headers);

        if (retryCount >= 3)
        {
            // Max retries exceeded → send to Dead Letter Queue
            channel.BasicNack(ea.DeliveryTag, false, requeue: false);
        }
        else
        {
            // Increment retry count in headers and requeue
            var props = channel.CreateBasicProperties();
            props.Headers = new Dictionary<string, object>
            {
                { "x-retry-count", retryCount + 1 }
            };
            channel.BasicPublish("retry-exchange", "retry", props, ea.Body.ToArray());
            channel.BasicAck(ea.DeliveryTag, false); // ack original
        }
    }
};
```

---

## 6. Dead Letter Queue (DLQ)

**Problem:** Some messages can never be processed (invalid format, unrecoverable error). Don't lose them.  
**Solution:** Route permanently failed messages to a Dead Letter Queue for inspection.

```
[main-queue] → max retries exceeded → [dlq-exchange] → [dead-letter-queue]
                                                              │
                                                         Manual review / replay
```

### Implementation

```csharp
// DLQ setup
channel.ExchangeDeclare("dlx", ExchangeType.Direct, durable: true);
channel.QueueDeclare("dead-letter-queue", durable: true, exclusive: false, autoDelete: false);
channel.QueueBind("dead-letter-queue", "dlx", "dlq");

// Main queue configured to use DLX
channel.QueueDeclare("main-queue", durable: true, exclusive: false, autoDelete: false,
    arguments: new Dictionary<string, object>
    {
        { "x-dead-letter-exchange",   "dlx" },
        { "x-dead-letter-routing-key","dlq" }
    });

// When max retries exceeded:
channel.BasicNack(ea.DeliveryTag, multiple: false, requeue: false);
// ↑ Message automatically forwarded to dead-letter-queue via dlx
```

**Messages go to DLQ when:**
- `BasicNack` / `BasicReject` with `requeue: false`
- Message TTL expires in the queue
- Queue max length exceeded

**Use when:** Any production system — always have a DLQ to catch unprocessable messages.

---

## Pattern Summary

| Pattern | Exchange | When to Use |
|---------|----------|-------------|
| Work Queue | Default / Direct | Distribute tasks across workers |
| Pub/Sub | Fanout | Broadcast events to many services |
| Routing | Direct / Topic | Filtered delivery by message type |
| RPC | Direct + Reply queue | Need a response from another service |
| Retry | Multiple queues + TTL | Transient failures, backoff retry |
| DLQ | Dead Letter Exchange | Capture unprocessable messages |

---

**Previous:** [06 — .NET Integration Basics ←](../06-dotnet-integration/README.md)  
**Next:** [08 — Microservices Integration →](../08-microservices-integration/README.md)
