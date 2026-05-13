# 3. Types of gRPC Communication

gRPC supports multiple communication patterns between client and server.

These communication types make gRPC flexible for different application scenarios.

---

# 1. Unary RPC

Unary RPC is the simplest and most common communication type in gRPC.

The client sends a single request and receives a single response.

---

## Flow

```text
Request → Response
```

---

## Example Use Cases

* Login requests
* Get product by id
* Create order
* Fetch user profile

---

## .proto Example

```proto
service ProductService {
  rpc GetProduct(ProductRequest) returns (ProductResponse);
}
```

---

## How It Works

1. Client sends one request
2. Server processes the request
3. Server returns one response
4. Connection closes after completion

---

## Visual Representation

```text
Client  ───── Request ─────► Server
Client ◄───── Response ───── Server
```

---

# 2. Server Streaming RPC

In Server Streaming RPC:

* Client sends one request
* Server sends multiple responses

The server continuously streams data back to the client.

---

## Flow

```text
Request → Multiple Responses
```

---

## Example Use Cases

* Live notifications
* Order tracking
* Real-time stock prices
* Streaming logs
* Chat updates

---

## .proto Example

```proto
service NotificationService {
  rpc StreamNotifications(NotificationRequest)
      returns (stream NotificationResponse);
}
```

---

## Understanding `stream`

The `stream` keyword means the server sends multiple messages.

---

## Visual Representation

```text
Client  ───── Request ─────► Server
Client ◄───── Response 1 ─── Server
Client ◄───── Response 2 ─── Server
Client ◄───── Response 3 ─── Server
```

---

# 3. Client Streaming RPC

In Client Streaming RPC:

* Client sends multiple requests
* Server returns one final response

The client continuously streams data to the server.

---

## Flow

```text
Multiple Requests → One Response
```

---

## Example Use Cases

* Uploading files
* Sending telemetry data
* Bulk inserts
* Sending analytics events
* Uploading images/videos

---

## .proto Example

```proto
service FileService {
  rpc UploadFile(stream FileChunk)
      returns (UploadStatus);
}
```

---

## Visual Representation

```text
Client  ───── Chunk 1 ─────► Server
Client  ───── Chunk 2 ─────► Server
Client  ───── Chunk 3 ─────► Server
Client ◄──── Final Response ─ Server
```

---

# 4. Bidirectional Streaming RPC

Bidirectional Streaming allows both:

* Client to send multiple messages
* Server to send multiple messages

Both sides communicate simultaneously.

---

## Flow

```text
Two-way streaming
```

---

## Example Use Cases

* Chat applications
* Multiplayer games
* Live collaboration systems
* Real-time dashboards
* Messaging systems

---

## .proto Example

```proto
service ChatService {
  rpc Chat(stream ChatMessage)
      returns (stream ChatMessage);
}
```

---

## How It Works

* Client sends messages continuously
* Server sends messages continuously
* Communication happens in parallel
* Both streams stay open simultaneously

---

## Visual Representation

```text
Client  ───── Message ─────► Server
Client ◄───── Message ───── Server
Client  ───── Message ─────► Server
Client ◄───── Message ───── Server
```

---

# Comparison of gRPC Communication Types

| Type                    | Client Requests | Server Responses | Streaming |
| ----------------------- | --------------- | ---------------- | --------- |
| Unary RPC               | One             | One              | No        |
| Server Streaming        | One             | Multiple         | Server    |
| Client Streaming        | Multiple        | One              | Client    |
| Bidirectional Streaming | Multiple        | Multiple         | Both      |

---

# Choosing the Right Communication Type

## Use Unary RPC When

* Simple request/response operations
* Standard CRUD APIs
* Fast synchronous operations

---

## Use Server Streaming When

* Server continuously pushes updates
* Real-time monitoring systems
* Notifications and event feeds

---

## Use Client Streaming When

* Uploading large data
* Sending multiple records
* Batch processing

---

## Use Bidirectional Streaming When

* Real-time communication is required
* Both client and server send updates continuously
* Building chat or collaborative systems

---

# Summary

In this section you learned:

* Unary RPC
* Server Streaming RPC
* Client Streaming RPC
* Bidirectional Streaming RPC
* Request/response flow of each type
* Real-world use cases
* `.proto` examples for each communication type
