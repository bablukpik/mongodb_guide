# Modeling Relationships in MongoDB

MongoDB is a NoSQL database and does not have built-in foreign key or primary key constraints like SQL databases. However, you can model relationships between collections using references and indexes.

## Table of Contents

- [Document Size Limitation](#document-size-limitation)
- [1. Primary Key Equivalent](#1-primary-key-equivalent)
- [2. Foreign Key Equivalent (References)](#2-foreign-key-equivalent-references)
- [3. Enforcing Relationships](#3-enforcing-relationships)
- [4. Join-like Queries with $lookup](#4-join-like-queries-with-lookup)
- [5. Embedding vs. Referencing](#5-embedding-vs-referencing)
  - [When to Embed (Preferred)](#when-to-embed-preferred)
  - [When to Reference (Required for Large Data)](#when-to-reference-required-for-large-data)
- [6. Advanced Patterns & Real-World Scenarios](#6-advanced-patterns--real-world-scenarios)
  - [1. Polymorphic References](#1-polymorphic-references)
  - [2. Many-to-Many Relationships (Join/Bridge Collections)](#2-many-to-many-relationships-joinbridge-collections)
  - [3. One-to-Many Relationships](#3-one-to-many-relationships)
    - [Example: Users and Posts](#example-users-and-posts)
    - [Querying One-to-Many Relationships](#querying-one-to-many-relationships)
    - [Example: Customers and Orders](#example-customers-and-orders)
    - [Querying Customer Orders](#querying-customer-orders)
  - [4. Tree Structures (Hierarchies)](#4-tree-structures-hierarchies)
    - [a) Parent Reference](#a-parent-reference)
    - [b) Materialized Path](#b-materialized-path)
    - [c) Array of Ancestors](#c-array-of-ancestors)
  - [5. Real-World Example: E-commerce Order](#5-real-world-example-e-commerce-order)
  - [6. Real-World Example: Social Network](#6-real-world-example-social-network)
  - [7. Large Data Scenarios Requiring References](#7-large-data-scenarios-requiring-references)
    - [Blog System with Large Content](#blog-system-with-large-content)
    - [E-commerce with Extensive Product Data](#e-commerce-with-extensive-product-data)
  - [8. Transactions & Data Consistency](#8-transactions--data-consistency)
- [9. Best Practices for Large Data](#9-best-practices-for-large-data)
  - [When to Split Documents](#when-to-split-documents)
  - [Performance Considerations](#performance-considerations)
  - [Example: Monitoring Document Size](#example-monitoring-document-size)
- [10. Notes](#10-notes)
- [11. Summary Table](#11-summary-table)
- [12. Cascading Deletes/Updates](#12-cascading-deletesupdates)
- [References](#references)

## Document Size Limitation

**Important**: MongoDB has a **16MB document size limit**. While MongoDB prefers embedding for performance, you must use referencing when:

- Embedded data would exceed 16MB
- Data is shared across multiple documents
- Data changes independently and frequently
- You need to avoid data duplication

## 1. Primary Key Equivalent

- Every document in MongoDB has a unique `_id` field (automatically created if not specified).
- You can create a unique index on any field to enforce uniqueness (like a primary key):

```js
db.products.createIndex({ sku: 1 }, { unique: true });
// Note: Same as: db.products.ensureIndex({ sku: 1 }, { unique: true }) (deprecated)
```

## 2. Foreign Key Equivalent (References)

- Store the `_id` (or another unique field) of a document from one collection as a field in another collection.
- **Example:** In a `sales` document, reference a product and a customer:

```js
{
  _id: ObjectId("..."),
  customer_id: ObjectId("..."), // references customers._id
  items: [
    { product_id: ObjectId("..."), quantity: 2, price: 25.99 } // references products._id
  ],
  // ...
}
```

- This is a manual reference. MongoDB does **not** enforce referential integrity—you must ensure referenced documents exist in your application logic.

## 3. Enforcing Relationships

- **No automatic enforcement:** Deleting a product will not update or remove references in `sales`.
- **Application-level checks:** Your code should check that referenced documents exist before inserting or updating.

## 4. Join-like Queries with $lookup

You can use the `$lookup` aggregation stage to perform a join-like operation between collections.

**Example:** Join `sales` with `products` to get product details for each sale:

```js
db.sales.aggregate([
  {
    $lookup: {
      from: 'products',
      localField: 'items.product_id',
      foreignField: '_id',
      as: 'product_details',
    },
  },
]);
```

This adds a `product_details` array to each sale, containing the matched products.

## 5. Embedding vs. Referencing

MongoDB allows you to either **embed** related data as subdocuments ( Store related data inside the parent document as an array) or **reference** related documents in other collections. Choosing the right approach depends on your data access patterns and requirements.

### When to Embed (Preferred)

- **Small, related data** that doesn't change frequently
- **Data that is always accessed together**
- **One-to-few or one-to-one relationships** (e.g., user profile with address)
- **Data that won't exceed 16MB** when embedded

**Example: Embedded User Profile**

```js
{
  _id: ObjectId("..."),
  username: "alice",
  email: "alice@example.com",
  profile: {
    firstName: "Alice",
    lastName: "Smith",
    address: {
      street: "123 Main St",
      city: "Metropolis",
      zip: "12345"
    },
    preferences: {
      theme: "dark",
      notifications: true
    }
  }
}
```

### When to Reference (Required for Large Data)

- **Data that would exceed 16MB** when embedded and you need to avoid document size limits
- **Frequently changing data** that affects multiple documents
- **Shared data** across multiple documents
- **Large arrays** or complex nested structures
- **Many-to-many or large one-to-many relationships**

**Example: Referencing Large Data**

```js
// User document (keeps basic info)
{
  _id: ObjectId("..."),
  username: "alice",
  email: "alice@example.com",
  profile_id: ObjectId("...") // Reference to detailed profile
}

// Profile document (detailed info that could be large)
{
  _id: ObjectId("..."),
  user_id: ObjectId("..."),
  bio: "Very long bio...",
  photos: ["url1", "url2", ...], // Could be hundreds of photos
  posts: ["post1", "post2", ...], // Could be thousands of posts
  // ... many more fields
}
```

## 6. Advanced Patterns & Real-World Scenarios

### 1. Polymorphic References

A field may reference documents from multiple collections. For example, a `notifications` collection might reference either a `user`, `order`, or `product`:

```js
{
  _id: ObjectId("..."),
  target_type: "user", // or "order", "product"
  target_id: ObjectId("...")
}
```

Your application logic uses `target_type` to know which collection to query for `target_id`.

### 2. Many-to-Many Relationships (Join/Bridge Collections)

For relationships like students and courses, use a join collection:

```js
// students_courses join collection
{
  student_id: ObjectId("..."),
  course_id: ObjectId("...")
}
```

To find all courses for a student:

```js
db.students_courses.find({ student_id: ObjectId('...') });
```

To find all students in a course:

```js
db.students_courses.find({ course_id: ObjectId('...') });
```

You can use `$lookup` to join with `students` or `courses`.

### 3. One-to-Many Relationships

For relationships where one document relates to many documents in another collection, use references in the "many" side.

#### Example: Users and Posts

```js
// User document (one)
{
  _id: ObjectId("..."),
  username: "alice",
  email: "alice@example.com",
  name: "Alice Smith"
}

// Post documents (many) - each post references the user
{
  _id: ObjectId("..."),
  title: "My First Post",
  content: "Hello world!",
  author_id: ObjectId("..."), // Reference to user
  created_at: new Date()
}

{
  _id: ObjectId("..."),
  title: "MongoDB Tips",
  content: "Here are some tips...",
  author_id: ObjectId("..."), // Same user, different post
  created_at: new Date()
}
```

#### Querying One-to-Many Relationships

```js
// Find all posts by a specific user
db.posts.find({ author_id: ObjectId('...') });

// Find user and their posts using $lookup
db.users.aggregate([
  { $match: { _id: ObjectId('...') } },
  {
    $lookup: {
      from: 'posts',
      localField: '_id',
      foreignField: 'author_id',
      as: 'user_posts',
    },
  },
]);
// Note: $lookup is more efficient than multiple queries for complex joins

// Count posts per user
db.posts.aggregate([
  {
    $group: {
      _id: '$author_id',
      post_count: { $sum: 1 },
    },
  },
]);
// Note: More efficient than: db.posts.find({ author_id: ObjectId('...') }).count()
```

#### Example: Customers and Orders

```js
// Customer document (one)
{
  _id: ObjectId("..."),
  name: "John Doe",
  email: "john@example.com",
  phone: "123-456-7890"
}

// Order documents (many) - each order references the customer
{
  _id: ObjectId("..."),
  customer_id: ObjectId("..."), // Reference to customer
  items: [
    { product_id: ObjectId("..."), quantity: 2, price: 25.99 }
  ],
  total: 51.98,
  status: "pending",
  created_at: new Date()
}

{
  _id: ObjectId("..."),
  customer_id: ObjectId("..."), // Same customer, different order
  items: [
    { product_id: ObjectId("..."), quantity: 1, price: 15.00 }
  ],
  total: 15.00,
  status: "delivered",
  created_at: new Date()
}
```

#### Querying Customer Orders

```js
// Find all orders for a specific customer
db.orders.find({ customer_id: ObjectId('...') });

// Find customer with their order history
db.customers.aggregate([
  { $match: { _id: ObjectId('...') } },
  {
    $lookup: {
      from: 'orders',
      localField: '_id',
      foreignField: 'customer_id',
      as: 'order_history',
    },
  },
  {
    $addFields: {
      total_spent: {
        $sum: '$order_history.total',
      },
      order_count: {
        $size: '$order_history',
      },
    },
  },
]);
// Note: This aggregation provides both data and calculated fields in a single query
```

### 4. Tree Structures (Hierarchies)

#### a) Parent Reference

Each document stores the `_id` of its parent:

```js
{ _id: 1, name: "Electronics", parent_id: null }
{ _id: 2, name: "Laptops", parent_id: 1 }
{ _id: 3, name: "Ultrabooks", parent_id: 2 }
```

#### b) Materialized Path

Each document stores the path from the root:

```js
{ _id: 3, name: "Ultrabooks", path: [1, 2, 3] }
```

#### c) Array of Ancestors

Each document stores an array of ancestor IDs:

```js
{ _id: 3, name: "Ultrabooks", ancestors: [1, 2] }
```

These patterns help with querying subtrees or breadcrumbs.

### 5. Real-World Example: E-commerce Order

```js
{
  _id: ObjectId("..."),
  customer_id: ObjectId("..."), // reference
  shipping_address: {
    street: "123 Main St",
    city: "Metropolis",
    zip: "12345"
  }, // embedded
  items: [
    { product_id: ObjectId("..."), quantity: 2, price: 25.99 },
    { product_id: ObjectId("..."), quantity: 1, price: 15.00 }
  ], // referenced
  status_history: [
    { status: "placed", date: ISODate("2024-07-01T10:00:00Z") },
    { status: "shipped", date: ISODate("2024-07-02T15:00:00Z") }
  ] // embedded
}
```

- Embedded address and status history for fast access
- Referenced customer and products for normalization

### 6. Real-World Example: Social Network

- **Users**: referenced by `_id`
- **Posts**: reference user, embed small comments
- **Comments**: reference post and user

```js
// Post with embedded comments
{
  _id: ObjectId("..."),
  user_id: ObjectId("..."),
  content: "Hello world!",
  comments: [
    { user_id: ObjectId("..."), text: "Nice post!", date: new Date() },
    { user_id: ObjectId("..."), text: "Thanks!", date: new Date() }
  ]
}
```

- For many comments, use a separate `comments` collection and reference the post.

### 7. Large Data Scenarios Requiring References

#### Blog System with Large Content

```js
// Post document (basic info)
{
  _id: ObjectId("..."),
  title: "My Blog Post",
  author_id: ObjectId("..."),
  created_at: new Date(),
  tags: ["mongodb", "nosql"]
}

// Post content document (large content)
{
  _id: ObjectId("..."),
  post_id: ObjectId("..."),
  content: "Very long article content...",
  images: ["url1", "url2", ...], // Could be many images
  attachments: ["file1", "file2", ...] // Could be large files
}
```

#### E-commerce with Extensive Product Data

```js
// Product document (basic info)
{
  _id: ObjectId("..."),
  sku: "PROD123",
  name: "Wireless Mouse",
  price: 25.99,
  category_id: ObjectId("...")
}

// Product details document (extensive data)
{
  _id: ObjectId("..."),
  product_id: ObjectId("..."),
  description: "Very detailed description...",
  specifications: { /* extensive specs */ },
  reviews: [/* hundreds of reviews */],
  images: [/* many product images */],
  variants: [/* many product variants */]
}
```

### 8. Transactions & Data Consistency

- MongoDB supports multi-document transactions (since v4.0) for replica sets and sharded clusters.
- Use transactions when you need atomic updates across multiple collections (e.g., updating stock and recording a sale).
- For most use cases, design your schema to minimize the need for multi-document transactions (favor embedding for atomicity).

## 9. Best Practices for Large Data

### When to Split Documents

- **Content exceeds 16MB**: Split into multiple documents
- **Frequently updated fields**: Separate from rarely updated fields
- **Shared data**: Reference instead of duplicating
- **Large arrays**: Consider separate collection for array items

### Performance Considerations

- **Index foreign keys**: Always index referenced fields
- **Use $lookup sparingly**: Can be expensive on large collections
- **Consider denormalization**: Duplicate small, frequently accessed data
- **Monitor document size**: Use `db.collection.stats()` to check sizes

### Example: Monitoring Document Size

```js
// Check collection statistics
db.products.stats();

// Check individual document size
db.products.find().forEach(function (doc) {
  var size = Object.bsonsize(doc);
  if (size > 10000000) {
    // 10MB warning
    print('Large document: ' + doc._id + ' - ' + size + ' bytes');
  }
});
// Note: Object.bsonsize() is more accurate than JSON.stringify().length for size estimation
```

## 10. Notes

- MongoDB is flexible: you can embed related data or use references, depending on your needs.
- Use unique indexes to enforce uniqueness.
- Use `$lookup` for join-like queries, but be aware of performance for large datasets.
- Always handle referential integrity in your application logic.
- **Remember the 16MB limit**: This is the most important constraint when choosing between embedding and referencing.

## 11. Summary Table

| SQL Concept           | MongoDB Equivalent                |
| --------------------- | --------------------------------- |
| Primary Key           | `_id` field, or unique index      |
| Foreign Key           | Manual reference (ObjectId, etc.) |
| Join                  | `$lookup` in aggregation          |
| Referential Integrity | Enforced in application code      |

## 12. Cascading Deletes/Updates

- MongoDB does **not** support cascading deletes or updates natively.
- You must handle cascading changes in your application logic (e.g., delete related sales when a product is deleted).

## 12. Advanced Patterns & Real-World Scenarios

### 1. Polymorphic References

A field may reference documents from multiple collections. For example, a `notifications` collection might reference either a `user`, `order`, or `product`:

```js
{
  _id: ObjectId("..."),
  target_type: "user", // or "order", "product"
  target_id: ObjectId("...")
}
```

Your application logic uses `target_type` to know which collection to query for `target_id`.

### 2. Many-to-Many Relationships (Join/Bridge Collections)

For relationships like students and courses, use a join collection:

```js
// students_courses join collection
{
  student_id: ObjectId("..."),
  course_id: ObjectId("...")
}
```

To find all courses for a student:

```js
db.students_courses.find({ student_id: ObjectId('...') });
```

To find all students in a course:

```js
db.students_courses.find({ course_id: ObjectId('...') });
```

You can use `$lookup` to join with `students` or `courses`.

### 3. Tree Structures (Hierarchies)

#### a) Parent Reference

Each document stores the `_id` of its parent:

```js
{ _id: 1, name: "Electronics", parent_id: null }
{ _id: 2, name: "Laptops", parent_id: 1 }
{ _id: 3, name: "Ultrabooks", parent_id: 2 }
```

#### b) Materialized Path

Each document stores the path from the root:

```js
{ _id: 3, name: "Ultrabooks", path: [1, 2, 3] }
```

#### c) Array of Ancestors

Each document stores an array of ancestor IDs:

```js
{ _id: 3, name: "Ultrabooks", ancestors: [1, 2] }
```

These patterns help with querying subtrees or breadcrumbs.

### 4. Real-World Example: E-commerce Order

```js
{
  _id: ObjectId("..."),
  customer_id: ObjectId("..."), // reference
  shipping_address: {
    street: "123 Main St",
    city: "Metropolis",
    zip: "12345"
  }, // embedded
  items: [
    { product_id: ObjectId("..."), quantity: 2, price: 25.99 },
    { product_id: ObjectId("..."), quantity: 1, price: 15.00 }
  ], // referenced
  status_history: [
    { status: "placed", date: ISODate("2024-07-01T10:00:00Z") },
    { status: "shipped", date: ISODate("2024-07-02T15:00:00Z") }
  ] // embedded
}
```

- Embedded address and status history for fast access
- Referenced customer and products for normalization

### 5. Real-World Example: Social Network

- **Users**: referenced by `_id`
- **Posts**: reference user, embed small comments
- **Comments**: reference post and user

```js
// Post with embedded comments
{
  _id: ObjectId("..."),
  user_id: ObjectId("..."),
  content: "Hello world!",
  comments: [
    { user_id: ObjectId("..."), text: "Nice post!", date: new Date() },
    { user_id: ObjectId("..."), text: "Thanks!", date: new Date() }
  ]
}
```

- For many comments, use a separate `comments` collection and reference the post.

### 6. Transactions & Data Consistency

- MongoDB supports multi-document transactions (since v4.0) for replica sets and sharded clusters.
- Use transactions when you need atomic updates across multiple collections (e.g., updating stock and recording a sale).
- For most use cases, design your schema to minimize the need for multi-document transactions (favor embedding for atomicity).

## References

- [MongoDB Documentation: Model Referenced One-to-Many Relationships](https://www.mongodb.com/docs/manual/tutorial/model-referenced-one-to-many-relationships-between-documents/)
- [MongoDB Atlas Data Federation: $lookup Stage](https://www.mongodb.com/docs/atlas/data-federation/supported-unsupported/pipeline/lookup-stage/)
