# NoSQL Databases — Overview

## What is NoSQL?

NoSQL (Not Only SQL) databases are non-relational data storage systems designed to handle large volumes of data, flexible schemas, and distributed architectures. Unlike traditional relational databases, NoSQL systems do not rely on fixed table structures or strict relationships between entities.

They are commonly used in modern applications that require high performance, scalability, and the ability to handle unstructured or semi-structured data.

---

## Key Characteristics

* Schema-less or flexible schema design
* Horizontal scalability (scale out instead of scale up)
* High performance for read/write operations
* Designed for distributed systems
* Support for large-scale data (Big Data)

---

## Types of NoSQL Databases

### 1. Document Databases

Store data in JSON-like documents.

Examples:

* MongoDB
* CouchDB

Use cases:

* Content management systems
* E-commerce catalogs
* User profiles

---

### 2. Key-Value Stores

Store data as simple key-value pairs.

Examples:

* Redis
* DynamoDB

Use cases:

* Caching
* Session storage
* Real-time applications

---

### 3. Column-Family Stores

Store data in columns instead of rows.

Examples:

* Cassandra
* HBase

Use cases:

* Analytics
* Large-scale distributed systems

---

### 4. Graph Databases

Store data as nodes and relationships.

Examples:

* Neo4j

Use cases:

* Social networks
* Recommendation systems
* Fraud detection

---

## NoSQL vs SQL

| Feature       | SQL Databases   | NoSQL Databases      |
| ------------- | --------------- | -------------------- |
| Structure     | Fixed schema    | Flexible schema      |
| Relationships | JOIN operations | Embed / Reference    |
| Scalability   | Vertical        | Horizontal           |
| Performance   | Moderate        | High (specific use)  |
| Transactions  | Strong (ACID)   | Eventual consistency |

---

## Data Modeling in NoSQL

### 1. Embedding (Denormalization)

Data is stored together in a single document.

Example:

```json
{
  "name": "iPhone 15",
  "category": {
    "id": 1,
    "name": "Mobiles"
  }
}
```

Advantages:

* Fast reads (single query)
* No joins required

Disadvantages:

* Data duplication
* Harder updates

---

### 2. Referencing

Data is stored separately and linked by ID.

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
* Slightly slower reads

---

### 3. Hybrid Approach

Combination of embedding and referencing.

Example:

```json
{
  "name": "iPhone 15",
  "categoryId": 1,
  "categoryName": "Mobiles"
}
```

Advantages:

* Fast reads
* Maintains reference integrity

---

## When to Use NoSQL

* Applications with large-scale data
* Real-time systems
* Microservices architecture
* Rapidly evolving data models
* High read/write throughput requirements

---

## When Not to Use NoSQL

* Complex transactions are required
* Strong relationships between data entities
* Heavy use of JOIN operations
* Strict consistency is critical

---

## Conclusion

NoSQL databases provide flexibility, scalability, and performance advantages over traditional relational databases in many modern use cases. However, choosing between SQL and NoSQL depends on the specific requirements of the application.

The key to using NoSQL effectively is understanding data access patterns and designing the schema accordingly, often using embedding, referencing, or a hybrid approach.

---

## License

This document is for educational purposes and can be used or modified freely.
