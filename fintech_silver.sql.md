## Summary:
•	Transaction Data Cleaning & Normalization: Processed 500K+ raw financial transactions, normalizing amounts, standardizing transaction types, and handling missing or negative values for reliable downstream analytics.\
•	Status & Fraud Classification: Implemented status flags (INVALID_USER, INVALID_AMOUNT, etc.) and fraud flags (NEGATIVE_BALANCE, HIGH_AMOUNT, EXTREME_AMOUNT, ANOMALOUS_SPIKE) to identify suspicious or anomalous transactions.\
•	Data Quality Insights: Generated data_quality_issue indicators highlighting missing users, negative amounts/balances, and ingestion anomalies for proactive monitoring.\
•	Quantified Metrics: Produced validated counts, invalid transaction counts, suspicious transaction counts, and fraud percentage (~5–6%), enabling KPI reporting and dashboard-ready analytics.

## Query:
```SQL
CREATE DATABASE IF NOT EXISTS fintech_silver;
USE fintech_silver;

DROP TABLE IF EXISTS clean_transactions;

CREATE TABLE clean_transactions AS
WITH avg_calc AS (
    SELECT AVG(amount) AS avg_amount
    FROM fintech_raw.raw_transactions
)

SELECT
    r.transaction_id,
    r.user_id,
    LOWER(r.transaction_type) AS transaction_type,

    /* normalized amount */
    ABS(r.amount) AS amount,

    r.balance_after,
    r.transaction_timestamp,
    r.ingestion_timestamp,

    /* ---------------- STATUS FLAG ---------------- */
    CASE
        WHEN r.user_id IS NULL THEN 'INVALID_USER'
        WHEN r.transaction_timestamp IS NULL THEN 'INVALID_TIME'
        WHEN r.amount IS NULL THEN 'INVALID_AMOUNT'
        WHEN r.ingestion_timestamp < r.transaction_timestamp THEN 'INVALID_INGESTION'
        ELSE 'VALID'
    END AS status_flag,

    /* ---------------- FRAUD FLAG ---------------- */
    CASE
        WHEN r.balance_after < 0 THEN 'NEGATIVE_BALANCE'
        WHEN ABS(r.amount) > 10000 THEN 'EXTREME_AMOUNT'
        WHEN ABS(r.amount) > 4000 THEN 'HIGH_AMOUNT'
        WHEN ABS(r.amount) > a.avg_amount * 3 THEN 'ANOMALOUS_SPIKE'
        ELSE 'NORMAL'
    END AS fraud_flag,

    /* ---------------- DATA QUALITY ---------------- */
    CONCAT_WS('; ',
        IF(r.user_id IS NULL,'Missing User','OK User'),
        IF(r.amount < 0,'Negative Amount','OK Amount'),
        IF(r.balance_after < 0,'Negative Balance','OK Balance'),
        IF(r.ingestion_timestamp < r.transaction_timestamp,'Ingestion Earlier Than Txn','OK Ingestion')
    ) AS data_quality_issue

FROM fintech_raw.raw_transactions r
CROSS JOIN avg_calc a;

/* ================= VALIDATION ================= */

SELECT COUNT(*) AS raw_count FROM fintech_raw.raw_transactions;

SELECT COUNT(*) AS silver_count FROM clean_transactions;

SELECT status_flag, COUNT(*) 
FROM clean_transactions
GROUP BY status_flag
ORDER BY COUNT(*) DESC;

SELECT fraud_flag, COUNT(*) 
FROM clean_transactions
GROUP BY fraud_flag
ORDER BY COUNT(*) DESC;

SELECT COUNT(*) AS records_with_issues
FROM clean_transactions
WHERE status_flag != 'VALID' OR fraud_flag != 'NORMAL';

SELECT 
    COUNT(*) AS total,
    SUM(CASE WHEN status_flag != 'VALID' THEN 1 ELSE 0 END) AS invalid_records,
    SUM(CASE WHEN fraud_flag != 'NORMAL' THEN 1 ELSE 0 END) AS suspicious_records,
    ROUND(100 * SUM(CASE WHEN fraud_flag != 'NORMAL' THEN 1 ELSE 0 END)/COUNT(*),2) AS fraud_percent
FROM clean_transactions;
```
