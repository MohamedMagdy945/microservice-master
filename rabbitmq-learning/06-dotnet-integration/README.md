# 06 — .NET Integration Basics

> **Goal:** Build a proper, reusable RabbitMQ integration layer in .NET with connection management, producer, and consumer patterns.

---

## Install the Library

```bash
dotnet add package RabbitMQ.Client
```

Current stable version: **6.x** (supports .NET 6/7/8)

---

## 1. Connection Management

### Basic Factory Setup

```csharp
var factory = new ConnectionFactory
{
    HostName         = "localhost",
    Port             = 5672,
    UserName         = "guest",
    Password         = "guest",
    VirtualHost      = "/",
    
    // Important for production
    AutomaticRecoveryEnabled = true,          // auto-reconnect on failure
    NetworkRecoveryInterval  = TimeSpan.FromSeconds(10),
    RequestedHeartbeat       = TimeSpan.FromSeconds(60),
    
    // Connection timeout
    RequestedConnectionTimeout = TimeSpan.FromSeconds(30),
};
```

### Using URI (Recommended for Config Files)

```csharp
var factory = new ConnectionFactory
{
    Uri = new Uri("amqp://guest:guest@localhost:5672/")
};
```

### Singleton Connection (Best Practice)

```csharp
// Register as Singleton in DI — one connection for the app
public class RabbitMqConnectionFactory : IDisposable
{
    private readonly IConnection _connection;

    public RabbitMqConnectionFactory(IConfiguration config)
    {
        var factory = new ConnectionFactory
        {
            Uri = new Uri(config["RabbitMQ:ConnectionString"]!),
            AutomaticRecoveryEnabled = true
        };
        _connection = factory.CreateConnection();
    }

    public IModel CreateChannel() => _connection.CreateModel();

    public void Dispose() => _connection?.Dispose();
}
```

---

## 2. Producer — Full Implementation

```csharp
public class MessageProducer : IDisposable
{
    private readonly IModel _channel;

    public MessageProducer(RabbitMqConnectionFactory connectionFactory)
    {
        _channel = connectionFactory.CreateChannel();
        SetupExchangesAndQueues();
    }

    private void SetupExchangesAndQueues()
    {
        // Declare exchange
        _channel.ExchangeDeclare(
            exchange: "orders",
            type: ExchangeType.Topic,
            durable: true,
            autoDelete: false
        );

        // Declare queue
        _channel.QueueDeclare(
            queue: "order-created",
            durable: true,
            exclusive: false,
            autoDelete: false,
            arguments: new Dictionary<string, object>
            {
                { "x-message-ttl", 86400000 }  // 24hr TTL in milliseconds
            }
        );

        // Bind queue to exchange
        _channel.QueueBind("order-created", "orders", routingKey: "order.created.#");
    }

    public void Publish<T>(string exchange, string routingKey, T message)
    {
        var json = JsonSerializer.Serialize(message);
        var body = Encoding.UTF8.GetBytes(json);

        // Set message properties
        var properties = _channel.CreateBasicProperties();
        properties.ContentType     = "application/json";
        properties.DeliveryMode    = 2;             // 2 = persistent (survives restart)
        properties.MessageId       = Guid.NewGuid().ToString();
        properties.Timestamp       = new AmqpTimestamp(DateTimeOffset.UtcNow.ToUnixTimeSeconds());
        properties.CorrelationId   = Guid.NewGuid().ToString(); // for tracing

        _channel.BasicPublish(
            exchange:        exchange,
            routingKey:      routingKey,
            basicProperties: properties,
            body:            body
        );
    }

    public void Dispose() => _channel?.Dispose();
}
```

### Usage
```csharp
var order = new { OrderId = 123, CustomerId = 456, Total = 99.99m };
producer.Publish("orders", "order.created.eu", order);
```

---

## 3. Consumer — Full Implementation

```csharp
public class MessageConsumer : BackgroundService  // Runs as hosted service
{
    private readonly RabbitMqConnectionFactory _connectionFactory;
    private readonly ILogger<MessageConsumer> _logger;
    private IModel? _channel;

    public MessageConsumer(RabbitMqConnectionFactory connectionFactory,
                           ILogger<MessageConsumer> logger)
    {
        _connectionFactory = connectionFactory;
        _logger = logger;
    }

    protected override Task ExecuteAsync(CancellationToken stoppingToken)
    {
        stoppingToken.ThrowIfCancellationRequested();

        _channel = _connectionFactory.CreateChannel();

        // Process one message at a time (fair dispatch)
        _channel.BasicQos(prefetchSize: 0, prefetchCount: 1, global: false);

        var consumer = new EventingBasicConsumer(_channel);
        consumer.Received += OnMessageReceived;

        _channel.BasicConsume(
            queue:    "order-created",
            autoAck:  false,    // ALWAYS false in production
            consumer: consumer
        );

        return Task.CompletedTask;
    }

    private void OnMessageReceived(object? sender, BasicDeliverEventArgs ea)
    {
        var deliveryTag = ea.DeliveryTag;
        try
        {
            var json    = Encoding.UTF8.GetString(ea.Body.ToArray());
            var order   = JsonSerializer.Deserialize<OrderCreatedEvent>(json)!;

            _logger.LogInformation("Processing order {OrderId}", order.OrderId);

            // TODO: Process the order
            ProcessOrder(order);

            // ✅ Acknowledge — remove from queue
            _channel!.BasicAck(deliveryTag, multiple: false);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to process message {DeliveryTag}", deliveryTag);

            // ❌ Nack — requeue for retry (set requeue: false to send to DLQ)
            _channel!.BasicNack(deliveryTag, multiple: false, requeue: true);
        }
    }

    public override void Dispose()
    {
        _channel?.Dispose();
        base.Dispose();
    }
}
```

---

## 4. Register in Dependency Injection

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

// Register connection factory as singleton
builder.Services.AddSingleton<RabbitMqConnectionFactory>();

// Register producer as singleton (shared channel — thread-safe for publishing)
builder.Services.AddSingleton<MessageProducer>();

// Register consumer as hosted background service
builder.Services.AddHostedService<MessageConsumer>();

var app = builder.Build();
app.Run();
```

---

## 5. Message Properties Reference

```csharp
var props = channel.CreateBasicProperties();

props.ContentType     = "application/json";  // payload format
props.ContentEncoding = "utf-8";             // encoding
props.DeliveryMode    = 2;                   // 1=transient, 2=persistent
props.Priority        = 5;                   // 0-9 (requires queue priority setting)
props.MessageId       = Guid.NewGuid().ToString(); // unique ID
props.CorrelationId   = requestId;           // link response to request
props.ReplyTo         = "response-queue";    // for RPC pattern
props.Expiration      = "60000";             // TTL in milliseconds (string!)
props.Timestamp       = new AmqpTimestamp(DateTimeOffset.UtcNow.ToUnixTimeSeconds());
props.Type            = "OrderCreatedEvent"; // message type hint
props.AppId           = "order-service";     // which app sent it

// Custom headers
props.Headers = new Dictionary<string, object>
{
    { "x-retry-count", 0 },
    { "x-source-region", "eu-west" }
};
```

---

## 6. Publisher Confirms (Reliable Publishing)

Guarantees the broker received and persisted your message:

```csharp
// Enable confirm mode on channel
channel.ConfirmSelect();

// Publish
channel.BasicPublish(exchange, routingKey, props, body);

// Wait for confirmation (blocking)
bool confirmed = channel.WaitForConfirms(timeout: TimeSpan.FromSeconds(5));

if (!confirmed)
{
    // Message may not have been received — retry or log
    throw new Exception("Message not confirmed by broker");
}
```

**Async confirms (higher throughput):**
```csharp
channel.ConfirmSelect();
channel.BasicAcks  += (sender, ea) => Console.WriteLine($"Confirmed: {ea.DeliveryTag}");
channel.BasicNacks += (sender, ea) => Console.WriteLine($"Failed: {ea.DeliveryTag}");

channel.BasicPublish(...);
channel.WaitForConfirmsOrDie(TimeSpan.FromSeconds(5));
```

---

## 7. appsettings.json Configuration

```json
{
  "RabbitMQ": {
    "ConnectionString": "amqp://guest:guest@localhost:5672/",
    "Exchange": "orders",
    "Queue": "order-created",
    "RoutingKey": "order.created.#"
  }
}
```

---

## Summary Checklist

- [x] Install `RabbitMQ.Client` NuGet package
- [x] Create `ConnectionFactory` with your settings
- [x] Reuse a single `IConnection` per application
- [x] Create one `IModel` (channel) per thread/operation
- [x] Enable `AutomaticRecoveryEnabled = true`
- [x] Always use `DeliveryMode = 2` for persistent messages
- [x] Always use `autoAck: false` and call `BasicAck/BasicNack` manually
- [x] Use `BasicQos(prefetchCount: 1)` to prevent overloading consumers
- [x] Register connection as `Singleton` in DI
- [x] Register consumer as `BackgroundService`

---

**Previous:** [05 — Hello World ←](../05-hello-world/README.md)  
**Next:** [07 — Messaging Patterns →](../07-messaging-patterns/README.md)
