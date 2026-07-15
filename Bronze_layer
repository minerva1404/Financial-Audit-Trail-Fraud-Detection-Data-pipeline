# 🥉 Bronze Layer

## Overview

The Bronze layer serves as the raw data ingestion stage of the Financial Audit Trail & Fraud Detection pipeline. It is responsible for generating and storing synthetic financial transaction data without applying any transformations, preserving the original records exactly as they are produced.

To simulate a realistic banking environment, the pipeline creates 10,000 customer accounts and 500,000+ financial transactions representing deposits, withdrawals, and transfers distributed across the 2025 calendar year. Alongside legitimate transactions, intentionally corrupted and anomalous records are introduced to emulate the types of data quality issues and suspicious financial activities commonly encountered in real-world financial systems.

By retaining the raw dataset in its original form, the Bronze layer establishes complete data lineage and provides a reliable source for downstream cleansing, validation, fraud detection, and analytical processing performed in the Silver and Gold layers.

---

## 🎯 Objectives

* Generate realistic synthetic customer and financial transaction data at scale.
* Simulate deposits, withdrawals, and transfer transactions with randomized timestamps throughout 2025.
* Preserve all generated records in their original format without modification.
* Introduce controlled data quality issues and fraud scenarios for testing downstream validation logic.
* Maintain complete auditability and data lineage across the Medallion Architecture.
* Provide the foundational dataset for Silver layer cleansing and Gold layer analytics.

---

## 🔄 Bronze Layer Workflow

User Generation
(10,000 Users)
          ↓
Transaction Generation
(500,000+ Transactions)
          ↓
Fraud & Data Quality Injection
(NULL IDs, Negative Values,
High-Value Transactions)
          ↓
Raw Data Storage
(users & raw_transactions)
          ↓
Validation Queries
          ↓
Bronze Layer Complete

---

## 🏦 Synthetic Data Generation

### Purpose

The Synthetic Data Generation component creates a large-scale financial dataset that closely resembles real-world banking activity. Instead of relying on publicly available financial records, the pipeline generates customer profiles and transaction events programmatically, enabling repeatable testing and realistic fraud detection scenarios.

Each transaction is assigned a unique identifier, transaction type, transaction amount, account balance, transaction timestamp, and ingestion timestamp. Randomization techniques are used to produce diverse financial behavior while maintaining realistic distributions of deposits, withdrawals, and transfers.

To support downstream data quality engineering, intentionally corrupted records and suspicious financial patterns are also injected into the dataset, allowing the Silver layer to demonstrate cleansing, validation, and anomaly detection capabilities.

### Key Responsibilities

* Creates the users table containing 10,000 synthetic customer accounts.
* Generates over 500,000 financial transactions using a batch-based stored procedure.
* Simulates three primary transaction types:
    * Deposits
    * Withdrawals
    * Transfers
* Assigns randomized transaction amounts and account balances.
* Generates transaction timestamps spanning the entire 2025 calendar year.
* Records ingestion timestamps occurring after the original transaction time.
* Injects controlled anomalies to emulate real-world data quality issues.
* Stores all raw records without applying validation or transformations.
* Executes validation queries to verify dataset completeness before Silver layer processing.

### ⚠️ Simulated Data Quality & Fraud Scenarios

To create realistic financial data suitable for fraud analytics, the Bronze layer deliberately introduces controlled anomalies into the generated dataset.

The simulated scenarios include:

* Approximately 2% NULL User IDs to represent incomplete or corrupted customer records.
* Negative transaction amounts to simulate invalid financial events.
* Negative account balances representing abnormal account states.
* High-value transaction spikes significantly exceeding normal transaction ranges to emulate potentially fraudulent activities.
* Randomized transaction timing to reflect natural banking activity throughout the year.

These anomalies are intentionally preserved within the Bronze layer, allowing downstream validation rules to identify, cleanse, and classify suspicious records.

--- 

## 📋 Validation & Monitoring

After data generation, several validation queries are executed to verify the integrity and completeness of the Bronze dataset before it progresses to the Silver layer.

The validation process measures:

* Total number of generated transactions.
* Number of records containing NULL User IDs.
* Number of negative transaction amounts.
* Number of negative account balances.
* Number of unusually high-value transactions.

These metrics provide an initial quality assessment of the generated dataset while confirming that the intended anomaly distributions have been successfully injected.

---

## 🗄️ Pipeline Role

The Bronze layer represents the raw ingestion stage of the Financial Audit Trail & Fraud Detection pipeline. Its primary responsibility is to preserve original financial transaction records exactly as they are generated, without applying cleansing, filtering, or business logic.

By maintaining an immutable copy of the source data, the Bronze layer ensures complete traceability, reproducibility, and auditability throughout the pipeline. This design enables downstream layers to perform data quality enforcement and fraud analysis while retaining access to the original records whenever required.

---

## 📄 Source Code

```SQL
CREATE DATABASE IF NOT EXISTS fintech_raw;
USE fintech_raw;

-- =========================
-- USERS TABLE
-- =========================
DROP TABLE IF EXISTS users;

CREATE TABLE users (
    user_id VARCHAR(50) PRIMARY KEY,
    full_name VARCHAR(100),
    email VARCHAR(100),
    created_at TIMESTAMP
);

INSERT INTO users (user_id, full_name, email, created_at)
SELECT
    CONCAT('USER_', LPAD(seq,5,'0')),
    CONCAT('User_', seq),
    CONCAT('user', seq, '@mail.com'),
    NOW() - INTERVAL FLOOR(RAND()*1000) DAY
FROM (
    SELECT @row:=@row+1 AS seq
    FROM information_schema.columns a,
         information_schema.columns b,
         (SELECT @row:=0) r
    LIMIT 10000
) numbers;


-- =========================
-- TRANSACTIONS TABLE
-- =========================
DROP TABLE IF EXISTS raw_transactions;

CREATE TABLE raw_transactions (
    event_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    transaction_id VARCHAR(50),
    user_id VARCHAR(50),
    transaction_type VARCHAR(20),
    amount DECIMAL(12,2),
    balance_after DECIMAL(12,2),
    transaction_timestamp TIMESTAMP,
    ingestion_timestamp TIMESTAMP
);


-- =========================
-- GENERATOR PROCEDURE
-- =========================
SET autocommit = 0;

DROP PROCEDURE IF EXISTS generate_mass_transactions;

DELIMITER //

CREATE PROCEDURE generate_mass_transactions(
    IN total_rows INT,
    IN batch_size INT
)
BEGIN
    DECLARE inserted INT DEFAULT 0;

    WHILE inserted < total_rows DO

        INSERT INTO raw_transactions (
            transaction_id,
            user_id,
            transaction_type,
            amount,
            balance_after,
            transaction_timestamp,
            ingestion_timestamp
        )

        SELECT
            UUID(),

            /* 2% NULL USER IDS (corrupt data simulation) */
            CASE WHEN RAND()<0.02 THEN NULL ELSE user_id END,

            /* realistic transaction types */
            CASE
                WHEN RAND()<0.45 THEN 'deposit'
                WHEN RAND()<0.90 THEN 'withdrawal'
                ELSE 'transfer'
            END,

            /* amount logic with fraud injection */
            CASE
                WHEN RAND()<0.02 THEN ROUND(RAND()*50000,2)     -- extreme fraud spikes
                WHEN RAND()<0.05 THEN -ROUND(RAND()*5000,2)     -- negative anomaly
                ELSE ROUND(RAND()*4000,2)
            END,

            /* balance logic */
            CASE
                WHEN RAND()<0.03 THEN -ROUND(RAND()*2000,2)     -- negative balance anomaly
                ELSE ROUND(RAND()*20000,2)
            END,

            /* transaction timestamp random in 2025 */
            @txn_time :=
                TIMESTAMP('2025-01-01')
                + INTERVAL FLOOR(RAND()*31536000) SECOND,

            /* ingestion always AFTER txn */
            @txn_time + INTERVAL FLOOR(RAND()*7200) SECOND

        FROM users
        ORDER BY RAND()
        LIMIT batch_size;

        SET inserted = inserted + batch_size;

        COMMIT;

    END WHILE;

END //

DELIMITER ;


-- =========================
-- GENERATE DATA
-- =========================
CALL generate_mass_transactions(500000,5000);

SET autocommit = 1;


-- =========================
-- QUICK VALIDATION
-- =========================
SELECT COUNT(*) total_transactions FROM raw_transactions;

SELECT
    SUM(user_id IS NULL) AS null_users,
    SUM(amount<0) AS negative_amounts,
    SUM(balance_after<0) AS negative_balance,
    SUM(amount>4000) AS high_amounts
FROM raw_transactions;
```

---

## 🖥️ Console Output

<img width="1920" height="1080" alt="Screenshot (54)" src="https://github.com/user-attachments/assets/f4512af4-a569-4cb3-b309-078c39600ed1" />

<img width="1920" height="1080" alt="Screenshot (55)" src="https://github.com/user-attachments/assets/55ece03a-ed03-444a-bc0f-37bdbf680578" />

<img width="1920" height="1080" alt="Screenshot (56)" src="https://github.com/user-attachments/assets/211b058f-5bfd-4f96-8015-3f0889e928e2" />


---

### Users Table

Each user record contains:

* User ID
* Full Name
* Email Address
* Account Creation Timestamp

### Raw Transactions Table

Each transaction record contains:

* Event ID
* Transaction ID
* User ID
* Transaction Type
* Transaction Amount
* Account Balance After Transaction
* Transaction Timestamp
* Ingestion Timestamp

These raw datasets serve as the input for the Silver layer, where validation, cleansing, standardization, and fraud classification are performed before generating business-ready analytical datasets.

---

## ✅ Bronze Layer Summary

The Bronze layer establishes the foundation of the Financial Audit Trail & Fraud Detection pipeline by generating and preserving over 500,000 synthetic financial transactions across 10,000 customer accounts. Through realistic transaction simulation and deliberate anomaly injection, it creates a comprehensive raw dataset that supports data quality engineering, fraud detection, and audit analytics. By maintaining immutable source records with complete lineage, the Bronze layer provides a trusted starting point for the Silver layer’s validation processes and the Gold layer’s business intelligence reporting.
