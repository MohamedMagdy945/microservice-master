# Event Sourcing Pattern

## Introduction

Event Sourcing is an architectural pattern where the system stores all changes to application state as a sequence of events instead of storing only the current state.

Traditional systems store:
- Current data only

Event Sourcing stores:
- Every state change
- Every action that happened in the system

The current state is rebuilt by replaying events.

---

# Traditional Data Storage

Traditional applications usually store only the latest state.

Example:

```text
Account Balance = 500$
```

When balance changes:
- Old value is overwritten
- History may be lost

---

# Event Sourcing Storage

Instead of storing current state directly, the system stores events.

Example:

```text
MoneyDeposited +1000$
MoneyWithdrawn -200$
MoneyWithdrawn -300$
```

Current balance is calculated by replaying events:

```text
1000 - 200 - 300 = 500$
```

---

# What is an Event?

An event represents:
- Something that already happened in the system

Events are:
- Immutable
- Historical facts

Examples:

```text
OrderCreated
PaymentCompleted
ProductAdded
UserRegistered
InventoryReduced
```

Events should describe the past.

Correct:

```text
OrderCreated
```

Incorrect:

```text
CreateOrder
```

---

# Core Idea of Event Sourcing

Instead of:

```text
Store Current State
```

We do:

```text
Store State Changes
```

The current application state is reconstructed from events.

---

# Event Sourcing Architecture

```text
Client
   ↓
Command
   ↓
Validate Business Rules
   ↓
Generate Event
   ↓
Store Event
   ↓
Replay Events → Build Current State
```

---

# Example: Bank Account System

## Step 1

Account created:

```text
AccountCreated
```

---

## Step 2

Deposit money:

```text
MoneyDeposited +1000$
```

---

## Step 3

Withdraw money:

```text
MoneyWithdrawn -300$
```

---

# Event Store

The database contains:

| Event ID | Event Type | Data |
|----------|-------------|------|
| 1 | AccountCreated | AccountId=1 |
| 2 | MoneyDeposited | Amount=1000 |
| 3 | MoneyWithdrawn | Amount=300 |

---

# Rebuilding State

System rebuilds account balance by replaying events:

```text
0 + 1000 - 300 = 700$
```

---

# Event Replay

Replay means:
- Reading all events in order
- Applying them sequentially

This recreates:
- Current state
- Historical state

---

# Benefits of Event Sourcing

## 1. Complete Audit History

Every change is permanently stored.

Example:

```text
Who changed balance?
When?
Why?
```

Everything is traceable.

---

## 2. Time Travel

You can rebuild system state at any point in time.

Example:

```text
Show account balance yesterday at 3 PM
```

---

## 3. Better Debugging

Historical events help analyze problems.

---

## 4. Natural Fit for Event-Driven Systems

Works well with:
- CQRS
- Kafka
- RabbitMQ
- Microservices

---

## 5. Easy Integration

Other services can subscribe to events.

Example:

```text
OrderCreated Event
```

Used by:
- Payment Service
- Inventory Service
- Notification Service

---

# Challenges of Event Sourcing

## 1. Increased Complexity

Event sourcing is harder than CRUD systems.

Requires:
- Event handling
- Replay logic
- Event versioning

---

## 2. Eventual Consistency

Read models may lag behind events.

---

## 3. Large Event Streams

Systems may generate millions of events.

Replaying everything can become slow.

---

## 4. Event Versioning

Events may change structure over time.

Example:

Old event:

```json
{
  "name": "Ahmed"
}
```

New event:

```json
{
  "firstName": "Ahmed",
  "lastName": "Ali"
}
```

Backward compatibility becomes important.

---

# Snapshots

To avoid replaying all events from the beginning, systems use snapshots.

A snapshot stores:
- Current calculated state at a specific moment

Example:

```text
Snapshot:
Balance = 900$
After Event #5000
```

System:
- Loads snapshot
- Replays only newer events

This improves performance.

---

# Event Store

An Event Store is a database optimized for storing events.

Common choices:

- EventStoreDB
- Kafka
- PostgreSQL
- MongoDB

Events are usually stored:
- Sequentially
- Append-only

---

# Event Sourcing with CQRS

Event Sourcing is commonly used with CQRS.

Flow:

```text
Command
   ↓
Generate Event
   ↓
Store Event
   ↓
Publish Event
   ↓
Update Read Models
```

---

# Example: E-Commerce System

## Customer creates order

Command:

```text
CreateOrder
```

Generated event:

```text
OrderCreated
```

---

## Payment succeeds

Generated event:

```text
PaymentCompleted
```

---

## Inventory updated

Generated event:

```text
InventoryReduced
```

---

# Event Stream Example

```text
OrderCreated
PaymentCompleted
InventoryReduced
OrderShipped
OrderDelivered
```

This stream represents the complete order history.

---

# Event Sourcing vs Traditional CRUD

| Feature | Traditional CRUD | Event Sourcing |
|----------|------------------|----------------|
| Stores Current State | Yes | No |
| Stores History | Limited | Complete |
| Audit Trail | Weak | Strong |
| Complexity | Low | High |
| Scalability | Medium | High |
| Event Replay | No | Yes |
| Time Travel | No | Yes |

---

# Eventual Consistency

Event Sourcing systems usually use:
- Asynchronous processing
- Eventual consistency

Meaning:
- Read models may not update instantly

Example:

```text
OrderCreated event stored
Dashboard updates 1 second later
```

---

# Best Practices

## Design Events Carefully

Events are permanent.

Bad naming:

```text
UpdateUser
```

Good naming:

```text
UserEmailChanged
```

---

## Keep Events Immutable

Never modify existing events.

---

## Use Snapshots

Improves replay performance.

---

## Version Events Properly

Handle schema evolution safely.

---

## Make Event Handlers Idempotent

Processing same event twice should not break the system.

---

# Common Use Cases

Event Sourcing works well for:

- Banking systems
- Financial systems
- E-Commerce platforms
- Audit-heavy systems
- Real-time analytics
- Distributed microservices

---

# Technologies Commonly Used

## Messaging Systems

- Kafka
- RabbitMQ
- Azure Service Bus

---

## Event Stores

- EventStoreDB
- PostgreSQL
- MongoDB

---

## Frameworks

- Axon Framework
- MassTransit
- NServiceBus
- Akka.NET

---

# Real-World Example

## Online Banking System

Events:

```text
AccountCreated
MoneyDeposited
MoneyTransferred
MoneyWithdrawn
AccountClosed
```

The system reconstructs account state from event history.

Benefits:
- Full transaction history
- Fraud investigation
- Auditing
- Recovery

---

# Summary

Event Sourcing is a pattern where:
- State changes are stored as events
- Current state is rebuilt by replaying events

Instead of storing only current data.

It provides:
- Full audit history
- Scalability
- Time travel
- Better integration with microservices

But introduces:
- Complexity
- Eventual consistency
- Event management challenges

Event Sourcing is most useful in:
- Large distributed systems
- Financial applications
- Event-driven architectures
- CQRS-based systems