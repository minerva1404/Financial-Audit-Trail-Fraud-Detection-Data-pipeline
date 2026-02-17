# 💰 Financial Audit Trail & Fraud Detection Data Pipeline

## Project Overview
This project implements a *medallion (Bronze-Silver-Gold) data pipeline* to process, clean, and analyze synthetic financial transaction data for *audit trail and fraud detection* purposes. It simulates a high-volume transaction environment across multiple users, identifies invalid and suspicious transactions, and provides analytical summaries for monitoring and reporting.

## *Key Outcomes:*
- Processed *500,000+ transactions across 10,000 users*  
- Detected *28,400 suspicious transactions (5.68%)* and *10,094 invalid records (2.02%)*  
- Generated *gold-layer aggregates* for daily KPI monitoring and fraud classification  
- Normalized and validated transaction data ensuring *analytics-ready datasets*  

---

## 🛠 Tech Stack
- *Python 3.x* – Data processing and simulation scripts  
- *MySQL* – Storage of raw and cleaned transaction data  
- *PySpark* – Data transformations and medallion pipeline  
- *SQL* – Analytical queries and aggregation  
- *Power BI* – Dashboard visualization of KPIs and fraud summaries  


## ⚙️ Setup & Usage

### 1️⃣ Clone the repository
```bash
git clone <https://github.com/minerva1404/Financial-Audit-Trail-Fraud-Detection-Data-pipeline>
cd financial-audit-fraud-pipeline
```
### 2️⃣ Set up Python environment
```bash
python -m venv venv
source venv/bin/activate  # Linux/Mac
venv\Scripts\activate     # Windows
pip install -r requirements.txt
```
### 3️⃣ Load Raw Transaction Data

-- Run this in MySQL to create synthetic transactions\
source: fintech_bronze.sql;

### 4️⃣ Transform to Silver Layer

-- Run the silver transformation queries\
source: fintech_silver.sql;

### 5️⃣ Aggregate Gold Layer

-- Run the gold aggregation queries\
source : fintech_gold.sql;

### 6️⃣ Visualize KPIs
	•	Open fintech_dashboard.pbix in Power BI to view metrics like fraud distribution, daily transactions, and top-risk accounts.



## 📊 Analytical Highlights
•	Fraud Classification: Normal, Negative Balance, Extreme Amount, High Amount.\
•	Transaction Quality: Invalid Users, Missing Timestamps, Negative Amounts.\
•	Daily Metrics: Total transactions, volume, average transaction amount, suspicious transactions.\
•	KPI Insights: Fraud percent, invalid percent, high-value transactions, and anomaly detection.



## 🔍 Key Learnings:
•	Built end-to-end medallion architecture from raw to analytical gold layer.\
•	Applied fraud detection logic with multiple thresholds and anomaly detection.\
•	Ensured data quality and validation for analytics-ready datasets.\
•	Generated quantitative insights for monitoring and reporting.

## 📈 Result Metrics (Sample)

Metric	Value:
Total Transactions	500,000\
Invalid Records	10,094 (2.02%)\
Suspicious Transactions	28,400 (5.68%)\
Normal Transactions	94.32%\
Negative Balance	2.97%\
Extreme Amount	1.53%\
High Amount	1.18%


## 💡 Next Steps / Extensions:
•	Integrate real-time streaming ingestion using Kafka or Spark Streaming.\
•	Add predictive fraud scoring using machine learning models.\
•	Build alerting dashboards with automated notifications for anomalous transactions.
