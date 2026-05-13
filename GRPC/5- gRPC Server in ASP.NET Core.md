# 4. gRPC Server in ASP.NET Core

ASP.NET Core provides built-in support for building high-performance gRPC services.

A gRPC server in ASP.NET Core is responsible for:

* Hosting gRPC services
* Receiving client requests
* Processing RPC calls
* Returning responses
* Managing streaming communication

---

# Creating gRPC Services

A gRPC service is created using:

1. A `.proto` contract file
2. Generated C# classes
3. Service implementation class

---

## Step 1: Create `.proto` File

Example:

```proto
syntax = "proto3";

option csharp_namespace = "GrpcDemo";

package product;

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

## Step 2: Add Protobuf Configuration

Inside `.csproj`:

```xml
<ItemGroup>
  <Protobuf Include="Protos\product.proto" GrpcServices="Server" />
</ItemGroup>
```

---

## Step 3: Implement Service

```csharp
using Grpc.Core;

public class ProductServiceImpl : ProductService.ProductServiceBase
{
    public override Task<ProductResponse> GetProduct(
        ProductRequest request,
        ServerCallContext context)
    {
        var response = new ProductResponse
        {
            Id = request.Id,
            Name = "Laptop",
            Price = 1500
        };

        return Task.FromResult(response);
    }
}
```

---

# Configuring ASP.NET Core

ASP.NET Core must be configured to support gRPC.

---

## Install Required Package

```bash
dotnet add package Grpc.AspNetCore
```

---

## Configure `Program.cs`

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddGrpc();

var app = builder.Build();

app.MapGrpcService<ProductServiceImpl>();

app.Run();
```

---

## Important Notes

* gRPC uses HTTP/2
* HTTPS is recommended
* ASP.NET Core automatically handles routing
* Services are mapped using `MapGrpcService()`

---

# Registering gRPC Services

gRPC services must be registered in the dependency injection container.

---

## Register gRPC Framework

```csharp
builder.Services.AddGrpc();
```

This enables:

* gRPC middleware
* Serialization
* Service activation
* Streaming support

---

## Map gRPC Services

```csharp
app.MapGrpcService<ProductServiceImpl>();
```

This exposes the service endpoint.

---

# Generated C# Classes

When the project builds:

* Protobuf compiler generates C# classes automatically
* Messages become DTO classes
* Services become abstract base classes

---

# Generated Message Class

From:

```proto
message ProductRequest {
  int32 id = 1;
}
```

Generated:

```csharp
public sealed partial class ProductRequest
{
    public int Id { get; set; }
}
```

---

# Generated Service Base Class

From:

```proto
service ProductService {
  rpc GetProduct(ProductRequest)
      returns (ProductResponse);
}
```

Generated:

```csharp
public abstract class ProductServiceBase
{
    public virtual Task<ProductResponse> GetProduct(
        ProductRequest request,
        ServerCallContext context)
    {
    }
}
```

---

# Dependency Injection

ASP.NET Core gRPC fully supports Dependency Injection (DI).

You can inject:

* Services
* Repositories
* DbContext
* Loggers
* Configuration

---

# Register Services

```csharp
builder.Services.AddScoped<IProductRepository, ProductRepository>();
```

---

# Inject into gRPC Service

```csharp
public class ProductServiceImpl : ProductService.ProductServiceBase
{
    private readonly IProductRepository _repository;

    public ProductServiceImpl(IProductRepository repository)
    {
        _repository = repository;
    }
}
```

---

# Service Implementation

The service implementation contains business logic.

It inherits from the generated base class.

---

# Example

```csharp
using Grpc.Core;

public class ProductServiceImpl : ProductService.ProductServiceBase
{
    public override async Task<ProductResponse> GetProduct(
        ProductRequest request,
        ServerCallContext context)
    {
        return new ProductResponse
        {
            Id = request.Id,
            Name = "Gaming Laptop",
            Price = 2500
        };
    }
}
```

---

# Understanding `ServerCallContext`

`ServerCallContext` contains metadata about the current RPC call.

It provides:

* Request headers
* Cancellation token
* Authentication information
* Deadlines
* Response trailers

---

# Example

```csharp
context.CancellationToken
```

Used for request cancellation.

---

# Project Structure Example

```text
GrpcServer/
│
├── Protos/
│   └── product.proto
│
├── Services/
│   └── ProductServiceImpl.cs
│
├── Program.cs
│
└── GrpcServer.csproj
```

---

# Request Flow

```text
Client Request
      ↓
ASP.NET Core gRPC Middleware
      ↓
Generated Service Base
      ↓
Service Implementation
      ↓
Business Logic
      ↓
Response Returned
```

---

# Summary

In this section you learned:

* Creating gRPC services
* Configuring ASP.NET Core for gRPC
* Registering gRPC services
* Generated C# classes
* Dependency Injection in gRPC
* Service implementation
* Request processing flow
* `ServerCallContext`
