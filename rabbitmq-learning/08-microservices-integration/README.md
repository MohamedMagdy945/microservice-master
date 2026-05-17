# 08 — Microservices Integration

> **Goal:** Use RabbitMQ to connect real microservices — order, inventory, payment, and notification.

---

## Example Architecture

```
User Places Order
       │
       ▼
┌─────────────────┐
│  Order Service  │ ──publishes──► "order.created" event
└─────────────────┘
                              │
              ┌───────────────┼────────────────┐
              ▼               ▼                ▼
  ┌───────────────────┐  ┌──────────────┐  ┌─────────────────────┐
  │ Inventory Service │  │Payment Svc   │  │ Notification Service │
  │ (updates stock)   │  │(charges card)│  │ (sends email/SMS)    │
  └───────────────────┘  └──────────────┘  └─────────────────────┘
              │               │
              ▼               ▼
   "inventory.updated"  "payment.processed"
              │               │
              └───────┬───────┘
                      ▼
           ┌─────────────────────┐
           │ Notification Service│  (listens to all final events)
           └─────────────────────┘
```

---

## 1. Shared Message Contracts

Define message schemas in a shared library (NuGet package or shared project):

```csharp
// Shared/Events/OrderEvents.cs
public record OrderCreatedEvent
{
    public Guid   OrderId    { get; init; }
    public Guid   CustomerId { get; init; }
    public decimal Total     { get; init; }
    public string Region     { get; init; } = "global";
    public DateTime CreatedAt { get; init; } = DateTime.UtcNow;
    public List<OrderItem> Items { get; init; } = new();
}

public record OrderItem
{
    public string ProductId { get; init; } = "";
    public int    Quantity  { get; init; }
    public decimal Price    { get; init; }
}

// Shared/Events/PaymentEvents.cs
public record PaymentProcessedEvent
{
    public Guid   PaymentId  { get; init; }
    public Guid   OrderId    { get; init; }
    public decimal Amount    { get; init; }
    public string Status     { get; init; } = ""; // "success" | "failed"
}

// Shared/Events/InventoryEvents.cs
public record InventoryUpdatedEvent
{
    public string ProductId    { get; init; } = "";
    public int    NewQuantity  { get; init; }
    public bool   LowStock     { get; init; }
}
```

---

## 2. Order Service (Publisher)

```csharp
// OrderService/Services/OrderService.cs
public class OrderService
{
    private readonly IOrderRepository _repo;
    private readonly IMessagePublisher _publisher;

    public OrderService(IOrderRepository repo, IMessagePublisher publisher)
    {
        _repo      = repo;
        _publisher = publisher;
    }

    public async Task<Order> CreateOrderAsync(CreateOrderRequest request)
    {
        // Business logic
        var order = new Order
        {
            OrderId    = Guid.NewGuid(),
            CustomerId = request.CustomerId,
            Items      = request.Items,
            Total      = request.Items.Sum(i => i.Price * i.Quantity),
            Status     = "Pending"
        };

        // Persist to database
        await _repo.SaveAsync(order);

        // Publish event — other services react asynchronously
        var @event = new OrderCreatedEvent
        {
            OrderId    = order.OrderId,
            CustomerId = order.CustomerId,
            Total      = order.Total,
            Region     = request.Region,
            Items      = request.Items.Select(i => new OrderItem
            {
                ProductId = i.ProductId,
                Quantity  = i.Quantity,
                Price     = i.Price
            }).ToList()
        };

        _publisher.Publish("orders", "order.created", @event);

        return order;
    }
}
```

```csharp
// OrderService/Messaging/MessagePublisher.cs
public class MessagePublisher : IMessagePublisher
{
    private readonly IModel _channel;

    public MessagePublisher(RabbitMqConnectionFactory factory)
    {
        _channel = factory.CreateChannel();
        _channel.ExchangeDeclare("orders", ExchangeType.Topic, durable: true);
    }

    public void Publish<T>(string exchange, string routingKey, T message)
    {
        var json  = JsonSerializer.Serialize(message);
        var body  = Encoding.UTF8.GetBytes(json);
        var props = _channel.CreateBasicProperties();

        props.ContentType  = "application/json";
        props.DeliveryMode = 2; // persistent
        props.MessageId    = Guid.NewGuid().ToString();
        props.Timestamp    = new AmqpTimestamp(DateTimeOffset.UtcNow.ToUnixTimeSeconds());

        _channel.BasicPublish(exchange, routingKey, props, body);
    }
}
```

---

## 3. Inventory Service (Consumer)

```csharp
// InventoryService/Consumers/OrderCreatedConsumer.cs
public class OrderCreatedConsumer : BackgroundService
{
    private readonly IModel _channel;
    private readonly IInventoryRepository _inventory;
    private readonly IMessagePublisher _publisher;

    protected override Task ExecuteAsync(CancellationToken stoppingToken)
    {
        _channel.ExchangeDeclare("orders",    ExchangeType.Topic, durable: true);
        _channel.ExchangeDeclare("inventory", ExchangeType.Topic, durable: true);

        _channel.QueueDeclare("inventory.order-created", durable: true,
            exclusive: false, autoDelete: false,
            arguments: new Dictionary<string, object>
            {
                { "x-dead-letter-exchange", "dlx" }
            });

        _channel.QueueBind("inventory.order-created", "orders", "order.created.#");
        _channel.BasicQos(0, 10, false); // process up to 10 concurrent messages

        var consumer = new EventingBasicConsumer(_channel);
        consumer.Received += OnOrderCreated;
        _channel.BasicConsume("inventory.order-created", autoAck: false, consumer);

        return Task.CompletedTask;
    }

    private async void OnOrderCreated(object? sender, BasicDeliverEventArgs ea)
    {
        try
        {
            var json  = Encoding.UTF8.GetString(ea.Body.ToArray());
            var order = JsonSerializer.Deserialize<OrderCreatedEvent>(json)!;

            // Update inventory for each item
            foreach (var item in order.Items)
            {
                var product = await _inventory.GetAsync(item.ProductId);
                product.Quantity -= item.Quantity;
                await _inventory.UpdateAsync(product);

                // Publish inventory updated event
                _publisher.Publish("inventory", "inventory.updated", new InventoryUpdatedEvent
                {
                    ProductId   = item.ProductId,
                    NewQuantity = product.Quantity,
                    LowStock    = product.Quantity < 10
                });
            }

            _channel.BasicAck(ea.DeliveryTag, false);
        }
        catch (Exception ex)
        {
            _channel.BasicNack(ea.DeliveryTag, false, requeue: false); // → DLQ
        }
    }
}
```

---

## 4. Payment Service (Consumer + Publisher)

```csharp
// PaymentService/Consumers/OrderCreatedConsumer.cs
private async void OnOrderCreated(object? sender, BasicDeliverEventArgs ea)
{
    var order = Deserialize<OrderCreatedEvent>(ea.Body.ToArray());

    try
    {
        // Process payment
        var result = await _paymentGateway.ChargeAsync(order.CustomerId, order.Total);

        // Publish result
        _publisher.Publish("payments", "payment.processed", new PaymentProcessedEvent
        {
            PaymentId = Guid.NewGuid(),
            OrderId   = order.OrderId,
            Amount    = order.Total,
            Status    = result.Success ? "success" : "failed"
        });

        _channel.BasicAck(ea.DeliveryTag, false);
    }
    catch (PaymentGatewayException ex)
    {
        // Transient failure → retry
        _channel.BasicNack(ea.DeliveryTag, false, requeue: true);
    }
    catch (Exception ex)
    {
        // Permanent failure → DLQ
        _channel.BasicNack(ea.DeliveryTag, false, requeue: false);
    }
}
```

---

## 5. Notification Service (Multi-Event Consumer)

```csharp
// NotificationService/Consumers/NotificationConsumer.cs
public class NotificationConsumer : BackgroundService
{
    protected override Task ExecuteAsync(CancellationToken stoppingToken)
    {
        // Listen to multiple exchanges
        BindQueue("order-notifications",   "orders",    "order.created.#");
        BindQueue("payment-notifications", "payments",  "payment.processed.#");
        BindQueue("inventory-alerts",      "inventory", "inventory.updated.#");

        StartConsuming("order-notifications",   OnOrderCreated);
        StartConsuming("payment-notifications", OnPaymentProcessed);
        StartConsuming("inventory-alerts",      OnInventoryUpdated);

        return Task.CompletedTask;
    }

    private void OnOrderCreated(object? sender, BasicDeliverEventArgs ea)
    {
        var order = Deserialize<OrderCreatedEvent>(ea.Body.ToArray());
        _emailService.Send(order.CustomerId, "Order Confirmed", $"Order #{order.OrderId} received!");
        _channel.BasicAck(ea.DeliveryTag, false);
    }

    private void OnPaymentProcessed(object? sender, BasicDeliverEventArgs ea)
    {
        var payment = Deserialize<PaymentProcessedEvent>(ea.Body.ToArray());
        if (payment.Status == "success")
            _emailService.Send(payment.OrderId, "Payment Received", $"${payment.Amount} charged.");
        else
            _emailService.Send(payment.OrderId, "Payment Failed", "Please update your payment method.");

        _channel.BasicAck(ea.DeliveryTag, false);
    }

    private void OnInventoryUpdated(object? sender, BasicDeliverEventArgs ea)
    {
        var inv = Deserialize<InventoryUpdatedEvent>(ea.Body.ToArray());
        if (inv.LowStock)
            _alertService.NotifyAdmins($"Low stock: {inv.ProductId} ({inv.NewQuantity} remaining)");

        _channel.BasicAck(ea.DeliveryTag, false);
    }
}
```

---

## 6. Docker Compose — Full Stack

```yaml
version: '3.8'

services:
  rabbitmq:
    image: rabbitmq:3-management
    ports: ["5672:5672", "15672:15672"]
    environment:
      RABBITMQ_DEFAULT_USER: admin
      RABBITMQ_DEFAULT_PASS: admin123

  order-service:
    build: ./OrderService
    environment:
      RabbitMQ__ConnectionString: "amqp://admin:admin123@rabbitmq:5672/"
    depends_on: [rabbitmq]

  inventory-service:
    build: ./InventoryService
    environment:
      RabbitMQ__ConnectionString: "amqp://admin:admin123@rabbitmq:5672/"
    depends_on: [rabbitmq]

  payment-service:
    build: ./PaymentService
    environment:
      RabbitMQ__ConnectionString: "amqp://admin:admin123@rabbitmq:5672/"
    depends_on: [rabbitmq]

  notification-service:
    build: ./NotificationService
    environment:
      RabbitMQ__ConnectionString: "amqp://admin:admin123@rabbitmq:5672/"
    depends_on: [rabbitmq]
```

---

## Exchange & Queue Topology

```
Exchanges:
  "orders"    (topic)    ← OrderService publishes here
  "payments"  (topic)    ← PaymentService publishes here
  "inventory" (topic)    ← InventoryService publishes here
  "dlx"       (direct)   ← Dead letter exchange

Queues:
  "inventory.order-created"   bound to "orders" / "order.created.#"
  "payment.order-created"     bound to "orders" / "order.created.#"
  "notification.orders"       bound to "orders" / "order.created.#"
  "notification.payments"     bound to "payments" / "payment.processed.#"
  "notification.inventory"    bound to "inventory" / "inventory.updated.#"
  "dead-letter-queue"         bound to "dlx" / "dlq"
```

---

**Previous:** [07 — Messaging Patterns ←](../07-messaging-patterns/README.md)  
**Next:** [09 — Clean Architecture + CQRS →](../09-clean-architecture-cqrs/README.md)
