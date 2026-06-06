## 1. Project Objective

The goal of this project is to analyze retail customer shopping behavior using a transactional dataset of 3,900 purchases. The analysis addresses four key business domains:

- **Spending Patterns** — How much do customers spend, and what drives higher spend?
- **Customer Segmentation** — Who are the most valuable customers?
- **Product Preferences** — Which products perform best by sales volume and review ratings?
- **Subscription & Discount Behavior** — Do subscribers spend more? Are discounts effective?

The project follows a full analytics pipeline: raw data → Python cleaning → SQL analysis → Power BI dashboard → business recommendations.

---

## 2. Dataset Overview

### Basic Statistics

| Metric | Value |
|---|---|
| Total Rows | 3,900 |
| Total Columns | 18 |
| Customer Age Range | 18 – 70 years |
| Average Customer Age | 44.07 years |
| Purchase Amount Range | $20 – $100 |
| Average Purchase Amount | $59.76 |
| Average Review Rating | 3.75 / 5.0 |
| Missing Values | 37 (Review Rating column only) |

### Feature Categories

**Demographics**

| Column | Type | Description |
|---|---|---|
| `customer_id` | Integer | Unique customer identifier |
| `age` | Integer | Customer age (18–70) |
| `gender` | Categorical | Male / Female |
| `location` | Categorical | US state |
| `subscription_status` | Categorical | Yes / No |

**Purchase Details**

| Column | Type | Description |
|---|---|---|
| `item_purchased` | Categorical | Product name (25 unique items) |
| `category` | Categorical | Clothing, Accessories, Footwear, Outerwear |
| `purchase_amount_usd` | Integer | Purchase value ($20–$100) |
| `season` | Categorical | Spring, Summer, Fall, Winter |
| `size` | Categorical | XS, S, M, L, XL, XXL |
| `color` | Categorical | 25 unique color options |

**Shopping Behavior**

| Column | Type | Description |
|---|---|---|
| `review_rating` | Float | Product rating (2.5–5.0) |
| `shipping_type` | Categorical | 5 types (Standard, Express, Free, etc.) |
| `discount_applied` | Categorical | Yes / No |
| `promo_code_used` | Categorical | Yes / No (dropped — redundant) |
| `previous_purchases` | Integer | Number of prior purchases (1–50) |
| `frequency_of_purchases` | Categorical | Weekly, Bi-Weekly, Monthly, etc. |
| `payment_method` | Categorical | PayPal, Credit Card, etc. |

---

## 3. Data Cleaning & Preparation (Python)

### 3.1 Missing Data Treatment

The only column with missing values was `review_rating` (37 nulls, ~0.95% of data).

**Approach:** Category-level median imputation — each null was filled with the median rating of its product category rather than the global median. This preserves category-specific rating distributions.

```python
df['review_rating'] = df.groupby('category')['review_rating'].transform(
    lambda x: x.fillna(x.median())
)
```

### 3.2 Column Standardization

All column names were renamed to `snake_case` for consistency across Python, SQL, and Power BI:

```python
df.columns = df.columns.str.lower().str.replace(' ', '_').str.replace('(', '').str.replace(')', '')
```

### 3.3 Feature Engineering

**Age Group Binning**

Customer ages were bucketed into four segments for demographic analysis:

```python
bins = [0, 30, 45, 60, 100]
labels = ['Young Adult', 'Adult', 'Middle-aged', 'Senior']
df['age_group'] = pd.cut(df['age'], bins=bins, labels=labels)
```

| Age Group | Age Range |
|---|---|
| Young Adult | 18 – 30 |
| Adult | 31 – 45 |
| Middle-aged | 46 – 60 |
| Senior | 61+ |

**Purchase Frequency Days**

The `frequency_of_purchases` text column was mapped to numeric day values:

```python
freq_map = {
    'Weekly': 7, 'Bi-Weekly': 14, 'Monthly': 30,
    'Quarterly': 90, 'Every 3 Months': 90, 'Annually': 365
}
df['purchase_frequency_days'] = df['frequency_of_purchases'].map(freq_map)
```

### 3.4 Redundancy Check & Column Drops

A cross-tabulation of `discount_applied` vs `promo_code_used` showed a near-perfect correlation — every row where a promo code was used also had a discount applied. The `promo_code_used` column was dropped to avoid multicollinearity.

### 3.5 Database Integration

The cleaned DataFrame was loaded into a PostgreSQL database using SQLAlchemy:

```python
from sqlalchemy import create_engine
engine = create_engine("postgresql://user:password@localhost:5432/shopping_db")
df.to_sql('customer_shopping', engine, if_exists='replace', index=False)
```

---

## 4. Exploratory Data Analysis

### 4.1 Summary Statistics

| Metric | Value |
|---|---|
| Most Common Gender | Male (2,652 out of 3,900) |
| Most Common Location | Montana |
| Most Popular Category | Clothing |
| Most Purchased Item | Blouse |
| Most Common Payment | PayPal |
| Most Common Shipping | Free Shipping |

### 4.2 Distribution Highlights

- **Purchase Amount** is relatively uniformly distributed between $20 and $100, with a median of $60.
- **Review Rating** ranges from 2.5 to 5.0, with a mean of 3.75 — slightly right of center, indicating general customer satisfaction.
- **Previous Purchases** range from 1 to 50, with a mean of ~25, indicating a largely repeat-purchase customer base.
- **Subscription Status**: 73% of customers are non-subscribers, suggesting significant conversion opportunity.

---

## 5. SQL Analysis — Business Questions

All queries were executed in PostgreSQL on the cleaned `customer_shopping` table.

---

### Q1. Revenue by Gender

**Business Question:** Which gender segment generates more revenue?

```sql
SELECT gender, SUM(purchase_amount_usd) AS revenue
FROM customer_shopping
GROUP BY gender
ORDER BY revenue DESC;
```

| Gender | Revenue (USD) |
|---|---|
| Male | $157,890 |
| Female | $75,191 |

**Insight:** Male customers account for over twice the revenue of female customers, largely due to higher transaction volume (2,652 vs 1,248 customers).

---

### Q2. High-Spending Discount Users

**Business Question:** Which customers used discounts but still spent above the average purchase amount?

```sql
SELECT customer_id, purchase_amount_usd
FROM customer_shopping
WHERE discount_applied = 'Yes'
  AND purchase_amount_usd > (SELECT AVG(purchase_amount_usd) FROM customer_shopping)
ORDER BY purchase_amount_usd DESC;
```

**Result:** 839 customers — these are high-value customers who are price-sensitive but not discount-dependent. They represent a strong retention target.

---

### Q3. Top 5 Products by Average Review Rating

```sql
SELECT item_purchased, ROUND(AVG(review_rating), 2) AS avg_rating
FROM customer_shopping
GROUP BY item_purchased
ORDER BY avg_rating DESC
LIMIT 5;
```

| Rank | Product | Avg Rating |
|---|---|---|
| 1 | Gloves | 3.86 |
| 2 | Sandals | 3.84 |
| 3 | Boots | 3.82 |
| 4 | Hat | 3.80 |
| 5 | Skirt | 3.78 |

---

### Q4. Shipping Type Comparison

```sql
SELECT shipping_type, ROUND(AVG(purchase_amount_usd), 2) AS avg_purchase
FROM customer_shopping
WHERE shipping_type IN ('Standard', 'Express')
GROUP BY shipping_type;
```

| Shipping Type | Avg Purchase (USD) |
|---|---|
| Express | $60.48 |
| Standard | $58.46 |

**Insight:** Express shipping customers spend ~$2 more on average — they may be higher-intent or premium shoppers.

---

### Q5. Subscribers vs. Non-Subscribers

```sql
SELECT subscription_status,
       COUNT(*) AS total_customers,
       ROUND(AVG(purchase_amount_usd), 2) AS avg_spend,
       SUM(purchase_amount_usd) AS total_revenue
FROM customer_shopping
GROUP BY subscription_status;
```

| Status | Customers | Avg Spend | Total Revenue |
|---|---|---|---|
| Yes | 1,053 | $59.49 | $62,645 |
| No | 2,847 | $59.87 | $170,436 |

**Insight:** Subscribers and non-subscribers spend nearly the same per transaction. Non-subscribers dominate total revenue purely due to volume — making subscription conversion a high-impact growth lever.

---

### Q6. Discount-Dependent Products

```sql
SELECT item_purchased,
       ROUND(100.0 * SUM(CASE WHEN discount_applied = 'Yes' THEN 1 ELSE 0 END) / COUNT(*), 2) AS discount_rate
FROM customer_shopping
GROUP BY item_purchased
ORDER BY discount_rate DESC
LIMIT 5;
```

| Rank | Product | Discount Rate (%) |
|---|---|---|
| 1 | Hat | 50.00 |
| 2 | Sneakers | 49.66 |
| 3 | Coat | 49.07 |
| 4 | Sweater | 48.17 |
| 5 | Pants | 47.37 |

**Insight:** These products may be struggling to sell at full price. Discounting at ~50% frequency signals potential pricing, quality, or positioning issues.

---

### Q7. Customer Segmentation

```sql
SELECT
  CASE
    WHEN previous_purchases >= 10 THEN 'Loyal'
    WHEN previous_purchases BETWEEN 3 AND 9 THEN 'Returning'
    ELSE 'New'
  END AS customer_segment,
  COUNT(*) AS number_of_customers
FROM customer_shopping
GROUP BY customer_segment
ORDER BY number_of_customers DESC;
```

| Segment | Customers |
|---|---|
| Loyal | 3,116 |
| Returning | 701 |
| New | 83 |

**Insight:** The overwhelming majority of customers (80%) are Loyal, indicating strong retention but limited new customer acquisition.

---

### Q8. Top 3 Products per Category

Used `RANK()` window function to rank products within each category by order volume.

| Category | Rank 1 | Rank 2 | Rank 3 |
|---|---|---|---|
| Accessories | Jewelry (171) | Sunglasses (161) | Belt (161) |
| Clothing | Blouse (171) | Pants (171) | Shirt (169) |
| Footwear | Sandals (160) | Shoes (150) | Sneakers (145) |
| Outerwear | Jacket (163) | Coat (161) | — |

---

### Q9. Repeat Buyers & Subscription Correlation

```sql
SELECT subscription_status, COUNT(*) AS repeat_buyers
FROM customer_shopping
WHERE previous_purchases > 5
GROUP BY subscription_status;
```

| Subscription | Repeat Buyers |
|---|---|
| No | 2,518 |
| Yes | 958 |

**Insight:** Even among heavy repeat buyers (>5 purchases), the majority are non-subscribers — suggesting the subscription program is not effectively capturing loyal customers.

---

### Q10. Revenue by Age Group

| Age Group | Total Revenue (USD) |
|---|---|
| Young Adult | $62,143 |
| Middle-aged | $59,197 |
| Adult | $55,978 |
| Senior | $55,763 |

**Insight:** Young Adults (18–30) are the highest-revenue age group. Marketing efforts targeting this segment could yield strong returns.

---

## 6. Power BI Dashboard

The interactive dashboard was built in Power BI Desktop using data imported from PostgreSQL.

### Dashboard Components

| Visual | Type | Purpose |
|---|---|---|
| Number of Customers | KPI Card | Total customer count (3.9K) |
| Avg Purchase Amount | KPI Card | Average transaction value ($59.76) |
| Avg Review Rating | KPI Card | Customer satisfaction proxy (3.75) |
| Subscription Split | Donut Chart | 27% Yes / 73% No |
| Revenue by Category | Bar Chart | Clothing leads in revenue |
| Sales by Category | Bar Chart | Volume comparison across categories |
| Revenue by Age Group | Horizontal Bar | Young Adults generate most revenue |
| Sales by Age Group | Horizontal Bar | Age-wise transaction volume |

### Slicers (Filters)

- Subscription Status (Yes / No)
- Gender (Male / Female)
- Category (Accessories, Clothing, Footwear, Outerwear)
- Shipping Type (5 options)

---

## 7. Key Insights Summary

| Area | Insight |
|---|---|
| Revenue | Male customers generate 2× more revenue than female customers |
| Discounts | 839 high-value customers use discounts but spend above average |
| Products | Accessories (Gloves, Sandals) top the ratings chart |
| Shipping | Express users spend marginally more per transaction |
| Subscriptions | Only 27% subscribe; avg spend is equal to non-subscribers |
| Segments | 80% of customers are Loyal; new customer acquisition is very low |
| Age | Young Adults (18–30) are the top revenue-generating group |
| Discounts | Hat, Sneakers, Coat are most discount-dependent (~50%) |

---

## 8. Business Recommendations

### 1. Subscription Conversion Campaign
Only 27% of customers subscribe, yet avg spend is nearly identical to non-subscribers. A targeted campaign offering exclusive benefits (early sale access, free next-day delivery, birthday offers) could significantly lift subscription rates and create a more predictable revenue stream.

### 2. Loyalty Tier Program
With 3,116 Loyal customers, a tiered loyalty program (Silver / Gold / Platinum) would reward top spenders, incentivize Returning customers to purchase more frequently, and provide a structured path for New customers to engage deeper.

### 3. Discount Policy Review
Products like Hat and Sneakers are discounted ~50% of the time. A review of the discount strategy for these SKUs — using A/B testing to measure discount vs. full-price conversion rates — could protect margins without sacrificing volume.

### 4. Targeted Marketing by Demographics
- **Young Adults (18–30)**: Highest revenue group — prioritize social media campaigns, influencer partnerships, and app-based promotions.
- **Express Shipping Users**: Higher avg spend — target with premium product recommendations and early access to new launches.

### 5. Product Promotion Strategy
Feature Gloves, Sandals, and Boots — the top-rated products — prominently in homepage banners, email campaigns, and social ads. High ratings correlate with customer satisfaction and repeat purchase likelihood.

### 6. New Customer Acquisition
With only 83 New customers in the dataset, the business may be overly reliant on its existing base. Investigate referral programs, affiliate marketing, or promotional offers for first-time buyers.

---

## 9. Challenges & Learnings

| Challenge | Solution / Learning |
|---|---|
| Missing values in Review Rating | Used category-level median imputation instead of global median to preserve segment accuracy |
| Redundant columns (Discount & Promo Code) | Cross-tabulation analysis revealed near-perfect correlation; dropped redundant column to simplify the model |
| Connecting Python to PostgreSQL | Used SQLAlchemy engine with `to_sql()` for seamless DataFrame upload |
| Window functions for ranked products | Applied `RANK() OVER (PARTITION BY category ORDER BY COUNT(*) DESC)` for per-category rankings |
| Power BI data modeling | Established proper data types and relationships in Power Query before building visuals |

---

*This report was prepared as part of a data analytics portfolio project. All analysis was performed on publicly available retail transactional data.*

**— Monali Padhi | [LinkedIn](https://linkedin.com/in/monalipadhi) | [GitHub](https://github.com/padhimonali9)**
