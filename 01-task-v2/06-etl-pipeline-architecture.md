# ETL / ELT Pipeline Architecture – Online Retail Project

## Overview

This document describes the architecture of the **ETL / ELT data pipeline** developed for the Online Retail dataset. The pipeline is designed following **modern data engineering best practices**, focusing on scalability, cost optimization, and incremental processing.

The goal of this pipeline is to transform raw retail transaction data into an **analytics-ready dataset in BigQuery**.

---

# Pipeline Architecture

The pipeline follows a **layered architecture model** commonly used in modern cloud data platforms.

Data flows through the following layers:

```text
Source Dataset
     ↓
Extraction Layer
     ↓
Raw Data Layer (BigQuery)
     ↓
Transformation Layer
     ↓
Curated Analytics Layer
     ↓
Analytics / BI
```

Each layer is responsible for a specific stage of the data lifecycle.

---

# Data Source

The pipeline uses a **sample dataset derived from the Online Retail II dataset**.

Source file:

```text
online_retail_sample.csv
```

Dataset characteristics:

* Transactional e-commerce data
* ~10,000 rows
* 8 columns
* Real-world retail transactions

This dataset simulates real production retail data while remaining small enough for development and testing.

---

# Extraction Layer

The extraction layer is responsible for ingesting the raw dataset into the data warehouse.

Responsibilities:

* Reading the CSV dataset
* Validating schema structure
* Uploading the dataset into BigQuery
* Ensuring correct data types during ingestion

At this stage, the data remains **unaltered and raw**.

---

# Raw Data Layer

The raw layer stores the dataset exactly as it was received from the source.

Characteristics of the raw layer:

* Append-only design
* No transformations applied
* Schema closely matches the source file
* Used for reproducibility and debugging

Example raw table:

```text
retail_pipeline.raw_online_retail
```

The raw layer acts as the **single source of truth for the pipeline**.

---

# Transformation Layer

The transformation layer prepares the data for analytics and reporting.

Typical operations include:

* Data type standardization
* Null value handling
* Deduplication
* Business rule enforcement
* Derived column creation

Example transformations:

Revenue calculation:

```text
revenue = quantity * price
```

Data filtering:

```text
quantity > 0
price > 0
```

These transformations convert raw transactional data into **clean business data**.

---

# Curated Data Layer

The curated layer contains analytics-ready datasets optimized for querying.

Characteristics:

* Cleaned and validated data
* Partitioned tables for performance
* Business-friendly schema
* Optimized for analytical queries

Example curated table:

```text
retail_pipeline.orders_curated
```

Partitioning strategy:

```text
PARTITION BY DATE(invoice_date)
```

Clustering example:

```text
CLUSTER BY customer_id, country
```

This design improves **query performance and cost efficiency** in BigQuery.

---

# Incremental Processing Strategy

The pipeline is designed to support **incremental data loading**.

Instead of reloading the entire dataset each time, only new records are processed.

Watermark logic example:

```text
MAX(invoice_date)
```

New data condition:

```text
invoice_date > last_loaded_timestamp
```

Benefits:

* Prevents duplicate records
* Reduces query cost
* Improves pipeline performance

---

# Cost Optimization Strategy

The pipeline follows several cost optimization practices for BigQuery.

Key principles:

* Avoid `SELECT *` queries
* Use partition filters
* Query only required columns
* Reduce unnecessary full table scans

These practices ensure the pipeline remains **efficient and free-tier friendly during development**.

---

# Monitoring and Logging (Future Implementation)

Future pipeline improvements may include:

* Execution logging
* Row count validation
* Error detection
* Pipeline performance monitoring

These features are typically implemented using:

* Cloud Logging
* Cloud Scheduler
* Alerting mechanisms

---

# Repository Structure

The project repository follows a structured layout:

```text
project-repo
│
├── data
│   └── online_retail_sample.csv
│
├── docs
│   ├── raw_data_preparation.md
│   └── etl-pipeline-architecture.md
│
├── sql
│   └── transformations.sql
│
└── README.md
```

This structure helps maintain a clean and organized data engineering project.

---

# Summary

This ETL / ELT pipeline demonstrates a typical **cloud data engineering workflow**:

* Raw data ingestion
* Structured data transformation
* Cost-efficient BigQuery design
* Incremental processing strategy

The architecture is designed to be **scalable, maintainable, and production-ready**, making it suitable for both portfolio demonstration and real-world implementation.

---
