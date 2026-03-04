# Incremental Loading – Watermark Strategy

## Overview

After building the full ETL pipeline with **Raw → Staging → Curated layers**, the next improvement is implementing **Incremental Loading**.

Incremental loading ensures that the pipeline **does not rebuild the entire table every time it runs**.

Instead, it loads **only the new records that were not previously processed**.

This approach is widely used in **production data pipelines** to reduce cost, improve performance, and support scalable architectures.

---

# Problem with Full Table Rebuild

Previously, the curated table was created using the following pattern:

```sql
CREATE OR REPLACE TABLE retail_curated.retail_orders AS
SELECT ...
FROM retail_staging.online_retail_clean
```

This approach recreates the entire table every time the pipeline runs.

Example scenario:

| Existing Rows | New Rows | Total Rows Processed |
| ------------- | -------- | -------------------- |
| 7373          | 10       | 7383                 |

Even if only **10 new records** arrive, the pipeline processes **all 7,383 rows again**.

This is inefficient when datasets grow to **millions or billions of records**.

---

# Incremental Loading Concept

Incremental loading solves this problem by processing **only the new records**.

Instead of rebuilding the curated table, the pipeline inserts only the data that **arrived after the previous run**.

The key idea is to identify the **latest processed timestamp** and load only records that are newer.

---

# Watermark Strategy

To detect new records, the pipeline uses a **watermark column**.

In this project, the watermark column is:

```text
invoice_timestamp
```

The pipeline checks the **latest timestamp already present in the curated table**.

Example query:

```sql
SELECT
MAX(invoice_timestamp) AS last_loaded_timestamp
FROM `retail_curated.retail_orders`
```

Example result:

```text
2010-12-02 15:22:00
```

This timestamp represents the **latest processed record**.

---

# Incremental Loading Logic

Once the watermark is identified, the pipeline loads only the records that are **newer than the watermark**.

Example logic:

```sql
WHERE invoice_timestamp >
(
SELECT MAX(invoice_timestamp)
FROM `retail_curated.retail_orders`
)
```

This ensures that previously processed data is **not loaded again**.

---

# Incremental Insert Query

The following SQL query implements incremental loading.

```sql
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

# Example Workflow

Initial state:

```text
Curated table rows: 7373
Latest timestamp: 2010-12-02 15:22:00
```

New incoming data:

```text
2010-12-03 09:10:00
2010-12-03 09:20:00
```

Pipeline behavior:

```text
Only the two new rows are inserted.
```

Final state:

```text
7373 + 2 = 7375 rows
```

The pipeline processes **only the new data instead of the entire dataset**.

---

# Benefits of Incremental Loading

Incremental pipelines provide several important advantages.

### Faster Pipeline Execution

Only new records are processed instead of the entire dataset.

### Lower Query Costs

BigQuery charges based on the amount of data scanned.
Incremental pipelines reduce scanned data significantly.

### Scalable Data Pipelines

Incremental loading allows pipelines to scale efficiently when datasets grow to **millions or billions of records**.

---

# Limitation of Basic Incremental Loading

Basic incremental loading using a timestamp watermark **does not automatically prevent duplicates**.

If the same record appears again with a newer timestamp, it could be inserted again.

To solve this problem, production pipelines typically use a **MERGE statement**.

---

# Production-Level Incremental Strategy

A more advanced incremental pipeline uses **MERGE** to prevent duplicates.

Example:

```sql
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

This approach ensures that:

* Existing records are not duplicated
* Only new transactions are inserted
* The pipeline remains idempotent

---

# Updated Data Pipeline Architecture

After implementing incremental loading, the pipeline becomes:

```text
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
Incremental Loading (Watermark / MERGE)
```

---

# Future Automation

In production environments, incremental pipelines are typically executed using **scheduled workflows**.

Example architecture:

```text
Cloud Scheduler
      │
      ▼
Scheduled BigQuery Query
      │
      ▼
Incremental Load
```

This allows the pipeline to run **daily or hourly without manual intervention**.

---

# Conclusion

Incremental loading is a critical design pattern in modern data engineering pipelines.

By loading **only new records instead of rebuilding entire tables**, pipelines become:

* Faster
* More cost-efficient
* Scalable for large datasets

The watermark strategy using **`MAX(invoice_timestamp)`** provides a simple and effective way to detect newly arriving data in batch pipelines.
