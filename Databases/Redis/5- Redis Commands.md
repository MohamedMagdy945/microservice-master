# Redis Commands

Redis commands are used to interact with Redis data structures efficiently.
Each Redis data type has its own specialized set of commands.

---

# Table of Contents

1. String Commands
2. Hash Commands
3. List Commands
4. Set Commands
5. Sorted Set Commands
6. Key Commands

---

# 1. String Commands

String commands operate on Redis string values.

## SET

Stores a value in a key.

```bash
SET username "Mohamed"
```

---

## GET

Retrieves the value of a key.

```bash
GET username
```

---

## MGET

Retrieves multiple values at once.

```bash
MGET user1 user2 user3
```

---

## MSET

Stores multiple key-value pairs.

```bash
MSET user1 "Ahmed" user2 "Ali"
```

---

## INCR

Increments a numeric value.

```bash
INCR page:views
```

---

## DECR

Decrements a numeric value.

```bash
DECR stock:count
```

---

## APPEND

Appends text to an existing string.

```bash
APPEND username " Magdy"
```

---

## GETSET

Returns the old value and sets a new one.

```bash
GETSET username "Ali"
```

---

# 2. Hash Commands

Hash commands manage field-value pairs.

## HSET

Sets fields in a hash.

```bash
HSET user:1 name "Mohamed" age 25
```

---

## HGET

Retrieves a field value.

```bash
HGET user:1 name
```

---

## HMGET

Retrieves multiple field values.

```bash
HMGET user:1 name age
```

---

## HGETALL

Returns all fields and values.

```bash
HGETALL user:1
```

---

## HDEL

Deletes fields from a hash.

```bash
HDEL user:1 age
```

---

# 3. List Commands

List commands operate on ordered collections.

## LPUSH

Adds elements to the beginning of a list.

```bash
LPUSH tasks "task1"
```

---

## RPUSH

Adds elements to the end of a list.

```bash
RPUSH tasks "task2"
```

---

## LPOP

Removes the first element.

```bash
LPOP tasks
```

---

## RPOP

Removes the last element.

```bash
RPOP tasks
```

---

## LRANGE

Retrieves a range of elements.

```bash
LRANGE tasks 0 -1
```

---

# 4. Set Commands

Set commands manage unordered unique values.

## SADD

Adds members to a set.

```bash
SADD tags "redis" "database"
```

---

## SMEMBERS

Returns all set members.

```bash
SMEMBERS tags
```

---

## SISMEMBER

Checks if a member exists.

```bash
SISMEMBER tags "redis"
```

---

## SREM

Removes members from a set.

```bash
SREM tags "database"
```

---

# 5. Sorted Set Commands

Sorted Sets store members ordered by score.

## ZADD

Adds members with scores.

```bash
ZADD leaderboard 100 "Mohamed"
```

---

## ZRANGE

Retrieves members by range.

```bash
ZRANGE leaderboard 0 -1 WITHSCORES
```

---

## ZREM

Removes members from a sorted set.

```bash
ZREM leaderboard "Mohamed"
```

---

## ZSCORE

Returns the score of a member.

```bash
ZSCORE leaderboard "Mohamed"
```

---

# 6. Key Commands

Key commands manage Redis keys.

## DEL

Deletes keys.

```bash
DEL user:1
```

---

## EXISTS

Checks if a key exists.

```bash
EXISTS user:1
```

---

## EXPIRE

Sets expiration time.

```bash
EXPIRE session:1 3600
```

---

## TTL

Returns remaining expiration time.

```bash
TTL session:1
```

---

## KEYS

Searches keys by pattern.

```bash
KEYS user:*
```

> Warning: Avoid using `KEYS` in production on large datasets.

---

## SCAN

Incrementally scans keys safely.

```bash
SCAN 0 MATCH user:* COUNT 10
```

---

# Summary

Redis provides powerful and optimized commands for every data structure.

Understanding these commands is essential for:

* Efficient data management
* High-performance applications
* Scalable system design
* Faster Redis development
