# 10 — Production Best Practices

> **Goal:** Make your RabbitMQ setup production-ready with durability, resilience, and observability.

---

## 1. Message Durability

Messages can be lost if RabbitMQ restarts unless you configure durability at every level.

### Three-Layer Durability

```
Layer 1: Durable Exchange
Layer 2: Durable Queue
Layer 3: Persistent Message (DeliveryMode = 2)

All three must be set for messages to survive a restart.
```

```csharp
// ✅ Durable exchange
channel.ExchangeDeclare("orders", ExchangeType.Topic, durable: true, autoDelete: false);

// ✅ Durable queue
channel.QueueDeclare("order-created", durable: true, exclusive: false, autoDelete: false);

// ✅ Persistent message
var props = channel.CreateBasicProperties();
props.DeliveryMode = 2; // 1 = transient (RAM only), 2 = persistent (disk)

channel.BasicPublish("orders", "order.created", props, body);
```

> **Warning:** Persistence has a performance cost. RabbitMQ must fsync to disk. Use it only for critical messages.

---

## 2. Idempotency (Avoid Duplicate Processing)

RabbitMQ may deliver a message more than once (network retries, consumer restarts). Your handler must be safe to run multiple times.

### Idempotency Key Pattern

```csharp
public class IdempotentOrderHandler
{
    private readonly IProcessedMessagesRepository _processed;

    public async Task HandleAsync(BasicDeliverEventArgs ea, OrderCreatedEvent @event)
    {
        var messageId = ea.BasicProperties.MessageId;

        // Check if already processed
        if (await _processed.ExistsAsync(messageId))
        {
            _logger.LogWarning("Duplicate message {MessageId} — skipping", messageId);
            channel.BasicAck(ea.DeliveryTag, false); // ack and skip
            return;
        }

        // Process
        await ProcessOrderAsync(@event);

        // Mark as processed (atomically with business logic if possible)
        await _processed.MarkAsync(messageId, expiresAt: DateTime.UtcNow.AddDays(7));

        channel.BasicAck(ea.DeliveryTag, false);
    }
}
```

### Database Idempotency (Upsert)

```csharp
// Instead of INSERT, use UPSERT
await _db.Database.ExecuteSqlRawAsync("""
    INSERT INTO processed_orders (order_id, processed_at)
    VALUES ({0}, {1})
    ON CONFLICT (order_id) DO NOTHING
""", @event.OrderId, DateTime.UtcNow);
```

---

## 3. Retry Policies with Exponential Backoff

Use exponential backoff to avoid hammering a failing dependency.

```csharp
// Infrastructure/Messaging/RetryPolicy.cs
public class RetryPolicy
{
    private static readonly TimeSpan[] BackoffIntervals =
    {
        TimeSpan.FromSeconds(5),   // retry 1: 5s
        TimeSpan.FromSeconds(30),  // retry 2: 30s
        TimeSpan.FromMinutes(5),   // retry 3: 5min
        TimeSpan.FromMinutes(30),  // retry 4: 30min
    };

    public static void Setup(IModel channel, string mainQueue, string mainExchange)
    {
        channel.ExchangeDeclare("dlx", ExchangeType.Direct, durable: true);

        // Create retry queues with escalating TTLs
        for (int i = 0; i < BackoffIntervals.Length; i++)
        {
            var retryQueue = $"{mainQueue}.retry.{i + 1}";
            int ttlMs = (int)BackoffIntervals[i].TotalMilliseconds;

            channel.QueueDeclare(retryQueue, durable: true, exclusive: false, autoDelete: false,
                arguments: new Dictionary<string, object>
                {
                    { "x-message-ttl",            ttlMs },
                    { "x-dead-letter-exchange",   mainExchange },
                    { "x-dead-letter-routing-key", mainQueue }
                });
            channel.QueueBind(retryQueue, "dlx", retryQueue);
        }

        // Final DLQ for permanently failed messages
        channel.QueueDeclare($"{mainQueue}.dlq", durable: true, exclusive: false, autoDelete: false);
        channel.QueueBind($"{mainQueue}.dlq", "dlx", $"{mainQueue}.dlq");
    }
}

// Consumer with retry logic
private void HandleWithRetry(BasicDeliverEventArgs ea, Action process)
{
    var retryCount = GetRetryCount(ea.BasicProperties.Headers);

    try
    {
        process();
        channel.BasicAck(ea.DeliveryTag, false);
    }
    catch (TransientException ex) when (retryCount < 4)
    {
        _logger.LogWarning("Transient failure (attempt {Count}), scheduling retry", retryCount + 1);

        var props = channel.CreateBasicProperties();
        props.Headers = new Dictionary<string, object> { { "x-retry-count", retryCount + 1 } };
        props.DeliveryMode = 2;

        var retryQueue = $"{_queueName}.retry.{retryCount + 1}";
        channel.BasicPublish("dlx", retryQueue, props, ea.Body.ToArray());
        channel.BasicAck(ea.DeliveryTag, false);
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Permanent failure, sending to DLQ");
        channel.BasicNack(ea.DeliveryTag, false, requeue: false); // → DLQ via x-dead-letter
    }
}

private int GetRetryCount(IDictionary<string, object>? headers)
{
    if (headers?.TryGetValue("x-retry-count", out var value) == true)
        return Convert.ToInt32(value);
    return 0;
}
```

---

## 4. Dead Letter Exchanges (DLX)

Every queue in production should have a DLX configured.

```csharp
// Setup DLX + DLQ
channel.ExchangeDeclare("dlx", ExchangeType.Direct, durable: true);

channel.QueueDeclare("orders.dlq", durable: true, exclusive: false, autoDelete: false,
    arguments: new Dictionary<string, object>
    {
        // Optional: DLQ messages expire after 30 days
        { "x-message-ttl", 2_592_000_000 } // 30 days in ms
    });

channel.QueueBind("orders.dlq", "dlx", "orders.dlq");

// Main queue with DLX configured
channel.QueueDeclare("orders", durable: true, exclusive: false, autoDelete: false,
    arguments: new Dictionary<string, object>
    {
        { "x-dead-letter-exchange",    "dlx" },
        { "x-dead-letter-routing-key", "orders.dlq" },
        { "x-message-ttl",             3_600_000 },   // max 1hr in queue
        { "x-max-length",              10_000 }        // max 10k messages (overflow → DLQ)
    });
```

### Replaying DLQ Messages

```csharp
// ReplayDlqService.cs — move messages from DLQ back to main queue
public async Task ReplayAsync(string dlqName, string targetExchange, string routingKey, int count)
{
    for (int i = 0; i < count; i++)
    {
        var result = channel.BasicGet(dlqName, autoAck: false);
        if (result is null) break;

        channel.BasicPublish(targetExchange, routingKey, result.BasicProperties, result.Body);
        channel.BasicAck(result.DeliveryTag, false);
    }
}
```

---

## 5. Monitoring (Logs + Metrics)

### Structured Logging

```csharp
// Always log with structured context
_logger.LogInformation(
    "Message {MessageId} processed successfully. Queue: {Queue}, CorrelationId: {CorrelationId}",
    ea.BasicProperties.MessageId,
    _queueName,
    ea.BasicProperties.CorrelationId
);

_logger.LogError(ex,
    "Failed to process message {MessageId} (attempt {RetryCount}). Queue: {Queue}",
    ea.BasicProperties.MessageId,
    retryCount,
    _queueName
);
```

### Key Metrics to Monitor

| Metric | Alert Threshold | Meaning |
|--------|----------------|---------|
| Queue depth | > 1000 | Consumers falling behind |
| Consumer count | = 0 | All consumers down |
| Message rate (publish) | Sudden drop | Producer issue |
| Message rate (consume) | Sudden drop | Consumer issue |
| DLQ depth | > 0 | Messages failing |
| Memory usage | > 80% | Risk of flow control |
| Disk space | > 70% | Risk of message loss |

### RabbitMQ REST API (Polling Metrics)

```csharp
// Poll RabbitMQ management API for queue stats
var response = await _httpClient.GetFromJsonAsync<QueueInfo>(
    "http://rabbitmq:15672/api/queues/%2F/order-created");

_metrics.RecordGauge("rabbitmq.queue.messages", response.Messages);
_metrics.RecordGauge("rabbitmq.queue.consumers", response.Consumers);
```

### Prometheus Integration

```bash
# Use rabbitmq_prometheus plugin
docker run -d \
  --name rabbitmq \
  -p 5672:5672 \
  -p 15672:15672 \
  -p 15692:15692 \  # Prometheus metrics endpoint
  rabbitmq:3-management

docker exec rabbitmq rabbitmq-plugins enable rabbitmq_prometheus
```

Metrics available at: `http://localhost:15692/metrics`

---

## 6. Connection Resilience

```csharp
// Polly for connection retry
public class ResilientConnectionFactory
{
    private readonly ConnectionFactory _factory;

    public IConnection CreateConnection()
    {
        var policy = Policy
            .Handle<BrokerUnreachableException>()
            .Or<SocketException>()
            .WaitAndRetry(
                retryCount: 5,
                sleepDurationProvider: attempt => TimeSpan.FromSeconds(Math.Pow(2, attempt)),
                onRetry: (ex, delay, attempt, _) =>
                    _logger.LogWarning("RabbitMQ connection attempt {Attempt} failed. Retrying in {Delay}s", attempt, delay.TotalSeconds)
            );

        return policy.Execute(() => _factory.CreateConnection());
    }
}
```

### Channel Exception Handling

```csharp
// Re-create channel if it's shut down
channel.ModelShutdown += (sender, ea) =>
{
    _logger.LogWarning("Channel shut down: {Reason}", ea.ReplyText);
    // Re-initialize consumer on new channel
    InitializeConsumer();
};
```

---

## Production Checklist

### Configuration
- [ ] All exchanges declared as `durable: true`
- [ ] All queues declared as `durable: true`
- [ ] All messages sent with `DeliveryMode = 2` (persistent)
- [ ] `AutomaticRecoveryEnabled = true` on connection factory
- [ ] `RequestedHeartbeat` set (60s recommended)
- [ ] `BasicQos(prefetchCount)` configured for each consumer

### Reliability
- [ ] Every queue has a Dead Letter Exchange configured
- [ ] DLQ exists and is monitored
- [ ] Retry logic with exponential backoff implemented
- [ ] Consumer uses `autoAck: false` (manual acknowledgment)
- [ ] Idempotency checks on all consumer handlers
- [ ] Publisher confirms enabled for critical messages

### Observability
- [ ] Structured logging on publish and consume
- [ ] Queue depth metrics collected
- [ ] DLQ depth alert configured
- [ ] Consumer count alert configured
- [ ] Message processing time tracked
- [ ] Correlation IDs propagated through messages

### Security
- [ ] Default `guest` user disabled/removed in production
- [ ] TLS enabled (`amqps://`)
- [ ] Per-service users with minimal permissions
- [ ] Virtual hosts used for environment separation

---

**Previous:** [09 — Clean Architecture + CQRS ←](../09-clean-architecture-cqrs/README.md)  
**Next:** [11 — Advanced Topics →](../11-advanced-topics/README.md)
