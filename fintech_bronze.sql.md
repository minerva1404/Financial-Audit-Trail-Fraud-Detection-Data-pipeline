## Summary:
•	Synthetic User & Transaction Data Generation: Created 10,000 users and 500,000+ transactions simulating deposits, withdrawals, transfers with realistic timestamps across 2025.\
•	Data Quality & Fraud Simulation: Injected 2% NULL user IDs, negative amounts, negative balances, and 2% extreme spikes to mimic anomalous and corrupt financial transactions.\
•	Medallion Pipeline Ready: Structured raw data in users and transactions tables to feed Bronze → Silver → Gold ETL layers for downstream analytics.\
•	Validation & Monitoring: Quick aggregation queries to verify null users, negative amounts, negative balances, and high-value transactions, ensuring data integrity before processing.


## Query:
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
