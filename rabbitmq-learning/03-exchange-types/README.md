# 03 — Exchange Types

> **Goal:** Master the four exchange types and know when to use each routing strategy.

---

## Overview

The exchange type determines *how* messages are routed to queues.

```
┌────────────┬────────────────────────────────────────┐
│ Type       │ Routes to queue when…                  │
├────────────┼────────────────────────────────────────┤
│ direct     │ routing key exactly matches binding key │
│ fanout     │ always (ignores routing key)            │
│ topic      │ routing key matches a wildcard pattern  │
│ headers    │ message headers match binding arguments │
└────────────┴────────────────────────────────────────┘
```

---

## 1. Direct Exchange

**Routes messages to queues where the binding key exactly matches the routing key.**

```
Producer
  routingKey: "error"
       │
       ▼
  [Direct Exchange]
       ├── binding "error"   ──► error-queue   ✅
       ├── binding "warning" ──► warning-queue ❌
       └── binding "info"    ──► info-queue    ❌
```

### Use Cases
- Task distribution to specific workers
- Log routing by severity level
- Sending to one specific service

### .NET Example

```csharp
// Declare
channel.ExchangeDeclare("logs-direct", ExchangeType.Direct, durable: true);
channel.QueueDeclare("error-queue", durable: true, exclusive: false, autoDelete: false);
channel.QueueBind("error-queue", "logs-direct", routingKey: "error");

// Producer
channel.BasicPublish("logs-direct", routingKey: "error", body: body);
channel.BasicPublish("logs-direct", routingKey: "warning", body: body);

// Consumer (only receives "error" messages)
channel.BasicConsume("error-queue", autoAck: false, consumer: consumer);
```

### Diagram
```
"error"   ──► Direct Exchange ──► [error-queue]   ──► ErrorLogger
"warning" ──► Direct Exchange ──► [warning-queue] ──► WarningLogger
"info"    ──► Direct Exchange ──► [info-queue]    ──► InfoLogger
```

> **Note:** The default exchange (`""`) is a direct exchange where the routing key equals the queue name.

---

## 2. Fanout Exchange

**Broadcasts every message to ALL queues bound to the exchange, ignoring the routing key.**

```
Producer
  routingKey: (ignored)
       │
       ▼
  [Fanout Exchange]
       ├──────────────────► queue-1 ──► Consumer A
       ├──────────────────► queue-2 ──► Consumer B
       └──────────────────► queue-3 ──► Consumer C
```

### Use Cases
- Broadcasting notifications (all services get the same message)
- Cache invalidation across multiple instances
- Live sports score updates
- System-wide announcements

### .NET Example

```csharp
// Declare
channel.ExchangeDeclare("notifications", ExchangeType.Fanout, durable: true);

// Each consumer creates its own queue and binds to the exchange
var emailQueue = channel.QueueDeclare("email-notifications", durable: true, exclusive: false, autoDelete: false);
var smsQueue   = channel.QueueDeclare("sms-notifications",   durable: true, exclusive: false, autoDelete: false);

channel.QueueBind(emailQueue.QueueName, "notifications", routingKey: ""); // key ignored
channel.QueueBind(smsQueue.QueueName,   "notifications", routingKey: "");

// Producer — routing key is irrelevant
channel.BasicPublish("notifications", routingKey: "", body: body);
```

### Diagram
```
OrderCreated Event
       │
       ▼
  [Fanout Exchange: "order-events"]
       ├──► [email-queue]    ──► EmailService      (sends confirmation email)
       ├──► [inventory-queue]──► InventoryService  (updates stock)
       └──► [analytics-queue]──► AnalyticsService  (records metrics)
```

---

## 3. Topic Exchange

**Routes messages based on wildcard pattern matching on the routing key.**

Routing keys use dot-separated words: `word1.word2.word3`

### Wildcard Rules
| Symbol | Matches |
|--------|---------|
| `*` | Exactly ONE word |
| `#` | Zero or MORE words |

```
Routing Key: "order.created.europe"

Binding Pattern: "order.created.europe"  ✅ exact match
Binding Pattern: "order.created.*"       ✅ * matches "europe"
Binding Pattern: "order.#"              ✅ # matches "created.europe"
Binding Pattern: "#"                    ✅ # matches everything
Binding Pattern: "order.*.asia"         ❌ "europe" ≠ "asia"
Binding Pattern: "order.created"        ❌ too short, missing "europe"
```

### Use Cases
- Multi-level log routing (by app, level, environment)
- Geographic message routing
- Feature-flag based routing
- Multi-tenant applications

### .NET Example

```csharp
// Declare
channel.ExchangeDeclare("app-logs", ExchangeType.Topic, durable: true);

channel.QueueDeclare("all-errors",    durable: true, exclusive: false, autoDelete: false);
channel.QueueDeclare("order-logs",    durable: true, exclusive: false, autoDelete: false);
channel.QueueDeclare("critical-logs", durable: true, exclusive: false, autoDelete: false);

// Bindings
channel.QueueBind("all-errors",    "app-logs", "*.error.#");     // any service, error level
channel.QueueBind("order-logs",    "app-logs", "order.#");       // all order logs
channel.QueueBind("critical-logs", "app-logs", "#.critical.#");  // critical anywhere

// Producer
channel.BasicPublish("app-logs", "order.error.production", body: body);
// ↑ Goes to: all-errors ✅, order-logs ✅, critical-logs ❌
```

### Diagram
```
"order.error.production"  ──►  [Topic Exchange]
                                  ├── "*.error.#"    ──► all-errors-queue    ✅
                                  ├── "order.#"      ──► order-logs-queue    ✅
                                  └── "#.critical.#" ──► critical-logs-queue ❌

"payment.critical.eu"     ──►  [Topic Exchange]
                                  ├── "*.error.#"    ──► all-errors-queue    ❌
                                  ├── "order.#"      ──► order-logs-queue    ❌
                                  └── "#.critical.#" ──► critical-logs-queue ✅
```

---

## 4. Headers Exchange

**Routes based on message header attributes instead of routing key.** The routing key is completely ignored.

Binding arguments include special key `x-match`:
- `x-match: all` → ALL specified headers must match
- `x-match: any` → AT LEAST ONE header must match

### Use Cases
- Content-type-based routing
- Priority-based routing
- Region + format combinations
- When routing key is insufficient

### .NET Example

```csharp
// Declare
channel.ExchangeDeclare("content-router", ExchangeType.Headers, durable: true);
channel.QueueDeclare("pdf-queue",  durable: true, exclusive: false, autoDelete: false);
channel.QueueDeclare("xml-queue",  durable: true, exclusive: false, autoDelete: false);

// Bind with header matching
var pdfArgs = new Dictionary<string, object>
{
    { "x-match", "all" },       // ALL must match
    { "format", "pdf" },
    { "type", "report" }
};
channel.QueueBind("pdf-queue", "content-router", routingKey: "", pdfArgs);

// Producer — set headers on message
var props = channel.CreateBasicProperties();
props.Headers = new Dictionary<string, object>
{
    { "format", "pdf" },
    { "type", "report" },
    { "region", "eu" }
};
channel.BasicPublish("content-router", routingKey: "", basicProperties: props, body: body);
// ↑ Goes to pdf-queue ✅ (format=pdf ✅, type=report ✅)
```

---

## Choosing the Right Exchange Type

```
Do you need to send to ONE specific queue?
    └── YES → Direct Exchange

Do you need to broadcast to ALL queues?
    └── YES → Fanout Exchange

Do you need pattern-based routing on a hierarchical key?
    └── YES → Topic Exchange

Do you need routing based on message metadata/attributes?
    └── YES → Headers Exchange
```

### Quick Reference

| Exchange | Routing Key | Wildcards | Broadcast | Complexity |
|----------|------------|-----------|-----------|-----------|
| Default  | = queue name| ❌ | ❌ | Minimal |
| Direct   | Exact match | ❌ | ❌ | Low |
| Fanout   | Ignored     | ❌ | ✅ | Low |
| Topic    | Pattern     | ✅ (`*`,`#`) | Partial | Medium |
| Headers  | Ignored     | ❌ | Partial | Medium |

---

**Previous:** [02 — Core Concepts ←](../02-core-concepts/README.md)  
**Next:** [04 — Environment Setup →](../04-environment-setup/README.md)
