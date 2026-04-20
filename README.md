# 📊 Customer Sales Trend Analysis (SQL + Tableau)

---

## 📊 Sales Trend

![Sales Trend](sales_trend.png)

## 🧠 Overview

This project analyzes customer sales performance using SQL (BigQuery) and Tableau dashboards.

The goal is to understand sales trends, customer behavior, and conversion performance across the funnel in order to generate actionable business insights.

The dataset is stored in Google BigQuery public/project dataset environment and analyzed using SQL queries, then visualized in Tableau.

---

## 🛠 Tools Used

- SQL (BigQuery) – data extraction, transformation, aggregation
- Tableau – data visualization and dashboard creation
- Google Cloud BigQuery – data warehouse

---

## 📌 Key Business Questions

This project focuses on answering:

- Which channels generate the most revenue?
- What type of customers (new vs returning) drive sales?
- Where do users drop off in the purchase funnel?
- Which part of the conversion process needs improvement?

---

## 📈 Key Insights (from Tableau Dashboard)

---

### 1. Sales by Channel

- Paid Ads: 7.5M (highest contributor)
- Organic Search: 5.0M
- Social: 2.5M
- Email: 1.8M

👉 Paid advertising is the strongest revenue driver  
👉 Organic search shows strong cost-efficient performance  
👉 Social and email act as supporting channels  

---

### 2. Customer Type & Sales

- New customers drive the majority of revenue (~66%)
- Returning customers contribute less (~37%)

👉 Strong acquisition performance  
👉 Potential weakness in retention strategy  

---

### 3. Sales Group Analysis

- Paid Ads (New): ~19B
- Organic (New): ~13B
- Paid Ads (Returning): ~10B
- Organic (Returning): ~7B

👉 Acquisition-focused strategy is highly effective  
👉 Retention segment is under-optimized  

---

### 4. Purchase Funnel Analysis

- Visitors → Viewers: 65%
- Visitors → Purchase: 7%
- Cart → Purchase: 30%
- Checkout → Purchase: 50%

👉 Major drop-off occurs before purchase stage  
👉 Checkout completion rate indicates UX/payment friction  

---

## 📉 Key Bottlenecks Identified

- Low overall conversion rate (7%)
- Drop-off between browsing and cart addition
- Moderate checkout abandonment (50%)

---

## 💡 Business Recommendations

- Improve checkout UX and payment flow
- Optimize product page engagement (view → cart)
- Strengthen retention strategy (email, loyalty, remarketing)
- Reduce friction in final purchase steps

---

## 🧾 SQL Analysis

All SQL queries used for data cleaning, aggregation, and analysis are included in the `/sql` folder.

---

## 📊 Tableau Dashboard

Dashboard visualizes:

- Sales performance by channel
- Customer segmentation (new vs returning)
- Funnel conversion analysis
- Trend over time

---

## 👤 Author

Yusei Hosoya





-- =========================================
-- 1. Views and Visitors Detail
-- =========================================

SELECT 
  COUNT(DISTINCT user_id) AS total_users,
  COUNT(DISTINCT CASE WHEN user_type = 'New' THEN user_id END) AS new_users,
  COUNT(DISTINCT CASE WHEN user_type = 'Returning' THEN user_id END) AS returning_users,

  ROUND(
    COUNT(DISTINCT CASE WHEN user_type = 'New' THEN user_id END) * 100.0
    / COUNT(DISTINCT user_id), 2
  ) AS new_user_percentage,

  ROUND(
    COUNT(DISTINCT CASE WHEN user_type = 'Returning' THEN user_id END) * 100.0
    / COUNT(DISTINCT user_id), 2
  ) AS returning_user_percentage,

  ROUND(SUM(CASE WHEN user_type = 'New' THEN revenue ELSE 0 END),0) AS new_customer_sales,
  ROUND(SUM(CASE WHEN user_type = 'Returning' THEN revenue ELSE 0 END),0) AS returning_customer_sales,

  ROUND(SUM(CASE WHEN user_type = 'New' THEN revenue ELSE 0 END) * 100.0 / SUM(revenue),0) AS new_customer_sales_percentage,
  ROUND(SUM(CASE WHEN user_type = 'Returning' THEN revenue ELSE 0 END) * 100.0 / SUM(revenue),0) AS returning_customer_sales_percentage

FROM `lunar-carving-457020-h5.clean.marketing`;

---

-- =========================================
-- 2. Sales by Channel
-- =========================================

SELECT 
  ROUND(SUM(CASE WHEN channel = 'Email' THEN revenue ELSE 0 END),0) AS email_sales,
  ROUND(SUM(CASE WHEN channel = 'Email' THEN revenue ELSE 0 END) / SUM(revenue) * 100,0) AS email_sales_percentage,

  ROUND(SUM(CASE WHEN channel = 'Organic' THEN revenue ELSE 0 END),0) AS organic_sales,
  ROUND(SUM(CASE WHEN channel = 'Organic' THEN revenue ELSE 0 END) / SUM(revenue) * 100,0) AS organic_sales_percentage,

  ROUND(SUM(CASE WHEN channel = 'Paid Ads' THEN revenue ELSE 0 END),0) AS paid_ads_sales,
  ROUND(SUM(CASE WHEN channel = 'Paid Ads' THEN revenue ELSE 0 END) / SUM(revenue) * 100,0) AS paid_ads_sales_percentage,

  ROUND(SUM(CASE WHEN channel = 'Social' THEN revenue ELSE 0 END),0) AS social_sales,
  ROUND(SUM(CASE WHEN channel = 'Social' THEN revenue ELSE 0 END) / SUM(revenue) * 100,0) AS social_sales_percentage

FROM `lunar-carving-457020-h5.clean.marketing`;

---

-- =========================================
-- 3. Channel × User Type Analysis
-- =========================================

SELECT
  SUM(CASE WHEN channel = 'Paid Ads' AND user_type = 'New' THEN revenue ELSE 0 END) AS paid_ads_new_revenue,
  SUM(CASE WHEN channel = 'Paid Ads' AND user_type = 'Returning' THEN revenue ELSE 0 END) AS paid_ads_returning_revenue,

  SUM(CASE WHEN channel = 'Organic' AND user_type = 'New' THEN revenue ELSE 0 END) AS organic_new_revenue,
  SUM(CASE WHEN channel = 'Organic' AND user_type = 'Returning' THEN revenue ELSE 0 END) AS organic_returning_revenue

FROM `lunar-carving-457020-h5.clean.marketing`;

---

-- =========================================
-- 4. Repeat Customers Analysis
-- =========================================

WITH user_summary AS (
  SELECT
    user_id,
    user_type,
    COUNTIF(purchase_completed = TRUE) AS purchase_count
  FROM `lunar-carving-457020-h5.clean.marketing`
  GROUP BY user_id, user_type
)

SELECT
  user_type,
  COUNT(*) AS total_users,
  COUNT(CASE WHEN purchase_count >= 2 THEN 1 END) AS repeat_users,
  ROUND(
    COUNT(CASE WHEN purchase_count >= 2 THEN 1 END) * 100.0 / COUNT(*),
    2
  ) AS repeat_rate
FROM user_summary
GROUP BY user_type;

---

-- =========================================
-- 5. Funnel Analysis
-- =========================================

WITH stages AS (
  SELECT
    SUM(CASE WHEN visited_website = TRUE THEN 1 ELSE 0 END) AS total_visitors,
    SUM(CASE WHEN viewed_product = TRUE THEN 1 ELSE 0 END) AS total_viewers,
    SUM(CASE WHEN added_to_cart = TRUE THEN 1 ELSE 0 END) AS total_cart,
    SUM(CASE WHEN checkout_started = TRUE THEN 1 ELSE 0 END) AS total_checkout,
    SUM(CASE WHEN purchase_completed = TRUE THEN 1 ELSE 0 END) AS total_purchase
  FROM `lunar-carving-457020-h5.clean.marketing`
)

SELECT 
  total_visitors,
  total_viewers,
  ROUND(total_viewers * 100.0 / total_visitors, 0) AS visitors_to_viewers,

  total_cart,
  ROUND(total_cart * 100.0 / total_viewers, 0) AS viewers_to_cart,

  total_checkout,
  ROUND(total_checkout * 100.0 / total_cart, 0) AS cart_to_checkout,

  total_purchase,
  ROUND(total_purchase * 100.0 / total_checkout, 0) AS checkout_to_purchase,

  ROUND(total_purchase * 100.0 / total_visitors, 0) AS visitors_to_purchase

FROM stages;

---

-- =========================================
-- 6. Revenue Analysis
-- =========================================

WITH base AS (
  SELECT
    COUNT(CASE WHEN visited_website = TRUE THEN user_id END) AS total_visitors,
    COUNT(CASE WHEN purchase_completed = TRUE THEN user_id END) AS total_buyers,
    SUM(CASE WHEN purchase_completed = TRUE THEN revenue ELSE 0 END) AS total_revenue
  FROM `lunar-carving-457020-h5.clean.marketing`
)

SELECT 
  total_visitors,
  total_buyers,
  total_revenue,
  ROUND(total_revenue / NULLIF(total_visitors, 0), 2) AS revenue_per_visitor,
  ROUND(total_revenue / NULLIF(total_buyers, 0), 2) AS revenue_per_buyer
FROM base;

---

-- =========================================
-- 7. Top Customers (LTV)
-- =========================================

SELECT
  user_id,
  SUM(revenue) AS lifetime_value,
  COUNT(*) AS purchase_times
FROM `lunar-carving-457020-h5.clean.marketing`
WHERE purchase_completed = TRUE
GROUP BY user_id
ORDER BY lifetime_value DESC;
