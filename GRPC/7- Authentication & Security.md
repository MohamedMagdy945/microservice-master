# 6. Authentication & Security

Security is a critical part of gRPC applications.

gRPC provides secure communication using:

* HTTPS
* TLS/SSL encryption
* Authentication
* Authorization
* Metadata headers
* Secure service-to-service communication

ASP.NET Core integrates these security features directly with gRPC.

---

# HTTPS

gRPC communication should use HTTPS in production environments.

HTTPS encrypts data transmitted between client and server.

---

# Why HTTPS is Important

HTTPS protects against:

* Data interception
* Man-in-the-middle attacks
* Credential theft
* Data tampering

---

# gRPC and HTTPS

gRPC uses HTTP/2.

Most browsers and platforms require HTTP/2 over HTTPS.

---

# Configure HTTPS in ASP.NET Core

ASP.NET Core enables HTTPS by default.

Example:

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddGrpc();

var app = builder.Build();

app.UseHttpsRedirection();

app.MapGrpcService<ProductService>();

app.Run();
```

---

# TLS/SSL

TLS (Transport Layer Security) encrypts communication between client and server.

SSL is the older version replaced by TLS.

---

# TLS in gRPC

TLS provides:

* Encryption
* Authentication
* Secure transport
* Data integrity

---

# How TLS Works

```text
Client
   ↓
TLS Handshake
   ↓
Encrypted Connection
   ↓
gRPC Communication
```

---

# Configure TLS Certificate

Example in `appsettings.json`:

```json
{
  "Kestrel": {
    "Endpoints": {
      "Https": {
        "Url": "https://localhost:5001"
      }
    }
  }
}
```

---

# JWT Authentication

JWT (JSON Web Token) is commonly used for authenticating gRPC clients.

The client sends a token with every request.

The server validates the token before processing requests.

---

# Install JWT Package

```bash
dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
```

---

# Configure JWT Authentication

Inside `Program.cs`:

```csharp
builder.Services
    .AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.Authority = "https://localhost:5001";
        options.Audience = "grpc-api";
    });
```

---

# Enable Authentication Middleware

```csharp
app.UseAuthentication();
app.UseAuthorization();
```

---

# Protect gRPC Services

```csharp
[Authorize]
public class ProductService : Product.ProductBase
{
}
```

---

# Sending JWT Token from Client

```csharp
var headers = new Metadata
{
    { "Authorization", "Bearer token_here" }
};

var response = await client.GetProductAsync(
    new ProductRequest { Id = 1 },
    headers);
```

---

# Authorization

Authorization controls access to specific resources and operations.

---

# Role-Based Authorization

```csharp
[Authorize(Roles = "Admin")]
public class AdminService : Admin.AdminBase
{
}
```

---

# Policy-Based Authorization

```csharp
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("AdminOnly", policy =>
    {
        policy.RequireRole("Admin");
    });
});
```

---

# Use Policy

```csharp
[Authorize(Policy = "AdminOnly")]
public class ProductService : Product.ProductBase
{
}
```

---

# Metadata Headers

gRPC uses metadata headers to send additional information.

Metadata works similarly to HTTP headers.

---

# Common Uses of Metadata

* Authentication tokens
* Correlation IDs
* Tenant IDs
* Custom request data
* Localization

---

# Sending Metadata from Client

```csharp
var headers = new Metadata
{
    { "Authorization", "Bearer token" },
    { "tenant-id", "tenant_1" }
};
```

---

# Reading Metadata on Server

```csharp
public override Task<ProductResponse> GetProduct(
    ProductRequest request,
    ServerCallContext context)
{
    var token = context.RequestHeaders
        .FirstOrDefault(x => x.Key == "authorization");

    return Task.FromResult(new ProductResponse());
}
```

---

# Secure Service Communication

In microservices architecture, gRPC services often communicate internally.

Secure service communication is essential.

---

# Best Practices

## Always Use HTTPS

Never expose production gRPC services over unsecured HTTP.

---

## Use JWT Authentication

Authenticate all sensitive operations.

---

## Validate Tokens

Always validate:

* Issuer
* Audience
* Expiration
* Signature

---

## Use Authorization Policies

Restrict access based on:

* Roles
* Permissions
* Claims

---

## Secure Internal Services

Use:

* Internal TLS
* API gateways
* Service mesh
* Network isolation

---

# Example Request Flow

```text
Client
   ↓
HTTPS + TLS
   ↓
JWT Token Sent
   ↓
gRPC Server
   ↓
Authentication
   ↓
Authorization
   ↓
Business Logic
   ↓
Secure Response
```

---

# Authentication vs Authorization

| Feature       | Authentication  | Authorization        |
| ------------- | --------------- | -------------------- |
| Purpose       | Verify identity | Control access       |
| Example       | JWT Login       | Admin access         |
| Happens First | Yes             | After authentication |

---

# Summary

In this section you learned:

* HTTPS in gRPC
* TLS/SSL encryption
* JWT Authentication
* Authorization
* Metadata headers
* Secure service communication
* Protecting gRPC endpoints
* Sending secure client requests
* Security best practices
