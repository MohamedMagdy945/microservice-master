# Phase 2 — Elasticsearch Basics

**Goal:** Understand how Elasticsearch stores and searches logs, run it locally with Docker, and explore logs in Kibana — **without** .NET code first.

**Time:** 3–5 days

**Prerequisites:** [Phase 1 — .NET Logging](../Phase-01-DotNet-Logging/README.md), Docker Desktop installed

---

## 1. What is Elasticsearch?

### Theory

**Elasticsearch** is a search and analytics engine. For logging, it:

- Stores log events as **JSON documents**
- Lets you **search** and **filter** millions of rows quickly
- Aggregates data (counts, averages) for dashboards

It is part of the **Elastic Stack**:

| Product | Role |
|---------|------|
| **Elasticsearch** | Store and search data |
| **Kibana** | Web UI to explore data and build dashboards |
| **Logstash** (optional) | Transform and ship logs |
| **Beats** (e.g. Filebeat) | Lightweight shippers from files/servers |

In Phase 3, your .NET app will send logs **directly** to Elasticsearch (via Serilog sink). In larger systems, logs may go to a file first, then **Filebeat** ships them — same destination.

---

## 2. Core concepts

### Document

One log entry as JSON:

```json
{
  "@timestamp": "2026-05-19T10:15:00.000Z",
  "level": "Information",
  "message": "Order 1001 created",
  "fields": {
    "OrderId": 1001,
    "UserId": 42
  }
}
```

**Think:** one row in a log table.

### Index

A **collection of documents**. Often one index per day:

- `myapp-logs-2026.05.19`
- `myapp-logs-2026.05.20`

**Think:** like a database table (but optimized for search and time series).

### Mapping

**Schema** for fields: types like `keyword` (exact match), `text` (full-text search), `date`, `long`.

Wrong mapping → slow queries or failed searches. Elastic’s .NET ECS packages help by using standard field names.

### Data stream (modern approach)

A **data stream** is a named stream of logs (e.g. `logs-myapp-default`) that manages backing indices automatically. Elastic’s official .NET sink often writes to data streams.

---

## 3. Why Elasticsearch for logs?

### Theory

| Problem | How Elasticsearch helps |
|---------|-------------------------|
| Logs on 50 servers | Central place to search |
| “Find all errors for user X yesterday” | Query on `level` + `UserId` + time range |
| Dashboards for managers | Kibana charts from the same data |
| Retention (delete old logs) | Index Lifecycle Management (ILM) — Phase 4 |

---

## 4. Run Elasticsearch + Kibana locally (Docker)

### Theory

Docker runs Elasticsearch and Kibana in containers so you do not install Java or Elastic manually on Windows.

### docker-compose.yml

Create a folder `elastic-docker` and add:

```yaml
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.17.0
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ports:
      - "9200:9200"
    volumes:
      - esdata:/usr/share/elasticsearch/data

  kibana:
    image: docker.elastic.co/kibana/kibana:8.17.0
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    ports:
      - "5601:5601"
    depends_on:
      - elasticsearch

volumes:
  esdata:
```

**What each part does:**

| Setting | Purpose |
|---------|---------|
| `single-node` | One-node cluster for learning (not production HA) |
| `xpack.security.enabled=false` | Easier local dev (no login); **enable security in production** |
| `ES_JAVA_OPTS` | Limits memory so Docker does not run out |
| Port `9200` | Elasticsearch HTTP API |
| Port `5601` | Kibana web UI |
| `volumes: esdata` | Data survives container restart |

### Start

```powershell
cd elastic-docker
docker compose up -d
```

Wait 1–2 minutes, then verify:

```powershell
curl http://localhost:9200
```

You should see JSON with `"tagline" : "You Know, for Search"`.

Open Kibana: [http://localhost:5601](http://localhost:5601)

---

## 5. Manually index a log document

### Theory

Before .NET sends logs, prove Elasticsearch works by POSTing JSON yourself.

### Using PowerShell

```powershell
$body = @{
  "@timestamp" = (Get-Date).ToUniversalTime().ToString("o")
  level = "Information"
  message = "Hello from manual test"
  service = "learning"
} | ConvertTo-Json

Invoke-RestMethod -Uri "http://localhost:9200/manual-logs/_doc" `
  -Method Post -Body $body -ContentType "application/json"
```

**What the code does:**

- `manual-logs` — index name (created automatically if missing)
- `_doc` — adds one document
- Body is your first “structured log”

### Search it

```powershell
Invoke-RestMethod -Uri "http://localhost:9200/manual-logs/_search?pretty"
```

**What you see:** `hits.hits[]` contains your documents.

---

## 6. Kibana — Discover

### Theory

**Discover** is where you search logs interactively.

### Steps

1. Open Kibana → **Management** → **Stack Management** → **Data Views** (or Index Patterns in older UIs)
2. Create data view: `manual-logs*` or `logs-*` (after Phase 3)
3. Set time field: `@timestamp`
4. Go to **Discover**, pick a time range, run searches:
   - `level: "Error"`
   - `message: *payment*`

**What you are learning:** Same queries you will use when debugging production issues.

---

## 7. Query language (KQL) — basics

### Theory

Kibana uses **KQL** (Kibana Query Language) in Discover:

| Query | Meaning |
|-------|---------|
| `level : "Error"` | Errors only |
| `service : "LoggingDemo" and level : "Warning"` | Combine conditions |
| `message : *timeout*` | Message contains “timeout” |

Later, .NET logs with fields `OrderId`, `CorrelationId` become searchable the same way.

---

## 8. OpenSearch (optional note)

**OpenSearch** is a fork of Elasticsearch (common on AWS). Concepts (index, document, Dashboards UI) are almost the same. This roadmap uses Elastic’s official stack; skills transfer.

---

## 9. Mini project for Phase 2

1. Start Docker Compose stack
2. POST 5 manual documents with different `level` values
3. Create a Kibana data view
4. In Discover, filter to Errors only
5. Save a simple saved search

---

## Phase 2 checklist

- [ ] I can explain document, index, and mapping in plain language
- [ ] Elasticsearch and Kibana run locally on 9200 / 5601
- [ ] I indexed and searched at least one document manually
- [ ] I used Kibana Discover with a time range and filter

**Next:** [Phase 3 — Connect .NET to Elasticsearch](../Phase-03-DotNet-To-Elasticsearch/README.md)
