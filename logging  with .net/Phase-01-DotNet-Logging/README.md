# Phase 1 — Logging in .NET 9

**Goal:** Use the standard .NET logging stack (`ILogger`) and **Serilog** in an ASP.NET Core 9 web API.

**Time:** 3–5 days

**Prerequisites:** [Phase 0 — Foundations](../Phase-00-Foundations/README.md)

---

## 1. Microsoft.Extensions.Logging (`ILogger`)

### Theory

.NET has a built-in logging **abstraction**:

- **`ILogger<T>`** — write log messages
- **`ILoggerFactory`** — creates loggers
- **Providers** — send logs to console, debug window, EventSource, etc.
- **Configuration** — control levels in `appsettings.json`

In ASP.NET Core, the framework registers logging for you. You **inject** `ILogger<YourClass>` via the constructor.

**Why `ILogger<T>`?** The generic `T` sets the **category** (usually the class name), so you can filter logs per namespace or type.

---

## 2. Logging in ASP.NET Core 9

### Create the project

```powershell
dotnet new webapi -n LoggingDemo -f net9.0
cd LoggingDemo
```

### Example: controller with logging

```csharp
using Microsoft.AspNetCore.Mvc;

namespace LoggingDemo.Controllers;

[ApiController]
[Route("api/[controller]")]
public class WeatherController : ControllerBase
{
    private readonly ILogger<WeatherController> _logger;

    // DI injects a logger whose category is "WeatherController"
    public WeatherController(ILogger<WeatherController> logger)
    {
        _logger = logger;
    }

    [HttpGet]
    public IActionResult Get()
    {
        _logger.LogInformation("Weather endpoint called at {Time}", DateTime.UtcNow);
        return Ok(new { Temperature = 22 });
    }
}
```

### What the code does

| Part | Explanation |
|------|-------------|
| `ILogger<WeatherController>` | Typed logger; category = full type name |
| Constructor injection | ASP.NET Core provides the logger automatically |
| `LogInformation(...)` | Records a normal event with structured `Time` |

### Configure levels in appsettings.json

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  }
}
```

**What this does:**

- Your app logs at **Information** and above by default
- Framework noise from `Microsoft.AspNetCore` is reduced to **Warning** only

`appsettings.Development.json` can set `Default: Debug` for local work.

---

## 3. Scopes — grouping logs

### Theory

A **scope** adds properties to every log inside a `using` block — useful for `OrderId`, `TenantId`, etc.

```csharp
using (_logger.BeginScope(new Dictionary<string, object> { ["OrderId"] = orderId }))
{
    _logger.LogInformation("Validating order");
    _logger.LogInformation("Charging payment");
    // Both lines include OrderId in the scope
}
```

**What the code does:** `BeginScope` pushes context onto an async-local stack; providers that support scopes attach those fields to each log event.

---

## 4. High-performance logging (`LoggerMessage`)

### Theory

Normal `LogInformation` with parameters is fast enough for most apps. For **very hot paths** (millions of calls per second), .NET can **generate** logging code at compile time with `[LoggerMessage]`.

### Example

```csharp
public static partial class LogMessages
{
    [LoggerMessage(
        EventId = 1,
        Level = LogLevel.Information,
        Message = "Order {OrderId} created for {UserId}")]
    public static partial void OrderCreated(ILogger logger, int orderId, int userId);
}

// Usage:
LogMessages.OrderCreated(_logger, 1001, 42);
```

**What the code does:** The compiler generates an efficient implementation that avoids boxing and unnecessary allocations. Use this when you have measured a performance problem — not on day one.

**.NET 9 note:** Logger can be captured from primary constructors; see [Microsoft docs on source generation](https://learn.microsoft.com/en-us/dotnet/core/extensions/logging/source-generation).

---

## 5. Log buffering (.NET 9)

### Theory

**.NET 9** can **buffer** logs and flush them when you choose (e.g. after a request completes or on failure). Useful when you want many debug lines in memory but only emit them if something goes wrong.

Read when you reach production tuning: [Log buffering](https://learn.microsoft.com/en-us/dotnet/core/extensions/log-buffering).

---

## 6. Serilog — production-friendly logging

### Theory

**Serilog** is a popular third-party library that:

- Writes **structured** logs naturally
- Supports many **sinks** (Console, File, Elasticsearch — Phase 3)
- Enriches logs (machine name, environment, thread id)
- Reads configuration from `appsettings.json`

Pattern: Serilog often **replaces** default console logging while still implementing `ILogger`, so your controllers keep using `ILogger<T>`.

### Install packages

```powershell
dotnet add package Serilog.AspNetCore
dotnet add package Serilog.Sinks.Console
dotnet add package Serilog.Sinks.File
dotnet add package Serilog.Settings.Configuration
```

### Program.cs (ASP.NET Core 9)

```csharp
using Serilog;

// Build a Serilog logger before the host starts — catches startup errors too
Log.Logger = new LoggerConfiguration()
    .WriteTo.Console()
    .WriteTo.File("logs/app-.txt", rollingInterval: RollingInterval.Day)
    .Enrich.FromLogContext()
    .Enrich.WithProperty("Application", "LoggingDemo")
    .CreateLogger();

try
{
    var builder = WebApplication.CreateBuilder(args);

    // Replace default logging with Serilog
    builder.Host.UseSerilog((context, services, configuration) => configuration
        .ReadFrom.Configuration(context.Configuration)
        .ReadFrom.Services(services)
        .Enrich.FromLogContext()
        .WriteTo.Console()
        .WriteTo.File("logs/app-.txt", rollingInterval: RollingInterval.Day));

    builder.Services.AddControllers();
    var app = builder.Build();

    app.MapControllers();
    app.Run();
}
catch (Exception ex)
{
    Log.Fatal(ex, "Application terminated unexpectedly");
}
finally
{
    Log.CloseAndFlush();  // Ensures all buffered logs are written before exit
}
```

### What each part does

| Part | Explanation |
|------|-------------|
| `LoggerConfiguration()` | Fluent API to configure Serilog |
| `WriteTo.Console()` | Sink: print to terminal |
| `WriteTo.File(..., rollingInterval: Day)` | Sink: new file each day (`app-20260519.txt`) |
| `Enrich.FromLogContext()` | Picks up properties from `LogContext.PushProperty` |
| `UseSerilog(...)` | Hooks Serilog into ASP.NET Core’s logging pipeline |
| `ReadFrom.Configuration` | Loads levels/sinks from `appsettings.json` |
| `Log.CloseAndFlush()` | On shutdown, sends any buffered events to sinks |

### appsettings.json Serilog section (optional)

```json
{
  "Serilog": {
    "MinimumLevel": {
      "Default": "Information",
      "Override": {
        "Microsoft": "Warning"
      }
    }
  }
}
```

---

## 7. Request logging middleware

### Theory

You want every HTTP request logged once with: method, path, status code, duration. Serilog provides middleware for this.

```csharp
var app = builder.Build();

app.UseSerilogRequestLogging(options =>
{
    options.EnrichDiagnosticContext = (diagnosticContext, httpContext) =>
    {
        diagnosticContext.Set("RequestHost", httpContext.Request.Host.Value);
        diagnosticContext.Set("UserAgent", httpContext.Request.Headers.UserAgent.ToString());
    };
});

app.MapControllers();
app.Run();
```

**What the code does:** After each request, Serilog writes one Information log like `HTTP GET /api/weather responded 200 in 45ms` with extra fields you added.

---

## 8. Correlation ID middleware (pattern)

### Theory

Generate or read `X-Correlation-Id` header and push it into the log context for the whole request.

```csharp
app.Use(async (context, next) =>
{
    var correlationId = context.Request.Headers["X-Correlation-Id"].FirstOrDefault()
        ?? Guid.NewGuid().ToString();

    context.Response.Headers["X-Correlation-Id"] = correlationId;

    using (Serilog.Context.LogContext.PushProperty("CorrelationId", correlationId))
    {
        await next();
    }
});
```

**What the code does:**

1. Reuse client’s correlation ID or create a new GUID
2. Return it in the response header (client can store it for support tickets)
3. `PushProperty` adds `CorrelationId` to every Serilog event in this request

Place this **before** `UseSerilogRequestLogging`.

---

## 9. Mini project for Phase 1

Build a Web API with:

1. One GET endpoint that logs Information
2. One GET endpoint that throws — caught and logged as Error
3. Serilog → Console + rolling file
4. Correlation ID middleware
5. Request logging middleware

**Verify:** Call the API with curl or browser; see structured logs in console and `logs/` folder.

---

## Phase 1 checklist

- [ ] I can inject and use `ILogger<T>` in controllers
- [ ] I can set log levels in `appsettings.json`
- [ ] I understand Serilog sinks, enrichers, and `UseSerilog`
- [ ] I added request logging and correlation ID

**Next:** [Phase 2 — Elasticsearch Basics](../Phase-02-Elasticsearch-Basics/README.md)
