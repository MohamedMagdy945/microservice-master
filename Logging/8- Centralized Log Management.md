# Centralized Log Management

## Overview

Centralized log management is the practice of collecting logs from multiple applications, services, or servers into a single system. This makes it easier to search, analyze, and monitor logs in one place instead of checking each machine separately.

It is essential in distributed systems and cloud environments.

---

## Why Centralization is Important

Without centralized logging:

* Logs are scattered across multiple servers
* Debugging production issues is slow
* Correlating events is difficult
* Monitoring becomes inconsistent

With centralized logging:

* All logs are in one place
* Fast searching and filtering
* Easier debugging across services
* Better observability and alerting

---

## Common Centralized Logging Tools

### Elasticsearch + Kibana

* Stores and indexes logs
* Provides powerful search and dashboards

### Seq

* Designed specifically for structured logs (like Serilog)
* Easy to use and developer-friendly

### Splunk

* Enterprise-grade log analytics platform
* Strong for large-scale systems

### Grafana Loki

* Lightweight log aggregation system
* Works well with Grafana dashboards

---

## How It Works

Typical flow:

```
Application → Logging Library → Log Sink → Central System → UI Dashboard
```

Example:

* App writes logs using Serilog
* Logs are sent to Elasticsearch
* Kibana displays and searches logs

---

## Example with Serilog + File + Central System

```csharp id="cl1"
Log.Logger = new LoggerConfiguration()
    .WriteTo.Console()
    .WriteTo.File("logs/app.log")
    .CreateLogger();
```

In production, file logs are often shipped to a central system.

---

## Structured Logs in Central Systems

Centralized logging works best with structured logs:

```csharp id="cl2"
logger.LogInformation("User {UserId} created Order {OrderId}", userId, orderId);
```

This allows filtering like:

* Find all logs for a specific UserId
* Track all events for an OrderId

---

## Log Shipping Methods

Logs are transferred to central systems using:

* Agents (Filebeat, Fluentd, Vector)
* Direct API shipping
* Streaming pipelines (Kafka, RabbitMQ)

---

## Benefits

* Single source of truth for logs
* Faster incident response
* Easier monitoring and alerting
* Better system visibility
* Historical log analysis

---

## Best Practices

* Always use structured logging
* Include correlation IDs in logs
* Avoid storing sensitive data
* Use log retention policies
* Separate environments (dev, staging, prod)
* Index logs properly for fast search

---

## Common Mistakes

* Sending unstructured logs (hard to analyze)
* Overloading system with unnecessary logs
* Not setting retention policies
* Mixing logs from different environments
* Ignoring performance impact of log shipping

---

## Summary

* Centralized logging collects logs in one system
* Essential for microservices and distributed systems
* Improves debugging, monitoring, and observability
* Works best with structured logging and correlation IDs

---

Next Topic: Testing Logging in Unit Tests
