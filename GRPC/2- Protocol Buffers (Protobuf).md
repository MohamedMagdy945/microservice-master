# 2. Protocol Buffers (Protobuf)

Protocol Buffers (Protobuf) is Google's language-neutral and platform-neutral serialization technology used by gRPC.

It is responsible for:

* Defining communication contracts
* Serializing data into binary format
* Generating strongly typed code
* Enabling fast communication between services

---

# .proto Files

A `.proto` file is the core file used in gRPC applications.

It contains:

* Messages
* Services
* RPC methods
* Enums
* Imports
* Packages

---

## Example

```proto
syntax = "proto3";

package ecommerce;
```

---

# Messages

A `message` defines the structure of data exchanged between client and server.

It is similar to a DTO or model class in .NET.

---

## Example

```proto
message Product {
  int32 id = 1;
  string name = 2;
  double price = 3;
}
```

---

# Scalar Data Types

Protobuf provides built-in scalar types.

| Protobuf Type | C# Equivalent |
| ------------- | ------------- |
| int32         | int           |
| int64         | long          |
| string        | string        |
| bool          | bool          |
| double        | double        |
| float         | float         |
| bytes         | byte[]        |

---

## Example

```proto
message User {
  int32 id = 1;
  string name = 2;
  bool isActive = 3;
}
```

---

# Field Numbers

Every field must have a unique numeric identifier.

---

## Example

```proto
string name = 2;
```

Where:

* `string` → data type
* `name` → field name
* `2` → field number

---

## Rules

* Field numbers must be unique
* Do not change existing field numbers
* Reusing numbers may break compatibility

---

# Repeated Fields

`repeated` is used for arrays and collections.

---

## Example

```proto
message ProductList {
  repeated string products = 1;
}
```

---

## Complex Example

```proto
message Order {
  repeated Product products = 1;
}
```

---

# Enums

Enums define a fixed set of constant values.

---

## Example

```proto
enum OrderStatus {
  PENDING = 0;
  COMPLETED = 1;
  CANCELLED = 2;
}
```

---

## Using Enum

```proto
message Order {
  int32 id = 1;
  OrderStatus status = 2;
}
```

---

# Nested Messages

Messages can contain other messages.

---

## Example

```proto
message Address {
  string city = 1;
  string country = 2;
}

message Customer {
  int32 id = 1;
  string name = 2;
  Address address = 3;
}
```

---

# Imports & Packages

## Packages

Packages organize Protobuf files and prevent naming conflicts.

---

## Example

```proto
package ecommerce.product;
```

---

## Imports

Imports allow reuse of messages from other `.proto` files.

---

## Example

```proto
import "common.proto";
```

---

# Service Definitions

Services define available RPC operations.

---

## Example

```proto
service ProductService {
  rpc GetProduct(ProductRequest) returns (ProductResponse);
}
```

---

# Understanding `message`

A `message` defines a request or response contract.

---

## Example

```proto
message ProductRequest {
  int32 id = 1;
}
```

---

# Understanding `service`

A `service` groups related RPC methods.

---

## Example

```proto
service UserService {
  rpc GetUser(UserRequest) returns (UserResponse);
}
```

---

# Understanding `rpc`

`rpc` defines a remote method callable by the client.

---

## Syntax

```proto
rpc MethodName(RequestType) returns (ResponseType);
```

---

## Example

```proto
rpc GetProduct(ProductRequest) returns (ProductResponse);
```

This means:

* Client sends `ProductRequest`
* Server returns `ProductResponse`

---

# Complete Example

```proto
syntax = "proto3";

package ecommerce;

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

# Summary

In this section you learned:

* `.proto` files
* Messages
* Scalar data types
* Field numbers
* Repeated fields
* Enums
* Nested messages
* Imports & packages
* Service definitions
* `message`
* `service`
* `rpc`
