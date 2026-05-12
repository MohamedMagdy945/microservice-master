# Redis Fundamentals

## Table of Contents

1. What is Redis
2. In-Memory Databases
3. Key-Value Databases
4. Redis Architecture
5. Single Thread Model
6. Event Loop
7. Redis Performance Model
8. Redis vs Traditional Databases
9. Redis Use Cases
10. Redis Installation
11. Redis CLI
12. Redis Configuration
13. Redis Persistence Overview
14. Redis Security Basics
15. Redis Networking
16. Redis Memory Management

---

# 1. What is Redis

## Definition

Redis stands for **Remote Dictionary Server**. It is an open-source, in-memory data structure store used as:

* Database
* Cache
* Message Broker
* Streaming Engine

Redis stores data primarily in memory, which makes it extremely fast compared to traditional disk-based databases.

## Key Features

* Extremely high performance
* In-memory storage
* Supports multiple data structures
* Persistence support
* Replication and clustering
* Pub/Sub messaging
* High availability

## Supported Data Types

Redis supports many data structures:

* Strings
* Lists
* Sets
* Sorted Sets
* Hashes
* Streams
* Bitmaps
* HyperLogLogs
* Geospatial indexes

## Why Redis is Popular

Redis is widely used because it provides:

* Very low latency
* High throughput
* Simplicity
* Scalability
* Easy integration

---

# 2. In-Memory Databases

## What is an In-Memory Database?

An in-memory database stores data directly inside RAM instead of relying mainly on disk storage.

Because RAM access is extremely fast, operations are completed in microseconds or milliseconds.

## Advantages

### Speed

RAM is much faster than hard drives or SSDs.

### Low Latency

Applications receive data almost instantly.

### High Throughput

Redis can handle hundreds of thousands of requests per second.

## Disadvantages

### Memory Cost

RAM is more expensive than disk storage.

### Volatility

Memory data may be lost if persistence is not configured.

### Capacity Limits

RAM size limits total data storage.

## Examples of In-Memory Databases

* Redis
* Memcached
* Hazelcast
* Apache Ignite

---

# 3. Key-Value Databases

## What is a Key-Value Database?

A key-value database stores data as:

```text
Key -> Value
```

Example:

```text
user:1 -> Mohamed
```

The key is unique and used to retrieve the value quickly.

## Redis as a Key-Value Store

Redis is one of the fastest key-value databases because all data is stored in memory.

## Benefits

* Simple structure
* Fast lookups
* Easy scaling
* Flexible values

## Example Commands

### Set a Value

```bash
SET username mohamed
```

### Get a Value

```bash
GET username
```

### Delete a Value

```bash
DEL username
```

---

# 4. Redis Architecture

## High-Level Architecture

Redis follows a client-server architecture.

```text
Client Applications
        |
        |
    Redis Server
        |
Persistence / Replication
```

## Main Components

### Redis Server

Handles requests and stores data.

### Client

Applications communicate with Redis using TCP.

### Memory Storage

Primary storage location.

### Persistence Layer

Used to save data to disk.

## Architecture Characteristics

* Single process
* Event-driven
* Non-blocking I/O
* Efficient memory usage

---

# 5. Single Thread Model

## What Does Single Thread Mean?

Redis processes commands using a single main thread.

This means:

* One command executes at a time
* No locking complexity
* No race conditions in command execution

## Why is Redis Still Fast?

Even with a single thread, Redis is fast because:

* Operations happen in memory
* Efficient data structures
* Minimal context switching
* Optimized event loop

## Advantages

### Simplicity

No thread synchronization issues.

### Predictable Performance

No thread contention.

### Lower Overhead

Less CPU overhead.

## Limitations

### CPU Bound Operations

Heavy operations may block other requests.

### Long Commands

Large operations can affect latency.

---

# 6. Event Loop

## What is an Event Loop?

The event loop is the mechanism Redis uses to handle many client connections efficiently.

Redis uses:

* Non-blocking I/O
* Multiplexing
* Event-driven processing

## Workflow

```text
1. Client sends request
2. Redis event loop receives event
3. Command processed
4. Response returned
```

## I/O Multiplexing

Redis uses system calls such as:

* epoll (Linux)
* kqueue (BSD/macOS)
* select

This allows Redis to manage thousands of connections efficiently.

## Benefits

* High concurrency
* Low latency
* Efficient CPU usage

---

# 7. Redis Performance Model

## Why Redis is Fast

### In-Memory Storage

No disk seek delays.

### Efficient Data Structures

Optimized internal algorithms.

### Single Thread Execution

Avoids locking overhead.

### Non-Blocking Networking

Efficient connection handling.

## Typical Performance

Redis can process:

* Hundreds of thousands of requests per second
* Sub-millisecond latency

## Performance Factors

### Memory Speed

RAM performance impacts Redis speed.

### Network Speed

Slow networks increase latency.

### Command Complexity

Some commands are O(1), others O(N).

## Big O Examples

| Command | Complexity |
| ------- | ---------- |
| GET     | O(1)       |
| SET     | O(1)       |
| LPUSH   | O(1)       |
| KEYS *  | O(N)       |

---

# 8. Redis vs Traditional Databases

## Redis vs Relational Databases

| Feature        | Redis               | Traditional DB |
| -------------- | ------------------- | -------------- |
| Storage        | In-Memory           | Disk-Based     |
| Speed          | Very Fast           | Slower         |
| Schema         | Flexible            | Fixed Schema   |
| Query Language | Commands            | SQL            |
| Transactions   | Limited             | Advanced       |
| Scaling        | Horizontal Friendly | Often Vertical |

## Redis Strengths

* Real-time applications
* Caching
* Session storage
* Queues
* Leaderboards

## Traditional Database Strengths

* Complex queries
* ACID guarantees
* Relational data
* Permanent storage

## Best Practice

Use Redis together with databases like:

* MySQL
* PostgreSQL
* SQL Server
* MongoDB

Redis is often used as a caching layer.

---

# 9. Redis Use Cases

## 1. Caching

Store frequently accessed data.

Example:

```text
Application -> Redis Cache -> Database
```

Benefits:

* Faster responses
* Reduced database load

## 2. Session Storage

Store user login sessions.

## 3. Real-Time Analytics

Track counters and metrics.

## 4. Pub/Sub Messaging

Used for chat systems and notifications.

## 5. Leaderboards

Sorted Sets are ideal for rankings.

## 6. Queue Systems

Redis Lists and Streams support queues.

## 7. Rate Limiting

Control API request frequency.

---

# 10. Redis Installation

## Install on Windows

### Option 1: Docker

```bash
docker run --name redis -p 6379:6379 redis
```

### Option 2: WSL

Install Redis inside Ubuntu on WSL.

## Install on Linux

### Ubuntu

```bash
sudo apt update
sudo apt install redis-server
```

### Start Redis

```bash
sudo systemctl start redis
```

### Check Status

```bash
sudo systemctl status redis
```

## Install on macOS

Using Homebrew:

```bash
brew install redis
```

Start Redis:

```bash
brew services start redis
```

---

# 11. Redis CLI

## What is Redis CLI?

`redis-cli` is the command-line interface used to communicate with Redis.

## Start CLI

```bash
redis-cli
```

## Test Connection

```bash
PING
```

Response:

```text
PONG
```

## Common Commands

### Set Value

```bash
SET name Mohamed
```

### Get Value

```bash
GET name
```

### Check Keys

```bash
KEYS *
```

### Delete Key

```bash
DEL name
```

### Expiration

```bash
EXPIRE name 60
```

---

# 12. Redis Configuration

## Redis Configuration File

Main configuration file:

```text
redis.conf
```

## Important Settings

### Port

```conf
port 6379
```

### Bind Address

```conf
bind 127.0.0.1
```

### Password

```conf
requirepass mypassword
```

### Max Memory

```conf
maxmemory 256mb
```

### Persistence

```conf
save 900 1
```

## Reload Configuration

```bash
CONFIG REWRITE
```

---

# 13. Redis Persistence Overview

## Why Persistence Matters

Redis stores data in memory, so persistence is needed to avoid data loss.

## Persistence Types

### RDB (Snapshotting)

Creates point-in-time snapshots.

Advantages:

* Compact files
* Faster recovery

Disadvantages:

* Possible data loss between snapshots

### AOF (Append Only File)

Logs every write operation.

Advantages:

* Better durability

Disadvantages:

* Larger files
* Slightly slower

## Hybrid Persistence

Redis can combine RDB and AOF.

---

# 14. Redis Security Basics

## Security Risks

Exposed Redis servers can be dangerous.

## Basic Security Practices

### Use Passwords

```conf
requirepass strongpassword
```

### Bind to Localhost

```conf
bind 127.0.0.1
```

### Disable Dangerous Commands

Example:

```conf
rename-command FLUSHALL ""
```

### Use Firewall Rules

Restrict access to trusted systems.

### Enable TLS

Encrypt network communication.

---

# 15. Redis Networking

## Communication Protocol

Redis uses TCP.

Default port:

```text
6379
```

## Client Connections

Applications connect using:

* Redis clients
* Drivers
* Libraries

## Example .NET Libraries

* StackExchange.Redis
* ServiceStack.Redis

## Connection Example

```csharp
var redis = ConnectionMultiplexer.Connect("localhost:6379");
```

## Networking Features

* Replication
* Clustering
* Sentinel
* Pub/Sub

---

# 16. Redis Memory Management

## Memory Optimization

Redis is memory-focused, so efficient memory usage is important.

## Memory Policies

### noeviction

No keys removed.

### allkeys-lru

Least recently used keys removed.

### volatile-ttl

Removes keys with expiration.

## Set Maximum Memory

```conf
maxmemory 512mb
```

## Monitor Memory

```bash
INFO memory
```

## Memory Fragmentation

Redis may consume more memory due to fragmentation.

## Best Practices

* Use expiration times
* Avoid huge keys
* Monitor memory continuously
* Use proper eviction policies

---

# Conclusion

Redis is one of the most powerful and fastest in-memory databases available today. It is widely used in modern applications for:

* Caching
* Real-time systems
* Session storage
* Messaging
* Analytics

Understanding Redis fundamentals is essential for backend developers, system architects, and cloud engineers.

---

# Additional Learning Resources

## Official Documentation

* Redis Documentation
* Redis University

## Recommended Topics to Learn Next

* Redis Data Types Deep Dive
* Redis Transactions
* Redis Replication
* Redis Sentinel
* Redis Clustering
* Redis Streams
* Redis with .NET
* Distributed Caching
* High Availability
* Performance Tuning

---

# Quick Redis Cheat Sheet

| Command            | Description          |
| ------------------ | -------------------- |
| SET key value      | Store value          |
| GET key            | Retrieve value       |
| DEL key            | Delete key           |
| EXISTS key         | Check existence      |
| EXPIRE key seconds | Set expiration       |
| TTL key            | Remaining expiration |
| KEYS *             | List all keys        |
| FLUSHALL           | Remove all data      |
| INFO               | Server information   |
| PING               | Health check         |

---

# Author Notes

This README is designed as beginner-friendly Redis fundamentals documentation suitable for:

* Students
* Backend Developers
* .NET Developers
* Full Stack Developers
* DevOps Engineers
* System Architects

It can also be used as:

* Course material
* Team onboarding documentation
* Interview preparation notes
* Personal study guide
