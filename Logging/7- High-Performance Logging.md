# High-Performance Logging

## Overview

High-performance logging focuses on reducing the impact of logging on application speed, memory usage, and CPU consumption. In high-load systems, logging can easily become a bottleneck if not handled correctly.

---

## Why Performance Matters

Logging runs on every request, so poor logging practices can cause:

* Increased response time
* High memory usage
* Excessive I/O operations
* Slower overall system performance

---

## Key Principles

### 1. Avoid unnecessary logs

Do not log everything. Only log what is useful for debugging and monitoring.

---

### 2. Use structured logging efficiently

Bad:

```csharp id="hp1"
logger.LogInformation($"Processing order {orderId}");
```

Good:

```csharp id="hp2"
logger.LogInformation("Processing order {OrderId}", orderId);
```

---

### 3. Use lazy evaluation

Avoid building expensive strings unless the log level is enabled:

```csharp id="hp3"
if (logger.IsEnabled(LogLevel.Debug))
{
    logger.LogDebug("Expensive data: {Data}", GenerateHeavyData());
}
```

---

### 4. Minimize I/O operations

Writing logs to disk or network is expensive. Batch or buffer logs when possible.

---

### 5. Use asynchronous logging

Some logging frameworks support async sinks to avoid blocking threads.

---

## Buffered vs Direct Logging

* Direct logging → immediate write (slower under load)
* Buffered logging → collects logs and writes in batches (faster)

---

## Sampling (Advanced)

In large systems, not all logs need to be recorded.

Sampling means:

* Log only a percentage of events
* Reduce noise and performance cost

---

## Common Optimizations

* Disable Debug logs in production
* Use minimal log levels (Warning or Error)
* Avoid logging large objects
* Use high-performance sinks (e.g., batching sinks)

---

## Best Practices

* Log only meaningful events
* Prefer structured logging over string concatenation
* Avoid heavy computations inside log calls
* Configure different log levels per environment
* Monitor logging performance regularly

---

## Summary

* Logging can impact performance if not optimized
* Use structured logging and lazy evaluation
* Reduce I/O overhead using batching or async sinks
* Control log levels carefully in production systems

---

Next Topic: Centralized Log Management
