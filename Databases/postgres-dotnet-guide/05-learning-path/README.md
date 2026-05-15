# 🗺️ Learning Path & Resources

## Overview
A structured, step-by-step plan to go from zero to production-ready with PostgreSQL and .NET.

---

## 📌 Concept Types in This Section

| Type | Description |
|------|-------------|
| **Milestone** | Key learning checkpoints |
| **Project** | Hands-on practice projects |
| **Resource** | Books, docs, and tools |

---

## Phase 1 — Foundations (Week 1–2)
> **Type: Milestone**

**Goals:** Get PostgreSQL running locally. Write confident SQL.

### Steps
1. Run PostgreSQL with Docker: `docker run -d -p 5432:5432 -e POSTGRES_PASSWORD=secret postgres:16`
2. Connect with pgAdmin or DBeaver
3. Practice: CREATE TABLE, INSERT, SELECT, UPDATE, DELETE
4. Learn JOINs: INNER, LEFT, RIGHT, FULL OUTER
5. Learn GROUP BY, HAVING, ORDER BY, LIMIT
6. Create indexes and observe performance changes

### Mini Project: Blog Database
> **Type: Project**

```sql
CREATE TABLE authors (id SERIAL PRIMARY KEY, name TEXT, email TEXT UNIQUE);
CREATE TABLE posts (
  id SERIAL PRIMARY KEY, title TEXT, body TEXT,
  author_id INT REFERENCES authors(id),
  created_at TIMESTAMPTZ DEFAULT NOW()
);
CREATE TABLE comments (
  id SERIAL PRIMARY KEY, content TEXT,
  post_id INT REFERENCES posts(id),
  author_id INT REFERENCES authors(id)
);

-- Practice query: posts with comment counts
SELECT p.title, a.name, COUNT(c.id) as comment_count
FROM posts p
JOIN authors a ON a.id = p.author_id
LEFT JOIN comments c ON c.post_id = p.id
GROUP BY p.id, p.title, a.name
ORDER BY comment_count DESC;
```

---

## Phase 2 — Npgsql & Dapper (Week 3–4)
> **Type: Milestone**

**Goals:** Connect .NET to PostgreSQL. Query safely. Map results to C# objects.

### Steps
1. Create a .NET console app: `dotnet new console -n TaskManager`
2. Add `Npgsql` NuGet package
3. Execute parameterized queries
4. Add `Dapper` for object mapping
5. Implement a simple Repository pattern

### Project: CLI Task Manager
> **Type: Project**

```csharp
public record TaskItem(int Id, string Title, bool Done, DateTime CreatedAt);

public class TaskRepository(NpgsqlDataSource db)
{
    public async Task<IEnumerable<TaskItem>> GetAll()
    {
        await using var conn = await db.OpenConnectionAsync();
        return await conn.QueryAsync<TaskItem>("SELECT * FROM tasks ORDER BY created_at");
    }

    public async Task Add(string title)
    {
        await using var conn = await db.OpenConnectionAsync();
        await conn.ExecuteAsync(
            "INSERT INTO tasks (title) VALUES (@title)", new { title });
    }

    public async Task Complete(int id)
    {
        await using var conn = await db.OpenConnectionAsync();
        await conn.ExecuteAsync(
            "UPDATE tasks SET done = true WHERE id = @id", new { id });
    }
}
```

---

## Phase 3 — Entity Framework Core (Week 5–6)
> **Type: Milestone**

**Goals:** Define entities and relationships. Use migrations. Query with LINQ.

### Steps
1. Create ASP.NET Core Web API: `dotnet new webapi -n MyApi`
2. Add EF Core + Npgsql packages
3. Define 3+ related entities
4. Create initial migration: `dotnet ef migrations add InitialCreate`
5. Apply migration: `dotnet ef database update`
6. Build CRUD endpoints

### Project: E-Commerce REST API
> **Type: Project**

```
GET    /api/products              list all products
POST   /api/products              create a product
GET    /api/orders                list orders for current user
POST   /api/orders                place an order (transaction!)
GET    /api/orders/{id}           order details with line items
DELETE /api/orders/{id}           cancel an order
```

---

## Phase 4 — Intermediate Skills (Week 7–8)
> **Type: Milestone**

**Goals:** Production-ready patterns. Handle larger datasets efficiently.

### Steps
1. Configure `AddNpgsqlDataSource` with pool settings in DI
2. Add `CancellationToken` to all async methods
3. Wrap multi-step operations in transactions
4. Test bulk insert with 100k+ rows (COPY vs AddRange — measure both)
5. Map a JSONB column to a C# class in EF Core
6. Add `AsNoTracking()` to all read-only EF queries

---

## Phase 5 — Advanced (Week 9–12)
> **Type: Milestone**

**Goals:** Performance profiling. PostgreSQL-specific features. Real-time capabilities.

### Steps
1. Enable `pg_stat_statements` and find your top 5 slowest queries
2. Run `EXPLAIN ANALYZE` on each and fix with indexes
3. Add GIN indexes for JSONB and array columns
4. Build a real-time feature using Listen/Notify + SignalR
5. Set up a read replica and route read queries to it

### Project: Real-Time Notification Dashboard
> **Type: Project**

- **Trigger:** INSERT into database → `pg_notify` fires
- **Backend:** ASP.NET Core listens on PostgreSQL channel → pushes via SignalR
- **Frontend:** React client receives live updates without polling
- **Goal:** Zero-polling, pure event-driven architecture

---

## 📚 Resources
> **Type: Resource**

### Official Documentation

| Resource | URL |
|----------|-----|
| PostgreSQL Docs | postgresql.org/docs |
| Npgsql Driver Docs | npgsql.org/doc |
| EF Core + Npgsql | npgsql.org/efcore |
| EF Core Official Docs | learn.microsoft.com/ef/core |

### Books

| Book | Focus |
|------|-------|
| *PostgreSQL: Up and Running* | PostgreSQL fundamentals |
| *Learning PostgreSQL 16* | Modern PostgreSQL features |
| *Entity Framework Core in Action* | EF Core deep dive |

### Tools

| Tool | Purpose |
|------|---------|
| pgAdmin 4 | PostgreSQL GUI admin |
| DBeaver | Free multi-DB GUI |
| DataGrip | JetBrains premium IDE |
| Postman / Bruno | API testing |
| Docker | Run PostgreSQL locally |
| explain.dalibo.com | Visual EXPLAIN ANALYZE |

### Online Practice

- **pgexercises.com** — Interactive SQL exercises
- **use-the-index-luke.com** — Indexing deep dive
- **sqlzoo.net** — SQL fundamentals practice

---

## 🎯 Skill Checkpoints

### Beginner ✅
- [ ] Write SQL CRUD queries confidently
- [ ] Connect .NET to PostgreSQL with Npgsql
- [ ] Use parameterized queries (no SQL injection)

### Intermediate ✅
- [ ] Build a REST API with EF Core + migrations
- [ ] Handle transactions for multi-step operations
- [ ] Configure connection pooling in ASP.NET Core

### Advanced ✅
- [ ] Optimize queries with EXPLAIN ANALYZE
- [ ] Use JSONB, arrays, full-text search in EF Core
- [ ] Implement real-time features with Listen/Notify
- [ ] Design for scalability with read replicas

---

## 💡 Pro Tips

1. **Always profile before optimizing** — don't guess, measure with `EXPLAIN ANALYZE`
2. **Use `AsNoTracking()`** for read-only EF Core queries (significant perf win)
3. **Never use `SELECT *` in production** — always list columns explicitly
4. **Set `statement_timeout`** in PostgreSQL to prevent runaway queries
5. **Use UTC everywhere** — `TIMESTAMPTZ` in Postgres, `DateTime.UtcNow` in .NET
6. **Test migrations on a copy of prod data** before deploying
7. **Monitor `pg_stat_activity`** for long-running queries in production
8. **Use connection string parameters, not hardcoded values** — environment variables or secrets management
