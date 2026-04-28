## Analytical Queries

---

# 🧠 1. GROUP BY — what’s really happening?

👉 One-liner:

> “GROUP BY aggregates rows into buckets based on one or more columns.”

---

### Basic Example

```sql
SELECT department, SUM(salary)
FROM employees
GROUP BY department;
```

👉 You’re saying:

* “Group all rows by department”
* Then compute sum inside each group

---

# ⚙️ 2. HAVING vs WHERE (VERY IMPORTANT)

This is almost guaranteed in interviews.

---

## 🔹 WHERE

* Filters **before aggregation**

## 🔹 HAVING

* Filters **after aggregation**

---

### Example

```sql
SELECT department, SUM(salary) AS total_salary
FROM employees
GROUP BY department
HAVING SUM(salary) > 100000;
```

👉 You cannot use `WHERE SUM(salary) > ...` ❌

---

### 🔥 Interview one-liner:

> “WHERE filters rows, HAVING filters groups.”

---

# 🧩 3. Common Analytical Patterns

Now let’s get into the good stuff.

---

# 📊 4. Pattern 1: Top Performing Groups

### ❓ “Which departments have highest revenue?”

```sql
SELECT department, SUM(revenue) AS total_revenue
FROM sales
GROUP BY department
ORDER BY total_revenue DESC;
```

---

### Add HAVING (filtering meaningful groups)

```sql
SELECT department, SUM(revenue) AS total_revenue
FROM sales
GROUP BY department
HAVING SUM(revenue) > 50000
ORDER BY total_revenue DESC;
```

---

# 📈 5. Pattern 2: Find Duplicates

Classic interview question.

```sql
SELECT customer_id, COUNT(*) AS cnt
FROM orders
GROUP BY customer_id
HAVING COUNT(*) > 1;
```

👉 Detect duplicate customers/orders/etc.

---

# 🧮 6. Pattern 3: Multi-level Grouping

```sql
SELECT department, job_role, COUNT(*) AS emp_count
FROM employees
GROUP BY department, job_role;
```

👉 Groups inside groups

---

# 🔥 7. Pattern 4: Conditional Aggregation (VERY IMPORTANT)

This is heavily asked.

---

## ❓ “Count how many orders are completed vs cancelled”

```sql
SELECT 
    customer_id,
    SUM(CASE WHEN status = 'completed' THEN 1 ELSE 0 END) AS completed_orders,
    SUM(CASE WHEN status = 'cancelled' THEN 1 ELSE 0 END) AS cancelled_orders
FROM orders
GROUP BY customer_id;
```

---

### 🔥 Why this is powerful:

* Avoids multiple queries
* Works like pivot

---

# 📅 8. Pattern 5: Time-based Analysis

---

## ❓ “Daily sales”

```sql
SELECT 
    DATE(order_timestamp) AS order_date,
    SUM(amount) AS total_sales
FROM orders
GROUP BY DATE(order_timestamp);
```

---

## ❓ “Monthly revenue”

```sql
SELECT 
    DATE_TRUNC('month', order_timestamp) AS month,
    SUM(amount) AS revenue
FROM orders
GROUP BY month;
```

---

# 🧠 9. Pattern 6: Filtering Before vs After Aggregation

---

### ❗ Scenario:

“Find departments where avg salary > 50k but only consider employees with salary > 30k”

---

```sql
SELECT department, AVG(salary) AS avg_salary
FROM employees
WHERE salary > 30000
GROUP BY department
HAVING AVG(salary) > 50000;
```

---

👉 This tests:

* WHERE first
* GROUP BY
* HAVING last

---

# 🧮 10. Pattern 7: Percentage Contribution

---

## ❓ “What % of total sales each department contributes?”

```sql
SELECT 
    department,
    SUM(amount) AS dept_sales,
    SUM(amount) * 100.0 / SUM(SUM(amount)) OVER () AS percentage
FROM sales
GROUP BY department;
```

---

👉 Combines:

* GROUP BY
* Window function 🔥

---

# 🧩 11. Pattern 8: Max per Group (classic tricky one)

---

## ❓ “Highest salary per department”

```sql
SELECT department, MAX(salary) AS max_salary
FROM employees
GROUP BY department;
```

---

## ❗ Follow-up (important):

👉 “Get full row of highest salary”

```sql
SELECT *
FROM (
    SELECT *,
           ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS rn
    FROM employees
) t
WHERE rn = 1;
```

---

# ⚠️ 12. Common Mistakes

---

### ❌ Missing column in GROUP BY

```sql
SELECT department, salary
FROM employees
GROUP BY department;
```

👉 ❌ Invalid (salary not aggregated)

---

### ❌ Using WHERE instead of HAVING

```sql
WHERE COUNT(*) > 1  -- ❌ wrong
```

---

### ❌ Not understanding aggregation order

---

# 🔥 13. Pro Interview Insight

👉 SQL execution order (simplified):

```text
FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY
```

---

# 🎯 14. Advanced Combo Question (VERY COMMON)

---

## ❓ “Top 3 customers by total spend”

```sql
WITH customer_spend AS (
    SELECT customer_id, SUM(amount) AS total_spend
    FROM orders
    GROUP BY customer_id
)
SELECT *
FROM (
    SELECT *,
           RANK() OVER (ORDER BY total_spend DESC) AS rnk
    FROM customer_spend
) t
WHERE rnk <= 3;
```

---

👉 Combines:

* GROUP BY
* CTE
* Window function

💥 This is exactly what real interviews look like.

---

# 🧠 Final Mental Model

Think:

> “GROUP BY = create buckets
> Aggregates = compute inside buckets
> HAVING = filter buckets”

---

