# Twelve-Factor App - Processes and Port Binding

# Overview

The Twelve-Factor App methodology defines best practices for building modern, scalable, and cloud-native applications.

Two important principles are:
- Processes
- Port Binding

These principles help applications become:
- Stateless
- Scalable
- Portable
- Independent

---

# 1. Processes

## Definition

Applications should execute as one or more stateless processes.

This means:
- Each process is independent
- Processes do not share memory
- Processes should not store persistent state locally

---

# Stateless Processes

A stateless process does not keep data between requests.

Any important data should be stored in external services such as:
- Databases
- Redis
- Distributed caches
- Message brokers

---

# Why Stateless Processes Matter

Stateless processes provide:

## Scalability
Multiple instances can run easily.

## Reliability
Failed processes can restart safely.

## Portability
Applications can move between environments.

## Better Cloud Support
Cloud platforms work best with stateless applications.

---

# Bad Practice

Storing application state in memory or local files.

## Example

```csharp
static List<string> users = new();
```

### Problems
- Data is lost after restart
- Difficult to scale
- Multiple instances become inconsistent

---

# Good Practice

Store state externally.

## Example

```csharp
await redis.SetStringAsync("user", userData);
```

---

# Process Types

Applications may run multiple process types.

## Examples

| Process Type | Responsibility |
|---|---|
| Web Process | Handles HTTP requests |
| Worker Process | Background jobs |
| Scheduler Process | Scheduled tasks |
| Queue Consumer | Message processing |

---

# Process Isolation

Each process should:
- Run independently
- Start quickly
- Stop gracefully

Processes should not depend on:
- Shared local files
- Shared memory
- Manual configuration

---

# Process Management

Modern platforms manage processes automatically.

## Examples
- Docker
- Kubernetes
- Azure App Services
- AWS ECS

These platforms:
- Restart failed processes
- Scale applications
- Monitor health

---

# 2. Port Binding

## Definition

Applications should export services through port binding.

The application becomes self-contained and listens on a specific port.

---

# Traditional Approach

Older applications depended on external web servers.

Example:
- IIS
- Apache
- NGINX

The application itself did not expose services directly.

---

# Twelve-Factor Approach

The application should:
- Start independently
- Bind directly to a port
- Handle HTTP traffic itself

---

# ASP.NET Core Example

```csharp
var builder = WebApplication.CreateBuilder(args);

var app = builder.Build();

app.MapGet("/", () => "Hello World");

app.Run();
```

The application binds directly to a port such as:
- 5000
- 8080

---

# Environment-Based Port Configuration

Applications usually receive ports through environment variables.

## Example

```env
PORT=8080
```

---

# ASP.NET Core Example

```csharp
builder.WebHost.UseUrls("http://*:8080");
```

---

# Benefits of Port Binding

## Self-Contained Applications
Applications run independently.

## Easier Deployment
Applications can run anywhere.

## Better Container Support
Works naturally with Docker and Kubernetes.

## Simpler Infrastructure
No need for heavy external servers.

---

# Port Binding in Docker

## Docker Example

```dockerfile
EXPOSE 8080
```

## Docker Run Example

```bash
docker run -p 8080:8080 myapp
```

---

# Port Binding in Kubernetes

Kubernetes routes traffic to application ports automatically.

## Example

```yaml
ports:
  - containerPort: 8080
```

---

# Processes and Scalability

Stateless processes combined with port binding allow:
- Horizontal scaling
- Load balancing
- Auto-scaling
- Cloud-native deployments

---

# Best Practices

## Processes
- Keep processes stateless
- Store persistent data externally
- Design for failure and restart

## Port Binding
- Use environment variables for ports
- Keep applications self-contained
- Avoid server-specific dependencies

---

# Common Mistakes

## Storing Local State
Causes scaling and reliability issues.

## Hardcoding Ports

Bad Example:

```csharp
app.Run("http://localhost:5000");
```

Prefer configuration-based ports.

---

# Real-World Example

A cloud-native ASP.NET Core microservice:
- Runs inside Docker
- Exposes port 8080
- Uses Redis for session storage
- Stores data in PostgreSQL
- Scales using Kubernetes

The service remains:
- Stateless
- Replaceable
- Easy to scale

---

# Conclusion

The Twelve-Factor principles of Processes and Port Binding help create applications that are:
- Cloud-ready
- Scalable
- Portable
- Resilient

By following these principles:
- Applications become easier to deploy
- Scaling becomes simpler
- Infrastructure becomes more flexible
- Systems become more reliable