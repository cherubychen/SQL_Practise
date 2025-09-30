# Banking Data Warehouse SQL Interview Questions

These questions are based on a banking data warehouse with the following star schema:

## Dimension Tables

- `dim_customer` (customer_id, first_name, last_name, email, phone, address, city, state, zip_code, date_joined)
- `dim_account` (account_id, customer_id, account_type, open_date, status)
- `dim_branch` (branch_id, branch_name, address, city, state, zip_code)
- `dim_date` (date_id, date, day, month, year, quarter, is_weekend)

## Fact Tables

- `fact_transactions` (transaction_id, account_id, date_id, branch_id, transaction_type, amount, balance_after)
- `fact_daily_balance` (balance_id, account_id, date_id, closing_balance)

---

## Question 1: Basic Aggregation and GROUP BY

**Interview Question:**

"Write a SQL query to analyze the performance of different account types. For each account type, show the number of transactions, the average transaction amount, and the total transaction amount."

**Skills Tested:**
- GROUP BY
- Aggregation functions (COUNT, AVG, SUM)
- JOINs
- ROUND function

**Solution:**

```sql
SELECT 
    account_type,
    COUNT(*) AS transaction_number,
    ROUND(AVG(ABS(amount)), 2) AS avg_transaction_amount,
    SUM(ABS(amount)) AS total_transaction_amount
FROM fact_transactions t 
JOIN dim_account a ON t.account_id = a.account_id
GROUP BY account_type;
```

---

## Question 2: Multi-Table Joins with Customer Analysis

**Interview Question:**

"Our marketing team needs to identify high-value customers for a targeted campaign. Write a SQL query that shows each customer's name, the number of accounts they have, the number of transactions they've made, and their total transaction amount."

**Skills Tested:**
- Multiple table JOINs
- COUNT DISTINCT
- String functions (CONCAT)
- GROUP BY with multiple columns
- ORDER BY

**Solution:**

```sql
SELECT 
    c.customer_id,
    CONCAT(first_name, ' ', last_name) AS customer_name,
    COUNT(*) AS transaction_number,
    COUNT(DISTINCT a.account_id) AS account_number,
    SUM(amount) AS net_tr_amount
FROM dim_account a 
JOIN dim_customer c ON a.customer_id = c.customer_id 
JOIN fact_transactions t ON a.account_id = t.account_id
GROUP BY c.customer_id, customer_name
ORDER BY net_tr_amount DESC;
```

---

## Question 3: Window Functions for Running Totals

**Interview Question:**

"For account auditing purposes, we need to track the running balance of account #101 over time. Write a SQL query that shows the account details, transaction date, transaction amount, balance after each transaction, and a running total."

**Skills Tested:**
- Window functions
- PARTITION BY
- ORDER BY within window functions
- ROWS BETWEEN frame specification
- Multiple JOINs

**Solution:**

```sql
SELECT 
    account_id, 
    date, 
    amount, 
    balance_after,
    SUM(amount) OVER (
        PARTITION BY account_id 
        ORDER BY date 
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total_amount
FROM fact_transactions t
JOIN dim_date d ON t.date_id = d.date_id 
WHERE account_id = 101;
```

---

## Question 4: Ranking Data with Window Functions

**Interview Question:**

"Our customer relationship team wants to identify our top customers based on their total transaction amounts. Write a SQL query that ranks all customers based on their total transaction amount in descending order."

**Skills Tested:**
- RANK() window function
- Aggregation within window functions
- GROUP BY
- Multiple table JOINs

**Solution - Method 1 (Using CTE):**

```sql
WITH temp_amount_total AS (
    SELECT 
        c.customer_id,
        CONCAT(first_name, ' ', last_name) AS customer_name,
        SUM(amount) AS amount_total
    FROM dim_customer c 
    JOIN dim_account a ON c.customer_id = a.customer_id 
    JOIN fact_transactions t ON a.account_id = t.account_id
    GROUP BY customer_id, customer_name
    ORDER BY amount_total DESC
)
SELECT 
    customer_name, 
    amount_total, 
    RANK() OVER (ORDER BY amount_total DESC) AS amount_rank 
FROM temp_amount_total;
```

**Solution - Method 2 (Better solution - Window function with GROUP BY):**

Note: Window functions like RANK() can be used with GROUP BY. You don't need to use PARTITION in the window function, just using ORDER BY with aggregation functions.

```sql
-- Window function for ranking
SELECT
    c.customer_id,
    CONCAT(c.first_name, ' ', c.last_name) AS customer_name,  -- MySQL uses CONCAT function
    ROUND(SUM(t.amount), 2) AS total_amount,
    RANK() OVER (ORDER BY SUM(t.amount) DESC) AS rank_by_amount
FROM dim_customer c
JOIN dim_account a ON c.customer_id = a.customer_id
JOIN fact_transactions t ON a.account_id = t.account_id
GROUP BY c.customer_id, customer_name;
```

---

## Question 5: Conditional Aggregation with CASE Statements

**Interview Question:**

"The finance department needs a monthly report of transaction activity by account type. Write a SQL query that shows, for each year, month, and account type: the number of transactions, total deposits, total withdrawals, and net change."

**Skills Tested:**
- CASE statements within aggregations
- Multiple JOINs
- GROUP BY with multiple columns
- Conditional logic in SQL

**Solution:**

**Important Note:** CASE WHEN clause should be used with an aggregation function.

The following will have an error:
```sql
CASE WHEN amount > 0 THEN SUM(amount) END
```

**Correct Solution:**

```sql
SELECT 
    year, 
    month, 
    account_type, 
    COUNT(*) AS tr_number, 
    SUM(amount) AS net_change,
    SUM(CASE WHEN amount > 0 THEN amount ELSE 0 END) AS total_deposit,
    SUM(CASE WHEN amount < 0 THEN ABS(amount) ELSE 0 END) AS total_withdraw
FROM fact_transactions t 
JOIN dim_date d ON d.date_id = t.date_id 
JOIN dim_account a ON t.account_id = a.account_id
GROUP BY year, month, account_type;
```

---

## Question 6: Moving Averages with Window Functions

**Interview Question:**

"To smooth out daily fluctuations in account balances for trend analysis, write a SQL query that calculates a 3-day moving average of closing balances for account #101. Include the account ID, account type, date, closing balance, and the 3-day moving average."

**Skills Tested:**
- Window functions for moving averages
- ROWS BETWEEN for defining the window frame
- AVG() function with window specification
- Multiple JOINs

**Solution:**

```sql
SELECT 
    a.account_id,
    account_type, 
    d.date, 
    closing_balance,
    ROUND(AVG(closing_balance) OVER (
        PARTITION BY account_id 
        ORDER BY date
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ), 2) AS 3_day_avg_close_bl
FROM fact_daily_balance b
JOIN dim_account a ON b.account_id = a.account_id
JOIN dim_date d ON b.date_id = d.date_id
WHERE a.account_id = '101';
```

---

## Question 7: Branch Performance Analysis

**Interview Question:**

"Management needs a performance report for all branches. Write a SQL query that shows each branch's ID, name, city, number of transactions, total deposits (positive amounts), total withdrawals (absolute values), and net amount."

**Skills Tested:**
- Conditional aggregation
- CASE statements
- ABS() function
- GROUP BY
- ORDER BY

**Solution:**

```sql
SELECT
    b.branch_id,
    branch_name, 
    city,
    COUNT(*) AS tr_number,
    SUM(CASE WHEN amount > 0 THEN amount ELSE 0 END) AS deposit_amount,
    SUM(CASE WHEN amount < 0 THEN ABS(amount) ELSE 0 END) AS withdraw_amount,
    SUM(amount) AS net_amount
FROM dim_branch b
JOIN fact_transactions t ON b.branch_id = t.branch_id
GROUP BY b.branch_id, branch_name, city
ORDER BY tr_number DESC;
```

---

## Question 8: Slowly Changing Dimension (Type 2)

**Interview Question:**

"Our compliance department requires us to maintain a complete history of customer address changes for audit purposes. Write a SQL query that retrieves the full address history for customer #1, showing..."

**Skills Tested:**
- Slowly Changing Dimension (Type 2) concepts
- Handling NULL values with COALESCE
- Date filtering and comparison
- Temporal data querying
- ORDER BY for chronological display

---

## Bonus Challenge Question

**Interview Question:**

"Our risk department wants to identify potentially fraudulent activity. Write a SQL query that finds all accounts with more than 3 transactions in a single day, where at least one transaction is over $1000."

**Skills Tested:**
- HAVING clause
- Subqueries or complex JOINs
- Conditional filtering
- Aggregation functions
- Multiple conditions

**Solution:**

```sql
SELECT
    a.account_id,
    date_id,
    COUNT(*) AS tr_num,
    SUM(amount) AS total_tr_amount,
    SUM(CASE WHEN ABS(amount) > 1000 THEN 1 ELSE 0 END) AS more_than_2k_count
FROM fact_transactions t
JOIN dim_account a ON t.account_id = a.account_id
GROUP BY a.account_id, date_id
HAVING tr_num > 1 AND more_than_2k_count >= 1;
```

**Alternative Solution using CTE:**

```sql
WITH temp_tr_view AS (
    SELECT 
        a.account_id, 
        date_id, 
        COUNT(*) AS tr_num, 
        SUM(amount) AS total_tr_amount,
        SUM(CASE WHEN ABS(amount) > 1000 THEN 1 ELSE 0 END) AS more_than_2k_count 
    FROM fact_transactions t 
    JOIN dim_account a ON t.account_id = a.account_id 
    GROUP BY a.account_id, date_id
    HAVING tr_num > 1 AND more_than_2k_count >= 1
)
SELECT 
    a.account_id, 
    account_type, 
    date, 
    tr_num, 
    total_tr_amount
FROM temp_tr_view AS v
JOIN dim_account a ON v.account_id = a.account_id
JOIN dim_date d ON v.date_id = d.date_id;
```
