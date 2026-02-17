## Summary:
•	User-Level Transaction Analytics: Built user_transaction_summary aggregating transaction counts, total deposits/withdrawals, volume, average/max/min amounts, and fraud/invalid percentages per user (~10K users).\
•	Fraud Distribution Insights: Created fraud_summary table capturing fraud types, total cases, percent of total, average transaction amounts, and maximum transaction amounts, enabling clear risk visualization.\
•	Daily KPI Aggregation: Developed daily_transaction_summary providing daily total transactions, total volume, average transaction, suspicious and invalid transaction percentages, and daily min/max transactions, ready for dashboard visualization.\
•	Quantified Operational Metrics: Derived suspicious transaction rate (~5.68%), invalid transaction rate (~2.02%), and per-category transaction patterns, enabling end-to-end monitoring of transaction health and operational intelligence.

## Code:
```SQL
CREATE DATABASE IF NOT EXISTS fintech_gold;
USE fintech_gold;

/* =====================================================
   DROP OLD TABLES
===================================================== */
DROP TABLE IF EXISTS user_transaction_summary;
DROP TABLE IF EXISTS fraud_summary;
DROP TABLE IF EXISTS daily_transaction_summary;

/* =====================================================
   USER LEVEL ANALYTICS MART
   - Summary per user for transactions, volume, frauds, and invalids
===================================================== */
CREATE TABLE user_transaction_summary AS
SELECT
    user_id,

    COUNT(*) AS total_transactions,

    SUM(CASE WHEN transaction_type='deposit' THEN amount ELSE 0 END) AS total_deposits,
    SUM(CASE WHEN transaction_type='withdrawal' THEN amount ELSE 0 END) AS total_withdrawals,

    SUM(amount) AS total_volume,
    AVG(amount) AS avg_transaction_amount,
    MAX(amount) AS max_transaction,
    MIN(amount) AS min_transaction,

    SUM(CASE WHEN fraud_flag!='NORMAL' THEN 1 ELSE 0 END) AS suspicious_txn_count,
    ROUND(100*SUM(CASE WHEN fraud_flag!='NORMAL' THEN 1 ELSE 0 END)/COUNT(*),2) AS suspicious_txn_percent,

    SUM(CASE WHEN status_flag!='VALID' THEN 1 ELSE 0 END) AS invalid_txn_count,
    ROUND(100*SUM(CASE WHEN status_flag!='VALID' THEN 1 ELSE 0 END)/COUNT(*),2) AS invalid_txn_percent,

    MAX(transaction_timestamp) AS last_transaction_time

FROM fintech_silver.clean_transactions
GROUP BY user_id;

/* =====================================================
   FRAUD DISTRIBUTION MART
   - Gives fraud categories, count, % of total, average, and max
===================================================== */
CREATE TABLE fraud_summary AS
SELECT
    fraud_flag,
    COUNT(*) AS total_cases,
    ROUND(100*COUNT(*)/(SELECT COUNT(*) FROM fintech_silver.clean_transactions),2) AS percent_of_total,
    ROUND(AVG(amount),2) AS avg_amount,
    MAX(amount) AS highest_amount
FROM fintech_silver.clean_transactions
GROUP BY fraud_flag
ORDER BY total_cases DESC;

/* =====================================================
   DAILY KPI MART
   - Daily transaction summary for dashboards
===================================================== */
CREATE TABLE daily_transaction_summary AS
SELECT
    DATE(transaction_timestamp) AS txn_date,

    COUNT(*) AS total_transactions,
    SUM(amount) AS total_volume,
    ROUND(AVG(amount),2) AS avg_amount,

    SUM(CASE WHEN fraud_flag!='NORMAL' THEN 1 ELSE 0 END) AS suspicious_count,
    ROUND(100*SUM(CASE WHEN fraud_flag!='NORMAL' THEN 1 ELSE 0 END)/COUNT(*),2) AS suspicious_percent,

    SUM(CASE WHEN status_flag!='VALID' THEN 1 ELSE 0 END) AS invalid_count,
    ROUND(100*SUM(CASE WHEN status_flag!='VALID' THEN 1 ELSE 0 END)/COUNT(*),2) AS invalid_percent,

    MAX(amount) AS highest_transaction,
    MIN(amount) AS lowest_transaction

FROM fintech_silver.clean_transactions
GROUP BY DATE(transaction_timestamp)
ORDER BY txn_date;

/* =====================================================
   VALIDATION CHECKS
===================================================== */
SELECT * FROM user_transaction_summary;
SELECT * FROM fraud_summary;
SELECT * FROM daily_transaction_summary;
```
