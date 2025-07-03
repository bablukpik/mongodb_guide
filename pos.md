# MongoDB POS/Inventory Example

This guide demonstrates how to model a simple Point of Sale (POS) or Inventory system using MongoDB.

## Collections

- `products` — Product catalog
- `customers` — Customer information
- `sales` — Sales transactions
- `inventory_movements` — (Optional) Stock changes log

---

## Example Documents

### products

```js
{
  _id: ObjectId("..."),
  sku: "SKU123",
  name: "Wireless Mouse",
  category: "Electronics",
  price: 25.99,
  stock: 100,
  supplier: "TechSupplier Inc.",
  tags: ["mouse", "wireless", "electronics"]
}
```

### customers

```js
{
  _id: ObjectId("..."),
  name: "John Doe",
  email: "john@example.com",
  phone: "123-456-7890",
  address: "123 Main St, City, Country"
}
```

### sales

```js
{
  _id: ObjectId("..."),
  date: ISODate("2024-07-01T10:00:00Z"),
  customer_id: ObjectId("..."),
  items: [
    { product_id: ObjectId("..."), quantity: 2, price: 25.99 },
    { product_id: ObjectId("..."), quantity: 1, price: 15.00 }
  ],
  total: 66.98,
  payment_method: "credit_card"
}
```

### inventory_movements (optional)

```js
{
  _id: ObjectId("..."),
  product_id: ObjectId("..."),
  type: "sale", // or 'restock'
  quantity: -2,
  date: new Date(),
  reference: "sale_id_or_note"
}
```

---

## Example Queries

### Insert a new product

```js
db.products.insertOne({
  sku: "SKU124",
  name: "USB Keyboard",
  category: "Electronics",
  price: 15.0,
  stock: 50,
  supplier: "TechSupplier Inc.",
  tags: ["keyboard", "usb", "electronics"],
});
```

### Find all products in stock

```js
db.products.find({ stock: { $gt: 0 } });
```

### Update stock after a sale

```js
// Decrease stock by 2 for a product
db.products.updateOne({ sku: "SKU123" }, { $inc: { stock: -2 } });
```

### Record a sale

```js
db.sales.insertOne({
  date: new Date(),
  customer_id: ObjectId("..."), // Use the actual customer _id
  items: [{ product_id: ObjectId("..."), quantity: 2, price: 25.99 }],
  total: 51.98,
  payment_method: "cash",
});
```

### List all sales for a customer

```js
db.sales.find({ customer_id: ObjectId("...") });
```

### Get total sales for today

```js
db.sales.aggregate([
  {
    $match: {
      date: {
        $gte: ISODate("2024-07-02T00:00:00Z"),
        $lt: ISODate("2024-07-03T00:00:00Z"),
      },
    },
  },
  { $group: { _id: null, totalSales: { $sum: "$total" } } },
]);
```

### Find low-stock products

```js
db.products.find({ stock: { $lt: 10 } });
```

### Track inventory movement (optional)

```js
db.inventory_movements.insertOne({
  product_id: ObjectId("..."),
  type: "sale",
  quantity: -2,
  date: new Date(),
  reference: "sale_id_or_note",
});
```

---

## Low Stock Alerts

You can monitor inventory and alert users when a product is running low. This is useful for restocking and preventing out-of-stock situations.

### Check Stock for a Specific Product

```js
// Find the stock level for a specific product by SKU
db.products.findOne({ sku: "SKU123" }, { name: 1, stock: 1, _id: 0 });
```

_Example result:_

```js
{ name: "Wireless Mouse", stock: 3 }
```

### Find All Low-Stock Products

```js
// Find all products with stock less than or equal to a threshold (e.g., 5)
db.products.find({ stock: { $lte: 5 } }, { name: 1, sku: 1, stock: 1, _id: 0 });
```

_Example result:_

```js
[
  { name: "Wireless Mouse", sku: "SKU123", stock: 3 },
  { name: "USB Keyboard", sku: "SKU124", stock: 2 },
];
```

**Tip:**

- Use these queries in your backend to power dashboard widgets or notifications.
- You can schedule checks or run them on-demand to alert staff when restocking is needed.

---

## Notes

- Use `ObjectId()` for references between collections.
- Expand schema as needed (discounts, taxes, users, suppliers, etc.).
- Use backend code to handle business logic and transactions in a real application.

---

## User/Staff Management

### Example: Users Collection

```js
db.users.insertOne({
  username: "admin",
  password_hash: "...", // Store hashed passwords only!
  role: "admin", // or "cashier", "manager"
  name: "Alice Smith",
});
```

### Example: Find All Staff

```js
db.users.find({ role: { $in: ["admin", "cashier", "manager"] } });
```

---

## Supplier Management

### Example: Suppliers Collection

```js
db.suppliers.insertOne({
  name: "TechSupplier Inc.",
  contact: "supplier@example.com",
  phone: "123-456-7890",
});
```

### Linking Products to Suppliers

Add a `supplier_id` field to products:

```js
db.products.insertOne({
  sku: "SKU125",
  name: "Webcam",
  price: 40.0,
  stock: 20,
  supplier_id: ObjectId("..."), // Reference to suppliers._id
  tags: ["webcam", "electronics"],
});
```

---

## Restocking Workflow

### Record a Restock Event

```js
db.inventory_movements.insertOne({
  product_id: ObjectId("..."),
  type: "restock",
  quantity: 50,
  date: new Date(),
  reference: "PO-2024-001",
});
```

### Update Product Stock

```js
db.products.updateOne({ _id: ObjectId("...") }, { $inc: { stock: 50 } });
```

---

## Sales Reporting

### Total Sales Per Product

```js
db.sales.aggregate([
  { $unwind: "$items" },
  {
    $group: {
      _id: "$items.product_id",
      totalSold: { $sum: "$items.quantity" },
    },
  },
  { $sort: { totalSold: -1 } },
]);
```

### Best-Selling Products (Top 5)

```js
db.sales.aggregate([
  { $unwind: "$items" },
  {
    $group: {
      _id: "$items.product_id",
      totalSold: { $sum: "$items.quantity" },
    },
  },
  { $sort: { totalSold: -1 } },
  { $limit: 5 },
]);
```

### Sales by Category

```js
db.sales.aggregate([
  { $unwind: "$items" },
  {
    $lookup: {
      from: "products",
      localField: "items.product_id",
      foreignField: "_id",
      as: "product",
    },
  },
  { $unwind: "$product" },
  {
    $group: {
      _id: "$product.category",
      totalSold: { $sum: "$items.quantity" },
    },
  },
]);
```

---

## Discounts and Promotions

### Model a Discount in a Sale

```js
db.sales.insertOne({
  date: new Date(),
  customer_id: ObjectId("..."),
  items: [{ product_id: ObjectId("..."), quantity: 2, price: 25.99 }],
  discount: { type: "percent", value: 10 }, // 10% off
  total: 46.78,
  payment_method: "cash",
});
```

---

## Returns and Refunds

### Record a Return

```js
db.sales.insertOne({
  date: new Date(),
  customer_id: ObjectId("..."),
  items: [{ product_id: ObjectId("..."), quantity: -1, price: 25.99 }],
  type: "return",
  total: -25.99,
  payment_method: "refund",
});
```

### Update Inventory for a Return

```js
db.products.updateOne({ _id: ObjectId("...") }, { $inc: { stock: 1 } });
```

---

## Indexing for Performance

### Create Indexes

```js
// Unique index on SKU
db.products.createIndex({ sku: 1 }, { unique: true });
// Index on stock for low-stock queries
db.products.createIndex({ stock: 1 });
// Index on category for reporting
db.products.createIndex({ category: 1 });
```

---

## More on Relationships

See [Modeling Relationships in MongoDB](./relationship.md) for advanced patterns, joins, and best practices.
