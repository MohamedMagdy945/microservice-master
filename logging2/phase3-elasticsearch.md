# Phase 3 — Elasticsearch

## Overview

Elasticsearch is a distributed search and analytics engine built on Apache Lucene. For logging, it acts as the central store where every log event becomes a JSON document, indexed and instantly searchable at scale.

---

## Core concepts for logging

### Documents
Each log entry is a JSON document stored in an index:

```json
{
  "@timestamp": "2025-05-19T10:32:45.123Z",
  "level": "Information",
  "message": "Order 4521 placed by user 99",
  "OrderId": 4521,
  "UserId": 99,
  "MachineName": "web-01",
  "Environment": "Production"
}
```

### Indices
A named collection of documents. For logs, use date-based rolling indices:

```
logs-myapp-2025.05.19
logs-myapp-2025.05.20
logs-myapp-2025.05.21
```

### Mappings
Define the data types of each field. For logs, dynamic mapping usually works — Elasticsearch infers types automatically. You may want to explicitly map fields like `@timestamp` (date) and `level` (keyword).

---

## Elastic Common Schema (ECS)

ECS is a standardized set of field names that makes logs from different services interoperable with Kibana dashboards and Elastic APM out of the box.

Key ECS fields for application logs:

| ECS field | Description |
|-----------|-------------|
| `@timestamp` | When the event occurred |
| `log.level` | Severity (info, warn, error…) |
| `log.logger` | Logger category / class name |
| `message` | Human-readable description |
| `error.message` | Exception message |
| `error.stack_trace` | Full stack trace |
| `service.name` | Application name |
| `service.version` | Deployment version |
| `host.name` | Machine / pod name |
| `trace.id` | Distributed trace ID |
| `transaction.id` | Span ID |

The Serilog Elasticsearch sink has ECS formatting built in (see Phase 4).

---

## Index Lifecycle Management (ILM)

ILM automatically manages index retention across four phases:

```
Hot  →  Warm  →  Cold  →  Delete
(active writes)  (read only)  (compressed)  (purge)
```

Example ILM policy — keep logs for 30 days, roll over daily or at 50 GB:

```json
PUT _ilm/policy/logs-policy
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
          "rollover": {
            "max_primary_shard_size": "50gb",
            "max_age": "1d"
          }
        }
      },
      "warm": {
        "min_age": "7d",
        "actions": { "shrink": { "number_of_shards": 1 } }
      },
      "delete": {
        "min_age": "30d",
        "actions": { "delete": {} }
      }
    }
  }
}
```

---

## Index template

Set up an index template so every new `logs-*` index inherits the right settings:

```json
PUT _index_template/logs-template
{
  "index_patterns": ["logs-*"],
  "template": {
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 1,
      "index.lifecycle.name": "logs-policy"
    },
    "mappings": {
      "properties": {
        "@timestamp": { "type": "date" },
        "level":      { "type": "keyword" },
        "message":    { "type": "text" },
        "MachineName":{ "type": "keyword" },
        "Environment":{ "type": "keyword" }
      }
    }
  }
}
```

---

## Official .NET client

Elastic ships two .NET packages:

| Package | Use case |
|---------|----------|
| `Elastic.Clients.Elasticsearch` | Modern 8.x client, generated from the API spec |
| `NEST` | Older 7.x high-level client (still widely used) |

```bash
dotnet add package Elastic.Clients.Elasticsearch
```

```csharp
var client = new ElasticsearchClient(new Uri("http://localhost:9200"));

var response = await client.SearchAsync<LogDocument>(s => s
    .Index("logs-*")
    .Query(q => q.Match(m => m.Field(f => f.Level).Query("Error")))
    .Size(50)
);
```

---

## Local Elasticsearch with Docker

```yaml
# docker-compose.yml
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.13.0
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
    ports:
      - "9200:9200"

  kibana:
    image: docker.elastic.co/kibana/kibana:8.13.0
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
```

```bash
docker-compose up -d
```

---

## Next step

→ **Phase 4** — Wire Serilog to Elasticsearch in your .NET app.
