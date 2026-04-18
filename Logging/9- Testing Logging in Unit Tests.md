# Testing Logging in Unit Tests

## Overview

Testing logging means verifying that your application writes the expected log messages during execution. This is useful when you want to ensure important events are being recorded correctly, especially in business-critical logic.

---

## Why Test Logging?

In real applications, logs are part of system behavior. Testing them helps you:

* Ensure important events are logged
* Detect missing error logs
* Validate correct log levels (Information, Warning, Error)
* Improve system observability reliability

---

## Basic Idea

Instead of writing logs to console or file, in unit tests you:

* Mock the `ILogger`
* Capture log calls
* Assert that logs were written correctly

---

## Example Setup (Mocking ILogger)

Using Moq:

```csharp id="lt1"
var loggerMock = new Mock<ILogger<OrderService>>();
var service = new OrderService(loggerMock.Object);
```

---

## Example Method

```csharp id="lt2"
public class OrderService
{
    private readonly ILogger<OrderService> _logger;

    public OrderService(ILogger<OrderService> logger)
    {
        _logger = logger;
    }

    public void CreateOrder(int orderId)
    {
        _logger.LogInformation("Order {OrderId} created", orderId);
    }
}
```

---

## Unit Test Example

```csharp id="lt3"
[Fact]
public void CreateOrder_ShouldLogInformation()
{
    var loggerMock = new Mock<ILogger<OrderService>>();
    var service = new OrderService(loggerMock.Object);

    service.CreateOrder(10);

    loggerMock.Verify(
        x => x.Log(
            LogLevel.Information,
            It.IsAny<EventId>(),
            It.Is<It.IsAnyType>((v, t) => v.ToString().Contains("Order 10 created")),
            It.IsAny<Exception>(),
            It.IsAny<Func<It.IsAnyType, Exception, string>>()),
        Times.Once);
}
```

---

## What This Test Checks

* A log message was written
* The log level is Information
* The message contains the correct OrderId
* It was logged exactly once

---

## Advanced Approach

Instead of verifying raw `Log()` calls, you can:

* Create a custom in-memory logger
* Capture logs into a list
* Assert against stored logs

---

## Example In-Memory Logger

```csharp id="lt4"
public class TestLogger : ILogger
{
    public List<string> Logs = new List<string>();

    public void Log<TState>(LogLevel logLevel, EventId eventId,
        TState state, Exception exception,
        Func<TState, Exception, string> formatter)
    {
        Logs.Add(formatter(state, exception));
    }

    public bool IsEnabled(LogLevel logLevel) => true;

    public IDisposable BeginScope<TState>(TState state) => null;
}
```

---

## Best Practices

* Only test important logs (errors, critical events)
* Don’t over-test simple logging statements
* Prefer verifying behavior over exact text when possible
* Use mocking libraries like Moq for cleaner tests
* Avoid coupling tests too tightly to log message wording

---

## Common Mistakes

* Testing every single log message (overkill)
* Depending too much on exact string matching
* Ignoring log level validation
* Not testing error logs in failure scenarios

---

## Summary

* Logging can and should be tested in critical cases
* Use mocking or in-memory loggers
* Focus on meaningful logs (errors, important actions)
* Keep tests maintainable and not overly strict

---

Next Topic: Real-World Logging Patterns & Production Tips
