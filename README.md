# MongoDB Guide

A practical MongoDB setup with guides for running MongoDB in Docker, performing CRUD operations, and modeling real-world data relationships. Includes a full POS/inventory example and advanced relationship patterns.

## Table of Contents

- [Key MongoDB Concepts](#key-mongodb-concepts)
- [Quick Links](#quick-links)
- [Prerequisites](#prerequisites)
- [Running MongoDB](#running-mongodb)
- [Connecting to MongoDB](#connecting-to-mongodb)
- [Some Useful Commands](#some-useful-commands)
- [Example Queries (CRUD & Operators)](#example-queries-crud--operators)
  - [Insert Documents](#insert-documents)
  - [Find (Read) Documents](#find-read-documents)
  - [Update Documents](#update-documents)
  - [Delete Documents](#delete-documents)
  - [Common Query and Update Operators](#common-query-and-update-operators)
  - [Comparison and Logical Operators Examples](#comparison-and-logical-operators-examples)
  - [Advanced Query Features](#advanced-query-features)
    - [Text Search](#text-search)
    - [Regular Expressions](#regular-expressions)
    - [Date/Time Operations](#datetime-operations)
    - [Bulk Operations](#bulk-operations)
    - [Count and Distinct](#count-and-distinct)
    - [Skip and Limit for Pagination](#skip-and-limit-for-pagination)
    - [Explain Plan](#explain-plan)
    - [Why Use .explain()?](#why-use-explain)
    - [Explain Modes](#explain-modes)
    - [Practical Examples](#practical-examples)
    - [Key Metrics to Look For](#key-metrics-to-look-for)
- [Indexing](#indexing)
  - [Creating Indexes](#creating-indexes)
  - [Index Types and Examples](#index-types-and-examples)
  - [Checking Index Usage](#checking-index-usage)
  - [Index Best Practices](#index-best-practices)
- [Capped Collections](#capped-collections)
  - [Creating Capped Collections](#creating-capped-collections)
  - [Working with Capped Collections](#working-with-capped-collections)
  - [Use Cases](#use-cases)
- [Schema Validation](#schema-validation)
  - [Creating Collections with Validation](#creating-collections-with-validation)
  - [Adding Validation to Existing Collections](#adding-validation-to-existing-collections)
  - [Validation Examples](#validation-examples)
  - [Validation Options](#validation-options)
  - [Testing Validation](#testing-validation)
- [Aggregation](#aggregation)
  - [Common Aggregation Operators](#common-aggregation-operators)
  - [Aggregation Example](#aggregation-example)
  - [Advanced Aggregation Operators](#advanced-aggregation-operators)
    - [$merge - Write to Existing Collection](#merge---write-to-existing-collection)
    - [$collStats - Collection Statistics](#collstats---collection-statistics)
    - [$queryHistory - Query History (Atlas Data Federation)](#queryhistory---query-history-atlas-data-federation)
    - [$sql - SQL Queries (Atlas Data Federation)](#sql---sql-queries-atlas-data-federation)
- [Contribution Guidelines](#contribution-guidelines)
- [License](#license)
- [Contact/Support](#contactsupport)

## Key MongoDB Concepts

- **Collection:** A group of MongoDB documents, similar to a table in relational databases.
- **Document:** A record in a collection, stored as a JSON-like object (BSON). Each document has a unique `_id` field.
- **Projection:** Selecting which fields to include or exclude in query results (e.g., `{ name: 1, _id: 0 }`).
- **Aggregation:** A framework for processing data and returning computed results, often used for reporting and analytics (e.g., using `$group`, `$match`, `$lookup`).
- **Index:** A data structure that improves the speed of data retrieval operations on a collection.
- **Query:** A request to find documents in a collection that match certain criteria.
- **Operator:** Special keywords used in queries and updates (e.g., `$gt`, `$set`, `$inc`).
- **Schema:** The structure of documents in a collection. MongoDB is schema-less, but you can enforce structure in your application or with validation rules.

## Quick Links

- [POS/Inventory Example](./pos.md)
- [Modeling Relationships in MongoDB](./relationship.md)

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/) installed
- [Docker Compose](https://docs.docker.com/compose/install/) installed
- [mongosh](https://www.mongodb.com/try/download/shell) (MongoDB Shell) installed on your host

## Running MongoDB

1. Start MongoDB with Docker Compose:

   ```sh
   docker compose up -d
   ```

   This will start a MongoDB container with the configuration from `docker-compose.yml`.

2. To stop and remove the container and data volume:
   ```sh
   docker compose down -v
   ```

## Connecting to MongoDB

- Connect as the root user (created by `MONGO_INITDB_ROOT_USERNAME` and `MONGO_INITDB_ROOT_PASSWORD`):

  ```sh
  mongosh "mongodb://admin:admin@localhost:27017"
  # or explicitly to the admin database:
  mongosh "mongodb://admin:admin@localhost:27017/admin"
  ```

- Once connected, you can switch to your application database:
  ```js
  use mongo_db
  ```

**Notes**

- The database (`mongo_db`) will only appear in `show dbs` after it contains at least one document.
- The root user is created in the `admin` database. You can create additional users as needed.
- To reset credentials, you must remove the data volume: `docker compose down -v` and then `docker compose up -d`.

## Some Useful Commands

```js
// Syntax:
// show dbs
show dbs

// Syntax:
// show collections
show collections

// Syntax:
// .drop()
db.users.drop()

// Syntax:
// .dropDatabase()
db.dropDatabase()
```

## Example Queries (CRUD & Operators)

### Insert Documents

```js
// Syntax:
// .insertOne(document)
db.users.insertOne({ name: 'Alice', age: 30, city: 'London' });

// Syntax:
// .insertMany([document, ...])
db.users.insertMany([
  { name: 'Bob', age: 25, city: 'Paris' },
  { name: 'Charlie', age: 35, city: 'Berlin' },
]);
```

### Find (Read) Documents

```js
// Syntax:
// .find(query, projection)
db.users.find();

// Syntax:
// .findOne(query)
db.users.findOne({ name: 'Alice' });

// Syntax:
// .find(query)
db.users.find({ age: { $gt: 25 } }); // age > 25

// Syntax:
// .find(query, projection)
db.users.find({}, { name: 1, city: 1, _id: 0 });

// Syntax:
// .find().sort(sort).limit(n)
db.users.find().sort({ age: -1 }).limit(2);
// Note: Same as: db.users.find().sort({ age: -1 }).limit(2).toArray()
```

### Update Documents

```js
// Syntax:
// .updateOne(filter, update)
db.users.updateOne({ name: 'Alice' }, { $set: { city: 'New York' } });

// Syntax:
// .updateMany(filter, update)
db.users.updateMany({ city: 'Paris' }, { $inc: { age: 1 } });

// Syntax:
// .replaceOne(filter, replacement)
db.users.replaceOne(
  { name: 'Charlie' },
  { name: 'Charlie', age: 36, city: 'Munich' },
);
// Note: replaceOne replaces the entire document, while updateOne only modifies specified fields

// Syntax:
// .updateOne(filter, { $unset: { field: "" } })
db.users.updateOne({ name: 'Alice' }, { $unset: { city: '' } });

// Syntax:
// .updateOne(filter, { $rename: { oldField: "newField" } })
db.users.updateOne({ name: 'Alice' }, { $rename: { name: 'fullName' } });

// Syntax:
// .updateOne(filter, { $set: { field: value } })
db.users.updateOne({ fullName: 'Alice' }, { $set: { country: 'UK' } });

// Syntax:
// .updateOne(filter, { $inc: { field: amount } })
db.users.updateOne({ fullName: 'Alice' }, { $inc: { age: 1 } });

// Syntax:
// .updateOne(filter, { $mul: { field: factor } })
db.users.updateOne({ fullName: 'Alice' }, { $mul: { age: 2 } });

// Syntax:
// .updateOne(filter, { $min: { field: value } })
db.users.updateOne({ fullName: 'Alice' }, { $min: { age: 18 } });

// Syntax:
// .updateOne(filter, { $max: { field: value } })
db.users.updateOne({ fullName: 'Alice' }, { $max: { age: 65 } });

// Syntax:
// .updateOne(filter, { $currentDate: { field: true } })
db.users.updateOne(
  { fullName: 'Alice' },
  { $currentDate: { lastModified: true } },
);

// Syntax:
// .updateOne(filter, { $push: { arrayField: value } })
db.users.updateOne({ fullName: 'Alice' }, { $push: { tags: 'new' } });

// Syntax:
// .updateOne(filter, { $addToSet: { arrayField: value } })
db.users.updateOne({ fullName: 'Alice' }, { $addToSet: { tags: 'unique' } });

// Syntax:
// .updateOne(filter, { $pull: { arrayField: value } })
db.users.updateOne({ fullName: 'Alice' }, { $pull: { tags: 'old' } });

// Syntax:
// .updateOne(filter, { $pop: { arrayField: 1 or -1 } })
db.users.updateOne({ fullName: 'Alice' }, { $pop: { tags: 1 } }); // Remove last element
db.users.updateOne({ fullName: 'Alice' }, { $pop: { tags: -1 } }); // Remove first element

// Syntax:
// .updateOne(filter, { $pop: { arrayField: 1 or -1 } })
db.users.updateOne({ fullName: 'Alice' }, { $pop: { tags: 1 } });
```

### Delete Documents

```js
// Syntax:
// .deleteOne(filter)
db.users.deleteOne({ name: 'Bob' });

// Syntax:
// .deleteMany(filter)
db.users.deleteMany({ age: { $lt: 30 } });
// Note: Same as: db.users.remove({ age: { $lt: 30 } }, { justOne: false }) (deprecated)
```

### Common Query and Update Operators

```js
// Comparison Operators
// $eq - equal (same as: db.users.find({ age: 30 }))
db.users.find({ age: { $eq: 30 } });

// $ne - not equal
db.users.find({ city: { $ne: 'London' } });

// $gt - greater than
db.users.find({ age: { $gt: 25 } });

// $gte - greater than or equal
db.users.find({ age: { $gte: 25 } });

// $lt - less than
db.users.find({ age: { $lt: 30 } });

// $lte - less than or equal
db.users.find({ age: { $lte: 30 } });

// $in - in array (more efficient than multiple $or conditions)
db.users.find({ city: { $in: ['London', 'Paris', 'Berlin'] } });

// $nin - not in array
db.users.find({ city: { $nin: ['London', 'Paris'] } });

// Logical Operators
// $and - and (usually implicit, but explicit for complex conditions)
db.users.find({ $and: [{ age: { $gt: 25 } }, { city: 'London' }] });
// Note: Same as: db.users.find({ age: { $gt: 25 }, city: 'London' })

// $or - or
db.users.find({ $or: [{ city: 'London' }, { city: 'Paris' }] });

// $not - not
db.users.find({ age: { $not: { $gt: 30 } } });
// Note: Same as: db.users.find({ age: { $lte: 30 } })

// $nor - nor (neither condition is true)
db.users.find({ $nor: [{ age: { $lt: 25 } }, { city: 'Paris' }] });
// Note: Returns users who are NOT under 25 AND NOT from Paris

// Element Operators
// $exists - field exists
db.users.find({ phone: { $exists: true } });

// $type - field type
db.users.find({ age: { $type: 'int' } });

// Array Operators
// $all - all elements match (order doesn't matter)
db.users.find({ tags: { $all: ['red', 'blue'] } });
// Note: Will match ['red', 'blue', 'green'] but not ['red', 'green']

// $size - array size
db.users.find({ tags: { $size: 2 } });

// $elemMatch - element match (for complex array element conditions)
db.users.find({ scores: { $elemMatch: { $gt: 80, $lt: 90 } } });
// Note: Finds array elements where score > 80 AND < 90
```

### Update Operators Reference

```js
// Field Update Operators
// $set - Set a field value (most common update operator)
db.users.updateOne({ name: 'Alice' }, { $set: { city: 'New York' } });

// $unset - Remove a field (sets field to undefined, then removes it)
db.users.updateOne({ name: 'Alice' }, { $unset: { city: '' } });
// Note: Empty string "" is required for $unset, but the value doesn't matter

// $rename - Rename a field (preserves value and type)
db.users.updateOne({ name: 'Alice' }, { $rename: { name: 'fullName' } });
// Note: More efficient than $unset + $set for renaming

// Numeric Update Operators
// $inc - Increment a field (can use negative values to decrement)
db.users.updateOne({ name: 'Alice' }, { $inc: { age: 1 } });
db.users.updateOne({ name: 'Alice' }, { $inc: { age: -1 } }); // Decrement

// $mul - Multiply a field (can use fractions for division)
db.users.updateOne({ name: 'Alice' }, { $mul: { age: 2 } });
db.users.updateOne({ name: 'Alice' }, { $mul: { age: 0.5 } }); // Divide by 2

// $min - Set to minimum value (only updates if current value is greater)
db.users.updateOne({ name: 'Alice' }, { $min: { age: 18 } });
// Note: Only updates if current age > 18, otherwise no change

// $max - Set to maximum value (only updates if current value is less)
db.users.updateOne({ name: 'Alice' }, { $max: { age: 65 } });
// Note: Only updates if current age < 65, otherwise no change

// Date Update Operators
// $currentDate - Set to current date (can use Date or timestamp)
db.users.updateOne({ name: 'Alice' }, { $currentDate: { lastModified: true } });
// Note: true = Date object, { $type: "timestamp" } = timestamp

// Array Update Operators
// $push - Add value to array (always adds, even if duplicate)
db.users.updateOne({ name: 'Alice' }, { $push: { tags: 'new' } });

// $addToSet - Add value to array if not present (prevents duplicates)
db.users.updateOne({ name: 'Alice' }, { $addToSet: { tags: 'unique' } });
// Note: Better than $push when you want to avoid duplicates

// $pull - Remove value from array (removes all matching elements)
db.users.updateOne({ name: 'Alice' }, { $pull: { tags: 'old' } });

// $pop - Remove first (-1) or last (1) element from array
db.users.updateOne({ name: 'Alice' }, { $pop: { tags: 1 } }); // Remove last element
db.users.updateOne({ name: 'Alice' }, { $pop: { tags: -1 } }); // Remove first element
// Note: $pop is more efficient than $pull for removing by position
```

### Comparison and Logical Operators Examples

```js
// Range queries
db.users.find({ age: { $gte: 25, $lte: 35 } }); // age between 25 and 35
db.users.find({ age: { $gt: 20, $lt: 40 } }); // age between 20 and 40 (exclusive)

// Multiple conditions with AND (implicit)
db.users.find({ age: { $gte: 25 }, city: 'London' }); // age >= 25 AND city is London

// Multiple conditions with AND (explicit)
db.users.find({ $and: [{ age: { $gte: 25 } }, { city: 'London' }] }); // age >= 25 AND city is London

// Multiple conditions with OR
db.users.find({ $or: [{ age: { $lt: 25 } }, { city: 'Paris' }] }); // age < 25 OR city is Paris

// Multiple conditions with NOR (neither condition is true)
db.users.find({ $nor: [{ age: { $lt: 25 } }, { city: 'Paris' }] }); // age is NOT < 25 AND city is NOT Paris

// Complex logical combinations
db.users.find({
  $and: [
    { age: { $gte: 18 } },
    { $or: [{ city: 'London' }, { city: 'Paris' }] },
  ],
}); // age >= 18 AND (city is London OR Paris)

// This requires explicit $and because we're using the same field twice
// prettier-ignore
db.users.find({
  $and: [
    { age: { $gte: 18 } },
    { age: { $lte: 65 } }
  ]
});

// NOT operator
db.users.find({ age: { $not: { $lt: 30 } } }); // age is NOT less than 30
db.users.find({ city: { $not: { $in: ['London', 'Paris'] } } }); // city is NOT London or Paris

// Field existence checks
db.users.find({ phone: { $exists: true } }); // has phone field
db.users.find({ phone: { $exists: false } }); // does not have phone field

// Type checking
db.users.find({ age: { $type: 'int' } }); // age field is integer
db.users.find({ age: { $type: 'string' } }); // age field is string

// Array operations
db.users.find({ tags: { $in: ['admin', 'manager'] } }); // has admin or manager tag
db.users.find({ tags: { $nin: ['banned', 'suspended'] } }); // does NOT have banned or suspended tag
db.users.find({ tags: { $all: ['admin', 'verified'] } }); // has both admin AND verified tags
db.users.find({ tags: { $size: 3 } }); // has exactly 3 tags
```

### Advanced Query Features

#### Text Search

```js
// Syntax:
// .createIndex({ field: "text" })
// Create text index on multiple fields
db.products.createIndex({ name: 'text', description: 'text' });

// Syntax:
// .find({ $text: { $search: "search terms" } })
// Text search query
db.products.find({ $text: { $search: 'wireless mouse' } });
```

#### Regular Expressions

```js
// Syntax:
// .find({ field: /pattern/ })
// Find users whose name starts with "A"
db.users.find({ name: /^A/ });

// Syntax:
// .find({ field: /pattern/i })
// Case-insensitive search
db.users.find({ name: /alice/i });

// Syntax:
// .find({ field: /pattern/ })
// Find users whose name contains "john"
db.users.find({ name: /john/ });
```

#### Date/Time Operations

```js
// Syntax:
// .find({ dateField: { $gte: new Date() } })
// Find documents created in the last 7 days
db.sales.find({
  date: { $gte: new Date(Date.now() - 7 * 24 * 60 * 60 * 1000) },
});

// Syntax:
// .find({ dateField: { $gte: ISODate(), $lt: ISODate() } })
// Find documents by specific date range
db.sales.find({
  date: {
    $gte: ISODate('2024-01-01'),
    $lt: ISODate('2024-02-01'),
  },
});
```

#### Bulk Operations

```js
// Syntax:
// .bulkWrite([operations])
// Bulk insert multiple documents
db.users.bulkWrite([
  { insertOne: { document: { name: 'Alice', age: 30 } } },
  { insertOne: { document: { name: 'Bob', age: 25 } } },
]);

// Syntax:
// .bulkWrite([operations])
// Bulk update multiple documents
db.users.bulkWrite([
  { updateOne: { filter: { name: 'Alice' }, update: { $set: { age: 31 } } } },
  { updateOne: { filter: { name: 'Bob' }, update: { $set: { age: 26 } } } },
]);
```

#### Count and Distinct

```js
// Syntax:
// .countDocuments(query)
// Count documents matching criteria
db.users.countDocuments({ age: { $gt: 25 } });
// Note: Same as: db.users.count({ age: { $gt: 25 } }) (deprecated)

// Syntax:
// .distinct(field, query)
// Get distinct values for a field
db.users.distinct('city');
// Note: More efficient than: db.users.find({}, { city: 1 }).distinct('city')

// Syntax:
// .distinct(field, query)
// Get distinct cities for users over 25
db.users.distinct('city', { age: { $gt: 25 } });
```

#### Skip and Limit for Pagination

```js
// Syntax:
// .find().skip(n).limit(n)
// Pagination example - get users 11-15
db.users.find().skip(10).limit(5);

// Syntax:
// .find().sort().skip().limit()
// Pagination with sorting
db.users.find().sort({ name: 1 }).skip(10).limit(5);
// Note: For better performance on large datasets, use cursor-based pagination with _id
```

#### Explain Plan

```js
// Syntax:
// .find(query).explain("executionStats")
// See how MongoDB executes a query
db.users.find({ age: { $gt: 25 } }).explain('executionStats');

// Syntax:
// .find(query).explain("queryPlanner")
// See the query plan without executing
db.users.find({ age: { $gt: 25 } }).explain('queryPlanner');
```

#### Why Use .explain()?

- **Performance Analysis**: Understand how MongoDB executes your queries
- **Index Usage**: See if your queries are using indexes effectively
- **Optimization**: Identify slow queries and optimize them
- **Debugging**: Understand why queries are slow or not using expected indexes

#### Explain Modes

```js
// queryPlanner (default) - Shows the plan without executing
db.users.find({ age: { $gt: 25 } }).explain('queryPlanner');

// executionStats - Shows actual execution statistics
db.users.find({ age: { $gt: 25 } }).explain('executionStats');

// allPlansExecution - Shows all considered plans
db.users.find({ age: { $gt: 25 } }).explain('allPlansExecution');
```

#### Practical Examples

```js
// Check if a query uses an index
db.users.find({ email: 'alice@example.com' }).explain('executionStats');
// Look for "winningPlan" → "inputStage" → "indexName" in the output

// Compare query performance with and without index
// Without index:
db.users.find({ age: { $gt: 25 } }).explain('executionStats');
// With index:
db.users.createIndex({ age: 1 });
db.users.find({ age: { $gt: 25 } }).explain('executionStats');

// Analyze aggregation pipeline performance
db.sales
  .aggregate([
    { $match: { date: { $gte: ISODate('2024-01-01') } } },
    { $group: { _id: '$productId', total: { $sum: '$amount' } } },
  ])
  .explain('executionStats');
```

#### Key Metrics to Look For

- **nReturned**: Number of documents returned
- **totalKeysExamined**: Number of index entries examined
- **totalDocsExamined**: Number of documents examined
- **executionTimeMillis**: Time taken to execute the query
- **winningPlan**: The plan MongoDB chose to execute

## Indexing

**What are Indexes?**
Indexes are data structures that improve the speed of data retrieval operations on a collection. Without indexes, MongoDB must scan every document in a collection to find matches.

**Why We Need Indexes:**

- **Performance**: Queries run much faster with proper indexes
- **Scalability**: Essential for large collections
- **Efficiency**: Reduces CPU and memory usage
- **User Experience**: Faster response times for applications

**Note**: An index allows for quick lookup of a field. And it takes up more memory and slows insert, update, and remove operations. So use it wisely. If there's a field you search for but you don't do many updates, I would recommend using an index.

### Creating Indexes

```js
// Syntax:
// .createIndex({ field: 1 }) // 1 for ascending, -1 for descending
// Single field index
db.users.createIndex({ email: 1 });

// Syntax:
// .createIndex({ field1: 1, field2: 1 })
// Compound index (multiple fields)
db.users.createIndex({ age: 1, city: 1 });
// Note: Order matters! Index { age: 1, city: 1 } is different from { city: 1, age: 1 }

// Syntax:
// .createIndex({ field: 1 }, { unique: true })
// Unique index (prevents duplicate values)
db.users.createIndex({ email: 1 }, { unique: true });

// Syntax:
// .createIndex({ field: "text" })
// Text index for text search
db.products.createIndex({ name: 'text', description: 'text' });

// Syntax:
// .createIndex({ field: 1 }, { sparse: true })
// Sparse index (only includes documents with the field)
db.users.createIndex({ phone: 1 }, { sparse: true });
// Note: Sparse indexes are smaller and faster but only work for documents with the indexed field
```

### Index Types and Examples

```js
// Single Field Index
db.products.createIndex({ sku: 1 }); // Ascending
db.products.createIndex({ price: -1 }); // Descending

// Compound Index (order matters!)
db.sales.createIndex({ customerId: 1, date: -1 });
// Good for queries like: db.sales.find({ customerId: "123", date: { $gte: startDate } })

// Unique Index
db.users.createIndex({ username: 1 }, { unique: true });
// Prevents duplicate usernames

// Text Index
// MongoDB comes with a full-text search engine that can be used to search for documents in a collection.
db.products.createIndex({ name: 'text', description: 'text' });
// Enables text search: db.products.find({ $text: { $search: "wireless" } })

// Sparse Index
db.users.createIndex({ phone: 1 }, { sparse: true });
// Only indexes documents that have a phone field
```

### Checking Index Usage

```js
// List all indexes on a collection
db.users.getIndexes();

// Check if a query uses an index
db.users.find({ email: 'alice@example.com' }).explain('executionStats');
// Look for "winningPlan" → "inputStage" → "indexName"

// Drop an index
db.users.dropIndex('email_1'); // Drop by name
db.users.dropIndex({ email: 1 }); // Drop by specification
```

### Index Best Practices

```js
// Create indexes for frequently queried fields
db.orders.createIndex({ customerId: 1, status: 1 });

// Create indexes for fields used in sorting
db.products.createIndex({ price: 1 });

// Create indexes for fields used in range queries
db.sales.createIndex({ date: 1 });

// Avoid creating too many indexes (each index uses storage and slows writes)
// Monitor index usage and remove unused indexes
```

## Capped Collections

**What are Capped Collections?**
Capped collections are fixed-size collections that automatically overwrite the oldest documents when they reach their maximum size. They maintain insertion order and are useful for logging, caching, and real-time data.

**Why Use Capped Collections:**

- **Fixed Size**: Prevents unlimited growth
- **Automatic Cleanup**: Oldest documents are automatically removed
- **Performance**: Fast writes and reads due to fixed size
- **Order Preservation**: Documents are stored in insertion order

### Creating Capped Collections

```js
// Syntax:
// db.createCollection(name, { capped: true, size: bytes, max: documents })
// Create a capped collection with size limit
db.createCollection('logs', { capped: true, size: 1000000 }); // 1MB size limit

// Syntax:
// db.createCollection(name, { capped: true, size: bytes, max: documents })
// Create a capped collection with both size and document count limits
db.createCollection('recent_orders', {
  capped: true,
  size: 5000000, // 5MB
  max: 1000, // Maximum 1000 documents
});

// Syntax:
// db.createCollection(name, { capped: true, size: bytes })
// Create a simple capped collection for recent activities
db.createCollection('user_activities', {
  capped: true,
  size: 2000000, // 2MB
});

// Syntax:
// db.createCollection(name, { capped: true, autoIndexID: false })
// Create a capped collection without automatic _id indexing
db.createCollection('high_performance_logs', {
  capped: true,
  size: 1000000, // 1MB
  autoIndexID: false, // Disable automatic _id index for better performance
});

// Syntax:
// db.createCollection(name, { capped: true, autoIndexID: true })
// Explicitly enable automatic _id indexing (default behavior)
db.createCollection('logs_with_id_index', {
  capped: true,
  size: 2000000, // 2MB
  autoIndexID: true, // Enable automatic _id index
});
```

### Working with Capped Collections

```js
// Insert documents (oldest will be overwritten when limit is reached)
db.logs.insertOne({
  timestamp: new Date(),
  message: 'User logged in',
  userId: '123',
});

// Query capped collection (returns documents in insertion order)
db.logs.find().sort({ $natural: 1 }); // Ascending (oldest first)
db.logs.find().sort({ $natural: -1 }); // Descending (newest first)

// Check collection stats
db.logs.stats();

// Convert existing collection to capped (requires empty collection)
db.runCommand({ convertToCapped: 'logs', size: 1000000 });
// Note: Collection must be empty to convert to capped
```

### Use Cases

```js
// Application logs
db.createCollection('app_logs', { capped: true, size: 10000000 }); // 10MB

// Recent user sessions
db.createCollection('sessions', { capped: true, size: 5000000, max: 5000 });

// Real-time metrics
db.createCollection('metrics', { capped: true, size: 2000000, max: 10000 });

// Chat messages (last 1000 messages)
db.createCollection('chat_messages', {
  capped: true,
  size: 1000000,
  max: 1000,
});
```

## Schema Validation

**What is Schema Validation?**
In MongoDB, there is no strict schema validation set up by default. Schema validation allows you to enforce document structure and data types in MongoDB collections. While MongoDB is schema-less, you can still add validation rules to ensure data consistency and quality.

**Why Use Schema Validation:**

- **Data Quality**: Ensures documents have required fields and correct data types
- **Application Consistency**: Prevents invalid data from being inserted
- **Error Prevention**: Catches data issues early in the development process
- **Documentation**: Serves as documentation for expected data structure

### Creating Collections with Validation

```js
// Syntax:
// db.createCollection(name, { validator: { $jsonSchema: { ... } } })
// Create collection with validation rules
db.createCollection('users', {
  validator: {
    $jsonSchema: {
      bsonType: 'object',
      required: ['name', 'email', 'age'],
      properties: {
        name: {
          bsonType: 'string',
          description: 'Name must be a string and is required',
        },
        email: {
          bsonType: 'string',
          pattern: '^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$',
          description: 'Email must be a valid email address',
        },
        age: {
          bsonType: 'int',
          minimum: 0,
          maximum: 120,
          description: 'Age must be an integer between 0 and 120',
        },
        phone: {
          bsonType: 'string',
          pattern: '^\\+?[1-9]\\d{1,14}$',
          description: 'Phone must be a valid international number',
        },
      },
    },
  },
});
```

### Adding Validation to Existing Collections

```js
// Syntax:
// db.runCommand({ collMod: collectionName, validator: { $jsonSchema: { ... } } })
// Add validation to existing collection
db.runCommand({
  collMod: 'products',
  validator: {
    $jsonSchema: {
      bsonType: 'object',
      required: ['name', 'price', 'category'],
      properties: {
        name: {
          bsonType: 'string',
          minLength: 1,
          maxLength: 100,
        },
        price: {
          bsonType: 'number',
          minimum: 0,
        },
        category: {
          enum: ['Electronics', 'Clothing', 'Books', 'Home'],
        },
        tags: {
          bsonType: 'array',
          items: {
            bsonType: 'string',
          },
        },
      },
    },
  },
});
```

### Validation Examples

```js
// Product validation with nested objects
db.createCollection('products', {
  validator: {
    $jsonSchema: {
      bsonType: 'object',
      required: ['sku', 'name', 'price', 'inventory'],
      properties: {
        sku: {
          bsonType: 'string',
          pattern: '^[A-Z]{2}\\d{4}$', // Format: AA1234
        },
        name: {
          bsonType: 'string',
          minLength: 1,
          maxLength: 200,
        },
        price: {
          bsonType: 'number',
          minimum: 0,
        },
        inventory: {
          bsonType: 'object',
          required: ['quantity', 'location'],
          properties: {
            quantity: {
              bsonType: 'int',
              minimum: 0,
            },
            location: {
              bsonType: 'string',
            },
          },
        },
        tags: {
          bsonType: 'array',
          items: {
            bsonType: 'string',
          },
          maxItems: 10,
        },
      },
    },
  },
});

// Order validation with complex rules
db.createCollection('orders', {
  validator: {
    $jsonSchema: {
      bsonType: 'object',
      required: ['customerId', 'items', 'total', 'status'],
      properties: {
        customerId: {
          bsonType: 'objectId',
        },
        items: {
          bsonType: 'array',
          minItems: 1,
          items: {
            bsonType: 'object',
            required: ['productId', 'quantity', 'price'],
            properties: {
              productId: {
                bsonType: 'objectId',
              },
              quantity: {
                bsonType: 'int',
                minimum: 1,
              },
              price: {
                bsonType: 'number',
                minimum: 0,
              },
            },
          },
        },
        total: {
          bsonType: 'number',
          minimum: 0,
        },
        status: {
          enum: ['pending', 'processing', 'shipped', 'delivered', 'cancelled'],
        },
        createdAt: {
          bsonType: 'date',
        },
      },
    },
  },
});
```

### Validation Options

```js
// Syntax:
// db.createCollection(name, { validator: {...}, validationLevel: 'strict' })
// Create collection with strict validation
db.createCollection('strict_users', {
  validator: {
    $jsonSchema: {
      bsonType: 'object',
      required: ['email'],
      properties: {
        email: {
          bsonType: 'string',
          pattern: '^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$',
        },
      },
    },
  },
  validationLevel: 'strict', // Reject invalid documents
  validationAction: 'error', // Throw error for invalid documents
});

// Syntax:
// db.createCollection(name, { validator: {...}, validationLevel: 'moderate' })
// Create collection with moderate validation (warnings only)
db.createCollection('moderate_users', {
  validator: {
    $jsonSchema: {
      bsonType: 'object',
      required: ['email'],
      properties: {
        email: {
          bsonType: 'string',
        },
      },
    },
  },
  validationLevel: 'moderate', // Allow invalid documents but log warnings
  validationAction: 'warn', // Log warnings for invalid documents
});
```

### Testing Validation

```js
// Test valid document
db.users.insertOne({
  name: 'John Doe',
  email: 'john@example.com',
  age: 30,
}); // ✅ Success

// Test invalid document
db.users.insertOne({
  name: 'Jane Doe',
  email: 'invalid-email',
  age: 150,
}); // ❌ Error: validation failed

// Test missing required field
db.users.insertOne({
  name: 'Bob Smith',
  age: 25,
}); // ❌ Error: missing required field 'email'
// Note: Validation errors can be caught in application code for better error handling
```

## Aggregation

Aggregation is MongoDB's framework for processing documents and returning computed results. It's similar to SQL's GROUP BY, but more powerful and flexible. In MongoDB aggregation, the pipeline executes from **top to bottom**, with each stage processing the output of the previous stage.

**Example Use Cases**

- Count total orders per user
- Sum sales per month
- Group products by category
- Filter, sort, reshape documents

### Common Aggregation Operators

| Stage                          | Purpose                                         |
| ------------------------------ | ----------------------------------------------- |
| `$match`                       | Filters documents (like `WHERE`)                |
| `$group`                       | Groups by a field (like `GROUP BY`)             |
| `$sum`, `$avg`, `$min`, `$max` | Calculations                                    |
| `$sort`                        | Sorts the output                                |
| `$project`                     | Shapes the result (select fields, rename, etc.) |
| `$lookup`                      | Performs joins                                  |
| `$limit`, `$skip`              | Pagination                                      |
| `$unwind`                      | Deconstructs arrays                             |
| `$addFields`                   | Adds new fields to documents                    |
| `$count`                       | Counts documents in the pipeline                |
| `$out`                         | Writes results to a new collection              |
| `$merge`                       | Writes results to existing collection           |
| `$collStats`                   | Returns collection statistics                   |
| `$queryHistory`                | Returns query history (Atlas Data Federation)   |
| `$sql`                         | Executes SQL queries (Atlas Data Federation)    |

### Aggregation Example

1.

```js
// Syntax:
// .aggregate([pipeline])
// Group by city and count users
db.users.aggregate([{ $group: { _id: '$city', count: { $sum: 1 } } }]);
// Note: More efficient than: db.users.find().forEach(function(doc) { /* count manually */ })
```

2.

```js
db.posts.aggregate([
  // Stage 1: Only find documents that have more than 1 like
  {
    $match: { likes: { $gt: 1 } },
  },
  // Stage 2: Group documents by the category field and sum each categories likes
  {
    $group: { _id: '$category', totalLikes: { $sum: '$likes' } },
  },
]);
```

3.

```js
db.sales.aggregate([
  // Stage 1: Group sales by product ID and sum the amounts
  { $group: { _id: '$productId', totalSales: { $sum: '$amount' } } },
  // Stage 2: Sort the results by total sales in descending order
  { $sort: { totalSales: -1 } },
]);
```

4.

```js
db.orders.aggregate([
  // Stage 1: Filter only delivered orders
  { $match: { status: 'delivered' } },
  // Stage 2: Group by customer ID and sum their total spent
  { $group: { _id: '$customerId', totalSpent: { $sum: '$amount' } } },
  // Stage 3: Sort customers by total spent in descending order
  { $sort: { totalSpent: -1 } },
]);
```

5.

```js
// Complex aggregation with multiple stages
db.sales.aggregate([
  // Stage 1: Filter sales from last month
  {
    $match: {
      date: {
        $gte: ISODate('2024-06-01'),
        $lt: ISODate('2024-07-01'),
      },
    },
  },
  // Stage 2: Unwind the items array
  { $unwind: '$items' },
  // Stage 3: Group by product and calculate totals
  {
    $group: {
      _id: '$items.product_id',
      totalQuantity: { $sum: '$items.quantity' },
      totalRevenue: {
        $sum: { $multiply: ['$items.price', '$items.quantity'] },
      },
    },
  },
  // Stage 4: Sort by revenue
  { $sort: { totalRevenue: -1 } },
  // Stage 5: Limit to top 10
  { $limit: 10 },
]);
```

**Performance Note:** Aggregation pipelines can be resource-intensive. Use indexes on fields used in `$match` stages, and consider using `$limit` early in the pipeline to reduce the number of documents processed.
// Note: Always use indexes on fields in $match stages for optimal performance

### Advanced Aggregation Operators

#### $merge - Write to Existing Collection

```js
// Syntax:
// .aggregate([{ $merge: { into: "collection", whenMatched: "replace" } }])
// Merge aggregation results into existing collection
db.sales.aggregate([
  { $group: { _id: '$productId', totalSales: { $sum: '$amount' } } },
  {
    $merge: {
      into: 'product_summary',
      whenMatched: 'replace',
      whenNotMatched: 'insert',
    },
  },
]);
```

#### $collStats - Collection Statistics

```js
// Syntax:
// .aggregate([{ $collStats: { latencyStats: { histograms: true } } }])
// Get detailed collection statistics
db.users.aggregate([
  {
    $collStats: {
      latencyStats: { histograms: true },
      storageStats: {},
      count: {},
    },
  },
]);

// Syntax:
// .aggregate([{ $collStats: { storageStats: {} } }])
// Get storage statistics only
db.products.aggregate([{ $collStats: { storageStats: {} } }]);
```

#### $queryHistory - Query History (Atlas Data Federation)

```js
// Syntax:
// .aggregate([{ $queryHistory: { ... } }])
// Get query history from federated data sources
db.federated_collection.aggregate([
  {
    $queryHistory: {
      startDate: ISODate('2024-01-01'),
      endDate: ISODate('2024-12-31'),
      limit: 100,
    },
  },
]);
```

#### $sql - SQL Queries (Atlas Data Federation)

```js
// Syntax:
// .aggregate([{ $sql: { query: "SELECT * FROM table" } }])
// Execute SQL query on federated data source
db.federated_collection.aggregate([
  {
    $sql: {
      query:
        'SELECT customer_id, SUM(amount) as total FROM sales GROUP BY customer_id',
    },
  },
]);

// Syntax:
// .aggregate([{ $sql: { query: "SELECT * FROM table WHERE condition" } }])
// SQL query with conditions
db.federated_collection.aggregate([
  {
    $sql: {
      query: "SELECT * FROM users WHERE age > 25 AND city = 'New York'",
    },
  },
]);
```

## Contribution Guidelines

_Contributions are welcome! Please open an issue or submit a pull request._

## License

_Add your license here (e.g., MIT, Apache 2.0, etc.)._

## Contact/Support

_For questions or support, please contact bablukpik@gmail.com or open an issue in this repository._
