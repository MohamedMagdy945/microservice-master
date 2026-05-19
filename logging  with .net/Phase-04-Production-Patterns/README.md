# Phase 4 — Production Patterns

**Goal:** Operate logging safely at scale: retention, security, reliability, and cost control.

**Time:** 1–2 weeks

**Prerequisites:** [Phase 3 — Connect .NET to Elasticsearch](../Phase-03-DotNet-To-Elasticsearch/README.md)

---

## 1. Log only what you need

### Theory

Every log line costs:

- **Disk** in Elasticsearch
- **Network** from app to cluster
- **Your time** reading noise

**Production defaults:**

| Environment | Typical minimum level |
|-------------|------------------------|
| Development | Debug |
| Staging | Information |
| Production | Information (Warning for noisy libraries) |

### Code — appsettings.Production.json

```json
{
  "Serilog": {
    "MinimumLevel": {
      "Default": "Information",
      "Override": {
        "Microsoft": "Warning",
        "System": "Warning"
      }
    }
  }
}
```

**What the code does:** When `ASPNETCORE_ENVIRONMENT=Production`, Serilog loads this file and suppresses verbose framework logs.

### Avoid expensive logs

```csharp
// BAD in a tight loop — string built even if Debug is off (older patterns)
_logger.LogDebug("State: " + expensiveObject.ToString());

// BETTER — use structured template; still check level for very heavy work
if (_logger.IsEnabled(LogLevel.Debug))
{
    _logger.LogDebug("State: {State}", expensiveObject.Summary());
}
```

---

## 2. Index Lifecycle Management (ILM)

### Theory

Logs grow forever unless you **delete or move** old data. **ILM** automates:

| Phase | Meaning |
|-------|---------|
| **Hot** | Recent logs, fast disks, frequent writes |
| **Warm** | Older, less queried |
| **Cold** | Archive tier |
| **Delete** | Remove after N days |

**Daily indices** (classic): `app-logs-2026.05.19` — delete whole index after 30 days.

**Data streams** (modern): ILM policy attached to the backing index template.

### What you configure (conceptually)

1. **Policy:** e.g. hot 7 days → delete at 90 days
2. **Index template:** applies policy to `logs-*`
3. **Monitor:** disk usage in Elasticsearch

You configure ILM in Kibana **Stack Management → Index Lifecycle Policies** or via API. Exact clicks change by version; the **idea** is fixed: automate retention.

---

## 3. Reliability — do not lose logs when ES is down

### Theory

If the app **blocks** waiting for Elasticsearch, a slow cluster slows your API. If the sink **drops** logs silently, you lose audit trail.

**Patterns:**

| Pattern | How it helps |
|---------|----------------|
| **Buffered sink** | Queue logs in memory, retry with backoff |
| **Fallback sink** | Always write to file or stdout; ES is best-effort |
| **File + Filebeat** | App only writes to disk; agent handles ES outages |

### Serilog — multiple sinks

```csharp
.WriteTo.Console()
.WriteTo.File("logs/fallback-.json", rollingInterval: RollingInterval.Day)
.WriteTo.Elasticsearch(/* ... */, configureTransport: transport =>
{
    // Elastic client handles retries; tune batching in sink options
})
```

**What the code does:** Even if Elasticsearch is unavailable, files on disk still capture events for later replay.

---

## 4. Security

### Theory

Logs and Elasticsearch often contain sensitive metadata. Production requirements:

| Topic | Practice |
|-------|----------|
| **TLS** | HTTPS to Elasticsearch (`https://...`) |
| **Authentication** | API keys or username/password — never anonymous in prod |
| **Secrets in logs** | Never log passwords, tokens, full PAN |
| **Network** | ES not exposed to public internet |
| **RBAC** | Kibana roles so teams see only their indices |

### Elastic Cloud

Managed service with TLS and API keys built in. Connect sink with:

```csharp
// Conceptual — see Elastic.Serilog.Sinks docs for exact API
// CloudId + ApiKey authentication
```

### Redacting in Serilog

```csharp
// Destructure policies — mask properties named Password, Token, etc.
// See Serilog.Expressions or custom destructuring policies
```

---

## 5. Field design for search

### Theory

| Field type | Use for | Example query |
|------------|---------|----------------|
| **keyword** | Exact match | `service.name: "OrderingApi"` |
| **text** | Full-text search | `message: timeout` |
| **date** | Time range | `@timestamp` last 24h |
| **long/double** | Numeric filters | `http.response.status_code: 500` |

**ECS** (Elastic Common Schema) names fields consistently: `service.name`, `trace.id`, `error.message`. Using `Elastic.CommonSchema.Serilog` aligns your .NET logs with Elastic samples.

---

## 6. Correlation across microservices

### Theory

With multiple APIs:

1. Gateway generates or forwards `X-Correlation-Id`
2. Each service logs the same ID
3. **OpenTelemetry** adds `trace.id` across HTTP calls (Phase 5)

In Kibana: `CorrelationId: "abc-123"` shows all services for one user action.

### HttpClient propagation (pattern)

```csharp
builder.Services.AddHttpClient("Downstream", client =>
{
    client.BaseAddress = new Uri("https://other-service");
})
.AddHttpMessageHandler(() => new ForwardCorrelationIdHandler());

// Handler adds X-Correlation-Id from LogContext or HttpContext to outgoing requests
```

**What the code does:** Downstream service receives the same ID and includes it in its logs.

---

## 7. Health checks vs logging

### Theory

- **Logs** — narrative events (“payment failed”)
- **Health checks** — `/health` endpoint for load balancers (“is DB up?”)

Use both; do not replace health endpoints with logs only.

```csharp
builder.Services.AddHealthChecks();
app.MapHealthChecks("/health");
```

---

## 8. Monitoring the logging pipeline

### Theory

Watch:

- Elasticsearch cluster health (yellow/red)
- Disk watermark (runs out of space → index read-only)
- Ingest rate drops (app or sink broken)
- Error logs spike (Kibana alert)

**Kibana rules:** alert when `log.level: Error` count > threshold in 5 minutes.

---

## 9. Deployment checklist

- [ ] `Production` log level is Information, not Debug
- [ ] Elasticsearch requires authentication + TLS
- [ ] ILM or retention policy deletes old logs
- [ ] Fallback file/console sink configured
- [ ] No secrets in log messages
- [ ] Correlation ID on all HTTP services
- [ ] Dashboards for errors and latency
- [ ] Alerts on error rate and cluster health

---

## Phase 4 checklist

- [ ] I can explain ILM and why daily indices / data streams matter
- [ ] I configured production log levels via `appsettings.Production.json`
- [ ] I have a fallback if Elasticsearch is down
- [ ] I know how to secure Elasticsearch in production

**Next:** [Phase 5 — Beyond Plain Logs](../Phase-05-Beyond-Plain-Logs/README.md)
