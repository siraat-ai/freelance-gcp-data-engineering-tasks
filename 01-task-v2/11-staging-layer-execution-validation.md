# Staging Layer Execution & Data Validation

## Overview

After successfully ingesting the dataset into the **Raw Layer**, the next step in the ETL pipeline was to build and execute the **Staging Layer transformation**.

The staging layer cleans and standardizes raw transactional data before it is used in analytical datasets.

This stage produced the following table:

```
retail_staging.online_retail_clean
```

The data for this table was sourced from:

```
retail_raw.online_retail_raw
```

---

# Objective of the Staging Layer

The purpose of the staging layer is to:

* Clean raw data
* Remove invalid records
* Standardize data types
* Enrich the dataset with calculated fields
* Remove duplicate records

This ensures that downstream datasets receive **consistent and reliable data**.

---

# Transformation Logic Applied

The staging query applied several transformations to the raw dataset.

## Timestamp Parsing

The raw dataset stored the **InvoiceDate** column as a string to prevent ingestion failures.

Example raw value:

```
01/12/2010 08:26
```

During the transformation stage, this value was converted into a proper **TIMESTAMP**.

Example transformation:

```
PARSE_TIMESTAMP('%d/%m/%Y %H:%M', InvoiceDate)
```

This created a new column:

```
invoice_timestamp
```

This allows time-based queries and supports future **table partitioning**.

---

# Removal of Return Transactions

Retail datasets often include product returns.

These appear as **negative quantities**.

Example:

```
Quantity = -6
```

Such rows were removed using the following condition:

```
Quantity > 0
```

This ensures that the dataset contains **only valid purchase transactions**.

---

# Removal of Null Customers

Some transactions in the dataset do not contain a valid **CustomerID**.

Example:

```
CustomerID = NULL
```

These records were removed because they cannot be used for **customer analytics or segmentation**.

Filtering condition:

```
CustomerID IS NOT NULL
```

---

# Revenue Calculation

The raw dataset contained two fields:

```
Quantity
Price
```

To enable revenue analysis, a new calculated column was added:

```
Revenue = Quantity × Price
```

Example:

```
6 × 2.55 = 15.30
```

This field enables sales and revenue analytics.

---

# Deduplication

Duplicate rows may appear due to:

* ingestion retries
* batch processing failures
* system synchronization issues

Duplicates were removed using a **window function**.

Example logic:

```
ROW_NUMBER() OVER(
PARTITION BY Invoice, StockCode, CustomerID
)
```

Only the first record from each duplicate group was retained.

---

# SQL Implementation

The following SQL query was executed to create the staging table:

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

# Execution Results

The staging transformation produced the following results:

| Layer           | Record Count   |
| --------------- | -------------- |
| Raw Dataset     | ~10,000 rows   |
| Staging Dataset | **7,373 rows** |

The reduction in rows occurred because:

* return transactions were removed
* records without CustomerID were removed
* duplicate records were eliminated

This confirms that **data cleaning logic was successfully applied**.

---

# Example Output Record

Example cleaned record from the staging table:

```
Invoice: 536785
StockCode: 22423
Description: REGENCY CAKESTAND 3 TIER
Quantity: 144
Price: 10.95
CustomerID: 15061
Country: United Kingdom
invoice_timestamp: 2010-12-02 15:22:00 UTC
revenue: 1576.8
```

This confirms that:

* timestamp parsing worked
* revenue calculation worked
* data cleaning rules were applied

---

# Data Pipeline Architecture

The ETL pipeline now follows this structure:

```
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

The staging layer ensures that only **clean and validated data** moves to the final analytics layer.

---

# Cost Consideration

The dataset size is small (~1 MB), meaning the query scanned a minimal amount of data.

Estimated cost:

```
~$0.00000
```

This is well within the **BigQuery free-tier limits**.

---

# Next Step – Curated Layer

The next stage of the pipeline is to create the **Curated Layer**, which will store analytics-ready data.

The final table will be:

```
retail_curated.retail_orders
```

This table will include:

* **Partitioned storage** using the invoice timestamp
* **Query performance optimization**
* **Cost-efficient data scanning**
* **Incremental loading logic**

Example design:

```
PARTITION BY DATE(invoice_timestamp)
CLUSTER BY CustomerID, Country
```

This layer will serve as the **primary dataset for dashboards, reporting, and analytical queries**.
