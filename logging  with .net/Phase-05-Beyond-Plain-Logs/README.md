# Phase 5 — Beyond Plain Logs (Optional)

**Goal:** Connect logs with **traces** and **metrics** using OpenTelemetry and optionally Elastic APM — so you can debug distributed .NET 9 systems end to end.

**Time:** 1–2 weeks (optional)

**Prerequisites:** Phases 0–4 completed

---

## 1. Why go beyond plain logs?

### Theory

Logs alone answer: *“What happened?”*

They struggle with:

- *“Why was this request slow?”* → need **traces** (timing per step)
- *“Is CPU high on all pods?”* → need **metrics** (aggregates)

**Unified observability** = logs + metrics + traces, linked by shared IDs (`trace.id`, `span.id`).

```
Browser → API Gateway → Order Service → Payment Service → Database
              │              │                │
              └──────────────┴────────────────┴── same trace.id in all logs
```

---

## 2. OpenTelemetry for .NET 9

### Theory

**OpenTelemetry (OTel)** is a vendor-neutral standard. The .NET SDK:

- Creates **spans** (units of work with duration)
- Instruments ASP.NET Core, `HttpClient`, EF Core (via packages)
- Exports to Jaeger, Elastic, Azure Monitor, etc.
- Correlates **logs** with active spans automatically

### Key packages

```powershell
dotnet add package OpenTelemetry.Extensions.Hosting
dotnet add package OpenTelemetry.Instrumentation.AspNetCore
dotnet add package OpenTelemetry.Instrumentation.Http
dotnet add package OpenTelemetry.Exporter.OpenTelemetryProtocol
```

### Program.cs — minimal tracing

```csharp
using OpenTelemetry.Resources;
using OpenTelemetry.Trace;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddOpenTelemetry()
    .ConfigureResource(r => r.AddService("LoggingDemo"))
    .WithTracing(tracing => tracing
        .AddAspNetCoreInstrumentation()
        .AddHttpClientInstrumentation()
        .AddOtlpExporter());  // Send to collector or Jaeger/Elastic

builder.Services.AddControllers();
var app = builder.Build();
app.MapControllers();
app.Run();
```

### What each part does

| Part | Explanation |
|------|-------------|
| `AddService("LoggingDemo")` | Sets `service.name` on all telemetry |
| `AddAspNetCoreInstrumentation()` | Auto span per incoming HTTP request |
| `AddHttpClientInstrumentation()` | Spans for outgoing HTTP calls |
| `AddOtlpExporter()` | Exports spans via OTLP protocol (industry standard) |

### Log correlation

When a span is active, `ILogger` logs often include:

- `TraceId`
- `SpanId`

No extra code in many ASP.NET Core setups. See [OpenTelemetry log correlation](https://opentelemetry.io/docs/languages/dotnet/logs/correlation/).

**In Kibana:** search logs by `trace.id` and open the linked trace in APM UI.

---

## 3. Jaeger for local trace practice

### Theory

**Jaeger** is a trace UI (like Kibana for traces). Run locally to learn before sending traces to Elastic.

**Docker (simplified):** use Jaeger all-in-one image on ports `16686` (UI) and `4317` (OTLP).

Point `AddOtlpExporter` at `http://localhost:4317`.

**Tutorial:** [OpenTelemetry in .NET 9 with Jaeger](https://www.c-sharpcorner.com/article/opentelemetry-in-net-9-trace-requests-end-to-end-with-jaeger/)

---

## 4. Elastic APM .NET agent

### Theory

Elastic’s **APM agent** auto-instruments:

- Incoming requests
- Database calls
- HTTP exits

And links to ECS logs when using Elastic logging packages.

### Install (overview)

```powershell
dotnet add package Elastic.Apm.NetCoreAll
```

```csharp
// Program.cs — early in pipeline
app.UseElasticApm(builder.Configuration);
```

Configure in `appsettings.json` with `ElasticApm:ServerUrl` pointing to APM Server or Elastic Cloud.

**Docs:** [APM .NET agent](https://www.elastic.co/guide/en/apm/agent/dotnet/current/index.html)

### Serilog enricher

```powershell
dotnet add package Elastic.Apm.SerilogEnricher
```

Adds trace/transaction IDs to Serilog events so logs and traces match in Kibana.

---

## 5. Metrics (brief)

### Theory

**Metrics** are numbers over time: request rate, error rate, GC pauses.

OpenTelemetry .NET supports metrics exporters. Often paired with **Prometheus** + **Grafana**, or sent to Elastic **Metricbeat** / OTel collector.

Logs roadmap ends at introduction; deepen metrics when you need SLO dashboards.

---

## 6. Seq — excellent dev alternative

### Theory

**Seq** is a structured log server for local/dev (.NET-first). It is not Elasticsearch but teaches structured querying fast.

```powershell
docker run -d --restart unless-stopped -e ACCEPT_EULA=Y -p 5341:80 datalust/seq
```

```csharp
.WriteTo.Seq("http://localhost:5341")
```

Use Seq during development; ship to Elasticsearch in staging/production if desired.

---

## 7. Choosing your stack

| Scenario | Suggestion |
|----------|------------|
| Learning / small team | Serilog → Elasticsearch + Kibana |
| Kubernetes at scale | File or stdout JSON → Filebeat → ES |
| Full Elastic shop | ECS logs + APM agent + Kibana |
| Multi-cloud / vendor-neutral | OpenTelemetry → collector → backend of choice |
| Local dev only | Serilog → Console + Seq |

---

## 8. Final capstone project

Build a **.NET 9** solution with:

1. **API A** — creates order, logs with `OrderId`, propagates correlation ID
2. **API B** — payment simulation, called by A via HttpClient
3. **Serilog** → Elasticsearch (ECS)
4. **OpenTelemetry** traces exported to Jaeger or Elastic
5. **Kibana** dashboard: error count, request duration (from logs or APM)
6. **ILM** policy on log data stream (Phase 4)

**Success criteria:** One user request visible as correlated logs in Kibana and as a trace spanning both APIs.

---

## Phase 5 checklist

- [ ] I understand difference between logs, metrics, and traces
- [ ] I ran OpenTelemetry with ASP.NET Core instrumentation
- [ ] I know how `trace.id` links logs and traces
- [ ] I reviewed Elastic APM or Seq as optional tools

---

## Congratulations

You completed the roadmap:

0. Foundations → 1. .NET logging → 2. Elasticsearch → 3. Integration → 4. Production → 5. Observability

Return to the [main README](../README.md) or [Learning Resources](../LEARNING-RESOURCES.md) anytime.
