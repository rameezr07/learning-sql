## CTE (Common Table Expression)

---

# 🧠 1. What is a CTE (Common Table Expression)?

A **CTE** is a temporary named result set that you can reference within a query.

👉 One-liner for interviews:

> “A CTE improves readability and modularity by breaking complex queries into logical steps.”

---

# ⚙️ 2. Basic Syntax

```sql
WITH cte_name AS (
    SELECT ...
)
SELECT *
FROM cte_name;
```

---

# 🧩 3. Why CTEs Matter (real-world angle)

Without CTE:

```sql
SELECT ...
FROM (
    SELECT ...
    FROM (
        SELECT ...
    ) t1
) t2;
```

👉 This becomes unreadable very fast.

---

With CTE:

```sql
WITH step1 AS (...),
     step2 AS (...)
SELECT ...
FROM step2;
```

👉 Cleaner, debuggable, interview-friendly.

---

# 🔥 4. Important Use Cases (you should mention these)

---

## ✅ 1. Breaking complex logic into steps

```sql
WITH sales_per_day AS (
    SELECT date, SUM(sales) AS total_sales
    FROM orders
    GROUP BY date
)
SELECT *
FROM sales_per_day
WHERE total_sales > 1000;
```

---

## ✅ 2. Using CTE with Window Functions

Very common in interviews.

```sql
WITH ranked_data AS (
    SELECT *,
           ROW_NUMBER() OVER (PARTITION BY dept ORDER BY salary DESC) AS rn
    FROM employees
)
SELECT *
FROM ranked_data
WHERE rn = 1;
```

👉 Cleaner than nesting window function query.

---

## ✅ 3. Multiple CTEs (pipeline style)

```sql
WITH step1 AS (
    SELECT customer_id, SUM(amount) AS total
    FROM orders
    GROUP BY customer_id
),
step2 AS (
    SELECT *
    FROM step1
    WHERE total > 1000
)
SELECT *
FROM step2;
```

👉 Think of it like a **mini data pipeline inside SQL**

---

## ✅ 4. Reuse logic (important)

```sql
WITH high_value_customers AS (
    SELECT customer_id
    FROM orders
    GROUP BY customer_id
    HAVING SUM(amount) > 1000
)
SELECT *
FROM high_value_customers h
JOIN customers c ON h.customer_id = c.id;
```

---

# ⚖️ 5. CTE vs Subquery (THIS is interview gold)

---

## 🔹 Readability

* CTE ✅ much better
* Subquery ❌ messy when nested

---

## 🔹 Reusability

* CTE ✅ reusable multiple times
* Subquery ❌ repeated logic

---

## 🔹 Debugging

* CTE ✅ can test each step separately
* Subquery ❌ hard to isolate

---

## 🔹 Performance (IMPORTANT — don’t answer this incorrectly)

👉 This is where many candidates mess up.

### ❗ Key point:

> “CTEs are not always faster than subqueries.”

---

### Behavior depends on DB engine:

#### 🔹 Older engines (like older PostgreSQL)

* CTE = **materialized (stored temporarily)**
* Can be slower

#### 🔹 Modern engines (BigQuery, Snowflake, newer PostgreSQL)

* CTE = **inlined (optimized like subquery)**

---

### 🔥 Best interview answer:

> “Performance depends on the query planner. In modern systems, CTEs are often optimized like subqueries, but in some engines they may be materialized, which can impact performance.”

---

## 🔹 When Subquery is better?

* Simple one-time logic
* Avoid overhead

---

## 🔹 When CTE is better?

* Complex queries
* Multiple references
* Window functions
* Debugging

---

# ⚠️ 6. Common Mistakes

* Thinking CTE improves performance ❌ (not always)
* Overusing CTEs for simple queries
* Forgetting execution cost when reused many times

---

# 🔁 7. Recursive CTE (Important but usually basic level in interviews)

Used for:

* Hierarchies (org chart)
* Tree traversal
* Graph problems

---

## ⚙️ Syntax

```sql
WITH RECURSIVE cte_name AS (
    -- base case
    SELECT ...

    UNION ALL

    -- recursive step
    SELECT ...
    FROM cte_name
    WHERE condition
)
SELECT * FROM cte_name;
```

---

## 🌳 Example: Employee Hierarchy

Table:

```
emp_id | manager_id
```

---

### Find all employees under a manager:

```sql
WITH RECURSIVE emp_tree AS (
    -- Base case
    SELECT emp_id, manager_id
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    -- Recursive case
    SELECT e.emp_id, e.manager_id
    FROM employees e
    JOIN emp_tree t
        ON e.manager_id = t.emp_id
)
SELECT *
FROM emp_tree;
```

---

### 🧠 Explanation

* Start with top-level manager
* Keep joining children
* Repeat until no more rows

---

### 🔥 Interview Tip

Say this line:

> “Recursive CTEs work like loops in SQL — they repeatedly execute until the termination condition is met.”

---

# 🎯 8. When Recursive CTEs are used in real life

* Organizational hierarchy
* Folder/file systems
* Bill of materials (BOM)
* Dependency graphs

---

# 🧠 Final Mental Model

Think of CTEs as:

> “Step-by-step transformation layers inside a query.”

---

