# Redis Persistence

Redis persistence mechanisms allow data to survive server restarts, crashes, and failures.

Persistence is essential for production systems that require durability, backup recovery, and long-term data storage.

---

# Table of Contents

1. RDB Snapshots
2. AOF Persistence
3. RDB vs AOF
4. Hybrid Persistence
5. Backup Strategies
6. Recovery Strategies
7. Durability Concepts

---

# 1. RDB Snapshots

RDB (Redis Database Backup) creates point-in-time snapshots of the entire dataset.

Redis saves the dataset into a compact binary file called:

```text
 dump.rdb
```

---

## Features

* Compact binary format
* Fast loading performance
* Minimal runtime overhead
* Efficient for backups
* Suitable for disaster recovery

---

## How RDB Works

Redis creates snapshots automatically based on configured conditions.

Example configuration:

```bash
save 900 1
save 300 10
save 60 10000
```

Meaning:

| Configuration | Description                               |
| ------------- | ----------------------------------------- |
| save 900 1    | Save after 1 change within 15 minutes     |
| save 300 10   | Save after 10 changes within 5 minutes    |
| save 60 10000 | Save after 10,000 changes within 1 minute |

---

## Advantages

* High performance
* Smaller file sizes
* Faster startup time
* Ideal for periodic backups
* Easy file transfer

---

## Disadvantages

* Possible data loss between snapshots
* Less durable than AOF
* Snapshot generation may impact performance on huge datasets

---

# 2. AOF Persistence

AOF (Append Only File) logs every write operation received by Redis.

These operations are replayed during startup to rebuild the dataset.

---

## Features

* Higher durability
* Better crash protection
* Incremental write logging
* Human-readable log format

---

## Enabling AOF

Example configuration:

```bash
appendonly yes
appendfilename "appendonly.aof"
```

---

## Appendfsync Policies

Redis supports multiple synchronization policies.

| Mode     | Description                         |
| -------- | ----------------------------------- |
| always   | Sync every write operation          |
| everysec | Sync every second                   |
| no       | Let operating system handle syncing |

Example:

```bash
appendfsync everysec
```

---

## Advantages

* Minimal data loss
* Better reliability
* Safer persistence mechanism
* More durable than RDB

---

## Disadvantages

* Larger file sizes
* Slower write performance
* Longer recovery times

---

# 3. RDB vs AOF

Redis supports both persistence mechanisms.

Each has different trade-offs.

---

| Feature           | RDB           | AOF             |
| ----------------- | ------------- | --------------- |
| Performance       | Faster        | Slightly slower |
| Durability        | Lower         | Higher          |
| File Size         | Smaller       | Larger          |
| Recovery Speed    | Faster        | Slower          |
| Data Safety       | Moderate      | Excellent       |
| Readability       | Binary format | Human-readable  |
| Backup Efficiency | Better        | Moderate        |

---

## Recommended Usage

| Scenario                    | Recommended Persistence |
| --------------------------- | ----------------------- |
| High performance caching    | RDB                     |
| Financial systems           | AOF                     |
| Critical production systems | RDB + AOF               |
| Backup-focused systems      | RDB                     |

---

# 4. Hybrid Persistence

Redis supports combining RDB and AOF together.

Hybrid persistence uses:

* RDB snapshots for faster startup
* AOF logs for durability

---

## Benefits

* Faster recovery times
* Better durability
* Reduced AOF rewrite overhead
* Improved reliability

---

## Why Hybrid Persistence Matters

It combines the strengths of both persistence mechanisms:

| Benefit                  | Explanation                  |
| ------------------------ | ---------------------------- |
| Fast startup             | Uses compact RDB snapshots   |
| Better safety            | Uses AOF incremental logging |
| Reduced storage overhead | Smaller AOF rewrite size     |

---

# 5. Backup Strategies

Reliable backup strategies are essential for production Redis deployments.

---

## Recommended Backup Approaches

### Regular Snapshots

Create scheduled RDB snapshots periodically.

---

### Offsite Backups

Store backups in:

* Cloud storage
* Remote servers
* Backup clusters

---

### Replication

Use Redis replicas for additional redundancy.

---

### Automated Backups

Use automation tools or scripts to schedule backups.

---

## Backup Best Practices

* Test restore procedures regularly
* Encrypt sensitive backups
* Monitor backup duration
* Verify backup integrity
* Maintain backup retention policies

---

# 6. Recovery Strategies

Recovery strategies define how Redis restores data after failures.

---

## Common Recovery Methods

### Restore RDB Snapshot

Replace the existing dump.rdb file and restart Redis.

---

### Replay AOF File

Redis automatically replays AOF commands during startup.

---

### Replica Failover

Promote a replica server to primary during failures.

---

### Cluster Recovery

Recover failed nodes in Redis Cluster deployments.

---

## Recovery Planning Concepts

| Concept     | Description                 |
| ----------- | --------------------------- |
| RTO         | Recovery Time Objective     |
| RPO         | Recovery Point Objective    |
| Failover    | Switching to backup systems |
| Consistency | Maintaining accurate data   |

---

# 7. Durability Concepts

Durability refers to Redis ability to safely preserve data.

---

## Factors Affecting Durability

### Persistence Configuration

The selected persistence mechanism directly affects durability.

---

### Disk Synchronization

Frequent disk syncing improves crash protection.

---

### Replication

Multiple replicas improve fault tolerance.

---

### Hardware Reliability

Reliable storage hardware improves data safety.

---

## Durability Trade-offs

Higher durability usually means:

* More disk operations
* Increased latency
* Better crash protection

Lower durability usually means:

* Better performance
* Reduced safety guarantees

---

# Production Recommendations

## Best Practices

* Use AOF with `everysec` for balanced performance and safety
* Enable RDB snapshots for backups
* Use replication for high availability
* Store backups offsite
* Continuously monitor persistence performance

---

# Summary

Redis persistence mechanisms ensure:

* Data durability
* Backup reliability
* Crash recovery
* Fault tolerance

Choosing the correct persistence strategy depends on:

* Performance requirements
* Recovery objectives
* Durability needs
* System architecture

Understanding Redis persistence is essential for building reliable and scalable production systems.
