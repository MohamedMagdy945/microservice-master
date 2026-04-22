# MongoDB Commands — Quick Reference Guide

## Introduction

This document provides a quick reference for commonly used MongoDB commands. It is designed to help developers perform basic database operations such as creating databases, inserting data, querying, updating, and deleting documents.

---

## Show Databases

```js
show dbs
```

Displays all databases.

---

## Create / Switch Database

```js
use myDatabase
```

Creates a new database if it does not exist, or switches to it if it does.

---

## Show Collections

```js
show collections
```

Lists all collections in the current database.

---

## Create Collection

```js
db.createCollection("products")
```

Creates a new collection named "products".

---

## Insert Documents

### Insert One

```js
db.products.insertOne({
  name: "iPhone 15",
  price: 1000,
  category: "Mobiles"
})
```

### Insert Many

```js
db.products.insertMany([
  { name: "iPhone 15", price: 1000 },
  { name: "Samsung S24", price: 900 }
])
```

---

## Find Documents

### Find All

```js
db.products.find()
```

### Find with Condition

```js
db.products.find({ category: "Mobiles" })
```

### Pretty Print

```js
db.products.find().pretty()
```

---

## Find One Document

```js
db.products.findOne({ name: "iPhone 15" })
```

---

## Update Documents

### Update One

```js
db.products.updateOne(
  { name: "iPhone 15" },
  { $set: { price: 1100 } }
)
```

### Update Many

```js
db.products.updateMany(
  { category: "Mobiles" },
  { $set: { discount: true } }
)
```

---

## Delete Documents

### Delete One

```js
db.products.deleteOne({ name: "iPhone 15" })
```

### Delete Many

```js
db.products.deleteMany({ category: "Mobiles" })
```

---

## Count Documents

```js
db.products.countDocuments()
```

---

## Sorting

```js
db.products.find().sort({ price: 1 })   // Ascending
db.products.find().sort({ price: -1 })  // Descending
```

---

## Limit Results

```js
db.products.find().limit(5)
```

---

## Skip Results

```js
db.products.find().skip(5)
```

---

## Indexing

### Create Index

```js
db.products.createIndex({ name: 1 })
```

### View Indexes

```js
db.products.getIndexes()
```

---

## Aggregation

```js
db.products.aggregate([
  { $match: { category: "Mobiles" } },
  { $group: { _id: "$category", total: { $sum: "$price" } } }
])
```

---

## Drop Collection

```js
db.products.drop()
```

---

## Drop Database

```js
db.dropDatabase()
```

---

## Useful Tips

* MongoDB stores data in BSON format (binary JSON).
* Collections do not require a fixed schema.
* Use indexes to improve performance.
* Avoid heavy use of aggregation in high-load APIs.

---

## Conclusion

These commands cover the most common operations needed to work with MongoDB. Mastering them will allow you to build and manage applications efficiently using a NoSQL database.

---

## License

This document is free to use and modify for educational purposes.
