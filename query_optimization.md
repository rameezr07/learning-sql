## Query Optimization

---

# 🧠 1. What is Query Optimization (in one line)

> “Query optimization is about reducing the amount of data scanned, moved, and processed.”

Everything you do should map back to that.

---

# ⚙️ 2. How SQL actually runs (mental model)

Execution (simplified):

```
FROM → JOIN → WHERE → GROUP BY → HAVING → SELECT → ORDER BY
```

👉 Optimization = **push work as early as possible** (especially filters).

---

# 🔍 3. What is EXPLAIN / EXPLAIN PLAN?

👉 It shows **how the database will execute your query**.

Think of it as:

> “The engine telling you its strategy.”

---

## Example

```sql
EXPLAIN
SELECT *
FROM orders
WHERE customer_id = 100;
```

---

## What you’ll see (varies by DB)

* Scan type (Seq Scan / Index Scan)
* Join type (Hash / Nested Loop / Merge)
* Estimated rows
* Cost

---

# 🧩 4. Key things to look for in EXPLAIN

---

## 🔹 1. Full Table Scan (🚨 warning sign)

```
Seq Scan on orders
```

👉 Means:

* DB scans entire table

---

## 🔹 2. Index Usage (✅ good)

```
Index Scan using idx_customer_id
```

👉 Faster lookup

---

## 🔹 3. Join Type (very important)

| Join Type   | When used       | Performance |
| ----------- | --------------- | ----------- |
| Nested Loop | small tables    | fast        |
| Hash Join   | large, no index | good        |
| Merge Join  | sorted data     | very fast   |

---

## 🔹 4. Rows & Cost

* Estimated rows vs actual rows (in ANALYZE)
* Big mismatch → bad statistics

---

# 🔥 5. Core Optimization Principles

---

## ✅ 1. Filter Early (push predicates)

---

### ❌ Bad

```sql
SELECT *
FROM orders
JOIN customers ON ...
WHERE orders.amount > 100;
```

---

### ✅ Better

```sql
SELECT *
FROM (
    SELECT * FROM orders WHERE amount > 100
) o
JOIN customers ON ...
```

👉 Reduce data before join

---

## ✅ 2. Avoid SELECT *

---

### ❌

```sql
SELECT * FROM huge_table;
```

👉 Reads unnecessary columns

---

### ✅

```sql
SELECT id, name FROM huge_table;
```

---

## ✅ 3. Use Indexes properly

---

### Index helps when:

```sql
WHERE column = value
JOIN on column
ORDER BY column
```

---

### ❌ Index not used:

```sql
WHERE YEAR(date_column) = 2024
```

👉 Function breaks index

---

### ✅ Fix:

```sql
WHERE date_column BETWEEN '2024-01-01' AND '2024-12-31'
```

---

## ✅ 4. Avoid functions on columns (BIG ONE)

---

```sql
LOWER(name) = 'john'   ❌
```

👉 index won’t work

---

### ✅

Store normalized data or use functional index

---

## ✅ 5. Reduce JOIN size

---

### ❌

Join raw tables directly

---

### ✅

Pre-aggregate

```sql
WITH order_totals AS (
    SELECT customer_id, SUM(amount) AS total
    FROM orders
    GROUP BY customer_id
)
SELECT *
FROM order_totals ot
JOIN customers c ON ...
```

---

## ✅ 6. Use EXISTS instead of JOIN (when needed)

---

### ❌

```sql
SELECT DISTINCT c.*
FROM customers c
JOIN orders o ON ...
```

---

### ✅

```sql
SELECT *
FROM customers c
WHERE EXISTS (
    SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id
);
```

👉 Stops early, avoids duplication

---

## ✅ 7. Avoid unnecessary DISTINCT

---

👉 It hides bad joins and is expensive

---

## ✅ 8. Partitioning (very important in DE)

---

👉 Tables split by:

* date
* region

---

### Benefit:

* scans only relevant partitions

---

# 📊 6. Join Optimization Tips

---

## 🔹 Join smaller table first (conceptually)

---

## 🔹 Ensure join keys are indexed

---

## 🔹 Avoid many-to-many explosion

---

# ⚠️ 7. Common Performance Killers

---

🚨 Full table scan on huge table
🚨 Cartesian join (missing join condition)
🚨 Functions on indexed columns
🚨 Too many joins
🚨 Data skew (one key dominating)
🚨 Overuse of CTEs (materialization in some DBs)

---

# 🔍 8. EXPLAIN ANALYZE (next level)

---

👉 Runs the query and shows actual stats

```sql
EXPLAIN ANALYZE SELECT ...
```

---

### Look for:

* Actual time vs estimated time
* Rows mismatch
* Bottlenecks

---

# 🧠 9. Real Example (thinking process)

---

## Query:

```sql
SELECT c.name, SUM(o.amount)
FROM customers c
JOIN orders o ON c.id = o.customer_id
GROUP BY c.name;
```

---

## Optimization thinking:

1. Is `orders.customer_id` indexed?
2. Can we pre-aggregate orders?
3. Are we scanning unnecessary columns?

---

### Optimized:

```sql
WITH order_totals AS (
    SELECT customer_id, SUM(amount) AS total
    FROM orders
    GROUP BY customer_id
)
SELECT c.name, ot.total
FROM customers c
JOIN order_totals ot ON c.id = ot.customer_id;
```

---

# 🎯 10. Interview-Level Insights (say these)

---

👉 “Indexes speed up reads but slow down writes.”

👉 “Query optimization is about minimizing data movement.”

👉 “EXPLAIN helps identify scan type, join strategy, and cost.”

👉 “Always check if the optimizer is using indexes.”

👉 “Pre-aggregation can significantly reduce join cost.”

---

# 🧠 Final Mental Model

Think like this:

> “How can I reduce rows, reduce columns, and reduce computation as early as possible?”

---

