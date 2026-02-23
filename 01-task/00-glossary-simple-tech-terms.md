# 📘 Data Engineering Glossary – Simple & Friendly Guide

Welcome 👋  
If you’re new to this repo, relax.  
Here are the key terms explained in **one-line simple meanings**.

No stress. Just clarity.

---

# 🏗 BigQuery Basics

## 📦 Project
👉 The top-level container in Google Cloud that holds everything.

## 🗂 Dataset
👉 A logical container (like a folder) that holds tables inside BigQuery.

## 📄 Table
👉 A structured collection of rows and columns where data is stored.

## 👁 View
👉 A saved SQL query that shows data without storing it again.

## ⚡ Materialized View
👉 A precomputed view that stores results for faster performance.

## 🧱 Partitioning
👉 Splitting a table by date or column to reduce query cost and improve speed.

## 🧩 Clustering
👉 Organizing data inside a table based on selected columns to speed up filtering.

---

# 🔄 ETL / ELT Concepts

## 🚚 Extract
👉 Bringing data from an external source into your controlled environment.

## 🧹 Transform
👉 Cleaning, modifying, and preparing raw data for analysis.

## 📥 Load
👉 Storing processed data into a data warehouse like BigQuery.

## 🔁 ETL
👉 Extract → Transform → Load (processing before storing).

## 🔄 ELT
👉 Extract → Load → Transform (processing inside the warehouse).

---

# 📡 Streaming & Messaging

## 📡 Pub/Sub
👉 Google’s messaging service that delivers real-time data between systems.

## 🔔 Message
👉 A unit of data sent from producer to consumer.

## 📬 ACK (Acknowledgment)
👉 A signal confirming that a message was successfully received.

## 🌊 Streaming
👉 Processing data continuously as it arrives.

## 📦 Batch
👉 Processing large chunks of data at scheduled intervals.

---

# ⚙️ Processing Tools

## ⚙️ Dataflow
👉 Google’s fully managed service for batch and streaming data processing.

## 🐍 Python ETL
👉 Custom script-based data processing using Python.

## 🧠 Apache Beam
👉 The programming model used by Dataflow.

---

# 🔐 Security & Governance

## 🔐 IAM (Identity and Access Management)
👉 Controls who can access what in Google Cloud.

## 🛡 Principle of Least Privilege
👉 Give only the minimum access required — nothing extra.

## 🔎 PII (Personally Identifiable Information)
👉 Data that can identify a person (email, credit card, etc.).

## 🧼 Redaction
👉 Hiding sensitive data without deleting it.

## 🧯 DLP (Data Loss Prevention)
👉 Google tool that detects and protects sensitive data automatically.

---

# 📊 Reliability & Performance

## 🔄 Fault Tolerance
👉 System continues working even if part of it fails.

## 🟢 High Availability
👉 System stays online with minimal downtime.

## 📉 Query Cost
👉 The amount of data scanned by a query (affects billing).

## 🧪 Data Quality Check
👉 Validating data for nulls, duplicates, or incorrect formats.

---

# 🏗 Architecture Layers

## 🗃 Raw Layer
👉 Unprocessed original data stored for reference.

## 🧹 Curated Layer
👉 Cleaned and structured data ready for analytics.

## 📊 Analytics Layer
👉 Optimized data used by dashboards and reports.

---

# 🚀 Final Note

If any term feels confusing:

Don’t panic.  
Scroll here.  
Read one line.  
Move forward.

Data engineering is not magic.  
It’s structured thinking.

Enjoy the build 💪
```

---


