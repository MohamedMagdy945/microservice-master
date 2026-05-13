# 7. Error Handling & Resilience

Reliable gRPC applications must handle failures correctly and recover gracefully.

Error handling and resiliency features in gRPC include:

* RpcException
* Status codes
* Deadlines
* Timeouts
* Cancellation tokens
* Retries

---

# RpcException

`RpcException` is the primary exception type in gRPC.

It is used to return errors between client and server.

---

## Throwing RpcException

```csharp
throw new RpcException(
    new Status(StatusCode.NotFound, "Product not found"));
```

---

## Handling RpcException

```csharp
try
{
    var response = await client.GetProductAsync(
        new ProductRequest { Id = 1 });
}
catch (RpcException ex)
{
    Console.WriteLine(ex.Status.StatusCode);
    Console.WriteLine(ex.Status.Detail);
}
```

---

# Status Codes

gRPC provides built-in status codes for communication errors.

---

## Common Status Codes

| Status Code      | Description             |
| ---------------- | ----------------------- |
| OK               | Successful request      |
| InvalidArgument  | Invalid request data    |
| NotFound         | Resource not found      |
| AlreadyExists    | Resource already exists |
| PermissionDenied | Access denied           |
| Unauthenticated  | Authentication failed   |
| DeadlineExceeded | Timeout exceeded        |
| Cancelled        | Request cancelled       |
| Internal         | Internal server error   |
| Unavailable      | Service unavailable     |

---

## Example

```csharp
throw new RpcException(
    new Status(StatusCode.InvalidArgument,
    "Invalid product id"));
```

---

# Deadlines

A deadline specifies the maximum time allowed for an RPC call.

If the deadline is exceeded, the request automatically fails.

---

## Example

```csharp
var response = await client.GetProductAsync(
    new ProductRequest { Id = 1 },
    deadline: DateTime.UtcNow.AddSeconds(5));
```

---

# Benefits of Deadlines

* Prevent hanging requests
* Protect server resources
* Improve application responsiveness
* Improve resiliency

---

# Timeouts

Timeouts stop requests that take too long.

---

## Example

```csharp
using var cts = new CancellationTokenSource(
    TimeSpan.FromSeconds(3));

await client.GetProductAsync(
    new ProductRequest { Id = 1 },
    cancellationToken: cts.Token);
```

---

# Cancellation Tokens

gRPC supports `CancellationToken` for graceful cancellation.

---

## Server Example

```csharp
public override async Task<ProductResponse> GetProduct(
    ProductRequest request,
    ServerCallContext context)
{
    context.CancellationToken.ThrowIfCancellationRequested();

    await Task.Delay(5000, context.CancellationToken);

    return new ProductResponse();
}
```

---

# Retries

Retries help recover from temporary failures.

Typical retry scenarios:

* Network failures
* Temporary outages
* Service unavailable errors

---

# Retry Best Practices

* Retry transient failures only
* Avoid retrying invalid requests
* Use exponential backoff
* Limit retry attempts

---

# Error Handling Flow

```text
Client Request
      ↓
gRPC Call
      ↓
RpcException
      ↓
Status Code Returned
      ↓
Client Handles Error
```

---

# Summary

In this section you learned:

* RpcException
* Status codes
* Deadlines
* Timeouts
* Cancellation tokens
* Retries
* Resilient gRPC communication

---

# 8. Streaming in Depth

Streaming is one of the most powerful features in gRPC.

It enables real-time communication between client and server.

Streaming types:

* Server Streaming
* Client Streaming
* Bidirectional Streaming

---

# Server Streaming

In server streaming:

* Client sends one request
* Server sends multiple responses

---

## Flow

```text
Request → Multiple Responses
```

---

# Real-time Updates

Server streaming is ideal for real-time updates.

Examples:

* Live dashboards
* Stock prices
* Tracking systems
* Monitoring systems

---

# Notifications

gRPC can continuously push notifications to clients.

Benefits:

* Persistent connection
* Low latency
* Real-time delivery

---

## .proto Example

```proto
service NotificationService {
  rpc StreamNotifications(NotificationRequest)
      returns (stream NotificationResponse);
}
```

---

## ASP.NET Core Example

```csharp
public override async Task StreamNotifications(
    NotificationRequest request,
    IServerStreamWriter<NotificationResponse> responseStream,
    ServerCallContext context)
{
    for (int i = 0; i < 5; i++)
    {
        await responseStream.WriteAsync(
            new NotificationResponse
            {
                Message = $"Notification {i}"
            });

        await Task.Delay(1000);
    }
}
```

---

# Client Streaming

In client streaming:

* Client sends multiple requests
* Server returns one response

---

## Flow

```text
Multiple Requests → One Response
```

---

# Uploading Data

Client streaming is commonly used for:

* File uploads
* Image uploads
* Video uploads
* Telemetry data

---

# Batch Processing

Client streaming is useful for:

* Bulk inserts
* Analytics processing
* Sending large datasets

---

## .proto Example

```proto
service FileService {
  rpc UploadFile(stream FileChunk)
      returns (UploadStatus);
}
```

---

## ASP.NET Core Example

```csharp
public override async Task<UploadStatus> UploadFile(
    IAsyncStreamReader<FileChunk> requestStream,
    ServerCallContext context)
{
    await foreach (var chunk in requestStream.ReadAllAsync())
    {
    }

    return new UploadStatus
    {
        Success = true
    };
}
```

---

# Bidirectional Streaming

Bidirectional streaming allows:

* Client sends multiple messages
* Server sends multiple messages
* Communication happens simultaneously

---

## Flow

```text
Two-way streaming
```

---

# Chat Systems

Bidirectional streaming is perfect for:

* Chat applications
* Messaging systems
* Multiplayer games
* Real-time collaboration

---

# Real-time Communication

Benefits:

* Persistent connection
* Instant communication
* Low latency
* Continuous data transfer

---

## .proto Example

```proto
service ChatService {
  rpc Chat(stream ChatMessage)
      returns (stream ChatMessage);
}
```

---

## ASP.NET Core Example

```csharp
public override async Task Chat(
    IAsyncStreamReader<ChatMessage> requestStream,
    IServerStreamWriter<ChatMessage> responseStream,
    ServerCallContext context)
{
    await foreach (var message in requestStream.ReadAllAsync())
    {
        await responseStream.WriteAsync(message);
    }
}
```

---

# Streaming Comparison

| Type                    | Client Messages | Server Messages |
| ----------------------- | --------------- | --------------- |
| Server Streaming        | One             | Multiple        |
| Client Streaming        | Multiple        | One             |
| Bidirectional Streaming | Multiple        | Multiple        |

---

# Summary

In this section you learned:

* Server Streaming
* Real-time updates
* Notifications
* Client Streaming
* Uploading data
* Batch processing
* Bidirectional Streaming
* Chat systems
* Real-time communication

---

# 9. Interceptors & Middleware

Interceptors are middleware components in gRPC.

They handle cross-cutting concerns such as:

* Logging
* Authentication
* Exception handling
* Request manipulation
* Response manipulation

---

# Logging Interceptors

Logging interceptors monitor requests and responses.

---

## Example

```csharp
public class LoggingInterceptor : Interceptor
{
    public override async Task<TResponse> UnaryServerHandler<TRequest, TResponse>(
        TRequest request,
        ServerCallContext context,
        UnaryServerMethod<TRequest, TResponse> continuation)
    {
        Console.WriteLine($"Request: {typeof(TRequest).Name}");

        var response = await continuation(request, context);

        Console.WriteLine($"Response: {typeof(TResponse).Name}");

        return response;
    }
}
```

---

# Registering Interceptors

```csharp
builder.Services.AddGrpc(options =>
{
    options.Interceptors.Add<LoggingInterceptor>();
});
```

---

# Authentication Interceptors

Authentication interceptors validate:

* Tokens
* Headers
* Credentials

---

## Example

```csharp
public class AuthInterceptor : Interceptor
{
}
```

---

# Exception Interceptors

Exception interceptors handle unhandled exceptions globally.

---

## Example

```csharp
public class ExceptionInterceptor : Interceptor
{
    public override async Task<TResponse> UnaryServerHandler<TRequest, TResponse>(
        TRequest request,
        ServerCallContext context,
        UnaryServerMethod<TRequest, TResponse> continuation)
    {
        try
        {
            return await continuation(request, context);
        }
        catch (Exception ex)
        {
            throw new RpcException(
                new Status(StatusCode.Internal, ex.Message));
        }
    }
}
```

---

# Request/Response Manipulation

Interceptors can inspect and modify:

* Requests
* Responses
* Metadata
* Headers

---

# Common Middleware Scenarios

| Scenario           | Purpose                |
| ------------------ | ---------------------- |
| Logging            | Monitor requests       |
| Authentication     | Validate users         |
| Exception Handling | Global error handling  |
| Metrics            | Performance monitoring |
| Tracing            | Distributed tracing    |

---

# Interceptor Flow

```text
Client Request
      ↓
Interceptor
      ↓
gRPC Service
      ↓
Interceptor
      ↓
Client Response
```

---

# Summary

In this section you learned:

* Logging interceptors
* Authentication interceptors
* Exception interceptors
* Request/response manipulation
* Middleware concepts in gRPC
* Cross-cutting concerns handling
