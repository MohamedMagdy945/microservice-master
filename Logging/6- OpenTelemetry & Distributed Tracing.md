# OpenTelemetry & Distributed Tracing

## Overview

OpenTelemetry is a modern observability framework used to collect **logs, metrics, and traces** from applications. It is widely used in distributed systems to understand how requests move across services.

Distributed tracing is the main concept here: tracking a single request as it flows through multiple services.

---

## What is Distributed Tracing?

In microservices architecture, a single request might pass through:

* API Gateway
* Auth Service
* Order Service
* Payment Service
* Database

Distributed tracing allows you to see the full path of that request.

---

## Key Concepts

### Trace

A full journey of a request across services.

### Span

A single unit of work within a trace (e.g., a database call or API call).

### Trace ID

A unique ID representing the entire request flow.

### Span ID

A unique ID for each operation inside the trace.

---

## Why OpenTelemetry?

Without it:

* You only see isolated logs
* Hard to debug performance issues
* No visibility across services

With it:

* Full request flow visibility
* Performance bottleneck detection
* Unified observability (logs + metrics + traces)

---

## Basic Architecture

OpenTelemetry collects data and sends it to a backend:

* Jaeger
* Zipkin
* Prometheus
* Grafana Tempo

---

## Example Setup in ASP.NET Core

### Install Packages

```bash id="ot1"
dotnet add package OpenTelemetry.Extensions.Hosting
dotnet add package OpenTelemetry.Instrumentation.AspNetCore
dotnet add package OpenTelemetry.Instrumentation.Http
```

---

### Configure OpenTelemetry

```csharp id="ot2"
using OpenTelemetry.Trace;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddOpenTelemetry()
    .WithTracing(tracing =>
    {
        tracing
            .AddAspNetCoreInstrumentation()
            .AddHttpClientInstrumentation()
            .AddConsoleExporter();
    });

var app = builder.Build();
app.Run();
```

---

## How Tracing Works

When a request enters the system:

1. A Trace ID is created
2. Each service adds its own Span
3. All spans are grouped under the same Trace ID
4. You can visualize the full flow

---

## Example Flow

```
Client Request
   ↓
API Gateway (Span 1)
   ↓
Auth Service (Span 2)
   ↓
Order Service (Span 3)
   ↓
Database Call (Span 4)
```

All of these belong to one Trace ID.

---

## Integration with Logging

OpenTelemetry works best with structured logging:

```csharp id="ot3"
logger.LogInformation("Processing order {OrderId} in Trace {TraceId}", orderId, Activity.Current?.TraceId);
```

---

## Exporters

OpenTelemetry sends data to external systems using exporters:

* Console Exporter (development)
* Jaeger Exporter
* Zipkin Exporter
* OTLP Exporter (modern standard)

---

## Benefits

* Full visibility into distributed systems
* Easier debugging of performance issues
* Detect slow services or endpoints
* Correlate logs, metrics, and traces

---

## Best Practices

* Always enable tracing in production systems
* Use consistent service naming
* Avoid high-cardinality data in traces
* Combine tracing with structured logging
* Export to centralized observability platform

---

## Common Mistakes

* Enabling tracing without filtering (performance issues)
* Not propagating trace context between services
* Ignoring sampling configuration
* Mixing multiple tracing systems incorrectly

---

## Summary

* OpenTelemetry is a unified observability framework
* Distributed tracing tracks requests across services
* Uses Trace ID and Spans to represent request flow
* Essential for microservices and cloud systems

---

Next Topic: High-Performance Logging
