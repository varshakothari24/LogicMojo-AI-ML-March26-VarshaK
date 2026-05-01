# End-to-End E-commerce Intelligence System: Building a Customer 360 Analytics Framework

## Project Report

---

## 1. Project Title

**End-to-End E-commerce Intelligence System: Building a Customer 360 Analytics Framework**

---

## 2. Business Problem Statement

In modern e-commerce ecosystems, data is generated across multiple independent systems — customer management, order processing, payments, product catalogs, seller networks, and customer feedback platforms. These datasets are typically fragmented and stored in separate tables, making it difficult to extract unified insights.

The objective of this project is to simulate a real-world data analytics scenario where we:

- Integrate multiple data sources into a unified analytical dataset
- Construct a Customer 360 View
- Analyze customer behavior, revenue patterns, and operational performance
- Identify key drivers of business growth and customer satisfaction
- Generate actionable, data-driven business recommendations

This project reflects how data analysts and data scientists work in real organizations, where raw data must be transformed into meaningful insights before applying machine learning models.

---

## 3. Dataset Description

The project uses a multi-table e-commerce dataset consisting of 9 relational CSV files:

### Core Tables

| Table | Rows | Columns | Description |
|-------|------|---------|-------------|
| `customers.csv` | ~99,441 | 5 | Customer demographic and location information (customer_id, unique_id, zip_code, city, state) |
| `orders.csv` | ~99,441 | 8 | Central fact table with order lifecycle details (order_id, status, purchase/approval/delivery timestamps) |
| `order_item.csv` | ~112,650 | 7 | Product-level details for each order (order_id, product_id, seller_id, price, freight_value) |
| `payments.csv` | ~103,886 | 5 | Payment information including type, installments, and value |
| `reviews.csv` | ~104,719 | 7 | Customer feedback with review scores (1-5), comment titles, and messages |

### Supporting Tables

| Table | Rows | Columns | Description |
|-------|------|---------|-------------|
| `products.csv` | ~32,951 | 9 | Product details including category, dimensions, weight, and photo count |
| `sellers.csv` | ~3,095 | 4 | Seller-level information (seller_id, zip_code, city, state) |
| `location.csv` | ~1,000,163 | 5 | Geographic coordinates mapped to zip codes |
| `category_translation.csv` | 70 | 2 | Mapping of Portuguese product category names to English |

### Key Relationships

The `orders` table serves as the central fact table connecting most datasets through the following keys:

- `customer_id` — links orders to customers
- `order_id` — links orders to order_items, payments, and reviews
- `product_id` — links order_items to products
- `seller_id` — links order_items to sellers
- `product_category_name` — links products to category_translation

---

## 4. Methodology

The project follows a structured, step-by-step analytical pipeline:

### Step 1: Data Loading and Initial Exploration
- Loaded all 9 datasets using Pandas
- Inspected structure using `.head()`, `.info()`, `.describe()`
- Identified primary and foreign keys across all tables
- Validated key uniqueness (e.g., customer_id, order_id, product_id)

### Step 2: Data Cleaning and Preprocessing
- **Missing Values:**
  - Review comment titles and messages (optional fields) — filled with empty strings
  - Product category names — filled with 'unknown'
  - Product numeric dimensions — filled with column median values
  - Order delivery dates (NaN for undelivered orders) — retained as-is (expected behavior)
- **Duplicates:** Removed duplicate rows from the geolocation table (retained one entry per zip code)
- **Date Conversion:** Converted all timestamp columns in orders, order_items, and reviews to `datetime` format
- **Validation:** Verified data types, value ranges for prices, freight, review scores, and order statuses

### Step 3: Data Integration
Constructed a unified **Master Dataset** by sequentially merging tables:

1. `orders` + `customers` (on `customer_id`)
2. Result + `order_items` (on `order_id`)
3. Result + `products` (on `product_id`)
4. Result + `category_translation` (on `product_category_name`)
5. Result + `sellers` (on `seller_id`)
6. Result + `payments` — aggregated per order (total value, max installments, primary type) to avoid row explosion
7. Result + `reviews` — deduplicated per order (latest review retained)

**Final Master Dataset: ~112,650 rows × 35+ columns**

### Step 4: Feature Engineering
Created meaningful derived features at two levels:

**Order-Level Features:**
- `order_item_total` — price + freight per item
- `delivery_time_days` — days from purchase to delivery
- `delivery_delay_days` — estimated vs actual delivery difference (positive = early)
- `items_per_order` — count of items per order
- `order_month`, `order_year`, `order_day_of_week`, `order_hour` — temporal features

**Customer-Level Features:**
- `purchase_count` — number of unique orders
- `total_spend` / `clv` — customer lifetime value (total spend)
- `avg_order_value` — average payment per order
- `avg_review_score` — mean review score given
- `avg_delivery_time` — mean delivery time experienced
- `tenure_days` — days between first and last purchase
- `is_repeat_customer` — binary flag (>1 order)
- `value_segment` — quartile-based segmentation (Low / Mid-Low / Mid-High / High Value)

### Step 5–6: Exploratory Data Analysis and Visualization
Performed structured analysis with supporting visualizations (Matplotlib & Seaborn) across five dimensions:

- **Customer Analysis** — retention, value segmentation, geographic distribution
- **Revenue & Order Analysis** — monthly trends, peak periods, payment types
- **Product Analysis** — top categories, price distributions, demand patterns
- **Seller Analysis** — performance ranking, revenue concentration (Pareto), geographic spread
- **Review & Satisfaction Analysis** — score distribution, delivery-rating correlation, dissatisfaction patterns

**Visualization types used:** Time series plots, bar charts, pie charts, histograms, box plots, heatmaps, scatter plots, Pareto curves

---

## 5. Key Findings

### Customer Insights
- The vast majority of customers are **one-time buyers** — the repeat customer rate is extremely low (approximately 3%)
- Repeat customers spend significantly more than one-time customers (roughly 2x higher average total spend)
- **São Paulo (SP)** dominates both customer count and revenue, followed by Rio de Janeiro (RJ) and Minas Gerais (MG)
- The top 25% of customers (High Value segment) contribute a disproportionately large share of total revenue

### Revenue & Order Insights
- Revenue shows a **clear upward trend** from late 2016 through mid-2018, with occasional seasonal peaks
- **Weekday afternoons/evenings** are peak shopping times; weekends see lower order volumes
- **Credit cards** dominate payment methods (~74% of all transactions), followed by boleto (bank slip)
- Average order value remains relatively stable over time

### Product Insights
- **Bed/Bath/Table, Health/Beauty, Sports/Leisure, Furniture/Decor, and Computers/Accessories** are consistently the top-selling categories by both volume and revenue
- Product price distribution is heavily right-skewed — most products are under R$200, with a long tail of expensive items
- Categories with the highest average prices (e.g., computers, furniture) don't necessarily have the highest sales volume

### Seller Insights
- Revenue is highly concentrated: **the top 20% of sellers generate approximately 80% of total revenue** (classic Pareto distribution)
- Most sellers are concentrated in São Paulo, reflecting the geographic skew of the marketplace
- A long tail of sellers contribute very little to overall revenue

### Satisfaction Insights
- Review score distribution is **bimodal** — heavily skewed toward 5-star ratings, with a secondary peak at 1 star
- **Delivery time is the strongest driver of dissatisfaction**: orders rated 1-2 stars have significantly longer delivery times and much higher late-delivery rates than orders rated 4-5 stars
- There is a **negative correlation** between delivery time and review score
- Late deliveries receive average scores of ~2.5 vs ~4.3 for on-time/early deliveries

---

## 6. Business Insights

1. **Retention is the Critical Gap:** With ~97% of customers making only one purchase, the platform is heavily dependent on continuous acquisition. Even a modest improvement in retention (e.g., converting 5% of one-time buyers to repeat) would yield substantial revenue gains given that repeat customers spend 2x more.

2. **Delivery Performance Directly Drives Revenue Quality:** The strong negative correlation between delivery time and review scores means that logistics inefficiency doesn't just hurt satisfaction — it damages the platform's reputation and reduces the likelihood of repeat purchases.

3. **Geographic Concentration is Both a Strength and a Risk:** São Paulo's dominance means that the platform's revenue is heavily tied to one region. Expansion into underserved states represents both a growth opportunity and a risk mitigation strategy.

4. **Seller Ecosystem Health is Uneven:** The Pareto-like revenue concentration among sellers means that the platform's revenue is vulnerable to the loss of a small number of top sellers. Meanwhile, a large number of low-performing sellers may be diluting overall quality.

5. **Freight Costs are a Hidden Barrier:** With median freight costs representing a significant percentage of product price, shipping expenses may be suppressing conversion rates, especially for lower-priced items.

6. **Credit Card Installment Culture Drives Sales:** The prevalence of installment-based credit card payments indicates that flexible payment options are key to conversion in this market.

---

## 7. Recommendations

### 1. Implement a Customer Retention Program
- Launch a **loyalty rewards program** with points-based incentives for repeat purchases
- Deploy **personalized re-engagement email campaigns** targeting customers 30/60/90 days after their first purchase
- Offer **first-repeat-purchase discounts** to convert one-time buyers

### 2. Optimize Delivery Operations
- Establish **regional fulfillment centers** or partner with local carriers in high-volume states outside São Paulo
- Implement **proactive delivery tracking notifications** to manage customer expectations
- Set **realistic delivery estimates** — customers tolerate longer timelines better than broken promises

### 3. Expand Geographic Reach
- Launch **state-specific marketing campaigns** for high-potential, low-penetration markets
- **Onboard sellers in underserved regions** to reduce delivery distances and costs for customers in those areas
- Consider **localized promotions** and category curation based on regional preferences

### 4. Address Freight Cost Barriers
- Introduce **free-shipping thresholds** (e.g., "Free shipping on orders over R$100") to increase average order value while removing a purchase barrier
- Explore **bundled shipping discounts** for multi-item orders
- Subsidize freight for high-margin categories where shipping costs are disproportionately high

### 5. Strengthen Seller Quality
- Develop a **seller certification and training program** to elevate mid-tier sellers
- Implement **quality scorecards** with metrics on delivery speed, return rates, and review scores
- **Flag and review underperforming sellers** who consistently receive low ratings

### 6. Category-Specific Strategies
- **Double down on top categories** (Bed/Bath, Health/Beauty, Sports) with targeted promotions and improved inventory
- For high-price categories (Computers, Furniture), focus on **trust-building measures** such as extended warranties and verified seller badges
- Address **category-specific dissatisfaction** by analyzing low-rated orders in each category and implementing corrective actions

### 7. Optimize Marketing Timing
- Schedule **promotional campaigns and flash sales** during peak activity windows (weekday afternoons, 10 AM – 4 PM)
- Reduce marketing spend during low-activity periods (late night, early morning, weekends)
- Align **email and push notification schedules** with peak browsing hours

---

*Report prepared as part of the End-to-End E-commerce Intelligence System project.*
