# Transformation Layer – Retail Staging (ETL Step)

## Overview

After successfully ingesting the raw dataset into **BigQuery**, the next step in the pipeline is to build the **Transformation Layer**.

In this stage, raw transactional data is cleaned, standardized, and enriched to make it suitable for analytics and downstream consumption.

The cleaned data will be stored in the following table:

```
retail_staging.online_retail_clean
```

This stage represents the **core ETL transformation process** where raw data is converted into structured and reliable analytical data.

---

# Transformation Tasks

The transformation layer performs the following operations:

1. **Timestamp Parsing**
2. **Remove Return Transactions**
3. **Remove Null Customers**
4. **Revenue Calculation**
5. **Deduplication**

Each transformation ensures the dataset becomes **clean, consistent, and analytics-ready**.

---

# 1. Timestamp Parsing

## Purpose

In the raw dataset, the **InvoiceDate** column is stored as a **STRING** during ingestion to avoid parsing errors during CSV loading.

However, analytical queries require proper **time-based data types**.

Therefore, we convert the raw date string into a **TIMESTAMP** format.

## Transformation Logic

The date string follows this format:

```
DD/MM/YYYY HH:MM
```

Example:

```
01/12/2010 08:26
```

Using BigQuery functions, we parse this value into a timestamp.

Example transformation:

```
PARSE_TIMESTAMP('%d/%m/%Y %H:%M', InvoiceDate)
```

## Benefits

This enables:

* **Time-based analytics**
* **Partitioned tables**
* **Efficient querying by date**
* **Accurate event ordering**

---

# 2. Remove Return Transactions

## Purpose

In retail transaction systems, product returns appear as **negative quantities**.

Example:

```
Quantity = -5
```

These rows represent **returned items**, not actual purchases.

For revenue analysis and sales analytics, these records must be removed from the cleaned dataset.

## Transformation Logic

We filter transactions where:

```
Quantity > 0
```

## Benefits

This ensures that:

* Only **valid purchase transactions** remain
* Revenue calculations remain accurate
* Analytical reports are not skewed by return records

---

# 3. Remove Null Customers

## Purpose

Some rows in the dataset contain **missing CustomerID values**.

Example:

```
CustomerID = NULL
```

These records represent **anonymous or incomplete transactions**.

Customer-based analytics such as:

* Customer Lifetime Value
* Purchase frequency
* Customer segmentation

cannot be performed on such records.

## Transformation Logic

We filter out rows where:

```
CustomerID IS NULL
```

## Benefits

This ensures that:

* All transactions are linked to identifiable customers
* Customer analytics remain accurate
* Downstream models receive clean data

---

# 4. Revenue Calculation

## Purpose

The raw dataset contains:

```
Quantity
Price
```

However, **total revenue per transaction** is not directly available.

To enable revenue analytics, we create a calculated field.

## Transformation Logic

```
Revenue = Quantity × Price
```

Example:

```
6 items × £2.55 = £15.30
```

## Benefits

This derived column enables:

* Revenue reporting
* Sales dashboards
* Product performance analysis
* Financial metrics

---

# 5. Deduplication

## Purpose

Transactional systems may contain **duplicate records** due to:

* Retry operations
* System failures
* Duplicate ingestion
* Data sync issues

Duplicate rows can inflate revenue and transaction counts.

Therefore, duplicates must be removed.

## Deduplication Strategy

We use a **window function** to detect duplicates.

Example logic:

```
ROW_NUMBER() OVER (
PARTITION BY Invoice, StockCode, CustomerID
)
```

Only the first occurrence is retained.

## Benefits

This ensures:

* Accurate transaction counts
* Correct revenue totals
* Reliable analytics

---

# Resulting Clean Table

After applying all transformations, the resulting dataset will be stored in:

```
retail_staging.online_retail_clean
```

This table contains:

* Parsed timestamps
* Valid purchase transactions
* Identifiable customers
* Revenue calculations
* Deduplicated records

---

# Updated Data Pipeline Architecture

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

---

# Why This Layer is Important

The **Transformation Layer** ensures that raw operational data becomes **analytics-ready data**.

This stage improves:

* **Data Quality**
* **Data Consistency**
* **Query Performance**
* **Business Intelligence Accuracy**

Without this step, downstream dashboards, machine learning models, and analytical queries would operate on unreliable data.

---

# Next Step

The next stage in the pipeline will build the **Curated Layer**, where we will:

* Create **partitioned BigQuery tables**
* Optimize **query performance**
* Prepare datasets for **business analytics and reporting**
* Implement **incremental data loading**

The curated dataset will be stored in:

```
retail_curated.retail_orders
```

This will serve as the **final analytics-ready dataset** for reporting and dashboards.
