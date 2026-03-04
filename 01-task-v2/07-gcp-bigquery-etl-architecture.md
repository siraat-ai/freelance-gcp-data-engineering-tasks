# Google Cloud ETL Pipeline Architecture – Online Retail Dataset

## Overview

This document describes the **data architecture and setup process** for the ETL pipeline built using **Google Cloud BigQuery**.

The pipeline processes an e-commerce transaction dataset derived from the **Online Retail II dataset**. The architecture follows a **modern layered data engineering design**, separating raw ingestion, transformation, and analytics layers.

The goal is to build a **scalable, cost-optimized, and production-style ETL pipeline** suitable for analytics workloads.

---

# Architecture Overview

The pipeline follows a **layered data architecture** commonly used in modern cloud data platforms.

```text
Raw Data Source (CSV)
        ↓
BigQuery Raw Layer
        ↓
Staging / Transformation Layer
        ↓
Curated Analytics Layer
        ↓
Analytics / BI / Reporting
```

Each layer has a clearly defined responsibility.

---

# High-Level Data Flow

```text
online_retail_sample.csv
        ↓
Upload to BigQuery
        ↓
Dataset: retail_raw
Table: online_retail_raw
        ↓
SQL Transformation
        ↓
Dataset: retail_staging
Table: online_retail_clean
        ↓
Business Transformations
        ↓
Dataset: retail_curated
Table: retail_orders
```

---

# BigQuery Architecture

The project uses **three logical datasets** to organize the data pipeline.

```text
Project: etl-production-lab

Datasets
│
├── retail_raw
│     └── online_retail_raw
│
├── retail_staging
│     └── online_retail_clean
│
└── retail_curated
      └── retail_orders
```

---

# Layer 1 — Raw Data Layer

Dataset:

```text
retail_raw
```

Table:

```text
online_retail_raw
```

Purpose:

* Store the dataset exactly as received
* No transformation applied
* Preserve original data for auditing
* Provide reproducible ingestion

Characteristics:

* Append-only
* Schema similar to source CSV
* Used as the pipeline's **single source of truth**

Example schema:

| Column      | Type      |
| ----------- | --------- |
| Invoice     | STRING    |
| StockCode   | STRING    |
| Description | STRING    |
| Quantity    | INTEGER   |
| InvoiceDate | TIMESTAMP |
| Price       | FLOAT     |
| CustomerID  | STRING    |
| Country     | STRING    |

---

# Layer 2 — Staging Layer

Dataset:

```text
retail_staging
```

Table:

```text
online_retail_clean
```

Purpose:

The staging layer performs **initial cleaning and validation** of the raw data.

Typical transformations:

* Convert data types
* Handle missing values
* Remove invalid rows
* Prepare dataset for analytics

Example transformations:

```text
quantity > 0
price > 0
customer_id IS NOT NULL
```

Additional operations may include:

* trimming text
* normalizing values
* removing duplicates

---

# Layer 3 — Curated Layer

Dataset:

```text
retail_curated
```

Table:

```text
retail_orders
```

Purpose:

The curated layer contains **analytics-ready data** designed for reporting and business intelligence.

Typical features:

* clean dataset
* standardized schema
* optimized for query performance

Example derived column:

```text
revenue = quantity * price
```

---

# Partitioning Strategy

To reduce query cost and improve performance, the curated table will be **partitioned by transaction date**.

Partition column:

```text
DATE(invoice_date)
```

Benefits:

* Reduced query scanning
* Faster queries
* Cost optimization

Example query using partition filter:

```sql
SELECT *
FROM retail_curated.retail_orders
WHERE DATE(invoice_date) = '2011-05-01'
```

---

# Clustering Strategy

The curated table may use clustering to optimize queries.

Example clustering columns:

```text
customer_id
country
```

Clustering improves performance for queries filtering on these fields.

---

# Cost Optimization Strategy

BigQuery cost is based on **bytes scanned per query**.

The pipeline follows these best practices:

* Avoid `SELECT *`
* Query only required columns
* Use partition filters
* Limit full table scans
* Perform transformations efficiently

These practices ensure the pipeline remains **free-tier friendly during development**.

---

# Creating Fresh Architecture in BigQuery

To build the architecture from scratch, follow these steps.

---

## Step 1 — Clean Existing Resources

Delete previous practice datasets if they exist:

```text
etl_raw
etl_curated
users_raw
users_staging
```

This ensures a clean environment for the pipeline.

---

## Step 2 — Create Raw Dataset

In BigQuery Console:

```
Create Dataset
```

Dataset name:

```text
retail_raw
```

Settings:

* Location: US
* Default table expiration: none

---

## Step 3 — Create Staging Dataset

Create another dataset:

```text
retail_staging
```

Purpose:

Intermediate transformation layer.

---

## Step 4 — Create Curated Dataset

Create the final analytics dataset:

```text
retail_curated
```

Purpose:

Store business-ready tables.

---

# Raw Data Ingestion

Upload the prepared dataset:

```text
online_retail_sample.csv
```

Steps:

```
BigQuery Console
→ retail_raw
→ Create Table
→ Upload CSV
```

Table name:

```text
online_retail_raw
```

---

# Repository Structure

The GitHub repository should follow this structure:

```text
online-retail-gcp-etl
│
├── data
│   └── online_retail_sample.csv
│
├── docs
│   ├── raw-data-preparation.md
│   └── gcp-bigquery-etl-architecture.md
│
├── sql
│   └── transformations.sql
│
└── README.md
```

---

# Summary

This architecture demonstrates a **modern cloud data engineering pipeline** built using Google Cloud BigQuery.

Key features:

* Layered data architecture
* Raw data preservation
* Staging transformation layer
* Curated analytics tables
* Cost-optimized BigQuery design
* Development within free tier limits

The pipeline design follows industry best practices and can be extended to support **incremental loading, scheduling, and automated ingestion**.

---
