# Retail Sales Data Analysis Project

## Project Overview

This project performs a comprehensive SQL-based analysis of a retail sales dataset. The goal is to explore trends, performance metrics, and consumer behavior through cleaning, transformation, and insightful querying. Key insights include top-selling categories, customer demographics, peak sales periods, and revenue trends.

## Objectives

1. **Set up a retail sales database**: Create and populate a retail sales database with the provided sales data.
2. **Data Cleaning**: Identify and remove any records with missing or null values.
3. **Exploratory Data Analysis (EDA)**: Perform basic exploratory data analysis to understand the dataset.
4. **Business Analysis**: Use SQL to answer specific business questions and derive insights from the sales data.

---

## Project Structure

---
## SQL Code with Necessary Comments

```sql
-- =============================================================
-- SQL Project: Retail Sales Analysis - Phase 1
-- Cleaned, Formatted, and Commented SQL Code
-- =============================================================

-- ============================
-- 1. SETUP AND TABLE CREATION
-- ============================

-- Drop the table if it already exists to avoid conflicts
DROP TABLE IF EXISTS retail_sales;

-- Create the main retail sales table
CREATE TABLE retail_sales (
    transactions_id INT PRIMARY KEY,       -- Unique identifier for each transaction
    sale_date DATE,                        -- Date when the sale occurred
    sale_time TIME,                        -- Time when the sale occurred
    customer_id INT,                       -- ID of the customer who made the purchase
    gender VARCHAR(15),                    -- Gender of the customer
    age INT,                               -- Age of the customer
    category VARCHAR(15),                  -- Product category
    quantity INT,                          -- Number of items sold
    price_per_unit FLOAT,                  -- Unit price of the product
    cogs FLOAT,                            -- Cost of Goods Sold
    total_sale FLOAT                       -- Total amount of the sale
);

-- ============================
-- 2. IMPORT DATA
-- ============================

-- Select the appropriate database
USE retail_sales_project;
GO

-- Import data from CSV file
BULK INSERT retail_sales
FROM 'B:\\SQL Datasets\\Retail-Sales-Analysis-SQL-Project--P1-main\\SQL - Retail Sales Analysis_utf .csv'
WITH (
    FIRSTROW = 2,
    FIELDTERMINATOR = ',',
    ROWTERMINATOR = '\n',
    TABLOCK
);
GO

-- ============================
-- 3. DATA CLEANING
-- ============================

-- View sample data
SELECT TOP 10 * FROM retail_sales;

-- Total rows in the dataset
SELECT COUNT(*) AS total_rows FROM retail_sales;

-- Check for NULL values in any column
SELECT COUNT(*) AS rows_with_nulls
FROM retail_sales
WHERE transactions_id IS NULL OR sale_date IS NULL OR sale_time IS NULL OR
      customer_id IS NULL OR gender IS NULL OR age IS NULL OR
      category IS NULL OR quantity IS NULL OR price_per_unit IS NULL OR
      cogs IS NULL OR total_sale IS NULL;

-- Delete records with NULL values
DELETE FROM retail_sales
WHERE transactions_id IS NULL OR sale_date IS NULL OR sale_time IS NULL OR
      customer_id IS NULL OR gender IS NULL OR age IS NULL OR
      category IS NULL OR quantity IS NULL OR price_per_unit IS NULL OR
      cogs IS NULL OR total_sale IS NULL;

-- ============================
-- 4. BASIC EXPLORATION
-- ============================

-- Number of sales records
SELECT COUNT(*) AS total_sales FROM retail_sales;

-- Total customer entries and unique customers
SELECT COUNT(customer_id) AS total_customer_entries FROM retail_sales;
SELECT COUNT(DISTINCT customer_id) AS unique_customers FROM retail_sales;

-- Product categories
SELECT DISTINCT category FROM retail_sales;

-- ============================
-- 5. DATE-BASED ANALYSIS
-- ============================

-- Sales on a specific date
SELECT * FROM retail_sales WHERE sale_date = '2022-11-05';

-- Clothing sales in November 2022 (quantity > 2)
SELECT *
FROM retail_sales
WHERE category = 'Clothing'
  AND quantity > 2
  AND sale_date BETWEEN '2022-11-01' AND '2022-11-30';

-- ============================
-- 6. CATEGORY-BASED ANALYSIS
-- ============================

-- Total quantity sold by category
SELECT category, SUM(quantity) AS total_quantity
FROM retail_sales
GROUP BY category;

-- Total sales by category
SELECT category, SUM(total_sale) AS net_sale
FROM retail_sales
GROUP BY category;

-- Transactions and total sales per category
SELECT category, COUNT(*) AS total_transactions, SUM(total_sale) AS net_sale
FROM retail_sales
GROUP BY category
ORDER BY net_sale DESC;

-- Average customer age per category and by gender
SELECT category, AVG(age) AS average_age
FROM retail_sales
WHERE age IS NOT NULL
GROUP BY category
ORDER BY average_age DESC;

SELECT category, gender, AVG(age) AS average_age
FROM retail_sales
WHERE age IS NOT NULL
GROUP BY category, gender
ORDER BY category, gender;

-- ============================
-- 7. HIGH VALUE SALES ANALYSIS
-- ============================

-- Transactions with total_sale > 1000
SELECT * FROM retail_sales WHERE total_sale > 1000;

-- Count and average age of high-value transactions
SELECT COUNT(*) AS high_value_sales_count FROM retail_sales WHERE total_sale > 1000;
SELECT AVG(age) AS avg_customer_age FROM retail_sales WHERE total_sale > 1000 AND age IS NOT NULL;

-- High-value sales by category and gender
SELECT category, COUNT(*) AS transaction_count, SUM(total_sale) AS total_revenue
FROM retail_sales
WHERE total_sale > 1000
GROUP BY category
ORDER BY total_revenue DESC;

SELECT gender, COUNT(*) AS transaction_count, SUM(total_sale) AS total_revenue
FROM retail_sales
WHERE total_sale > 1000
GROUP BY gender
ORDER BY total_revenue DESC;

-- ============================
-- 8. ADVANCED AGGREGATIONS
-- ============================

-- Transactions by gender and category
SELECT gender, category, COUNT(transactions_id) AS total_transactions
FROM retail_sales
WHERE transactions_id IS NOT NULL
GROUP BY gender, category
ORDER BY gender, total_transactions DESC;

-- Average monthly sales and best-performing month
WITH MonthlySales AS (
    SELECT YEAR(sale_date) AS sale_year, MONTH(sale_date) AS sale_month, AVG(total_sale) AS avg_monthly_sale
    FROM retail_sales
    WHERE total_sale IS NOT NULL
    GROUP BY YEAR(sale_date), MONTH(sale_date)
),
RankedSales AS (
    SELECT *, RANK() OVER (PARTITION BY sale_year ORDER BY avg_monthly_sale DESC) AS monthly_rank
    FROM MonthlySales
)
SELECT sale_year, sale_month, avg_monthly_sale
FROM RankedSales
WHERE monthly_rank = 1
ORDER BY sale_year;

-- Top 5 customers by total spending
SELECT TOP 5 customer_id, SUM(total_sale) AS total_sales
FROM retail_sales
GROUP BY customer_id
ORDER BY total_sales DESC;

-- Unique customers per category
SELECT category, COUNT(DISTINCT customer_id) AS unique_customers
FROM retail_sales
GROUP BY category
ORDER BY unique_customers DESC;

-- Orders per shift (morning/afternoon/evening)
SELECT
    CASE 
        WHEN DATEPART(HOUR, sale_time) < 12 THEN 'Morning'
        WHEN DATEPART(HOUR, sale_time) BETWEEN 12 AND 17 THEN 'Afternoon'
        ELSE 'Evening'
    END AS shift,
    COUNT(*) AS number_of_orders
FROM retail_sales
GROUP BY
    CASE 
        WHEN DATEPART(HOUR, sale_time) < 12 THEN 'Morning'
        WHEN DATEPART(HOUR, sale_time) BETWEEN 12 AND 17 THEN 'Afternoon'
        ELSE 'Evening'
    END
ORDER BY shift;

-- ============================
-- 9. BUSINESS METRICS
-- ============================

-- Monthly revenue trend
SELECT FORMAT(sale_date, 'yyyy-MM') AS month, SUM(total_sale) AS monthly_revenue
FROM retail_sales
GROUP BY FORMAT(sale_date, 'yyyy-MM')
ORDER BY month;

-- Most popular category by quantity sold
SELECT category, SUM(quantity) AS total_quantity
FROM retail_sales
GROUP BY category
ORDER BY total_quantity DESC;

-- Customer lifetime value (CLV)
SELECT customer_id, SUM(total_sale) AS lifetime_value
FROM retail_sales
GROUP BY customer_id
ORDER BY lifetime_value DESC;

-- Sales distribution by gender
SELECT gender, COUNT(*) AS total_transactions, SUM(total_sale) AS total_revenue
FROM retail_sales
GROUP BY gender;

-- Average sale value by category
SELECT category, AVG(total_sale) AS average_sale
FROM retail_sales
GROUP BY category;

-- Repeat customers (customers with more than 1 transaction)
SELECT COUNT(DISTINCT customer_id) AS repeat_customers
FROM (
    SELECT customer_id
    FROM retail_sales
    GROUP BY customer_id
    HAVING COUNT(*) > 1
) AS sub;

-- Hourly sales analysis
SELECT DATEPART(HOUR, sale_time) AS hour, COUNT(*) AS transactions, SUM(total_sale) AS total_sales
FROM retail_sales
GROUP BY DATEPART(HOUR, sale_time)
ORDER BY hour;

-- Average spend by customer age group
SELECT 
    CASE 
        WHEN age < 20 THEN 'Under 20'
        WHEN age BETWEEN 20 AND 29 THEN '20s'
        WHEN age BETWEEN 30 AND 39 THEN '30s'
        WHEN age BETWEEN 40 AND 49 THEN '40s'
        ELSE '50+'
    END AS age_group,
    AVG(total_sale) AS avg_spend
FROM retail_sales
GROUP BY 
    CASE 
        WHEN age < 20 THEN 'Under 20'
        WHEN age BETWEEN 20 AND 29 THEN '20s'
        WHEN age BETWEEN 30 AND 39 THEN '30s'
        WHEN age BETWEEN 40 AND 49 THEN '40s'
        ELSE '50+'
    END;

-- Profitability analysis (COGS vs Revenue)
SELECT SUM(cogs) AS total_cost, SUM(total_sale) AS total_revenue,
       (SUM(total_sale) - SUM(cogs)) AS profit,
       ROUND((SUM(total_sale) - SUM(cogs)) * 100.0 / SUM(cogs), 2) AS profit_margin_pct
FROM retail_sales;

-- High-value transactions (above average sale)
SELECT *
FROM retail_sales
WHERE total_sale > (SELECT AVG(total_sale) FROM retail_sales);
```

---

## Author - Okoth Onyango
