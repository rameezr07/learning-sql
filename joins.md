## Joins

---

# 🧠 1. Quick refresher (but sharp)

👉 One-liner:

> “Joins combine rows from multiple tables based on a condition.”

---

# ⚙️ Types (just anchor quickly)

* `INNER JOIN` → matching only
* `LEFT JOIN` → all left + matches
* `RIGHT JOIN` → all right + matches
* `FULL JOIN` → everything

---

Now the important part 👇

---

# 🔥 2. Pattern 1: LEFT JOIN + NULL filter (ANTI JOIN)

---

## ❓ “Find customers who did NOT place any orders”

```sql
SELECT c.*
FROM customers c
LEFT JOIN orders o
    ON c.customer_id = o.customer_id
WHERE o.customer_id IS NULL;
```

---

### 🧠 Why it works:

* LEFT JOIN keeps all customers
* Non-matching → NULL
* Filter NULL → no orders

---

### 🔥 Interview line:

> “LEFT JOIN + IS NULL acts like an anti-join.”

---

# 🔁 3. Pattern 2: SELF JOIN (very important)

---

## ❓ “Find employees earning more than their manager”

```sql
SELECT e.name, e.salary, m.name AS manager_name
FROM employees e
JOIN employees m
    ON e.manager_id = m.emp_id
WHERE e.salary > m.salary;
```

---

### 🧠 Use cases:

* Hierarchies
* Comparisons within same table

---

# 📊 4. Pattern 3: JOIN + GROUP BY

---

## ❓ “Total orders per customer”

```sql
SELECT c.customer_id, c.name, COUNT(o.order_id) AS total_orders
FROM customers c
LEFT JOIN orders o
    ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.name;
```

---

### ⚠️ Important:

👉 LEFT JOIN ensures customers with 0 orders are included

---

# ⚠️ 5. Pattern 4: Duplicate explosion problem

This is where many candidates fail.

---

## ❗ Problem:

Joining tables with duplicates multiplies rows

---

### Example:

* Orders (1 row)
* Order_items (3 rows)

👉 Join → 3 rows (not 1)

---

### ❓ “Fix incorrect aggregation after join”

---

### ❌ Wrong

```sql
SELECT c.customer_id, SUM(o.amount)
FROM customers c
JOIN orders o
    ON c.customer_id = o.customer_id
JOIN order_items oi
    ON o.order_id = oi.order_id
GROUP BY c.customer_id;
```

👉 Amount gets duplicated!

---

### ✅ Correct (pre-aggregate)

```sql
WITH order_totals AS (
    SELECT order_id, SUM(amount) AS total_amount
    FROM order_items
    GROUP BY order_id
)
SELECT c.customer_id, SUM(ot.total_amount)
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
JOIN order_totals ot ON o.order_id = ot.order_id
GROUP BY c.customer_id;
```

---

### 🔥 Interview line:

> “Always aggregate before joining to avoid duplication errors.”

---

# 🔄 6. Pattern 5: Many-to-Many Join

---

## ❓ “Students enrolled in multiple courses”

Tables:

* students
* courses
* enrollments (bridge)

---

```sql
SELECT s.name, c.course_name
FROM students s
JOIN enrollments e ON s.student_id = e.student_id
JOIN courses c ON e.course_id = c.course_id;
```

---

👉 Classic bridge table pattern

---

# 🧩 7. Pattern 6: JOIN with Window Functions

---

## ❓ “Latest order per customer”

```sql
SELECT *
FROM (
    SELECT o.*, 
           ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date DESC) AS rn
    FROM orders o
) t
WHERE rn = 1;
```

---

👉 Sometimes no join needed — trick question in interviews

---

### With join:

```sql
SELECT c.name, o.order_id, o.order_date
FROM customers c
JOIN (
    SELECT *,
           ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date DESC) AS rn
    FROM orders
) o
ON c.customer_id = o.customer_id
WHERE o.rn = 1;
```

---

# 🔀 8. Pattern 7: FULL OUTER JOIN (rare but tricky)

---

## ❓ “Find mismatched records between two tables”

```sql
SELECT *
FROM table_a a
FULL OUTER JOIN table_b b
    ON a.id = b.id
WHERE a.id IS NULL OR b.id IS NULL;
```

---

👉 Finds:

* Missing in A
* Missing in B

---

# 🔍 9. Pattern 8: Semi Join (EXISTS)

---

## ❓ “Customers who placed at least one order”

```sql
SELECT *
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
);
```

---

### 🔥 Why better than JOIN?

* Stops at first match → faster
* Avoids duplicates

---

# ⚔️ 10. EXISTS vs JOIN (important comparison)

---

| Feature         | JOIN               | EXISTS          |
| --------------- | ------------------ | --------------- |
| Returns columns | Yes                | No              |
| Performance     | Can duplicate rows | Stops early     |
| Use case        | Data retrieval     | Existence check |

---

### 🔥 Interview line:

> “Use EXISTS when you only care about existence, not data.”

---

# 🧠 11. Pattern 9: Conditional JOIN

---

## ❓ “Join only recent orders”

```sql
SELECT *
FROM customers c
LEFT JOIN orders o
    ON c.customer_id = o.customer_id
    AND o.order_date >= CURRENT_DATE - INTERVAL '30 days';
```

---

👉 Condition in JOIN vs WHERE matters!

---

### ⚠️ Difference:

| Condition in | Effect             |
| ------------ | ------------------ |
| WHERE        | filters after join |
| JOIN         | affects matching   |

---

# 🔥 12. Pattern 10: Anti-pattern (important)

---

### ❌ Filtering LEFT JOIN in WHERE

```sql
SELECT *
FROM customers c
LEFT JOIN orders o
    ON c.customer_id = o.customer_id
WHERE o.amount > 100;
```

👉 Turns into INNER JOIN unintentionally ❌

---

### ✅ Correct

```sql
AND o.amount > 100
```

(inside JOIN)

---

# 🎯 13. Real Interview Combo Question

---

## ❓ “Customers with no orders in last 30 days”

```sql
SELECT c.*
FROM customers c
LEFT JOIN orders o
    ON c.customer_id = o.customer_id
    AND o.order_date >= CURRENT_DATE - INTERVAL '30 days'
WHERE o.order_id IS NULL;
```

---

👉 Combines:

* LEFT JOIN
* Conditional join
* NULL filtering

💥 Very common interview question

---

# 🧠 Final Mental Model

Think of joins as:

> “How rows multiply, match, or disappear based on conditions.”

---
