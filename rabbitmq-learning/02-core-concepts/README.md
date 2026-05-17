# 02 — RabbitMQ Core Concepts

> **Goal:** Understand the internal components of RabbitMQ and how messages flow through the system.

---

## The Big Picture

```
┌─────────────────────────────────────────────────────────────┐
│                        RabbitMQ Broker                       │
│                                                              │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐              │
│  │ Exchange │───►│ Binding  │───►│  Queue   │              │
│  │          │    │(routing) │    │          │              │
│  └──────────┘    └──────────┘    └──────────┘              │
│       ▲                                  │                  │
└───────┼──────────────────────────────────┼──────────────────┘
        │                                  │
   ┌────┴─────┐                      ┌─────▼────┐
   │ Producer │                      │ Consumer │
   └──────────┘                      └──────────┘
```

---

## 1. Producer

A **producer** is any application that sends (publishes) messages to RabbitMQ. It never sends directly to a queue — it always publishes to an **exchange**.

```csharp
// A producer publishes a message
channel.BasicPublish(
    exchange: "my-exchange",    // where to send
    routingKey: "order.created", // how to route
    body: Encoding.UTF8.GetBytes("Hello!")
);
```

**Producer responsibilities:**
- Connect to RabbitMQ
- Declare the exchange (if not already created)
- Publish messages with a routing key
- Optionally wait for confirmation (publisher confirms)

---

## 2. Consumer

A **consumer** is any application that receives messages from a queue and processes them.

```csharp
// A consumer listens for messages
channel.BasicConsume(
    queue: "my-queue",
    autoAck: true,
    consumer: consumer
);
```

**Consumer responsibilities:**
- Connect to RabbitMQ
- Declare the queue (if not already created)
- Bind the queue to an exchange
- Process messages
- Acknowledge or reject messages

---

## 3. Queue

A **queue** is a buffer that stores messages until a consumer retrieves and processes them. Queues follow the **FIFO** principle (First In, First Out).

```
┌─────────────────────────────────────────┐
│                  Queue                   │
│  ┌───┐  ┌───┐  ┌───┐  ┌───┐  ┌───┐    │
│  │ 1 │  │ 2 │  │ 3 │  │ 4 │  │ 5 │    │ ◄── New messages added (tail)
│  └───┘  └───┘  └───┘  └───┘  └───┘    │
│    ▲                                    │
│  Consumer reads from here (head)        │
└─────────────────────────────────────────┘
```

### Queue Properties

| Property | Description | Default |
|----------|-------------|---------|
| `durable` | Survives RabbitMQ restart | `false` |
| `exclusive` | Only one consumer, deleted when connection closes | `false` |
| `autoDelete` | Deleted when last consumer disconnects | `false` |
| `arguments` | Extra settings (TTL, max length, DLX) | `null` |

```csharp
channel.QueueDeclare(
    queue: "order-queue",
    durable: true,      // survive restarts
    exclusive: false,   // multiple consumers allowed
    autoDelete: false,  // don't auto-delete
    arguments: null
);
```

### Queue Lifecycle
- Queues must be **declared** before use (idempotent — safe to call multiple times)
- If a queue with the same name already exists and settings match, nothing changes
- If settings differ, RabbitMQ throws an error

---

## 4. Exchange

An **exchange** receives messages from producers and routes them to queues based on rules. The producer never touches a queue directly.

```
Producer
    │
    ▼
Exchange ──[routing logic]──► Queue A
                          └──► Queue B
                          └──► Queue C
```

### Exchange Types Overview
| Type | Routing Strategy |
|------|-----------------|
| `direct` | Exact routing key match |
| `fanout` | Broadcast to all bound queues |
| `topic` | Pattern matching on routing key |
| `headers` | Match on message headers |

> See [03 — Exchange Types](../03-exchange-types/README.md) for full details.

### Default Exchange
RabbitMQ has a built-in **default exchange** (empty name `""`). When you publish to it with a routing key, it routes to the queue with that exact name.

```csharp
// Publishes directly to "my-queue" via default exchange
channel.BasicPublish(
    exchange: "",          // default exchange
    routingKey: "my-queue", // queue name = routing key
    body: body
);
```

---

## 5. Binding

A **binding** is the link between an exchange and a queue. It tells the exchange: *"Send messages with this routing key to that queue."*

```
Exchange ──[binding: routing_key = "order"]──► Queue
```

```csharp
channel.QueueBind(
    queue: "order-queue",       // destination queue
    exchange: "my-exchange",    // source exchange
    routingKey: "order.created" // binding key
);
```

**Key rules:**
- One queue can have multiple bindings (receive from multiple exchanges)
- One exchange can bind to multiple queues
- The binding key matches (or pattern-matches) the routing key from the producer

---

## 6. Routing Key

A **routing key** is a string set by the producer when publishing a message. The exchange uses it to decide which queue(s) the message goes to.

```
Producer publishes:
  exchange:    "orders-exchange"
  routing_key: "order.created.europe"
  body:        { orderId: 123 }

Exchange checks bindings:
  "order.created.europe" → matches binding → sent to "europe-orders-queue"
  "order.created.europe" → matches binding → sent to "all-orders-queue"
  "order.created.europe" → no match      → discarded (or sent to alternate exchange)
```

**Routing key conventions:**
- Dot-separated words: `order.created`, `user.registered.premium`
- All lowercase, descriptive
- More specific → less specific (left to right)

---

## 7. Virtual Hosts (vHosts)

Virtual hosts provide logical separation within a single RabbitMQ instance — like separate namespaces.

```
RabbitMQ Server
├── vhost: /              (default)
│   ├── exchanges
│   └── queues
├── vhost: /development
│   ├── exchanges
│   └── queues
└── vhost: /production
    ├── exchanges
    └── queues
```

Each vhost has its own exchanges, queues, bindings, and permissions. Users are granted access per vhost.

---

## 8. Channels

A **channel** is a virtual connection inside a TCP connection. Opening a new TCP connection is expensive, so RabbitMQ multiplexes multiple channels over a single connection.

```
TCP Connection
├── Channel 1  (producer)
├── Channel 2  (consumer 1)
├── Channel 3  (consumer 2)
└── Channel 4  (admin)
```

**Best practice:** Use one channel per thread. Channels are not thread-safe.

```csharp
var connection = factory.CreateConnection();
var channel = connection.CreateModel(); // create a channel
```

---

## 9. Message Acknowledgment (Ack/Nack)

RabbitMQ needs to know if a message was processed successfully.

### Auto Acknowledgment
Message is acknowledged the moment it's delivered (risky — if consumer crashes mid-processing, message is lost).

```csharp
channel.BasicConsume(queue: "q", autoAck: true, consumer: consumer);
```

### Manual Acknowledgment
Consumer explicitly tells RabbitMQ after processing.

```csharp
channel.BasicConsume(queue: "q", autoAck: false, consumer: consumer);

// In the handler:
// ✅ Success — remove from queue
channel.BasicAck(deliveryTag: ea.DeliveryTag, multiple: false);

// ❌ Failure — requeue for retry
channel.BasicNack(deliveryTag: ea.DeliveryTag, multiple: false, requeue: true);

// ❌ Failure — discard (or send to DLQ)
channel.BasicNack(deliveryTag: ea.DeliveryTag, multiple: false, requeue: false);
```

---

## Component Summary

| Component | Role | Analogy |
|-----------|------|---------|
| **Producer** | Sends messages | Mail sender |
| **Exchange** | Routes messages | Post office sorting room |
| **Binding** | Routing rule | Sorting rule |
| **Routing Key** | Message label | Address on envelope |
| **Queue** | Stores messages | Mailbox |
| **Consumer** | Receives messages | Mail recipient |
| **Channel** | Virtual connection | Phone extension |

---

**Previous:** [01 — Fundamentals ←](../01-fundamentals/README.md)  
**Next:** [03 — Exchange Types →](../03-exchange-types/README.md)
