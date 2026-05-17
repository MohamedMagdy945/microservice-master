# 🐇 RabbitMQ Learning Guide

A complete, hands-on guide to RabbitMQ from fundamentals to production — structured for .NET developers.

---

## 📚 Table of Contents

| # | Topic | Description |
|---|-------|-------------|
| 01 | [Fundamentals](./01-fundamentals/README.md) | Messaging concepts, brokers, sync vs async, events vs commands |
| 02 | [Core Concepts](./02-core-concepts/README.md) | Producer, consumer, queue, exchange, binding, routing key |
| 03 | [Exchange Types](./03-exchange-types/README.md) | Direct, Fanout, Topic, Headers — with diagrams |
| 04 | [Environment Setup](./04-environment-setup/README.md) | Docker setup, management dashboard, docker-compose |
| 05 | [Hello World](./05-hello-world/README.md) | First producer & consumer in .NET |
| 06 | [.NET Integration](./06-dotnet-integration/README.md) | Connection management, DI registration, message properties |
| 07 | [Messaging Patterns](./07-messaging-patterns/README.md) | Work queues, pub/sub, routing, RPC, retry, DLQ |
| 08 | [Microservices](./08-microservices-integration/README.md) | Order, inventory, payment, notification services |
| 09 | [Clean Arch + CQRS](./09-clean-architecture-cqrs/README.md) | Domain events, MediatR, aggregate roots |
| 10 | [Production Best Practices](./10-production-best-practices/README.md) | Durability, idempotency, monitoring, resilience |
| 11 | [Advanced Topics](./11-advanced-topics/README.md) | Clustering, HA, Shovel, Federation, performance |

---

## 🚀 Quick Start

```bash
# 1. Start RabbitMQ
docker run -d --name rabbitmq \
  -p 5672:5672 -p 15672:15672 \
  rabbitmq:3-management

# 2. Open Management Dashboard
# http://localhost:15672
# username: guest | password: guest

# 3. Create a .NET project
dotnet new console -n MyRabbitMQ
cd MyRabbitMQ
dotnet add package RabbitMQ.Client
```

---

## 🗺️ Learning Path

```
Beginner    → Topics 01-05   (concepts + first code)
Intermediate → Topics 06-08  (real .NET patterns)
Advanced     → Topics 09-11  (architecture + production)
```

---

## 💡 Key Concepts at a Glance

```
Producer ──► Exchange ──[binding]──► Queue ──► Consumer
                │
           routing key
         determines path
```

| Exchange Type | When to Use |
|--------------|-------------|
| Direct | One specific queue |
| Fanout | Broadcast to all |
| Topic | Pattern-based routing |
| Headers | Attribute-based routing |

---

*Built with ❤️ for developers learning event-driven architecture.*
