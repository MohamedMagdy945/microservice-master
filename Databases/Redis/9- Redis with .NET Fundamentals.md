# Redis with .NET Fundamentals

This guide covers how to use Redis in .NET applications using StackExchange.Redis, along with best practices for configuration, dependency injection, async operations, and serialization.

---

# Table of Contents

1. StackExchange.Redis
2. ConnectionMultiplexer
3. IDatabase
4. Dependency Injection
5. Redis Configuration in ASP.NET Core
6. Async Redis Operations
7. Serialization
8. JSON Serialization

---

# 1. StackExchange.Redis

StackExchange.Redis is the official high-performance Redis client for .NET.

## Features

* High performance
* Thread-safe
* Supports async operations
* Widely used in production

## Installation

```bash
dotnet add package StackExchange.Redis
```

---

# 2. ConnectionMultiplexer

ConnectionMultiplexer is the core object used to connect to Redis.

It manages:

* Connection pooling
* Thread safety
* Reconnection handling

## Example

```csharp
using StackExchange.Redis;

var redis = ConnectionMultiplexer.Connect("localhost:6379");
```

---

## Best Practice

👉 Use a single shared instance (Singleton)

```csharp
private static readonly ConnectionMultiplexer redis =
    ConnectionMultiplexer.Connect("localhost:6379");
```

---

# 3. IDatabase

IDatabase is used to execute Redis commands.

## Example

```csharp
IDatabase db = redis.GetDatabase();

await db.StringSetAsync("username", "Mohamed");
var value = await db.StringGetAsync("username");
```

---

## Key Notes

* Lightweight object
* Thread-safe
* Represents logical database
* Supports async APIs

---

# 4. Dependency Injection

In ASP.NET Core, Redis should be registered using DI.

---

## Register Redis

```csharp
builder.Services.AddSingleton<IConnectionMultiplexer>(sp =>
{
    var configuration = ConfigurationOptions.Parse("localhost:6379");
    return ConnectionMultiplexer.Connect(configuration);
});
```

---

## Use in Service

```csharp
public class RedisService
{
    private readonly IDatabase _db;

    public RedisService(IConnectionMultiplexer redis)
    {
        _db = redis.GetDatabase();
    }
}
```

---

# 5. Redis Configuration in ASP.NET Core

Configuration is usually stored in appsettings.json.

## appsettings.json

```json
{
  "Redis": {
    "ConnectionString": "localhost:6379"
  }
}
```

---

## Configuration Setup

```csharp
var redisConfig = builder.Configuration["Redis:ConnectionString"];

builder.Services.AddSingleton<IConnectionMultiplexer>(
    ConnectionMultiplexer.Connect(redisConfig)
);
```

---

# 6. Async Redis Operations

Redis operations should always use async methods for scalability.

---

## Example

```csharp
await db.StringSetAsync("key", "value");
var result = await db.StringGetAsync("key");
```

---

## Why Async?

* Non-blocking I/O
* Better scalability
* Improved API performance
* Handles high concurrency

---

# 7. Serialization

Redis stores data as strings or bytes.

Complex objects must be serialized.

---

## Example Object

```csharp
public class User
{
    public int Id { get; set; }
    public string Name { get; set; }
}
```

---

## Serialize Object

```csharp
var user = new User { Id = 1, Name = "Mohamed" };
var json = JsonSerializer.Serialize(user);

await db.StringSetAsync("user:1", json);
```

---

## Deserialize Object

```csharp
var json = await db.StringGetAsync("user:1");
var user = JsonSerializer.Deserialize<User>(json);
```

---

# 8. JSON Serialization

JSON is the most common format for Redis data in .NET apps.

---

## Using System.Text.Json

### Serialize

```csharp
var json = JsonSerializer.Serialize(obj);
```

### Deserialize

```csharp
var obj = JsonSerializer.Deserialize<MyClass>(json);
```

---

## Alternative: Newtonsoft.Json

```bash
dotnet add package Newtonsoft.Json
```

```csharp
var json = JsonConvert.SerializeObject(obj);
var obj = JsonConvert.DeserializeObject<MyClass>(json);
```

---

## Best Practices

* Prefer System.Text.Json (faster, built-in)
* Use compression for large objects
* Avoid storing huge JSON blobs
* Keep models lightweight

---

# Summary

Using Redis with .NET involves:

* StackExchange.Redis client
* ConnectionMultiplexer for connections
* IDatabase for operations
* Dependency Injection for clean architecture
* Async operations for scalability
* JSON serialization for complex objects

Mastering these fundamentals is essential for building high-performance distributed .NET systems with Redis.
