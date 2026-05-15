# ⚙️ Intermediate Topics

## Overview
Level up your PostgreSQL + .NET skills with connection pooling, async patterns, bulk operations, and JSONB mapping.

---

## 📌 Concept Types in This Section

| Type | Description |
|------|-------------|
| **Performance** | Speed and efficiency optimizations |
| **Pattern** | Code design and best practices |
| **Security** | Protecting your application |

---

## 1. Connection Pooling
> **Type: Performance + Configuration**

Npgsql has built-in connection pooling. Connections are expensive to create — pooling reuses them.

### How It Works
- On `OpenAsync()`, Npgsql checks the pool for an available connection
- On `Dispose()`, the connection returns to the pool (not actually closed)
- Pool is scoped per connection string

### Configuration
```csharp
var connStr = new NpgsqlConnectionStringBuilder
{
    Host = "localhost",
    Database = "myapp",
    Username = "postgres",
    Password = "secret",
    MinPoolSize = 2,
    MaxPoolSize = 100,
    ConnectionIdleLifetime = 300,
    ConnectionPruningInterval = 10
}.ToString();
```

### In ASP.NET Core (DI)
```csharp
// Program.cs
builder.Services.AddNpgsqlDataSource(connStr);

// Inject NpgsqlDataSource
public class UserRepo(NpgsqlDataSource db)
{
    public async Task<User?> GetById(int id)
    {
        await using var conn = await db.OpenConnectionAsync();
        // ...
    }
}
```

### With EF Core
```csharp
builder.Services.AddDbContextPool<AppDbContext>(opt =>
    opt.UseNpgsql(connStr), poolSize: 128);
```

---

## 2. Async/Await Patterns
> **Type: Pattern**

Always use async methods in web apps to avoid blocking threads.

### Core Async Methods
```csharp
// Connection
await conn.OpenAsync();

// Command execution
await cmd.ExecuteNonQueryAsync();   // INSERT/UPDATE/DELETE
await cmd.ExecuteScalarAsync();     // Single value
await cmd.ExecuteReaderAsync();     // Result set

// Reading rows
await using var reader = await cmd.ExecuteReaderAsync();
while (await reader.ReadAsync())
{
    var id = reader.GetInt32(0);
    var name = reader.GetString(1);
}

// EF Core
await db.Users.ToListAsync();
await db.SaveChangesAsync();
```

### Cancellation Tokens
```csharp
public async Task<List<User>> GetUsers(CancellationToken ct)
{
    await using var conn = new NpgsqlConnection(connStr);
    await conn.OpenAsync(ct);

    return await db.Users
        .Where(u => u.Active)
        .ToListAsync(ct);
}
```

---

## 3. Transactions
> **Type: Pattern**

### With Npgsql
```csharp
await using var conn = new NpgsqlConnection(connStr);
await conn.OpenAsync();
await using var tx = await conn.BeginTransactionAsync(IsolationLevel.ReadCommitted);

try
{
    // Debit account
    await using var cmd1 = new NpgsqlCommand(
        "UPDATE accounts SET balance = balance - @amount WHERE id = @from", conn, tx);
    cmd1.Parameters.AddWithValue("amount", 100m);
    cmd1.Parameters.AddWithValue("from", 1);
    await cmd1.ExecuteNonQueryAsync();

    // Credit account
    await using var cmd2 = new NpgsqlCommand(
        "UPDATE accounts SET balance = balance + @amount WHERE id = @to", conn, tx);
    cmd2.Parameters.AddWithValue("amount", 100m);
    cmd2.Parameters.AddWithValue("to", 2);
    await cmd2.ExecuteNonQueryAsync();

    await tx.CommitAsync();
}
catch
{
    await tx.RollbackAsync();
    throw;
}
```

### With EF Core
```csharp
await using var tx = await db.Database.BeginTransactionAsync();
try
{
    db.Users.Add(new User { Name = "Alice" });
    await db.SaveChangesAsync();

    db.Orders.Add(new Order { UserId = 1, Total = 99.99m });
    await db.SaveChangesAsync();

    await tx.CommitAsync();
}
catch
{
    await tx.RollbackAsync();
    throw;
}
```

### Isolation Levels

| Level | Dirty Read | Non-Repeatable Read | Phantom Read |
|-------|-----------|---------------------|--------------|
| ReadUncommitted | ✅ | ✅ | ✅ |
| ReadCommitted *(default)* | ❌ | ✅ | ✅ |
| RepeatableRead | ❌ | ❌ | ✅ |
| Serializable | ❌ | ❌ | ❌ |

---

## 4. Parameterized Queries & SQL Injection Prevention
> **Type: Security**

**ALWAYS use parameters. Never interpolate user input into SQL.**

```csharp
// ❌ DANGEROUS
var sql = $"SELECT * FROM users WHERE name = '{userInput}'";

// ✅ SAFE - Npgsql
var cmd = new NpgsqlCommand("SELECT * FROM users WHERE name = @name", conn);
cmd.Parameters.AddWithValue("name", userInput);

// ✅ SAFE - Dapper
var user = await conn.QueryFirstAsync<User>(
    "SELECT * FROM users WHERE name = @name",
    new { name = userInput });

// ✅ SAFE - EF Core (LINQ is always parameterized)
var user = await db.Users.FirstAsync(u => u.Name == userInput);
```

---

## 5. Bulk Inserts
> **Type: Performance**

### Using COPY (Fastest — millions of rows)
```csharp
await using var writer = await conn.BeginBinaryImportAsync(
    "COPY users (name, email) FROM STDIN (FORMAT BINARY)");

foreach (var user in users)
{
    await writer.StartRowAsync();
    await writer.WriteAsync(user.Name, NpgsqlDbType.Text);
    await writer.WriteAsync(user.Email, NpgsqlDbType.Text);
}

await writer.CompleteAsync();
```

### Using unnest (Moderate — thousands of rows)
```csharp
var names = users.Select(u => u.Name).ToArray();
var emails = users.Select(u => u.Email).ToArray();

await conn.ExecuteAsync(
    @"INSERT INTO users (name, email)
      SELECT * FROM unnest(@names::text[], @emails::text[])",
    new { names, emails });
```

### EF Core AddRange (Convenient — small batches)
```csharp
db.Users.AddRange(users);
await db.SaveChangesAsync();
```

### Performance Comparison

| Method | Rows/sec | Use When |
|--------|----------|----------|
| COPY | 500k+ | Huge imports |
| unnest | 50k+ | Medium batches |
| AddRange | 5k | Small batches |

---

## 6. JSONB Mapping in EF Core
> **Type: Pattern**

```csharp
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public ProductMetadata Metadata { get; set; } = new();
}

public class ProductMetadata
{
    public string Color { get; set; } = "";
    public int Size { get; set; }
    public List<string> Tags { get; set; } = [];
}

// In DbContext
modelBuilder.Entity<Product>()
    .Property(p => p.Metadata)
    .HasColumnType("jsonb");

// Query JSONB in LINQ
var redProducts = await db.Products
    .Where(p => p.Metadata.Color == "red")
    .ToListAsync();
```

---

## ✅ Checklist
- [ ] Configure connection pool min/max sizes
- [ ] Use `AddDbContextPool` in ASP.NET Core
- [ ] Use `CancellationToken` in all async methods
- [ ] Always use parameterized queries
- [ ] Implement transactions for multi-step operations
- [ ] Try bulk insert with COPY for large datasets
- [ ] Map a JSONB column to a C# class in EF Core
