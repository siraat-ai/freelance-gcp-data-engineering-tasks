# Curated Layer – Retail Orders (Analytics Ready Table)

## Overview

After successfully cleaning and validating the data in the **Staging Layer**, the next step in the ETL pipeline is to build the **Curated Layer**.

The curated layer contains **analytics-ready datasets** that are optimized for:

* Business Intelligence dashboards
* Analytical queries
* Reporting systems
* Data science workflows

In this stage, we create the following table:

```
retail_curated.retail_orders
```

This table is built using data from the staging dataset:

```
retail_staging.online_retail_clean
```

The curated table represents the **final structured dataset** used by analysts and reporting tools.

---

# ETL Pipeline Architecture

The pipeline now follows a **three-layer architecture**:

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

Each layer has a specific purpose:

| Layer         | Purpose                                     |
| ------------- | ------------------------------------------- |
| Raw Layer     | Stores original ingested data               |
| Staging Layer | Cleans and standardizes data                |
| Curated Layer | Provides optimized analytics-ready datasets |

---

# Objective of the Curated Layer

The curated layer focuses on:

* Query performance optimization
* Cost-efficient data processing
* Scalable data storage
* Analytics-ready schema

This layer is designed to support **high-performance analytical workloads**.

---

# Partitioning Strategy

## What is Partitioning?

Partitioning divides a large table into **smaller logical segments based on a column**, usually a date or timestamp.

In this pipeline, the table is partitioned by:

```
invoice_timestamp
```

Example configuration:

```
PARTITION BY DATE(invoice_timestamp)
```

---

## Benefits of Partitioning

Partitioning improves:

* Query performance
* Cost efficiency
* Data scanning efficiency

When a query filters by date, BigQuery scans **only the relevant partition** instead of the entire table.

Example:

```
WHERE DATE(invoice_timestamp) = '2010-12-01'
```

---

# Clustering Strategy

## What is Clustering?

Clustering organizes table data based on frequently filtered columns.

In this dataset, clustering is applied using:

```
CustomerID
Country
```

Example configuration:

```
CLUSTER BY CustomerID, Country
```

---

## Benefits of Clustering

Clustering improves:

* Filtering performance
* Group-by operations
* Query execution speed

It helps BigQuery locate relevant records faster.

---

# Curated Table Schema

The curated dataset contains the following columns:

```
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

This schema provides a **clean and structured dataset** suitable for analytics.

---

# SQL Implementation

The following SQL query creates the curated table.

```sql
CREATE OR REPLACE TABLE `retail_curated.retail_orders`
PARTITION BY DATE(invoice_timestamp)
CLUSTER BY CustomerID, Country
AS

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
```

---

# Execution Result

After running the query:

* The curated table is created in the **retail_curated dataset**
* Data from the staging layer is copied into the optimized table
* Partitioning and clustering are applied

Example validation query:

```sql
SELECT COUNT(*)
FROM `retail_curated.retail_orders`
```

Expected result:

```
7373 rows
```

This confirms that the cleaned staging dataset has been successfully loaded into the curated layer.

---

# Partition Validation

To verify partitions, the following query can be executed:

```sql
SELECT
DATE(invoice_timestamp),
COUNT(*)
FROM `retail_curated.retail_orders`
GROUP BY 1
ORDER BY 1
```

This query shows how records are distributed across **date-based partitions**.

---

# Cost Considerations

BigQuery uses a **pay-per-query model**, where costs depend on the amount of data scanned.

Partitioning and clustering help reduce costs by:

* Limiting the amount of scanned data
* Improving query efficiency
* Reducing unnecessary full-table scans

Since the dataset is small (~1 MB), the query cost is effectively:

```
$0.00000
```

This pipeline remains fully within the **BigQuery free tier limits**.

---

# Resulting Data Warehouse Architecture

The final data warehouse structure is:

```
project
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

This architecture follows a **modern cloud data warehouse design**.

---

# Next Step – Incremental Data Loading

The next step in the pipeline is to implement **incremental loading**.

Instead of rebuilding the curated table each time, the pipeline will load **only new records** using a watermark strategy.

Example approach:

```
MAX(invoice_timestamp)
```

This ensures that:

* Only new transactions are processed
* Query costs remain low
* Pipeline execution remains efficient

---

# Conclusion

The curated layer provides a **final analytics-ready dataset** optimized for performance and scalability.

By applying **partitioning, clustering, and structured transformations**, the pipeline ensures:

* High data quality
* Efficient query performance
* Cost-effective analytics

This completes the core **ETL pipeline architecture** for the retail dataset.
