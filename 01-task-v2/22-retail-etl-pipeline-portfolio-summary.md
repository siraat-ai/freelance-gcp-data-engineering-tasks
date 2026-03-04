# Retail ETL Pipeline – BigQuery Data Engineering Portfolio Project

## Project Summary

This project demonstrates the implementation of a **batch ETL pipeline in Google BigQuery** using a real-world retail dataset.

The source data was provided as a **CSV file**, so instead of using streaming technologies such as Pub/Sub, the pipeline was implemented as a **batch data processing workflow directly in BigQuery**.

The pipeline extracts data from the CSV dataset into a **raw ingestion layer**, performs cleaning and transformation operations in a **staging layer**, and finally loads optimized analytical data into a **curated warehouse table**.

The curated dataset is designed for analytics and reporting and includes **partitioning, clustering, incremental loading, and duplicate protection mechanisms**.

All transformations and pipeline operations were implemented using **SQL queries executed within the BigQuery console**.

This project demonstrates key data engineering concepts such as:

* multi-layer data warehouse architecture
* data cleaning and transformation
* partitioned and clustered tables
* incremental data pipelines
* duplicate-safe ingestion strategies
* query cost optimization

---

# Project Architecture

The pipeline follows a **three-layer data warehouse architecture**.

```text id="5nkjr1"
CSV Source Dataset
        │
        ▼
Raw Layer
retail_raw.online_retail_raw
        │
        ▼
Staging Layer
retail_staging.online_retail_clean
        │
        ▼
Curated Layer
retail_curated.retail_orders
```

Each layer serves a specific role in the pipeline.

---

# Data Source

The dataset used in this project is the **Online Retail dataset**, which contains e-commerce transactions including:

* invoice numbers
* product identifiers
* quantities
* prices
* customer identifiers
* transaction timestamps
* country information

A subset of approximately **10,000 rows** was selected from the larger dataset for pipeline implementation and testing.

---

# Raw Layer

The raw layer stores the original ingested dataset without applying major transformations.

Table:

```text id="9y6o2h"
retail_raw.online_retail_raw
```

Purpose of the raw layer:

* preserve original data
* enable data traceability
* protect against schema changes
* allow reprocessing if needed

The CSV dataset was loaded directly into this table using **BigQuery's file ingestion functionality**.

---

# Staging Layer

The staging layer is responsible for **data cleaning and transformation**.

Table:

```text id="u1s3a0"
retail_staging.online_retail_clean
```

Key transformations applied:

* timestamp parsing
* removal of return transactions
* removal of records with missing customer identifiers
* revenue calculation
* duplicate record detection

Example transformation:

```text id="t4g0k2"
revenue = Quantity × Price
```

Deduplication logic was implemented using a **window function with ROW_NUMBER()**.

The staging layer ensures that only **clean and validated data** proceeds to the analytics layer.

---

# Curated Layer

The curated layer contains the **final analytics-ready dataset**.

Table:

```text id="qv1n8s"
retail_curated.retail_orders
```

This table was optimized using **partitioning and clustering strategies**.

Partitioning strategy:

```text id="g7m3r4"
PARTITION BY DATE(invoice_timestamp)
```

Clustering strategy:

```text id="r8s0w6"
CLUSTER BY CustomerID, Country
```

These optimizations improve:

* query performance
* cost efficiency
* scalability for large datasets

---

# Incremental Loading

Instead of rebuilding the entire curated table during every pipeline execution, the project implements **incremental loading**.

The pipeline uses a **timestamp watermark strategy**.

Watermark query:

```text id="0n8wq4"
SELECT MAX(invoice_timestamp)
FROM retail_curated.retail_orders
```

New records are detected using the following condition:

```text id="e0y7u2"
invoice_timestamp > MAX(invoice_timestamp)
```

This ensures that only **newly arrived records** are inserted into the curated dataset.

Benefits include:

* faster pipeline execution
* reduced query costs
* scalable data ingestion

---

# Duplicate Handling

Watermark-based incremental loading does not automatically prevent duplicates.

To address this issue, a **MERGE-based incremental strategy** was implemented.

MERGE compares incoming records with existing records using business keys such as:

```text id="k0r2n1"
Invoice
StockCode
```

If a record already exists, the insertion is skipped.

If the record is new, it is inserted into the curated table.

This ensures that the pipeline remains **duplicate-safe and idempotent**.

---

# Cost Optimization

Several strategies were used to optimize query cost in BigQuery.

Partitioning

Limits the amount of data scanned during queries.

Clustering

Improves filtering performance for frequently queried columns.

Selective queries

Avoids unnecessary full-table scans.

These techniques ensure that the pipeline remains **cost-efficient even as data volume grows**.

---

# Failure Handling Considerations

The pipeline design accounts for common failure scenarios such as:

* schema changes in source data
* duplicate transactions
* scheduler retries
* late arriving records
* data quality issues

Mitigation strategies include:

* staging layer validation rules
* deduplication logic
* incremental pipeline execution
* partition-based query filtering

---

# Skills Demonstrated

This project demonstrates several important data engineering skills.

Data ingestion into BigQuery
SQL-based data transformation
Data warehouse modeling
Incremental pipeline design
Partitioned and clustered tables
Duplicate-safe ingestion strategies
Cost optimization in BigQuery

---

# Conclusion

This project demonstrates the design and implementation of a **modern batch ETL pipeline using Google BigQuery**.

By implementing:

* layered data warehouse architecture
* data transformation and validation
* partitioned analytical tables
* incremental loading
* duplicate-safe ingestion

the pipeline reflects many of the design patterns used in **production data engineering systems**.

This project serves as a **portfolio demonstration of practical data engineering skills using Google Cloud technologies**.
