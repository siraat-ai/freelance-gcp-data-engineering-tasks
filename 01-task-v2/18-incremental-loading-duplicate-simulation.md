# Incremental Loading – Duplicate Scenario Simulation

## Overview

After successfully implementing and validating **incremental loading using the watermark strategy**, the next step was to simulate a **duplicate record scenario**.

This exercise demonstrates an important real-world behavior in incremental pipelines:

* Watermark-based incremental loading can insert duplicate records.
* Production pipelines solve this problem using **MERGE-based incremental logic**.

This test replicates a typical **data engineering debugging scenario** used to validate pipeline reliability.

---

# Objective of This Test

The goal of this exercise was to:

1. Insert a **duplicate transaction** into the staging table.
2. Run the incremental loading query.
3. Observe how the watermark pipeline behaves.
4. Demonstrate how **MERGE prevents duplicate records**.

---

# Step 1 – Insert a Duplicate Record into the Staging Table

A duplicate transaction was intentionally inserted into the staging table.

The record used the **same business keys** but had a **new timestamp**.

Query executed:

```sql id="s9w6ra"
INSERT INTO `retail_staging.online_retail_clean`
(
Invoice,
StockCode,
Description,
Quantity,
Price,
CustomerID,
Country,
invoice_timestamp,
revenue
)

VALUES
(
'TEST10001',
'TESTSKU1',
'TEST PRODUCT',
5,
10.00,
'99999',
'Spain',
TIMESTAMP('2010-12-04 09:00:00'),
50.00
)
```

Key characteristics of this record:

| Column    | Value      |
| --------- | ---------- |
| Invoice   | TEST10001  |
| StockCode | TESTSKU1   |
| Timestamp | 2010-12-04 |

The **Invoice and StockCode values already existed** in the curated table.

However, the timestamp was newer.

---

# Step 2 – Execute Incremental Loading Query

The incremental loading query was executed again.

```sql id="0nq83q"
INSERT INTO `retail_curated.retail_orders`

SELECT
    Invoice,
    StockCode,
    Description,
    Quantity,
    Price,
    CustomerID,
    Country,
    invoice_timestamp,
    revenue

FROM `retail_staging.online_retail_clean`

WHERE invoice_timestamp >
(
SELECT MAX(invoice_timestamp)
FROM `retail_curated.retail_orders`
)
```

---

# Pipeline Behavior

The watermark logic evaluates only the **timestamp condition**.

```text id="l1m7l9"
invoice_timestamp > MAX(invoice_timestamp)
```

Since the new record had a later timestamp, the pipeline inserted it into the curated table.

This resulted in **duplicate records**.

---

# Step 3 – Validate Duplicate Records

The following query was used to verify the duplicate scenario.

```sql id="r7uy8b"
SELECT *
FROM `retail_curated.retail_orders`
WHERE Invoice = 'TEST10001'
```

Example result:

| Invoice   | Timestamp  |
| --------- | ---------- |
| TEST10001 | 2010-12-03 |
| TEST10001 | 2010-12-04 |

This confirms that **watermark-based incremental loading alone does not prevent duplicates**.

---

# Why the Duplicate Occurred

The watermark strategy only checks whether a record is **newer than the previously processed timestamp**.

It does not validate whether the transaction already exists.

Therefore:

```text id="5s15pi"
New timestamp → record inserted
Existing business key → duplicate created
```

---

# Step 4 – Preventing Duplicates Using MERGE

To prevent duplicates, production pipelines typically use a **MERGE statement**.

MERGE compares incoming records with existing records in the target table.

Example query:

```sql id="n2td4h"
MERGE `retail_curated.retail_orders` T
USING `retail_staging.online_retail_clean` S

ON T.Invoice = S.Invoice
AND T.StockCode = S.StockCode

WHEN NOT MATCHED THEN
INSERT (
Invoice,
StockCode,
Description,
Quantity,
Price,
CustomerID,
Country,
invoice_timestamp,
revenue
)

VALUES (
S.Invoice,
S.StockCode,
S.Description,
S.Quantity,
S.Price,
S.CustomerID,
S.Country,
S.invoice_timestamp,
S.revenue
)
```


Result:

duplicate insert nahi hoga
---

# MERGE Logic

| Method                | Duplicate Safe |
| --------------------- | -------------- |
| Watermark Incremental | ❌ No           |
| MERGE Incremental     | ✅ Yes          |


The MERGE statement performs the following operations:

| Condition             | Action            |
| --------------------- | ----------------- |
| Record exists         | Skip insertion    |
| Record does not exist | Insert new record |

This ensures that the curated dataset remains **duplicate-free**.



---

# Incremental Pipeline with Duplicate Protection

A production-grade incremental pipeline usually follows this design:

```text id="p6m07x"
Staging Table
      │
      ▼
Watermark Filter
      │
      ▼
MERGE into Curated Table
```

---

# Advantages of This Approach

Combining watermark filtering and MERGE provides several benefits.

### Efficient Data Processing

Only recently arrived records are processed.

### Duplicate Prevention

Existing records are detected before insertion.

### Reliable Data Pipeline

The pipeline becomes **idempotent and safe for repeated execution**.

---

# Key Takeaway

Watermark-based incremental loading identifies **which records should be processed**, but it does not guarantee uniqueness.

MERGE ensures **data integrity** by preventing duplicate transactions from entering the curated dataset.

Using both techniques together results in a **production-ready incremental data pipeline**.

---

# Pipeline Architecture After This Exercise

The final pipeline architecture now looks like this:

```text id="c9t0hx"
CSV Source
      │
      ▼
retail_raw.online_retail_raw
      │
      ▼
retail_staging.online_retail_clean
      │
      ▼
retail_curated.retail_orders
      │
      ▼
Incremental Loading (Watermark)
      │
      ▼
Duplicate Protection (MERGE)
```

This structure represents a **modern cloud data engineering pipeline** capable of handling incremental updates and duplicate prevention.
