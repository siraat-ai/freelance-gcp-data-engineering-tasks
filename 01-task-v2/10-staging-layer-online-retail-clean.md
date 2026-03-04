# Staging Layer – Data Transformation (online_retail_clean)

## Overview

After successfully ingesting raw data into **BigQuery**, the next step in the ETL pipeline is to build the **Staging Layer**.

The staging layer is responsible for **cleaning, validating, and transforming raw data** before it moves to the curated analytics layer.

In this stage, we create the following table:

```text
retail_staging.online_retail_clean
```

This table is derived from the raw ingestion table:

```text
retail_raw.online_retail_raw
```

The goal of the staging layer is to **polish raw data and prepare it for reliable analytical use**.

---

# Data Pipeline Flow

The pipeline now follows this structure:

```text
CSV Dataset
      │
      ▼
retail_raw.online_retail_raw
      │
      ▼
retail_staging.online_retail_clean
      │
      ▼
retail_curated.retail_orders
```

The staging layer acts as the **data cleaning and transformation stage**.

---

# Transformations Applied in the Staging Layer

The following transformations are applied to the raw dataset:

1. **Timestamp Parsing**
2. **Remove Return Transactions**
3. **Remove Null Customers**
4. **Revenue Calculation**
5. **Deduplication**

These transformations ensure the dataset becomes **clean, structured, and analytics-ready**.

---

# 1. Timestamp Parsing

## Problem

In the raw dataset, the **InvoiceDate** field is stored as a **STRING** to avoid ingestion failures caused by inconsistent date formats.

Example raw value:

```text
01/12/2010 08:26
```

However, analytics and time-based queries require a proper **TIMESTAMP** data type.

---

## Solution

We convert the raw string into a timestamp using BigQuery's parsing function:

```text
PARSE_TIMESTAMP('%d/%m/%Y %H:%M', InvoiceDate)
```

This produces a new column:

```text
invoice_timestamp
```

---

## Benefits

* Enables **time-based analytics**
* Supports **partitioned tables**
* Allows efficient filtering by date
* Ensures accurate event ordering

---

# 2. Remove Return Transactions

Retail datasets often contain **product returns**, which appear as **negative quantities**.

Example:

```text
Quantity = -6
```

These records represent returned products, not actual purchases.

---

## Transformation Logic

We remove such records using the following condition:

```text
Quantity > 0
```

---

## Benefits

* Ensures only **valid sales transactions** remain
* Prevents incorrect revenue calculations
* Improves data quality for analytics

---

# 3. Remove Null Customers

Some records in the dataset contain missing **CustomerID** values.

Example:

```text
CustomerID = NULL
```

These transactions cannot be used in **customer-level analysis**.

---

## Transformation Logic

We filter out records where:

```text
CustomerID IS NULL
```

---

## Benefits

* Enables accurate customer analytics
* Supports customer segmentation
* Improves dataset consistency

---

# 4. Revenue Calculation

The raw dataset provides:

```text
Quantity
Price
```

However, **total revenue per transaction** is not available directly.

---

## Transformation Logic

We create a new derived column:

```text
Revenue = Quantity × Price
```

Example:

```text
6 items × £2.55 = £15.30
```

---

## Benefits

* Enables revenue analytics
* Supports sales performance reporting
* Provides key financial metrics

---

# 5. Deduplication

Duplicate records may appear due to:

* ingestion retries
* data synchronization issues
* batch reprocessing
* system failures

Duplicates can inflate revenue and transaction counts.

---

## Deduplication Strategy

We detect duplicates using a **window function**:

```text
ROW_NUMBER()
```

Partitioning is performed using the following columns:

```text
Invoice
StockCode
CustomerID
```

Only the **first occurrence** of each record is retained.

---

# SQL Implementation

The following SQL query creates the staging table:

```sql
CREATE OR REPLACE TABLE `retail_staging.online_retail_clean` AS

SELECT *
FROM (
    SELECT
        Invoice,
        StockCode,
        Description,
        Quantity,
        Price,
        CustomerID,
        Country,

        PARSE_TIMESTAMP('%d/%m/%Y %H:%M', InvoiceDate) AS invoice_timestamp,

        Quantity * Price AS revenue,

        ROW_NUMBER() OVER(
            PARTITION BY Invoice, StockCode, CustomerID
            ORDER BY InvoiceDate
        ) AS row_num

    FROM `retail_raw.online_retail_raw`

    WHERE Quantity > 0
      AND CustomerID IS NOT NULL
)

WHERE row_num = 1
```

---

# Resulting Table Structure

The staging table contains the following columns:

```text
Invoice
StockCode
Description
Quantity
Price
CustomerID
Country
invoice_timestamp
revenue
```

This dataset is now **cleaned and enriched** for downstream analytics.

---

# Why the Staging Layer is Important

The staging layer improves:

* **Data Quality**
* **Data Consistency**
* **Pipeline Reliability**
* **Analytical Accuracy**

It ensures that the curated layer receives **validated and standardized data**.

---

# Cost Considerations

The transformation query scans approximately:

```text
~1 MB of data
```

This is extremely small and well within **BigQuery free-tier limits**.

---

# Next Step

After building the staging layer, the next step is to create the **Curated Layer**, which will include:

* Partitioned tables
* Cost-optimized query design
* Incremental data loading
* Analytics-ready schema

The final analytics table will be:

```text
retail_curated.retail_orders
```

This table will serve as the **primary dataset for business intelligence and reporting**.


💡 Next step:
Ab hum console par actual query run karenge jo:

retail_staging.online_retail_clean

table create karegi.

Uske baad curated layer (partitioned table) banayenge jo real production-level BigQuery warehouse table hoti hai. 🚀
