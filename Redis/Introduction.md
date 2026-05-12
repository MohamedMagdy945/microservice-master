# 🔴 Redis + .NET — Complete Documentation

> A comprehensive guide to using Redis with .NET — from theory and fundamentals to advanced production patterns.

---

## What is Redis?

**Redis** (Remote Dictionary Server) is an open-source, **in-memory data structure store**.
It was created by Salvatore Sanfilippo in 2009 and is now one of the most popular databases in the world.

**Redis** (Remote Dictionary Server) is an open-source, in-memory data structure store used as a database, cache, message broker, and streaming engine. It was created by **Salvatore Sanfilippo** in 2009 and is now one of the most popular databases in the world.

Redis stores data **in memory (RAM)**, which makes it extremely fast — capable of handling millions of operations per second with sub-millisecond latency.


### The Core Idea

Traditional databases store data **on disk**. Every read/write involves I/O operations, which are slow.
Redis stores everything **in RAM**, which is orders of magnitude faster:

```
Disk (HDD/SSD)  ──►  Read latency: ~1–10 ms
RAM             ──►  Read latency: ~0.1–0.5 ms   ← Redis lives here
CPU Cache       ──►  Read latency: ~0.001 ms
```

## Key Features

| Feature | Description |
|---|---|
| ⚡ **In-Memory Storage** | All data is stored in RAM for ultra-fast access |
| 🔄 **Persistence** | Optional disk persistence via RDB snapshots or AOF logs |
| 🗂️ **Rich Data Types** | Strings, Lists, Sets, Hashes, Sorted Sets, Streams, and more |
| 🔁 **Replication** | Master-replica replication for high availability |
| 🔀 **Pub/Sub Messaging** | Built-in publish/subscribe messaging system |
| 🧩 **Lua Scripting** | Atomic server-side scripts |
| 📦 **Clustering** | Horizontal scaling via Redis Cluster |
| 🔐 **Security** | AUTH passwords, TLS, ACL-based access control |

## Common Use Cases

- 🚀 **Caching** — Cache database query results, API responses, HTML pages
- 🔐 **Session Storage** — Fast, TTL-based session management
- 📊 **Leaderboards** — Sorted Sets for real-time rankings
- 📬 **Message Queues** — Lists or Streams as lightweight queues
- 🔔 **Pub/Sub** — Real-time notifications and event broadcasting
- 🔒 **Rate Limiting** — Atomic counters with expiry
- 🌐 **Geospatial** — `GEOADD` / `GEODIST` for location-aware apps



This makes Redis ideal for use cases where **speed is critical** — caching, sessions, real-time counters, leaderboards, and message passing.

### How Redis Differs from a Regular Database

| Feature | Relational DB (SQL) | Redis |
|---------|-------------------|-------|
| Storage | Disk | RAM (+ optional persistence) |
| Data model | Tables & rows | Key-Value with rich types |
| Query language | SQL | Simple commands (GET, SET, ZADD…) |
| Latency | 1–10ms | < 1ms |
| Persistence | Always | Optional (RDB / AOF) |
| Primary use | Source of truth | Cache, speed layer |

> **Important:** Redis is usually used **alongside** a database, not instead of it.
> Redis = fast access layer. SQL/NoSQL = permanent source of truth.

---

## Redis Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│                   .NET Application                       │
│                                                         │
│   ┌──────────┐    ┌──────────┐    ┌──────────┐         │
│   │   API    │    │ Services │    │  Workers │         │
│   └────┬─────┘    └────┬─────┘    └────┬─────┘         │
│        └───────────────┴───────────────┘                │
│                         │                               │
│         StackExchange.Redis / IDistributedCache         │
└─────────────────────────┬───────────────────────────────┘
                          │  TCP :6379
               ┌──────────▼──────────┐
               │     Redis Server     │
               │                     │
               │  String   Hash      │
               │  List     Set       │
               │  SortedSet Stream   │
               │  Pub/Sub  Lua       │
               └─────────────────────┘
```

Redis uses a **single-threaded event loop** for command execution — meaning it processes one command at a time, in order. This design eliminates race conditions and makes every individual command atomic by default.

---

## Single-Threaded Model — Why Is It Fast?

Many developers are surprised that Redis is single-threaded yet incredibly fast. Here's why:

```
Why single-threaded is OK for Redis:
──────────────────────────────────────────────────────────

  ❌ Traditional servers: threads wait for disk I/O
     Thread 1 → [waiting for disk...........] → respond
     Thread 2 → [waiting for disk...........] → respond
     (Need many threads to keep CPU busy)

  ✅ Redis: everything is in RAM, no waiting
     Event loop → GET key → respond in ~0.1ms → GET next → ...
     (No waiting = no need for threads)
```

Redis 6+ introduced **I/O threading** for reading/writing from network sockets,
but command execution itself remains single-threaded.

---

## Persistence — Does Redis Lose Data on Restart?

Redis supports two persistence mechanisms:

```
┌──────────────────────────────────────────────────────┐
│                  Persistence Options                  │
├───────────────────────┬──────────────────────────────┤
│   RDB (Snapshots)     │   AOF (Append Only File)     │
│                       │                              │
│   Save a snapshot of  │   Log every write command    │
│   the dataset at      │   to a file. Replay on       │
│   intervals.          │   restart.                   │
│                       │                              │
│   ✅ Compact file      │   ✅ More durable             │
│   ✅ Fast restart      │   ✅ Loses less data          │
│   ❌ May lose minutes  │   ❌ Larger files             │
│      of data          │   ❌ Slower restart           │
└───────────────────────┴──────────────────────────────┘
```

For a **pure cache**, you may disable persistence entirely — if Redis restarts, it simply re-populates from the database. For **session storage or queues**, enable AOF.

---

## Documentation Structure

```
redis-dotnet-docs/
│
├── 📄 README.md           ← You are here — theory & overview
├── 📄 SETUP.md            ← Installation & connection
├── 📄 BASICS.md           ← Data types & core operations
├── 📄 CACHING.md          ← Caching strategies & patterns
├── 📄 PATTERNS.md         ← Pub/Sub, Locking, Rate Limiting
├── 📄 CONFIGURATION.md    ← Production-ready settings
└── 📄 TROUBLESHOOTING.md  ← Common issues & diagnostics
```

---

## Learning Path

| Level | File | Topics |
|-------|------|--------|
| 🟢 Beginner | [SETUP.md](./SETUP.md) | Install Redis, first connection |
| 🟢 Beginner | [BASICS.md](./BASICS.md) | String, Hash, List, Set, SortedSet |
| 🟡 Intermediate | [CACHING.md](./CACHING.md) | Cache-Aside, IDistributedCache, Output Cache |
| 🔴 Advanced | [PATTERNS.md](./PATTERNS.md) | Pub/Sub, Distributed Lock, Rate Limiting |
| 🔴 Advanced | [CONFIGURATION.md](./CONFIGURATION.md) | Sentinel, Cluster, TLS, Memory |
| 🛠️ Support | [TROUBLESHOOTING.md](./TROUBLESHOOTING.md) | Errors, diagnostics, checklist |

---

## Prerequisites

- .NET 8+ SDK
- Docker (for quick Redis setup) or Redis installed locally
- Basic knowledge of C# and ASP.NET Core

---

## Core NuGet Packages

```bash
# Primary low-level client (most powerful)
dotnet add package StackExchange.Redis

# ASP.NET Core IDistributedCache integration
dotnet add package Microsoft.Extensions.Caching.StackExchangeRedis

# ASP.NET Core Output Cache integration
dotnet add package Microsoft.AspNetCore.OutputCaching.StackExchangeRedis
```

---

## 60-Second Quick Start

```bash
# 1. Run Redis via Docker
docker run -d -p 6379:6379 redis:latest

# 2. Add the NuGet package
dotnet add package StackExchange.Redis
```

```csharp
using StackExchange.Redis;

// Connect (create once, reuse everywhere — it's thread-safe)
var redis = ConnectionMultiplexer.Connect("localhost:6379");
var db = redis.GetDatabase();

// Write
await db.StringSetAsync("hello", "Redis is working! 🎉");

// Read
string? value = await db.StringGetAsync("hello");
Console.WriteLine(value); // Redis is working! 🎉

// Write with expiry (TTL)
await db.StringSetAsync("session:abc", "user:42", TimeSpan.FromHours(2));
```

---

> 💡 **Tip:** Start with [SETUP.md](./SETUP.md) if this is your first time using Redis with .NET.
# Redis — Introduction & Overview

![Redis](https://img.shields.io/badge/Redis-DC382D?style=for-the-badge&logo=redis&logoColor=white)
![License](https://img.shields.io/badge/License-BSD_3--Clause-blue.svg)

