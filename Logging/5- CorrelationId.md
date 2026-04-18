# Correlation IDs & Request Context

## Overview

In real-world applications, especially APIs and distributed systems, a single request often travels through multiple services. To track this journey, we use a **Correlation ID**.

A Correlation ID is a unique identifier attached to a request that allows you to trace all related logs across the system.

---

## Why Correlation IDs Matter

Without correlation IDs:

* Logs from different services look unrelated
* Debugging production issues becomes difficult
* Tracing a single request is nearly impossible

With correlation IDs:

* You can follow a request end-to-end
* You can group logs across microservices
* You can quickly identify where failures happened

---

## Basic Concept

Each incoming request gets a unique ID:

```
RequestId: a1b2c3
```

This ID is passed through:

* Controllers
* Services
* External API calls
* Databases (optional tracing)

---

## Example in ASP.NET Core

### Middleware to Generate Correlation ID

```csharp id="cr1"
public class CorrelationIdMiddleware
{
    private readonly RequestDelegate _next;

    public CorrelationIdMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task Invoke(HttpContext context)
    {
        var correlationId = Guid.NewGuid().ToString();

        context.Items["CorrelationId"] = correlationId;
        context.Response.Headers.Add("X-Correlation-ID", correlationId);

        await _next(context);
    }
}
```

---

## Using Correlation ID in Logging

```csharp id="cr2"
public class OrderService
{
    private readonly ILogger<OrderService> _logger;
    private readonly IHttpContextAccessor _httpContextAccessor;

    public OrderService(ILogger<OrderService> logger, IHttpContextAccessor accessor)
    {
        _logger = logger;
        _httpContextAccessor = accessor;
    }

    public void CreateOrder(int orderId)
    {
        var correlationId = _httpContextAccessor.HttpContext?.Items["CorrelationId"];

        _logger.LogInformation(
            "Creating order {OrderId} with CorrelationId {CorrelationId}",
            orderId,
            correlationId);
    }
}
```

---

## Passing Correlation ID Across Services

In microservices architecture:

1. Service A receives request → generates ID
2. Service A calls Service B → passes same ID
3. Service B logs using same ID

This ensures full traceability.

---

## Request Context

Request context is a storage mechanism that holds data during the lifetime of a request.

Common data stored:

* Correlation ID
* User ID
* Tenant ID
* Request metadata

---

## Example Request Context Usage

```csharp id="cr3"
public class RequestContext
{
    public string CorrelationId { get; set; }
    public string UserId { get; set; }
}
```

---

## Benefits

* Easier debugging in production
* Full traceability across services
* Better monitoring and observability
* Faster root cause analysis

---

## Best Practices

* Always generate a correlation ID for every request
* Pass it across all service boundaries
* Include it in every log entry
* Use middleware to avoid repeating code
* Never expose sensitive data in correlation context

---

## Common Mistakes

* Not forwarding correlation ID between services
* Generating multiple IDs per request
* Forgetting to include it in logs
* Mixing formats across services

---

## Summary

* Correlation ID uniquely identifies a request
* It helps trace logs across multiple systems
* Request context stores useful per-request data
* Essential for microservices and distributed systems

---

Next Topic: OpenTelemetry & Distributed Tracing
