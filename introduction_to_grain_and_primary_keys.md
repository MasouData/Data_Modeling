# Module 1: Grain and Primary Keys

Welcome to the foundation of dimensional modeling! Before writing a single line of SQL or designing massive tables, we must answer one fundamental question: **What does a single row in this table represent?**

---

## 1. What is Grain?

The **grain** is the precise, unambiguous definition of what an individual row in a table represents. It is the single most important concept in data modeling because it dictates how data is measured, aggregated, and joined.

* **Intuition:** Think of grain as the resolution of a photograph. A low-resolution photo (coarse grain) shows monthly totals. A high-resolution photo (fine grain) shows every single transaction down to the exact second and item.
* **Why it matters:** If you do not know the grain, you will inevitably double-count metrics, join tables incorrectly, or build models that fail to answer business questions.

### Example: Fact Table Grain (Line-Item Level)
In a retail sales system, a coarse grain might be "one row per order." However, a more powerful analytical grain is **one row per line-item** (each specific product purchased within an order).

| order_id | product_id | quantity | sales_amount |
| :--- | :--- | :--- | :--- |
| 101 | 501 | 2 | $20.00 |
| 101 | 502 | 1 | $15.00 |

Here, order `101` has two rows because two different products were purchased. The grain is explicitly: **One row per product sold per order**.

---

## 2. Types of Grain

Grain isn't limited to transactions. Depending on your business process, you will encounter different types of grains:

1. **Transaction Grain:** Captures discrete events as they happen (e.g., an individual clickstream event, a POS line-item scan).
2. **Periodic Snapshot Grain:** Summarizes activity over a fixed interval (e.g., total monthly inventory count per warehouse).
3. **Accumulating Snapshot Grain:** Tracks the entire lifecycle of a workflow from start to finish (e.g., an order moving from *Placed* $\rightarrow$ *Shipped* $\rightarrow$ *Delivered*).

---

## 3. Defining the Primary Key (PK) from Grain

**Grain always comes first.** Once you define what a row represents, the Primary Key naturally follows to enforce uniqueness at that exact level of detail.

When a single column is not enough to guarantee uniqueness (such as our line-item example where `order_id` repeats across different products), we use a **Composite Primary Key** combining two or more columns.

### SQL Implementation Example

Here is how you define a composite primary key in SQL for a daily inventory fact table (where the grain is *one row per daily inventory count for a specific warehouse*):

```sql
CREATE TABLE inventory_fact (
    inventory_date DATE,
    warehouse_id INT,
    quantity_on_hand INT,
    PRIMARY KEY (inventory_date, warehouse_id)
);
```

* **`inventory_date` + `warehouse_id`:** Together, they act as a composite key, ensuring that a specific warehouse only has **one** entry per calendar day.

---

## Up Next
In the next module, we will explore **Surrogate Keys** and look at why decoupling our internal data warehouse keys from natural business keys protects historical data integrity.