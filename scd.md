## SCDs (Slowly Changing Dimensions)

---

# 🧠 1. What are SCDs?

👉 One-liner:

> “SCDs are techniques to handle changes in dimension data over time.”

---

## 🧩 Example

Customer table:

```text
customer_id | name  | city
--------------------------
101         | John  | Delhi
```

Later:

```text
101         | John  | Mumbai
```

👉 Question:

* Overwrite?
* Keep history?
* Keep both?

👉 That’s exactly what SCD types solve.

---

# 🔢 2. Types of SCD (focus on 1, 2, 3)

---

# 🔹 SCD Type 1 (Overwrite)

👉 Replace old data with new data

---

## Example

Before:

```text
101 | John | Delhi
```

After:

```text
101 | John | Mumbai
```

---

## ✅ SQL Logic

```sql
UPDATE customers
SET city = 'Mumbai'
WHERE customer_id = 101;
```

---

## 🧠 Use when:

* History not needed
* Latest value only matters

---

## ⚠️ Downside:

* Data loss (no history)

---

# 🔹 SCD Type 2 (MOST IMPORTANT ⭐)

👉 Keep full history by creating new rows

---

## Structure

```text
customer_id | city   | start_date | end_date   | is_current
----------------------------------------------------------
101         | Delhi  | 2020-01-01 | 2023-01-01 | 0
101         | Mumbai | 2023-01-02 | NULL       | 1
```

---

## 🧠 Key idea:

* Old record → expired
* New record → inserted

---

## ✅ Implementation Logic (step-by-step)

---

### Step 1: Identify changed records

```sql
SELECT s.*
FROM source s
JOIN target t
  ON s.customer_id = t.customer_id
WHERE s.city <> t.city
  AND t.is_current = 1;
```

---

### Step 2: Expire old records

```sql
UPDATE target
SET end_date = CURRENT_DATE,
    is_current = 0
WHERE customer_id IN (
    SELECT customer_id FROM changed_records
);
```

---

### Step 3: Insert new records

```sql
INSERT INTO target (
    customer_id, city, start_date, end_date, is_current
)
SELECT 
    customer_id,
    city,
    CURRENT_DATE,
    NULL,
    1
FROM source;
```

---

## 🔥 Interview line:

> “SCD Type 2 maintains full history using effective dates and a current flag.”

---

# 🔹 SCD Type 3 (Limited History)

👉 Keep previous value in extra column

---

## Example

```text
customer_id | current_city | previous_city
------------------------------------------
101         | Mumbai       | Delhi
```

---

## ✅ SQL

```sql
UPDATE customers
SET previous_city = current_city,
    current_city = 'Mumbai'
WHERE customer_id = 101;
```

---

## 🧠 Use when:

* Only limited history needed

---

## ⚠️ Limitation:

* Cannot track long history

---

# 🧩 3. Other Types (brief)

---

### 🔹 Type 0 → No change allowed

### 🔹 Type 4 → History stored in separate table

### 🔹 Type 6 → Hybrid (1 + 2 + 3)

👉 Mentioning Type 2 properly is enough for most interviews.

---

# ⚙️ 4. Important Columns in SCD Type 2

---

* `start_date` (effective_from)
* `end_date` (effective_to)
* `is_current`
* surrogate key (VERY IMPORTANT)

---

# 🔑 5. Surrogate Key vs Business Key

---

## 🔹 Business key

* Natural identifier (`customer_id`)

## 🔹 Surrogate key

* Artificial key (`customer_sk`)

---

### Example

```text
customer_sk | customer_id | city
--------------------------------
1           | 101         | Delhi
2           | 101         | Mumbai
```

---

### 🔥 Interview line:

> “Surrogate keys uniquely identify each version of a record.”

---

# ⚠️ 6. Important Considerations (this is what interviewers care about)

---

## 🔹 1. Change Detection

👉 How do you detect change?

* Hash comparison
* Column comparison

---

## 🔹 2. Late-arriving data

👉 What if old data comes late?

* Need to adjust history carefully

---

## 🔹 3. Slowly vs rapidly changing

👉 If data changes too frequently:

* Type 2 may explode table size

---

## 🔹 4. Data volume

👉 SCD2 tables grow fast → need partitioning

---

## 🔹 5. Query performance

👉 Filtering current records:

```sql
WHERE is_current = 1
```

---

## 🔹 6. Idempotency (VERY IMPORTANT in pipelines)

👉 Running job twice should not duplicate data

---

# 🔄 7. MERGE Statement (Modern approach)

---

## Example (BigQuery / Snowflake)

```sql
MERGE INTO target t
USING source s
ON t.customer_id = s.customer_id AND t.is_current = 1

WHEN MATCHED AND t.city <> s.city THEN
  UPDATE SET 
    t.end_date = CURRENT_DATE,
    t.is_current = 0

WHEN NOT MATCHED THEN
  INSERT (customer_id, city, start_date, end_date, is_current)
  VALUES (s.customer_id, s.city, CURRENT_DATE, NULL, 1);
```

---

👉 Cleaner + production-friendly

---

# 🎯 8. Real Interview Question

---

## ❓ “Design SCD Type 2 for customer table”

👉 Expected answer:

* Use surrogate key
* Track `start_date`, `end_date`, `is_current`
* Detect changes
* Expire old records
* Insert new version

---

# 🧠 9. When to use which SCD?

---

| Type   | Use Case                   |
| ------ | -------------------------- |
| Type 1 | Latest data only           |
| Type 2 | Full history (most common) |
| Type 3 | Limited history            |

---

# 🔥 10. Golden Answer (if asked open-ended)

You can say:

> “I typically use SCD Type 2 for dimensional tables where history matters. I detect changes using column comparison or hashing, expire old records using end_date, insert new versions with start_date, and maintain a surrogate key for uniqueness. I also ensure idempotency and optimize queries by filtering current records.”

---

# 🧠 Final Mental Model

Think:

> “SCD = how I track changes over time in dimension tables.”

---

