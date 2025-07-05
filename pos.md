# MongoDB POS/Inventory Example

This guide demonstrates how to model a simple Point of Sale (POS) or Inventory system using MongoDB.

## Table of Contents

- [Collections](#collections)
- [Example Documents](#example-documents)
  - [products](#products)
  - [customers](#customers)
  - [sales](#sales)
  - [inventory_movements](#inventory_movements)
  - [users](#users)
  - [suppliers](#suppliers)
  - [categories](#categories)
- [Example Queries](#example-queries)
- [User/Staff Management](#userstaff-management)
- [Supplier Management](#supplier-management)
- [Restocking Workflow](#restocking-workflow)
- [Sales Reporting](#sales-reporting)
- [Low Stock Alerts](#low-stock-alerts)
- [Discounts and Promotions](#discounts-and-promotions)
- [Returns and Refunds](#returns-and-refunds)
- [Payment Processing](#payment-processing)
- [Inventory Alerts](#inventory-alerts)
- [Reporting & Analytics](#reporting--analytics)
- [Indexing for Performance](#indexing-for-performance)
- [API Examples](#api-examples)
- [Notes](#notes)

## Collections

- `products` — Product catalog
- `customers` — Customer information
- `sales` — Sales transactions
- `inventory_movements` — (Optional) Stock changes log
- `users` — Staff/Admin users
- `suppliers` — Supplier information
- `categories` — Product categories

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
  cost: 15.00,
  stock: 100,
  min_stock: 10,
  supplier_id: ObjectId("..."),
  tags: ["mouse", "wireless", "electronics"],
  created_at: new Date(),
  updated_at: new Date()
}
```

### customers

```js
{
  _id: ObjectId("..."),
  name: "John Doe",
  email: "john@example.com",
  phone: "123-456-7890",
  address: {
    street: "123 Main St",
    city: "Metropolis",
    state: "CA",
    zip: "12345"
  },
  loyalty_points: 150,
  created_at: new Date()
}
```

### sales

```js
{
  _id: ObjectId("..."),
  sale_number: "SALE-2024-001",
  customer_id: ObjectId("..."),
  cashier_id: ObjectId("..."),
  items: [
    {
      product_id: ObjectId("..."),
      sku: "SKU123",
      name: "Wireless Mouse",
      quantity: 2,
      unit_price: 25.99,
      total_price: 51.98
    }
  ],
  subtotal: 51.98,
  tax: 5.20,
  discount: 0,
  total: 57.18,
  payment_method: "credit_card",
  status: "completed",
  created_at: new Date()
}
```

### inventory_movements (optional)

```js
{
  _id: ObjectId("..."),
  product_id: ObjectId("..."),
  type: "sale", // or 'restock', 'adjustment', 'return'
  quantity: -2,
  reference: "SALE-2024-001",
  notes: "Sold to customer",
  created_by: ObjectId("..."),
  created_at: new Date()
}
```

### users

```js
{
  _id: ObjectId("..."),
  username: "cashier1",
  name: "Alice Smith",
  email: "alice@store.com",
  role: "cashier", // admin, manager, cashier
  permissions: ["create_sale", "view_reports"],
  is_active: true,
  created_at: new Date()
}
```

### suppliers

```js
{
  _id: ObjectId("..."),
  name: "TechSupplier Inc.",
  contact_person: "Bob Johnson",
  email: "bob@techsupplier.com",
  phone: "555-123-4567",
  address: {
    street: "456 Supplier Ave",
    city: "Supplier City",
    state: "CA",
    zip: "54321"
  },
  payment_terms: "Net 30",
  created_at: new Date()
}
```

### categories

```js
{
  _id: ObjectId("..."),
  name: "Electronics",
  description: "Electronic devices and accessories",
  parent_id: null, // for subcategories
  is_active: true,
  created_at: new Date()
}
```

---

## Example Queries

### Insert a new product

```js
db.products.insertOne({
  sku: 'SKU124',
  name: 'USB Keyboard',
  category: 'Electronics',
  price: 15.0,
  cost: 8.0,
  stock: 50,
  min_stock: 5,
  supplier_id: ObjectId('...'),
  tags: ['keyboard', 'usb', 'electronics'],
  created_at: new Date(),
  updated_at: new Date(),
});
// Note: Same as: db.products.insert({ ... }) (deprecated)
```

### Find all products in stock

```js
db.products.find({ stock: { $gt: 0 } });
// Note: Same as: db.products.find({ stock: { $gte: 1 } })
```

### Update stock after a sale

```js
db.products.updateOne(
  { sku: 'SKU123' },
  {
    $inc: { stock: -2 },
    $set: { updated_at: new Date() },
  },
);
```

### Record a sale

```js
db.sales.insertOne({
  sale_number: 'SALE-2024-002',
  customer_id: ObjectId('...'),
  cashier_id: ObjectId('...'),
  items: [
    {
      product_id: ObjectId('...'),
      sku: 'SKU123',
      name: 'Wireless Mouse',
      quantity: 2,
      unit_price: 25.99,
      total_price: 51.98,
    },
  ],
  subtotal: 51.98,
  tax: 5.2,
  total: 57.18,
  payment_method: 'cash',
  status: 'completed',
  created_at: new Date(),
});
```

### List all sales for a customer

```js
db.sales.find({ customer_id: ObjectId('...') });
```

### Get total sales for today

```js
db.sales.aggregate([
  {
    $match: {
      created_at: {
        $gte: new Date(new Date().setHours(0, 0, 0, 0)),
        $lt: new Date(new Date().setHours(23, 59, 59, 999)),
      },
    },
  },
  {
    $group: {
      _id: null,
      totalSales: { $sum: '$total' },
      totalTransactions: { $sum: 1 },
    },
  },
]);
// Note: More efficient than: db.sales.find({ created_at: { $gte: startOfDay, $lt: endOfDay } }).forEach(...)
```

### Find low-stock products

```js
db.products.find({ stock: { $lte: '$min_stock' } });
// Note: This query uses field comparison - stock must be less than or equal to the min_stock field value
```

### Track inventory movement (optional)

```js
db.inventory_movements.insertOne({
  product_id: ObjectId('...'),
  type: 'sale',
  quantity: -2,
  reference: 'SALE-2024-002',
  notes: 'Sold to customer',
  created_by: ObjectId('...'),
  created_at: new Date(),
});
```

---

## User/Staff Management

### Example: Users Collection

```js
db.users.insertOne({
  username: 'admin',
  name: 'Admin User',
  email: 'admin@store.com',
  role: 'admin',
  permissions: [
    'create_sale',
    'view_reports',
    'manage_inventory',
    'manage_users',
  ],
  is_active: true,
  created_at: new Date(),
});
```

### Example: Find All Staff

```js
db.users.find({ role: { $in: ['admin', 'cashier', 'manager'] } });
```

### User Authentication Check

```js
db.users.findOne({
  username: 'cashier1',
  is_active: true,
});
```

---

## Supplier Management

### Example: Suppliers Collection

```js
db.suppliers.insertOne({
  name: 'TechSupplier Inc.',
  contact_person: 'Bob Johnson',
  email: 'bob@techsupplier.com',
  phone: '555-123-4567',
  address: {
    street: '456 Supplier Ave',
    city: 'Supplier City',
    state: 'CA',
    zip: '54321',
  },
  payment_terms: 'Net 30',
  created_at: new Date(),
});
```

### Linking Products to Suppliers

Add a `supplier_id` field to products:

```js
db.products.insertOne({
  sku: 'SKU125',
  name: 'Webcam',
  price: 40.0,
  cost: 20.0,
  stock: 20,
  min_stock: 3,
  supplier_id: ObjectId('...'), // Reference to suppliers._id
  tags: ['webcam', 'electronics'],
  created_at: new Date(),
  updated_at: new Date(),
});
```

---

## Restocking Workflow

### Record a Restock Event

```js
db.inventory_movements.insertOne({
  product_id: ObjectId('...'),
  type: 'restock',
  quantity: 50,
  reference: 'PO-2024-001',
  notes: 'Restocked from supplier',
  created_by: ObjectId('...'),
  created_at: new Date(),
});
```

### Update Product Stock

```js
db.products.updateOne(
  { _id: ObjectId('...') },
  {
    $inc: { stock: 50 },
    $set: { updated_at: new Date() },
  },
);
```

---

## Sales Reporting

### Total Sales Per Product

```js
db.sales.aggregate([
  { $unwind: '$items' },
  {
    $group: {
      _id: '$items.product_id',
      totalSold: { $sum: '$items.quantity' },
      totalRevenue: { $sum: '$items.total_price' },
    },
  },
  { $sort: { totalRevenue: -1 } },
]);
```

### Best-Selling Products (Top 5)

```js
db.sales.aggregate([
  { $unwind: '$items' },
  {
    $group: {
      _id: '$items.product_id',
      totalSold: { $sum: '$items.quantity' },
      totalRevenue: { $sum: '$items.total_price' },
    },
  },
  { $sort: { totalSold: -1 } },
  { $limit: 5 },
]);
```

### Sales by Category

```js
db.sales.aggregate([
  { $unwind: '$items' },
  {
    $lookup: {
      from: 'products',
      localField: 'items.product_id',
      foreignField: '_id',
      as: 'product',
    },
  },
  { $unwind: '$product' },
  {
    $group: {
      _id: '$product.category',
      totalSold: { $sum: '$items.quantity' },
      totalRevenue: { $sum: '$items.total_price' },
    },
  },
]);
```

### Daily Sales Report

```js
db.sales.aggregate([
  {
    $match: {
      created_at: {
        $gte: new Date(new Date().setHours(0, 0, 0, 0)),
        $lt: new Date(new Date().setHours(23, 59, 59, 999)),
      },
    },
  },
  {
    $group: {
      _id: {
        hour: { $hour: '$created_at' },
        payment_method: '$payment_method',
      },
      totalSales: { $sum: '$total' },
      transactionCount: { $sum: 1 },
    },
  },
  { $sort: { '_id.hour': 1 } },
]);
```

---

## Low Stock Alerts

You can monitor inventory and alert users when a product is running low. This is useful for restocking and preventing out-of-stock situations.

### Check Stock for a Specific Product

```js
db.products.findOne(
  { sku: 'SKU123' },
  { name: 1, stock: 1, min_stock: 1, _id: 0 },
);
```

_Example result:_

```js
{ name: "Wireless Mouse", stock: 3, min_stock: 10 }
```

### Find All Low-Stock Products

```js
db.products.find(
  { stock: { $lte: '$min_stock' } },
  { name: 1, sku: 1, stock: 1, min_stock: 1, supplier_id: 1, _id: 0 },
);
```

_Example result:_

```js
[
  {
    name: 'Wireless Mouse',
    sku: 'SKU123',
    stock: 3,
    min_stock: 10,
    supplier_id: ObjectId('...'),
  },
  {
    name: 'USB Keyboard',
    sku: 'SKU124',
    stock: 2,
    min_stock: 5,
    supplier_id: ObjectId('...'),
  },
];
```

### Low Stock Alert with Supplier Info

```js
db.products.aggregate([
  { $match: { stock: { $lte: '$min_stock' } } },
  {
    $lookup: {
      from: 'suppliers',
      localField: 'supplier_id',
      foreignField: '_id',
      as: 'supplier',
    },
  },
  { $unwind: '$supplier' },
  {
    $project: {
      name: 1,
      sku: 1,
      stock: 1,
      min_stock: 1,
      supplier_name: '$supplier.name',
      supplier_email: '$supplier.email',
    },
  },
]);
```

**Tip:**

- Use these queries in your backend to power dashboard widgets or notifications.
- You can schedule checks or run them on-demand to alert staff when restocking is needed.

---

## Discounts and Promotions

### Model a Discount in a Sale

```js
db.sales.insertOne({
  sale_number: 'SALE-2024-003',
  customer_id: ObjectId('...'),
  cashier_id: ObjectId('...'),
  items: [
    {
      product_id: ObjectId('...'),
      sku: 'SKU123',
      name: 'Wireless Mouse',
      quantity: 2,
      unit_price: 25.99,
      total_price: 51.98,
    },
  ],
  subtotal: 51.98,
  discount: {
    type: 'percentage',
    value: 10,
    amount: 5.2,
  },
  tax: 4.68,
  total: 51.46,
  payment_method: 'cash',
  status: 'completed',
  created_at: new Date(),
});
```

### Promotional Discounts

```js
db.sales.insertOne({
  sale_number: 'SALE-2024-004',
  customer_id: ObjectId('...'),
  cashier_id: ObjectId('...'),
  items: [
    {
      product_id: ObjectId('...'),
      sku: 'SKU123',
      name: 'Wireless Mouse',
      quantity: 1,
      unit_price: 25.99,
      total_price: 25.99,
    },
  ],
  subtotal: 25.99,
  discount: {
    type: 'fixed',
    value: 5.0,
    amount: 5.0,
    promotion_code: 'SAVE5',
  },
  tax: 2.1,
  total: 23.09,
  payment_method: 'credit_card',
  status: 'completed',
  created_at: new Date(),
});
```

---

## Returns and Refunds

### Record a Return

```js
db.sales.insertOne({
  sale_number: 'RETURN-2024-001',
  customer_id: ObjectId('...'),
  cashier_id: ObjectId('...'),
  items: [
    {
      product_id: ObjectId('...'),
      sku: 'SKU123',
      name: 'Wireless Mouse',
      quantity: -1,
      unit_price: 25.99,
      total_price: -25.99,
    },
  ],
  subtotal: -25.99,
  tax: -2.6,
  total: -28.59,
  payment_method: 'refund',
  status: 'returned',
  return_reason: 'Defective product',
  original_sale: 'SALE-2024-001',
  created_at: new Date(),
});
```

### Update Inventory for a Return

```js
db.products.updateOne(
  { sku: 'SKU123' },
  {
    $inc: { stock: 1 },
    $set: { updated_at: new Date() },
  },
);
```

### Track Return in Inventory Movement

```js
db.inventory_movements.insertOne({
  product_id: ObjectId('...'),
  type: 'return',
  quantity: 1,
  reference: 'RETURN-2024-001',
  notes: 'Customer return - defective product',
  created_by: ObjectId('...'),
  created_at: new Date(),
});
```

---

## Payment Processing

### Different Payment Methods

```js
// Cash Payment
db.sales.insertOne({
  sale_number: 'SALE-2024-005',
  customer_id: ObjectId('...'),
  cashier_id: ObjectId('...'),
  items: [
    /* items */
  ],
  subtotal: 100.0,
  tax: 8.5,
  total: 108.5,
  payment_method: 'cash',
  payment_details: {
    cash_received: 110.0,
    change_given: 1.5,
  },
  status: 'completed',
  created_at: new Date(),
});

// Credit Card Payment
db.sales.insertOne({
  sale_number: 'SALE-2024-006',
  customer_id: ObjectId('...'),
  cashier_id: ObjectId('...'),
  items: [
    /* items */
  ],
  subtotal: 75.0,
  tax: 6.38,
  total: 81.38,
  payment_method: 'credit_card',
  payment_details: {
    card_type: 'Visa',
    last_four: '1234',
    transaction_id: 'TXN123456789',
  },
  status: 'completed',
  created_at: new Date(),
});
```

---

## Inventory Alerts

### Create Alert for Low Stock

```js
db.alerts.insertOne({
  type: 'low_stock',
  product_id: ObjectId('...'),
  product_name: 'Wireless Mouse',
  current_stock: 3,
  min_stock: 10,
  severity: 'high',
  is_resolved: false,
  created_at: new Date(),
});
```

### Find Active Alerts

```js
db.alerts
  .find({
    is_resolved: false,
  })
  .sort({ created_at: -1 });
```

---

## Reporting & Analytics

### Sales Performance by Cashier

```js
db.sales.aggregate([
  {
    $match: {
      created_at: {
        $gte: new Date(new Date().setMonth(new Date().getMonth() - 1)),
      },
    },
  },
  {
    $group: {
      _id: '$cashier_id',
      totalSales: { $sum: '$total' },
      transactionCount: { $sum: 1 },
      averageTransaction: { $avg: '$total' },
    },
  },
  {
    $lookup: {
      from: 'users',
      localField: '_id',
      foreignField: '_id',
      as: 'cashier',
    },
  },
  { $unwind: '$cashier' },
  {
    $project: {
      cashier_name: '$cashier.name',
      totalSales: 1,
      transactionCount: 1,
      averageTransaction: 1,
    },
  },
  { $sort: { totalSales: -1 } },
]);
```

### Customer Loyalty Analysis

```js
db.sales.aggregate([
  {
    $group: {
      _id: '$customer_id',
      totalSpent: { $sum: '$total' },
      visitCount: { $sum: 1 },
      lastVisit: { $max: '$created_at' },
    },
  },
  {
    $lookup: {
      from: 'customers',
      localField: '_id',
      foreignField: '_id',
      as: 'customer',
    },
  },
  { $unwind: '$customer' },
  {
    $project: {
      customer_name: '$customer.name',
      customer_email: '$customer.email',
      totalSpent: 1,
      visitCount: 1,
      lastVisit: 1,
      averageSpent: { $divide: ['$totalSpent', '$visitCount'] },
    },
  },
  { $sort: { totalSpent: -1 } },
]);
```

### Product Performance Analysis

```js
db.sales.aggregate([
  { $unwind: '$items' },
  {
    $group: {
      _id: '$items.product_id',
      totalSold: { $sum: '$items.quantity' },
      totalRevenue: { $sum: '$items.total_price' },
      averagePrice: { $avg: '$items.unit_price' },
    },
  },
  {
    $lookup: {
      from: 'products',
      localField: '_id',
      foreignField: '_id',
      as: 'product',
    },
  },
  { $unwind: '$product' },
  {
    $project: {
      product_name: '$product.name',
      product_sku: '$product.sku',
      category: '$product.category',
      totalSold: 1,
      totalRevenue: 1,
      averagePrice: 1,
      profit_margin: {
        $subtract: [
          '$totalRevenue',
          { $multiply: ['$totalSold', '$product.cost'] },
        ],
      },
    },
  },
  { $sort: { totalRevenue: -1 } },
]);
```

---

## Indexing for Performance

### Create Indexes

```js
// Syntax:
// .createIndex(fields, options)
// Unique index on SKU
db.products.createIndex({ sku: 1 }, { unique: true });

// Index on stock for low-stock queries
db.products.createIndex({ stock: 1 });

// Index on category for reporting
db.products.createIndex({ category: 1 });

// Compound index for sales queries
db.sales.createIndex({ customer_id: 1, created_at: -1 });

// Index for date range queries
db.sales.createIndex({ created_at: -1 });

// Text index for product search
db.products.createIndex({ name: 'text', description: 'text' });
```

---

## API Examples

### Get Product by SKU

```js
db.products.findOne({ sku: 'SKU123' });
// Note: More efficient than: db.products.find({ sku: 'SKU123' }).limit(1)
```

### Search Products

```js
db.products.find({
  $text: { $search: 'wireless mouse' },
});
// Note: Requires text index: db.products.createIndex({ name: 'text', description: 'text' })
```

### Get Customer Orders

```js
db.sales.aggregate([
  { $match: { customer_id: ObjectId('...') } },
  { $sort: { created_at: -1 } },
  { $limit: 10 },
]);
// Note: More efficient than: db.sales.find({ customer_id: ObjectId('...') }).sort({ created_at: -1 }).limit(10)
```

### Get Low Stock Products with Supplier Info

```js
db.products.aggregate([
  { $match: { stock: { $lte: '$min_stock' } } },
  {
    $lookup: {
      from: 'suppliers',
      localField: 'supplier_id',
      foreignField: '_id',
      as: 'supplier',
    },
  },
  { $unwind: '$supplier' },
  {
    $project: {
      name: 1,
      sku: 1,
      stock: 1,
      min_stock: 1,
      supplier_name: '$supplier.name',
      supplier_email: '$supplier.email',
      supplier_phone: '$supplier.phone',
    },
  },
]);
```

---

## More on Relationships

See [Modeling Relationships in MongoDB](./relationship.md) for advanced patterns, joins, and best practices.

---

## Notes

- Use `ObjectId()` for references between collections.
- Expand schema as needed (discounts, taxes, users, suppliers, etc.).
- Use backend code to handle business logic and transactions in a real application.
- Consider implementing data validation using MongoDB's schema validation features.
- Monitor performance and create appropriate indexes for your query patterns.
