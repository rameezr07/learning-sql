## PIVOT / UNPIVOT / UNNEST

---

# 🔄 1. PIVOT (Rows → Columns)

👉 One-liner:

> “Pivot converts row values into columns using aggregation.”

---

## 🧠 When do we use it?

* Reporting (monthly sales per region)
* Converting categorical values into columns
* Making data dashboard-friendly

---

## 📊 Example Data

```
sales
-------------------------
region | month | revenue
-------------------------
East   | Jan   | 100
East   | Feb   | 150
West   | Jan   | 200
West   | Feb   | 250
```

---

## ✅ PIVOT Query (Standard SQL style varies by DB)

### 🔹 Using CASE (works everywhere)

```sql
SELECT 
    region,
    SUM(CASE WHEN month = 'Jan' THEN revenue END) AS Jan,
    SUM(CASE WHEN month = 'Feb' THEN revenue END) AS Feb
FROM sales
GROUP BY region;
```

---

### 🔹 Using PIVOT (e.g., in BigQuery / SQL Server)

```sql
SELECT *
FROM sales
PIVOT (
    SUM(revenue)
    FOR month IN ('Jan', 'Feb')
);
```

---

## 📈 Output

```
region | Jan | Feb
------------------
East   | 100 | 150
West   | 200 | 250
```

---

### 🔥 Interview Tip

> “Pivot requires aggregation because multiple rows can map to one column.”

---

# 🔁 2. UNPIVOT (Columns → Rows)

👉 Opposite of pivot.

---

## 🧠 When do we use it?

* Normalize wide data
* Prepare data for analysis or ML
* Convert reports → raw format

---

## 📊 Input

```
region | Jan | Feb
------------------
East   | 100 | 150
West   | 200 | 250
```

---

## ✅ UNPIVOT Query

### 🔹 Standard SQL approach (UNION ALL)

```sql
SELECT region, 'Jan' AS month, Jan AS revenue FROM sales
UNION ALL
SELECT region, 'Feb', Feb FROM sales;
```

---

### 🔹 Using UNPIVOT (if supported)

```sql
SELECT *
FROM sales
UNPIVOT (
    revenue FOR month IN (Jan, Feb)
);
```

---

## 📈 Output

```
region | month | revenue
------------------------
East   | Jan   | 100
East   | Feb   | 150
West   | Jan   | 200
West   | Feb   | 250
```

---

### 🔥 Interview Tip

> “Unpivot is useful for transforming wide tables into long format for analytics.”

---

# 📦 3. UNNEST (Very important for Data Engineering)

This is BIG in:

* BigQuery
* Snowflake (FLATTEN)
* Spark (explode)

---

👉 One-liner:

> “UNNEST expands arrays or nested data into rows.”

---

## 🧠 Why this matters?

Because **real-world data = JSON / nested data**

---

## 📊 Example (Nested Data)

```json
{
  "order_id": 1,
  "items": ["apple", "banana", "milk"]
}
```

---

## ✅ Query (BigQuery style)

```sql
SELECT order_id, item
FROM orders,
UNNEST(items) AS item;
```

---

## 📈 Output

```
order_id | item
----------------
1        | apple
1        | banana
1        | milk
```

---

# 🔥 4. UNNEST with STRUCT (real DE use case)

---

## 📊 Example

```json
{
  "order_id": 1,
  "items": [
    {"product": "apple", "price": 10},
    {"product": "banana", "price": 5}
  ]
}
```

---

## ✅ Query

```sql
SELECT 
    order_id,
    item.product,
    item.price
FROM orders,
UNNEST(items) AS item;
```

---

---

# ⚠️ 5. CROSS JOIN behavior (IMPORTANT)

👉 UNNEST behaves like a **CROSS JOIN**

* 1 row → many rows
* Can explode data size

---

### 🔥 Interview line:

> “UNNEST increases row count and must be used carefully to avoid data explosion.”

---

# 🧩 6. Combine UNNEST + Aggregation

---

## ❓ “Total items per order”

```sql
SELECT 
    order_id,
    COUNT(*) AS item_count
FROM orders,
UNNEST(items) AS item
GROUP BY order_id;
```

---

---

# ⚠️ 7. Common Mistakes

---

### ❌ Forgetting aggregation in PIVOT

---

### ❌ Hardcoding too many columns

---

### ❌ UNNEST explosion

```sql
FROM table1,
UNNEST(array1),
UNNEST(array2)
```

👉 ❌ creates Cartesian explosion

---

### ❌ Losing original rows

👉 Use `LEFT JOIN UNNEST()` if needed

---

# 🔥 8. LEFT JOIN UNNEST (Important)

---

## ❓ “Keep rows even if array is empty”

```sql
SELECT *
FROM orders
LEFT JOIN UNNEST(items) AS item;
```

---

👉 Keeps rows where `items` is NULL/empty

---

# ⚖️ 9. Quick Comparison

| Operation | Input         | Output    |
| --------- | ------------- | --------- |
| PIVOT     | Rows          | Columns   |
| UNPIVOT   | Columns       | Rows      |
| UNNEST    | Nested arrays | Flat rows |

---

# 🎯 10. Real Interview Combo Question

---

## ❓ “Total revenue per product from nested order data”

```sql
SELECT 
    item.product,
    SUM(item.price) AS total_revenue
FROM orders,
UNNEST(items) AS item
GROUP BY item.product;
```

---

👉 Combines:

* UNNEST
* GROUP BY
  💥 Very real-world DE problem

---

# 🧠 Final Mental Model

Think:

* **PIVOT** → reshape for reporting
* **UNPIVOT** → normalize data
* **UNNEST** → flatten nested structures

---
