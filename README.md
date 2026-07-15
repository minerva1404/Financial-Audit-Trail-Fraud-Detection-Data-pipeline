# 💰 Financial Audit Trail & Fraud Detection Pipeline

End-to-End Financial Data Engineering Pipeline using MySQL, PySpark & Medallion Architecture

A production-inspired batch data pipeline that processes large-scale financial transactions through a Bronze → Silver → Gold Medallion Architecture, ensuring data quality, detecting suspicious activities, and generating analytics-ready datasets for audit reporting and fraud monitoring.

---

## 📑 Table of Contents

- [Project Overview](#project-overview)
- [Solution Architecture](#️-solution-architecture)
- [Key Features](#-key-features)
- [Project Highlights](#-project-highlights)
- [Key Business Insights](#-key-business-insights)
- [Technology Stack](#️-technology-stack)
- [Repository Structure](#-repository-structure)
- [Pipeline Workflow](#-pipeline-workflow)
- [Getting Started](#-getting-started)
- [Running the Pipeline](#️-running-the-pipeline)
- [Engineering Decisions](#-engineering-decisions)
- [Future Enhancements](#-future-enhancements)
- [License](#-license)

---

## Project Overview

Financial institutions process millions of transactions every day, making data quality, auditability, and fraud detection critical business requirements.

This project demonstrates how a scalable Medallion Architecture transforms raw financial transaction data into trusted analytical datasets using MySQL, PySpark, and SQL. The pipeline progressively improves data quality through validation, cleansing, enrichment, and aggregation before producing business-ready KPIs for fraud analysis and reporting.

The project simulates a high-volume financial environment with over 500,000 transactions, enabling realistic audit and anomaly detection scenarios.

---

## 🏗️ Solution Architecture

The pipeline follows the Medallion Architecture to progressively refine financial transaction data.

<img width="1536" height="1024" alt="finance_pipeline_architecture" src="https://github.com/user-attachments/assets/860f57cd-6ec7-48cf-bd23-6ed79b128351" />

---

## ✨ Key Features

* Large-scale financial transaction processing
* Bronze → Silver → Gold Medallion Architecture
* Automated data validation and cleansing
* Duplicate detection and schema standardization
* Fraud classification using business rules
* Invalid transaction detection
* Daily financial KPI generation
* Analytics-ready Gold layer
* Modular ETL pipeline design
* Power BI dashboard integration

---

## 📊 Project Highlights

| Metric |	Value |
|--------|--------|
| Transactions Processed |	500,000+ |
| Users | 	10,000+ |
| Invalid Records |	10,094 (2.02%) |
| Suspicious Transactions |	28,400 (5.68%) |
| Pipeline Architecture |	Bronze → Silver → Gold |
| Processing Framework |	PySpark |
| Database |	MySQL |
| Visualization |	Power BI |

---

## 📈 Key Business Insights

The pipeline generated several analytical insights for financial monitoring:

* 💰 Processed over 500,000 financial transactions
* 🚨 Identified 28,400 suspicious transactions requiring further investigation
* ⚠️ Detected 10,094 invalid records through data quality validation
* 📊 Classified transactions into Normal, High Amount, Extreme Amount, and Negative Balance categories
* 📅 Produced daily KPIs for transaction volume, fraud rate, and financial activity
* 📈 Delivered analytics-ready datasets for audit reporting and executive dashboards

These insights demonstrate how data engineering pipelines can support financial governance, compliance, and fraud detection.

---

## ⚙️ Technology Stack

### Programming

* Python
* SQL
* PySpark

### Database

* MySQL

### Visualization

* Power BI

### Engineering Concepts

* Medallion Architecture
* ETL Pipelines
* Batch Data Processing
* Data Validation
* Data Cleansing
* Fraud Detection
* Data Quality Engineering
* Financial Analytics

---

## 🔄 Pipeline Workflow

### Bronze Layer

Raw financial transactions are loaded into MySQL and stored without modification, preserving the original records for auditability and lineage.

### Silver Layer

The Silver layer performs data quality improvements, including:

* Schema validation
* Null value handling
* Duplicate removal
* Transaction validation
* Data normalization
* Fraud flag generation

This layer produces trusted datasets suitable for downstream processing.

### Gold Layer

The Gold layer computes business-level metrics including:

* Daily transaction summaries
* Fraud classifications
* Transaction volume analysis
* Invalid record statistics
* Average transaction value
* Fraud rate KPIs

These aggregated datasets are optimized for business intelligence and reporting.

---

## 🚀 Getting Started

### 1. Clone the Repository
```
git clone https://github.com/minerva1404/Financial-Audit-Trail-Fraud-Detection-Data-pipeline.git
cd Financial-Audit-Trail-Fraud-Detection-Data-pipeline
```
### 2. Create a Virtual Environment

``` python -m venv venv ```
Linux / macOS
``` source venv/bin/activate ```
Windows
``` venv\Scripts\activate ```

### 3. Install Dependencies

``` pip install -r requirements.txt ```

### 4. Load Bronze Data

Run the Bronze SQL script:

source fintech_bronze.sql;

---

## ▶️ Running the Pipeline

### Step 1 — Bronze Layer

source fintech_bronze.sql;

Loads raw financial transaction data into the Bronze layer.

### Step 2 — Silver Layer

source fintech_silver.sql;

Performs validation, cleansing, normalization, and fraud flag generation.

### Step 3 — Gold Layer

source fintech_gold.sql;

Generates aggregated KPIs and business-ready analytical tables.

### Step 4 — Dashboard

Open

fintech_dashboard.pbix

to explore fraud trends, transaction quality metrics, and financial KPIs.

---

## 💡 Engineering Decisions

### Why Medallion Architecture?

Separating Bronze, Silver, and Gold layers improves maintainability, auditability, and analytical reliability.

### Why MySQL?

MySQL provides a robust relational storage layer for structured financial transactions while supporting efficient SQL-based transformations.

### Why PySpark?

PySpark enables scalable processing of hundreds of thousands of transaction records while simplifying data transformation workflows.

### Why Rule-Based Fraud Detection?

Business-rule thresholds provide transparent, explainable fraud identification that is easy to audit and extend for future machine learning models.

---

## 🔮 Future Enhancements

* Apache Kafka for real-time transaction ingestion
* Spark Structured Streaming support
* Delta Lake integration
* Machine Learning fraud scoring
* Apache Airflow orchestration
* Docker deployment
* AWS cloud deployment
* Automated data quality testing
* Real-time fraud alerting
* CI/CD using GitHub Actions

---

## 📄 License

This project is intended for educational and portfolio purposes.

---
## Summary

This project demonstrates modern data engineering practices including Medallion Architecture, scalable ETL pipelines, data quality engineering, and rule-based fraud detection, providing a strong foundation for production-grade financial analytics systems.
