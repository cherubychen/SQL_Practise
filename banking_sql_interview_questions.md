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

"Write a SQL query to analyze the performance of different account types. For each account type, show the number of transactions, the average transaction amount, and the total transaction amount. This will help us understand which account types generate the most activity and value."

**Skills Tested:**
- GROUP BY
- Aggregation functions (COUNT, AVG, SUM)
- JOINs
- ROUND function

---

## Question 2: Multi-Table Joins with Customer Analysis

**Interview Question:**

"Our marketing team needs to identify high-value customers for a targeted campaign. Write a SQL query that shows each customer's name, the number of accounts they have, the number of transactions they've made, and their net transaction amount. Order the results by net transaction amount in descending order to identify our most valuable customers."

**Skills Tested:**
- Multiple table JOINs
- COUNT DISTINCT
- String functions (CONCAT)
- GROUP BY with multiple columns
- ORDER BY

---

## Question 3: Window Functions for Running Totals

**Interview Question:**

"For account auditing purposes, we need to track the running balance of account #101 over time. Write a SQL query that shows the account details, transaction date, transaction amount, balance after each transaction, and a running total of all transactions up to that point. This will help us verify that our balance calculations are correct."

**Skills Tested:**
- Window functions
- PARTITION BY
- ORDER BY within window functions
- ROWS BETWEEN frame specification
- Multiple JOINs

---

## Question 4: Ranking Data with Window Functions

**Interview Question:**

"Our customer relationship team wants to identify our top customers based on their total transaction amounts. Write a SQL query that ranks all customers based on their total transaction amount in descending order. Include the customer's name and their total transaction amount in the results."

**Skills Tested:**
- RANK() window function
- Aggregation within window functions
- GROUP BY
- Multiple table JOINs

---

## Question 5: Conditional Aggregation with CASE Statements

**Interview Question:**

"The finance department needs a monthly report of transaction activity by account type. Write a SQL query that shows, for each year, month, and account type: the number of transactions, total deposits(positive amounts), total withdrawals(negative amounts), and the net change. This will help them track cash flow trends over time."

**Skills Tested:**
- CASE statements within aggregations
- Multiple JOINs
- GROUP BY with multiple columns
- Conditional logic in SQL

---

## Question 6: Moving Averages with Window Functions

**Interview Question:**

"To smooth out daily fluctuations in account balances for trend analysis, write a SQL query that calculates a 3-day moving average of closing balances for account #101. Include the account ID, account type, date, actual closing balance, and the 3-day moving average. This will help our analysts identify underlying trends in customer balances."

**Skills Tested:**
- Window functions for moving averages
- ROWS BETWEEN for defining the window frame
- AVG() function with window specification
- Multiple JOINs

---

## Question 7: Branch Performance Analysis

**Interview Question:**

"Management needs a performance report for all branches. Write a SQL query that shows each branch's ID, name, city, number of transactions, total deposits (positive amounts), total withdrawals (absolute value of negative amounts), and net change. Order the results by name of transactions in descending order to identify our busiest branches."

**Skills Tested:**
- Conditional aggregation
- CASE statements
- ABS() function
- GROUP BY
- ORDER BY

---

## Question 8: Slowly Changing Dimension (Type 2)

**Interview Question:**

"Our compliance department requires us to maintain a complete history of customer address changes for audit purposes. Write a SQL query that retrieves the full address history for customer #1, showing when each address was active. Include the customer's name, full address details, effective date, and end date (showing 'Present' for the current address). This will help us demonstrate our compliance with record-keeping regulations."