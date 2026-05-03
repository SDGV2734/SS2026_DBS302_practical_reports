# PRACTICAL 3 MongoDB E-Commerce

#### **Name:** Sonam Dorji Ghalley

#### **Student ID:** 02230299

#### **Course/Unit:** DBS302

## 1. AIM

To design and implement a scalable **e-commerce database schema** using MongoDB, perform advanced aggregation queries, and optimize performance using indexes and query analysis tools. 

## 2. OBJECTIVE

- Design collections for users, products, orders, and categories
- Implement schema using MongoDB
- Perform aggregation queries for analytics
- Optimize queries using indexes
- Analyze performance using `explain()`

## 3. THEORY BACKGROUND

MongoDB is a NoSQL document-oriented database that stores data in flexible JSON-like documents (BSON).

###  Key Concepts Used

- **Embedding vs Referencing**
    - Orders → Embedded items
    - Products → Referenced
- **Aggregation Framework**
    - `$match`, `$group`, `$project`, `$lookup`
- **Indexing**
    - Improves performance by avoiding full scans

## 4. SYSTEM DESIGN

### collection used

| Collection | Purpose |
| --- | --- |
| users | Stores customer data |
| products | Product catalog |
| orders | Order transactions |
| categories | Product grouping |

## 5. IMPLEMENTATION

### **PART 1: COLLECTION AND DATABASE SETUP**

**Step 1.1: Create Database**

```json
use ecommerce;
```

**Step 1.2:** Create collections (optional; MongoDB creates them automatically on first insert)

```json
db.createCollection("users");
db.createCollection("categories");
db.createCollection("products");
db.createCollection("orders");
```

![image.png](PRACTICAL%203%20MongoDB%20E-Commerce/image.png)

![image.png](PRACTICAL%203%20MongoDB%20E-Commerce/image%201.png)

the `users` collection stores customer information including contact details and address.

### PART 2: Insert Sample Data

**Step 2.1: Insert Users**

```json
db.users.insertMany([
  {
    name: "Tashi Dorji",
    email: "tashi@example.com",
    phone: "+975-17-123-456",
    address: {
      line1: "Building 12",
      city: "Thimphu",
      country: "Bhutan",
      postalCode: "11001"
    },
    createdAt: new Date("2026-04-18T08:00:00Z")
  },
  {
    name: "Sonam Choden",
    email: "sonam@example.com",
    phone: "+975-17-654-321",
    address: {
      line1: "Flat 3B",
      city: "Phuntsholing",
      country: "Bhutan",
      postalCode: "21001"
    },
    createdAt: new Date("2026-04-19T10:30:00Z")
  }
]);
```

![image.png](PRACTICAL%203%20MongoDB%20E-Commerce/image%202.png)

![image.png](PRACTICAL%203%20MongoDB%20E-Commerce/image%203.png)

****categories help organize products and support filtering in the catalog.

**Step 2.2: Insert Categories**

```json
const electronicsId = ObjectId();
const accessoriesId = ObjectId();

db.categories.insertMany([
  { _id: electronicsId, name: "Electronics", slug: "electronics", parentCategoryId: null },
  { _id: accessoriesId, name: "Accessories", slug: "accessories", parentCategoryId: electronicsId }
]);
```

![image.png](PRACTICAL%203%20MongoDB%20E-Commerce/image%204.png)

![image.png](PRACTICAL%203%20MongoDB%20E-Commerce/image%205.png)

**Step 2.3 Insert Products**

This example uses an attribute map to handle variable attributes.

```json
const headphonesId = ObjectId();
const cableId = ObjectId();
const keyboardId = ObjectId();

db.products.insertMany([
  {
    _id: headphonesId,
    name: "Wireless Bluetooth Headphones",
    slug: "wireless-bluetooth-headphones",
    categoryId: electronicsId,
    price: 129.99,
    currency: "USD",
    stock: 200,
    attributes: {
      brand: "Acme Audio",
      color: "black",
      wireless: true,
      batteryLifeHours: 24
    },
    tags: ["audio", "wireless", "headphones"],
    createdAt: new Date("2026-04-18T10:00:00Z")
  },
  {
    _id: cableId,
    name: "USB‑C Cable 1m",
    slug: "usb-c-cable-1m",
    categoryId: accessoriesId,
    price: 9.99,
    currency: "USD",
    stock: 500,
    attributes: {
      brand: "Acme Tech",
      lengthMeters: 1,
      color: "white"
    },
    tags: ["cable", "usb-c"],
    createdAt: new Date("2026-04-18T11:00:00Z")
  },
  {
    _id: keyboardId,
    name: "Mechanical Keyboard",
    slug: "mechanical-keyboard",
    categoryId: electronicsId,
    price: 79.99,
    currency: "USD",
    stock: 150,
    attributes: {
      brand: "Acme Input",
      layout: "US",
      switchType: "blue",
      backlight: true
    },
    tags: ["keyboard", "mechanical", "backlit"],
    createdAt: new Date("2026-04-19T09:00:00Z")
  }
]);
```

![image.png](PRACTICAL%203%20MongoDB%20E-Commerce/image%206.png)

![image.png](PRACTICAL%203%20MongoDB%20E-Commerce/image%207.png)

- products use a flexible `attributes` object
- this shows MongoDB’s schema flexibility
- it is suitable for e-commerce because different products have different attributes

**Step 2.4 Insert Orders**

```json
const tashi = db.users.findOne({ email: "tashi@example.com" });
const sonam = db.users.findOne({ email: "sonam@example.com" });

db.orders.insertMany([
  {
    userId: tashi._id,
    status: "PAID",
    items: [
      {
        productId: headphonesId,
        productName: "Wireless Bluetooth Headphones",
        unitPrice: 129.99,
        quantity: 2,
        lineTotal: 259.98
      },
      {
        productId: cableId,
        productName: "USB‑C Cable 1m",
        unitPrice: 9.99,
        quantity: 1,
        lineTotal: 9.99
      }
    ],
    grandTotal: 269.97,
    currency: "USD",
    createdAt: new Date("2026-04-19T15:30:00Z"),
    paymentMethod: "CARD"
  },
  {
    userId: sonam._id,
    status: "PAID",
    items: [
      {
        productId: keyboardId,
        productName: "Mechanical Keyboard",
        unitPrice: 79.99,
        quantity: 1,
        lineTotal: 79.99
      }
    ],
    grandTotal: 79.99,
    currency: "USD",
    createdAt: new Date("2026-04-20T09:15:00Z"),
    paymentMethod: "COD"
  }
]);
```

![image.png](PRACTICAL%203%20MongoDB%20E-Commerce/image%208.png)

![image.png](PRACTICAL%203%20MongoDB%20E-Commerce/image%209.png)

`items` are embedded:

- order items are read together with the order
- embedding improves read efficiency
- it also preserves product snapshot details like price at purchase time

### PART 3: Aggregation Queries

#### Query 1: Daily Sales Totals

```json
db.orders.aggregate([
  {
    $match: { status: "PAID" } // filter only completed orders
  },
  {
    $group: {
      _id: {
        year: { $year: "$createdAt" },
        month: { $month: "$createdAt" },
        day: { $dayOfMonth: "$createdAt" }
      },
      totalRevenue: { $sum: "$grandTotal" },
      orderCount: { $sum: 1 }
    }
  },
  {
    $project: {
      _id: 0,
      date: {
        $dateFromParts: {
          year: "$_id.year",
          month: "$_id.month",
          day: "$_id.day"
        }
      },
      totalRevenue: 1,
      orderCount: 1
    }
  },
  {
    $sort: { date: 1 }
  }
]);
```

**output** 

![image.png](PRACTICAL%203%20MongoDB%20E-Commerce/image%2010.png)

- `$match`: It filters paid orders
- `$group` : It calculates total revenue and number of orders per day
- `$project` : It formats the output
- `$sort` : It arranges by date

#### **Query 2: Top N Products by Revenue**

```json
db.orders.aggregate([
  { $match: { status: "PAID" } },
  { $unwind: "$items" },
  {
    $group: {
      _id: "$items.productId",
      productName: { $first: "$items.productName" },
      totalRevenue: { $sum: "$items.lineTotal" },
      totalQuantity: { $sum: "$items.quantity" }
    }
  },
  { $sort: { totalRevenue: -1 } },
  { $limit: 5 }
]).pretty()
```

**output**

![image.png](PRACTICAL%203%20MongoDB%20E-Commerce/image%2011.png)

- `$unwind` breaks the `items` array into separate documents
- `$group` summarizes revenue and quantity per product
- `$sort` ranks the highest revenue products first
- `$limit` returns top 5

#### **Query 3: Average Order Value per User**

```json
db.orders.aggregate([
  { $match: { status: "PAID" } },
  {
    $group: {
      _id: "$userId",
      totalOrders: { $sum: 1 },
      totalSpent: { $sum: "$grandTotal" },
      minOrderValue: { $min: "$grandTotal" },
      maxOrderValue: { $max: "$grandTotal" },
      avgOrderValue: { $avg: "$grandTotal" }
    }
  },
  {
    $lookup: {
      from: "users",
      localField: "_id",
      foreignField: "_id",
      as: "user"
    }
  },
  { $unwind: "$user" },
  {
    $project: {
      _id: 0,
      userId: "$_id",
      userName: "$user.name",
      totalOrders: 1,
      totalSpent: 1,
      minOrderValue: 1,
      maxOrderValue: 1,
      avgOrderValue: 1
    }
  },
  { $sort: { totalSpent: -1 } }
]);
```

**output**

![image.png](PRACTICAL%203%20MongoDB%20E-Commerce/image%2012.png)

this query joins user information with sales statistics using `$lookup`

#### Query 4: Product Catalog View with Category Name

```json
db.products.aggregate([
  {
    $lookup: {
      from: "categories",
      localField: "categoryId",
      foreignField: "_id",
      as: "category"
    }
  },
  { $unwind: "$category" },
  {
    $project: {
      _id: 0,
      name: 1,
      price: 1,
      "attributes.brand": 1,
      "attributes.color": 1,
      categoryName: "$category.name"
    }
  },
  { $sort: { categoryName: 1, name: 1 } }
]).pretty()
```

**output**

![image.png](PRACTICAL%203%20MongoDB%20E-Commerce/image%2013.png)

it shows and help us understand catalog design and joining collections.

### PART 4 : Query Performance Optimization

#### Index 1: orders by user and date

```json
db.orders.createIndex(
  { userId: 1, createdAt: -1 },
  { name: "idx_orders_user_createdAt" }
);
```

**output**

![image.png](PRACTICAL%203%20MongoDB%20E-Commerce/image%2014.png)

#### **Index 2: orders by status and date**

```json
db.orders.createIndex(
  { status: 1, createdAt: -1, grandTotal: 1 },
  { name: "idx_orders_status_createdAt_grandTotal" }
);
```

**output**

![image.png](PRACTICAL%203%20MongoDB%20E-Commerce/image%2015.png)

testing the indexes applied

```json
db.products.find({ categoryId: electronicsId }).sort({ price: 1 });
```

![image.png](PRACTICAL%203%20MongoDB%20E-Commerce/image%2016.png)

#### **Index 3: products by category and price**

This query decreases the time to find the products

```json
db.products.createIndex(
  { categoryId: 1, price: 1 },
  { name: "idx_products_category_price" }
);
```

**output**

![image.png](PRACTICAL%203%20MongoDB%20E-Commerce/image%2017.png)

```json
db.products
  .find(
    { $text: { $search: "wireless keyboard" } },
    { score: { $meta: "textScore" }, name: 1, price: 1 }
  )
  .sort({ score: { $meta: "textScore" } });
```

**output**

![image.png](PRACTICAL%203%20MongoDB%20E-Commerce/image%2018.png)

#### **Index 4: text index for product search**

```json
db.products.createIndex(
  { name: "text", tags: "text" },
  {
    name: "idx_products_text",
    weights: { name: 10, tags: 5 }
  }
);
```

**output**

![image.png](PRACTICAL%203%20MongoDB%20E-Commerce/image%2019.png)

**example search**

```json
db.products
  .find(
    { $text: { $search: "wireless keyboard" } },
    { score: { $meta: "textScore" }, name: 1, price: 1 }
  )
  .sort({ score: { $meta: "textScore" } });
```

output

![image.png](PRACTICAL%203%20MongoDB%20E-Commerce/image%2020.png)

### PART 5: Use `explain()` to show optimization

#### Query 1: Run a Query Without an Index

```json
db.orders.find(
  { status: "PAID", createdAt: { $gte: new Date("2026-04-19") } }
).sort({ createdAt: -1 }).explain("executionStats");
```

output

![image.png](PRACTICAL%203%20MongoDB%20E-Commerce/image%2021.png)

![image.png](PRACTICAL%203%20MongoDB%20E-Commerce/image%2022.png)

- before indexing, MongoDB  scans the collection
- after creating a suitable index, the query should use `IXSCAN`
- this reduces document examination and improves query efficiency

#### Query 4: Create an Index

```json
db.orders.createIndex(
  { status: 1, createdAt: -1 },
  { name: "idx_orders_status_createdAt" }
);
```

Re‑run `explain()`

```json
db.orders.find(
  { status: "PAID", createdAt: { $gte: new Date("2026-04-19") } }
).sort({ createdAt: -1 }).explain("executionStats");
```

output 

![image.png](PRACTICAL%203%20MongoDB%20E-Commerce/image%2023.png)

![image.png](PRACTICAL%203%20MongoDB%20E-Commerce/image%2024.png)

![image.png](PRACTICAL%203%20MongoDB%20E-Commerce/image%2025.png)

### PART 6: Additional useful Queries

#### Aggregation with Index‑Friendly `$match` and `$sort`

```json
db.orders.aggregate([
  {
    $match: {
      status: "PAID",
      createdAt: { $gte: new Date("2026-04-19") }
    }
  },
  { $sort: { createdAt: -1 } },
  {
    $project: {
      _id: 0,
      userId: 1,
      createdAt: 1,
      grandTotal: 1,
      itemCount: { $size: "$items" }
    }
  },
  { $limit: 20 }
]);
```

output

![image.png](PRACTICAL%203%20MongoDB%20E-Commerce/image%2026.png)

### Suggested Lab Exercises (Hands‑On Tasks)

#### **EXERCISE 1: Inventory Monitoring (Low Stock)**

Aim: Find products where stock is low (e.g., < 100)

![image.png](PRACTICAL%203%20MongoDB%20E-Commerce/image%2027.png)

after running the pipeline 

![image.png](PRACTICAL%203%20MongoDB%20E-Commerce/image%2028.png)

The query selects products which have low stock levels and arranges them in a sequence from lowest to highest stock. The system enables you to detect products which need restocking while demonstrating the inventory management capabilities of MongoDB.

#### **EXERCISE 2: Customer Segmentation**

![image.png](PRACTICAL%203%20MongoDB%20E-Commerce/image%2029.png)

after running the pipeline 

![image.png](PRACTICAL%203%20MongoDB%20E-Commerce/image%2030.png)

This aggregation groups customers based on total spending and classifies them into segments. This is useful for marketing strategies and customer targeting.

#### EXERCISE 3: Product Search (Text Index)

AIM: Enable keyword search

![image.png](PRACTICAL%203%20MongoDB%20E-Commerce/image%2031.png)

![image.png](PRACTICAL%203%20MongoDB%20E-Commerce/image%2032.png)

A text index enables efficient keyword-based searching across product names and tags, improving user experience in e-commerce platforms.

#### EXERCISE 4: Index Performance Comparison

Show difference between indexed vs non-indexed query

Before adding indexing 

![image.png](PRACTICAL%203%20MongoDB%20E-Commerce/image%2033.png)

After adding indexing 

![image.png](PRACTICAL%203%20MongoDB%20E-Commerce/image%2034.png)

![image.png](PRACTICAL%203%20MongoDB%20E-Commerce/image%2035.png)

Before indexing, MongoDB performs a full collection scan (COLLSCAN), which is inefficient. After creating an index, the query uses an index scan (IXSCAN), significantly improving performance.

#### **EXERCISE 5: Order Size Analysis**

Aim: Find number of items in each order

![image.png](PRACTICAL%203%20MongoDB%20E-Commerce/image%2036.png)

![image.png](PRACTICAL%203%20MongoDB%20E-Commerce/image%2037.png)

This query calculates the number of items in each order, providing insights into customer purchasing behavior.

### Conclusion

The practical demonstration proved the successful development of a simplified e-commerce platform which used MongoDB to showcase the document-oriented database method's ability to handle actual business requirements. The schema was designed following a query-first approach, ensuring that data structures aligned with common access patterns such as order retrieval, product search, and sales analytics.

The user and product and category and order collections were established and populated with authentic sample data through the MongoDB Compass tool. The design achieved equilibrium through the combination of embedding and referencing methods which enabled order items to be embedded for fast reading while products and users needed to be referenced for data consistency and system expansion.

The practical further explored the aggregation framework, which proved to be a powerful tool for performing complex analytical queries. The database capabilities of MongoDB enable users to execute business intelligence operations through daily sales calculations and top product identification and customer spending analysis.

The process of indexing was used to enhance the performance of database queries. The creation of suitable indexes together with execution plan assessment through the explain() function, resulted in substantial improvements to query performance, which eliminated the requirement for complete collection scans. The study demonstrated that proper index design according to query patterns, serves as a critical factor for achieving peak database performance.

The proposed laboratory activities extended their practical application through the introduction of advanced use cases which included inventory monitoring and customer segmentation and text-based product search. The exercises demonstrated MongoDB's capability to solve actual e-commerce problems through its flexible application across different business situations.

The practical session taught students about NoSQL database design principles and aggregation-based analytics methods and performance optimization techniques. The study showed that MongoDB enables developers to create applications which operate at high performance while remaining flexible and scalable throughout contemporary data-driven environments.