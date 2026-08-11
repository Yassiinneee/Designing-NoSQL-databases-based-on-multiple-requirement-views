# Designing NoSQL Databases Based on Multiple Requirement Views

<p align="center"> <img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/mongodb/mongodb-original.svg" width="70" alt="MongoDB"/> <img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/json/json-original.svg" width="70" alt="JSON"/> </p>

<p align="center"> <strong>MongoDB • NoSQL • JSON • JSON Schema • Database Architecture</strong> </p>

## Overview

This project focuses on the **design, evaluation, and refactoring of a MongoDB-based NoSQL database for an e-commerce platform**.

The objective is to demonstrate how a NoSQL database can be designed according to different application requirements, particularly:

* High-volume transactional workloads
* Product browsing and search
* Customer order management
* Query performance
* Scalability
* Availability and fault tolerance
* Analytical workloads
* Denormalization
* Data consistency
* Horizontal scaling through sharding

The project begins with an **initial MongoDB document model** optimized for operational e-commerce queries and progressively evolves into a **refactored architecture** capable of supporting large-scale analytics, high availability, and horizontal scalability.

---

## Project Objectives

The main objectives of this project are to:

1. Design a document-oriented NoSQL database for an e-commerce application.
2. Model users, products, and orders using MongoDB documents.
3. Identify the main application query patterns.
4. Design appropriate MongoDB indexes.
5. Apply denormalization where it improves read performance.
6. Separate operational and analytical workloads.
7. Introduce pre-aggregated analytics collections.
8. Design a scalable MongoDB architecture.
9. Address consistency and availability requirements.
10. Evaluate the trade-offs introduced by the refactored design.

---

## Technology

| Technology               | Purpose                                  |
| ------------------------ | ---------------------------------------- |
| **MongoDB**              | Primary NoSQL database                   |
| **JSON**                 | Data representation and document samples |
| **MongoDB Indexes**      | Query optimization                       |
| **MongoDB Sharding**     | Horizontal scalability                   |
| **MongoDB Replica Sets** | High availability and fault tolerance    |
| **JSON Schema**          | Document structure validation            |
| **Markdown**             | Technical documentation                  |

---

# 1. Project Structure

The repository is organized into two main database designs:

```text
Designing-NoSQL-databases-based-on-multiple-requirement-views-main/
│
├── initial-design/
│   ├── users.json
│   ├── products.json
│   ├── orders.json
│   └── indexes.md
│
├── refactored-design/
│   ├── users.json
│   ├── products.json
│   ├── orders.json
│   ├── sales_analytics.json
│   ├── product_analytics.json
│   └── architecture.md
│
├── Screenshots/
│   ├── Initial design.png
│   ├── Refactored design.png
│   ├── Schema-Initial design.png
│   └── Schema-refactored design.png
│
├── schema-nosql-ecommerce-design-initial-design-standardJSON.json
├── schema-nosql-ecommerce-design-refactored-design-standardJSON.json
│
├── reflection-report.md
│
└── README.md
```

---

# 2. Initial Database Design

The initial design provides the operational foundation of the e-commerce system.

It contains three primary collections:

```text
ecommerce_nosql
│
├── users
├── products
└── orders
```

These collections support the application's core transactional operations.

### Users

The `users` collection stores customer information such as:

* User identifier
* Name
* Email
* Address
* Account creation date

Example structure:

```json
{
  "_id": "USR001",
  "name": "Yassine Kaltoum",
  "email": "yassine@example.com",
  "address": {
    "street": "123 Main Street",
    "city": "Sousse",
    "country": "Tunisia"
  },
  "createdAt": "2026-08-11T10:00:00Z"
}
```

### Products

The `products` collection represents the product catalog.

It contains:

* Product identifier
* Product name
* Description
* Category
* Brand
* Price
* Currency
* Stock quantity
* Tags
* Creation date

The document structure is designed to support product browsing, filtering, sorting, and text-based search.

### Orders

The `orders` collection represents customer purchases.

An order contains:

* Customer information
* User identifier
* Purchased products
* Quantities
* Unit prices
* Line totals
* Total order amount
* Order status
* Delivery information
* Creation and update timestamps

A major NoSQL design decision is the embedding of order items directly inside the order document.

This allows the complete order to be retrieved using a single document query.

---

# 3. Initial Data Modeling Strategy

The initial design follows the MongoDB document-oriented model.

Instead of splitting an order into multiple relational tables, information that is frequently accessed together is stored together.

For example:

```text
Order
│
├── Customer
│
├── Items
│   ├── Product
│   ├── Quantity
│   ├── Unit Price
│   └── Total
│
├── Total Amount
│
└── Delivery
```

This approach reduces the need for joins and is particularly appropriate for queries such as:

```text
Get an order by ID
Get all information about an order
Display the customer's order history
Display purchased products
Display delivery status
```

---

# 4. MongoDB Index Strategy

Indexes were designed according to expected query patterns rather than indexing every available field.

The initial design includes indexes for:

### Users

```json
{
  "email": 1
}
```

This supports fast user lookup by email.

### Products

```json
{
  "category": 1
}
```

Supports category filtering.

```json
{
  "brand": 1
}
```

Supports brand filtering.

```json
{
  "price": 1
}
```

Supports price filtering and sorting.

The product catalog also uses a text index:

```json
{
  "name": "text",
  "description": "text",
  "tags": "text"
}
```

This supports product searches based on textual content.

### Orders

Customer order history:

```json
{
  "userId": 1,
  "createdAt": -1
}
```

Order status queries:

```json
{
  "status": 1,
  "createdAt": -1
}
```

Recent order queries:

```json
{
  "createdAt": -1
}
```

The complete index strategy is documented in:

```text
initial-design/indexes.md
```

---

# 5. JSON Schema Validation

The project includes JSON Schema definitions for both database designs.

### Initial Schema

```text
schema-nosql-ecommerce-design-initial-design-standardJSON.json
```

### Refactored Schema

```text
schema-nosql-ecommerce-design-refactored-design-standardJSON.json
```

These schemas describe the expected document structure and provide a formal representation of the database model.

The schemas define properties such as:

* `_id`
* `userId`
* `email`
* `products`
* `items`
* `price`
* `quantity`
* `totalAmount`
* `status`
* `delivery`
* `createdAt`
* `updatedAt`

This provides an additional layer of structural validation for the NoSQL document model.

---

# 6. Refactored Database Design

As the e-commerce platform grows, the initial design becomes insufficient for large-scale analytical workloads.

The refactored architecture introduces two additional collections:

```text
ecommerce_nosql
│
├── users
├── products
├── orders
├── sales_analytics
└── product_analytics
```

The operational collections remain:

* `users`
* `products`
* `orders`

The new analytical collections are:

* `sales_analytics`
* `product_analytics`

---

# 7. Denormalization

The refactored design applies **controlled denormalization**.

Product information is intentionally duplicated inside order items.

For example:

```json
{
  "productId": "PROD001",
  "name": "Laptop Lenovo ThinkPad",
  "category": "Computers",
  "quantity": 1,
  "unitPrice": 3200,
  "total": 3200
}
```

The product's name and category are stored directly inside the order.

This avoids repeatedly retrieving product information when processing historical orders or generating analytical information.

### Benefits

* Faster reads
* Fewer database operations
* Reduced dependency on joins
* Better performance for historical order access
* Greater query locality

### Trade-off

The same information may exist in more than one document.

Therefore, denormalization increases:

* Storage requirements
* Data maintenance complexity
* Potential synchronization requirements

This is an intentional trade-off in document-oriented database design.

---

# 8. Sales Analytics

The `sales_analytics` collection provides **pre-aggregated sales information**.

Example:

```json
{
  "_id": "2026-08-11",
  "date": "2026-08-11",
  "totalOrders": 12540,
  "totalRevenue": 4587000,
  "currency": "TND",
  "totalProductsSold": 18950,
  "averageOrderValue": 365.94
}
```

Additional information such as top-performing categories is also stored.

Instead of scanning millions of orders whenever a dashboard requests daily sales statistics, the application can directly query the corresponding analytical document.

### Example workload

Without pre-aggregation:

```text
orders
   ↓
Scan large dataset
   ↓
Aggregation pipeline
   ↓
Calculate revenue
   ↓
Calculate orders
   ↓
Calculate products sold
   ↓
Return dashboard
```

With pre-aggregation:

```text
sales_analytics
       ↓
Pre-calculated metrics
       ↓
Return dashboard
```

This significantly reduces the computational workload for frequently requested analytical queries.

---

# 9. Product Analytics

The `product_analytics` collection stores aggregated product performance.

Example:

```json
{
  "_id": "PROD001",
  "productId": "PROD001",
  "productName": "Laptop Lenovo ThinkPad",
  "category": "Computers",
  "totalUnitsSold": 15820,
  "totalRevenue": 50624000,
  "numberOfOrders": 12350,
  "averagePrice": 3200
}
```

This collection supports queries such as:

* Which products sell the most?
* Which products generate the most revenue?
* How many orders contain a particular product?
* Which product categories perform best?

These queries can be answered without repeatedly scanning the entire `orders` collection.

---

# 10. Analytics Architecture

The two analytical collections should be considered **derived data** rather than the primary source of transactional truth.

The operational flow is:

```text
                 Customer Order
                       │
                       ▼
                    orders
                       │
                       ▼
              Analytics Processing
                 /           \
                /             \
               ▼               ▼
     sales_analytics    product_analytics
```

The `orders` collection remains the source of truth for transactional information.

The analytics collections provide optimized read models for analytical workloads.

This follows an important NoSQL principle:

> Data should be modeled according to the queries and workloads the system must support.

---

# 11. Scalability Strategy

The refactored architecture is designed for horizontal scalability.

The `orders` collection is expected to become one of the largest collections in the system and is therefore a strong candidate for sharding.

A possible shard-key strategy is based on:

```text
userId + createdAt
```

Conceptually:

```text
                 MongoDB Cluster
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
     Shard 1        Shard 2        Shard 3
        │              │              │
     Orders         Orders         Orders
```

Sharding distributes data and workload across multiple nodes.

As traffic and data volume increase, additional shards can be introduced.

The final shard key should be validated against actual production query patterns and data distribution before deployment.

---

# 12. High Availability and Replication

The production architecture can use a MongoDB replica set.

A replica set contains multiple copies of the database:

```text
                 MongoDB Replica Set
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
       Primary       Secondary      Secondary
          │              │              │
          └──────────────┼──────────────┘
                    Replication
```

The primary node handles normal write operations.

Secondary nodes maintain replicated copies of the data.

If the primary becomes unavailable, an eligible secondary can be elected as the new primary.

Replication provides:

* High availability
* Fault tolerance
* Data redundancy
* Automatic failover
* Improved recovery capabilities

---

# 13. Consistency Strategy

Not every workload requires the same consistency model.

The architecture therefore differentiates between **transactional data** and **analytical data**.

## Transactional Workloads

The `orders` collection requires strong consistency for critical operations.

Examples include:

* Order creation
* Payment confirmation
* Stock updates
* Order status changes
* Delivery status

Incorrect or stale transactional information could lead to business-critical errors.

Therefore, these operations should prioritize correctness and consistency.

## Analytical Workloads

Analytics can tolerate a small delay.

For example:

```text
Order created
     ↓
Analytics processing
     ↓
Analytics document updated
```

A dashboard does not necessarily need to reflect an order immediately.

Therefore, analytical data can use an **eventual consistency approach**, prioritizing:

* Read performance
* Availability
* Scalability

over immediate synchronization.

---

# 14. Performance Optimization

The refactored architecture combines several NoSQL optimization techniques.

### Indexing

Indexes accelerate frequently executed queries.

### Denormalization

Frequently accessed information is stored together to reduce database operations.

### Pre-aggregation

Frequently requested analytical results are calculated in advance.

### Sharding

Large datasets and workloads can be distributed across multiple nodes.

### Replication

Multiple database nodes provide availability and fault tolerance.

Together, these mechanisms allow the database to support both operational and analytical workloads more efficiently.

---

# 15. Initial vs Refactored Architecture

| Aspect                | Initial Design | Refactored Design |
| --------------------- | -------------- | ----------------- |
| Users                 | Yes            | Yes               |
| Products              | Yes            | Yes               |
| Orders                | Yes            | Yes               |
| Product analytics     | No             | Yes               |
| Sales analytics       | No             | Yes               |
| Denormalization       | Limited        | Controlled        |
| Indexing              | Yes            | Yes               |
| Pre-aggregation       | No             | Yes               |
| Sharding strategy     | Not defined    | Defined           |
| Replica set           | Not defined    | Defined           |
| High availability     | Limited        | Designed          |
| Large-scale analytics | Limited        | Optimized         |
| Scalability           | Basic          | Horizontal        |

The refactored architecture therefore extends the original operational model without unnecessarily redesigning collections that already satisfy their primary requirements.

---

# 16. Architecture Overview

The complete architecture can be represented as:

```text
                    E-COMMERCE APPLICATION
                              │
                              ▼
                       MongoDB Cluster
                              │
          ┌───────────────────┼───────────────────┐
          │                   │                   │
          ▼                   ▼                   ▼
        users              products             orders
                                                  │
                                                  ▼
                                      Analytics Processing
                                          /           \
                                         /             \
                                        ▼               ▼
                              sales_analytics    product_analytics


                  MongoDB Infrastructure
                  ┌─────────────────────────┐
                  │                         │
                  │       Sharding          │
                  │    ┌────┬────┬────┐     │
                  │    │ S1 │ S2 │ S3 │     │
                  │    └────┴────┴────┘     │
                  │                         │
                  │      Replication        │
                  │  Primary + Secondaries  │
                  │                         │
                  └─────────────────────────┘
```

---

# 17. Design Trade-offs

The refactored architecture improves scalability and performance, but these improvements introduce additional complexity.

## Advantages

* Better horizontal scalability
* Improved analytical performance
* Reduced workload on the operational database
* Higher availability
* Fault tolerance
* Faster dashboard queries
* Efficient large-volume data processing
* Query-oriented document modeling

## Disadvantages

* Increased storage requirements
* More complex data maintenance
* Additional infrastructure requirements
* More complex monitoring
* Analytics synchronization overhead
* Potential temporary inconsistency in derived analytics data
* Shard-key selection requires careful planning

The architecture therefore follows a deliberate trade-off:

```text
More complexity + more storage
                ↓
Better scalability + availability + performance
```

---

# 18. Screenshots and Visual Documentation

The repository contains visual representations of both database designs.

### Initial Design

```text
Screenshots/Initial design.png
Screenshots/Schema-Initial design.png
```

### Refactored Design

```text
Screenshots/Refactored design.png
Screenshots/Schema-refactored design.png
```

These diagrams provide a visual comparison between the original document model and the optimized architecture.

---

# 19. Reflection

The main challenge in this project was balancing transactional performance with the additional requirements introduced by large-scale analytics, availability, and scalability.

The initial design focused primarily on operational workloads such as product browsing, product search, customer management, and order processing. Embedding related information inside order documents reduced the need for joins and made common transactional queries efficient.

As analytical requirements increased, directly executing complex aggregation pipelines against a very large `orders` collection would potentially affect the performance of the operational application.

The solution was to introduce `sales_analytics` and `product_analytics` as pre-aggregated read models.

The architecture was further enhanced through denormalization, replication, and a sharding strategy. These mechanisms improve scalability and availability while introducing additional storage and operational complexity.

The final design therefore represents a balance between:

```text
Consistency
Availability
Performance
Scalability
Storage
Complexity
```

Critical transactional operations prioritize correctness and consistency, while analytical workloads can tolerate a small delay in exchange for improved performance and availability.

The complete reflection is available in:

```text
reflection-report.md
```

---

# 20. Key NoSQL Design Principles Demonstrated

This project demonstrates several important NoSQL design principles:

### 1. Model for Access Patterns

MongoDB documents are designed around how the application retrieves and uses data.

### 2. Embed When Data Belongs Together

Order items are embedded inside orders because they are normally accessed together.

### 3. Denormalize When Appropriate

Frequently accessed product information is duplicated inside order documents to reduce lookup operations.

### 4. Index According to Queries

Indexes are created based on actual query requirements rather than indiscriminately indexing every field.

### 5. Pre-aggregate Expensive Queries

Frequently requested analytical metrics are stored in dedicated collections.

### 6. Separate Workloads

Operational transactions and analytical workloads are represented by different data models.

### 7. Scale Horizontally

Sharding provides a path toward distributing large workloads across multiple nodes.

### 8. Design for Failure

Replication provides redundancy and supports high availability.

### 9. Accept Controlled Trade-offs

NoSQL architecture often exchanges normalization and strict consistency for performance, scalability, and availability where appropriate.

---

# 21. Conclusion

This project demonstrates the evolution of an e-commerce MongoDB database from a basic operational document model into a scalable architecture designed around multiple requirement views.

The initial design provides an efficient foundation for:

* Customer management
* Product catalog operations
* Order processing
* Product search
* Order history

The refactored architecture extends this foundation by introducing:

* Controlled denormalization
* Pre-aggregated sales analytics
* Product performance analytics
* Query-oriented data models
* Sharding for horizontal scalability
* Replica sets for high availability
* Workload-specific consistency strategies

The final architecture demonstrates how MongoDB can be designed not simply as a replacement for a relational database, but as a **query-oriented, scalable NoSQL platform** where data structures are intentionally optimized for application access patterns.

---

## Repository Deliverables

| Deliverable                                | Description                           |
| ------------------------------------------ | ------------------------------------- |
| `initial-design/users.json`                | Initial users documents               |
| `initial-design/products.json`             | Initial product catalog               |
| `initial-design/orders.json`               | Initial order model                   |
| `initial-design/indexes.md`                | MongoDB indexing strategy             |
| `refactored-design/users.json`             | Refactored users documents            |
| `refactored-design/products.json`          | Refactored product catalog            |
| `refactored-design/orders.json`            | Refactored order model                |
| `refactored-design/sales_analytics.json`   | Pre-aggregated sales metrics          |
| `refactored-design/product_analytics.json` | Product performance metrics           |
| `refactored-design/architecture.md`        | Refactored architecture documentation |
| `schema-*-standardJSON.json`               | JSON Schema definitions               |
| `reflection-report.md`                     | Design reflection and trade-offs      |
| `Screenshots/`                             | Database and schema diagrams          |

---

## Author

**Yassine Kaltoum**

**Project:** Designing NoSQL Databases Based on Multiple Requirement Views

**Database:** MongoDB

**Domain:** E-Commerce
