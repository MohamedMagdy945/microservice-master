# Phase 3 — Connect .NET 9 to Elasticsearch

**Goal:** Send logs from an ASP.NET Core 9 app to Elasticsearch using official Elastic packages and view them in Kibana.

**Time:** 5–7 days

**Prerequisites:** [Phase 1](../Phase-01-DotNet-Logging/README.md), [Phase 2](../Phase-02-Elasticsearch-Basics/README.md)

---

## 1. Architecture options

### Theory

```
Option A — Direct (simplest for learning)
┌─────────────┐    Serilog + Elastic.Serilog.Sinks    ┌───────────────┐
│  .NET 9 API │ ────────────────────────────────────► │ Elasticsearch │
└─────────────┘                                       └───────┬───────┘
                                                              │
                                                      ┌───────▼───────┐
                                                      │    Kibana     │
                                                      └───────────────┘

Option B — File then ship (common in production)
┌─────────────┐    JSON logs to file    ┌──────────┐    ┌───────────────┐
│  .NET 9 API │ ───────────────────────► │ Filebeat │ ─► │ Elasticsearch │
└─────────────┘                         └──────────┘    └───────────────┘
```

**This phase uses Option A** so you see end-to-end flow quickly. Option B decouples your app from Elasticsearch outages (Phase 4).

---

## 2. Packages to use

### Theory

Elastic maintains **ECS Logging .NET** — logs shaped for the [Elastic Common Schema](https://www.elastic.co/guide/en/ecs/current/index.html), which makes Kibana dashboards easier.

| Package | Purpose |
|---------|---------|
| `Serilog.AspNetCore` | Serilog + ASP.NET Core integration |
| `Elastic.Serilog.Sinks` | Send Serilog events to Elasticsearch |
| `Elastic.CommonSchema.Serilog` | ECS-formatted JSON (optional but recommended) |

### Install

```powershell
dotnet new webapi -n LoggingToElastic -f net9.0
cd LoggingToElastic

dotnet add package Serilog.AspNetCore
dotnet add package Elastic.Serilog.Sinks
dotnet add package Elastic.CommonSchema.Serilog
```

---

## 3. Program.cs — full example

```csharp
using Elastic.CommonSchema.Serilog;
using Elastic.Ingest.Elasticsearch;
using Elastic.Ingest.Elasticsearch.DataStreams;
using Elastic.Serilog.Sinks;
using Serilog;

Log.Logger = new LoggerConfiguration()
    .Enrich.FromLogContext()
  .WriteTo.Console(new EcsTextFormatter())  // Human-readable ECS JSON in dev
    .WriteTo.Elasticsearch(
        new[] { new Uri("http://localhost:9200") },
        opts =>
        {
            opts.DataStream = new DataStreamName("logs", "logging-to-elastic", "demo");
            opts.BootstrapMethod = BootstrapMethod.Failure;
        })
    .CreateLogger();

try
{
    var builder = WebApplication.CreateBuilder(args);
    builder.Host.UseSerilog();

    builder.Services.AddControllers();

    var app = builder.Build();

    app.Use(async (context, next) =>
    {
        var correlationId = context.Request.Headers["X-Correlation-Id"].FirstOrDefault()
            ?? Guid.NewGuid().ToString();
        context.Response.Headers["X-Correlation-Id"] = correlationId;
        using (Serilog.Context.LogContext.PushProperty("CorrelationId", correlationId))
            await next();
    });

    app.UseSerilogRequestLogging();
    app.MapControllers();
    app.Run();
}
catch (Exception ex)
{
    Log.Fatal(ex, "Startup failed");
}
finally
{
    Log.CloseAndFlush();
}
```

### What each part does

| Code | Explanation |
|------|-------------|
| `EcsTextFormatter()` | Formats console output as ECS-compatible JSON |
| `WriteTo.Elasticsearch(...)` | Registers the sink that batches and POSTs logs to ES |
| `Uri("http://localhost:9200")` | Elasticsearch address (Docker from Phase 2) |
| `DataStreamName("logs", "logging-to-elastic", "demo")` | Target data stream: type `logs`, dataset `logging-to-elastic`, namespace `demo` |
| `BootstrapMethod.Failure` | If template setup fails, log failure (safe default while learning) |
| `UseSerilog()` | ASP.NET uses Serilog for `ILogger` |
| Correlation middleware | Adds `CorrelationId` to every log event |
| `UseSerilogRequestLogging()` | One log per HTTP request with timing |

---

## 4. Controller example

```csharp
using Microsoft.AspNetCore.Mvc;

namespace LoggingToElastic.Controllers;

[ApiController]
[Route("api/[controller]")]
public class OrdersController : ControllerBase
{
    private readonly ILogger<OrdersController> _logger;

    public OrdersController(ILogger<OrdersController> logger) => _logger = logger;

    [HttpPost("{id:int}")]
    public IActionResult Create(int id)
    {
        _logger.LogInformation("Creating order {OrderId}", id);
        return Ok(new { OrderId = id, Status = "Created" });
    }

    [HttpGet("fail")]
    public IActionResult Fail()
    {
        _logger.LogError("Simulated failure for order pipeline");
        return StatusCode(500, "Simulated error");
    }
}
```

**What the code does:** Uses standard `ILogger` — Serilog captures it and ships to Elasticsearch with structured `OrderId`.

---

## 5. appsettings.json (environment-specific URLs)

### Theory

Do not hardcode production URLs. Use configuration:

```json
{
  "Elastic": {
    "Uri": "http://localhost:9200"
  },
  "Serilog": {
    "MinimumLevel": {
      "Default": "Information",
      "Override": {
        "Microsoft.AspNetCore": "Warning"
      }
    }
  }
}
```

In `Program.cs`, read `builder.Configuration["Elastic:Uri"]` when building `WriteTo.Elasticsearch`. For **Elastic Cloud**, use the cloud endpoint and API key (see Elastic docs).

---

## 6. View logs in Kibana

### Steps

1. Run Docker Elasticsearch + Kibana (Phase 2)
2. Run your API: `dotnet run`
3. Call endpoints:
   ```powershell
   curl -X POST http://localhost:5000/api/orders/1001
   curl http://localhost:5000/api/orders/fail
   ```
4. In Kibana → **Discover**
5. Create data view: `logs-*` or `logs-logging-to-elastic-*` (exact name depends on data stream created)
6. Time field: `@timestamp`
7. Search: `log.level: "Error"` or `CorrelationId: *`

**What you should see:** Request logs, your `Creating order` message, and the simulated error — all with timestamps and fields.

---

## 7. Alternative: Elastic.Extensions.Logging (no Serilog)

### Theory

If you prefer only Microsoft logging:

```csharp
builder.Logging.AddElasticsearch();
```

See: [Elastic.Extensions.Logging](https://www.elastic.co/guide/en/ecs-logging/dotnet/current/extensions-logging-data-shipper.html)

Serilog is more common for flexible sinks and enrichers; both are valid.

---

## 8. Troubleshooting

| Symptom | Check |
|---------|--------|
| No logs in Kibana | Is Elasticsearch up? `curl localhost:9200` |
| Connection refused | Wrong URI, or Docker not running |
| Wrong data view | List indices: `GET http://localhost:9200/_cat/indices?v` |
| App starts but no logs | Minimum level too high; sink exceptions in console on startup |
| Security enabled on ES | Use HTTPS + API key in sink config |

Enable Serilog **SelfLog** during debugging:

```csharp
Serilog.Debugging.SelfLog.Enable(msg => Console.WriteLine("SERILOG: " + msg));
```

---

## 9. Mini project for Phase 3

1. Web API on `net9.0` with Serilog + Elastic sink
2. Docker ES + Kibana
3. Correlation ID + request logging
4. Two endpoints: success + error
5. Kibana Discover saved search for errors
6. Optional: one Kibana visualization (count of errors per hour)

---

## Phase 3 checklist

- [ ] .NET 9 app sends logs to local Elasticsearch
- [ ] I understand data stream naming
- [ ] I can find my logs in Kibana Discover
- [ ] Correlation ID appears on request logs

**Next:** [Phase 4 — Production Patterns](../Phase-04-Production-Patterns/README.md)
