# Built-in Providers & Configuration

## Overview

In .NET, logging is not limited to just writing messages. You can control where logs are written and how they are managed using **logging providers** and configuration settings.

A provider is responsible for sending log data to a specific destination such as the console, debug window, or external systems.nex

---

## What are Logging Providers?

Logging providers define the output destination of logs.

Common built-in providers in .NET:

* Console Provider → writes logs to terminal
* Debug Provider → writes logs to debugger output
* EventSource Provider → used for diagnostics tools

These providers can be enabled or disabled through configuration.

---

## How Logging Works Internally

When you use `ILogger`, the message is not written directly. Instead:

1. ILogger receives the log message
2. It passes it to registered providers
3. Each provider decides where to output it

---

## Example Setup

In `Program.cs`:

```csharp
var builder = WebApplication.CreateBuilder(args);

// Logging is enabled by default
builder.Logging.ClearProviders();
builder.Logging.AddConsole();
builder.Logging.AddDebug();

var app = builder.Build();
app.Run();
```

---

## Configuration Using appsettings.json

You can control logging behavior without changing code.

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  }
}
```

---

## Common Built-in Providers

### Console Provider

* Displays logs in terminal
* Useful for development

### Debug Provider

* Outputs logs to Visual Studio debug window
* Helps during debugging sessions

### EventSource Provider

* Sends logs to system diagnostics tools
* Used for performance monitoring

---

## Filtering Logs

You can control what gets logged per provider or namespace:

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "System": "Warning",
      "Microsoft.AspNetCore": "Error"
    }
  }
}
```

---

## Environment-based Configuration

Different environments can use different logging settings:

* Development → detailed logs
* Production → minimal logs

Example:

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Debug"
    }
  }
}
```

---

## Best Practices

* Use Console logging only in development
* Reduce log verbosity in production
* Avoid logging sensitive data
* Use configuration instead of hardcoding settings
* Enable only necessary providers to improve performance

---

## Summary

* Providers define where logs go
* .NET includes built-in providers like Console and Debug
* Logging behavior is configurable via `appsettings.json`
* Proper configuration improves performance and maintainability

---

Next Topic: Serilog — The Industry Standard
