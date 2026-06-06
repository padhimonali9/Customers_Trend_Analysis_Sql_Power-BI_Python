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

---

## 📁 Repository Structure

```
customer-shopping-behavior-analysis/
│
├── data/
│   └── customer_shopping_data.csv        # Raw dataset
│
├── notebooks/
│   └── shopping_behavior_eda.ipynb       # Python EDA & cleaning notebook
│
├── sql/
│   ├── 01_revenue_by_gender.sql
│   ├── 02_high_spending_discount_users.sql
│   ├── 03_top_products_by_rating.sql
│   ├── 04_shipping_type_comparison.sql
│   ├── 05_subscribers_vs_nonsubscribers.sql
│   ├── 06_discount_dependent_products.sql
│   ├── 07_customer_segmentation.sql
│   ├── 08_top_products_per_category.sql
│   ├── 09_repeat_buyers_subscriptions.sql
│   └── 10_revenue_by_age_group.sql
│
├── dashboard/
│   └── customer_behavior_dashboard.pbix  # Power BI file
│
├── reports/
│   └── Customer_Shopping_Behavior_Analysis.pdf
│
└── README.md
```

---

## 🚀 How to Run

### Prerequisites
```bash
pip install pandas numpy psycopg2-binary sqlalchemy
```

### Steps

1. **Clone the repository**
   ```bash
   git clone https://github.com/padhimonali9/customer-shopping-behavior-analysis.git
   cd customer-shopping-behavior-analysis
   ```

2. **Run the Python notebook**  
   Open `notebooks/shopping_behavior_eda.ipynb` in Jupyter or VS Code and run all cells. This cleans the data and loads it into PostgreSQL.

3. **Set up PostgreSQL**  
   Update the connection string in the notebook with your local credentials:
   ```python
   engine = create_engine("postgresql://username:password@localhost:5432/shopping_db")
   ```

4. **Run SQL queries**  
   Execute scripts from the `sql/` folder in your PostgreSQL client (pgAdmin or psql).

5. **Open Power BI Dashboard**  
   Open `dashboard/customer_behavior_dashboard.pbix` in Power BI Desktop. Update the data source to your PostgreSQL connection if needed.

---

## 👩‍💻 Author

**Monali Padhi**  
BSc Computer Science | Data Analyst  
📧 [LinkedIn](https://linkedin.com/in/monalipadhi) | 💻 [GitHub](https://github.com/padhimonali9)

---

## 📄 License

This project is for portfolio and educational purposes.
