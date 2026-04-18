# Core Abstractions — ILogger & Log Levels

## Overview

Logging is a fundamental part of any application. It helps developers understand what the application is doing, diagnose issues, and monitor behavior in production.

In .NET, logging is built around the `ILogger` abstraction, which provides a consistent way to write logs regardless of where they are sent (console, file, cloud, etc.).

---

## What is ILogger?

`ILogger` is an interface used to write log messages. It is part of the built-in logging system in .NET and is typically injected into classes using dependency injection.

### Example

```csharp
public class UserService
{
    private readonly ILogger<UserService> _logger;

    public UserService(ILogger<UserService> logger)
    {
        _logger = logger;
    }

    public void CreateUser(string username)
    {
        _logger.LogInformation("Creating user: {Username}", username);
    }
}
```

---

## Log Levels

Log levels define the severity or importance of a log message. Choosing the correct level is critical for filtering and monitoring.

### Available Levels

| Level       | Description                                                            |
| ----------- | ---------------------------------------------------------------------- |
| Trace       | Very detailed logs, typically only used for debugging low-level issues |
| Debug       | Useful for debugging during development                                |
| Information | General application flow and important events                          |
| Warning     | Something unexpected happened, but the app can continue                |
| Error       | A failure occurred in a specific operation                             |
| Critical    | A serious failure that may stop the application                        |

---

## Example Usage

```csharp
_logger.LogTrace("This is a trace log");
_logger.LogDebug("Debugging value: {Value}", value);
_logger.LogInformation("User logged in");
_logger.LogWarning("Low disk space");
_logger.LogError("Error while processing request");
_logger.LogCritical("System crash!");
```

---

## Why Log Levels Matter

* They allow filtering logs based on environment (e.g., Development vs Production)
* They reduce noise by showing only relevant information
* They help monitoring systems trigger alerts based on severity

---

## Configuration Example

Log levels are typically configured in `appsettings.json`:

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "System": "Warning"
    }
  }
}
```

---

## Best Practices

* Use `Information` for normal application flow
* Use `Warning` for recoverable issues
* Use `Error` for failures
* Avoid excessive logging in production
* Always use structured logging with placeholders

---

## Summary

* `ILogger` is the core abstraction for logging in .NET
* Log levels define the importance of messages
* Proper use of logging improves debugging, monitoring, and system reliability

---

Next Topic: Built-in Providers & Configuration
