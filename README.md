# Pareto_Analysis_of_Pet_Shop_Sales
Identify top customers driving pet shop revenue using Pareto Analysis

## Project Overview
This project demonstrates a Pareto Analysis on pet shop sales data to identify the small percentage of customers generating the majority of revenue. Using SQL in BigQuery, the analysis helps answer key business questions and provides actionable insights for customer prioritization.
The analysis addresses questions such as:
- Which customers generate 80% (or any target %) of total sales?
- How many customers should the business focus on to maximize revenue impact?
- How is revenue distributed among all customers?

## Tools & Technologies
- SQL (BigQuery) – for data processing, aggregation, and analysis
- Analytical concepts – Pareto principle (80/20 rule), cumulative revenue, customer segmentation

## Business Problem
In retail, a small group of customers often contributes the most to overall revenue. Identifying these high-value customers enables:
Targeted marketing and loyalty programs
Personalized offers to increase retention
Efficient allocation of sales resources
Using Pareto Analysis, businesses can focus on what matters most-whether it’s products or customers-ensuring resources are allocated for maximum impact.

## Approach & Solution
Two approaches were implemented using SQL:
1️⃣ Step-by-Step Views

```sql
 -- View 1: Calculate revenue per transaction

CREATE OR REPLACE VIEW `pareto-4.paretodemo.sales_v1` AS
SELECT
  CustomerID,
  (Quantity * UnitPrice) AS sales
FROM `pareto-4.paretodemo.pet_shop_sales`;

-- View 2: Aggregate revenue per customer

CREATE OR REPLACE VIEW `pareto-4.paretodemo.sales_v2` AS
SELECT
  CustomerID,
  SUM(sales) AS customer_revenue
FROM `pareto-4.paretodemo.sales_v1`
GROUP BY CustomerID;

-- View 3: Rank customers and calculate cumulative revenue
CREATE OR REPLACE VIEW `pareto-4.paretodemo.sales_v3` AS
SELECT
  CustomerID,
  customer_revenue,
  ROW_NUMBER() OVER(ORDER BY customer_revenue DESC) AS cum_customers,
  COUNT(*) OVER() AS total_customers,
  SUM(customer_revenue) OVER(ORDER BY customer_revenue DESC
      ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS cum_revenue,
  SUM(customer_revenue) OVER() AS total_revenue
FROM `pareto-4.paretodemo.sales_v2`;

-- View 4: Calculate cumulative sales share and customer percentage
CREATE OR REPLACE VIEW `pareto-4.paretodemo.sales_v4` AS
SELECT
  CustomerID,
  customer_revenue,
  cum_customers,
  total_customers,
  cum_revenue,
  total_revenue,
  cum_revenue / total_revenue AS cum_sales_share,
  cum_customers / total_customers AS cum_pct_customers
FROM `pareto-4.paretodemo.sales_v3`;

-- Final query: Identify top customers reaching target revenue
DECLARE target_sales_pct FLOAT64 DEFAULT 0.80;

SELECT
  MIN(cum_customers) AS number_of_customers,
  MIN(total_customers) AS total_customers,
  MIN(cum_revenue) AS cum_revenue,
  MIN(total_revenue) AS total_revenue,
  target_sales_pct * 100 AS target_sales_percent,
  MIN(total_revenue * target_sales_pct) AS target_sales,
  MIN(cum_sales_share) AS cum_sales_share,
  MIN(cum_pct_customers) AS cum_pct_customers
FROM `pareto-4.paretodemo.sales_v4`
WHERE cum_sales_share >= target_sales_pct;
```
```markdown
2️⃣ Using CTEs
This version uses Common Table Expressions (CTEs) to make the query more readable and structured.
- base_sales – Calculates revenue per transaction (Quantity * UnitPrice).
- customer_sales – Aggregates total revenue for each CustomerID.
- ranked – Ranks customers by revenue and calculates cumulative revenue and total customers.
- with_pct – Computes cumulative sales share (cum_sales_share) and cumulative percentage of customers (cum_pct_customers).
- Final SELECT – Finds the minimum number of customers required to reach the target revenue percentage (e.g., 60%).
```

```sql
-- Pareto Analysis using CTEs
DECLARE target_sales_pct FLOAT64 DEFAULT 0.60;

WITH base_sales AS (
  SELECT
    CustomerID,
    (Quantity * UnitPrice) AS sales
  FROM `pareto-4.paretodemo.pet_shop_sales`
),
customer_sales AS (
  SELECT
    CustomerID,
    SUM(sales) AS customer_revenue
  FROM base_sales
  GROUP BY CustomerID
),
ranked AS (
  SELECT
    CustomerID,
    customer_revenue,
    ROW_NUMBER() OVER(ORDER BY customer_revenue DESC) AS cum_customers,
    COUNT(*) OVER() AS total_customers,
    SUM(customer_revenue) OVER(ORDER BY customer_revenue DESC
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS cum_revenue,
    SUM(customer_revenue) OVER() AS total_revenue
  FROM customer_sales
),
with_pct AS (
  SELECT
    CustomerID,
    customer_revenue,
    cum_customers,
    total_customers,
    cum_revenue,
    total_revenue,
    cum_revenue / total_revenue AS cum_sales_share,
    cum_customers / total_customers AS cum_pct_customers
  FROM ranked
)
SELECT
  MIN(cum_customers) AS number_of_customers,
  MIN(total_customers) AS total_customers,
  MIN(cum_revenue) AS cum_revenue,
  MIN(total_revenue) AS total_revenue,
  target_sales_pct * 100 AS target_sales_percent,
  MIN(total_revenue * target_sales_pct) AS target_sales,
  MIN(cum_sales_share) AS cum_sales_share,
  MIN(cum_pct_customers) AS cum_pct_customers
FROM with_pct
WHERE cum_sales_share >= target_sales_pct;
```

### Key Metrics Computed
customer_revenue – Total revenue per customer
cum_revenue – Cumulative revenue as customers are ranked by revenue
cum_sales_share – Cumulative revenue percentage of total revenue
cum_pct_customers – Cumulative percentage of total customers
number_of_customers – Minimum number of customers needed to reach target revenue

## Key Takeaways
- A small percentage of customers typically drives the majority of sales, confirming the Pareto principle.
- Focusing on top customers can help maximize revenue efficiently.
- This approach is scalable and can be applied to any target sales percentage for strategic decision-making.
