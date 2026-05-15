# 🔌 .NET Data Access Options

## Overview
Three main ways to connect .NET apps to PostgreSQL, each with different trade-offs between control and convenience.

---

## 📌 Concept Types in This Section

| Type | Description |
|------|-------------|
| **Library** | NuGet packages and drivers |
| **Pattern** | Code design and usage patterns |
| **Configuration** | Setup and connection options |

---

## 1. Npgsql — The Core Driver
> **Type: Library + Pattern**

The official open-source .NET driver for PostgreSQL. Every other option builds on top of it.

### Install
```bash
dotnet add package Npgsql
```

### Basic Usage
```csharp
using Npgsql;

var connStr = "Host=localhost;Database=myapp;Username=postgres;Password=secret";

await using var conn = new NpgsqlConnection(connStr);
await conn.OpenAsync();

await using var cmd = new NpgsqlCommand(
    "SELECT id, name FROM users WHERE email = @email", conn);
cmd.Parameters.AddWithValue("email", "user@example.com");

await using var reader = await cmd.ExecuteReaderAsync();
while (await reader.ReadAsync())
{
    Console.WriteLine($"{reader.GetInt32(0)}: {reader.GetString(1)}");
}
```

### Insert & Get ID Back
```csharp
await using var cmd = new NpgsqlCommand(
    "INSERT INTO users (name, email) VALUES (@name, @email) RETURNING id", conn);
cmd.Parameters.AddWithValue("name", "Alice");
cmd.Parameters.AddWithValue("email", "alice@example.com");

var newId = (int)await cmd.ExecuteScalarAsync();
```

### Transactions
```csharp
await using var tx = await conn.BeginTransactionAsync();
try
{
    // Multiple operations...
    await tx.CommitAsync();
}
catch
{
    await tx.RollbackAsync();
    throw;
}
```

---

## 2. Dapper — Lightweight ORM
> **Type: Library + Pattern**

Micro-ORM that maps SQL results directly to C# objects. Best of both worlds: raw SQL control with easy object mapping.

### Install
```bash
dotnet add package Dapper
dotnet add package Npgsql
```

### Basic Queries
```csharp
using Dapper;
using Npgsql;

public record User(int Id, string Name, string Email);

await using var conn = new NpgsqlConnection(connStr);

// Query multiple rows
var users = await conn.QueryAsync<User>(
    "SELECT id, name, email FROM users WHERE active = @active",
    new { active = true });

// Query single row
var user = await conn.QueryFirstOrDefaultAsync<User>(
    "SELECT * FROM users WHERE id = @id",
    new { id = 42 });

// Execute (INSERT/UPDATE/DELETE)
var affected = await conn.ExecuteAsync(
    "UPDATE users SET name = @name WHERE id = @id",
    new { name = "Bob", id = 1 });
```

### Multi-Mapping (JOINs)
```csharp
public record Order(int Id, decimal Total, User User);

var orders = await conn.QueryAsync<Order, User, Order>(
    @"SELECT o.id, o.total, u.id, u.name, u.email
      FROM orders o INNER JOIN users u ON u.id = o.user_id",
    (order, user) => order with { User = user },
    splitOn: "id");
```

---

## 3. Entity Framework Core — Full ORM
> **Type: Library + Pattern + Configuration**

Full-featured ORM with LINQ, migrations, and change tracking.

### Install
```bash
dotnet add package Microsoft.EntityFrameworkCore
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet tool install --global dotnet-ef
```

### Define Entities
```csharp
public class User
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public string Email { get; set; } = "";
    public DateTime CreatedAt { get; set; }
    public List<Order> Orders { get; set; } = [];
}

public class Order
{
    public int Id { get; set; }
    public decimal Total { get; set; }
    public int UserId { get; set; }
    public User User { get; set; } = null!;
}
```

### DbContext
```csharp
public class AppDbContext : DbContext
{
    public DbSet<User> Users => Set<User>();
    public DbSet<Order> Orders => Set<Order>();

    protected override void OnConfiguring(DbContextOptionsBuilder options)
        => options.UseNpgsql("Host=localhost;Database=myapp;Username=postgres;Password=secret");

    protected override void OnModelCreating(ModelBuilder model)
    {
        model.Entity<User>(e => {
            e.HasKey(u => u.Id);
            e.Property(u => u.Email).IsRequired();
            e.HasIndex(u => u.Email).IsUnique();
        });
    }
}
```

### LINQ Queries
```csharp
await using var db = new AppDbContext();

// Basic query
var users = await db.Users
    .Where(u => u.CreatedAt > DateTime.UtcNow.AddDays(-30))
    .OrderBy(u => u.Name)
    .ToListAsync();

// Include related data
var usersWithOrders = await db.Users
    .Include(u => u.Orders)
    .Where(u => u.Orders.Any(o => o.Total > 100))
    .ToListAsync();

// Projection
var summary = await db.Users
    .Select(u => new { u.Name, OrderCount = u.Orders.Count })
    .ToListAsync();
```

### Migrations
```bash
dotnet ef migrations add InitialCreate
dotnet ef database update
dotnet ef migrations add AddUserRole
dotnet ef database update
```

---

## 🆚 Comparison Table

| Feature | Npgsql | Dapper | EF Core |
|---------|--------|--------|---------|
| Control | Full | High | Medium |
| Boilerplate | High | Low | Very Low |
| Performance | Fastest | Fast | Moderate |
| Learning Curve | Medium | Low | High |
| Migrations | Manual | Manual | Built-in |
| LINQ | No | No | Yes |
| Best For | Low-level ops | SQL lovers | CRUD apps |

---

## ✅ Checklist
- [ ] Install Npgsql and connect to PostgreSQL
- [ ] Execute parameterized queries safely
- [ ] Try Dapper for object mapping
- [ ] Set up EF Core with DbContext
- [ ] Create and apply first migration
- [ ] Query with LINQ using EF Core
