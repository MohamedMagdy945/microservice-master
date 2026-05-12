# ASP.NET Core Distributed Caching with Redis

Distributed caching in ASP.NET Core improves application performance and scalability by storing cached data outside the application memory, typically in Redis.

This guide covers the core components and configuration for using Redis as a distributed cache in ASP.NET Core.

---

# Table of Contents

1. IDistributedCache
2. AddStackExchangeRedisCache
3. Cache Expiration Policies
4. Sliding Expiration
5. Absolute Expiration
6. Distributed Session State

---

# 1. IDistributedCache

`IDistributedCache` is the core abstraction in ASP.NET Core for working with distributed caching systems.

It supports multiple backends such as:

* Redis
* SQL Server
* In-memory distributed cache

---

## Features

* Simple key-value API
* Async operations
* Backend-agnostic
* Supports serialization

---

## Example Usage

```csharp
public class CacheService
{
    private readonly IDistributedCache _cache;

    public CacheService(IDistributedCache cache)
    {
        _cache = cache;
    }

    public async Task SetValueAsync()
    {
        await _cache.SetStringAsync("key", "value");
    }

    public async Task<string?> GetValueAsync()
    {
        return await _cache.GetStringAsync("key");
    }
}
```

---

# 2. AddStackExchangeRedisCache

ASP.NET Core provides built-in support for Redis distributed caching.

---

## Installation

```bash
dotnet add package Microsoft.Extensions.Caching.StackExchangeRedis
```

---

## Configuration

### appsettings.json

```json
{
  "Redis": {
    "ConnectionString": "localhost:6379"
  }
}
```

---

## Register Redis Cache

```csharp
builder.Services.AddStackExchangeRedisCache(options =>
{
    options.Configuration = builder.Configuration["Redis:ConnectionString"];
    options.InstanceName = "MyApp_";
});
```

---

## Why Use It?

* Centralized caching
* Shared across multiple servers
* High performance
* Production-ready

---

# 3. Cache Expiration Policies

Expiration policies control how long cached data stays in Redis.

---

## Types of Expiration

* Sliding Expiration
* Absolute Expiration

---

## Cache Options Example

```csharp
var options = new DistributedCacheEntryOptions
{
    AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(10)
};

await _cache.SetStringAsync("key", "value", options);
```

---

# 4. Sliding Expiration

Sliding expiration resets the TTL every time the cache is accessed.

---

## Example

```csharp
var options = new DistributedCacheEntryOptions
{
    SlidingExpiration = TimeSpan.FromMinutes(5)
};

await _cache.SetStringAsync("key", "value", options);
```

---

## Behavior

| Access | TTL Reset         |
| ------ | ----------------- |
| Yes    | Resets expiration |
| No     | Expires naturally |

---

## Use Cases

* User sessions
* Frequently accessed data
* Active user state

---

# 5. Absolute Expiration

Absolute expiration sets a fixed lifetime for cached data.

---

## Example

```csharp
var options = new DistributedCacheEntryOptions
{
    AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(10)
};

await _cache.SetStringAsync("key", "value", options);
```

---

## Behavior

| Time         | Action                             |
| ------------ | ---------------------------------- |
| After 10 min | Cache expires regardless of access |

---

## Use Cases

* API response caching
* Static data
* Configuration values

---

# Sliding vs Absolute Expiration

| Feature         | Sliding  | Absolute    |
| --------------- | -------- | ----------- |
| Reset on access | Yes      | No          |
| Fixed lifetime  | No       | Yes         |
| Best for        | Sessions | Static data |

---

# 6. Distributed Session State

Redis can be used to store ASP.NET Core session state in a distributed way.

---

## Enable Session

### Register Services

```csharp
builder.Services.AddStackExchangeRedisCache(options =>
{
    options.Configuration = "localhost:6379";
});

builder.Services.AddSession();
```

---

## Configure Middleware

```csharp
app.UseSession();
```

---

## Using Session

```csharp
HttpContext.Session.SetString("username", "Mohamed");

var value = HttpContext.Session.GetString("username");
```

---

## Advantages of Distributed Session

* Works across multiple servers
* Survives application restart
* Scales horizontally
* Centralized session storage

---

## When to Use Redis Sessions

* Load-balanced applications
* Microservices architectures
* Cloud deployments

---

# Summary

ASP.NET Core distributed caching with Redis provides:

* High performance caching layer
* Shared cache across servers
* Flexible expiration strategies
* Reliable session storage

Using Redis with `IDistributedCache` is essential for building scalable and high-performance ASP.NET Core applications.
