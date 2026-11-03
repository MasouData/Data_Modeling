# 📐 Data Modeling & Data Architecture Masterclass

Welcome to the **Data Modeling** repository! This repository provides a structured, practical, and intuitive documentation series designed to help data engineers, analysts, and architects master dimensional modeling, data warehousing fundamentals, and robust ETL/pipeline patterns.

Each module emphasizes real-world edge cases, concrete SQL examples, and clear conceptual intuition over dry academic theory.

---

## 📂 Repository Structure & Curriculum

The documentation is organized into a progressive learning path. Start from Module 1 and work your way through to master modern data architecture:

### [1. Introduction to Grain and Primary Keys](./introduction_to_grain_and_primary_keys.md)
* **What you'll learn:** The foundational concept of **Grain** (what an individual row represents), why grain dictates everything in data modeling, composite keys, and how to define primary keys correctly in SQL.

### [2. Fact and Dimension Tables](./fact_and_dimension_tables.md)
* **What you'll learn:** The building blocks of the **Star Schema**. Understand how descriptive **Dimension tables** (`Dim_Customer`, `Dim_Product`) provide context using Surrogate Keys (`_sk`), while quantitative **Fact tables** (`Fact_Sales`) capture immutable business measurements.

### [3. Slowly Changing Dimensions & CDC](./slowly_changing_dimensions_and_cdc.md)
* **What you'll learn:** How to handle attribute changes over time without breaking historical reports. Explores **SCD Type 1 (Overwrite)** vs. **SCD Type 2 (History Preservation)** and how automated **Change Data Capture (CDC)** transaction log streams feed these updates.

### [4. Advanced Fact Table Patterns](./advanced_fact_table_patterns.md)
* **What you'll learn:** Real-world pipeline resilience. Covers how to handle product returns and cancellations using **Compensating Rows (Reversals with negative numbers)** and how to accurately process **Late-Arriving Facts** without distorting historical dates.

### [5. The Snowflake Schema](./snowflake_schema.md)
* **What you'll learn:** Exploring normalized dimension hierarchies. Compares the **Snowflake Schema** with the standard **Star Schema**, weighing storage efficiency and normalization against query complexity and join performance.

---

## 💡 Core Architectural Principles

Throughout this documentation, we adhere to several golden rules of data warehousing:
1. **Grain First:** Always define the exact resolution of your data before writing a single line of SQL or creating tables.
2. **Surrogate Keys for Dimensions:** Decouple internal warehouse joins from volatile natural business keys.
3. **Immutable Fact Tables:** Fact tables are append-only event logs. Never run mutating `UPDATE` statements on historical transactional data.
4. **Star Schema Preference:** Prioritize wide, denormalized star schemas for lightning-fast BI queries and optimal columnar storage performance.

---

## 🚀 Getting Started

Clone the repository and explore the individual markdown files in order to build a rock-solid mental and practical foundation in modern data architecture!