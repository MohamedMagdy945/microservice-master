# Redis Memory Optimization

Redis is an in-memory database, which means memory management is critical for performance, scalability, and cost efficiency.

Optimizing Redis memory usage helps:

* Reduce infrastructure costs
* Improve performance
* Prevent out-of-memory errors
* Increase scalability
* Improve cache efficiency

---

# Table of Contents

1. Understanding Redis Memory Usage
2. Memory Management Basics
3. Redis Memory Policies
4. Data Structure Optimization
5. Key Expiration Strategies
6. Memory-Efficient Data Modeling
7. Compression Techniques
8. Redis Eviction Policies
9. Lazy Freeing
10. Memory Monitoring Commands
11. Fragmentation Management
12. Production Best Practices

---

# 1. Understanding Redis Memory Usage

Redis stores all data in RAM.

Memory usage depends on:

* Number of keys
* Key size
* Value size
* Data structure type
* Internal metadata overhead

---

## Important Concepts

| Concept       | Description                     |
| ------------- | ------------------------------- |
| RAM           | Main memory used by Redis       |
| Dataset       | Total stored Redis data         |
| Overhead      | Internal metadata memory        |
| Fragmentation | Wasted memory due to allocation |

---

# 2. Memory Management Basics

Redis provides configuration options to limit and manage memory.

---

## Setting Maximum Memory

```bash
maxmemory 2gb
```

This limits Redis memory usage to 2 GB.

---

## Why Memory Limits Matter

Without limits:

* Redis may consume all server RAM
* System instability may occur
* Applications may crash

---

# 3. Redis Memory Policies

When Redis reaches the memory limit, eviction policies determine what data gets removed.

---

## Common Policies

| Policy          | Description                                     |
| --------------- | ----------------------------------------------- |
| noeviction      | Reject writes when memory is full               |
| allkeys-lru     | Remove least recently used keys                 |
| volatile-lru    | Remove least recently used keys with expiration |
| allkeys-random  | Remove random keys                              |
| volatile-random | Remove random expiring keys                     |
| volatile-ttl    | Remove keys with shortest TTL                   |

---

## Example Configuration

```bash
maxmemory-policy allkeys-lru
```

---

# 4. Data Structure Optimization

Choosing the correct data structure significantly reduces memory usage.

---

## Use Hashes for Small Objects

Instead of:

```text
user:1:name
user:1:email
user:1:country
```

Use:

```bash
HSET user:1 name "Mohamed" email "test@mail.com"
```

---

## Benefits of Hashes

* Lower memory overhead
* Better organization
* Faster retrieval
* Reduced key count

---

## Prefer Smaller Keys

Avoid long key names.

Bad:

```text
application:production:user:profile:1001
```

Better:

```text
usr:1001
```

---

# 5. Key Expiration Strategies

Expiration prevents unused data from consuming memory forever.

---

## Setting Expiration

```bash
SET session:1 "data"
EXPIRE session:1 3600
```

---

## Benefits

* Automatic cleanup
* Reduced memory growth
* Better cache efficiency

---

## Common Expiration Use Cases

| Use Case      | TTL Example |
| ------------- | ----------- |
| Sessions      | 30 minutes  |
| API Cache     | 5 minutes   |
| OTP Codes     | 60 seconds  |
| Rate Limiting | 1 minute    |

---

# 6. Memory-Efficient Data Modeling

Efficient schema design improves Redis memory utilization.

---

## Best Practices

* Avoid duplicate data
* Use compact key names
* Store only required fields
* Remove stale data
* Prefer hashes over multiple strings

---

## Example

Instead of storing:

```text
user:1:name
user:1:age
user:1:country
```

Use:

```bash
HSET user:1 name "Mohamed" age 25 country "Egypt"
```

---

# 7. Compression Techniques

Compression reduces memory consumption for large datasets.

---

## Common Approaches

* Compress JSON payloads
* Store binary formats
* Use shorter field names
* Use serialization formats like MessagePack

---

## Trade-offs

Compression improves memory usage but may increase:

* CPU usage
* Serialization overhead
* Response latency

---

# 8. Redis Eviction Policies

Eviction policies automatically remove data when memory is full.

---

## LRU (Least Recently Used)

Removes keys that were not accessed recently.

```bash
maxmemory-policy allkeys-lru
```

---

## LFU (Least Frequently Used)

Removes rarely accessed keys.

```bash
maxmemory-policy allkeys-lfu
```

---

## TTL-Based Eviction

Prioritizes keys with shorter expiration.

```bash
maxmemory-policy volatile-ttl
```

---

# 9. Lazy Freeing

Lazy freeing allows Redis to free memory asynchronously.

This reduces blocking operations.

---

## Enable Lazy Freeing

```bash
lazyfree-lazy-eviction yes
lazyfree-lazy-expire yes
```

---

## Benefits

* Reduced latency spikes
* Better performance under heavy load
* Faster key deletion

---

# 10. Memory Monitoring Commands

Redis provides commands to inspect memory usage.

---

## INFO MEMORY

Displays memory statistics.

```bash
INFO MEMORY
```

---

## MEMORY USAGE

Checks memory usage of a specific key.

```bash
MEMORY USAGE user:1
```

---

## MEMORY STATS

Shows detailed allocator statistics.

```bash
MEMORY STATS
```

---

## MEMORY DOCTOR

Analyzes memory problems.

```bash
MEMORY DOCTOR
```

---

# 11. Fragmentation Management

Memory fragmentation happens when allocated memory becomes inefficiently organized.

---

## Symptoms

* High memory usage
* Low available RAM
* Increased fragmentation ratio

---

## Monitoring Fragmentation

```bash
INFO MEMORY
```

Check:

```text
mem_fragmentation_ratio
```

---

## Solutions

* Restart Redis periodically
* Upgrade allocator
* Optimize data structures
* Reduce large object churn

---

# 12. Production Best Practices

## Recommended Strategies

* Set maxmemory limits
* Use expiration aggressively
* Choose correct eviction policy
* Monitor memory continuously
* Use hashes for small objects
* Avoid oversized values
* Enable lazy freeing
* Optimize key naming

---

# Common Optimization Checklist

| Optimization      | Benefit                  |
| ----------------- | ------------------------ |
| Short keys        | Lower overhead           |
| Hashes            | Better memory efficiency |
| TTL usage         | Automatic cleanup        |
| Compression       | Reduced RAM usage        |
| Eviction policies | Prevent crashes          |
| Monitoring        | Early issue detection    |

---

# Summary

Redis memory optimization is essential for:

* High performance
* Scalability
* Cost efficiency
* System stability

Effective memory management combines:

* Proper data modeling
* Expiration strategies
* Monitoring tools
* Eviction policies
* Efficient persistence configuration

Mastering Redis memory optimization helps build reliable and scalable production systems.
