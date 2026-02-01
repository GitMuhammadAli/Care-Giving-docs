# 📊 Data Warehousing - Complete Guide

> A comprehensive guide to data warehousing - ETL/ELT, OLAP vs OLTP, data lakes, dimensional modeling, and building analytics infrastructure.

---

## 🧠 MUST REMEMBER TO IMPRESS

### 1-Liner Definition
> "A data warehouse is a centralized repository optimized for analytical queries (OLAP), consolidating data from multiple sources through ETL/ELT pipelines for reporting, BI, and data science workloads."

### Key Terms
| Term | Meaning |
|------|---------|
| **OLTP** | Online Transaction Processing (your app's database) |
| **OLAP** | Online Analytical Processing (warehouse for analytics) |
| **ETL** | Extract, Transform, Load (transform before loading) |
| **ELT** | Extract, Load, Transform (transform in warehouse) |
| **Star schema** | Fact table + dimension tables |
| **Data lake** | Raw data storage (files in S3) |

---

## Core Concepts

```
OLTP vs OLAP:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  OLTP (PostgreSQL, MySQL)        OLAP (Snowflake, BigQuery)    │
│  ─────────────────────           ──────────────────────────    │
│  • Operational queries           • Analytical queries           │
│  • INSERT, UPDATE, DELETE        • SELECT (mostly)              │
│  • Normalized schema             • Denormalized/Star schema     │
│  • Row-oriented storage          • Column-oriented storage      │
│  • Millisecond response          • Seconds to minutes           │
│  • Current state                 • Historical data              │
│                                                                  │
│  EXAMPLE QUERIES:                                               │
│  OLTP: Get order #12345                                        │
│  OLAP: Total revenue by product category for Q3 2024           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Star Schema

```
STAR SCHEMA (Dimensional Modeling):
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│                    ┌──────────────┐                             │
│                    │ dim_product  │                             │
│                    │ ────────────│                             │
│                    │ product_id   │                             │
│                    │ name         │                             │
│                    │ category     │                             │
│                    └──────┬───────┘                             │
│                           │                                     │
│  ┌──────────────┐   ┌─────┴─────┐   ┌──────────────┐          │
│  │ dim_customer │   │fact_sales │   │ dim_date     │          │
│  │ ────────────│◄──│ ─────────│──►│ ──────────── │          │
│  │ customer_id  │   │ sale_id   │   │ date_id      │          │
│  │ name         │   │ date_id   │   │ date         │          │
│  │ segment      │   │customer_id│   │ month        │          │
│  └──────────────┘   │ product_id│   │ quarter      │          │
│                     │ quantity  │   │ year         │          │
│                     │ amount    │   └──────────────┘          │
│                     └───────────┘                              │
│                                                                  │
│  Query: SELECT SUM(amount), category, year                     │
│         FROM fact_sales f                                      │
│         JOIN dim_product p ON f.product_id = p.product_id     │
│         JOIN dim_date d ON f.date_id = d.date_id              │
│         GROUP BY category, year                                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### ETL vs ELT

```
ETL (Traditional):
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  Sources → EXTRACT → TRANSFORM → LOAD → Warehouse              │
│                        ↑                                        │
│            ETL server does heavy lifting                       │
│                                                                  │
│  + Transform logic in one place                                │
│  - Bottleneck at ETL server                                    │
│  - Schema must be defined upfront                              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

ELT (Modern):
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  Sources → EXTRACT → LOAD → TRANSFORM in Warehouse             │
│                              ↑                                  │
│               Warehouse does transforms (dbt)                  │
│                                                                  │
│  + Leverage warehouse compute power                            │
│  + Keep raw data (can re-transform)                            │
│  + Schema-on-read flexibility                                  │
│  - Requires powerful warehouse (BigQuery, Snowflake)           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Interview Questions

**Q: "What's the difference between OLTP and OLAP?"**
> "OLTP is your application database - optimized for transactions, row-oriented, normalized. OLAP is your analytics warehouse - optimized for aggregations, column-oriented, denormalized. Don't run analytics on OLTP - it'll impact your app's performance."

**Q: "ETL vs ELT?"**
> "ETL transforms data before loading (traditional, good when warehouse is limited). ELT loads raw data then transforms in warehouse (modern, leverages warehouse compute). ELT is preferred with cloud warehouses like Snowflake/BigQuery."

---

## Quick Reference

```
DATA WAREHOUSING CHEAT SHEET:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  CONCEPTS:                                                      │
│  • OLTP: App DB (row-oriented, normalized)                     │
│  • OLAP: Analytics (column-oriented, star schema)              │
│  • Data Lake: Raw files in S3                                  │
│  • Data Warehouse: Structured analytics                        │
│                                                                  │
│  TOOLS:                                                         │
│  • Warehouses: Snowflake, BigQuery, Redshift                   │
│  • ETL: Airflow, Fivetran, Airbyte                             │
│  • Transform: dbt                                              │
│  • BI: Looker, Metabase, Tableau                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```
