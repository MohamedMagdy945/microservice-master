# Microservices Architecture - Introduction to Data Management

## 1. Overview

In a **Microservices Architecture**, each service is designed to be **independently deployable, scalable, and maintainable**. One of the most critical challenges in this architecture is **data management**.

Unlike monolithic systems where a single database is shared, microservices encourage **decentralized data management**.

---

## 2. Core Principle: Database per Service

Each microservice should own its **own private database**.

### Why?
- Avoid tight coupling between services
- Enable independent scaling
- Allow different technologies per service (polyglot persistence)
- Prevent schema conflicts

### Example:
- User Service → PostgreSQL
- Order Service → MongoDB
- Payment Service → MySQL

No service directly accesses another service’s database.

---

## 3. Data Ownership

Each service is:
- The **single source of truth** for its data
- Responsible for **reading, writing, and maintaining consistency**
- Exposes data only through APIs or events

👉 Other services must not query its database directly.

---

## 4. Communication Between Services (Data Sharing)

Since databases are isolated, services share data using:

### 1. Synchronous Communication (API Calls)
- REST or gRPC
- Real-time request/response
- Example: Order Service calls User Service to get user details

### 2. Asynchronous Communication (Events)
- Messaging systems like RabbitMQ or Kafka
- Event-driven architecture
- Example:
  - User Created event
  - Order Service listens and updates its local data

---

## 5. Data Consistency Challenges

In microservices, we move from **strong consistency** (monoliths) to **eventual consistency**.

### Types of Consistency:
- **Strong Consistency** → immediate update everywhere (rare in microservices)
- **Eventual Consistency** → data becomes consistent over time

### Problem Example:
- User updates address
- Order service still has old address for a short time

---

## 6. Distributed Data Management Patterns

### 1. Saga Pattern
Used to manage distributed transactions.

- **Choreography**: services react to events
- **Orchestration**: central coordinator controls flow

### 2. CQRS (Command Query Responsibility Segregation)
- Separate read and write models
- Optimized performance and scalability

### 3. Event Sourcing
- Store state changes as a sequence of events
- Rebuild state from events

---

## 7. Data Duplication (Trade-off)

To avoid cross-service DB queries, services often **duplicate data**.

### Example:
- Order Service stores:
  - userId
  - userName (copied from User Service)

### Pros:
- Faster reads
- Less coupling

### Cons:
- Data inconsistency risk
- Extra storage

---

## 8. Common Mistakes

❌ Sharing one database across all services  
❌ Direct DB access between services  
❌ Ignoring eventual consistency  
❌ Over-complicating event systems early  

---

## 9. Best Practices

✔ Each service owns its database  
✔ Communicate via APIs or events  
✔ Design for eventual consistency  
✔ Use Saga for distributed transactions  
✔ Avoid tight coupling at all costs  

---

## 10. Summary

Data management in microservices is about:
- **Decentralization**
- **Autonomy**
- **Loose coupling**
- **Event-driven synchronization**

It trades **simplicity (monolith)** for **scalability and flexibility**.

---