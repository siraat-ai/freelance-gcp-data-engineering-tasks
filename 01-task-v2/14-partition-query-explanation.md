# Partition Query Explanation – Retail Orders Dataset

## Overview

After creating the curated analytics table **`retail_curated.retail_orders`**, a validation query was executed to examine how records are distributed across partitions.

The query used was:

```sql
SELECT
DATE(invoice_timestamp),
COUNT(*)
FROM `retail_curated.retail_orders`
GROUP BY 1
ORDER BY 1
```

This query helps confirm that **partitioning is working correctly** and provides visibility into how records are distributed by date.

---

# Query Result

The output of the query produced the following results:

| Date       | Records |
| ---------- | ------- |
| 2009-12-01 | 2133    |
| 2009-12-02 | 1659    |
| 2010-12-01 | 1849    |
| 2010-12-02 | 1732    |

This result indicates that the dataset contains records for **four distinct transaction dates**.

---

# Why Only Four Dates Appear

The curated table was built using a **sample subset of the original Online Retail dataset**.

While the full dataset spans multiple months and years, the extracted sample contained transactions from only the following dates:

* 2009-12-01
* 2009-12-02
* 2010-12-01
* 2010-12-02

As a result, the partition distribution reflects these **four unique transaction dates**.

If a larger portion of the original dataset were used, additional partitions would appear automatically.

---

# Understanding `GROUP BY 1`

In SQL, `GROUP BY 1` is a shorthand syntax.

It means:

```sql
GROUP BY first column in the SELECT clause
```

In this query, the first column is:

```sql
DATE(invoice_timestamp)
```

Therefore the query is equivalent to:

```sql
SELECT
DATE(invoice_timestamp),
COUNT(*)
FROM `retail_curated.retail_orders`
GROUP BY DATE(invoice_timestamp)
ORDER BY DATE(invoice_timestamp)
```

Using numeric references simplifies queries and is commonly used in analytical SQL.

---

# Understanding `ORDER BY 1`

Similarly, `ORDER BY 1` instructs SQL to sort the results using the **first column in the SELECT clause**.

Equivalent statement:

```sql
ORDER BY DATE(invoice_timestamp)
```

This ensures that the results appear in **chronological order**.

---

# Partition Structure in BigQuery

Since the curated table was created using:

```sql
PARTITION BY DATE(invoice_timestamp)
```

BigQuery internally stores the data in separate partitions.

The logical structure of the table looks like this:

```
retail_orders
│
├── partition: 2009-12-01 (2133 rows)
├── partition: 2009-12-02 (1659 rows)
├── partition: 2010-12-01 (1849 rows)
└── partition: 2010-12-02 (1732 rows)
```

Each partition contains only the records belonging to that specific date.

---

# Benefits of Partitioned Storage

Partitioning provides several advantages in data warehouses:

### Faster Queries

Queries filtering by date scan only the relevant partition.

Example:

```sql
SELECT *
FROM `retail_curated.retail_orders`
WHERE DATE(invoice_timestamp) = '2010-12-01'
```

Only that partition will be scanned.

---

### Lower Query Cost

BigQuery charges based on the amount of data scanned.

By scanning only one partition instead of the entire table, query costs are significantly reduced.

---

### Scalable Data Architecture

Partitioned tables scale efficiently when datasets grow to **millions or billions of records**.

New partitions are automatically created when new dates appear in the data.

---

# Data Engineering Best Practice

Using shorthand syntax such as `GROUP BY 1` and `ORDER BY 1` is common in analytical SQL because it:

* Keeps queries concise
* Simplifies column references
* Improves readability in complex queries

Example:

```sql
SELECT
Country,
SUM(revenue)
FROM `retail_curated.retail_orders`
GROUP BY 1
ORDER BY 2 DESC
```

Equivalent query:

```sql
SELECT
Country,
SUM(revenue)
FROM `retail_curated.retail_orders`
GROUP BY Country
ORDER BY SUM(revenue) DESC
```

---

# Summary

The partition validation query confirms that:

* The curated table was successfully partitioned by `invoice_timestamp`
* Records are distributed across four date partitions
* SQL shorthand (`GROUP BY 1`, `ORDER BY 1`) correctly references columns in the SELECT clause

This validation ensures that the curated data warehouse table is **properly structured, optimized, and ready for analytical queries**.
