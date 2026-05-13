# 5. gRPC Client in .NET

A gRPC client in .NET is responsible for communicating with gRPC servers.

The client:

* Connects to the gRPC server
* Sends requests
* Receives responses
* Handles streaming communication
* Uses generated strongly typed classes

ASP.NET Core provides built-in support for creating gRPC clients.

---

# Creating Clients

A gRPC client is generated automatically from `.proto` files.

The generated client allows calling remote RPC methods like local methods.

---

# Step 1: Add Required Packages

```bash
dotnet add package Grpc.Net.Client
dotnet add package Google.Protobuf
dotnet add package Grpc.Tools
```

---

# Step 2: Add `.proto` File

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

# Step 3: Configure `.csproj`

```xml
<ItemGroup>
  <Protobuf Include="Protos\product.proto" GrpcServices="Client" />
</ItemGroup>
```

---

# Step 4: Create gRPC Channel

```csharp
using Grpc.Net.Client;

var channel = GrpcChannel.ForAddress("https://localhost:5001");
```

The channel represents the connection to the gRPC server.

---

# Step 5: Create Client

```csharp
var client = new ProductService.ProductServiceClient(channel);
```

This generated client allows calling server methods.

---

# Calling Services

Clients call RPC methods asynchronously.

---

# Unary RPC Example

```csharp
var response = await client.GetProductAsync(
    new ProductRequest
    {
        Id = 1
    });
```

---

# Access Response

```csharp
Console.WriteLine(response.Name);
Console.WriteLine(response.Price);
```

---

# Request Flow

```text
.NET Client
     ↓
Generated gRPC Client
     ↓
HTTP/2 Request
     ↓
gRPC Server
     ↓
Response Returned
```

---

# Client Factory

ASP.NET Core provides gRPC client factory support.

The factory simplifies:

* Client creation
* Dependency Injection
* Lifetime management
* Configuration
* Logging
* Retry policies

---

# Install Package

```bash
dotnet add package Grpc.Net.ClientFactory
```

---

# Register gRPC Client

Inside `Program.cs`:

```csharp
builder.Services.AddGrpcClient<ProductService.ProductServiceClient>(o =>
{
    o.Address = new Uri("https://localhost:5001");
});
```

---

# Benefits of Client Factory

* Centralized configuration
* Automatic dependency injection
* Better resource management
* Easier testing
* HttpClient integration

---

# Typed Clients

Typed clients are strongly typed generated gRPC clients.

They can be injected directly using Dependency Injection.

---

# Example

```csharp
public class ProductManager
{
    private readonly ProductService.ProductServiceClient _client;

    public ProductManager(
        ProductService.ProductServiceClient client)
    {
        _client = client;
    }

    public async Task GetProductAsync()
    {
        var response = await _client.GetProductAsync(
            new ProductRequest { Id = 1 });
    }
}
```

---

# Advantages of Typed Clients

* Strong typing
* Cleaner code
* Easier testing
* Built-in DI support
* Better maintainability

---

# HttpClientFactory Integration

gRPC clients integrate with `HttpClientFactory`.

This provides:

* Connection pooling
* DNS refresh handling
* Resiliency
* Retry support
* Logging
* Better performance

---

# Configure HttpClient

```csharp
builder.Services
    .AddGrpcClient<ProductService.ProductServiceClient>(o =>
    {
        o.Address = new Uri("https://localhost:5001");
    })
    .ConfigurePrimaryHttpMessageHandler(() =>
    {
        return new HttpClientHandler();
    });
```

---

# Add Custom Headers

```csharp
builder.Services
    .AddGrpcClient<ProductService.ProductServiceClient>(o =>
    {
        o.Address = new Uri("https://localhost:5001");
    })
    .AddCallCredentials((context, metadata) =>
    {
        metadata.Add("Authorization", "Bearer token");
        return Task.CompletedTask;
    });
```

---

# Using gRPC Client in ASP.NET Core

```csharp
app.MapGet("/products", async (
    ProductService.ProductServiceClient client) =>
{
    var product = await client.GetProductAsync(
        new ProductRequest { Id = 1 });

    return product;
});
```

---

# Streaming Client Example

## Server Streaming

```csharp
using var call = client.StreamProducts(new Empty());

await foreach (var item in call.ResponseStream.ReadAllAsync())
{
    Console.WriteLine(item.Name);
}
```

---

# Error Handling

gRPC clients throw `RpcException` when errors occur.

---

# Example

```csharp
try
{
    var response = await client.GetProductAsync(
        new ProductRequest { Id = 1 });
}
catch (RpcException ex)
{
    Console.WriteLine(ex.Status.Detail);
}
```

---

# Project Structure Example

```text
GrpcClient/
│
├── Protos/
│   └── product.proto
│
├── Services/
│   └── ProductManager.cs
│
├── Program.cs
│
└── GrpcClient.csproj
```

---

# Summary

In this section you learned:

* Creating gRPC clients
* Calling services
* Using gRPC client factory
* Typed clients
* HttpClientFactory integration
* Dependency Injection support
* Streaming client calls
* Error handling
* Request flow in gRPC clients
