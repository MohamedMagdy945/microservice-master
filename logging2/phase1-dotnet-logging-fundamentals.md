# Phase 1 — .NET Logging Fundamentals

## Overview

`Microsoft.Extensions.Logging` is the built-in logging abstraction in .NET. It provides a unified API so your application code is decoupled from any specific logging implementation (console, file, Serilog, etc.).

---

## Core abstractions

### `ILogger`
The main interface you inject into your classes to write log entries.

```csharp
public class OrderService
{
    private readonly ILogger<OrderService> _logger;

    public OrderService(ILogger<OrderService> logger)
    {
        _logger = logger;
    }

    public void PlaceOrder(int orderId)
    {
        _logger.LogInformation("Placing order {OrderId}", orderId);
    }
}
```

### `ILoggerFactory`
Creates `ILogger` instances. Usually managed by the DI container — you rarely use it directly.

### `ILoggerProvider`
Pluggable output target (console, file, Elasticsearch, etc.). Multiple providers can be active simultaneously.

---

## Log levels

| Level | Value | Use when |
|-------|-------|----------|
| `Trace` | 0 | Very detailed diagnostic info, high volume |
| `Debug` | 1 | Useful during development and debugging |
| `Information` | 2 | Normal application flow milestones |
| `Warning` | 3 | Unexpected but recoverable situations |
| `Error` | 4 | Failures that need attention |
| `Critical` | 5 | System-wide failures, crash-level events |
| `None` | 6 | Disables logging entirely |

```csharp
_logger.LogTrace("Entering method with param {Param}", param);
_logger.LogDebug("Cache miss for key {Key}", key);
_logger.LogInformation("User {UserId} logged in", userId);
_logger.LogWarning("Retry attempt {Attempt} for {Url}", attempt, url);
_logger.LogError(ex, "Failed to process order {OrderId}", orderId);
_logger.LogCritical("Database connection pool exhausted");
```

---

## Structured logging

Always use **message templates** with named placeholders — not string interpolation. This lets the logging provider store properties as searchable fields in Elasticsearch.

```csharp
// GOOD — structured, searchable in Elasticsearch
_logger.LogInformation("Order {OrderId} placed by {UserId} for {Amount:C}", orderId, userId, amount);

// BAD — plain text, loses structure
_logger.LogInformation($"Order {orderId} placed by {userId} for {amount:C}");
```

---

## Setup in `Program.cs`

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Logging
    .ClearProviders()
    .AddConsole()
    .AddDebug()
    .SetMinimumLevel(LogLevel.Information);
```

### Configuration via `appsettings.json`

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning",
      "MyApp.Services": "Debug"
    }
  }
}
```

---

## Log categories

Each `ILogger<T>` uses the fully qualified type name as its category. This lets you filter logs per namespace:

```json
{
  "Logging": {
    "LogLevel": {
      "MyApp.Services.PaymentService": "Debug",
      "MyApp.Controllers": "Warning"
    }
  }
}
```

---

## Scopes

Group related log entries with a shared context (e.g., a request ID):

```csharp
using (_logger.BeginScope("RequestId: {RequestId}", requestId))
{
    _logger.LogInformation("Processing started");
    // all logs here include RequestId
    _logger.LogInformation("Processing complete");
}
```

---

## Next step

→ **Phase 2** — Add Serilog as a provider for richer sinks and enrichment.
