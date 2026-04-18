# Serilog — The Industry Standard

## Overview

Serilog is one of the most widely used logging libraries in .NET. It provides structured logging, flexible output destinations (sinks), and powerful filtering capabilities.

Unlike basic logging, Serilog focuses on **structured data**, which makes logs easier to search, analyze, and send to external systems.

---

## Why Serilog?

Built-in logging in .NET is good, but Serilog adds:

* Structured logging (key-value based logs)
* Multiple outputs (files, databases, cloud services)
* Better formatting and readability
* Rich ecosystem of extensions

---

## Installing Serilog

You typically install these packages:

```bash id="ser1"
dotnet add package Serilog
dotnet add package Serilog.AspNetCore
dotnet add package Serilog.Sinks.Console
```

---

## Basic Setup

In `Program.cs`:

```csharp id="ser2"
using Serilog;

var builder = WebApplication.CreateBuilder(args);

Log.Logger = new LoggerConfiguration()
    .WriteTo.Console()
    .CreateLogger();

builder.Host.UseSerilog();

var app = builder.Build();

app.MapGet("/", (ILogger<Program> logger) =>
{
    logger.LogInformation("Application started");
    return "Hello World";
});

app.Run();
```

---

## Structured Logging

Instead of plain text logs, Serilog uses structured data.

### Example

```csharp id="ser3"
logger.LogInformation("User {UserId} logged in at {Time}", 123, DateTime.UtcNow);
```

This creates structured fields:

* UserId = 123
* Time = timestamp

---

## What makes it powerful

Structured logs allow:

* Easier searching in log tools
* Better filtering
* Faster debugging
* Integration with monitoring systems

---

## Serilog Sinks

A sink is where logs are sent.

Common sinks:

* Console
* File
* Database
* Elasticsearch
* Seq

### File Sink Example

```csharp id="ser4"
Log.Logger = new LoggerConfiguration()
    .WriteTo.File("logs/app.log")
    .CreateLogger();
```

---

## Configuration with appsettings.json

Serilog can also be configured externally.

```json id="ser5"
{
  "Serilog": {
    "MinimumLevel": "Information",
    "WriteTo": [
      { "Name": "Console" }
    ]
  }
}
```

---

## Log Levels in Serilog

Same concept as ILogger:

* Verbose
* Debug
* Information
* Warning
* Error
* Fatal

---

## Best Practices

* Always use structured logging instead of plain strings
* Avoid logging sensitive data (passwords, tokens)
* Use multiple sinks for production systems
* Keep log levels appropriate per environment
* Centralize logging configuration

---

## Summary

* Serilog is a powerful structured logging library
* It replaces or enhances default .NET logging
* Uses sinks to send logs anywhere
* Makes debugging and monitoring much easier

---

Next Topic: Structured Logging Best Practices
