# Redis for Microservices

Redis plays a key role in modern microservices architectures by enabling caching, messaging, coordination, and scalability across distributed systems.

This guide explains how Redis is used in microservices-based systems with practical patterns.

---

# Table of Contents

1. Shared Distributed Cache
2. Service Communication
3. Event-Driven Architecture
4. Redis Streams for Messaging
5. API Gateway Caching
6. Distributed Sessions

---

# 1. Shared Distributed Cache

In microservices, each service typically runs independently. A shared cache like Redis helps reduce database load and improve performance.

---

## Why Shared Cache?

* Avoid duplicate database calls
* Improve response time
* Share data across services
* Reduce system load

---

## Example Pattern

```text
User Service → Redis Cache → Database
Order Service → Redis Cache → Database
```

---

## Example Usage

```csharp
await cache.SetStringAsync("product:1", json);
var product = await cache.GetStringAsync("product:1");
```

---

# 2. Service Communication

Redis can be used for lightweight communication between microservices.

---

## Approaches

* Pub/Sub messaging
* Streams-based messaging
* Cache-based state sharing

---

## Benefits

* Loose coupling
* Fast communication
* Scalable architecture

---

## Example Pub/Sub

```csharp
var sub = redis.GetSubscriber();

await sub.PublishAsync("order-events", "OrderCreated");
```

---

# 3. Event-Driven Architecture

Redis supports event-driven systems where services react to events asynchronously.

---

## Architecture Flow

```text
Service A → Publish Event → Redis → Service B → Process Event
```

---

## Advantages

* Asynchronous processing
* Loose coupling
* High scalability
* Better fault tolerance

---

## Common Events

* User created
* Order placed
* Payment completed
* Inventory updated

---

# 4. Redis Streams for Messaging

Redis Streams provide a durable message queue for microservices.

---

## Add Event to Stream

```csharp
await db.StreamAddAsync("orders", new NameValueEntry[]
{
    new("orderId", "123"),
    new("status", "created")
});
```

---

## Read Stream Messages

```csharp
var messages = await db.StreamReadAsync("orders", "0-0");
```

---

## Consumer Groups

Redis Streams support multiple consumers.

---

## Use Cases

* Order processing
* Event sourcing
* Background jobs
* Data pipelines

---

# 5. API Gateway Caching

API Gateways often use Redis to cache responses and reduce backend load.

---

## Flow

```text
Client → API Gateway → Redis Cache → Microservices
```

---

## Example

```csharp
var cachedResponse = await cache.GetStringAsync("api:products");

if (cachedResponse == null)
{
    cachedResponse = await FetchFromService();
    await cache.SetStringAsync("api:products", cachedResponse);
}
```

---

## Benefits

* Faster API responses
* Reduced service load
* Better scalability
* Lower latency

---

# 6. Distributed Sessions

Redis is commonly used to store session data across microservices.

---

## Why Distributed Sessions?

In microservices:

* Services are stateless
* Sessions must be shared
* Load balancers distribute traffic

---

## Example Flow

```text
Client → Gateway → Redis Session Store → Microservices
```

---

## ASP.NET Core Example

```csharp
builder.Services.AddStackExchangeRedisCache(options =>
{
    options.Configuration = "localhost:6379";
});

builder.Services.AddSession();
```

---

## Usage

```csharp
HttpContext.Session.SetString("userId", "1");
var userId = HttpContext.Session.GetString("userId");
```

---

## Benefits

* Centralized session storage
* Works across multiple services
* Supports horizontal scaling
* Reliable user state management

---

# Microservices Patterns with Redis

| Pattern      | Use Case                 |
| ------------ | ------------------------ |
| Shared Cache | Performance optimization |
| Pub/Sub      | Real-time messaging      |
| Streams      | Event-driven systems     |
| API Caching  | Gateway optimization     |
| Sessions     | User state management    |

---

# Best Practices

* Keep services stateless
* Use Redis for fast shared state only
* Prefer Streams for durability
* Avoid overusing Pub/Sub for critical data
* Use cache expiration wisely
* Monitor Redis performance

---

# Summary

Redis is a core component in microservices architecture, enabling:

* Fast distributed caching
* Event-driven communication
* Scalable messaging systems
* Centralized session management
* API performance optimization

When used correctly, Redis significantly improves microservices scalability, responsiveness, and reliability.
