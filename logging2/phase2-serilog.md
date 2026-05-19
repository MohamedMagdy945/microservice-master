# Phase 2 — Serilog

## Overview

Serilog is the most popular third-party logging library for .NET. It plugs into `Microsoft.Extensions.Logging` as a provider and adds powerful features: structured sinks, enrichers, and a rich ecosystem — including the Elasticsearch sink you'll use in Phase 4.

---

## NuGet packages

```bash
dotnet add package Serilog.AspNetCore
dotnet add package Serilog.Settings.Configuration
dotnet add package Serilog.Enrichers.Environment
dotnet add package Serilog.Enrichers.Thread
dotnet add package Serilog.Enrichers.Process
```

---

## Basic setup in `Program.cs`

```csharp
using Serilog;

Log.Logger = new LoggerConfiguration()
    .ReadFrom.Configuration(builder.Configuration)
    .Enrich.FromLogContext()
    .WriteTo.Console()
    .CreateLogger();

builder.Host.UseSerilog();
```

### `appsettings.json` configuration

```json
{
  "Serilog": {
    "MinimumLevel": {
      "Default": "Information",
      "Override": {
        "Microsoft": "Warning",
        "System": "Warning"
      }
    },
    "WriteTo": [
      { "Name": "Console" }
    ],
    "Enrich": ["FromLogContext", "WithMachineName", "WithThreadId"]
  }
}
```

---

## Enrichers

Enrichers attach additional properties to every log event automatically.

```csharp
Log.Logger = new LoggerConfiguration()
    .Enrich.FromLogContext()               // picks up LogContext.PushProperty(...)
    .Enrich.WithMachineName()             // adds MachineName
    .Enrich.WithThreadId()                // adds ThreadId
    .Enrich.WithEnvironmentName()         // adds EnvironmentName
    .Enrich.WithProperty("AppVersion", "2.1.0")  // static custom property
    .WriteTo.Console()
    .CreateLogger();
```

#### Pushing context properties dynamically

```csharp
using (LogContext.PushProperty("OrderId", orderId))
using (LogContext.PushProperty("UserId", userId))
{
    _logger.LogInformation("Payment started");
    _logger.LogInformation("Payment complete");
    // both entries carry OrderId and UserId
}
```

---

## Sinks

Sinks are output targets. Multiple sinks can be active at once.

| Sink | NuGet package |
|------|---------------|
| Console | `Serilog.Sinks.Console` |
| File | `Serilog.Sinks.File` |
| Elasticsearch | `Serilog.Sinks.Elasticsearch` |
| Seq | `Serilog.Sinks.Seq` |
| Application Insights | `Serilog.Sinks.ApplicationInsights` |

```csharp
.WriteTo.Console()
.WriteTo.File("logs/app-.log", rollingInterval: RollingInterval.Day)
.WriteTo.Seq("http://localhost:5341")
```

---

## Destructuring

Control how complex objects appear in log properties:

```csharp
// Default — calls .ToString()
_logger.LogInformation("Order: {Order}", order);

// Destructure — captures all properties as structured fields
_logger.LogInformation("Order: {@Order}", order);

// Capture as string explicitly
_logger.LogInformation("Order: {$Order}", order);
```

Custom destructuring policy:

```csharp
.Destructure.ByTransforming<Order>(o => new { o.Id, o.Amount })
```

---

## Request logging middleware

Replace per-action logging with a single middleware that logs every HTTP request:

```csharp
app.UseSerilogRequestLogging(opts =>
{
    opts.EnrichDiagnosticContext = (diagnosticContext, httpContext) =>
    {
        diagnosticContext.Set("RequestHost", httpContext.Request.Host.Value);
        diagnosticContext.Set("UserAgent", httpContext.Request.Headers["User-Agent"]);
    };
});
```

---

## Bootstrap logger (safe startup logging)

```csharp
Log.Logger = new LoggerConfiguration()
    .WriteTo.Console()
    .CreateBootstrapLogger();

try
{
    // build and run app
}
catch (Exception ex)
{
    Log.Fatal(ex, "Application failed to start");
}
finally
{
    Log.CloseAndFlush();
}
```

---

## Next step

→ **Phase 3** — Learn how Elasticsearch stores and indexes these log events.
