# Introduction to Grain, Primary Keys, and Database Normalization

Data modeling starts with defining the **grain** of your data and structuring your tables cleanly using keys and normalization. This document covers grain, key types, and the complete spectrum of normalization forms from 1NF to 4NF using a visual, step-by-step example.

---

## 1. Grain and Keys: The Building Blocks

### What is Grain?
The **grain** defines what a single row in a table represents. 
* *Example:* In an `OrderItems` table, the grain is **one row per product per order**.

### Understanding Key Types
* **Primary Key (PK):** A column (or set of columns) chosen to uniquely identify every row.
* **Candidate Key:** Any column (or combination) that *could* uniquely identify a row.
* **Super-Key:** **Any** combination of columns that uniquely identifies a row (can contain redundant columns). Every primary key is a super-key, but not vice versa.
* **Surrogate Key:** An artificial, system-generated ID used as a primary key instead of a messy business key.
* **Foreign Key (FK):** A column referencing the primary key of another table, acting as the "glue" that reconnects split tables.

---

## 2. Visual Walkthrough: The Order Management System Example

Let’s trace a single online store order through each normalization layer using data tables to see how redundancy is eliminated.

### Stage 0: The Unnormalized Spreadsheet (Before 1NF)
*The messy, NoSQL/JSON-like starting point where multiple items and details are jammed into single cells.*

| OrderID | CustomerName | CustomerCity | Products & Quantities (Jammed) |
| :--- | :--- | :--- | :--- |
| **101** | Alice Smith | New York | Laptop (Qty: 1, $1200), Mouse (Qty: 2, $25) |
| **102** | Bob Jones | Chicago | Laptop (Qty: 1, $1200) |

---

### Stage 1: First Normal Form (1NF)
**Rule:** Atomic values in every cell (no lists, arrays, or comma-separated bundles).
* *Action:* Break multi-valued items into individual rows.

| OrderID | CustomerName | CustomerCity | ProductID | ProductName | Price | Quantity |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 101 | Alice Smith | New York | P1 | Laptop | $1200 | 1 |
| 101 | Alice Smith | New York | P2 | Mouse | $25 | 2 |
| 102 | Bob Jones | Chicago | P1 | Laptop | $1200 | 1 |

* **Primary Key:** Composite key `(OrderID, ProductID)`

---

### Stage 2: Second Normal Form (2NF)
**Rule:** Must be in 1NF + **No partial dependencies** (non-key attributes must depend on the *entire* primary key).
* *Action:* Split tables to remove partial key dependencies. This naturally exposes the **Many-to-Many ($N:N$) relationship** via a junction table.

#### 1. `Orders` Table (Depends only on OrderID)
| OrderID | CustomerName | CustomerCity |
| :--- | :--- | :--- |
| 101 | Alice Smith | New York |
| 102 | Bob Jones | Chicago |

#### 2. `Products` Table (Depends only on ProductID)
| ProductID | ProductName | Price |
| :--- | :--- | :--- |
| P1 | Laptop | $1200 |
| P2 | Mouse | $25 |

#### 3. `OrderItems` Table (Junction Table - Depends on both keys)
| OrderID | ProductID | Quantity |
| :--- | :--- | :--- |
| 101 | P1 | 1 |
| 101 | P2 | 2 |
| 102 | P1 | 1 |

---

### Stage 3: Third Normal Form (3NF)
**Rule:** Must be in 2NF + **No transitive dependencies** (non-key attributes cannot depend on other non-key attributes).
* *Action:* In the 2NF `Orders` table, `CustomerCity` depended on `CustomerName` (a non-key attribute). We split customers into their own table.

#### 1. `Customers` Table (New Table)
| CustomerID | CustomerName | CustomerCity |
| :--- | :--- | :--- |
| C1 | Alice Smith | New York |
| C2 | Bob Jones | Chicago |

#### 2. `Orders` Table (Cleaned up with Foreign Key)
| OrderID | CustomerID (FK) |
| :--- | :--- |
| 101 | C1 |
| 102 | C2 |

#### 3. `Products` & `OrderItems` *(Unchanged from 2NF)*

---

## 3. Advanced Normal Forms: BCNF and 4NF

### Boyce-Codd Normal Form (BCNF)
* **Rule:** Every "Boss" (determinant $A \rightarrow B$) **must be a candidate key**. No ordinary, non-unique columns are allowed to act as determinants.
* **Why it matters:** Catches rare overlapping key anomalies that slip past 3NF.

### Fourth Normal Form (4NF)
* **Rule:** Must be in BCNF + **No Multi-Valued Dependencies**. A table cannot hold two or more independent multi-valued facts about the same entity (e.g., independent Product Colors and Product Storage).
* **Why it matters:** Prevents massive cartesian product explosions of redundant rows.

---

## Summary Checklist

| Normal Form | Key Requirement | What it Eliminates |
| :--- | :--- | :--- |
| **1NF** | Atomic values in every cell | Multi-valued cells & repeating groups |
| **2NF** | 1NF + No partial dependencies | Partial key dependencies (composite key splits) |
| **3NF** | 2NF + No transitive dependencies | Non-key dependent on non-key attributes |
| **BCNF** | Every determinant must be a super-key | Subtle overlapping candidate key anomalies |
| **4NF** | No multi-valued dependencies | Cartesian product explosions from independent lists |
