# Phase 4 — Connecting .NET Logging → Elasticsearch

## Overview

This phase wires together everything from Phases 1–3: .NET's logging abstraction, Serilog's enrichment pipeline, and Elasticsearch as the destination. You have two main approaches — the Serilog sink (simpler) and OpenTelemetry (more portable).

---

## Approach A — Serilog Elasticsearch sink

### Packages

```bash
dotnet add package Serilog.AspNetCore
dotnet add package Serilog.Sinks.Elasticsearch
```

### Configuration

```csharp
// Program.cs
using Serilog;
using Serilog.Sinks.Elasticsearch;

Log.Logger = new LoggerConfiguration()
    .ReadFrom.Configuration(builder.Configuration)
    .Enrich.FromLogContext()
    .Enrich.WithMachineName()
    .Enrich.WithEnvironmentName()
    .WriteTo.Console()
    .WriteTo.Elasticsearch(new ElasticsearchSinkOptions(new Uri("http://localhost:9200"))
    {
        IndexFormat          = "logs-myapp-{0:yyyy.MM.dd}",
        AutoRegisterTemplate = true,
        NumberOfShards       = 1,
        NumberOfReplicas     = 0,
        DetectElasticsearchVersion = true,
        ModifyConnectionSettings = conn =>
            conn.BasicAuthentication("elastic", "changeme")
    })
    .CreateLogger();

builder.Host.UseSerilog();
```

### Via `appsettings.json`

```json
{
  "Serilog": {
    "MinimumLevel": { "Default": "Information" },
    "WriteTo": [
      { "Name": "Console" },
      {
        "Name": "Elasticsearch",
        "Args": {
          "nodeUris": "http://localhost:9200",
          "indexFormat": "logs-myapp-{0:yyyy.MM.dd}",
          "autoRegisterTemplate": true
        }
      }
    ],
    "Enrich": ["FromLogContext", "WithMachineName", "WithEnvironmentName"]
  }
}
```

---

## ECS formatting (recommended)

Use `Elastic.CommonSchema.Serilog` to emit logs in Elastic Common Schema format — Kibana dashboards and APM work out of the box:

```bash
dotnet add package Elastic.CommonSchema.Serilog
```

```csharp
.WriteTo.Elasticsearch(new ElasticsearchSinkOptions(new Uri("http://localhost:9200"))
{
    CustomFormatter = new EcsTextFormatter()
})
```

---

## Approach B — OpenTelemetry (modern, vendor-neutral)

```bash
dotnet add package OpenTelemetry.Extensions.Hosting
dotnet add package OpenTelemetry.Exporter.OpenTelemetryProtocol
```

```csharp
builder.Services.AddOpenTelemetry()
    .WithLogging(logging =>
    {
        logging.AddOtlpExporter(o =>
        {
            o.Endpoint = new Uri("http://localhost:4317"); // OTel collector
        });
    });
```

The OTel collector forwards to Elasticsearch via its `elasticsearchexporter` or the Elastic APM fleet. This approach keeps your app code independent of Elasticsearch's SDK.

---

## Buffering and resilience

Logs must not be lost when Elasticsearch is temporarily unreachable.

### Durable file buffer (Serilog)

```bash
dotnet add package Serilog.Sinks.Elasticsearch
```

```csharp
.WriteTo.Elasticsearch(new ElasticsearchSinkOptions(new Uri("http://localhost:9200"))
{
    BufferBaseFilename = "./logs/buffer",       // durable disk buffer
    BufferFileSizeLimitBytes = 104857600,       // 100 MB per buffer file
    BufferFileCountLimit = 5,
    BatchPostingLimit = 500,                    // max docs per bulk request
    Period = TimeSpan.FromSeconds(2)            // flush interval
})
```

### Bulk indexing
The sink already uses Elasticsearch's `_bulk` API internally — each flush sends a batch, not individual HTTP calls.

---

## Correlation and distributed tracing

Attach the current `Activity` (from `System.Diagnostics`) to every log entry:

```csharp
.Enrich.WithProperty("TraceId",
    Activity.Current?.TraceId.ToString() ?? string.Empty)
.Enrich.WithProperty("SpanId",
    Activity.Current?.SpanId.ToString() ?? string.Empty)
```

Or use the dedicated enricher:

```bash
dotnet add package Serilog.Enrichers.Span
```

```csharp
.Enrich.WithSpan()
```

This adds `TraceId` and `SpanId` to every log, letting you pivot from a Kibana log entry directly to the matching APM trace.

---

## Filtering sensitive data

Strip PII before it reaches Elasticsearch:

```csharp
.Filter.ByExcluding(le =>
    le.Properties.ContainsKey("Password") ||
    le.Properties.ContainsKey("CreditCard"))
```

Or destructure with a sanitising policy:

```csharp
.Destructure.ByTransforming<UserDto>(u => new
{
    u.Id,
    Email = "***"   // mask PII
})
```

---

## Full working `Program.cs` example

```csharp
using Serilog;
using Serilog.Sinks.Elasticsearch;

var builder = WebApplication.CreateBuilder(args);

Log.Logger = new LoggerConfiguration()
    .ReadFrom.Configuration(builder.Configuration)
    .Enrich.FromLogContext()
    .Enrich.WithMachineName()
    .Enrich.WithEnvironmentName()
    .Enrich.WithSpan()
    .Filter.ByExcluding(le => le.Properties.ContainsKey("Password"))
    .WriteTo.Console()
    .WriteTo.Elasticsearch(new ElasticsearchSinkOptions(
        new Uri(builder.Configuration["Elasticsearch:Uri"]!))
    {
        IndexFormat          = $"logs-{builder.Environment.ApplicationName.ToLower()}-{{0:yyyy.MM.dd}}",
        AutoRegisterTemplate = true,
        BufferBaseFilename   = "./logs/buffer",
        BatchPostingLimit    = 500,
        Period               = TimeSpan.FromSeconds(2)
    })
    .CreateLogger();

builder.Host.UseSerilog();

// ... rest of setup
```

---

## Next step

→ **Phase 5** — Search and visualize logs in Kibana; harden for production.
