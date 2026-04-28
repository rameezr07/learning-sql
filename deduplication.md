## Deduplication

---

# 🧠 1. What do we mean by duplicates?

👉 Two types (this distinction matters a lot in interviews):

### 🔹 Exact duplicates

All columns identical

### 🔹 Business duplicates

Same **logical entity**, but multiple records
(e.g., same `customer_id` with different timestamps)

---

# 🔍 2. Finding duplicates

---

## ✅ Exact duplicates

```sql
SELECT *, COUNT(*) 
FROM table
GROUP BY col1, col2, col3
HAVING COUNT(*) > 1;
```

---

## ✅ Based on business key

```sql
SELECT customer_id, COUNT(*) 
FROM orders
GROUP BY customer_id
HAVING COUNT(*) > 1;
```

---

### 🔥 Interview tip:

> Always clarify: “What defines a duplicate — all columns or business key?”

---

# ⚙️ 3. Removing duplicates (basic level)

---

## ✅ Using DISTINCT

```sql
SELECT DISTINCT *
FROM table;
```

👉 Simple but:

* Expensive on large data
* No control over which row to keep

---

# 🔥 4. Deduplication using ROW_NUMBER (MOST IMPORTANT)

This is **the most expected answer in interviews**.

---

## Pattern:

```sql
SELECT *
FROM (
    SELECT *,
           ROW_NUMBER() OVER (
               PARTITION BY customer_id 
               ORDER BY updated_at DESC
           ) AS rn
    FROM table
) t
WHERE rn = 1;
```

---

## 🧠 What’s happening:

* Group by business key (`customer_id`)
* Order rows (latest first)
* Keep only best row (`rn = 1`)

---

### 🔥 Interview line:

> “ROW_NUMBER allows deterministic deduplication based on business logic.”

---

# 📊 5. Choosing the “right” record

This is where you stand out.

---

## Common rules:

* Latest record → `ORDER BY updated_at DESC`
* Highest value → `ORDER BY amount DESC`
* Earliest → `ASC`

---

👉 Always ask:

> “Which record should we keep?”

---

# 🔁 6. Deduplication using RANK / DENSE_RANK

---

## When ties matter:

```sql
RANK() OVER (PARTITION BY customer_id ORDER BY score DESC)
```

👉 Keeps multiple rows if tied

---

# 🧩 7. Deduplication with JOIN (anti-pattern vs correct)

---

## ❌ Bad way

```sql
SELECT DISTINCT customer_id
FROM orders;
```

👉 Loses information

---

## ✅ Better

Use window function or aggregation

---

# 📦 8. Deduplication in INSERT (very real-world)

---

## ❓ “Insert only new records”

```sql
INSERT INTO target t
SELECT *
FROM source s
WHERE NOT EXISTS (
    SELECT 1
    FROM target t
    WHERE t.id = s.id
);
```

---

👉 Prevent duplicates during ingestion

---

# 🔄 9. Deduplication in streaming / incremental data

---

## Pattern:

* Use **timestamp / version column**
* Keep latest record

---

```sql
SELECT *
FROM (
    SELECT *,
           ROW_NUMBER() OVER (
               PARTITION BY id 
               ORDER BY ingestion_time DESC
           ) AS rn
    FROM stream_data
) t
WHERE rn = 1;
```

---

# ⚠️ 10. Common pitfalls

---

## ❌ Missing ORDER BY in ROW_NUMBER

```sql
ROW_NUMBER() OVER (PARTITION BY id)
```

👉 Non-deterministic result ❌

---

## ❌ Using DISTINCT blindly

👉 hides data issues

---

## ❌ Dedup after JOIN (too late)

👉 duplicates already exploded

---

## ❌ Not defining business key

👉 biggest mistake in interviews

---

# 🔥 11. Advanced Pattern: Dedup + Aggregation

---

## ❓ “Total spend per unique customer (latest record only)”

```sql
WITH dedup AS (
    SELECT *,
           ROW_NUMBER() OVER (
               PARTITION BY customer_id 
               ORDER BY updated_at DESC
           ) AS rn
    FROM customers
)
SELECT customer_id, SUM(amount)
FROM dedup
WHERE rn = 1
GROUP BY customer_id;
```

---

# 🧠 12. Handling duplicates after JOIN (VERY IMPORTANT)

---

## Problem:

JOIN creates duplicates

---

## Solution:

👉 Deduplicate **before** join

```sql
WITH dedup_orders AS (
    SELECT *,
           ROW_NUMBER() OVER (
               PARTITION BY order_id 
               ORDER BY updated_at DESC
           ) AS rn
    FROM orders
)
SELECT *
FROM dedup_orders
WHERE rn = 1;
```

---

# 🧩 13. Delete duplicates (real interview question)

---

## Keep latest row

```sql
DELETE FROM table
WHERE id IN (
    SELECT id
    FROM (
        SELECT id,
               ROW_NUMBER() OVER (
                   PARTITION BY customer_id 
                   ORDER BY updated_at DESC
               ) AS rn
        FROM table
    ) t
    WHERE rn > 1
);
```

---

# 🎯 14. Real Interview Question

---

## ❓ “Remove duplicate events but keep latest per user per day”

```sql
SELECT *
FROM (
    SELECT *,
           ROW_NUMBER() OVER (
               PARTITION BY user_id, DATE(event_time)
               ORDER BY event_time DESC
           ) AS rn
    FROM events
) t
WHERE rn = 1;
```

---

# 🧠 Final Mental Model

Think:

> “Define duplicate → decide which row to keep → enforce using window function.”

---

# 🔥 Golden Answer (if interviewer asks open-ended)

You can say:

> “I identify duplicates using GROUP BY, define a business key, and use ROW_NUMBER with ORDER BY to deterministically retain the correct record. I also ensure deduplication happens before joins or aggregations to avoid data explosion.”

---

