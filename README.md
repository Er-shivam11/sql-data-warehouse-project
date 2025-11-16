# 📚 Overview

This repository implements a **Medallion Architecture (Bronze → Silver → Gold)** for building an enterprise-grade data warehouse.

This document explains:

- What the **Gold layer** is  
- How the **Dimensional Views** (Dimensions & Facts) are created  
- Dependencies on the **Silver Layer**  
- Purpose of each view  
- How to use the Gold models for analytics  

The scripts provided generate **business-ready, analytics-optimized datasets** by joining, cleaning, normalizing, and enriching data in the Silver layer.

---

# 🏛️ Medallion Architecture Summary

## 1. 🥉 Bronze Layer
- Raw ingestion from operational systems  
- CRM tables (customers, products, sales)  
- ERP tables (customer info, product categories, locations)  
- No transformations except technical cleanup  

## 2. 🥈 Silver Layer
- Cleaned, standardized, deduped data  
- Trimmed names  
- Gender/marital normalization  
- Latest record selected using `ROW_NUMBER()`  
- ID alignment between CRM & ERP sources  
- Acts as the **clean source** for modeling  

## 3. 🥇 Gold Layer
- Final business entities for BI dashboards  
- **Dimensions:** Customers, Products  
- **Fact Tables:** Sales  
- Surrogate keys using `ROW_NUMBER()`  
- Star-schema optimized for reporting  
- Inner/outer joins for enriched attributes  

---

# ⭐ Gold Layer Views

This script creates three key Gold Views:

---

## 1️⃣ `gold.dim_customers`

### **Purpose**
Customer dimension that blends CRM + ERP data into a complete customer profile.

### **Key Highlights**
- Creates `customer_key` (surrogate key)  
- CRM as primary source  
- ERP fallback for missing attributes (birthdate, gender)  
- Location enrichment (country)  
- Ensures clean, standardized, one-record-per-customer  

### **Joins**
- `silver.crm_cust_info` → core identity  
- `silver.erp_cust_az12` → birthdate & gender fallback  
- `silver.erp_loc_a101` → customer location  

---

## 2️⃣ `gold.dim_products`

### **Purpose**
Standardized product dimension combining CRM product definitions and ERP category data.

### **Key Highlights**
- Surrogate `product_key` using `ROW_NUMBER()`  
- Includes category, subcategory, maintenance attributes  
- Includes product cost, product line, start date  
- Filters out inactive products (`WHERE prd_end_dt IS NULL`)  

### **Joins**
- `silver.crm_prd_info` → product core  
- `silver.erp_px_cat_g1v2` → category enrichment  

---

## 3️⃣ `gold.fact_sales`

### **Purpose**
A transaction-level fact table used for sales analytics.

### **Key Highlights**
- Links transactions to:
  - `product_key`
  - `customer_key`
- Includes all important sales dates  
- Includes financial metrics:
  - sales amount  
  - quantity  
  - unit price  

### **Joins**
- `silver.crm_sales_details`  
- `gold.dim_products`  
- `gold.dim_customers`  

---

# 📌 How to Use These Gold Views

## **BI / Analytics Teams Can**
- Build dashboards in Power BI, Tableau, Looker, Excel  
- Perform customer analytics (segmentation, retention, demographics)  
- Analyze product performance & profitability  
- Study sales trends (daily/monthly/yearly)  
- Report category performance  

## **Data Engineers / SQL Users**
```sql
SELECT * FROM gold.dim_customers;

SELECT * FROM gold.dim_products;

SELECT * 
FROM gold.fact_sales 
WHERE order_date >= '2023-01-01';
```

---

# 🧩 Dependency Order

Ensure the following Silver layer tables exist before generating Gold views:

```
silver.crm_cust_info
silver.crm_prd_info
silver.crm_sales_details
silver.erp_cust_az12
silver.erp_loc_a101
silver.erp_px_cat_g1v2
```

Then run:

```
ddl_gold.sql
```

---

# 🚀 Benefits of This Gold Layer

- Fully denormalized **Star Schema**  
- Surrogate keys for indexing + BI tools  
- Clean, standardized business attributes  
- Easy to understand for analysts  
- Optimized for reporting performance  
- Future-friendly for incremental loads  

---

# 📄 Files Included

| File | Description |
|------|-------------|
| `ddl_gold.sql` | Script to create all Gold layer Dim & Fact views |
| `proc_load__silver.sql` | All dedupe & cleaning logic |
| `proc_load_bronze.sql` | Raw ingestion setup |

