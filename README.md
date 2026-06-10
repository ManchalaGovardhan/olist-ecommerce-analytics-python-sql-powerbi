# 🛒 Olist E-Commerce — End-to-End Data Analysis Project

An end-to-end data analytics project built on the **Olist Brazilian E-Commerce dataset**. A single Python script connects to SQL Server, extracts **9 raw tables**, cleans and compresses them into **6 analysis-ready DataFrames**, and delivers them directly into Power BI via **Get Data → Python Script**. The Power BI report then solves **10 business questions** across sales, customer behavior, product performance, pricing, and payments.



## 📁 Project Structure

```
olist-ecommerce-analysis/
│
├── cleaning_script.py        # Single script: SQL Server → clean 6 tables → Power BI
├── PowerBI_Report.pbix       # Power BI report built on the 6 output tables
└── README.md
```

---

## 🔄 How the Pipeline Works

```
SQL Server (9 tables)
        │
        ▼
Python Script  ←── Power BI: Get Data → Python Script
  • Connects to SQL Server via pyodbc
  • Extracts all 9 raw tables
  • Cleans & transforms into 6 DataFrames
        │
        ▼
Power BI picks up all 6 DataFrames automatically
        │
        ▼
10 Business Insight Reports built in Power BI
```

> Power BI runs the Python script at data refresh time. All 6 cleaned tables appear as separate queryable objects inside Power BI — no manual import steps needed.

---

## ⚙️ Setup & Requirements

### Python Dependencies

```bash
pip install pandas numpy pyodbc unidecode
```

### Additional Requirements

- **ODBC Driver 17 for SQL Server** installed on the machine running Power BI
- **Python enabled in Power BI Desktop** — go to `File → Options → Python scripting` and point to your Python installation
- SQL Server instance running with the Olist database loaded

### Running in Power BI

1. Open Power BI Desktop
2. **Get Data → Python Script**
3. Paste the full contents of `cleaning_script.py`
4. Click **OK** — Power BI executes the script and surfaces all 6 DataFrames as tables
5. Select all 6 tables and click **Load**
6. Build relationships and reports on top of the clean data

---

## 🗄️ Source Data (9 SQL Tables)

| Table | Description |
|---|---|
| `olist_customers_dataset` | Customer details and location |
| `olist_geolocation_dataset` | ZIP code to city/state mapping |
| `olist_products_dataset` | Product catalog with photo counts |
| `product_category_name_translation` | Portuguese → English category names |
| `olist_sellers_dataset` | Seller information |
| `olist_orders_dataset` | Order lifecycle timestamps and status |
| `olist_order_payments_dataset` | Payment type and value |
| `olist_order_items_dataset` | Items per order with price and freight |
| `olist_order_reviews_dataset` | Customer review scores and timestamps |

---

## 🧹 Cleaning & Transformation Logic

The script processes all 9 tables and outputs 6 clean DataFrames.

### `customers`
- Title-cased city names
- Removed accents via `unidecode`
- Stripped numerals, single quotes, and leading `...` / `*` artifacts from city names
- Dropped `customer_unique_id` (redundant)

### `geo`
- Dropped latitude and longitude (not needed for analysis)
- Applied identical city name normalization as customers

### `products`
- Dropped physical dimension and weight columns
- Merged with `product_category_name_translation` to get English category names
- Manually resolved 2 untranslated categories:
  - `pc_gamer` → `computers`
  - `portateis_cozinha_e_preparadores_de_alimentos` → `small_appliances_home_oven_and_coffee`

### `seller`
- Title-cased city names

### `orders` ← core fact table
- Parsed all 5 date/timestamp columns to `datetime`
- Dropped logically invalid rows (e.g. approval before purchase, delivery before dispatch)
- Removed canceled orders that had a delivery date
- Merged with **payments** — aggregated payment value by order and payment type, dropped `payment_sequential` and `payment_installments`
- Merged with **order items** — summed price and freight per order; dropped `order_item_id`
- Applied shipping date sanity filter (shipping limit must fall between purchase and delivery)

### `reviews`
- Filtered to only orders present in the cleaned `orders` table
- Dropped `review_comment_title`
- Converted `review_answer_timestamp` to date

---

## 📦 Output — 6 Clean DataFrames Loaded into Power BI

| DataFrame | Rows from | Key Use |
|---|---|---|
| `customers` | `olist_customers_dataset` | Customer segmentation, churn |
| `geo` | `olist_geolocation_dataset` | Geographic mapping |
| `products` | `olist_products_dataset` + translation | Category analysis, photo analysis |
| `seller` | `olist_sellers_dataset` | Seller performance |
| `orders` | orders + payments + items (3 tables merged) | All sales analysis |
| `reviews` | `olist_order_reviews_dataset` | Satisfaction analysis |

---

## 📊 Power BI Reports — 10 Business Questions Solved

### 1. 🔢 Business Overview (KPI Summary)
A high-level snapshot of the business displayed as KPI cards:
- **Total Customers** — distinct count of all customers
- **Total Orders** — total number of orders placed
- **Total Sales** — sum of all payment values
- **Average Order Value** — Total Sales ÷ Total Orders
- **Inactive Customers (Last 12 Months)** — count of customers who have not placed any order in the past 12 months

---

### 2. 🏷️ Product Category Sales Contribution
Analyzes each product category's **percentage share of overall sales**.
- Color-coded by contribution tier:
  - 🟢 **Green** — contribution > 5%
  - 🟤 **Cream** — contribution between 3%–5%
  - 🟡 **Yellow** — contribution < 3%
- Percentage values displayed with a **blue gradient** for intuitive visual comparison

---

### 3. 📉 Year-on-Year Customer Churn
Identifies **churned customers per year** — customers who purchased in a given year but placed no orders the following year.
- Tracks churned counts year by year
- Reveals periods of high attrition and customer retention trends over time

---

### 4. 📸 Product Photos vs. Quantity Sold
Investigates whether the **number of product photos** on the website correlates with **quantity sold** (each order row = 1 unit).
- Determines if products with more photos tend to sell in higher quantities
- Provides evidence-based insight into the impact of product visuals on sales performance

---

### 5. 🧺 Market Basket Analysis
Performs **association analysis** to identify products frequently purchased together in the same order.
- Uncovers the most common product pairings and combinations
- Supports **cross-selling, bundling, and targeted marketing** strategies based on real purchase behavior

---

### 6. 🚀 New Product Performance by Category
Evaluates category performance based on products **launched within the last 6 months** from the latest date in the dataset.
- Reports **Total Sales** and **Total Orders** per category for new products only
- Measures how well each category receives fresh launches

---

### 7. 📅 Product Seasonality Analysis
Identifies products with **recurring monthly sales patterns** — predictable demand cycles across the year.
- Month-by-month sales breakdown per product or category
- Informs **inventory planning, seasonal promotions, and marketing timing**

---

### 8. 💰 Price Change Impact on Sales
Calculates the **average price change over the last year** per category and correlates it with sales performance.
- Reveals whether price increases or decreases shifted customer purchasing behavior
- Guides **future pricing strategy** per category based on observed demand elasticity

---

### 9. 💳 Sales by Payment Type
Analyzes total sales grouped by **payment method** — credit card, boleto, voucher, debit card, etc.
- Shows distribution of revenue across payment types
- Identifies the most preferred payment options to inform financial planning and payment-specific promotions

---

### 10. ⭐ Review Score Analysis
Analyzes customer satisfaction through **review scores** linked to orders.




## 🛠️ Tech Stack

| Tool | Purpose |

**SQL Server (SQLEXPRESS)**  Source database 
**pyodbc** Python → SQL Server connection inside the Power BI script 
**Python — Pandas, NumPy**  Data cleaning and transformation 
**unidecode** Accent/unicode normalization for Brazilian city names 
**Power BI Desktop**  Runs the Python script at Get Data, builds all reports |

---

## 📌 Notes

- The script runs **inside Power BI's Python Script connector** at data load/refresh time — no separate script execution needed outside Power BI.
- On each refresh, Power BI re-runs the full Python script, re-connects to SQL Server, and reloads all 6 clean tables automatically.
- The `orders` table is the central fact table joining payments and order items, and is the primary source for most of the 10 insights.
- City name normalization was critical — the Brazilian dataset had accented characters, embedded numerals, and punctuation prefixes requiring careful standardization before any geographic analysis.

---

## 👤 Author

**Govardhan**
Data Analyst | SQL · Python · Power BI.Excel
