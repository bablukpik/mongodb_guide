# Modeling Relationships in MongoDB

MongoDB is a NoSQL database and does not have built-in foreign key or primary key constraints like SQL databases. However, you can model relationships between collections using references and indexes.

---

## 1. Primary Key Equivalent

- Every document in MongoDB has a unique `_id` field (automatically created if not specified).
- You can create a unique index on any field to enforce uniqueness (like a primary key):

```js
db.products.createIndex({ sku: 1 }, { unique: true });
```

---

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

---

## 3. Enforcing Relationships

- **No automatic enforcement:** Deleting a product will not update or remove references in `sales`.
- **Application-level checks:** Your code should check that referenced documents exist before inserting or updating.

---

## 4. Join-like Queries with $lookup

You can use the `$lookup` aggregation stage to perform a join-like operation between collections.

**Example:** Join `sales` with `products` to get product details for each sale:

```js
db.sales.aggregate([
  {
    $lookup: {
      from: "products",
      localField: "items.product_id",
      foreignField: "_id",
      as: "product_details",
    },
  },
]);
```

This adds a `product_details` array to each sale, containing the matched products.

---

## 5. Embedding vs. Referencing

MongoDB allows you to either **embed** related data as subdocuments or **reference** related documents in other collections. Choosing the right approach depends on your data access patterns and requirements.

### Embedding

- Store related data inside the parent document as an array or subdocument.
- **Best for:**
  - Data that is always accessed together
  - One-to-few or one-to-one relationships
  - Data that does not grow unbounded

**Example: Embedded Order Items**

```js
{
  _id: ObjectId("..."),
  customer_id: ObjectId("..."),
  items: [
    { sku: "SKU123", name: "Wireless Mouse", quantity: 2, price: 25.99 },
    { sku: "SKU124", name: "USB Keyboard", quantity: 1, price: 15.00 }
  ],
  total: 66.98
}
```

### Referencing

- Store only the reference (e.g., `_id`) to related documents in other collections.
- **Best for:**
  - Large, shared, or frequently changing related data
  - Many-to-many or large one-to-many relationships
  - When you need to avoid document size limits

**Example: Reference Pattern**

```js
{
  _id: ObjectId("..."),
  customer_id: ObjectId("..."),
  items: [
    { product_id: ObjectId("..."), quantity: 2, price: 25.99 }
  ],
  total: 51.98
}
```

### Best Practices

- Avoid deep nesting for large or frequently changing arrays.
- Use embedding for small, tightly coupled data that is always accessed together.
- Use referencing for large, shared, or frequently updated data.
- Consider denormalization (duplicating some data) for performance-critical queries.

### Cascading Deletes/Updates

- MongoDB does **not** support cascading deletes or updates natively.
- You must handle cascading changes in your application logic (e.g., delete related sales when a product is deleted).

### Performance Considerations

- `$lookup` can be slow on large collections; use with care.
- Embedding can improve read performance for related data, but may increase document size.
- Use indexes to optimize queries on referenced fields.

---

## 6. Summary Table

| SQL Concept           | MongoDB Equivalent                |
| --------------------- | --------------------------------- |
| Primary Key           | `_id` field, or unique index      |
| Foreign Key           | Manual reference (ObjectId, etc.) |
| Join                  | `$lookup` in aggregation          |
| Referential Integrity | Enforced in application code      |

---

## 7. Notes

- MongoDB is flexible: you can embed related data or use references, depending on your needs.
- Use unique indexes to enforce uniqueness.
- Use `$lookup` for join-like queries, but be aware of performance for large datasets.
- Always handle referential integrity in your application logic.

---

## 8. Advanced Patterns & Real-World Scenarios

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

---

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
db.students_courses.find({ student_id: ObjectId("...") });
```

To find all students in a course:

```js
db.students_courses.find({ course_id: ObjectId("...") });
```

You can use `$lookup` to join with `students` or `courses`.

---

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

---

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

---

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

---

### 6. Transactions & Data Consistency

- MongoDB supports multi-document transactions (since v4.0) for replica sets and sharded clusters.
- Use transactions when you need atomic updates across multiple collections (e.g., updating stock and recording a sale).
- For most use cases, design your schema to minimize the need for multi-document transactions (favor embedding for atomicity).

---
