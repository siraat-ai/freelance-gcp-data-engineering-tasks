# Handling Duplicate Records in Incremental Pipelines

## Watermark Strategy vs MERGE Operation

## Introduction

When implementing **incremental loading** in data pipelines, a common question arises:

> **If the watermark strategy already loads only new records, why do we still need a MERGE operation?**

This question highlights an important distinction between **detecting newly arrived data** and **preventing duplicate records** in production pipelines.

Understanding the difference between these two mechanisms is essential for designing **robust and reliable data engineering pipelines**.

---

# What the Watermark Strategy Does

The **watermark strategy** is used to detect new data that has arrived since the last pipeline execution.

It typically relies on a **timestamp column** such as:

```text id="x9xt3g"
invoice_timestamp
```

The pipeline checks the most recent processed timestamp using:

```sql id="y2l3tf"
SELECT MAX(invoice_timestamp)
FROM `retail_curated.retail_orders`
```

This value represents the **latest processed record**.

The pipeline then loads only records with a newer timestamp.

Example logic:

```sql id="sv2n8d"
WHERE invoice_timestamp >
(
SELECT MAX(invoice_timestamp)
FROM `retail_curated.retail_orders`
)
```

This ensures that the pipeline processes **only newly arrived data**.

---

# Limitation of the Watermark Strategy

While the watermark strategy identifies new records by timestamp, it **does not guarantee that the records are unique**.

Consider the following scenario.

## Existing Records in the Curated Table

| Invoice | Timestamp        |
| ------- | ---------------- |
| 1001    | 2010-12-02 15:20 |
| 1002    | 2010-12-02 15:21 |

Latest processed timestamp:

```text id="7x78zq"
2010-12-02 15:21
```

---

## Incoming Record in the Staging Table

| Invoice | Timestamp        |
| ------- | ---------------- |
| 1002    | 2010-12-03 09:00 |

This record has:

* a **newer timestamp**
* but the **same Invoice number**

Because the timestamp is newer, the watermark logic will insert the record again.

Result:

```text id="b3gox8"
Duplicate record inserted
```

This demonstrates that **watermark filtering alone does not prevent duplicates**.

---

# What MERGE Does

The **MERGE statement** is used to compare incoming data with existing records in the target table.

It allows the pipeline to determine whether a record:

* already exists
* should be updated
* should be inserted

MERGE performs a conditional operation known as **UPSERT**.

UPSERT combines two actions:

```text id="30jkmf"
UPDATE + INSERT
```

---

# Example MERGE Condition

A typical MERGE operation compares business keys such as:

```sql id="xg8a7j"
ON T.Invoice = S.Invoice
AND T.StockCode = S.StockCode
```

This condition tells the pipeline:

```text id="vtpnd7"
If the same Invoice and StockCode exist,
the record already exists.
```

---

# MERGE Behavior

MERGE supports three primary actions.

## 1. When the Record Already Exists

```text id="meum97"
WHEN MATCHED
```

Possible actions:

* Update existing record
* Ignore the incoming record

---

## 2. When the Record Is New

```text id="e3r6wk"
WHEN NOT MATCHED
```

Action:

```text id="s8a3rp"
INSERT the new record
```

---

## Example Scenario

### Existing Curated Table

| Invoice | StockCode |
| ------- | --------- |
| 1001    | A12       |
| 1002    | B33       |

---

### Incoming Staging Data

| Invoice | StockCode |
| ------- | --------- |
| 1002    | B33       |
| 1003    | C11       |

---

### MERGE Result

| Invoice | StockCode |
| ------- | --------- |
| 1001    | A12       |
| 1002    | B33       |
| 1003    | C11       |

Explanation:

* Record **1002** already existed → skipped
* Record **1003** was new → inserted

This prevents duplicate transactions from entering the curated dataset.

---

# Combining Watermark and MERGE

In modern production pipelines, both techniques are used together.

### Step 1 – Watermark Filter

The watermark reduces the amount of data processed by selecting only recently arrived records.

```text id="q9dlzc"
Process only the newest time window
```

---

### Step 2 – MERGE Operation

MERGE ensures that duplicate records are not inserted into the curated table.

```text id="0zq7s6"
Validate uniqueness before inserting records
```

---

# Production Architecture

The combined pipeline structure typically looks like this:

```text id="3rj3aj"
Staging Layer
      │
      ▼
Watermark Filter
      │
      ▼
MERGE into Curated Table
```

This approach ensures:

* Efficient data processing
* Duplicate protection
* Reliable incremental pipelines

---

# Summary

| Component          | Purpose                   |
| ------------------ | ------------------------- |
| Watermark Strategy | Detect newly arrived data |
| MERGE Operation    | Prevent duplicate records |
| Partitioning       | Reduce query scan cost    |
| Clustering         | Improve query performance |

Together, these techniques form the foundation of **modern production-grade data pipelines**.

---

# Key Takeaway

The watermark strategy determines **which records should be processed**, while the MERGE operation ensures **data integrity by preventing duplicates**.

Using both techniques together enables pipelines to remain:

* scalable
* cost-efficient
* reliable in production environments.
