## Window Functions

---

# 🧠 1. What are Window Functions (in one sharp line)

A **window function** performs a calculation **across a set of rows related to the current row** — *without collapsing rows* (unlike `GROUP BY`).

👉 That “without collapsing rows” line = interview gold.

---

# ⚙️ 2. Basic Syntax (anchor this)

```sql
SELECT 
    column,
    window_function(...) OVER (
        PARTITION BY ...
        ORDER BY ...
        ROWS / RANGE ...
    )
FROM table;
```

---

# 🧩 3. PARTITION BY vs ORDER BY

### 🔹 PARTITION BY

* Like `GROUP BY`, but **does NOT reduce rows**
* Creates logical groups

### 🔹 ORDER BY

* Defines **row sequence inside each partition**
* Required for ranking, lag/lead, cumulative sums

---

# 🔢 4. Ranking Functions (VERY frequently asked)

### 1. `ROW_NUMBER()`

```sql
ROW_NUMBER() OVER (PARTITION BY dept ORDER BY salary DESC)
```

* Unique sequence (no ties)
* Use when: deduplication

---

### 2. `RANK()`

```sql
RANK() OVER (PARTITION BY dept ORDER BY salary DESC)
```

* Same rank for ties
* Skips numbers

👉 Example:

```
100 → rank 1
100 → rank 1
90  → rank 3
```

---

### 3. `DENSE_RANK()`

```sql
DENSE_RANK() OVER (PARTITION BY dept ORDER BY salary DESC)
```

* No gaps

```
100 → 1
100 → 1
90  → 2
```

---

### 🔥 Interview Tip

👉 “When would you use which?”

* Dedup → `ROW_NUMBER()`
* Leaderboard → `RANK()`
* Compact ranking → `DENSE_RANK()`

---

# 🔄 5. LAG & LEAD (VERY IMPORTANT)

These are used for **time-based comparisons**

---

### 🔹 LAG (look back)

```sql
LAG(salary, 1) OVER (PARTITION BY dept ORDER BY date)
```

👉 Example use:

* Previous day sales
* Detect change

---

### 🔹 LEAD (look forward)

```sql
LEAD(salary, 1) OVER (PARTITION BY dept ORDER BY date)
```

---

### 🔥 Classic Interview Question

👉 “Find day-over-day difference”

```sql
SELECT 
    date,
    sales,
    sales - LAG(sales) OVER (ORDER BY date) AS diff
FROM sales_table;
```

---

# 📊 6. Aggregate Window Functions

These are super important.

---

### Running Total

```sql
SUM(sales) OVER (
    PARTITION BY dept 
    ORDER BY date
)
```

---

### Moving Average

```sql
AVG(sales) OVER (
    ORDER BY date 
    ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
)
```

---

# 📏 7. ROWS vs RANGE (CRITICAL DIFFERENCE)

This is where most candidates mess up.

---

## 🔹 ROWS → Physical rows

```sql
ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
```

👉 Means:

* Current row
* 2 rows before it

---

## 🔹 RANGE → Logical value range

```sql
RANGE BETWEEN 100 PRECEDING AND CURRENT ROW
```

👉 Means:

* All rows with values within range (based on ORDER BY column)

---

### ⚠️ Important Difference

| Feature    | ROWS               | RANGE            |
| ---------- | ------------------ | ---------------- |
| Based on   | Row count          | Value range      |
| Duplicates | Treated separately | Treated together |

---

### 🔥 Interview line:

👉 “ROWS is deterministic, RANGE depends on values and can expand unexpectedly with duplicates.”

---

# 📌 8. Frame Clauses (MOST IMPORTANT CONCEPT)

---

## Syntax:

```sql
ROWS BETWEEN ... AND ...
```

---

## Keywords you MUST know:

### 🔹 UNBOUNDED PRECEDING

→ From **start of partition**

### 🔹 CURRENT ROW

→ Current row

### 🔹 UNBOUNDED FOLLOWING

→ Till **end of partition**

---

## Examples

---

### ✅ Cumulative Sum

```sql
SUM(sales) OVER (
    ORDER BY date 
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
)
```

👉 From start → current row

---

### ✅ Full Window

```sql
SUM(sales) OVER (
    ORDER BY date 
    ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
)
```

👉 Whole partition

---

### ✅ Moving Window

```sql
SUM(sales) OVER (
    ORDER BY date 
    ROWS BETWEEN 2 PRECEDING AND 1 FOLLOWING
)
```

---

# 🔥 9. Common Interview Problems

---

## ✅ 1. Deduplicate records

```sql
SELECT *
FROM (
    SELECT *,
           ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY updated_at DESC) AS rn
    FROM table
) t
WHERE rn = 1;
```

---

## ✅ 2. Top N per group

```sql
SELECT *
FROM (
    SELECT *,
           RANK() OVER (PARTITION BY dept ORDER BY salary DESC) AS rnk
    FROM emp
) t
WHERE rnk <= 3;
```

---

## ✅ 3. Find gaps in dates

```sql
SELECT 
    date,
    LAG(date) OVER (ORDER BY date) AS prev_date
FROM table;
```

---

## ✅ 4. Running total vs rolling average

👉 Expect follow-up questions here

---

# ⚠️ 10. Common Mistakes (interview traps)

* Forgetting `ORDER BY` → wrong results
* Using `RANGE` instead of `ROWS`
* Not handling ties properly (`RANK vs ROW_NUMBER`)
* Thinking window = group by ❌
* Not knowing default frame:

  ```sql
  RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
  ```

---

# 🎯 11. Pro-Level Insight (this impresses interviewers)

👉 Window functions are executed **after WHERE, before ORDER BY**

Execution order (simplified):

```text
FROM → WHERE → GROUP BY → HAVING → WINDOW → SELECT → ORDER BY
```

---

# 🧠 Final Mental Model

Think of window functions as:

> “Give me context around this row, without collapsing the dataset.”

---
