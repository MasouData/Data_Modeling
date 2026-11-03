# Module 2: Fact Tables, Dimension Tables, and Relationships

Now that we know how to define the **grain** and set up primary keys, we can assemble the building blocks of dimensional modeling: **Fact Tables** and **Dimension Tables**. 

In a data warehouse, we organize data into a **Star Schema** to make queries lightning-fast and business-friendly.

---

## 1. Dimension Tables (The Context)

Dimensions answer the *Who, What, Where, When, and Why* of a business event. They contain descriptive attributes that provide context to our numbers.

* **Characteristics:** Wide tables with many text columns, relatively small row counts, and changes tracked via **Slowly Changing Dimensions (SCD)**.
* **Primary Key:** Uses an artificial **Surrogate Key (SK)** (e.g., `customer_sk`) rather than a natural business key.

### Example: `Dim_Customer`
| customer_sk (PK) | customer_email | name | city |
| :--- | :--- | :--- | :--- |
| 101 | alice@company.com | Alice Smith | New York |
| 102 | bob@company.com | Bob Jones | Chicago |

---

## 2. Fact Tables (The Measurements)

Fact tables capture the measurable, quantitative results of a business process (the *How Much*). 

* **Characteristics:** Narrow tables with massive row counts (billions of rows) containing numeric metrics (`quantity`, `sales_amount`, `discount`) and Foreign Keys (FKs) pointing to dimensions.
* **Design Rule:** Fact tables are **append-only**. We do not update past history in place; instead, corrections are handled via compensating rows or late-arriving fact insertions.

### Example: `Fact_Sales`
| sale_date | product_sk | customer_sk | quantity | sales_amount |
| :--- | :--- | :--- | :--- | :--- |
| 2026-06-01 | 501 | 101 | 2 | \$40.00 |

---

## 3. Table Relationships: The Star Schema

How do these tables talk to each other? 

1. **Fact-to-Dimension Relationships:** A fact table connects to multiple dimension tables using Foreign Keys (`customer_sk`, `product_sk`, etc.). This forms a **Star Schema**, where the Fact table sits in the center like a star, surrounded by its dimension tables as points.
2. **Fact-to-Fact Relationships (The Anti-Pattern):** **Never join a fact table directly to another fact table.** If you need to analyze sales data alongside inventory data, do not link `Fact_Sales` directly to `Fact_Inventory`. Instead, connect both facts through **Conformed Dimensions** (like `Dim_Date` or `Dim_Product`) that both tables share.

### SQL Join Example

Here is how we query our star schema using surrogate keys to get meaningful context:

```sql
SELECT 
    f.sale_date,
    c.name AS customer_name,
    c.city,
    f.quantity,
    f.sales_amount
FROM Fact_Sales f
JOIN Dim_Customer c ON f.customer_sk = c.customer_sk;
```

---

## Up Next

In the next module, we will explore **Slowly Changing Dimensions (SCD Types 1 & 2)** and how automated **Change Data Capture (CDC)** feeds stream updates into these dimensions without breaking historical integrity.