# Phase 0 — Foundations

**Goal:** Understand *why* we log and *what* good logs look like — before writing any Elasticsearch or advanced .NET code.

**Time:** 1–2 days

---

## 1. What is logging?

### Theory

When your application runs, you cannot always watch it line by line. **Logging** means recording events that happened while the program ran: startup, user actions, errors, slow operations.

Think of logs like a **diary for your software**:

- *“The app started at 10:00.”*
- *“User 42 placed order 1001.”*
- *“Payment service returned an error.”*

Without logs, when something breaks in production, you only know “it failed” — not **why**, **when**, or **for whom**.

### Why not just use `Console.WriteLine`?

`Console.WriteLine("Error!")` works for tiny scripts. In real apps you need:

| Need | Why `WriteLine` falls short |
|------|---------------------------|
| **Severity** | Is this debug noise or a critical outage? |
| **Structure** | Can you search “all errors for UserId=42”? |
| **Destinations** | Console, file, and Elasticsearch at the same time |
| **Configuration** | Turn debug on in dev, off in production — without recompiling |
| **Performance** | Cheap logging when thousands of events fire per second |

Professional .NET apps use **`ILogger`** (built-in) and often **Serilog** on top (Phase 1).

---

## 2. Log levels (severity)

### Theory

Every log message has a **level** — how important or noisy it is. From least to most severe:

| Level | When to use | Example |
|-------|-------------|---------|
| **Trace** | Very detailed flow (rare in production) | “Entering method X with param Y” |
| **Debug** | Developer troubleshooting | “Cache miss for key abc” |
| **Information** | Normal business events | “Order 1001 created” |
| **Warning** | Something odd but app continues | “Retry 2/3 for payment API” |
| **Error** | Operation failed | “Could not save order to database” |
| **Critical** | App or subsystem may be unusable | “Database unreachable on startup” |

**Rule of thumb for production:**

- Default: **Information** and above
- **Debug** / **Trace**: only when investigating a bug (temporarily)

If everything is logged at Information, you cannot find real problems in the noise.

### Code example (conceptual)

```csharp
// Pseudocode — shows intent; real apps use ILogger (Phase 1)
LogLevel.Information  →  "User signed in"
LogLevel.Warning      →  "Email took 5 seconds"
LogLevel.Error        →  "Login failed: invalid password"
```

**What the code means:** You choose a level so filters (in config or Kibana later) can hide or show messages by severity.

---

## 3. Structured logging

### Theory

**Unstructured (bad for search):**

```text
User 42 placed order 1001 for $59.99
```

**Structured (good):**

```json
{
  "message": "Order placed",
  "UserId": 42,
  "OrderId": 1001,
  "Amount": 59.99
}
```

Each piece of data is a **field**. In Elasticsearch/Kibana you can filter: `UserId: 42` or `Amount > 50`.

In .NET, structured logging uses **message templates** with placeholders:

```csharp
logger.LogInformation("Order {OrderId} placed by {UserId} for {Amount}", 1001, 42, 59.99m);
```

**What the code does:**

- `"Order {OrderId} placed..."` — template with named holes
- `1001, 42, 59.99m` — values bound to `OrderId`, `UserId`, `Amount`
- The logging system stores them as separate properties, not one mashed string

---

## 4. Correlation — following one request

### Theory

One HTTP request might call a database, a cache, and another API. You want all related log lines to share an ID — a **correlation ID** (often `TraceId` or `RequestId`).

```text
RequestId: abc-123
  → Log: "Received GET /orders"
  → Log: "Querying database"
  → Log: "Calling payment service"
  → Log: "Response 200 OK"
```

In Kibana you search `RequestId: "abc-123"` and see the full story.

**What you will implement later:** Middleware in ASP.NET Core that reads or creates a correlation ID and adds it to every log in that request.

---

## 5. Logs vs metrics vs traces

### Theory

Observability has three pillars:

| Type | Answers | Example |
|------|---------|---------|
| **Logs** | What happened? (discrete events) | “Payment failed for order 1001” |
| **Metrics** | How much / how often? (numbers over time) | “95% of requests under 200ms” |
| **Traces** | Where did time go across services? | Span: API → DB → external API |

This roadmap focuses on **logs** and Elasticsearch. Phase 5 introduces **traces** with OpenTelemetry.

---

## 6. What you must NOT log

### Theory

Logs are often copied to shared systems. Never log:

- Passwords, API keys, connection strings
- Full credit card numbers
- Personal data you are not allowed to store (check GDPR/policies)

**Safe pattern:** Log an ID, not the secret.

```csharp
// BAD
logger.LogInformation("User logged in with password {Password}", password);

// GOOD
logger.LogInformation("User {UserId} logged in successfully", userId);
```

---

## 7. Hands-on exercise (no Elasticsearch yet)

### Step 1 — Create a console app

```powershell
dotnet new console -n LoggingBasics -f net9.0
cd LoggingBasics
dotnet add package Microsoft.Extensions.Logging
dotnet add package Microsoft.Extensions.Logging.Console
```

### Step 2 — Program.cs

```csharp
using Microsoft.Extensions.Logging;

// LoggerFactory creates loggers and applies configuration (e.g. console output)
using ILoggerFactory factory = LoggerFactory.Create(builder =>
{
    builder.AddConsole();           // Send logs to the terminal
    builder.SetMinimumLevel(LogLevel.Debug);  // Show Debug and above
});

// "Program" is the category name — helps identify which part of the app logged
ILogger logger = factory.CreateLogger("Program");

logger.LogDebug("Debug detail — usually hidden in production");
logger.LogInformation("Application started");
logger.LogWarning("This is a warning");
logger.LogError("This is an error");

// Structured: OrderId and UserId become searchable fields
logger.LogInformation("Order {OrderId} created for user {UserId}", 1001, 42);
```

### What each part does

| Line / block | Purpose |
|--------------|---------|
| `LoggerFactory.Create` | Builds the logging infrastructure |
| `builder.AddConsole()` | Registers the console “provider” (output target) |
| `SetMinimumLevel` | Ignores messages below Debug |
| `CreateLogger("Program")` | Gets a logger instance for category `Program` |
| `LogInformation(...)` | Writes an Information-level event |
| `{OrderId}` placeholders | Structured properties attached to the event |

### Step 3 — Run

```powershell
dotnet run
```

You should see colored output with levels and the structured property names.

---

## Phase 0 checklist

- [ ] I can explain why we log instead of only debugging locally
- [ ] I know the six common log levels and when to use each
- [ ] I understand structured logging with `{Named}` placeholders
- [ ] I know what a correlation ID is for
- [ ] I know what not to put in logs

**Next:** [Phase 1 — .NET Logging](../Phase-01-DotNet-Logging/README.md)
