# Redis Repository Patterns in .NET

This guide explains how to structure Redis usage in .NET applications using clean architecture principles, repository patterns, caching abstractions, CQRS, and MediatR.

---

# Table of Contents

1. Redis Service Layer
2. Repository Pattern
3. Generic Cache Services
4. Cache Abstractions
5. Clean Architecture with Redis
6. CQRS + Redis
7. MediatR + Redis

---

# 1. Redis Service Layer

A Redis service layer abstracts Redis operations away from business logic.

## Purpose

* Centralize Redis access
* Improve testability
* Reduce duplication
* Enforce consistency

---

## Example

```csharp
public interface IRedisService
{
    Task SetAsync(string key, string value);
    Task<string?> GetAsync(string key);
}
```

```csharp
public class RedisService : IRedisService
{
    private readonly IDatabase _db;

    public RedisService(IConnectionMultiplexer redis)
    {
        _db = redis.GetDatabase();
    }

    public async Task SetAsync(string key, string value)
    {
        await _db.StringSetAsync(key, value);
    }

    public async Task<string?> GetAsync(string key)
    {
        return await _db.StringGetAsync(key);
    }
}
```

---

# 2. Repository Pattern

The repository pattern abstracts data access logic.

When using Redis, it can represent:

* Cache repository
* Key-value repository
* Domain-specific cache storage

---

## Example Interface

```csharp
public interface IUserCacheRepository
{
    Task SetUserAsync(User user);
    Task<User?> GetUserAsync(int id);
}
```

---

## Implementation

```csharp
public class UserCacheRepository : IUserCacheRepository
{
    private readonly IDatabase _db;

    public UserCacheRepository(IConnectionMultiplexer redis)
    {
        _db = redis.GetDatabase();
    }

    public async Task SetUserAsync(User user)
    {
        var json = JsonSerializer.Serialize(user);
        await _db.StringSetAsync($"user:{user.Id}", json);
    }

    public async Task<User?> GetUserAsync(int id)
    {
        var data = await _db.StringGetAsync($"user:{id}");
        return data.IsNull ? null : JsonSerializer.Deserialize<User>(data);
    }
}
```

---

# 3. Generic Cache Services

A generic cache service reduces duplication across repositories.

---

## Interface

```csharp
public interface ICacheService
{
    Task SetAsync<T>(string key, T value);
    Task<T?> GetAsync<T>(string key);
}
```

---

## Implementation

```csharp
public class CacheService : ICacheService
{
    private readonly IDatabase _db;

    public CacheService(IConnectionMultiplexer redis)
    {
        _db = redis.GetDatabase();
    }

    public async Task SetAsync<T>(string key, T value)
    {
        var json = JsonSerializer.Serialize(value);
        await _db.StringSetAsync(key, json);
    }

    public async Task<T?> GetAsync<T>(string key)
    {
        var data = await _db.StringGetAsync(key);
        return data.IsNull ? default : JsonSerializer.Deserialize<T>(data);
    }
}
```

---

# 4. Cache Abstractions

Cache abstractions decouple application logic from Redis implementation.

---

## Benefits

* Easier unit testing
* Swappable cache provider
* Cleaner architecture
* Better maintainability

---

## Example Abstraction

```csharp
public interface ICacheProvider
{
    Task SetAsync(string key, string value);
    Task<string?> GetAsync(string key);
    Task RemoveAsync(string key);
}
```

---

## Redis Implementation

```csharp
public class RedisCacheProvider : ICacheProvider
{
    private readonly IDatabase _db;

    public RedisCacheProvider(IConnectionMultiplexer redis)
    {
        _db = redis.GetDatabase();
    }

    public Task SetAsync(string key, string value)
        => _db.StringSetAsync(key, value);

    public async Task<string?> GetAsync(string key)
        => await _db.StringGetAsync(key);

    public Task RemoveAsync(string key)
        => _db.KeyDeleteAsync(key);
}
```

---

# 5. Clean Architecture with Redis

Redis is typically placed in the Infrastructure layer.

---

## Layer Structure

| Layer          | Responsibility       |
| -------------- | -------------------- |
| Domain         | Business rules       |
| Application    | Use cases            |
| Infrastructure | Redis implementation |
| API            | Controllers          |

---

## Rule

👉 Domain should NOT depend on Redis

Only Infrastructure should contain Redis logic.

---

## Example Flow

```text
Controller → MediatR → Application → Cache Service → Redis
```

---

# 6. CQRS + Redis

CQRS separates read and write operations.

Redis is commonly used in the READ model.

---

## Write Side

Writes data to database.

```csharp
public async Task Handle(CreateUserCommand request)
{
    // Save to DB
    // Then invalidate cache
}
```

---

## Read Side (Redis Cache)

```csharp
public async Task<User?> Handle(GetUserQuery request)
{
    var cacheKey = $"user:{request.Id}";

    var cached = await _cache.GetAsync<User>(cacheKey);
    if (cached != null)
        return cached;

    var user = await _db.Users.FindAsync(request.Id);

    if (user != null)
        await _cache.SetAsync(cacheKey, user);

    return user;
}
```

---

## Benefits

* Faster reads
* Reduced database load
* Better scalability

---

# 7. MediatR + Redis

MediatR is used to implement clean CQRS pipelines.

Redis is integrated inside handlers or pipeline behaviors.

---

## Example Handler

```csharp
public class GetUserHandler : IRequestHandler<GetUserQuery, User>
{
    private readonly ICacheService _cache;
    private readonly AppDbContext _db;

    public GetUserHandler(ICacheService cache, AppDbContext db)
    {
        _cache = cache;
        _db = db;
    }

    public async Task<User> Handle(GetUserQuery request, CancellationToken cancellationToken)
    {
        var key = $"user:{request.Id}";

        var cached = await _cache.GetAsync<User>(key);
        if (cached != null)
            return cached;

        var user = await _db.Users.FindAsync(request.Id);

        if (user != null)
            await _cache.SetAsync(key, user);

        return user;
    }
}
```

---

## Advanced Pattern: Cache Behavior

You can create a MediatR pipeline behavior:

* Auto caching
* Auto invalidation
* Cross-cutting caching logic

---

# Summary

Using Redis with repository patterns in .NET provides:

* Clean architecture separation
* Reusable cache services
* Better testability
* CQRS optimization
* MediatR pipeline integration

This approach helps build scalable, maintainable, and high-performance distributed systems.
