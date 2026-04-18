# Structured Logging Best Practices

## Overview

Structured logging is a way of writing logs using **named properties instead of plain text**. This makes logs easier to search, filter, and analyze in production systems.

Instead of logging strings, you log **data with meaning**.

---

## Problem with Plain Text Logging

Traditional logging:

```csharp id="st1"
logger.LogInformation("User 123 logged in at 10:30");
```

Issues:

* Hard to search by user or time
* Difficult to filter specific data
* Not machine-friendly

---

## Structured Logging Approach

Structured logging separates message template from data:

```csharp id="st2"
logger.LogInformation("User {UserId} logged in at {Time}", 123, DateTime.UtcNow);
```

Now logs are stored as structured fields:

* UserId = 123
* Time = actual timestamp

---

## Why Structured Logging Matters

* Enables powerful searching in log tools
* Improves observability in production systems
* Works well with tools like Serilog, Elasticsearch, Seq
* Makes debugging faster and more accurate

---

## Best Practices

### 1. Always use placeholders

Instead of:

```csharp id="bp1"
logger.LogInformation($"User {userId} logged in");
```

Use:

```csharp id="bp2"
logger.LogInformation("User {UserId} logged in", userId);
```

---

### 2. Keep log messages consistent

Good:

```csharp id="bp3"
"Order {OrderId} created successfully"
```

Avoid inconsistent formats like:

* "Order created"
* "New order: 123 created"

---

### 3. Log meaningful context

Include important metadata:

```csharp id="bp4"
logger.LogInformation("Payment processed for Order {OrderId} with Amount {Amount}", orderId, amount);
```

---

### 4. Avoid logging sensitive data

Never log:

* Passwords
* Tokens
* Credit card details

Bad example:

```csharp id="bp5"
logger.LogInformation("User login: {Password}", password);
```

---

### 5. Use correct log levels

* Information → normal flow
* Warning → unexpected but recoverable
* Error → failure
* Critical → system-level failure

---

### 6. Don’t over-log

Too many logs:

* Increase storage cost
* Slow performance
* Make debugging harder

---

### 7. Use correlation-friendly logs

Structured logging works best when combined with correlation IDs:

```csharp id="bp6"
logger.LogInformation("Request {RequestId} processed for User {UserId}", requestId, userId);
```

---

## Common Mistakes

* Using string interpolation instead of structured logging
* Logging too much low-value data
* Missing context in logs
* Mixing formats across the system

---

## Summary

* Structured logging stores data, not just text
* Always use placeholders instead of string concatenation
* Improves searchability and observability
* Essential for modern distributed systems

---

Next Topic: Correlation IDs & Request Context
