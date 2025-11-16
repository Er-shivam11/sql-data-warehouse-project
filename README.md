📚 Overview

This repository implements a Medallion Architecture (Bronze → Silver → Gold) for building an enterprise-grade data warehouse.

This document explains:

What the Gold layer is

How the dimensional views (Dimensions & Facts) are created

Dependencies on the Silver layer

Purpose of each view

How to use the Gold models for analytics

The scripts provided generate business-ready, analytics-optimized datasets by joining, cleaning, normalizing, and enriching data in the Silver layer.

🏛️ Medallion Architecture Summary
1. Bronze Layer

Raw ingestion from operational systems

CRM tables (customers, products, sales)

ERP tables (customer info, product categories, locations)

No transformations except technical cleanup.

2. Silver Layer

Cleaned, standardized, deduped data

Trimmed names

Gender/marital normalization

Latest record selection with ROW_NUMBER()

ID alignment between CRM & ERP sources

Silver layer acts as the “clean” source for modeling.

3. Gold Layer

Final business entities for BI dashboards

Dimensions (Customers, Products)

Fact Tables (Sales)

Surrogate keys (ROW_NUMBER())

Star schema optimized for reporting tools

Inner/outer joins to supply enriched attributes

⭐ Gold Layer Views

This script creates three key views:

1️⃣ gold.dim_customers
Purpose

Customer dimension that blends CRM + ERP data to produce a complete customer profile.

Key Highlights

Creates customer_key (surrogate key)

Uses CRM as primary source for customer attributes

Falls back to ERP data (birthdate, gender if missing)

Merges location data (country)

Ensures clean, standardized, one-record-per-customer

Joins

silver.crm_cust_info → core identity

silver.erp_cust_az12 → birthdate & gender fallback

silver.erp_loc_a101 → customer location

2️⃣ gold.dim_products
Purpose

Standardized product dimension blending CRM product definitions and ERP category lookup.

Key Highlights

Surrogate product_key via ROW_NUMBER

Category + subcategory + maintenance attributes

Includes product cost, product line, and start date

Filters out historical products (WHERE prd_end_dt IS NULL)

Joins

silver.crm_prd_info → Product core

silver.erp_px_cat_g1v2 → Category enrichments

3️⃣ gold.fact_sales
Purpose

A transaction-level fact table for analytics (sales metrics, trends, product/customer behavior).

Key Highlights

Links sales to both dimensions:

product_key

customer_key

Includes sales date attributes (order, ship, due)

Includes financial metrics:

sales amount

quantity

unit price

Joins

silver.crm_sales_details (source transactions)

gold.dim_products (via product number → surrogate key)

gold.dim_customers (via customer ID → surrogate key)

📌 How to Use These Gold Views
BI/Analytics Teams Can:

Build dashboards in Power BI, Tableau, Looker, Excel

Perform customer analytics (segmentation, retention, demographics)

Product profitability/cost analysis

Sales pipeline analysis

Daily/monthly/yearly sales trends

Category performance reporting

Data Engineers / SQL Users Can:
SELECT * FROM gold.dim_customers;
SELECT * FROM gold.dim_products;
SELECT * FROM gold.fact_sales WHERE order_date >= '2023-01-01';

🧩 Dependency Order

Ensure Silver layer is built before generating Gold views:

silver.crm_cust_info

silver.crm_prd_info

silver.crm_sales_details

silver.erp_cust_az12

silver.erp_loc_a101

silver.erp_px_cat_g1v2

Then run the Gold view script.

🚀 Benefits of This Gold Layer

Fully denormalized star schema

Surrogate keys for indexing and BI compatibility

Clean, standardized business attributes

Easy for analysts to use

Efficient performance for reporting queries

Flexible for future incremental loads

📄 Files Included
File	Description
gold_views.sql	Script to create all Gold layer dimension & fact views
silver_transformations.sql	All dedupe & cleaning logic (ROW_NUMBER, trims, CASE, etc.)
bronze_raw_loading.sql	Raw ingestion setup