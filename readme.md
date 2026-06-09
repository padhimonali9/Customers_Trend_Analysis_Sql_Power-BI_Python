# 🛍️ Customer Shopping Behavior Analysis

> End-to-end data analysis project using Python, PostgreSQL, and Power BI to uncover actionable insights from 3,900 customer transactions.

---

## 📌 Project Overview

This project analyzes customer shopping behavior across various product categories to understand spending patterns, customer segments, product preferences, and subscription behavior. The insights derived are designed to guide strategic business decisions such as targeted marketing, loyalty programs, and discount policy optimization.

---

## 🗂️ Dataset

| Attribute | Details |
|---|---|
| **Total Records** | 3,900 transactions |
| **Total Columns** | 18 features |
| **Source Type** | Retail transactional dataset |
| **Missing Data** | 37 null values in `Review Rating` (imputed) |

### Key Features

- **Customer Demographics** — Age, Gender, Location, Subscription Status
- **Purchase Details** — Item Purchased, Category, Purchase Amount (USD), Season, Size, Color
- **Shopping Behavior** — Discount Applied, Promo Code Used, Previous Purchases, Frequency of Purchases, Review Rating, Shipping Type, Payment Method

---

## 🧰 Tech Stack

| Tool | Purpose |
|---|---|
| **Python (Pandas, NumPy)** | Data cleaning, EDA, feature engineering |
| **PostgreSQL** | Structured SQL analysis |
| **Power BI** | Interactive dashboard & visualization |

---

## 🔄 Project Workflow

```
Raw Dataset → Python EDA & Cleaning → PostgreSQL (SQL Analysis) → Power BI Dashboard → Business Insights
```

### 1. Python — Data Preparation & EDA

- Loaded dataset using `pandas`; explored structure with `df.info()` and `.describe()`
- Handled 37 missing `Review Rating` values by **imputing with category-level median**
- Renamed all columns to **snake_case** for consistency
- **Feature Engineering:**
  - `age_group` — age bins (Young Adult, Adult, Middle-aged, Senior)
  - `purchase_frequency_days` — derived from purchase frequency strings
- Verified redundancy between `discount_applied` and `promo_code_used` → dropped `promo_code_used`
- Loaded cleaned DataFrame into **PostgreSQL** via `psycopg2`

### 2. PostgreSQL — Business SQL Analysis

10 business questions answered with structured SQL queries:

| # | Analysis |
|---|---|
| 1 | Revenue by Gender |
| 2 | High-Spending Discount Users (above avg spend despite discount) |
| 3 | Top 5 Products by Average Review Rating |
| 4 | Shipping Type Comparison (Standard vs. Express) |
| 5 | Subscribers vs. Non-Subscribers (avg spend & total revenue) |
| 6 | Discount-Dependent Products (top 5 by discount rate %) |
| 7 | Customer Segmentation (New / Returning / Loyal) |
| 8 | Top 3 Products per Category (using window functions) |
| 9 | Repeat Buyers & Subscription Correlation (>5 purchases) |
| 10 | Revenue by Age Group |

### 3. Power BI — Interactive Dashboard

Built a fully interactive **Customer Behavior Dashboard** featuring:
- KPI cards: Total Customers (3.9K), Avg Purchase Amount ($59.76), Avg Review Rating (3.75)
- Donut chart: Subscription Status split (27% Yes / 73% No)
- Bar charts: Revenue by Category, Sales by Category
- Horizontal bar charts: Revenue by Age Group, Sales by Age Group
- Slicers: Subscription Status, Gender, Category, Shipping Type

---

## 📊 Key Findings

| Insight | Finding |
|---|---|
| **Gender Revenue Gap** | Male customers generated $157,890 vs. Female $75,191 — a 2× difference |
| **High-Spend Discount Users** | 839 customers used discounts but still spent above average |
| **Top Rated Products** | Gloves (3.86), Sandals (3.84), Boots (3.82) |
| **Shipping & Spend** | Express shipping users spend slightly more ($60.48 vs $58.46) |
| **Subscription Impact** | Near-identical avg spend (~$59.5) — non-subscribers drive more total revenue due to volume |
| **Discount-Dependent Products** | Hat (50%), Sneakers (49.66%), Coat (49.07%) most discount-reliant |
| **Customer Segments** | 3,116 Loyal / 701 Returning / 83 New |
| **Top Revenue Age Group** | Young Adults ($62,143) lead, followed closely by Middle-aged ($59,197) |

---

## 💡 Business Recommendations

1. **Boost Subscriptions** — Only 27% of customers subscribe. Promote exclusive subscriber perks (early access, free shipping) to convert the 73% non-subscribers.
2. **Loyalty Programs** — With 3,116 Loyal customers, introduce tiered rewards to retain them and incentivize Returning (701) customers to move up.
3. **Review Discount Policy** — Products like Hat and Sneakers see ~50% discounted purchases. Evaluate margin impact and consider limiting discount frequency on high-demand items.
4. **Product Positioning** — Feature top-rated products (Gloves, Sandals, Boots) prominently in marketing campaigns.
5. **Targeted Marketing** — Focus ad spend on Young Adults and Express-shipping users, who show higher revenue contribution and purchase value.




