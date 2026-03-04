# Curated Layer – Retail Orders (Analytics Ready Dataset)

## Overview

The **Curated Layer** is the final stage of the ETL pipeline where cleaned and validated data from the **staging layer** is transformed into an **analytics-ready dataset**.

This layer is designed for:

* Business Intelligence dashboards
* Data analysis
* Reporting
* Machine learning pipelines

The curated dataset will be stored in the following BigQuery table:

```text
retail_curated.retail_orders
```

Unlike raw and staging layers, curated tables are **optimized for performance, scalability, and cost efficiency**.

---

# Curated Layer Objectives

The curated dataset aims to provide:

* Clean transactional data
* Reliable business metrics
* Optimized query performance
* Scalable analytical storage

This dataset becomes the **primary source for analytics and reporting tools**.

---

# 1. Partitioned Tables

## What is Partitioning?

Partitioning divides a large table into **smaller segments based on a specific column**, typically a date or timestamp.

Example:

```text
PARTITION BY DATE(invoice_timestamp)
```

Each partition stores records for a specific date.

---

## Why Partitioning is Important

Without partitioning, BigQuery scans the **entire table** for every query.

With partitioning, BigQuery scans **only relevant partitions**, reducing:

* Query execution time
* Amount of scanned data
* Query processing cost

---

## Example

Query with partition filter:

```text
WHERE invoice_date = '2010-12-01'
```

BigQuery scans only the **partition for that specific day** instead of the full dataset.

---

# 2. Incremental Data Loading

## What is Incremental Loading?

Incremental loading means **only new or updated data is processed**, rather than reprocessing the entire dataset.

This approach is essential for **scalable data pipelines**.

---

## Why Incremental Loads Are Important

Processing the full dataset repeatedly can:

* Increase processing costs
* Slow down pipelines
* Waste computing resources

Incremental loading ensures that the pipeline processes **only the latest records**.

---

## Watermark Strategy

A common incremental strategy is to use a **watermark column**, such as:

```text
InvoiceDate
```

The pipeline tracks the **maximum processed timestamp**.

Example logic:

```text
SELECT MAX(invoice_timestamp)
FROM retail_curated.retail_orders
```

New records are loaded only if their timestamp is **greater than the previous watermark**.

---

# 3. Cost Optimization

BigQuery uses a **pay-per-query model**, meaning users are charged based on the amount of data scanned.

Several strategies help reduce cost.

---

## Partition Filtering

Queries should always include a **partition filter**:

```text
WHERE invoice_date >= '2011-01-01'
```

This prevents scanning the entire table.

---

## Column Selection

Instead of:

```text
SELECT *
```

It is better to select only required columns:

```text
SELECT invoice_id, revenue
```

This reduces scanned data.

---

## Clustering (Optional Enhancement)

Clustering organizes table storage based on frequently filtered columns.

Example:

```text
CLUSTER BY CustomerID, Country
```

Benefits include:

* Faster filtering
* Improved query performance

---

# 4. Interview Explanation

When explaining this pipeline in an interview, the curated layer can be described as follows:

---

## How the Pipeline Works

The ETL pipeline follows a **three-layer architecture**:

```text
Raw Layer
   ↓
Staging Layer
   ↓
Curated Layer
```

---

## Raw Layer

Stores the original dataset as ingested from the source.

Example table:

```text
retail_raw.online_retail_raw
```

This layer preserves the **original data structure**.

---

## Staging Layer

Applies cleaning and transformation operations such as:

* Timestamp parsing
* Removing invalid records
* Deduplication
* Revenue calculation

Example table:

```text
retail_staging.online_retail_clean
```

---

## Curated Layer

Provides optimized analytical datasets.

Example table:

```text
retail_curated.retail_orders
```

Features include:

* Partitioned tables
* Clean business metrics
* Optimized query performance
* Incremental loading support

---

# Example Architecture

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

---

# Scalability Considerations

This architecture can easily scale to handle:

* Millions of daily transactions
* Large analytical workloads
* Real-time reporting systems

BigQuery's serverless architecture ensures the pipeline remains **highly scalable and cost-efficient**.

---

# Conclusion

The curated layer provides a **final, business-ready dataset** that supports:

* Business intelligence dashboards
* Analytical reporting
* Data science workflows
* Operational decision-making

By using **partitioning, incremental loads, and optimized queries**, this layer ensures the pipeline remains **scalable, reliable, and cost-efficient**.
