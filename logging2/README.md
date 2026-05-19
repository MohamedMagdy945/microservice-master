# .NET Logging with Elasticsearch — Roadmap

A practical, phase-by-phase guide to implementing structured logging in .NET applications and shipping logs to Elasticsearch for search, visualization, and alerting.

---

## Phases

| Phase | File | Topics |
|-------|------|--------|
| 1 | [phase1-dotnet-logging-fundamentals.md](./phase1-dotnet-logging-fundamentals.md) | `ILogger`, log levels, structured logging, providers |
| 2 | [phase2-serilog.md](./phase2-serilog.md) | Serilog setup, enrichers, sinks, destructuring |
| 3 | [phase3-elasticsearch.md](./phase3-elasticsearch.md) | Indices, ECS, ILM, .NET client |
| 4 | [phase4-connecting-dotnet-to-elasticsearch.md](./phase4-connecting-dotnet-to-elasticsearch.md) | Serilog ES sink, OpenTelemetry, buffering, correlation |
| 5 | [phase5-kibana-production-observability.md](./phase5-kibana-production-observability.md) | Kibana, dashboards, alerting, APM, production hardening |

---

## Recommended learning path

```
Phase 1 → Phase 2 → Phase 3 → Phase 4 → Phase 5
```

If you already know .NET logging basics, start at **Phase 2**.
If you only need the wiring code, jump to **Phase 4**.

---

## Quick start (minimum viable setup)

```bash
dotnet add package Serilog.AspNetCore
dotnet add package Serilog.Sinks.Elasticsearch
```

```csharp
// Program.cs
Log.Logger = new LoggerConfiguration()
    .Enrich.FromLogContext()
    .WriteTo.Console()
    .WriteTo.Elasticsearch(new ElasticsearchSinkOptions(new Uri("http://localhost:9200"))
    {
        IndexFormat = "logs-myapp-{0:yyyy.MM.dd}",
        AutoRegisterTemplate = true
    })
    .CreateLogger();

builder.Host.UseSerilog();
```

---

## Prerequisites

- .NET 6 / 7 / 8
- Elasticsearch 7.x or 8.x (local via Docker or hosted on Elastic Cloud)
- Kibana (optional, for visualization)

---

## Key packages

| Package | Purpose |
|---------|---------|
| `Microsoft.Extensions.Logging` | Built-in logging abstraction (included in .NET SDK) |
| `Serilog.AspNetCore` | Serilog integration for ASP.NET Core |
| `Serilog.Sinks.Elasticsearch` | Ships logs to Elasticsearch |
| `Elastic.CommonSchema.Serilog` | ECS-formatted log documents |
| `Elastic.Clients.Elasticsearch` | Query Elasticsearch from .NET |
| `Elastic.Apm.AspNetCore` | Full APM (traces + metrics + logs) |
