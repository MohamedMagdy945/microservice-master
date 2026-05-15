# 🗄️ Core Foundations

## Overview
Before diving into .NET integration, you need a solid understanding of PostgreSQL itself and fundamental SQL concepts.

---

## 📌 Concept Types in This Section

| Type | Description |
|------|-------------|
| **Conceptual** | Theory, design patterns, mental models |
| **Practical** | Hands-on commands and code |
| **Configuration** | Setup and environment |

---

## 1. SQL Fundamentals
> **Type: Practical + Conceptual**

Core SQL operations every developer must know:

```sql
-- CREATE
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  email TEXT UNIQUE,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- READ
SELECT id, name, email FROM users WHERE created_at > '2024-01-01';

-- UPDATE
UPDATE users SET name = 'John Doe' WHERE id = 1;

-- DELETE
DELETE FROM users WHERE id = 1;

-- JOIN
SELECT u.name, o.total
FROM users u
INNER JOIN orders o ON o.user_id = u.id;
```

### Key Concepts:
- **Indexes** — Speed up reads, slow down writes. Use on columns in WHERE/JOIN clauses.
- **Transactions** — Group operations into atomic units (all succeed or all fail).
- **Normalization** — Organize data to reduce redundancy (1NF → 3NF).

---

## 2. PostgreSQL-Specific Features
> **Type: Conceptual + Practical**

### JSONB (Binary JSON)
```sql
CREATE TABLE products (
  id SERIAL PRIMARY KEY,
  metadata JSONB
);

INSERT INTO products (metadata) VALUES ('{"color": "red", "size": 42}');

-- Query inside JSON
SELECT * FROM products WHERE metadata->>'color' = 'red';
SELECT * FROM products WHERE (metadata->>'size')::int > 40;

-- Index on JSONB field
CREATE INDEX idx_products_color ON products ((metadata->>'color'));
```

### Arrays
```sql
CREATE TABLE posts (
  id SERIAL PRIMARY KEY,
  tags TEXT[]
);

INSERT INTO posts (tags) VALUES (ARRAY['dotnet', 'postgres', 'backend']);
SELECT * FROM posts WHERE 'dotnet' = ANY(tags);
```

### Common Table Expressions (CTEs)
```sql
WITH active_users AS (
  SELECT * FROM users WHERE last_login > NOW() - INTERVAL '30 days'
),
user_orders AS (
  SELECT user_id, COUNT(*) as order_count FROM orders GROUP BY user_id
)
SELECT u.name, uo.order_count
FROM active_users u
LEFT JOIN user_orders uo ON uo.user_id = u.id;
```

### Window Functions
```sql
SELECT
  name,
  salary,
  RANK() OVER (PARTITION BY department ORDER BY salary DESC) as rank,
  AVG(salary) OVER (PARTITION BY department) as dept_avg
FROM employees;
```

### Full-Text Search
```sql
ALTER TABLE articles ADD COLUMN search_vector TSVECTOR;
UPDATE articles SET search_vector = to_tsvector('english', title || ' ' || body);

SELECT title FROM articles WHERE search_vector @@ to_tsquery('postgres & dotnet');
```

---

## 3. Schema Design & Normalization
> **Type: Conceptual**

| Normal Form | Rule |
|-------------|------|
| **1NF** | Atomic values, no repeating groups |
| **2NF** | Every non-key column depends on the whole primary key |
| **3NF** | No transitive dependencies |

### Practical Design Tips
- Use `SERIAL` or `UUID` for primary keys
- Always add `created_at` and `updated_at` timestamps
- Use `NOT NULL` constraints liberally
- Add `CHECK` constraints for business rules

---

## 4. Setting Up PostgreSQL
> **Type: Configuration**

### Docker (Recommended for Dev)
```bash
docker run --name postgres-dev \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_DB=myapp \
  -p 5432:5432 \
  -d postgres:16
```

### Connection String (.NET)
```
Host=localhost;Port=5432;Database=myapp;Username=postgres;Password=secret;
```

### GUI Tools
| Tool | Best For |
|------|----------|
| **pgAdmin** | Full-featured admin UI |
| **DBeaver** | Multi-database, free |
| **DataGrip** | JetBrains, paid |
| **psql** | CLI, always available |

---

## ✅ Checklist
- [ ] Understand CRUD operations in SQL
- [ ] Know when and how to use indexes
- [ ] Practice transactions with COMMIT/ROLLBACK
- [ ] Explore JSONB for flexible schemas
- [ ] Set up a local PostgreSQL instance
- [ ] Connect with a GUI tool
