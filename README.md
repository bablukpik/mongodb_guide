# MongoDB Guide

A practical MongoDB setup with guides for running MongoDB in Docker, performing CRUD operations, and modeling real-world data relationships. Includes a full POS/inventory example and advanced relationship patterns.

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

## Table of Contents

- [1. Prerequisites](#1-prerequisites)
- [2. Running MongoDB](#2-running-mongodb)
- [3. Connecting to MongoDB](#3-connecting-to-mongodb)
- [4. Example Queries (CRUD & Operators)](#4-example-queries-crud--operators)
- [5. Notes](#5-notes)
- [6. POS/Inventory Example](#6-posinventory-example)
- [7. Modeling Relationships in MongoDB](#7-modeling-relationships-in-mongodb)
- [8. Contribution Guidelines](#contribution-guidelines)
- [9. License](#license)
- [10. Contact/Support](#contactsupport)

## 1. Prerequisites

- [Docker](https://docs.docker.com/get-docker/) installed
- [Docker Compose](https://docs.docker.com/compose/install/) installed
- [mongosh](https://www.mongodb.com/try/download/shell) (MongoDB Shell) installed on your host

## 2. Running MongoDB

1. Start MongoDB with Docker Compose:

   ```sh
   docker compose up -d
   ```

   This will start a MongoDB container with the configuration from `docker-compose.yml`.

2. To stop and remove the container and data volume:
   ```sh
   docker compose down -v
   ```

## 3. Connecting to MongoDB

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

## 4. Example Queries (CRUD & Operators)

### Insert Documents

```js
// Syntax:
// .insertOne(document)
db.users.insertOne({ name: "Alice", age: 30, city: "London" });

// Syntax:
// .insertMany([document, ...])
db.users.insertMany([
  { name: "Bob", age: 25, city: "Paris" },
  { name: "Charlie", age: 35, city: "Berlin" },
]);
```

### Find (Read) Documents

```js
// Syntax:
// .find(query, projection)
db.users.find();

// Syntax:
// .findOne(query)
db.users.findOne({ name: "Alice" });

// Syntax:
// .find(query)
db.users.find({ age: { $gt: 25 } }); // age > 25

// Syntax:
// .find(query, projection)
db.users.find({}, { name: 1, city: 1, _id: 0 });

// Syntax:
// .find().sort(sort).limit(n)
db.users.find().sort({ age: -1 }).limit(2);
```

### Update Documents

```js
// Syntax:
// .updateOne(filter, update)
db.users.updateOne({ name: "Alice" }, { $set: { city: "New York" } });

// Syntax:
// .updateMany(filter, update)
db.users.updateMany({ city: "Paris" }, { $inc: { age: 1 } });

// Syntax:
// .replaceOne(filter, replacement)
db.users.replaceOne(
  { name: "Charlie" },
  { name: "Charlie", age: 36, city: "Munich" }
);

// Syntax:
// .updateOne(filter, { $unset: { field: "" } })
db.users.updateOne({ name: "Alice" }, { $unset: { city: "" } });

// Syntax:
// .updateOne(filter, { $rename: { oldField: "newField" } })
db.users.updateOne({ name: "Alice" }, { $rename: { name: "fullName" } });

// Syntax:
// .updateOne(filter, { $set: { field: value } })
db.users.updateOne({ fullName: "Alice" }, { $set: { country: "UK" } });

// Syntax:
// .updateOne(filter, { $inc: { field: amount } })
db.users.updateOne({ fullName: "Alice" }, { $inc: { age: 1 } });

// Syntax:
// .updateOne(filter, { $mul: { field: factor } })
db.users.updateOne({ fullName: "Alice" }, { $mul: { age: 2 } });

// Syntax:
// .updateOne(filter, { $min: { field: value } })
db.users.updateOne({ fullName: "Alice" }, { $min: { age: 18 } });

// Syntax:
// .updateOne(filter, { $max: { field: value } })
db.users.updateOne({ fullName: "Alice" }, { $max: { age: 65 } });

// Syntax:
// .updateOne(filter, { $currentDate: { field: true } })
db.users.updateOne(
  { fullName: "Alice" },
  { $currentDate: { lastModified: true } }
);

// Syntax:
// .updateOne(filter, { $push: { arrayField: value } })
db.users.updateOne({ fullName: "Alice" }, { $push: { tags: "new" } });

// Syntax:
// .updateOne(filter, { $addToSet: { arrayField: value } })
db.users.updateOne({ fullName: "Alice" }, { $addToSet: { tags: "unique" } });

// Syntax:
// .updateOne(filter, { $pull: { arrayField: value } })
db.users.updateOne({ fullName: "Alice" }, { $pull: { tags: "old" } });

// Syntax:
// .updateOne(filter, { $pop: { arrayField: 1 or -1 } })
db.users.updateOne({ fullName: "Alice" }, { $pop: { tags: 1 } });
```

### Delete Documents

```js
// Syntax:
// .deleteOne(filter)
db.users.deleteOne({ name: "Bob" });

// Syntax:
// .deleteMany(filter)
db.users.deleteMany({ age: { $lt: 30 } });
```

### Common Query and Update Operators

```js
// Comparison
$eq; // equal
db.users.find({ age: { $eq: 30 } });
$ne; // not equal
db.users.find({ city: { $ne: "London" } });
$gt; // greater than
$gte; // greater than or equal
$lt; // less than
$lte; // less than or equal
$in; // in array
db.users.find({ city: { $in: ["London", "Paris"] } });
$nin; // not in array

// Logical
$and; // and
db.users.find({ $and: [{ age: { $gt: 25 } }, { city: "London" }] });
$or; // or
db.users.find({ $or: [{ city: "London" }, { city: "Paris" }] });
$not; // not
db.users.find({ age: { $not: { $gt: 30 } } });
$nor; // nor

// Element
$exists; // field exists
db.users.find({ phone: { $exists: true } });
$type; // field type
db.users.find({ age: { $type: "int" } });

// Array
$all; // all elements match
db.users.find({ tags: { $all: ["red", "blue"] } });
$size; // array size
db.users.find({ tags: { $size: 2 } });
$elemMatch; // element match
db.users.find({ scores: { $elemMatch: { $gt: 80, $lt: 90 } } });

// Update Operators
$set; // set a field value
$unset; // remove a field
$inc; // increment a field
$mul; // multiply a field
$min; // set to min value
$max; // set to max value
$currentDate; // set to current date
$rename; // rename a field
$push; // add value to array
$addToSet; // add value to array if not present
$pull; // remove value from array
$pop; // remove first or last element from array
```

### Aggregation Example

```js
// Syntax:
// .aggregate([pipeline])
// Group by city and count users
db.users.aggregate([{ $group: { _id: "$city", count: { $sum: 1 } } }]);
```

### Other Useful Commands

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

## 5. Notes

- The database (`mongo_db`) will only appear in `show dbs` after it contains at least one document.
- The root user is created in the `admin` database. You can create additional users as needed.
- To reset credentials, you must remove the data volume: `docker compose down -v` and then `docker compose up -d`.

## 6. Contribution Guidelines

_Contributions are welcome! Please open an issue or submit a pull request._

## 7. License

_Add your license here (e.g., MIT, Apache 2.0, etc.)._

## 8. Contact/Support

_For questions or support, please contact bablukpik@gmail.com or open an issue in this repository._
