# gRPC with .NET Learning Roadmap

## 1. Fundamentals

* What is gRPC
* RPC vs REST
* Why gRPC is fast
* HTTP/2 basics
* Binary serialization
* Protocol Buffers overview

---

## 2. Protocol Buffers (Protobuf)

* `.proto` files
* Messages
* Scalar data types
* Field numbers
* Repeated fields
* Enums
* Nested messages
* Imports & packages
* Service definitions

### Example Concepts

* `message`
* `service`
* `rpc`

---

## 3. Types of gRPC Communication

### Unary RPC

Request → Response

### Server Streaming

Request → Multiple Responses

### Client Streaming

Multiple Requests → One Response

### Bidirectional Streaming

Two-way streaming

---

## 4. gRPC Server in ASP.NET Core

* Creating gRPC services
* Configuring ASP.NET Core
* Registering gRPC services
* Generated C# classes
* Dependency Injection
* Service implementation

---

## 5. gRPC Client in .NET

* Creating clients
* Calling services
* Client factory
* Typed clients
* HttpClientFactory integration

---

## 6. Authentication & Security

* HTTPS
* TLS/SSL
* JWT Authentication
* Authorization
* Metadata headers
* Secure service communication

---

## 7. Error Handling & Resilience

* `RpcException`
* Status codes
* Deadlines
* Timeouts
* Cancellation tokens
* Retries

---

## 8. Streaming in Depth

### Server Streaming

* Real-time updates
* Notifications

### Client Streaming

* Uploading data
* Batch processing

### Bidirectional Streaming

* Chat systems
* Real-time communication

---

## 9. Interceptors & Middleware

* Logging interceptors
* Authentication interceptors
* Exception interceptors
* Request/response manipulation

---

## 10. gRPC in Microservices

* Service-to-service communication
* Internal APIs
* API Gateway integration
* Sync communication
* Distributed systems architecture

---

## 11. gRPC-Web & Angular Integration

* Browser limitations
* gRPC-Web
* ASP.NET Core gRPC-Web support
* Angular integration
* Envoy proxy basics

---

## 12. Performance & Optimization

* Compression
* Connection reuse
* Efficient serialization
* Streaming optimization
* Benchmarking REST vs gRPC

---

## 13. Production Topics

* Health checks
* Reflection
* Service discovery
* Load balancing
* Monitoring
* Observability
* Versioning

---

## 14. Docker & Deployment

* Dockerizing gRPC services
* Docker Compose
* Kubernetes basics
* Reverse proxy configuration

---

# Suggested Project Progression

## Beginner Projects

* Calculator Service
* Product Service
* User Service

---

## Intermediate Projects

* Notification System
* File Upload Service
* Chat Application

---

## Advanced Projects

* E-commerce Microservices
* Real-Time Tracking System
* Payment Communication Services

---

# Recommended Learning Order

## Phase 1 — Basics

1. HTTP/2
2. RPC Concepts
3. Protobuf

---

## Phase 2 — Core gRPC

4. Unary RPC
5. Streaming
6. ASP.NET Core Integration

---

## Phase 3 — Production Features

7. Security
8. Error Handling
9. Interceptors

---

## Phase 4 — Microservices

10. Service Communication
11. API Gateway
12. Docker

---

## Phase 5 — Advanced

13. Performance
14. Observability
15. Scaling
16. Deployment
