# 📊 AtliQ Hardware Data Analytics Project | SQL Challenge

## 📌 Problem Statement

AtliQ Hardwares is a leading computer hardware manufacturer based in India with a global presence. The management identified a lack of sufficient insights for making fast, data-driven business decisions.

To address this challenge, the company planned to strengthen its data analytics team by recruiting junior data analysts. Tony Sharma, the Data Analytics Director, organized a SQL challenge to evaluate candidates' technical and interpersonal skills.

This project focuses on analyzing business data using **MySQL and Power BI** to generate actionable insights and support strategic decision-making.

## 🎯 1. ASK — Business Questions

The project addresses the following 10 business questions:

1. Identify the markets in which AtliQ Exclusive operates in the APAC region.
2. Calculate the percentage increase in unique products in FY 2021 compared with FY 2020.
3. Report the unique product count for each segment, sorted in descending order.
4. Identify the segment with the highest increase in unique products from FY 2020 to FY 2021.
5. Identify products with the highest and lowest manufacturing costs.
6. Find the top five customers with the highest average pre-invoice discount percentages in India for FY 2021.
7. Analyze monthly gross sales for AtliQ Exclusive.
8. Identify the quarter with the highest total sold quantity in FY 2020.
9. Identify the sales channel contributing the most gross sales in FY 2021 and calculate its percentage contribution.
10. Find the top three best-selling products in each division for FY 2021.

## 🗂️ 2. PREPARE — Data Collection

**Dataset source:** [Codebasics Resume Project Challenge](https://codebasics.io/challenge/codebasics-resume-project-challenge)

The project uses the `gdb023` database (`atliq_hardware_db`) and its associated metadata file.

### Database Tables

| Table | Description |
|---|---|
| `dim_customer` | Customer details, markets, regions, platforms, and channels |
| `dim_product` | Product details, categories, segments, and divisions |
| `fact_gross_price` | Product gross prices by fiscal year |
| `fact_manufacturing_cost` | Product manufacturing costs |
| `fact_pre_invoice_deductions` | Customer pre-invoice discount information |
| `fact_sales_monthly` | Monthly product sales and quantities |

### Key Data Fields

- **Customer:** `customer_code`, `customer`, `market`, `region`, `channel`, `platform`
- **Product:** `product_code`, `product`, `segment`, `division`, `category`
- **Sales:** `date`, `sold_quantity`, `fiscal_year`
- **Pricing:** `gross_price`, `manufacturing_cost`
- **Discounts:** `pre_invoice_discount_pct`

## ⚙️ 3. PROCESS — Tools & Techniques

### Tools Used

- **MySQL:** Data extraction, querying, aggregation, and business analysis.
- **Power BI:** Data validation and data preparation.

### SQL Concepts Applied

- SELECT, WHERE, GROUP BY, and ORDER BY
- INNER JOIN and multi-table joins
- Aggregate functions: `COUNT()`, `SUM()`, `AVG()`, `MIN()`, and `MAX()`
- Common Table Expressions (CTEs)
- Subqueries
- CASE statements
- Date and month functions
- Window functions: `RANK()`
- Percentage calculations and rounding
- Filtering with `LIMIT`

### Data Cleaning & Validation

- Replaced `Newzealand` with `New Zealand` in Power BI.
- Performed data validation across the datasets.
- Joined customer, product, pricing, discount, and sales tables to answer business questions.

## 🔍 4. ANALYZE — SQL Business Analysis

All 10 business questions were solved using MySQL queries.

### Q1. Markets Served by AtliQ Exclusive in APAC

```sql
SELECT DISTINCT market
FROM dim_customer
WHERE customer = 'Atliq Exclusive'
  AND region = 'APAC';
```

**Insight:** AtliQ Exclusive operates in eight APAC markets: India, Indonesia, Japan, the Philippines, South Korea, Australia, New Zealand, and Bangladesh.

### Q2. Unique Product Growth: FY 2020 vs. FY 2021

```sql
WITH cte AS (
    SELECT COUNT(DISTINCT product_code) AS unique_products_2020
    FROM fact_sales_monthly
    WHERE fiscal_year = 2020
),
cte1 AS (
    SELECT COUNT(DISTINCT product_code) AS unique_products_2021
    FROM fact_sales_monthly
    WHERE fiscal_year = 2021
)
SELECT *,
       ROUND(
           (unique_products_2021 - unique_products_2020)
           / unique_products_2020 * 100, 2
       ) AS percentage_chg
FROM cte, cte1;
```

**Insight:** Unique products increased from 245 in FY 2020 to 334 in FY 2021, representing approximately **36% growth**.

### Q3. Unique Products by Segment

```sql
SELECT segment,
       COUNT(product_code) AS product_count
FROM dim_product
GROUP BY segment
ORDER BY product_count DESC;
```

**Insight:** Notebooks, Accessories, and Peripherals were identified as the three leading segments.

### Q4. Segment-Wise Product Growth

```sql
WITH cte1 AS (
    SELECT d.segment,
           COUNT(DISTINCT f.product_code) AS product_count_2020
    FROM dim_product d
    JOIN fact_sales_monthly f
      ON d.product_code = f.product_code
    WHERE fiscal_year = 2020
    GROUP BY d.segment
),
cte2 AS (
    SELECT d.segment,
           COUNT(DISTINCT f.product_code) AS product_count_2021
    FROM dim_product d
    JOIN fact_sales_monthly f
      ON d.product_code = f.product_code
    WHERE fiscal_year = 2021
    GROUP BY d.segment
)
SELECT c1.segment,
       product_count_2020,
       product_count_2021,
       product_count_2021 - product_count_2020 AS difference
FROM cte1 c1
JOIN cte2 c2
  ON c1.segment = c2.segment
ORDER BY difference DESC;
```

**Insight:** Accessories recorded the highest increase in unique products in FY 2021 compared with FY 2020.

### Q5. Highest and Lowest Manufacturing Costs

```sql
SELECT d.product_code,
       d.product,
       f.manufacturing_cost
FROM dim_product d
JOIN fact_manufacturing_cost f
  ON d.product_code = f.product_code
WHERE manufacturing_cost = (
    SELECT MAX(manufacturing_cost)
    FROM fact_manufacturing_cost
)
OR manufacturing_cost = (
    SELECT MIN(manufacturing_cost)
    FROM fact_manufacturing_cost
)
ORDER BY manufacturing_cost DESC;
```

**Insight:**
- Highest manufacturing cost: `AQ HOME Allin1 Gen 2`
- Lowest manufacturing cost: `AQ Master wired x1 MS`

### Q6. Top Five Customers by Average Pre-Invoice Discount

```sql
SELECT d.customer_code,
       d.customer,
       ROUND(AVG(pre_invoice_discount_pct), 4)
           AS average_discount_percentage
FROM dim_customer d
JOIN fact_pre_invoice_deductions f
  ON d.customer_code = f.customer_code
WHERE f.fiscal_year = 2021
  AND d.market = 'India'
GROUP BY d.customer_code, d.customer
ORDER BY average_discount_percentage DESC
LIMIT 5;
```

**Insight:** Flipkart, Viveks, Ezone, Croma, and Amazon were identified among the customers receiving the highest average discounts in the Indian market for FY 2021.

### Q7. Monthly Gross Sales for AtliQ Exclusive

```sql
SELECT MONTHNAME(s.date) AS month,
       s.fiscal_year,
       ROUND(SUM(g.gross_price * s.sold_quantity), 2)
           AS gross_sales_amount
FROM fact_gross_price g
JOIN fact_sales_monthly s
  ON g.product_code = s.product_code
JOIN dim_customer d
  ON d.customer_code = s.customer_code
WHERE d.customer = 'Atliq Exclusive'
GROUP BY s.fiscal_year, MONTH(s.date), MONTHNAME(s.date)
ORDER BY s.fiscal_year, MONTH(s.date);
```

**Insight:** November was identified as the highest-performing month in both 2019 and 2020. September was the lowest-performing month in 2019, while March was the lowest-performing month in 2020.

### Q8. Quarterly Sales Quantity in FY 2020

```sql
WITH cte AS (
    SELECT MONTH(date) AS month_number,
           SUM(sold_quantity) AS total_quantity
    FROM fact_sales_monthly
    WHERE fiscal_year = 2020
    GROUP BY MONTH(date)
)
SELECT CASE
           WHEN month_number IN (9, 10, 11) THEN 'Q1'
           WHEN month_number IN (12, 1, 2) THEN 'Q2'
           WHEN month_number IN (3, 4, 5) THEN 'Q3'
           ELSE 'Q4'
       END AS quarter,
       SUM(total_quantity) AS total_sold_quantity
FROM cte
GROUP BY quarter
ORDER BY total_sold_quantity DESC;
```

**Insight:** Q1 (September, October, and November) recorded the highest total sold quantity in FY 2020.

### Q9. Gross Sales Contribution by Channel in FY 2021

```sql
WITH cte AS (
    SELECT d.channel,
           ROUND(
               SUM(f.sold_quantity * g.gross_price) / 1000000,
               2
           ) AS gross_sales_mln
    FROM dim_customer d
    JOIN fact_sales_monthly f
      ON d.customer_code = f.customer_code
    JOIN fact_gross_price g
      ON f.product_code = g.product_code
    WHERE f.fiscal_year = 2021
    GROUP BY d.channel
)
SELECT *,
       ROUND(
           gross_sales_mln /
           (SELECT SUM(gross_sales_mln) FROM cte) * 100,
           2
       ) AS percentage
FROM cte
ORDER BY gross_sales_mln DESC;
```

**Insight:** The Retailer channel contributed **73.23%** of gross sales in FY 2021, making it the leading sales channel.

### Q10. Top Three Products in Each Division

```sql
WITH cte AS (
    SELECT d.division,
           f.product_code,
           d.product,
           SUM(f.sold_quantity) AS total_sold_quantity,
           RANK() OVER (
               PARTITION BY d.division
               ORDER BY SUM(f.sold_quantity) DESC
           ) AS rank_order
    FROM dim_product d
    JOIN fact_sales_monthly f
      ON d.product_code = f.product_code
    WHERE f.fiscal_year = 2021
    GROUP BY d.division, f.product_code, d.product
)
SELECT division,
       product_code,
       product,
       total_sold_quantity,
       rank_order
FROM cte
WHERE rank_order IN (1, 2, 3)
ORDER BY division, rank_order;
```

**Insight:** Top-selling products included AQ Pen Drive 2 in 1 and AQ Pen Drive DRC in Network & Storage; AQ Gamers MS and AQ Maxima MS in Peripherals & Accessories; and AQ Digit and AQ Velocity in the PC division.

## 📈 5. SHARE — Project Results

The analysis identified product growth, segment performance, manufacturing cost differences, customer discount patterns, monthly sales trends, quarterly demand, and sales channel contributions.

The project also includes screenshots of query outputs for sharing and review.

## 💡 6. ACT — Key Business Insights

1. **Market reach:** AtliQ Exclusive operates in eight APAC markets.
2. **Product growth:** The unique product count increased by approximately 36% from FY 2020 to FY 2021.
3. **Segment performance:** Notebooks, Accessories, and Peripherals were the leading segments.
4. **Product expansion:** Accessories experienced the highest growth in unique products.
5. **Manufacturing costs:** AQ HOME Allin1 Gen 2 had the highest manufacturing cost, while AQ Master wired x1 MS had the lowest.
6. **Customer discounts:** The analysis highlighted the five customers receiving the highest average discounts in India.
7. **Monthly sales:** November was the strongest month in the reported 2019 and 2020 trends.
8. **Quarterly demand:** Q1 (September–November) recorded the highest sales quantity in FY 2020.
9. **Sales channels:** Retailers contributed 73.23% of FY 2021 gross sales.
10. **Product rankings:** The top-selling products were identified across Network & Storage, Peripherals & Accessories, and PC divisions.

## 🚀 Business Recommendations

- **Optimize online promotions:** Review discount strategies for Flipkart, Viveks, Ezone, Croma, and Amazon.
- **Improve low-performing months:** Consider targeted promotions during March and September.
- **Control manufacturing costs:** Investigate cost drivers for AQ HOME Allin1 Gen 2 and identify opportunities for improvement.
- **Replicate successful strategies:** Analyze the factors behind Q1's strong sales quantity and apply successful practices to other quarters.
- **Expand product opportunities:** Use segment-level growth and product rankings to support inventory planning and product strategy.

## 🧠 Key Learnings

Through this project, I strengthened my understanding of:

- Writing SQL queries to solve business problems.
- Joining multiple relational tables.
- Using CTEs and subqueries to structure complex queries.
- Applying aggregate functions for business metrics.
- Using window functions to rank products within divisions.
- Calculating growth percentages and sales channel contributions.
- Translating query results into actionable business insights.

## 🛠️ Skills Demonstrated

`MySQL` `SQL` `Data Analysis` `Business Insights` `CTEs` `JOINs` `Subqueries` `Window Functions` `Power BI` `Data Validation`

## 🙏 Acknowledgment

A big thank you to **Codebasics and Dhaval Patel** for providing valuable learning resources and the Resume Project Challenge, which helped me apply SQL concepts to a practical business scenario.

**Thank you for visiting and reviewing my repository!** ⭐

🔗 **Project Challenge:** [Codebasics Resume Project Challenge](https://codebasics.io/challenge/codebasics-resume-project-challenge)

#SQL #MySQL #DataAnalytics #Codebasics #DataAnalyst #BusinessIntelligence
