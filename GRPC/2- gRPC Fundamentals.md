# gRPC Fundamentals with .NET

## 1. What is gRPC

gRPC is a high-performance Remote Procedure Call (RPC) framework created by urlGoogle[https://grpc.io](https://grpc.io). It allows applications and services to communicate with each other efficiently across networks.

With gRPC, a client can directly call methods on a server application as if they were local methods.

### Key Features

* Built on top of HTTP/2
* Uses Protocol Buffers (Protobuf) for serialization
* Strongly typed contracts
* High performance and low latency
* Supports multiple programming languages
* Supports streaming communication
* Excellent for microservices architecture

### Common Use Cases

* Microservices communication
* Real-time systems
* Distributed systems
* Internal backend APIs
* Mobile and IoT applications
* High-performance APIs

---

# 2. RPC vs REST

## What is RPC?

RPC stands for Remote Procedure Call.

In RPC, the client calls a specific method on the server.

Example:

```text
CreateOrder()
GetProductById()
CalculateTotal()
```

The focus is on actions and functions.

---

## What is REST?

REST is an architectural style based on resources.

In REST APIs, communication usually happens using:

* HTTP verbs (GET, POST, PUT, DELETE)
* JSON payloads
* URL-based resources

Example:

```http
GET /products/1
POST /orders
```

The focus is on resources.

---

## gRPC (RPC) vs REST Comparison

| Feature         | gRPC              | REST            |
| --------------- | ----------------- | --------------- |
| Protocol        | HTTP/2            | HTTP/1.1 mostly |
| Data Format     | Binary (Protobuf) | JSON            |
| Performance     | Very Fast         | Slower          |
| Streaming       | Supported         | Limited         |
| Browser Support | Limited           | Excellent       |
| Contract        | Strongly Typed    | Flexible        |
| Payload Size    | Small             | Larger          |
| Learning Curve  | Medium            | Easy            |

---

## When to Use gRPC

Use gRPC when:

* Performance matters
* Services communicate internally
* You need streaming
* You use microservices
* Low latency is important

## When to Use REST

Use REST when:

* Building public APIs
* Browser compatibility is required
* Simplicity is preferred
* Third-party integrations are needed

---

# 3. Why gRPC is Fast

gRPC is designed for high performance.

## Main Reasons

### 1. HTTP/2

gRPC uses HTTP/2 instead of HTTP/1.1.

Benefits:

* Multiplexing
* Header compression
* Persistent connections
* Faster communication

---

### 2. Protocol Buffers

gRPC uses Protobuf instead of JSON.

Benefits:

* Binary serialization
* Smaller payloads
* Faster parsing
* Less bandwidth usage

---

### 3. Strongly Typed Contracts

The contract is predefined in `.proto` files.

This reduces:

* Runtime parsing overhead
* Ambiguity
* Validation complexity

---

### 4. Streaming Support

gRPC supports:

* Server streaming
* Client streaming
* Bidirectional streaming

This enables efficient real-time communication.

---

### 5. Persistent Connections

HTTP/2 keeps connections open.

This avoids repeatedly opening and closing TCP connections.

---

# 4. HTTP/2 Basics

HTTP/2 is the transport protocol used by gRPC.

It improves performance significantly compared to HTTP/1.1.

## Important Features

### Multiplexing

Multiple requests can be sent simultaneously over a single connection.

This removes the blocking problem in HTTP/1.1.

---

### Header Compression

HTTP/2 compresses headers.

This reduces:

* Network traffic
* Request size
* Latency

---

### Persistent Connection

A single TCP connection remains open for multiple requests.

Benefits:

* Faster communication
* Lower overhead
* Better scalability

---

### Streaming

HTTP/2 supports full duplex communication.

This allows:

* Real-time updates
* Continuous data transfer
* Bidirectional communication

---

## HTTP/1.1 vs HTTP/2

| Feature            | HTTP/1.1 | HTTP/2            |
| ------------------ | -------- | ----------------- |
| Multiplexing       | No       | Yes               |
| Header Compression | No       | Yes               |
| Streaming          | Limited  | Full              |
| Performance        | Slower   | Faster            |
| Connection Usage   | Multiple | Single Persistent |

---

# 5. Binary Serialization

Serialization means converting objects into a transferable format.

## JSON Serialization

REST commonly uses JSON.

Example:

```json
{
  "id": 1,
  "name": "Laptop"
}
```

JSON is:

* Human readable
* Flexible
* Larger in size
* Slower to parse

---

## Binary Serialization in gRPC

gRPC uses binary serialization through Protocol Buffers.

Benefits:

* Compact data size
* Faster serialization
* Faster deserialization
* Better network efficiency

---

## Why Binary is Faster

Binary data:

* Requires fewer bytes
* Needs less CPU processing
* Transfers faster over networks
* Consumes less bandwidth

---

## Performance Impact

Compared to JSON:

* Smaller payloads
* Reduced latency
* Better throughput
* Improved scalability

---

# 6. Protocol Buffers Overview

Protocol Buffers (Protobuf) is Google's language-neutral serialization format used by gRPC.

It defines contracts between client and server.

---

## .proto Files

Services and messages are defined inside `.proto` files.

Example:

```proto
syntax = "proto3";

service ProductService {
  rpc GetProduct(ProductRequest) returns (ProductResponse);
}

message ProductRequest {
  int32 id = 1;
}

message ProductResponse {
  int32 id = 1;
  string name = 2;
  double price = 3;
}
```

---

## Main Components

### Service

Defines available RPC methods.

Example:

```proto
service ProductService
```

---

### RPC Method

Defines operations.

Example:

```proto
rpc GetProduct(ProductRequest) returns (ProductResponse);
```

---

### Message

Defines data structures.

Example:

```proto
message ProductResponse
```

---

### Field Numbers

Each field has a unique number.

Example:

```proto
string name = 2;
```

These numbers are important for binary serialization.

---

## Advantages of Protobuf

* Compact binary format
* Strong typing
* Versioning support
* Cross-platform support
* Faster than JSON
* Auto-generated code

---

# Summary

After learning these fundamentals, you should understand:

* What gRPC is
* How RPC differs from REST
* Why gRPC is high performance
* The role of HTTP/2
* Binary serialization concepts
* How Protocol Buffers work

These concepts are the foundation for building gRPC applications with .NET.
