# Online-retail-analysis_stephen-kimani
E-commerce sales analysis of 541,909 transactions using Python, SQL (BigQuery), and Power BI — covering revenue trends, RFM customer segmentation, product performance, and cancellation insights.
# Online Retail Sales Analysis

A deep-dive data analytics project by Stephen Kimani Irungu that transforms raw online retail transaction data into clear business intelligence using SQL, Python, and Power BI [file:1].

This project analyzes 541,909 raw transaction rows from a UK-based e-commerce retailer covering Dec 2010 to Dec 2011, then cleans the dataset down to 397,884 valid records for analysis [file:1]. The final output reveals revenue trends, customer behavior, top-performing products, geographic concentration, repeat purchase patterns, and operational risks such as cancellations [file:1].

## Project Overview

This project investigates how an online retail business performed across a 13-month period and answers the questions that matter to management: which customers are most valuable, which products generate the most revenue, when demand peaks, which countries contribute most, and why so many orders are cancelled [file:1].

The analysis combines structured SQL, Python cleaning and segmentation, and dashboard-style visualization to turn raw transaction data into actionable decisions [file:1]. The project is designed to be both a portfolio showcase and a practical business analysis workflow [file:1].

## Business Problem

The retailer had a large amount of transaction data but lacked a clear operational view of the business [file:1]. Management needed answers to questions such as: who are the best customers, what products should be stocked, when does revenue peak, which markets matter most, and what is driving the high cancellation rate [file:1].

The business was effectively operating without visibility into revenue drivers, customer value, demand timing, and geographic dependence [file:1]. That creates risks in inventory planning, staffing, customer retention, and market growth [file:1].

## Objectives

The project was built to do the following:

- Clean and validate raw transaction data.
- Measure revenue performance over time.
- Identify top-selling products and countries.
- Segment customers using RFM analysis.
- Detect peak shopping periods.
- Quantify repeat purchase behavior.
- Surface business risks and growth opportunities [file:1].

## Dataset

The analysis uses the UCI Machine Learning Repository Online Retail Dataset from Dec 2010 to Dec 2011 [file:1].

### Raw dataset highlights

- Total raw records: 541,909 [file:1].
- Clean records after filtering: 397,884 [file:1].
- Countries covered: 38 [file:1].
- Cancelled orders: 9,288 [file:1].
- Missing CustomerIDs: 135,080 [file:1].

### Main fields

| Column | Type | Description | Example |
|---|---|---|---|
| InvoiceNo | String | Unique order ID; begins with C for cancellations | 536365, C541433 [file:1] |
| StockCode | String | Product code | 85123A [file:1] |
| Description | String | Product name | WHITE HANGING HEART T-LIGHT HOLDER [file:1] |
| Quantity | Integer | Units ordered; negative values indicate returns | 6, -1 [file:1] |
| InvoiceDate | DateTime | Date and time of transaction | 2011-01-18 10:17 [file:1] |
| UnitPrice | Float | Price per unit in GBP | 2.55 [file:1] |
| CustomerID | Float | Unique customer identifier | 17850.0 [file:1] |
| Country | String | Customer country | United Kingdom [file:1] |

## Tools and Technologies

The project uses the following tools:

- Python 3 for data cleaning, transformation, and RFM scoring [file:1].
- Pandas and NumPy for data wrangling and analysis [file:1].
- SQL for aggregation, filtering, and segmentation logic [file:1].
- BigQuery-compatible SQL for structured querying [file:1].
- Power BI and Looker Studio for dashboard creation and interactive reporting [file:1].
- Microsoft Excel for early exploration and profiling [file:1].
- Jupyter Notebook for step-by-step documentation and EDA [file:1].
- Git and GitHub for version control and portfolio hosting [file:1].

## Methodology

The analysis followed a structured process:

1. Loaded the raw retail dataset.
2. Inspected data quality issues.
3. Cleaned invalid rows and created a usable transaction table.
4. Built revenue fields and time-based summaries.
5. Calculated product, country, and customer metrics.
6. Performed RFM customer segmentation.
7. Built dashboard visuals for stakeholder communication [file:1].

This workflow ensures the findings are not just descriptive, but tied to concrete business actions [file:1].

## SQL Queries Step by Step

The SQL layer was used to inspect, clean, aggregate, and segment the dataset [file:1].

### 1. Inspect raw data structure

```sql
-- Preview first 10 rows to understand the dataset
SELECT *
FROM onlineretail.transactions
LIMIT 10;
Purpose:
Check column names.
Confirm data formats.
Understand the raw structure before cleaning
###2. Check nulls and data quality issues
-- Count missing values per column
SELECT
  COUNTIF(InvoiceNo IS NULL) AS nullinvoice,
  COUNTIF(CustomerID IS NULL) AS nullcustomer,
  COUNTIF(Description IS NULL) AS nulldescription,
  COUNTIF(Quantity < 0) AS zeroornegativeqty,
  COUNTIF(UnitPrice < 0) AS zeroornegativeprice,
  COUNTIF(STARTSWITH(InvoiceNo, 'C')) AS cancelledorders
FROM onlineretail.transactions;
Purpose:
Measure missing values.
Identify negative quantities and prices.
Count cancelled orders
### 3. Create a clean working table
-- Clean table: only valid, completed, identified transactions
CREATE OR REPLACE TABLE onlineretail.cleantransactions AS
SELECT
  InvoiceNo,
  StockCode,
  Description,
  Quantity,
  PARSEDATETIME('%m/%d/%Y %H:%M', InvoiceDate) AS InvoiceDate,
  UnitPrice,
  CustomerID,
  Country,
  Quantity * UnitPrice AS Revenue
FROM onlineretail.transactions
WHERE CustomerID IS NOT NULL
  AND Quantity > 0
  AND UnitPrice > 0
  AND NOT STARTSWITH(InvoiceNo, 'C');
Purpose:
Remove anonymous orders.
Remove cancelled rows.
Remove non-positive quantities and prices.
Add a revenue metric
### 4. Monthly revenue trend
SELECT
  FORMAT_DATE('%Y-%m', InvoiceDate) AS Month,
  ROUND(SUM(Revenue), 2) AS MonthlyRevenue,
  COUNT(DISTINCT InvoiceNo) AS TotalOrders,
  COUNT(DISTINCT CustomerID) AS ActiveCustomers
FROM onlineretail.cleantransactions
GROUP BY Month
ORDER BY Month;
Purpose:
Track performance over time.
See revenue seasonality.
Monitor active customers and order volume
### 5. Revenue by country
SELECT
  Country,
  ROUND(SUM(Revenue), 2) AS TotalRevenue,
  COUNT(DISTINCT CustomerID) AS Customers,
  ROUND(SUM(Revenue) * 100.0 / SUM(SUM(Revenue)) OVER (), 2) AS RevenueSharePct
FROM onlineretail.cleantransactions
GROUP BY Country
ORDER BY TotalRevenue DESC
LIMIT 10;
Purpose:
Identify dominant markets.
Measure revenue concentration.
Rank top international markets
### 6. Top products by revenue
SELECT
  StockCode,
  Description,
  SUM(Quantity) AS UnitsSold,
  ROUND(AVG(UnitPrice), 2) AS AvgPrice,
  ROUND(SUM(Revenue), 2) AS TotalRevenue
FROM onlineretail.cleantransactions
GROUP BY StockCode, Description
ORDER BY TotalRevenue DESC
LIMIT 10;
Purpose:
Find most profitable products.
Support inventory planning.
Reveal bulk-order items
###7. Customer RFM segmentation
WITH rfmbase AS (
  SELECT
    CustomerID,
    DATEDIFF('2011-12-10', MAX(DATE(InvoiceDate)), DAY) AS Recency,
    COUNT(DISTINCT InvoiceNo) AS Frequency,
    ROUND(SUM(Revenue), 2) AS Monetary
  FROM onlineretail.cleantransactions
  GROUP BY CustomerID
),
rfmscored AS (
  SELECT
    *,
    NTILE(4) OVER (ORDER BY Recency DESC) AS R,
    NTILE(4) OVER (ORDER BY Frequency) AS F,
    NTILE(4) OVER (ORDER BY Monetary) AS M
  FROM rfmbase
)
SELECT
  *,
  CONCAT(CAST(R AS STRING), CAST(F AS STRING), CAST(M AS STRING)) AS RFMSegment
FROM rfmscored
ORDER BY Monetary DESC;
Purpose:
Score customers by recency, frequency, and monetary value.
Identify champions and low-value groups.
Support retention targeting
### 8. Peak shopping hour
SELECT
  EXTRACT(HOUR FROM InvoiceDate) AS HourofDay,
  COUNT(DISTINCT InvoiceNo) AS NumberofOrders,
  ROUND(SUM(Revenue), 2) AS Revenue
FROM onlineretail.cleantransactions
GROUP BY HourofDay
ORDER BY NumberofOrders DESC;
Purpose:
Determine busiest trading hours.
Optimize staffing and operations
### 9. Repeat purchase rate
SELECT
  COUNT(DISTINCT CustomerID) AS TotalCustomers,
  COUNTIF(ordercount > 1) AS RepeatCustomers,
  ROUND(COUNTIF(ordercount > 1) * 100.0 / COUNT(*), 1) AS RetentionRatePct
FROM (
  SELECT
    CustomerID,
    COUNT(DISTINCT InvoiceNo) AS ordercount
  FROM onlineretail.cleantransactions
  GROUP BY CustomerID
);
Purpose:
Measure customer retention.
Quantify repeat buying behavior
###Python was used for cleaning, EDA, and customer segmentation
###1. Load the dataset
import pandas as pd
import numpy as np

df = pd.read_csv("OnlineRetail.csv", encoding="latin1")
Purpose:
Load the raw dataset.
Use latin1 encoding to handle special characters
###2. Parse dates and standardize types
df["InvoiceDate"] = pd.to_datetime(df["InvoiceDate"])
df["InvoiceNo"] = df["InvoiceNo"].astype(str)
Purpose:
Prepare time-based analysis.
Ensure invoice values are treated consistently
###3. Create revenue column
df["Revenue"] = df["Quantity"] * df["UnitPrice"]
Purpose:
Build the core financial metric used throughout the analysis
###4. Clean the dataset
df_clean = df[
    (df["Quantity"] > 0) &
    (df["UnitPrice"] > 0) &
    (df["CustomerID"].notna()) &
    (~df["InvoiceNo"].str.startswith("C"))
].copy()
Purpose:
Remove returns, cancellations, zero/negative values, and missing customers
###5. Validate raw vs clean record counts
The process reduced the data from 541,909 raw rows to 397,884 clean rows.That means the analysis uses only validated transactions for trustworthy business conclusions
###6. Build RFM model
snapshot_date = df_clean["InvoiceDate"].max() + pd.Timedelta(days=1)

rfm = df_clean.groupby("CustomerID").agg(
    Recency=("InvoiceDate", lambda x: (snapshot_date - x.max()).days),
    Frequency=("InvoiceNo", "nunique"),
    Monetary=("Revenue", "sum")
).reset_index()
Purpose:
Calculate customer behavior metrics.
Measure how recently, how often, and how much each customer buys
###7. Score customers
rfm["RScore"] = pd.qcut(rfm["Recency"], 4, labels=[1])
rfm["FScore"] = pd.qcut(rfm["Frequency"].rank(method="first"), 4, labels=[1])
rfm["MScore"] = pd.qcut(rfm["Monetary"], 4, labels=[1])
rfm["RFMScore"] = rfm["RScore"].astype(str) + rfm["FScore"].astype(str) + rfm["MScore"].astype(str)
Purpose:
Turn raw behavior into customer segments.
Identify high-value and at-risk groups
###8. Identify champion customers
champions = rfm[rfm["RFMScore"] == "444"]
The analysis found 328 champion customers who buy recently, frequently, and spend the most
##Power BI Dashboard
Power BI was used to turn the analysis into a business-facing dashboard . The dashboard presents KPI cards, trend visuals, product rankings, country performance, hourly patterns, and customer retention in an interactive format
###Dashboard sections
KPI overview for revenue, orders, customers, average order value, repeat rate, and product count 
Monthly revenue trend to show seasonality and growth
Revenue by country to show geographic dependence .
Top products by revenue to guide inventory and merchandising
Hourly order distribution to support operational planning .
Retention and repeat purchase visuals to support customer strategy
The dashboard is important because it translates analysis into something decision-makers can use quickly without reading code
###Charts Included
These charts were included in the analysis report 
Monthly Revenue Trend
Shows monthly revenue from Dec 2010 to Dec 2011, with a strong Q4 surge and a peak in November 2011 at 1.16M 
Revenue by Country
A horizontal bar chart showing that the United Kingdom contributes about 82% of total revenue, with the Netherlands, Ireland, Germany, and France trailing behind .
Top Products by Revenue
A product ranking chart showing that PAPER CRAFT , LITTLE BIRDIE leads with about 168K revenue, followed by REGENCY CAKESTAND 3 TIER and WHITE HANGING HEART T-LIGHT HOLDER .
Orders by Hour of Day
A bar chart showing peak activity between 10 AM and 2 PM, with 12 PM as the busiest hour at 3,130 orders 
Customer Purchase Frequency
A doughnut chart showing 65.6% repeat customers versus 34.4% one-time customers
## Key Findings
The analysis produced several business insights:
Revenue peaked in November 2011 at 1.16M, driven by pre-Christmas demand ].
The UK contributed 82% of revenue, creating concentration risk .
65.6% of customers bought more than once, showing strong retention potential .
12 PM was the busiest ordering hour, with 3,130 orders .
9,288 cancelled orders represent a meaningful revenue leakage issue .
328 customers were classified as RFM 444 champions.
The most valuable products were mostly gift and homeware items, many ordered in bulk .
International markets such as the Netherlands, Germany, and France show growth potential
##Business Recommendations
Based on the analysis, the report recommends the following:
Investigate and reduce cancellations, since 9,288 cancelled orders may hide stock, payment, or checkout issues .
Launch a VIP retention programme for the 328 champion customers .
Build Q4 inventory and staffing plans around the August-to-November surge .
Expand international marketing in markets with strong average order value .
Capture customer IDs at checkout to reduce anonymous orders .
Explore a formal B2B wholesale channel for high-volume products
##Project Outcome
This project converted a noisy raw dataset into a validated analytical model with measurable outcomes . It produced a clean dataset, revenue reporting, customer segmentation, and dashboard-ready visuals that answer the most important business questions .
In practical terms, the project shows how SQL, Python, and Power BI can work together to support revenue growth, retention strategy, inventory planning, and market expansion
###how to run
Clone the repository.
Place the dataset in the project folder.
Run the SQL scripts to build the clean transaction table.
Execute the Python notebook for cleaning and RFM analysis.
Open the Power BI file to explore the dashboard.
Review the exported charts and final report
###Repository Structure
.
├── data/
│   └── OnlineRetail.csv
├── sql/
│   ├── 01_data_quality.sql
│   ├── 02_clean_table.sql
│   ├── 03_revenue_analysis.sql
│   ├── 04_customer_rfm.sql
│   └── 05_hourly_analysis.sql
├── notebooks/
│   ├── data_cleaning.ipynb
│   └── rfm_analysis.ipynb
├── powerbi/
│   └── online_retail_dashboard.pbix
├── visuals/
│   ├── monthly_revenue.png
│   ├── revenue_by_country.png
│   ├── top_products.png
│   ├── hourly_orders.png
│   └── retention_chart.png
└── README.md
##AUTHOR
##STEPHEN KIMANI IRUNGU
##STEPHENKIMANIIRUNGU6542@GMAIL.COM
##DATA ENGINEER AND BUSINESS INTELLIGENCE ANALYST
##NAIROBI,KENYA

