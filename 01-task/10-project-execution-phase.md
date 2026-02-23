# 🚀 Task 1 – Project Execution Phase  
## Real-World ETL Implementation Using Google Public Dataset

---

# 📌 Project Objective

The objective of this execution phase is to implement a production-style ETL/ELT pipeline using a real Google BigQuery public dataset instead of dummy CSV files.

This approach ensures:

- Realistic business data structure
- Enterprise-relevant schema handling
- Portfolio-ready architecture
- Interview-defensible implementation

---

# 🏢 Selected Dataset

Dataset Used:

`bigquery-public-data.thelook_ecommerce`

This dataset simulates a real-world e-commerce business including:

- Orders
- Users
- Products
- Events

Primary table used for transformation:

`thelook_ecommerce.orders`

---

# 🏗 Architecture Overview

Public Dataset (BigQuery)
        ↓
Raw Dataset (Project Controlled)
        ↓
Transformation Layer
        ↓
Curated Dataset (Partitioned + Clustered)
        ↓
Analytics-Ready Table

---

# 🧱 Execution Phases

---

## 1️⃣ Create Project & Enable APIs

- Create new Google Cloud project
- Enable:
  - BigQuery API
  - Dataflow API (if used)
  - Cloud Scheduler API
  - Cloud Storage API

---

## 2️⃣ Create Controlled Raw Dataset

Create a new dataset:

`task1_raw`

Copy subset of public dataset into controlled environment:

Example:

```sql
CREATE OR REPLACE TABLE task1_raw.orders_raw AS
SELECT *
FROM `bigquery-public-data.thelook_ecommerce.orders`
LIMIT 100000;
````

This creates a working boundary under your project.

---

## 3️⃣ Data Quality Assessment

Perform checks:

* Identify NULL values
* Detect duplicates
* Validate schema consistency
* Check date formatting

Example:

```sql
SELECT order_id, COUNT(*)
FROM task1_raw.orders_raw
GROUP BY order_id
HAVING COUNT(*) > 1;
```

---

## 4️⃣ Transformation Layer

Create curated dataset:

`task1_curated`

Apply transformations:

* Remove duplicates
* Drop NULL critical fields
* Standardize timestamps
* Add derived columns (year, month)
* Normalize country codes

Example:

```sql
CREATE OR REPLACE TABLE task1_curated.orders_clean
PARTITION BY DATE(created_at)
CLUSTER BY user_id AS
SELECT DISTINCT
    order_id,
    user_id,
    created_at,
    DATE(created_at) AS order_date,
    EXTRACT(YEAR FROM created_at) AS order_year,
    EXTRACT(MONTH FROM created_at) AS order_month,
    status,
    sale_price
FROM task1_raw.orders_raw
WHERE created_at IS NOT NULL;
```

---

## 5️⃣ Storage Optimization

Applied:

* Partitioning by `DATE(created_at)`
* Clustering by `user_id`

Benefits:

* Reduced query scan cost
* Faster analytical queries
* Improved performance for filtered queries

---

## 6️⃣ Query Cost Testing

Compare:

Without partition filter:

```sql
SELECT COUNT(*) FROM task1_curated.orders_clean;
```

With partition filter:

```sql
SELECT COUNT(*)
FROM task1_curated.orders_clean
WHERE order_date = '2024-01-01';
```

Observe difference in scanned bytes.

---

## 7️⃣ Automation Layer

Implement scheduled transformation:

* Create scheduled query OR
* Trigger pipeline via Cloud Scheduler
* Enable logging & monitoring

This converts manual execution into production-grade workflow.

---

# 🧠 Architecture Decisions

* ELT approach preferred (BigQuery-native transformation)
* Partition-first warehouse design
* Controlled raw ingestion boundary
* Curated analytics layer separation

---

# 💼 Market Positioning

This implementation demonstrates:

* Real dataset handling
* Data quality engineering
* Cost optimization awareness
* Partitioning & clustering expertise
* Production automation capability

This aligns directly with enterprise Google Cloud Data Engineering requirements.

---

# 🎯 Final Outcome

The project now includes:

✔ Real-world dataset ingestion
✔ Controlled raw data boundary
✔ Cleaned & analytics-ready dataset
✔ Optimized BigQuery storage
✔ Automated execution pipeline

Task 1 – Execution Phase Complete.

---

