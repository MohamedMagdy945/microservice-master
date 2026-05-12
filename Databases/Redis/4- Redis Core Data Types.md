# Redis Core Data Types

Redis provides multiple specialized data structures optimized for speed, flexibility, and scalability.
Understanding these core data types is essential for building high-performance applications with Redis.

---

# Table of Contents

1. [Strings](#1-strings)
2. [Hashes](#2-hashes)
3. [Lists](#3-lists)
4. [Sets](#4-sets)
5. [Sorted Sets (ZSET)](#5-sorted-sets-zset)
6. [Bitmaps](#6-bitmaps)
7. [HyperLogLog](#7-hyperloglog)
8. [Streams](#8-streams)
9. [Geospatial Indexes](#9-geospatial-indexes)
10. [JSON Support](#10-json-support)
11. [Vector Data](#11-vector-data)

---

# 1. Strings

Strings are the most basic Redis data type.

They can store:

* Text
* Numbers
* Binary data
* Serialized objects

## Features

* Maximum size: 512 MB
* Atomic operations
* Extremely fast read/write performance
* Simple key-value storage model

## Common Commands

```bash
SET user:name "Mohamed"
GET user:name
INCR page:views
DECR stock:count
APPEND user:name " Magdy"
```

## Use Cases

* Caching
* Session storage
* Counters
* Rate limiting
* Feature flags

---

# 2. Hashes

Hashes store field-value pairs inside a single Redis key.

They are ideal for representing objects.

## Features

* Memory efficient
* Fast field access
* Supports partial updates
* Great for structured data

## Common Commands

```bash
HSET user:1 name "Mohamed" age 25 country "Egypt"
HGET user:1 name
HGETALL user:1
HDEL user:1 age
```

## Example Structure

```text
user:1
 ├── name: Mohamed
 ├── age: 25
 └── country: Egypt
```

## Use Cases

* User profiles
* Product information
* Metadata storage
* Configuration management

---

# 3. Lists

Lists are ordered collections of strings.

Redis Lists are implemented as linked lists.

## Features

* Maintains insertion order
* Fast insertion/removal at head and tail
* Supports queue and stack operations

## Common Commands

```bash
LPUSH tasks "task1"
RPUSH tasks "task2"
LPOP tasks
RPOP tasks
LRANGE tasks 0 -1
```

## Use Cases

* Queues
* Messaging systems
* Activity feeds
* Task processing
* Job scheduling

---

# 4. Sets

Sets are unordered collections of unique values.

## Features

* No duplicate values
* Fast membership testing
* Supports mathematical set operations

## Common Commands

```bash
SADD tags "redis" "database" "cache"
SMEMBERS tags
SISMEMBER tags "redis"
SREM tags "cache"
```

## Set Operations

```bash
SUNION set1 set2
SINTER set1 set2
SDIFF set1 set2
```

## Use Cases

* Tags
* Unique visitors
* Friend relationships
* Recommendation systems
* Access control

---

# 5. Sorted Sets (ZSET)

Sorted Sets are similar to Sets but every member has a score.

Elements are automatically sorted based on their score.

## Features

* Unique values
* Ordered by score
* Efficient ranking operations
* Range queries support

## Common Commands

```bash
ZADD leaderboard 100 "Mohamed"
ZADD leaderboard 250 "Ali"
ZRANGE leaderboard 0 -1 WITHSCORES
ZREVRANK leaderboard "Ali"
```

## Use Cases

* Leaderboards
* Ranking systems
* Priority queues
* Time-series data
* Analytics

---

# 6. Bitmaps

Bitmaps allow bit-level operations on strings.

They are extremely memory efficient.

## Features

* Stores binary states efficiently
* Supports bitwise operations
* Great for boolean tracking

## Common Commands

```bash
SETBIT online_users 1001 1
GETBIT online_users 1001
BITCOUNT online_users
```

## Use Cases

* User online tracking
* Daily activity tracking
* Feature toggles
* Analytics systems

---

# 7. HyperLogLog

HyperLogLog is a probabilistic data structure used for counting unique values.

It provides approximate cardinality estimation using very little memory.

## Features

* Extremely memory efficient
* Approximate counting
* Suitable for massive datasets

## Common Commands

```bash
PFADD visitors user1 user2 user3
PFCOUNT visitors
PFMERGE combined_visitors visitors1 visitors2
```

## Use Cases

* Unique visitors counting
* Analytics
* Large-scale statistics
* Traffic estimation

---

# 8. Streams

Streams are append-only log data structures.

They are designed for event-driven and messaging systems.

## Features

* Persistent event storage
* Consumer groups support
* Ordered event processing
* Message replay capability

## Common Commands

```bash
XADD orders * product "Laptop" price 1200
XRANGE orders - +
XREAD COUNT 2 STREAMS orders 0
```

## Use Cases

* Event sourcing
* Message queues
* Real-time analytics
* Audit logs
* Microservices communication

---

# 9. Geospatial Indexes

Redis supports geospatial data using sorted sets internally.

## Features

* Store coordinates
* Radius searches
* Distance calculations
* Location-based queries

## Common Commands

```bash
GEOADD cities 29.9792 31.1342 "Giza"
GEODIST cities "Giza" "Cairo" KM
GEORADIUS cities 31 30 100 KM
```

## Use Cases

* Ride-sharing apps
* Delivery systems
* Store locators
* Nearby places search

---

# 10. JSON Support

Redis supports JSON documents through the RedisJSON module.

This allows storing and querying structured JSON data directly.

## Features

* Store full JSON documents
* Partial updates
* JSON path querying
* Nested object support

## Common Commands

```bash
JSON.SET user:1 $ '{"name":"Mohamed","age":25}'
JSON.GET user:1
JSON.SET user:1 $.age 26
```

## Use Cases

* Modern web applications
* API caching
* Document databases
* Dynamic schemas

---

# 11. Vector Data

Redis supports vector similarity search using vector embeddings.

This is commonly used in AI and machine learning applications.

## Features

* Vector indexing
* Similarity search
* AI semantic search
* High-performance vector retrieval

## Common Concepts

* Embeddings
* Cosine similarity
* Vector indexing
* Nearest neighbor search

## Example Use Cases

* AI chatbots
* Recommendation engines
* Semantic search
* Image similarity search
* Retrieval-Augmented Generation (RAG)

---

# Choosing the Right Data Type

| Data Type   | Best For                    |
| ----------- | --------------------------- |
| Strings     | Simple key-value storage    |
| Hashes      | Objects and structured data |
| Lists       | Queues and ordered data     |
| Sets        | Unique collections          |
| Sorted Sets | Rankings and scoring        |
| Bitmaps     | Boolean tracking            |
| HyperLogLog | Approximate unique counting |
| Streams     | Event-driven systems        |
| Geospatial  | Location-based features     |
| JSON        | Document storage            |
| Vector Data | AI and semantic search      |

---

# Summary

Redis offers a rich set of powerful data structures beyond simple key-value storage.

Choosing the correct data type can significantly improve:

* Performance
* Scalability
* Memory efficiency
* Application design

Mastering Redis data structures is essential for building modern high-performance distributed systems.
