# 09 — Clean Architecture + CQRS

> **Goal:** Integrate RabbitMQ messaging cleanly into a domain-driven, CQRS-structured .NET application.

---

## Architecture Overview

```
┌──────────────────────────────────────────────────────────┐
│                    Presentation Layer                      │
│               (API Controllers / gRPC)                    │
└─────────────────────────┬────────────────────────────────┘
                          │ Commands / Queries
┌─────────────────────────▼────────────────────────────────┐
│                   Application Layer                        │
│    Command Handlers | Query Handlers | Event Handlers      │
│    (MediatR pipeline, validation, authorization)           │
└──────┬──────────────────────────────────────┬────────────┘
       │ Domain Events                         │ Repository interfaces
┌──────▼──────────────┐           ┌───────────▼────────────┐
│    Domain Layer      │           │  Infrastructure Layer   │
│  Entities, Aggregates│           │  DB, RabbitMQ, Email    │
│  Domain Events       │           │  (implements interfaces)│
└──────────────────────┘           └────────────────────────┘
```

---

## 1. Domain Layer — Events

```csharp
// Domain/Events/IDomainEvent.cs
public interface IDomainEvent
{
    Guid   EventId   { get; }
    DateTime OccurredOn { get; }
}

// Domain/Events/OrderCreatedEvent.cs
public class OrderCreatedEvent : IDomainEvent
{
    public Guid     EventId    { get; } = Guid.NewGuid();
    public DateTime OccurredOn { get; } = DateTime.UtcNow;

    public Guid    OrderId    { get; }
    public Guid    CustomerId { get; }
    public decimal Total      { get; }
    public string  Region     { get; }

    public OrderCreatedEvent(Guid orderId, Guid customerId, decimal total, string region)
    {
        OrderId    = orderId;
        CustomerId = customerId;
        Total      = total;
        Region     = region;
    }
}
```

---

## 2. Domain Layer — Aggregate Root

```csharp
// Domain/Common/AggregateRoot.cs
public abstract class AggregateRoot
{
    private readonly List<IDomainEvent> _domainEvents = new();

    public IReadOnlyList<IDomainEvent> DomainEvents => _domainEvents.AsReadOnly();

    protected void RaiseDomainEvent(IDomainEvent domainEvent)
    {
        _domainEvents.Add(domainEvent);
    }

    public void ClearDomainEvents() => _domainEvents.Clear();
}

// Domain/Entities/Order.cs
public class Order : AggregateRoot
{
    public Guid    OrderId    { get; private set; }
    public Guid    CustomerId { get; private set; }
    public decimal Total      { get; private set; }
    public string  Status     { get; private set; } = "Pending";
    public string  Region     { get; private set; } = "";

    private Order() { } // EF Core

    public static Order Create(Guid customerId, decimal total, string region)
    {
        var order = new Order
        {
            OrderId    = Guid.NewGuid(),
            CustomerId = customerId,
            Total      = total,
            Region     = region,
            Status     = "Pending"
        };

        // Raise domain event — will be dispatched after persistence
        order.RaiseDomainEvent(new OrderCreatedEvent(
            order.OrderId, order.CustomerId, order.Total, order.Region
        ));

        return order;
    }

    public void Complete()
    {
        Status = "Completed";
        RaiseDomainEvent(new OrderCompletedEvent(OrderId));
    }
}
```

---

## 3. Application Layer — Command

```csharp
// Application/Commands/CreateOrder/CreateOrderCommand.cs
public record CreateOrderCommand : IRequest<CreateOrderResult>
{
    public Guid    CustomerId { get; init; }
    public decimal Total      { get; init; }
    public string  Region     { get; init; } = "";
    public List<OrderItemDto> Items { get; init; } = new();
}

public record CreateOrderResult(Guid OrderId, string Status);
```

---

## 4. Application Layer — Command Handler

```csharp
// Application/Commands/CreateOrder/CreateOrderCommandHandler.cs
public class CreateOrderCommandHandler : IRequestHandler<CreateOrderCommand, CreateOrderResult>
{
    private readonly IOrderRepository       _orderRepository;
    private readonly IDomainEventDispatcher _eventDispatcher;

    public CreateOrderCommandHandler(
        IOrderRepository orderRepository,
        IDomainEventDispatcher eventDispatcher)
    {
        _orderRepository = orderRepository;
        _eventDispatcher = eventDispatcher;
    }

    public async Task<CreateOrderResult> Handle(
        CreateOrderCommand command, CancellationToken cancellationToken)
    {
        // 1. Create domain entity (raises internal domain event)
        var order = Order.Create(command.CustomerId, command.Total, command.Region);

        // 2. Persist to database
        await _orderRepository.AddAsync(order, cancellationToken);
        await _orderRepository.SaveChangesAsync(cancellationToken);

        // 3. Dispatch domain events (publishes to RabbitMQ)
        await _eventDispatcher.DispatchAsync(order.DomainEvents, cancellationToken);
        order.ClearDomainEvents();

        return new CreateOrderResult(order.OrderId, order.Status);
    }
}
```

---

## 5. Infrastructure — Domain Event Dispatcher

```csharp
// Infrastructure/Messaging/DomainEventDispatcher.cs
public class DomainEventDispatcher : IDomainEventDispatcher
{
    private readonly IMessagePublisher _publisher;
    private readonly ILogger<DomainEventDispatcher> _logger;

    // Maps domain event types to exchange + routing key
    private static readonly Dictionary<Type, (string Exchange, string RoutingKey)> _routingMap = new()
    {
        [typeof(OrderCreatedEvent)]   = ("orders",   "order.created"),
        [typeof(OrderCompletedEvent)] = ("orders",   "order.completed"),
        [typeof(OrderCancelledEvent)] = ("orders",   "order.cancelled"),
    };

    public DomainEventDispatcher(IMessagePublisher publisher, ILogger<DomainEventDispatcher> logger)
    {
        _publisher = publisher;
        _logger    = logger;
    }

    public async Task DispatchAsync(
        IReadOnlyList<IDomainEvent> events, CancellationToken cancellationToken = default)
    {
        foreach (var domainEvent in events)
        {
            var eventType = domainEvent.GetType();

            if (!_routingMap.TryGetValue(eventType, out var routing))
            {
                _logger.LogWarning("No routing found for event {EventType}", eventType.Name);
                continue;
            }

            _logger.LogInformation(
                "Publishing {EventType} with id {EventId} to {Exchange}/{RoutingKey}",
                eventType.Name, domainEvent.EventId, routing.Exchange, routing.RoutingKey);

            _publisher.Publish(routing.Exchange, routing.RoutingKey, domainEvent);
        }
    }
}
```

---

## 6. Application Layer — Event Handler (Consumer Side)

```csharp
// Application/Events/OrderCreated/SendOrderConfirmationEmailHandler.cs
public class SendOrderConfirmationEmailHandler : IEventHandler<OrderCreatedEvent>
{
    private readonly IEmailService _emailService;
    private readonly ICustomerRepository _customers;

    public SendOrderConfirmationEmailHandler(IEmailService emailService, ICustomerRepository customers)
    {
        _emailService = emailService;
        _customers    = customers;
    }

    public async Task HandleAsync(OrderCreatedEvent @event, CancellationToken cancellationToken)
    {
        var customer = await _customers.GetByIdAsync(@event.CustomerId, cancellationToken);
        if (customer is null) return;

        await _emailService.SendAsync(new EmailMessage
        {
            To      = customer.Email,
            Subject = $"Order #{@event.OrderId} Confirmed",
            Body    = $"Thank you! Your order of ${@event.Total:F2} has been received."
        });
    }
}
```

---

## 7. Infrastructure — Consumer (Bridges RabbitMQ → Handlers)

```csharp
// Infrastructure/Messaging/Consumers/OrderCreatedConsumer.cs
public class OrderCreatedConsumer : BackgroundService
{
    private readonly IServiceProvider _services; // for scoped handler resolution
    private readonly IModel _channel;

    protected override Task ExecuteAsync(CancellationToken stoppingToken)
    {
        _channel.ExchangeDeclare("orders", ExchangeType.Topic, durable: true);
        _channel.QueueDeclare("notification.order-created", durable: true,
            exclusive: false, autoDelete: false);
        _channel.QueueBind("notification.order-created", "orders", "order.created.#");
        _channel.BasicQos(0, 5, false);

        var consumer = new EventingBasicConsumer(_channel);
        consumer.Received += OnMessageReceived;
        _channel.BasicConsume("notification.order-created", autoAck: false, consumer);

        return Task.CompletedTask;
    }

    private async void OnMessageReceived(object? sender, BasicDeliverEventArgs ea)
    {
        try
        {
            var json  = Encoding.UTF8.GetString(ea.Body.ToArray());
            var @event = JsonSerializer.Deserialize<OrderCreatedEvent>(json)!;

            // Resolve handler from DI (scoped — gets its own DbContext etc.)
            using var scope   = _services.CreateScope();
            var handler = scope.ServiceProvider.GetRequiredService<IEventHandler<OrderCreatedEvent>>();
            await handler.HandleAsync(@event, CancellationToken.None);

            _channel.BasicAck(ea.DeliveryTag, false);
        }
        catch (Exception ex)
        {
            _channel.BasicNack(ea.DeliveryTag, false, requeue: false);
        }
    }
}
```

---

## 8. Full Registration (Program.cs)

```csharp
// Program.cs
builder.Services.AddMediatR(cfg =>
    cfg.RegisterServicesFromAssembly(typeof(CreateOrderCommandHandler).Assembly));

// Domain event infrastructure
builder.Services.AddSingleton<RabbitMqConnectionFactory>();
builder.Services.AddSingleton<IMessagePublisher, MessagePublisher>();
builder.Services.AddSingleton<IDomainEventDispatcher, DomainEventDispatcher>();

// Event handlers (scoped — they use DbContext)
builder.Services.AddScoped<IEventHandler<OrderCreatedEvent>, SendOrderConfirmationEmailHandler>();

// Consumers (background services)
builder.Services.AddHostedService<OrderCreatedConsumer>();
```

---

## Flow Summary

```
HTTP POST /orders
    │
    ▼
CreateOrderCommand (MediatR)
    │
    ▼
CreateOrderCommandHandler
    ├── Order.Create() → raises OrderCreatedEvent (in memory)
    ├── Save to DB
    └── DomainEventDispatcher.DispatchAsync()
                │
                ▼
          MessagePublisher
                │
         [orders exchange]
                │
         "order.created"
                │
    ┌───────────┼───────────┐
    ▼           ▼           ▼
EmailConsumer  InventoryConsumer  AnalyticsConsumer
    │               │                   │
Handler.HandleAsync Handler.HandleAsync Handler.HandleAsync
```

---

**Previous:** [08 — Microservices Integration ←](../08-microservices-integration/README.md)  
**Next:** [10 — Production Best Practices →](../10-production-best-practices/README.md)
