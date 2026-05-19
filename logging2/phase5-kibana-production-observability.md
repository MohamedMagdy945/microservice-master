# Phase 5 — Kibana, Production & Observability

## Overview

With logs flowing into Elasticsearch, this phase covers searching and visualizing them in Kibana, hardening your setup for production, and extending into full observability with Elastic APM.

---

## Kibana Discover

Discover is the primary interface for searching logs.

### First-time setup — create a Data View

1. Open Kibana → **Stack Management → Data Views**
2. Create a new data view matching your index pattern: `logs-myapp-*`
3. Set `@timestamp` as the time field

### Useful KQL queries

```kql
# All errors in the last hour
log.level: "Error"

# Errors for a specific service
log.level: "Error" AND service.name: "order-service"

# Trace a specific request end-to-end
trace.id: "abc123def456"

# Find slow operations (if duration is logged)
duration_ms > 1000

# Errors with a specific exception type
error.type: "System.TimeoutException"
```

### Saved searches
Save your most-used KQL filters as named searches and pin them to dashboards.

---

## Dashboards with Lens

Create visualizations in **Analytics → Dashboards → Create dashboard**:

| Visualization | What to build |
|---------------|---------------|
| Line chart | Error rate over time (`log.level: Error`) |
| Bar chart | Log volume by service |
| Data table | Top 10 error messages |
| Metric | Count of critical alerts today |
| Heat map | Errors by hour-of-day × day-of-week |

---

## Alerting

Set up alerts in **Stack Management → Alerting**:

```
Rule: Error spike
Condition: count of log.level:"Error" > 50 in last 5 minutes
Action: Send Slack notification / PagerDuty webhook
```

---

## ELK / EFK stack alternatives

For high-volume or multi-source logging, consider an ingestion layer before Elasticsearch:

### ELK (Logstash)
```
.NET app → Logstash → Elasticsearch → Kibana
```
Logstash can parse, transform, and route logs from many sources. Config example:

```ruby
input {
  beats { port => 5044 }
}
filter {
  json { source => "message" }
  date { match => ["@timestamp", "ISO8601"] }
}
output {
  elasticsearch {
    hosts => ["http://elasticsearch:9200"]
    index => "logs-%{[service][name]}-%{+yyyy.MM.dd}"
  }
}
```

### EFK (Fluentd / Filebeat)
```
.NET app → Filebeat → Elasticsearch → Kibana
```
Filebeat is lightweight (Go binary, ~10 MB RAM) and ideal for shipping file-based logs or container stdout.

---

## Production best practices

### 1. Minimum log level per environment
```json
{
  "Serilog": {
    "MinimumLevel": {
      "Default": "Warning",
      "Override": {
        "MyApp": "Information"
      }
    }
  }
}
```
Log `Warning` and above in production by default; keep `Information` only for your own namespaces.

### 2. Sampling high-volume logs
```csharp
// Only log 10% of Trace/Debug entries
.Filter.ByIncludingOnly(le =>
    le.Level > LogEventLevel.Debug ||
    (new Random().Next(10) == 0))
```

### 3. PII and sensitive field masking
- Never log passwords, tokens, credit card numbers, or personal emails
- Use destructuring policies (see Phase 4) to scrub before events leave the app

### 4. Index sizing and cost control
- Set ILM policies (Phase 3) — delete indices older than your retention requirement
- Use `_forcemerge` on cold indices to reduce segment count
- Disable `_source` on high-cardinality fields you never retrieve

### 5. Health monitoring
Add a liveness probe for the Elasticsearch sink:

```csharp
builder.Services.AddHealthChecks()
    .AddElasticsearch("http://localhost:9200");
```

---

## Elastic APM — beyond plain logs

Elastic APM gives you traces, metrics, and logs in one unified view.

```bash
dotnet add package Elastic.Apm.AspNetCore
dotnet add package Elastic.Apm.SerilogEnricher
```

```csharp
app.UseAllElasticApm(builder.Configuration);
```

`appsettings.json`:
```json
{
  "ElasticApm": {
    "ServerUrl": "http://localhost:8200",
    "ServiceName": "order-service",
    "Environment": "production",
    "LogLevel": "Warning"
  }
}
```

APM automatically captures:
- HTTP request transactions (latency, status codes)
- Database query spans (EF Core, Dapper)
- External HTTP calls
- Exception details with stack traces

Serilog enricher adds `transaction.id` and `trace.id` to every log entry, so you can click from a log line directly to the APM trace — and vice versa.

---

## Observability maturity model

```
Level 1 — Logs only (Phases 1–5)
Level 2 — Logs + Traces (add Elastic APM or OpenTelemetry)
Level 3 — Logs + Traces + Metrics (add Metricbeat / Prometheus + ES exporter)
Level 4 — Correlated (trace.id links all three pillars in Kibana)
```

---

## Checklist before going to production

- [ ] ILM policy configured with appropriate retention
- [ ] Minimum log levels tuned per environment
- [ ] Sensitive fields masked in destructuring policies
- [ ] Durable buffer enabled on the Elasticsearch sink
- [ ] Kibana Data View and error-rate dashboard created
- [ ] Alerting rule for error spikes configured
- [ ] Health check endpoint wired up
- [ ] `Log.CloseAndFlush()` called on app shutdown
