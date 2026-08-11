# Refactored MongoDB Architecture

## 1. Overview

The initial e-commerce database was designed to support product browsing, product search, customer management, and order processing. As the application grows, new requirements introduce the need for large-scale analytics, high availability, scalability, and partition tolerance.

The refactored architecture continues to use MongoDB as the main NoSQL database but introduces denormalization, pre-aggregated analytics, replication, and a sharding strategy.

---

## 2. Database Structure

The refactored database contains the following collections:

```text
ecommerce_nosql
│
├── users
├── products
├── orders
├── sales_analytics
└── product_analytics
```

The `users` and `products` collections remain largely unchanged because the new requirements do not require a different document structure for these entities.

The `orders` collection continues to store complete order information, including customer details, purchased products, prices, quantities, and delivery status.

Two new collections are introduced for analytics:

* `sales_analytics`
* `product_analytics`

---

## 3. Denormalization

The refactored design uses denormalization to improve query performance.

The `orders` documents contain information such as:

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

Some product information is intentionally duplicated inside orders.

This avoids repeatedly looking up product information when processing orders or generating analytical data.

Denormalization increases storage requirements, but it improves read performance and reduces the need for joins or multiple database operations.

---

## 4. Analytics Collections

### sales_analytics

The `sales_analytics` collection stores pre-aggregated sales information.

Example:

```json
{
  "_id": "2026-08-11",
  "date": "2026-08-11",
  "totalOrders": 12540,
  "totalRevenue": 4587000,
  "totalProductsSold": 18950,
  "averageOrderValue": 365.94
}
```

This allows the application to retrieve daily sales statistics without scanning the entire `orders` collection.

### product_analytics

The `product_analytics` collection stores aggregated information about product performance.

Example:

```json
{
  "_id": "PROD001",
  "productId": "PROD001",
  "productName": "Laptop Lenovo ThinkPad",
  "category": "Computers",
  "totalUnitsSold": 15820,
  "totalRevenue": 50624000,
  "numberOfOrders": 12350
}
```

This makes queries such as "Which products sell the most?" much faster.

---

## 5. Sharding Strategy

To support thousands of transactions per second and large amounts of data, the production MongoDB deployment can use sharding.

The `orders` collection is a strong candidate for horizontal scaling because it is expected to contain a large number of documents.

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

Sharding distributes data and requests across multiple servers instead of placing the entire workload on one server.

This improves horizontal scalability and allows the system to handle larger workloads.

The exact shard key should be validated using production query patterns and data distribution before deployment.

---

## 6. Replication and High Availability

The production database should use a MongoDB replica set.

A replica set contains multiple copies of the data:

```text
                 MongoDB Replica Set
                        │
          ┌─────────────┼─────────────┐
          ▼             ▼             ▼
       Primary       Secondary      Secondary
          │             │             │
          └─────────────┼─────────────┘
                    Replication
```

The primary node handles normal write operations while secondary nodes maintain replicated copies.

If the primary node fails, another eligible node can be elected as the new primary.

Replication improves:

* High availability
* Fault tolerance
* Data redundancy
* Recovery from node failures

---

## 7. Consistency and Availability

Different parts of the application have different consistency requirements.

### Orders

Order processing requires strong consistency because incorrect order information could result in incorrect payments, stock levels, or delivery status.

Examples include:

```text
Payment confirmed
Order created
Order shipped
Order delivered
```

These operations should prioritize data correctness.

### Analytics

Analytical data can tolerate some delay.

For example, if a dashboard displays today's sales, it is generally acceptable for the analytics data to be updated a few seconds or minutes after an order is created.

Therefore, analytics can prioritize availability and performance over immediate consistency.

---

## 8. Query Performance

The refactored architecture improves query performance by combining indexes and pre-aggregated data.

Instead of calculating:

```text
Millions of orders
        ↓
Aggregation
        ↓
Calculate total revenue
        ↓
Return result
```

the application can query:

```text
sales_analytics
        ↓
Pre-calculated total revenue
        ↓
Return result
```

This significantly reduces the amount of data that must be processed for frequently requested analytical queries.

---

## 9. Scalability

The architecture supports horizontal scaling through sharding.

As the number of users and orders increases, additional database resources can be added to the cluster.

The architecture can therefore evolve from:

```text
Single MongoDB deployment
```

to:

```text
MongoDB Cluster
│
├── Shard 1
├── Shard 2
├── Shard 3
└── Additional shards when required
```

This is more suitable for an e-commerce application handling thousands of transactions per second.

---

## 10. Trade-offs

The refactored architecture provides significant performance and availability benefits, but it also introduces trade-offs.

### Advantages

* Better horizontal scalability
* Higher availability
* Improved fault tolerance
* Faster analytical queries
* Reduced workload on the orders collection
* Better support for large data volumes

### Disadvantages

* More complex architecture
* Additional storage due to denormalization
* Additional maintenance for analytics collections
* More complex deployment and monitoring
* Pre-aggregated data may not always be immediately up to date

---

## 11. Final Architecture

```text
                         E-COMMERCE APPLICATION
                                  │
                                  ▼
                           MongoDB Cluster
                                  │
              ┌───────────────────┼───────────────────┐
              │                   │                   │
              ▼                   ▼                   ▼
           users              products              orders
                                                      │
                                                      │
                                  ┌───────────────────┘
                                  ▼
                           Analytics Processing
                                  │
                       ┌──────────┴──────────┐
                       ▼                     ▼
                sales_analytics       product_analytics


             MongoDB Infrastructure
             ┌───────────────────────────────┐
             │                               │
             │       Sharding                │
             │   ┌────┬────┬────┐            │
             │   │ S1 │ S2 │ S3 │            │
             │   └────┴────┴────┘            │
             │                               │
             │       Replication             │
             │   Primary + Secondaries       │
             │                               │
             └───────────────────────────────┘
```

## Conclusion

The refactored MongoDB architecture extends the initial document-based design to support the company's growth. Denormalization and pre-aggregated analytics improve query performance, while sharding provides horizontal scalability and replication improves availability and fault tolerance.

The design accepts additional storage and architectural complexity in exchange for better performance, scalability, and availability. Critical transactional operations such as orders should prioritize consistency, while analytical workloads can tolerate some delay in exchange for better availability and query performance.
