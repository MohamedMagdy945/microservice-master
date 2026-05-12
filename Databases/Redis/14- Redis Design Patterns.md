# Redis Design Patterns

Redis is widely used in real-world systems to solve common distributed system problems. This guide presents practical design patterns used in production systems.

---

# Table of Contents

1. Leaderboards
2. Notification Systems
3. Chat Applications
4. Real-Time Tracking
5. Shopping Cart Systems
6. Token Storage
7. Authentication Systems
8. OTP Systems
9. Rate Limiting Systems

---

# 1. Leaderboards

Leaderboards rank users based on scores using Redis Sorted Sets (ZSET).

---

## Why Redis?

* Fast ranking operations
* Automatic sorting
* Efficient score updates

---

## Example

```bash
ZADD leaderboard 100 "user1"
ZADD leaderboard 250 "user2"
ZADD leaderboard 180 "user3"
```

---

## Get Top Players

```bash
ZRANGE leaderboard 0 10 WITHSCORES
```

---

## Use Cases

* Gaming leaderboards
* Competitive ranking systems
* Learning platforms

---

# 2. Notification Systems

Redis Pub/Sub is used for real-time notifications.

---

## Flow

```text
Service → Publish Event → Redis → Subscribers
```

---

## Example

```bash
PUBLISH notifications "New message received"
```

---

## Use Cases

* Push notifications
* Email triggers
* System alerts

---

# 3. Chat Applications

Redis enables real-time chat systems using Pub/Sub or Streams.

---

## Pub/Sub Example

```bash
PUBLISH chat "Hello everyone"
```

---

## Features

* Real-time messaging
* Lightweight communication
* Horizontal scaling support

---

# 4. Real-Time Tracking

Redis is used for tracking live data like location or user activity.

---

## Example

```bash
SET user:location:1 "31.2001,29.9187"
```

---

## Advanced Approach

* Use Streams for event tracking
* Use Geo commands for location tracking

---

## Use Cases

* Delivery tracking
* Ride sharing apps
* User activity monitoring

---

# 5. Shopping Cart Systems

Redis stores shopping cart data for fast access.

---

## Example Structure

```bash
HSET cart:user1 product1 2
HSET cart:user1 product2 1
```

---

## Benefits

* Fast read/write
* Session-based carts
* Temporary storage

---

## Expiration Example

```bash
EXPIRE cart:user1 3600
```

---

# 6. Token Storage

Redis stores authentication and access tokens securely.

---

## Example

```bash
SET token:abc123 "userId:1"
EXPIRE token:abc123 3600
```

---

## Use Cases

* API tokens
* Refresh tokens
* Session tokens

---

# 7. Authentication Systems

Redis is used for session-based authentication.

---

## Flow

```text
Login → Store Session in Redis → Validate on Requests
```

---

## Example

```bash
SET session:user1 "authenticated"
EXPIRE session:user1 1800
```

---

## Benefits

* Stateless servers
* Fast validation
* Scalable authentication

---

# 8. OTP Systems

Redis is ideal for storing One-Time Passwords (OTP).

---

## Example

```bash
SET otp:user1 "123456"
EXPIRE otp:user1 120
```

---

## Why Redis?

* Auto expiration
* Fast retrieval
* Secure temporary storage

---

# 9. Rate Limiting Systems

Redis is commonly used for API rate limiting.

---

## Simple Counter Example

```bash
INCR rate:user1
EXPIRE rate:user1 60
```

---

## Sliding Window Approach

* Use Sorted Sets for advanced rate limiting
* Track request timestamps

---

## Use Cases

* API protection
* Prevent abuse
* Traffic control

---

# Summary

Redis design patterns solve real-world distributed system problems such as:

* Real-time communication
* Authentication and security
* High-performance caching
* Scalable event processing
* System protection and rate limiting

These patterns are essential for building modern scalable backend systems using Redis.
