# Twelve-Factor App - Backing Services

## Overview

In the Twelve-Factor App methodology, a **Backing Service** is any external service that the application consumes over the network as part of its normal operation.

Examples include:
- Databases
- Caching systems
- Message brokers
- Email services
- Cloud storage
- External APIs

The core idea is:

> Treat backing services as attached resources.

This means the application should access services through configuration and should be able to replace them without changing the application code.

---

# What Is a Backing Service?

A backing service is any external dependency connected to the application.

## Examples

| Service Type | Examples |
|---|---|
| Database | PostgreSQL, MySQL, SQL Server |
| Cache | Redis |
| Message Broker | RabbitMQ, Kafka |
| Storage | AWS S3, Azure Blob Storage |
| Email Service | SendGrid |
| Monitoring | Prometheus |

---

# Core Principle

Applications should not tightly depend on specific service implementations.

Instead:
- Services are connected through configuration
- Credentials are stored externally
- Services can be replaced easily

---

# Why Is This Important?

Modern cloud applications run in multiple environments:
- Development
- Testing
- Staging
- Production

A backing service may also change over time:
- Local database → Cloud database
- RabbitMQ → Kafka
- Local Redis → Managed Redis Cache

The application should continue working without modifying the codebase.

---

# Bad Practice (Hardcoded Service)

```csharp
var connectionString =
    "Server=localhost;Database=AppDb;User Id=sa;Password=123";
```

## Problems
- Difficult to deploy
- Environment-specific
- Security risks
- Hard to scale

---

# Good Practice (Configuration-Based)

## appsettings.json

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=db;Database=AppDb;"
  }
}
```

## ASP.NET Core Example

```csharp
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(
        builder.Configuration.GetConnectionString("DefaultConnection")));
```

---

# Environment Variables

The Twelve-Factor App methodology strongly recommends using environment variables.

## Example

```env
DATABASE_URL=postgres://user:password@localhost/appdb
REDIS_URL=redis://localhost:6379
RABBITMQ_HOST=rabbitmq
```

---

# Benefits of Backing Services

## Flexibility
Services can be replaced without code changes.

## Better Deployment
Applications become portable across environments.

## Improved Security
Sensitive information stays outside source code.

## Easier Scaling
Infrastructure can evolve independently from application logic.

---

# Real-World Example

## Development Environment
- Local SQL Server
- Local RabbitMQ
- Local Redis

## Production Environment
- Azure SQL Database
- Managed RabbitMQ Cluster
- Azure Redis Cache

The application code remains unchanged.

Only configuration changes.

---

# Backing Services in Docker

## Docker Compose Example

```yaml
services:

  api:
    build: .
    environment:
      - ConnectionStrings__Default=Server=db

  db:
    image: postgres

  redis:
    image: redis

  rabbitmq:
    image: rabbitmq
```

---

# Key Twelve-Factor Rule

A backing service should be:
- Attached through configuration
- Replaceable
- Independent from application code

---

# Best Practices

- Use environment variables
- Avoid hardcoded credentials
- Separate configuration from code
- Use secrets managers in production
- Make services replaceable
- Use dependency injection

---

# Common Mistakes

## Hardcoding Credentials
Never store secrets inside source code.

## Tight Coupling
Avoid writing code tied to one specific provider.

## Environment-Specific Logic

Avoid:

```csharp
if (environment == "Production")
{
    // different implementation
}
```

Prefer configuration-driven approaches.

---

# Conclusion

Backing Services are a core part of cloud-native and microservices architectures.

The Twelve-Factor App principle encourages:
- Loose coupling
- Externalized configuration
- Service portability
- Infrastructure flexibility

By treating services as attached resources, applications become:
- Easier to deploy
- Easier to scale
- Easier to maintain
- More cloud-ready