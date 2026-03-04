# Data Engineering Notes – ETL Pipeline Concepts

This document explains several important **data engineering concepts** used while building the ETL pipeline for the **Online Retail dataset** in BigQuery.

These notes clarify key design decisions taken during the pipeline development.

---

# 1. Parsing Errors

## Definition

**Parsing errors** occur when a data processing system tries to read and interpret input data but the **data format does not match the expected data type or structure**.

In data pipelines, parsing errors commonly occur during **data ingestion** when a system attempts to convert raw text into structured data types.

---

## Common Causes of Parsing Errors

Parsing errors usually happen due to:

* Incorrect **date or timestamp formats**
* Numeric fields containing **text values**
* **Extra commas or missing delimiters** in CSV files
* **Inconsistent column structures**
* Unexpected **NULL or blank values**

---

## Example

Expected timestamp format:

```text
2010-12-01 08:26:00
```

But the dataset may contain:

```text
01/12/2010 08:26
```

Since the format does not match the expected structure, the system cannot automatically convert the value, resulting in a **parsing error**.

---

## Practical Solution

To avoid ingestion failures:

* Raw data fields are temporarily stored as **STRING**
* Proper conversion is performed later during the **transformation stage**

---

# 2. Why the Raw Layer Stores a Flexible Schema

The **Raw Layer** is the first stage of the pipeline where data is ingested exactly as received from the source.

Instead of enforcing strict data types immediately, raw tables often use a **flexible schema**.

---

## Reasons for Flexible Schema

### 1. Prevent Ingestion Failures

Strict schemas can cause ingestion jobs to fail when unexpected data formats appear.

Storing raw fields as **STRING** prevents ingestion from breaking.

---

### 2. Preserve Original Data

The raw layer acts as a **data backup layer**, preserving the original dataset exactly as received.

This allows engineers to:

* Reprocess data later
* Debug ingestion issues
* Trace original records

---

### 3. Enable Reprocessing

If transformation logic changes, engineers can rebuild downstream datasets using the raw data.

---

# 3. Why the Staging Layer is Important

The **Staging Layer** is where raw data is **cleaned and standardized** before being used in analytics or reporting.

This layer acts as an intermediate step between raw ingestion and curated analytical datasets.

---

## Responsibilities of the Staging Layer

The staging layer performs operations such as:

* **Data cleaning**
* **Data validation**
* **Format standardization**
* **Data enrichment**
* **Data filtering**

---

## Example Transformations

In the retail dataset pipeline, staging transformations include:

* Parsing **timestamps**
* Removing **invalid transactions**
* Filtering **null customers**
* Creating **derived metrics**

---

## Benefits

The staging layer ensures that downstream datasets receive **clean and reliable data**.

This improves:

* Data quality
* Query accuracy
* Analytical reliability

---

# 4. Why Deduplication is Required in Data Pipelines

Duplicate records frequently occur in data pipelines due to system or ingestion issues.

---

## Common Causes of Duplicates

Duplicates may appear because of:

* Retry mechanisms in ingestion jobs
* Network failures during data transfer
* Batch ingestion reprocessing
* API response duplication
* Parallel data ingestion processes

---

## Problems Caused by Duplicate Records

If duplicates are not removed, they can lead to:

* Incorrect revenue calculations
* Inflated transaction counts
* Incorrect customer metrics
* Misleading business insights

---

## Deduplication Strategy

Data pipelines often remove duplicates using:

* **Window functions**
* **Primary key combinations**
* **ROW_NUMBER() ranking**

Example logic:

```text
ROW_NUMBER() OVER (
PARTITION BY Invoice, StockCode, CustomerID
)
```

Only the first occurrence is retained.

---

# 5. Why Partitioning Reduces BigQuery Cost

BigQuery pricing is based on the **amount of data scanned during queries**.

Without optimization, queries may scan entire tables even when only a small portion of data is required.

---

## What is Partitioning?

Partitioning divides a large table into **smaller logical segments based on a column**, typically a date or timestamp.

Example:

```text
PARTITION BY DATE(invoice_timestamp)
```

---

## How Partitioning Reduces Cost

When queries include a partition filter, BigQuery scans **only the relevant partitions** instead of the entire dataset.

Example:

```text
WHERE invoice_date = '2010-12-01'
```

Instead of scanning millions of rows, BigQuery scans only the **data for that specific day**.

---

## Benefits of Partitioning

Partitioned tables provide several advantages:

* Reduced query costs
* Faster query performance
* Improved scalability
* Efficient time-based analytics

---

# Summary

These design principles are essential when building scalable **data engineering pipelines**:

* Avoid ingestion failures by handling **parsing errors**
* Preserve raw data using a **flexible schema**
* Clean and standardize data in the **staging layer**
* Maintain accurate datasets through **deduplication**
* Reduce processing costs using **table partitioning**

Together, these practices help create **reliable, scalable, and cost-efficient data pipelines** in modern cloud data platforms such as **Google BigQuery**.
