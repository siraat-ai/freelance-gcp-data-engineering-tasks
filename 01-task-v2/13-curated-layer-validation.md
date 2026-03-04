# Curated Layer Validation – Retail Orders

## Overview

After creating the curated analytics table **`retail_curated.retail_orders`**, the next step was to validate the data warehouse layer.

The purpose of validation is to ensure that:

* The expected number of records were loaded
* Partitioning has been applied correctly
* Data transformations remain intact
* The curated dataset is ready for analytics workloads

This step confirms that the **final data warehouse table has been successfully built**.

---

# Curated Table Details

Table created:

```
retail_curated.retail_orders
```

Source table:

```
retail_staging.online_retail_clean
```

Table features:

* **Partitioned by**: `invoice_timestamp`
* **Clustered by**: `CustomerID`, `Country`
* **Analytics ready schema**

---

# Row Count Validation

To confirm that the curated layer contains the correct number of records, the following query was executed.

```sql
SELECT COUNT(*)
FROM `retail_curated.retail_orders`
```

Result:

```
7373 rows
```

This confirms that the curated dataset contains the **same number of cleaned records as the staging dataset**, indicating that no data loss occurred during the transformation.

---

# Partition Validation

To confirm that partitioning was applied correctly, the following query was executed.

```sql
SELECT
DATE(invoice_timestamp),
COUNT(*)
FROM `retail_curated.retail_orders`
GROUP BY 1
ORDER BY 1
```

Results:

| Date       | Records |
| ---------- | ------- |
| 2009-12-01 | 2133    |
| 2009-12-02 | 1659    |
| 2010-12-01 | 1849    |
| 2010-12-02 | 1732    |

This confirms that the table has been **successfully partitioned by date**.

---

# Partition Structure

Internally, BigQuery organizes the table into partitions based on the date extracted from the invoice timestamp.

```
retail_orders
│
├── partition: 2009-12-01
├── partition: 2009-12-02
├── partition: 2010-12-01
└── partition: 2010-12-02
```

This structure improves query performance and reduces the amount of scanned data.

---

# Clustering Validation

The table was clustered using:

```
CustomerID
Country
```

Clustering improves query performance when filtering or aggregating data using these columns.

Example optimized analytical query:

```sql
SELECT
Country,
SUM(revenue)
FROM `retail_curated.retail_orders`
GROUP BY Country
ORDER BY SUM(revenue) DESC
```

Because of clustering, BigQuery can locate relevant records faster.

---

# Data Warehouse Architecture

The final ETL architecture follows a **three-layer warehouse design**.

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

Each layer has a distinct responsibility:

| Layer   | Purpose                              |
| ------- | ------------------------------------ |
| Raw     | Stores original ingested data        |
| Staging | Performs cleaning and transformation |
| Curated | Provides analytics-ready datasets    |

---

# Cost Optimization

The curated table benefits from two major cost optimization strategies:

### Partitioning

Queries scanning a specific date only access the relevant partition.

Example:

```sql
WHERE DATE(invoice_timestamp) = '2010-12-01'
```

### Clustering

Grouping and filtering operations on **CustomerID** and **Country** execute faster.

Because the dataset is small (~710 KB), the query cost remains extremely low and falls well within the **BigQuery free tier limits**.

---

# Example Analytics Queries

### Revenue by Country

```sql
SELECT
Country,
SUM(revenue) AS total_revenue
FROM `retail_curated.retail_orders`
GROUP BY Country
ORDER BY total_revenue DESC
```

### Daily Sales Trend

```sql
SELECT
DATE(invoice_timestamp) AS sales_date,
SUM(revenue) AS daily_revenue
FROM `retail_curated.retail_orders`
GROUP BY sales_date
ORDER BY sales_date
```

### Top Customers by Revenue

```sql
SELECT
CustomerID,
SUM(revenue) AS total_revenue
FROM `retail_curated.retail_orders`
GROUP BY CustomerID
ORDER BY total_revenue DESC
LIMIT 10
```

---

# ETL Pipeline Status

The ETL pipeline is now fully operational.

Completed stages:

* Raw data ingestion
* Data cleaning and transformation
* Deduplication
* Revenue calculation
* Partitioned warehouse table creation
* Clustering optimization
* Data validation

---

# Next Step – Incremental Loading

The next stage is to implement **incremental loading logic**.

Instead of rebuilding the curated table each time, the pipeline will load only **new records** based on the latest timestamp.

Example watermark strategy:

```
MAX(invoice_timestamp)
```

Benefits of incremental loading:

* Faster pipeline execution
* Lower query costs
* Scalable production pipeline

---

# Conclusion

The curated layer successfully provides a **clean, optimized, and analytics-ready dataset**.

By applying **partitioning, clustering, and validation checks**, the pipeline now follows a modern **cloud data warehouse architecture** suitable for production-scale analytics.
