# 🚀 Advanced Topics

## Overview
Master PostgreSQL-specific EF Core features, migrations strategy, performance tuning, and real-time patterns.

---

## 📌 Concept Types in This Section

| Type | Description |
|------|-------------|
| **Performance** | Query optimization and monitoring |
| **Architecture** | System design and scalability |
| **Real-time** | Event-driven and pub/sub patterns |
| **Security** | Row-level access control |

---

## 1. PostgreSQL-Specific EF Core Features
> **Type: Pattern**

### Array Columns
```csharp
public class Post
{
    public int Id { get; set; }
    public string[] Tags { get; set; } = [];
}

// Query arrays
var posts = await db.Posts
    .Where(p => p.Tags.Contains("dotnet"))
    .ToListAsync();
```

### ILIKE (Case-Insensitive Search)
```csharp
var users = await db.Users
    .Where(u => EF.Functions.ILike(u.Name, "%alice%"))
    .ToListAsync();
```

### Full-Text Search
```csharp
var articles = await db.Articles
    .Where(a => EF.Functions.ToTsVector("english", a.Body)
        .Matches(EF.Functions.ToTsQuery("english", "postgres & dotnet")))
    .ToListAsync();
```

### Date/Time Grouping
```csharp
var dailySales = await db.Orders
    .Where(o => o.CreatedAt > DateTime.UtcNow.AddDays(-7))
    .GroupBy(o => o.CreatedAt.Date)
    .Select(g => new { Date = g.Key, Total = g.Sum(o => o.Total) })
    .ToListAsync();
```

---

## 2. Database Migrations Strategy
> **Type: Architecture**

### EF Core Migrations (Code-First)
```bash
dotnet ef migrations add AddUserRole --output-dir Migrations
dotnet ef database update
dotnet ef migrations script --output migration.sql  # review before prod
dotnet ef database update PreviousMigrationName     # rollback
```

### Safe Migration Practices
```csharp
public partial class AddUserRole : Migration
{
    protected override void Up(MigrationBuilder m)
    {
        // Safe: add column with default (non-breaking)
        m.AddColumn<string>("role", "users", defaultValue: "user", nullable: false);

        // Safe: add index without locking the table
        m.Sql("CREATE INDEX CONCURRENTLY idx_users_role ON users(role)");
    }

    protected override void Down(MigrationBuilder m)
    {
        m.DropIndex("idx_users_role", "users");
        m.DropColumn("role", "users");
    }
}
```

### Flyway / Liquibase (SQL-First Alternative)
```sql
-- V1__initial_schema.sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT UNIQUE NOT NULL
);

-- V2__add_role.sql
ALTER TABLE users ADD COLUMN role TEXT NOT NULL DEFAULT 'user';
```

---

## 3. Performance Tuning
> **Type: Performance**

### EXPLAIN ANALYZE
```sql
EXPLAIN ANALYZE
SELECT u.name, COUNT(o.id) as order_count
FROM users u
LEFT JOIN orders o ON o.user_id = u.id
WHERE u.created_at > '2024-01-01'
GROUP BY u.id, u.name;
```

**What to look for:**
- `Seq Scan` on large tables → add an index
- `Nested Loop` on large result sets → may need Hash Join hint
- Large gap between estimated vs actual rows → run `ANALYZE`

### Indexing Strategy
```sql
-- Simple B-tree index
CREATE INDEX idx_users_email ON users(email);

-- Composite index (equality columns first, then range)
CREATE INDEX idx_orders_user_date ON orders(user_id, created_at DESC);

-- Partial index (only index rows matching condition)
CREATE INDEX idx_active_users ON users(email) WHERE active = true;

-- Expression index
CREATE INDEX idx_users_lower_email ON users(LOWER(email));

-- GIN for arrays and JSONB
CREATE INDEX idx_posts_tags ON posts USING GIN(tags);
CREATE INDEX idx_products_meta ON products USING GIN(metadata);
```

### pg_stat_statements
```sql
-- Enable in postgresql.conf:
-- shared_preload_libraries = 'pg_stat_statements'

-- Find slowest queries
SELECT query, calls, mean_exec_time, total_exec_time
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 10;
```

### N+1 Query Problem
```csharp
// ❌ N+1: 1 query for users + N queries for orders
var users = await db.Users.ToListAsync();
foreach (var user in users)
    Console.WriteLine(user.Orders.Count); // Lazy load = extra query!

// ✅ Eager loading: 1 query with JOIN
var users = await db.Users
    .Include(u => u.Orders)
    .ToListAsync();

// ✅ Projection: even better, only fetch what you need
var result = await db.Users
    .Select(u => new { u.Name, OrderCount = u.Orders.Count })
    .ToListAsync();
```

### AsNoTracking for Read-Only Queries
```csharp
// Skip change-tracking overhead for reads
var users = await db.Users
    .AsNoTracking()
    .Where(u => u.Active)
    .ToListAsync();
```

---

## 4. Read Replicas & Connection Routing
> **Type: Architecture**

```csharp
// Register separate contexts
builder.Services.AddDbContext<AppDbContext>(opt =>
    opt.UseNpgsql(writeConnStr));

builder.Services.AddDbContext<ReadOnlyDbContext>(opt =>
    opt.UseNpgsql(readConnStr)
       .UseQueryTrackingBehavior(QueryTrackingBehavior.NoTracking));

// Use read replica for queries, write db for mutations
public class OrderService(ReadOnlyDbContext readDb, AppDbContext writeDb)
{
    public Task<List<Order>> GetRecentOrders() =>
        readDb.Orders.OrderByDescending(o => o.CreatedAt).Take(100).ToListAsync();

    public async Task PlaceOrder(Order order)
    {
        writeDb.Orders.Add(order);
        await writeDb.SaveChangesAsync();
    }
}
```

---

## 5. Listen/Notify (Real-Time Events)
> **Type: Real-time + Architecture**

PostgreSQL has built-in pub/sub. Use for real-time notifications without polling.

### Publish from .NET
```csharp
await using var cmd = new NpgsqlCommand(
    "SELECT pg_notify(@channel, @payload)", conn);
cmd.Parameters.AddWithValue("channel", "user_created");
cmd.Parameters.AddWithValue("payload", JsonSerializer.Serialize(new { id = 42 }));
await cmd.ExecuteNonQueryAsync();
```

### Subscribe in .NET
```csharp
await using var conn = new NpgsqlConnection(connStr);
await conn.OpenAsync();

conn.Notification += (sender, args) =>
{
    var data = JsonSerializer.Deserialize<MyEvent>(args.Payload);
    // Handle event...
};

await using var cmd = new NpgsqlCommand("LISTEN user_created", conn);
await cmd.ExecuteNonQueryAsync();

while (true)
    await conn.WaitAsync(CancellationToken.None);
```

### Auto-Notify via Database Trigger
```sql
CREATE OR REPLACE FUNCTION notify_on_insert()
RETURNS TRIGGER AS $$
BEGIN
  PERFORM pg_notify('user_changes', row_to_json(NEW)::text);
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER user_created_trigger
AFTER INSERT ON users
FOR EACH ROW EXECUTE FUNCTION notify_on_insert();
```

---

## 6. Row-Level Security (RLS)
> **Type: Security + Architecture**

Enforce tenant isolation at the database level — not just in application code.

```sql
-- Enable RLS on table
ALTER TABLE documents ENABLE ROW LEVEL SECURITY;

-- Policy: users only see their own documents
CREATE POLICY user_isolation ON documents
  FOR ALL
  USING (owner_id = current_setting('app.current_user_id')::int);
```

```csharp
// Set context in .NET before running queries
await using var cmd = new NpgsqlCommand(
    "SET LOCAL app.current_user_id = @userId", conn, tx);
cmd.Parameters.AddWithValue("userId", currentUserId);
await cmd.ExecuteNonQueryAsync();
```

---

## ✅ Checklist
- [ ] Use `EF.Functions.ILike` for case-insensitive search
- [ ] Review migrations with SQL script before deploying to prod
- [ ] Run `EXPLAIN ANALYZE` on your slowest queries
- [ ] Add GIN indexes for JSONB and array columns
- [ ] Fix N+1 queries with Include or projections
- [ ] Use `AsNoTracking()` on read-only EF queries
- [ ] Implement Listen/Notify for a real-time feature
- [ ] Explore Row-Level Security for multi-tenant apps
