# Module 3: Slowly Changing Dimensions (SCD) and Change Data Capture (CDC)

In Module 2, we built our Star Schema using **Surrogate Keys (SK)**. But business entities change over time: customers move to new cities, products change categories, and employees get promoted. 

How do we handle these changes in our dimension tables without ruining our historical sales reports? Welcome to the world of **Slowly Changing Dimensions (SCD)** and **Change Data Capture (CDC)**.

---

## 1. The Core Dilemma: The Relocation Problem

Imagine Alice. When she first signs up, she lives in **New York**. She buys \$40 worth of goods. Later, she moves to **Seattle**. 

If we just overwrite her city in the database from *New York* to *Seattle*, what happens when management runs a historical sales report breaking down revenue by city? 
* **The Distortion:** The report will retroactively show that Alice made her past New York purchases while living in Seattle! History has been rewritten incorrectly.

To solve this, data architects use different **SCD Types**.

---

## 2. SCD Type 1 vs. SCD Type 2

### SCD Type 1: Overwrite (No History)
* **What it does:** Updates the existing record in place. The old value is permanently erased and replaced by the new value.
* **When to use it:** For corrections of typos (e.g., correcting a misspelled name like "Alise" to "Alice") or attributes where history doesn't matter.
* **Intuition:** Like using a white-out pen on a paper form. The old entry is gone forever.

### SCD Type 2: Add New Row (Preserve History)
* **What it does:** Instead of overwriting, we **expire** the old row by setting an end date, and **insert a brand-new row** with a fresh surrogate key (`customer_sk`), new attributes, and a start date.
* **When to use it:** For critical business attributes like home addresses, department changes, or product pricing tiers where past transactions must tie to the context of *that specific point in time*.
* **Intuition:** Like a passport history. When you renew or change your details, you don't erase your old passport; you keep it as an archive and get a new one with a new ID number.

### Example: SCD Type 2 in Action (`Dim_Customer`)

| customer_sk | customer_email | name | city | effective_date | expiration_date | is_current |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **101** | alice@company.com | Alice | New York | 2025-01-01 | 2026-05-31 | `FALSE` (Expired) |
| **105** | alice@company.com | Alice | Seattle | 2026-06-01 | 9999-12-31 | `TRUE` (Active) |

* **Old Sales (Before June 1):** Point to `customer_sk = 101` (New York).
* **New Sales (After June 1):** Point to `customer_sk = 105` (Seattle).
* **Result:** Historical integrity is 100% preserved! 🎯

---

## 3. Feeding the Pipeline: Change Data Capture (CDC)

How do these changes travel automatically from the operational app database to our data warehouse? Through **Change Data Capture (CDC)**.

Instead of scanning entire tables every night (which is slow and expensive), CDC reads the operational database's **transaction logs** (the low-level ledger where the database records every `INSERT`, `UPDATE`, and `DELETE` as it happens).

### Intuition: The Security Camera
Think of CDC like a security camera watching the database door. Instead of counting every item in the store at the end of the day to see what changed, the security camera records every single entry and exit in real-time.

### Example: CDC Event Stream Log
When Alice moves to Seattle and a new user (Charlie) signs up, the CDC log captures these micro-events:

| operation | customer_email | name | new_city | timestamp |
| :--- | :--- | :--- | :--- | :--- |
| `UPDATE` | alice@company.com | Alice | Seattle | 2026-06-01 08:00:00 |
| `INSERT` | charlie@company.com | Charlie | Boston | 2026-06-01 08:05:00 |

### How CDC Drives SCD Logic:
1. The pipeline reads the CDC stream.
2. It sees an `UPDATE` event for `alice@company.com`.
3. It triggers **SCD Type 2 logic**: finds her active row, sets `is_current = FALSE`, and inserts the new Seattle row with a brand-new surrogate key.
4. It sees an `INSERT` event for `charlie@company.com` and inserts a brand-new dimension record.

---

## Up Next

In the next module, we will explore **Advanced Fact Table Patterns**, covering how to handle edge cases like compensating returns and late-arriving data packets without breaking your historical pipelines.