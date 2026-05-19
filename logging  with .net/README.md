# .NET 9 Logging → Elasticsearch Roadmap

Welcome! This folder is a **step-by-step learning guide** for developers who are starting from scratch. You do not need prior experience with logging, Elasticsearch, or advanced .NET.

## How to use this guide

1. Read phases **in order** (Phase 0 → Phase 5).
2. Each phase has its own folder with a `README.md`.
3. Every README explains **theory first**, then **code** and what each line does.
4. Type the examples yourself when possible — that helps memory more than copy-paste alone.

## What you will build (end state)

By the end, you will understand how to:

- Write structured logs in a **.NET 9** application
- Send those logs to **Elasticsearch**
- Search and visualize them in **Kibana**
- Apply basic **production** practices (retention, security, correlation)

## Roadmap overview

| Phase | Folder | Topic | Estimated time |
|-------|--------|--------|----------------|
| 0 | [Phase-00-Foundations](./Phase-00-Foundations/README.md) | What logging is, levels, structured logs | 1–2 days |
| 1 | [Phase-01-DotNet-Logging](./Phase-01-DotNet-Logging/README.md) | `ILogger`, Serilog, ASP.NET Core | 3–5 days |
| 2 | [Phase-02-Elasticsearch-Basics](./Phase-02-Elasticsearch-Basics/README.md) | Elasticsearch & Kibana concepts | 3–5 days |
| 3 | [Phase-03-DotNet-To-Elasticsearch](./Phase-03-DotNet-To-Elasticsearch/README.md) | Wire .NET 9 to Elasticsearch | 5–7 days |
| 4 | [Phase-04-Production-Patterns](./Phase-04-Production-Patterns/README.md) | ILM, security, reliability | 1–2 weeks |
| 5 | [Phase-05-Beyond-Plain-Logs](./Phase-05-Beyond-Plain-Logs/README.md) | OpenTelemetry, APM (optional) | 1–2 weeks |

## Prerequisites

- [.NET 9 SDK](https://dotnet.microsoft.com/download/dotnet/9.0) installed
- A code editor (Visual Studio, VS Code, or Rider)
- [Docker Desktop](https://www.docker.com/products/docker-desktop/) (for Phase 2+ — runs Elasticsearch locally)
- Basic C# knowledge (variables, classes, `async`/`await` helps but is not required for Phase 0–1)

## Quick glossary

| Term | Simple meaning |
|------|----------------|
| **Log** | A text record of something that happened in your app |
| **Structured log** | A log stored as fields (JSON), not one long sentence |
| **Sink** | Where logs are written (console, file, Elasticsearch) |
| **Index** | Elasticsearch storage for logs (like a table) |
| **Kibana** | Web UI to search and chart logs in Elasticsearch |

## Extra: learning links

See [LEARNING-RESOURCES.md](./LEARNING-RESOURCES.md) for official Microsoft, Elastic, and OpenTelemetry documentation (.NET 9 focused).

## Download this guide

This folder lives in your workspace. To get a single file you can share or move:

- **Windows (PowerShell):** run from the parent folder:
  ```powershell
  Compress-Archive -Path "dotnet-logging-elasticsearch-roadmap" -DestinationPath "dotnet-logging-elasticsearch-roadmap.zip" -Force
  ```
- Then open `dotnet-logging-elasticsearch-roadmap.zip` from File Explorer.

In **Cursor**, you can also right-click the folder in the file tree and use your environment’s download/export option if available.

---

**Start here:** [Phase 00 — Foundations](./Phase-00-Foundations/README.md)
