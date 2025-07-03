# MongoDB Guide

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
// Insert one document
db.users.insertOne({ name: "Alice", age: 30, city: "London" });

// Insert multiple documents
db.users.insertMany([
  { name: "Bob", age: 25, city: "Paris" },
  { name: "Charlie", age: 35, city: "Berlin" },
]);
```

### Find (Read) Documents

```js
// Find all documents
db.users.find();

// Find one document
db.users.findOne({ name: "Alice" });

// Find with a query operator
db.users.find({ age: { $gt: 25 } }); // age > 25

// Projection (only show name and city)
db.users.find({}, { name: 1, city: 1, _id: 0 });

// Sort and limit
db.users.find().sort({ age: -1 }).limit(2);
```

### Update Documents

```js
// Update one document
db.users.updateOne({ name: "Alice" }, { $set: { city: "New York" } });

// Update multiple documents
db.users.updateMany({ city: "Paris" }, { $inc: { age: 1 } });

// Replace a document
db.users.replaceOne(
  { name: "Charlie" },
  { name: "Charlie", age: 36, city: "Munich" }
);
```

### Delete Documents

```js
// Delete one document
db.users.deleteOne({ name: "Bob" });

// Delete multiple documents
db.users.deleteMany({ age: { $lt: 30 } });
```

### Common Query Operators

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
```

### Aggregation Example

```js
// Group by city and count users
db.users.aggregate([{ $group: { _id: "$city", count: { $sum: 1 } } }]);
```

### Other Useful Commands

```js
// Show all databases
show dbs

// Show collections in current database
show collections

// Drop a collection
db.users.drop()

// Drop the current database
db.dropDatabase()
```

## 5. Notes

- The database (`mongo_db`) will only appear in `show dbs` after it contains at least one document.
- The root user is created in the `admin` database. You can create additional users as needed.
- To reset credentials, you must remove the data volume: `docker compose down -v` and then `docker compose up -d`.
