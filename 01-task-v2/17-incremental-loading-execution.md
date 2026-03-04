# Incremental Loading Execution – Watermark Strategy

## Overview

After implementing the **Raw → Staging → Curated ETL pipeline**, the next step was to validate and execute **incremental loading**.

Incremental loading ensures that the pipeline **does not rebuild the entire curated table** during every run.
Instead, it loads **only newly arrived records** based on a timestamp watermark.

This execution demonstrates how incremental loading works in practice using **BigQuery SQL queries**.

---

# Objective of This Execution

The purpose of this exercise was to simulate a real production scenario where:

1. A new transaction arrives in the staging table.
2. The pipeline detects that the record is newer than the latest processed record.
3. The pipeline inserts only that new record into the curated table.

This confirms that the **incremental pipeline is functioning correctly**.

---

# Step 1 – Identify the Current Watermark

The first step was to identify the **latest processed timestamp** in the curated table.

Query executed:

```sql
SELECT
MAX(invoice_timestamp) AS last_loaded_timestamp
FROM `retail_curated.retail_orders`
```

Result:

```
2010-12-02 18:08:00 UTC
```

This timestamp represents the **watermark** of the pipeline.

Meaning:

```
All records with timestamps ≤ this value have already been processed.
```

Only records **newer than this timestamp** should be loaded.

---

# Step 2 – Insert a New Record into the Staging Table

To simulate a new incoming transaction, a test record was manually inserted into the staging table.

Query executed:

```sql
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
TIMESTAMP('2010-12-03 10:00:00'),
50.00
)
```

Important condition:

```
The timestamp must be greater than the watermark
```

Example:

```
2010-12-03 10:00:00 > 2010-12-02 18:08:00
```

Therefore, this record qualifies as **new data**.

---

# Step 3 – Execute Incremental Loading Query

After inserting the new record, the incremental loading query was executed.

Query:

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

# How the Incremental Query Works

The query performs the following operations:

1. Reads the **latest timestamp from the curated table**
2. Filters staging records using the condition:

```
invoice_timestamp > watermark
```

3. Inserts only the **new records** into the curated table.

This prevents the pipeline from reprocessing the entire dataset.

---

# Step 4 – Validate the Result

After executing the incremental query, the curated table was validated.

Query executed:

```sql
SELECT COUNT(*)
FROM `retail_curated.retail_orders`
```

Previous record count:

```
7373
```

New record count:

```
7374
```

This confirms that:

* The incremental pipeline detected the new record
* Only one new record was inserted
* Existing records were not reprocessed

---

# Verify the Inserted Record

To confirm that the test record was successfully loaded, the following query can be executed:

```sql
SELECT *
FROM `retail_curated.retail_orders`
WHERE Invoice = 'TEST10001'
```

The result should return the inserted test transaction.

---

# Pipeline Architecture After Incremental Loading

The ETL pipeline now follows this structure:

```
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
Incremental Loading (Watermark Strategy)
```

---

# Benefits Demonstrated

The incremental execution demonstrates several advantages.

### Faster Processing

Only new records are processed instead of rebuilding the entire table.

### Lower Query Cost

BigQuery scans significantly less data when only new records are processed.

### Scalable Pipeline Design

Incremental pipelines allow datasets to grow without increasing processing overhead.

---

# Important Limitation

The watermark strategy alone **does not prevent duplicate records**.

If the same transaction appears again with a newer timestamp, it may be inserted again.

Production pipelines solve this problem using a **MERGE-based incremental strategy**, which checks whether a record already exists before inserting it.

---

# Next Step

The next improvement to this pipeline is implementing a **MERGE-based incremental load**, which provides:

* duplicate protection
* idempotent pipeline execution
* production-grade data reliability

This enhancement will transform the pipeline into a **fully production-ready incremental data pipeline**.
