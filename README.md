# 🛍️ Customer Shopping Behavior Analysis

An end-to-end data analysis project exploring shopping patterns, customer segments, and revenue drivers across **3,900 transactions**, using **Python** for data cleaning, **MySQL** for structured business-question analysis, and **Power BI** for interactive visualization.

---

## 📌 Project Overview

This project analyzes customer shopping behavior using transactional data from 3,900 purchases across various product categories. The goal is to uncover insights into spending patterns, customer segments, product preferences, and subscription behavior in order to guide strategic business decisions.

**Workflow:** `Python (cleaning & EDA)` → `MySQL (business analysis)` → `Power BI (dashboard)`

---

## 🗂️ Dataset Summary

| Detail | Value |
|---|---|
| Rows | 3,900 |
| Columns | 18 |
| Missing data | 37 values in `review_rating` |

**Key features:**
- **Customer demographics:** Age, Gender, Location, Subscription Status
- **Purchase details:** Item Purchased, Category, Purchase Amount, Season, Size, Color
- **Shopping behavior:** Discount Applied, Promo Code Used, Previous Purchases, Frequency of Purchases, Review Rating, Shipping Type

---

## 🐍 Data Preparation (Python)

Cleaning and feature engineering was done in `pandas` before loading the data into MySQL:

- **Data Loading** — imported the raw dataset with `pandas`
- **Initial Exploration** — used `df.info()` and `.describe()` to inspect structure and summary statistics
- **Missing Data Handling** — imputed missing `review_rating` values using the median rating per product category
- **Column Standardization** — renamed all columns to `snake_case`
- **Feature Engineering**
  - `age_group` — created by binning customer ages
  - `purchase_frequency_days` — derived from purchase frequency data
- **Data Consistency Check** — found `discount_applied` and `promo_code_used` were redundant; dropped `promo_code_used`
- **Database Load** — pushed the cleaned DataFrame into MySQL via SQLAlchemy

```python
from sqlalchemy import create_engine

engine = create_engine(
    f"mysql+mysqlconnector://{username}:{password}@{host}:{port}/{database}"
)

df.to_sql("customer", engine, if_exists="replace", index=False)
```

---

## 🗄️ SQL Analysis (MySQL)

All business questions were answered in MySQL Workbench — the full script is in [`MySQL_File.sql`](./MySQL_File.sql). It covers database setup, a column rename, and 10 business questions using `GROUP BY`, `CASE`, `HAVING`, `CTEs`, and window functions (`ROW_NUMBER() OVER (PARTITION BY ...)`).

| # | Business Question | Technique Used |
|---|---|---|
| 1 | Total revenue by gender | `GROUP BY` + `SUM` |
| 2 | Discount users who still spent above average | `WHERE` filter on a fixed benchmark |
| 3 | Top 5 products by average review rating | `GROUP BY` + `ORDER BY ... LIMIT` |
| 4 | Avg. purchase amount: Standard vs. Express shipping | `GROUP BY` + `AVG` |
| 5 | Subscriber vs. non-subscriber spend & revenue | `GROUP BY` + `SUM`/`AVG` |
| 6 | Top 5 products by discount usage | `WHERE` + `GROUP BY` + `ORDER BY` |
| 7 | Customer segmentation: New / Returning / Loyal | `CTE` + `CASE WHEN` |
| 8 | Top 3 products per category | `CTE` + `ROW_NUMBER() OVER (PARTITION BY ...)` |
| 9 | Repeat buyers (5+ purchases) vs. subscription status | `WHERE` + `GROUP BY` |
| 10 | Revenue contribution by age group | `GROUP BY` + `SUM` |

<details>
<summary><strong>Example — Customer segmentation (CTE + CASE)</strong></summary>

```sql
WITH customer_type AS (
    SELECT customer_id, previous_purchases,
    CASE 
        WHEN previous_purchases = 1 THEN 'New'
        WHEN previous_purchases BETWEEN 2 AND 10 THEN 'Returning'
        ELSE 'Loyal'
    END AS customer_segment
    FROM customer
)
SELECT customer_segment, COUNT(*) AS "Number of Customers"
FROM customer_type
GROUP BY customer_segment;
```
</details>

<details>
<summary><strong>Example — Top 3 products per category (window function)</strong></summary>

```sql
WITH item_counts AS (
    SELECT category, item_purchased,
    COUNT(customer_id) AS total_orders,
    ROW_NUMBER() OVER (PARTITION BY category ORDER BY COUNT(customer_id) DESC) AS item_rank
    FROM customer
    GROUP BY category, item_purchased
)
SELECT item_rank, category, item_purchased, total_orders
FROM item_counts
WHERE item_rank <= 3;
```
</details>

---

## 📊 Key Findings

| Insight | Result |
|---|---|
| **Male customers generate far more revenue than female customers** | $157,890 vs. $75,191 |
| **Non-subscribers drive most of the revenue** | $170,436 (non-subscribers) vs. $62,645 (subscribers) |
| **Most customers fall into the "Loyal" segment** | Loyal: 3,116 · Returning: 701 · New: 83 |
| **Highest-rated products** | Gloves (3.86), Sandals (3.84), Boots (3.82) |
| **Express shipping customers spend slightly more on average** | $60.48 vs. $58.46 (Standard) |
| **Most discount-dependent product** | Hat — 50% of purchases used a discount |
| **Repeat buyers (5+ purchases) mostly don't subscribe** | 2,518 non-subscribers vs. 958 subscribers |
| **Highest revenue-generating age group** | Young Adult ($62,143) |

---

## 📈 Dashboard (Power BI)

An interactive Power BI dashboard was built on top of the MySQL database, with filters for **Subscription Status**, **Gender**, **Category**, and **Shipping Type**.




<img width="1272" height="691" alt="dashboard" src="https://github.com/user-attachments/assets/513aa9cb-8d1f-40b0-97a5-c1f603b41def" />







**Visuals included:**
- KPI cards — Number of Customers, Average Purchase Amount, Average Review Rating
- Donut chart — % of Customers by Subscription Status
- Bar charts — Revenue by Category, Sales by Category, Revenue by Age Group, Sales by Age Group

---

## 💡 Business Recommendations

- **Boost Subscriptions** — promote exclusive benefits for subscribers, since non-subscribers currently generate ~73% of revenue.
- **Customer Loyalty Programs** — reward repeat buyers to move more of them into the "Loyal" segment.
- **Review Discount Policy** — balance sales boosts with margin control on heavily discounted items like Hats and Sneakers.
- **Product Positioning** — highlight top-rated and best-selling products (Gloves, Sandals, Jewelry, Blouse) in campaigns.
- **Targeted Marketing** — focus efforts on high-revenue age groups (Young Adult) and express-shipping users.

---

## 🛠️ Tech Stack

- **Python** — pandas (data cleaning, feature engineering)
- **MySQL / MySQL Workbench** — structured business-question analysis (CTEs, window functions, aggregations)
- **SQLAlchemy** — Python ↔ MySQL connection
- **Power BI** — interactive dashboard

---

## 📁 Repository Structure

```
├── MySQL_File.sql              # Full SQL script: setup + 10 business queries
├── dashboard.png               # Power BI dashboard screenshot
├── Customer_Shopping_Behavior_Analysis.docx   # Full written report
└── README.md
```

---

## ▶️ How to Reproduce

1. Clone this repo.
2. Load the cleaned dataset into MySQL:
   ```sql
   CREATE DATABASE custo_101;
   USE custo_101;
   ```
   Then load your cleaned CSV/DataFrame into a `customer` table (via `df.to_sql()` from Python, or MySQL Workbench's Table Data Import Wizard).
3. Run [`MySQL_File.sql`](./MySQL_File.sql) top to bottom in MySQL Workbench.
4. Connect Power BI to the same MySQL database (`Get Data → MySQL database`) to rebuild the dashboard.
