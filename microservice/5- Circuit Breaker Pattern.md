# Circuit Breaker Pattern

## Overview

The Circuit Breaker Pattern is a resilience pattern used in distributed systems and microservices architectures to prevent cascading failures.

When a service repeatedly fails or becomes unavailable, the circuit breaker stops sending requests to that service temporarily.  
This helps protect the system from overload and improves overall stability.

---

# Why Do We Need Circuit Breakers?

In microservices, services communicate with each other frequently.

If one service becomes slow or unavailable:
- Requests start timing out
- Threads become blocked
- System resources become exhausted
- Other services may also fail

This creates a **cascading failure** across the system.

The Circuit Breaker Pattern prevents this problem.

---

# How It Works

The circuit breaker monitors service calls and switches between different states depending on failures and recovery.

---

# Circuit Breaker States

## 1. Closed State

The circuit is working normally.

- Requests are allowed
- Failures are monitored
- Failure count is tracked

### Behavior
If failures exceed a threshold, the circuit moves to the **Open State**.

---

## 2. Open State

The circuit blocks requests temporarily.

- No requests are sent to the failing service
- Requests fail immediately
- The system avoids wasting resources

### Behavior
After a timeout period, the circuit moves to the **Half-Open State**.

---

## 3. Half-Open State

The circuit allows a small number of test requests.

### Behavior
- If requests succeed → circuit returns to **Closed**
- If requests fail → circuit returns to **Open**

This state checks whether the service has recovered.

---

# Example Scenario

Imagine an Order Service communicating with a Payment Service.

## Without Circuit Breaker
- Payment Service becomes slow
- Order Service waits for responses
- Threads become blocked
- System performance degrades

---

## With Circuit Breaker
- Multiple failures are detected
- Circuit opens
- Requests stop temporarily
- System remains responsive

---

# Benefits

## Improved System Stability
Prevents cascading failures.

## Faster Failure Handling
Requests fail immediately instead of waiting for timeouts.

## Better Resource Usage
Prevents unnecessary thread and connection usage.

## Automatic Recovery
The system automatically retries after recovery periods.

---

# Common Configuration Settings

| Setting | Description |
|---|---|
| Failure Threshold | Number of failures before opening the circuit |
| Timeout Duration | Time before retrying requests |
| Success Threshold | Successful requests needed to close the circuit |
| Exception Types | Which exceptions should trigger failures |

---

# Circuit Breaker Flow

```text
Closed → Failures Detected → Open
Open → Timeout Expired → Half-Open
Half-Open → Success → Closed
Half-Open → Failure → Open