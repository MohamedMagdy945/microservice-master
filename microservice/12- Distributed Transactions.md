# Distributed Transactions in Microservices Architecture

## Introduction

A Distributed Transaction occurs when a single business operation involves multiple microservices and databases, where all operations must succeed or fail together.

In monolithic applications, transactions are simple because all operations usually happen inside one database.

Example:

```sql
BEGIN TRANSACTION;

UPDATE Accounts SET Balance = Balance - 100 WHERE Id = 1;
UPDATE Accounts SET Balance = Balance + 100 WHERE Id = 2;

COMMIT;
```

If any operation fails:
- The transaction rolls back
- Data remains consistent

In microservices architecture, this becomes much harder because:
- Each service owns its own database
- Services are distributed across networks
- Multiple databases may be involved

This creates the challenge known as Distributed Transactions.

---

# Why Distributed Transactions Are Difficult

## 1. Independent Databases

Each microservice manages its own database independently.

Example:

```text
Order Service → Order Database
Payment Service → Payment Database
Inventory Service → Inventory Database
```

A single global transaction across all databases is difficult.

---

## 2. Network Failures

Services communicate over the network.

Possible problems:
- Timeout
- Packet loss
- Service unavailable
- Partial success

Example:

```text
Payment completed
Inventory update failed
```

This leaves the system in an inconsistent state.

---

## 3. Different Database Technologies

Microservices may use different databases:

- SQL Server
- PostgreSQL
- MongoDB
- Redis

Not all databases support distributed transaction protocols.

---

## 4. Scalability Problems

Traditional distributed transaction mechanisms reduce scalability because services become tightly coupled.

---

# Traditional Solution: Two-Phase Commit (2PC)

Two-Phase Commit is a protocol used to coordinate transactions across multiple services or databases.

---

## How 2PC Works

### Phase 1: Prepare Phase

Coordinator asks all participants:

```text
Can you commit?
```

Each service:
- Executes operation temporarily
- Locks resources
- Responds:
  - YES
  - NO

---

### Phase 2: Commit Phase

If all services respond YES:

```text
Coordinator → Commit
```

Otherwise:

```text
Coordinator → Rollback
```

---

# Example of 2PC

## Money Transfer System

### Step 1

Bank Service A:
```text
Withdraw $100
```

### Step 2

Bank Service B:
```text
Deposit $100
```

### Step 3

Coordinator ensures:
- Both succeed
- Or both rollback

---

# Problems with 2PC

## 1. Tight Coupling

Services become highly dependent on the coordinator.

---

## 2. Blocking

Resources remain locked while waiting for all participants.

This reduces:
- Performance
- Scalability

---

## 3. Single Point of Failure

If coordinator crashes:
- System may become stuck

---

## 4. Poor Cloud-Native Compatibility

Modern microservices prefer:
- Loose coupling
- High scalability
- Independent deployment

2PC conflicts with these goals.

---

# Modern Solution: Saga Pattern

The Saga Pattern is the preferred approach for distributed transactions in microservices.

Instead of one large transaction:
- Transaction is divided into smaller local transactions

Each service:
- Executes its own local transaction
- Publishes an event

If something fails:
- Compensating transactions undo previous work

---

# Saga Workflow Example

## E-Commerce Order Processing

Services:
- Order Service
- Payment Service
- Inventory Service

---

## Successful Flow

### Step 1

Order Service:
```text
Create Order
```

Publishes:
```text
OrderCreated
```

---

### Step 2

Payment Service:
```text
Process Payment
```

Publishes:
```text
PaymentCompleted
```

---

### Step 3

Inventory Service:
```text
Reduce Stock
```

Publishes:
```text
InventoryUpdated
```

Transaction completed successfully.

---

# Failure Scenario

Suppose payment succeeds but inventory fails.

---

## Compensation Flow

### Step 1

Inventory Service fails:
```text
Out of stock
```

---

### Step 2

Payment Service performs compensation:
```text
Refund Payment
```

---

### Step 3

Order Service performs compensation:
```text
Cancel Order
```

System becomes eventually consistent again.

---

# Types of Saga Pattern

## 1. Choreography-Based Saga

Services communicate only through events.

Example:

```text
OrderCreated → PaymentProcessed → InventoryUpdated
```

### Advantages

- No central controller
- Loose coupling

### Disadvantages

- Harder debugging
- Complex event chains

---

## 2. Orchestration-Based Saga

A central orchestrator controls the workflow.

Example:

```text
Saga Orchestrator
    ↓
Order Service
    ↓
Payment Service
    ↓
Inventory Service
```

### Advantages

- Easier monitoring
- Centralized control

### Disadvantages

- Extra component required
- Possible bottleneck

---

# Comparison: 2PC vs Saga

| Feature | Two-Phase Commit | Saga Pattern |
|----------|------------------|---------------|
| Consistency | Strong | Eventual |
| Coupling | High | Low |
| Scalability | Low | High |
| Performance | Slower | Faster |
| Fault Tolerance | Weak | Strong |
| Cloud-Native Friendly | No | Yes |

---

# Eventual Consistency

Saga Pattern uses:
- Eventual Consistency

This means:
- System may be temporarily inconsistent
- Eventually all services become consistent

Example:

```text
Payment completed
Inventory update pending
```

Temporary inconsistency is acceptable in microservices.

---

# Technologies Commonly Used

## Messaging Systems

- RabbitMQ
- Kafka
- Azure Service Bus

---

## Workflow Engines

- Temporal
- Camunda
- MassTransit Saga
- NServiceBus

---

# Best Practices

## Keep Transactions Small

Smaller transactions:
- Fail less often
- Recover more easily

---

## Use Idempotency

Operations should be safe to retry.

Example:

```text
Processing same event twice should not duplicate payment
```

---

## Implement Compensation Carefully

Each action should have a rollback strategy.

Example:

```text
Reserve Stock → Release Stock
```

---

## Prefer Asynchronous Communication

Events improve:
- Scalability
- Resilience

---

## Monitor Saga Flows

Distributed transactions are difficult to debug.

Use:
- Logging
- Tracing
- Correlation IDs

---

# Common Anti-Patterns

## Shared Database

Using one database for all services destroys microservice independence.

---

## Distributed Locks Everywhere

Too many locks reduce scalability.

---

## Long-Running Transactions

Transactions lasting too long increase failure probability.

---

# Real-World Example

## Food Delivery Application

Services:
- Order Service
- Payment Service
- Restaurant Service
- Delivery Service

---

## Order Flow

### Step 1

Order Service:
```text
Create Order
```

---

### Step 2

Payment Service:
```text
Charge Customer
```

---

### Step 3

Restaurant Service:
```text
Prepare Food
```

---

### Step 4

Delivery Service:
```text
Assign Driver
```

---

# Failure Example

Suppose:
- Payment succeeds
- Restaurant rejects order

Compensations:
- Refund payment
- Cancel order

---

# Summary

Distributed Transactions are one of the hardest problems in microservices architecture.

Traditional database transactions do not work well because:
- Services are distributed
- Databases are isolated
- Networks can fail

Modern microservices avoid global transactions by using:
- Saga Pattern
- Events
- Compensation actions
- Eventual consistency

The main goal is to achieve:
- Scalability
- Resilience
- Loose coupling
- Fault tolerance

Instead of relying on strict global consistency.