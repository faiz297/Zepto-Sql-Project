# Zepto-Sql-Project
# 🛒 Zepto Product Analytics — SQL Project

> End-to-end SQL analysis of a Zepto grocery product catalog  
> **Language:** PostgreSQL | **Table:** `zepto` | **Domain:** E-commerce / Quick Commerce

---

## 📌 Project Overview

This project performs **data exploration, cleaning, and business analysis** on a Zepto product dataset. It covers everything from identifying null values and duplicate SKUs to computing estimated revenue, price-per-gram value, and inventory weight — giving actionable insights into product pricing, discounting, and stock management.

---

## 🗂️ Database Schema

**Table: `zepto`**

| Column | Type | Description |
|---|---|---|
| `sku_id` | SERIAL (PK) | Auto-incremented unique product ID |
| `category` | VARCHAR(120) | Product category (e.g., Dairy, Snacks) |
| `name` | VARCHAR(150) | Product name — NOT NULL |
| `mrp` | NUMERIC(8,2) | Maximum Retail Price (in ₹ after cleaning) |
| `discountPercent` | NUMERIC(5,2) | Discount percentage offered |
| `availableQuantity` | INTEGER | Units currently available in stock |
| `discountedSellingPrice` | NUMERIC(8,2) | Final selling price after discount (in ₹) |
| `weightInGms` | INTEGER | Product weight in grams |
| `outOfStock` | BOOLEAN | TRUE if product is out of stock |
| `quantity` | INTEGER | Pack quantity / units per pack |

---

## ⚙️ Setup

```sql
-- Create and use the database
CREATE TABLE zepto ( ... );  -- see sqlProject1.sql for full DDL

-- Run the full script
\i sqlProject1.sql
```

> **Requires:** PostgreSQL 12+ (uses `SERIAL`, `BOOLEAN`, `NUMERIC`, `DISTINCT`, `CASE`)

---

## 🔧 Data Pipeline

### Step 1 — Exploration
| Check | Query Goal |
|---|---|
| Row count | Verify data loaded correctly |
| Sample rows | Understand data shape |
| NULL values | Find missing/incomplete records |
| Distinct categories | Understand product taxonomy |
| Stock status | In-stock vs out-of-stock split |
| Duplicate names | Products with multiple SKUs |

### Step 2 — Cleaning
| Issue | Fix Applied |
|---|---|
| Products with MRP = 0 | Deleted from table |
| Prices stored in paise | Divided by 100.0 → converted to ₹ |

### Step 3 — Analysis
8 business queries covering pricing, discounts, inventory, and revenue.

---

## 🔍 Query Index

| # | Query | Key Technique |
|---|---|---|
| Q1 | Top 10 best-value products by discount | `ORDER BY`, `LIMIT`, `DISTINCT` |
| Q2 | High-MRP out-of-stock products | `WHERE` with boolean + threshold |
| Q3 | Estimated revenue per category | `SUM()`, `GROUP BY` |
| Q4 | High-price, low-discount products | Compound `WHERE` filter |
| Q5 | Top 5 categories by avg discount | `AVG()`, `ROUND()`, `LIMIT` |
| Q6 | Price per gram (products ≥ 100g) | Derived column, value sort |
| Q7 | Weight-based product segmentation | `CASE WHEN` categorisation |
| Q8 | Total inventory weight per category | `SUM()` with multi-column product |

---

## 📋 Query Details

### Q1 — Top 10 Best-Value Products by Discount
**Business use:** Highlight deals for homepage banners or push notifications.

```sql
SELECT DISTINCT name, mrp, discountPercent
FROM zepto
ORDER BY discountPercent DESC
LIMIT 10;
```

---

### Q2 — High-MRP Out-of-Stock Products
**Business use:** Prioritise restocking high-value items to recover lost revenue.

```sql
SELECT DISTINCT name, mrp
FROM zepto
WHERE outOfStock = TRUE AND mrp > 300
ORDER BY mrp DESC;
```

---

### Q3 — Estimated Revenue Per Category
**Business use:** Understand which categories drive the most revenue — inform buying strategy.

```sql
SELECT category,
  SUM(discountedSellingPrice * availableQuantity) AS total_revenue
FROM zepto
GROUP BY category
ORDER BY total_revenue;
```

---

### Q4 — High-Price, Low-Discount Products
**Business use:** Identify premium items with minimal promotions — candidates for discount campaigns.

```sql
SELECT DISTINCT name, mrp, discountPercent
FROM zepto
WHERE mrp > 500 AND discountPercent < 10
ORDER BY mrp DESC, discountPercent DESC;
```

---

### Q5 — Top 5 Categories by Average Discount
**Business use:** See which categories are most promotion-heavy — useful for margin analysis.

```sql
SELECT category,
  ROUND(AVG(discountPercent), 2) AS avg_discount
FROM zepto
GROUP BY category
ORDER BY avg_discount DESC
LIMIT 5;
```

---

### Q6 — Price Per Gram (Products ≥ 100g)
**Business use:** Surface best value-for-money products — useful for customer comparison tools.

```sql
SELECT DISTINCT name, weightInGms, discountedSellingPrice,
  ROUND(discountedSellingPrice / weightInGms, 2) AS price_per_gram
FROM zepto
WHERE weightInGms >= 100
ORDER BY price_per_gram;
```

---

### Q7 — Weight-Based Product Segmentation
**Business use:** Classify products by pack size for logistics planning and shelf placement.

```sql
SELECT DISTINCT name, weightInGms,
  CASE
    WHEN weightInGms < 1000 THEN 'Low'
    WHEN weightInGms < 5000 THEN 'Medium'
    ELSE 'Bulk'
  END AS weight_category
FROM zepto;
```

---

### Q8 — Total Inventory Weight Per Category
**Business use:** Estimate warehouse load and logistics cost per category.

```sql
SELECT category,
  SUM(weightInGms * availableQuantity) AS total_weight
FROM zepto
GROUP BY category
ORDER BY total_weight;
```

---

## 🧠 SQL Concepts Covered

- `CREATE TABLE` with data types: `SERIAL`, `BOOLEAN`, `NUMERIC`, `VARCHAR`, `INTEGER`
- `DELETE` for data cleaning
- `UPDATE … SET` for bulk data transformation (paise → rupees)
- `DISTINCT` to deduplicate results
- `GROUP BY` + `HAVING` for aggregation filtering
- `ORDER BY` with multi-column sorting
- `LIMIT` for top-N queries
- `CASE WHEN … THEN … END` for custom segmentation
- `ROUND()`, `AVG()`, `SUM()` aggregate functions
- Derived/computed columns (`price_per_gram`, `total_revenue`)
- NULL checking across multiple columns

---

## 📁 Repository Structure

```
zepto-sql-analytics/
├── README.md                              ← You are here
├── sqlProject1.sql                        ← Full SQL script (DDL + cleaning + analysis)
└── Zepto_SQL_Project_Documentation.docx  ← Detailed project documentation
```

---

## 💡 Key Insights (from analysis)

- Products with **MRP = 0** were removed as data quality artifacts
- Prices were stored in **paise** and converted to **rupees** during cleaning
- Weight segmentation (Low / Medium / Bulk) enables logistics and shelf planning
- Revenue estimation uses `discountedSellingPrice × availableQuantity` as a proxy

---

## 📝 Notes

- This project uses **PostgreSQL** syntax. Minor adjustments needed for MySQL (e.g., `SERIAL` → `AUTO_INCREMENT`, `BOOLEAN` → `TINYINT(1)`).
- All analysis queries use `DISTINCT` where products appear as multiple SKUs under the same name.
- Q3 and Q8 use `availableQuantity` — results reflect current stock snapshot only.
