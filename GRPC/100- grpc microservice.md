# 10. gRPC in Microservices

gRPC is widely used in microservices architectures because it provides:

- High performance
- Low latency
- Strong contracts
- Efficient communication
- Streaming support
- Excellent scalability

It is commonly used for internal communication between services.

---

# Why gRPC is Good for Microservices

Microservices require fast and reliable communication.

gRPC solves many common microservice challenges:

- Faster than REST
- Smaller payloads
- Strongly typed contracts
- Better performance under load
- Built-in streaming
- Multi-language support

---

# Service-to-Service Communication

In microservices architecture, services communicate with each other over the network.

gRPC is commonly used for:

- Internal service calls
- Data exchange
- Event communication
- Real-time systems

---

## Example Architecture

```text
Order Service
      ↓
gRPC Call
      ↓
Product Service
      ↓
Database
```

---

# Example Scenario

When creating an order:

1. Order Service calls Product Service
2. Product Service validates stock
3. Product Service returns product details
4. Order Service completes the order

---

# .proto Contract Example

```proto
syntax = "proto3";

service ProductService {
  rpc GetProduct(ProductRequest)
      returns (ProductResponse);
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

# ASP.NET Core Service Example

```csharp
public class ProductServiceImpl
    : ProductService.ProductServiceBase
{
    public override Task<ProductResponse> GetProduct(
        ProductRequest request,
        ServerCallContext context)
    {
        return Task.FromResult(
            new ProductResponse
            {
                Id = request.Id,
                Name = "Laptop",
                Price = 2000
            });
    }
}
```

---

# Calling Another Microservice

```csharp
public class OrderService
{
    private readonly ProductService.ProductServiceClient _client;

    public OrderService(
        ProductService.ProductServiceClient client)
    {
        _client = client;
    }

    public async Task CreateOrder()
    {
        var product = await _client.GetProductAsync(
            new ProductRequest { Id = 1 });
    }
}
```

---

# Internal APIs

gRPC is commonly used for internal APIs inside organizations.

Internal APIs are APIs used only between internal services.

---

# Why gRPC for Internal APIs

Benefits:

- High speed
- Binary communication
- Shared contracts
- Strong typing
- Better performance

---

# REST vs gRPC for Internal APIs

| Feature | REST | gRPC |
|---|---|---|
| Data Format | JSON | Binary |
| Performance | Moderate | High |
| Typing | Flexible | Strongly Typed |
| Streaming | Limited | Built-in |
| Payload Size | Larger | Smaller |

---

# API Gateway Integration

In microservices systems, external clients usually do not communicate directly with gRPC services.

Instead, requests pass through an API Gateway.

---

# API Gateway Responsibilities

An API Gateway handles:

- Authentication
- Authorization
- Routing
- Rate limiting
- Logging
- Load balancing

---

# Architecture Example

```text
Client
   ↓
API Gateway
   ↓
gRPC Services
   ↓
Databases
```

---

# Common API Gateways

Popular gateways used with gRPC:

- YARP
- Envoy
- NGINX
- Ocelot

---

# ASP.NET Core YARP Example

```bash
dotnet add package Yarp.ReverseProxy
```

---

# Sync Communication

gRPC is primarily used for synchronous communication.

This means:

- Client waits for server response
- Request/response pattern
- Immediate feedback

---

# Example

```text
Order Service
      ↓
Request Product Data
      ↓
Product Service
      ↓
Immediate Response
```

---

# Benefits of Sync Communication

- Simpler implementation
- Immediate validation
- Easier debugging
- Strong consistency

---

# Challenges of Sync Communication

- Tight coupling
- Service dependency
- Cascading failures
- Latency propagation

---

# Best Practices

To reduce issues:

- Use retries carefully
- Implement timeouts
- Use circuit breakers
- Add caching
- Monitor latency

---

# Distributed Systems Architecture

Microservices are distributed systems.

A distributed system consists of multiple independent services communicating over the network.

---

# Characteristics of Distributed Systems

- Independent services
- Network communication
- Scalability
- Fault tolerance
- Decentralized deployment

---

# gRPC in Distributed Systems

gRPC helps distributed systems by providing:

- Fast communication
- Efficient serialization
- Streaming
- Contract-first development
- Multi-language interoperability

---

# Typical Microservices Architecture

```text
API Gateway
     ↓
--------------------------------
|        |        |            |
User   Order   Product    Payment
Svc     Svc      Svc        Svc
```

---

# Challenges in Distributed Systems

Common challenges:

- Network failures
- Latency
- Data consistency
- Service discovery
- Monitoring
- Distributed tracing

---

# Important Patterns

## Retry Pattern

Retries temporary failures.

---

## Circuit Breaker

Stops repeated failing requests.

---

## Timeout Pattern

Prevents hanging requests.

---

## Service Discovery

Finds available service instances.

---

# Service Discovery Tools

Common tools:

- Consul
- Kubernetes DNS
- Eureka

---

# Observability

Microservices need monitoring and tracing.

Important tools:

- OpenTelemetry
- Prometheus
- Grafana
- Jaeger

---

# Security in Microservices

Secure communication using:

- HTTPS
- TLS
- JWT
- API Gateway authentication

---

# gRPC + Kubernetes

gRPC works very well with Kubernetes because both support:

- Scalability
- Load balancing
- Service discovery
- Containerized deployments

---

# Summary

In this section you learned:

- gRPC in microservices
- Service-to-service communication
- Internal APIs
- API Gateway integration
- Sync communication
- Distributed systems architecture
- Microservices patterns
- Scalability and resiliency concepts