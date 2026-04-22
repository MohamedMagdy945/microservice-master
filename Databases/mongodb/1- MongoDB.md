# MongoDB — Overview

## What is MongoDB?

MongoDB is a NoSQL document-oriented database that stores data in flexible, JSON-like documents (BSON format). It is designed for scalability, high performance, and ease of development.

Unlike traditional relational databases, MongoDB does not use tables and rows. Instead, it uses collections and documents, allowing developers to store complex and nested data structures.

---

## Key Features

* Document-based storage (BSON format)
* Schema flexibility (no fixed structure required)
* High performance for read and write operations
* Horizontal scalability using sharding
* Built-in replication for high availability
* Rich query language

---

## Core Concepts

### 1. Database

A container for collections.

### 2. Collection

A group of documents (similar to a table in SQL).

### 3. Document

A single record stored in BSON format (similar to a row in SQL).

Example:

```json
{
  "_id": 1,
  "name": "iPhone 15",
  "price": 1000,
  "category": {
    "id": 1,
    "name": "Mobiles"
  }
}
```

---

## Data Modeling

MongoDB supports multiple ways to design data:

### Embedding

Store related data inside a single document.

Advantages:

* Fast reads
* Fewer queries

Disadvantages:

* Data duplication
* Harder updates

---

### Referencing

Store relationships using IDs.

Example:

```json
{
  "name": "iPhone 15",
  "categoryId": 1
}
```

Advantages:

* No duplication
* Easier updates

Disadvantages:

* Requires multiple queries

---

### Hybrid Approach

Combine embedding and referencing for better performance and flexibility.

---

## Indexing

Indexes improve query performance by allowing MongoDB to quickly locate data.

Example:

```js
db.products.createIndex({ name: 1 })
```

Types of indexes:

* Single field
* Compound
* Text
* Geospatial

---

## Aggregation Framework

MongoDB provides a powerful aggregation framework to process and transform data.

Example:

```js
db.products.aggregate([
  { $match: { categoryId: 1 } },
  { $group: { _id: "$categoryId", total: { $sum: "$price" } } }
])
```

---

## Replication

MongoDB uses replica sets to provide high availability.

* Primary node handles writes
* Secondary nodes replicate data
* Automatic failover if primary fails

---

## Sharding

Sharding distributes data across multiple servers to handle large datasets and high traffic.

Benefits:

* Horizontal scaling
* Improved performance
* Load distribution

---

## When to Use MongoDB

* Applications with flexible or evolving schemas
* High-performance APIs
* Real-time analytics
* Content management systems
* E-commerce platforms

---

## When Not to Use MongoDB

* Applications requiring complex joins
* Systems with strict ACID transactions across multiple entities
* Highly relational data models

---

## Conclusion

MongoDB is a powerful and flexible NoSQL database that fits modern application needs, especially when scalability and performance are priorities. Proper schema design is essential to take full advantage of its capabilities.

---

## License

This document is for educational purposes and can be freely used and modified.
