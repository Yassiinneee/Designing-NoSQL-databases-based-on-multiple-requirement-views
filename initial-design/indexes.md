# MongoDB Indexes

## 1. Users Collection

### Email Index

```json
{
  "email": 1
}
```

**Purpose:**
Allows fast lookup of users by email address. This is useful for login, account verification, and retrieving a user's profile.

---

## 2. Products Collection

### Category Index

```json
{
  "category": 1
}
```

**Purpose:**
Improves queries that retrieve products belonging to a specific category.

Example:

```text
Find all products where category = "Computers"
```

### Brand Index

```json
{
  "brand": 1
}
```

**Purpose:**
Improves product filtering by brand.

Example:

```text
Find all products where brand = "Lenovo"
```

### Price Index

```json
{
  "price": 1
}
```

**Purpose:**
Improves queries that filter or sort products by price.

Example:

```text
Find products between 100 TND and 1000 TND
```

### Text Search Index

```json
{
  "name": "text",
  "description": "text",
  "tags": "text"
}
```

**Purpose:**
Supports text-based product searches using the product name, description, and tags.

Example:

```text
Search for "wireless mouse"
```

---

## 3. Orders Collection

### Customer Orders Index

```json
{
  "userId": 1,
  "createdAt": -1
}
```

**Purpose:**
Allows fast retrieval of a customer's orders, with the most recent orders appearing first.

Example:

```text
Find all orders for user USR001
sorted by creation date
```

### Order Status Index

```json
{
  "status": 1,
  "createdAt": -1
}
```

**Purpose:**
Improves queries for orders based on their status and creation date.

Example:

```text
Find all processing orders
sorted by newest first
```

### Creation Date Index

```json
{
  "createdAt": -1
}
```

**Purpose:**
Allows efficient retrieval and sorting of recent orders.

---

## Index Design Summary

| Collection | Index                      | Purpose                 |
| ---------- | -------------------------- | ----------------------- |
| users      | `email: 1`                 | Fast user lookup        |
| products   | `category: 1`              | Category filtering      |
| products   | `brand: 1`                 | Brand filtering         |
| products   | `price: 1`                 | Price filtering/sorting |
| products   | text index                 | Product search          |
| orders     | `userId: 1, createdAt: -1` | Customer order history  |
| orders     | `status: 1, createdAt: -1` | Order status queries    |
| orders     | `createdAt: -1`            | Recent orders           |

## Scalability Considerations

Indexes improve read performance by allowing MongoDB to locate documents efficiently without scanning the entire collection. However, indexes also consume storage and can increase write overhead because indexes must be updated when documents are inserted or modified.

For a high-traffic e-commerce system, indexes should therefore be created according to actual query patterns rather than indexing every field.
