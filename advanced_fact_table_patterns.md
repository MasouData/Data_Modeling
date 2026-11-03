# Module 4: Advanced Fact Table Patterns (Returns & Late Arrivals)

In Module 2, I established a golden rule of data architecture: **Fact tables are append-only (immutable event logs).** I do not run mutating `UPDATE` statements on billions of historical rows when real-world changes happen.

So, how do professional data engineers handle real-world messy scenarios like product returns, order cancellations, and delayed data packets without breaking historical integrity? 

Let's explore two essential advanced patterns.

## 1. Compensating Rows (Reversals for Returns)

Imagine a customer buys 2 units of a product for \$40.00 on June 1st. Two days later, they return both items. 

If I was to update or delete the original row in the fact table, I would destroy our audit trail and falsify total daily sales volume for June 1st. Instead, I insert a **compensating row** (a reversal).

### Intuition: The Ledger Book
Think of a traditional accountant's ledger. Accountants never erase a mistaken entry with white-out. If a mistake or return happens, they write a second balancing entry with negative numbers so the net total adjusts correctly while preserving the complete audit history.

### Example: Fact Sales Table with a Return

| sale_date | product_sk | customer_sk | quantity | sales_amount | event_type | 
| ----- | ----- | ----- | ----- | ----- | ----- | 
| 2026-06-01 | 501 | 101 | 2 | \$40.00 | `SALE` | 
| 2026-06-03 | 501 | 101 | -2 | -\$40.00 | `RETURN` | 

* **The Result:** If you sum the `quantity` column for Product 501, the net quantity correctly becomes `0`. Yet, your system retains a crystal-clear record of *both* the original sale and the subsequent return.

## 2. Late-Arriving Facts

In an ideal world, every transaction is loaded into the data warehouse the exact second it occurs. In reality, systems go offline, trucks drive through dead zones, and offline retail scanners batch-upload data days later. 

If a sale happened on June 1st, but the data packet doesn't arrive at your data warehouse until June 3rd, how do you load it?

* **The Rookie Mistake:** Stamping the record with the arrival date (June 3rd) because "that's when it entered the warehouse." This ruins historical sales reports for June 1st.
* **The Professional Approach:** Always insert the fact record using its **original transaction timestamp** (`sale_date = 2026-06-01`), regardless of when the pipeline actually processed it.

### Example: Handling Late Arrivals

| sale_date (Business Date) | product_sk | quantity | sales_amount | loaded_at (Audit Timestamp) | 
| ----- | ----- | ----- | ----- | ----- | 
| 2026-06-01 | 502 | 1 | \$15.00 | 2026-06-01 08:30:00 | 
| **2026-06-01** | **503** | **1** | **\$25.00** | **2026-06-03 14:15:00** | 

*Notice the second row:* Even though it was loaded into the warehouse two days late (`loaded_at` is June 3rd), its `sale_date` remains June 1st. When executives run a report for June 1st sales next week, that revenue is accurately accounted for.

## Summary Checklist for Fact Tables

1. **Immutable:** Append new rows; never update or delete historical fact records in place.
2. **Surrogate Keys:** Always join facts to dimensions using surrogate keys (`customer_sk`, `product_sk`), never natural business keys.
3. **Compensate, Don't Mutate:** Use negative values in new rows to handle returns and refunds.
4. **Preserve Original Time:** Always anchor fact tables to the business event date (`sale_date`), not the pipeline processing date.

