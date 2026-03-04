# Raw Data Preparation – Online Retail Dataset

## Overview

This document describes the preparation process for the **raw dataset** used in the ETL/ELT pipeline project. The source data comes from the **Online Retail II dataset**, which contains transactional records from an online retail store.

The objective of this step was to prepare a **manageable sample dataset** for development and testing of the data pipeline while preserving the original structure of the data.

---

# Source Dataset

Dataset Name: **Online Retail II**

Characteristics:

* Real-world e-commerce transaction dataset
* Contains approximately **1 million records**
* Covers **two years of transactions**
* Includes multiple products per invoice

Original Excel file contains two worksheets:

1. `Year 2009-2010`
2. `Year 2010-2011`

Each sheet contains the same schema representing retail transaction records.

---

# Sampling Strategy

Since the full dataset contains over one million rows, a **sample dataset** was created for development purposes.

The following sampling strategy was applied:

* **5000 rows** were selected from the sheet **Year 2009-2010**
* **5000 rows** were selected from the sheet **Year 2010-2011**

These rows were combined into a single dataset.

This resulted in a dataset containing approximately:

```
Total Rows: 10,000
Total Columns: 8
```

This dataset size is sufficient to simulate real transactional behavior while remaining efficient for development and testing within the **BigQuery free tier**.

---

# Data Merge Process

The following steps were performed in Microsoft Excel:

1. Open the original `Online Retail II.xlsx` dataset.
2. Navigate to the sheet **Year 2009-2010**.
3. Keep the header row and copy the first **5000 data rows**.
4. Navigate to the sheet **Year 2010-2011**.
5. Copy the first **5000 data rows** from this sheet (excluding the header).
6. Paste these rows below the first dataset in a single sheet.

The final sheet therefore contains:

```
Header
Rows 1 – 5000   → Year 2009-2010
Rows 5001 – 10000 → Year 2010-2011
```

No transformations, cleaning, or modifications were applied at this stage. The data remains **raw and unchanged** to preserve the integrity of the original source.

---

# Dataset Schema

The resulting dataset contains the following columns:

| Column Name | Description                                |
| ----------- | ------------------------------------------ |
| Invoice     | Unique invoice number for each transaction |
| StockCode   | Product identifier                         |
| Description | Product description                        |
| Quantity    | Number of units purchased                  |
| InvoiceDate | Timestamp of the transaction               |
| Price       | Price per unit                             |
| Customer ID | Unique identifier of the customer          |
| Country     | Country where the transaction occurred     |

---

# File Export

After merging the rows into a single sheet, the dataset was exported to CSV format.

Export settings:

```
File Name: online_retail_sample.csv
File Format: CSV (Comma Delimited)
Encoding: UTF-8 compatible
```

CSV format was selected because it is the most compatible format for ingestion into modern data warehouse systems such as **Google BigQuery**.

---

# Purpose of This Dataset

The prepared dataset will be used as the **raw ingestion layer** for the data pipeline.

Future pipeline stages will include:

* Raw data ingestion
* Data transformation
* Data cleaning
* Deduplication
* Incremental loading
* Partitioned BigQuery tables

All transformations will occur **inside the pipeline**, not during the raw data preparation stage.

---

# Repository Structure

The dataset will be stored in the repository as:

```
data/
   online_retail_sample.csv
```

This file represents the **raw input dataset** for the ETL/ELT pipeline.

---

---

✅ Is file ko repo mein is tarah rakhna best hoga:

```
repo
│
├─ data
│   └─ online_retail_sample.csv
│
├─ docs
│   └─ raw_data_preparation.md
│
└─ README.md
```

---

💡 **Next step (important for repo):**
Main chahta hoon next tum repo ke liye **ETL Architecture documentation (.md)** bhi bana lo — jo **Data Engineer portfolio ka sabse powerful part hota hai**.
