# Advanced Redis with .NET

This guide covers advanced Redis features in .NET applications, including performance optimization, messaging patterns, and distributed coordination.

---

# Table of Contents

1. Redis Pipelines
2. Batch Operations
3. Lua Scripting
4. Transactions in .NET
5. Pub/Sub in ASP.NET Core
6. Streams in .NET
7. Distributed Locking in .NET
8. SignalR + Redis Backplane

---

# 1. Redis Pipelines

Pipelining allows sending multiple commands to Redis without waiting for each response.

## Benefits

* Reduces network round trips
* Improves throughput
* Increases performance under load

---

## Example

```csharp
var db = redis.GetDatabase();

var batch = db.CreateBatch();

var task1 = batch.StringSetAsync("key1", "value1");
var task2 = batch.StringSetAsync("key2", "value2");

batch.Execute();

await Task.WhenAll(task1, task2);
```

---

# 2. Batch Operations

Batch operations group multiple commands into a single execution flow.

---

## Example

```csharp
var db = redis.GetDatabase();
var batch = db.CreateBatch();

for (int i = 0; i < 10; i++)
{
    batch.StringSetAsync($"key:{i}", i);
}

batch.Execute();
```

---

## When to Use

* Bulk inserts
* Cache warming
* Large dataset updates

---

# 3. Lua Scripting

Lua scripts execute atomically inside Redis.

---

## Benefits

* Atomic execution
* Reduced network calls
* Complex logic support

---

## Example

```csharp
var script = @"
    local current = redis.call('GET', KEYS[1])
    if current == false then
        redis.call('SET', KEYS[1], ARGV[1])
        return ARGV[1]
    end
    return current
";

var result = await db.ScriptEvaluateAsync(
    script,
    new RedisKey[] { "counter" },
    new RedisValue[] { "1" }
);
```

---

# 4. Transactions in .NET

Redis transactions ensure multiple commands are executed atomically.

---

## Example

```csharp
var tran = db.CreateTransaction();

tran.StringSetAsync("key1", "value1");
tran.StringSetAsync("key2", "value2");

bool committed = await tran.ExecuteAsync();
```

---

## Important Notes

* Redis transactions are NOT rollback-based
* They are executed as a batch
* Failures do not rollback previous commands

---

# 5. Pub/Sub in ASP.NET Core

Redis Pub/Sub enables real-time messaging.

---

## Publish Message

```csharp
var sub = redis.GetSubscriber();

await sub.PublishAsync("chat", "Hello World");
```

---

## Subscribe

```csharp
await sub.SubscribeAsync("chat", (channel, message) =>
{
    Console.WriteLine(message);
});
```

---

## Use Cases

* Chat applications
* Notifications
* Real-time updates
* Event broadcasting

---

# 6. Streams in .NET

Redis Streams provide persistent event logs.

---

## Add Event

```csharp
await db.StreamAddAsync("orders", new NameValueEntry[]
{
    new("product", "Laptop"),
    new("price", "1200")
});
```

---

## Read Events

```csharp
var entries = await db.StreamReadAsync("orders", "0-0");
```

---

## Use Cases

* Event sourcing
* Messaging systems
* Audit logs
* Data pipelines

---

# 7. Distributed Locking in .NET

Distributed locks prevent multiple instances from accessing the same resource.

---

## Simple Lock Example

```csharp
var db = redis.GetDatabase();

bool locked = await db.StringSetAsync(
    "lock:order",
    "locked",
    TimeSpan.FromSeconds(10),
    When.NotExists
);
```

---

## Release Lock

```csharp
await db.KeyDeleteAsync("lock:order");
```

---

## Use Cases

* Background jobs
* Scheduled tasks
* Prevent duplicate processing

---

# 8. SignalR + Redis Backplane

Redis enables scaling SignalR across multiple servers.

---

## Installation

```bash
dotnet add package Microsoft.AspNetCore.SignalR.StackExchangeRedis
```

---

## Configuration

```csharp
builder.Services.AddSignalR()
    .AddStackExchangeRedis("localhost:6379");
```

---

## Why Use Redis Backplane?

Without Redis:

* Messages stay on single server

With Redis:

* Messages broadcast across all servers
* Enables horizontal scaling

---

## Use Cases

* Chat systems
* Live dashboards
* Real-time notifications
* Multiplayer applications

---

# Summary

Advanced Redis features in .NET enable:

* High-performance batching
* Atomic scripting with Lua
* Reliable transactions
* Real-time messaging (Pub/Sub)
* Event streaming (Streams)
* Distributed coordination (Locks)
* Scalable real-time systems (SignalR)

These patterns are essential for building enterprise-grade distributed systems with Redis and .NET.
