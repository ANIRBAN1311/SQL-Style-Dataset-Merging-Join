# n8n – SQL-Style Dataset Merging & Join Examples

A no-code workflow demonstrating how to combine two datasets in n8n using SQL-style join logic—no external database required.

---

## 🚀 Project Overview

- **Name:** n8n – SQL-Style Dataset Merging & Join Examples  
- **Purpose:** Showcase n8n’s Merge node to:
  - Append two lists (union all)  
  - Keep only new items (left join + filter)  
  - Keep only existing items (inner join)  
  - Enrich one dataset with matching fields from another (left join + merge)  
- **Key Benefits:**
  - Eliminates the need for manual data merging  
  - Demonstrates SQL-like logic entirely in a visual, no-code environment  
  - Applies to many real-world data-integration scenarios  
  - Scales without writing any custom code  

---

## 💡 Use Case Overview

When you have two separate sets of records—say **A** and **B**—you often need to merge them based on a common key. In n8n, the **Merge** node offers four modes that mimic SQL behavior:

1. **Union All (Append Datasets)**  
   - Combine all records from A and B into a single array, preserving duplicates.

2. **Inner Join (Keep Only Existing Items)**  
   - Return only records whose key exists in both A and B.

3. **Left Join + Filter (Keep Only New Items)**  
   - Return records in A that do **not** have a matching key in B.

4. **Left Join + Enrich (Enrich Data)**  
   - Combine A and B by adding B’s fields onto matching A records; unmatched A records remain with their original fields.

---

## 🌍 Real-World Applications

- **Reporting Pipelines**  
  Merge “CRM Leads” (A) with “Campaign Responses” (B) to produce a unified “Lead Status” report.

- **Inventory Management**  
  Combine “Products Needed” (A) with “Stock on Hand” (B) to identify which items require reordering.

- **Customer 360 Profiles**  
  Join “User Profiles” (A) with “Transactional Data” (B) to enrich profiles with purchase history before pushing to a CRM.

- **Analytics & BI**  
  Aggregate “Ad Spend Logs” (A) with “Performance Metrics” (B) to calculate ROI per channel.

---

## 🛠️ Tech Stack

- **n8n.io**  
  - Visual workflow orchestrator with a built-in **Merge** node.  
  - Supports modes: `unionAll`, `innerJoin`, `leftJoin`, `enrichInput1`.

- **Code Node** (JavaScript)  
  - Generate or normalize sample datasets A and B.  

- **Storage Nodes** (e.g., Google Sheets, CSV)  
  - Source external datasets or persist merged results.

- **Optional Database**  
  - For complex lookups, though not required for this demo.

---

## 🔧 Workflow Breakdown

1. **Manual Trigger**  
   - Node: `Manual Trigger`  
   - Kick off the workflow on demand to test merging scenarios.

2. **Dataset A (Code Node)**  
   - Example:  
     ```js
     return [
       { "Name": "Wheat" },
       { "Name": "Eggs" },
       { "Name": "Milk" },
       { "Name": "Lemon" },
       { "Name": "Sugar" }
     ];
     ```
   - This represents “Ingredients Needed.”

3. **Dataset B (Code Node)**  
   - Example:  
     ```js
     return [
       { "Name": "Eggs" },
       { "Name": "Lemon" },
       { "Name": "Sugar" }
     ];
     ```
   - This represents “Ingredients in Stock.”

4. **Inner Join (Keep Only Existing Items)**  
   - Node: `Merge`  
   - Mode: `innerJoin`  
   - Keys: `["Name"]`  
   - Result:  
     ```jsonc
     [
       { "Name": "Eggs" },
       { "Name": "Lemon" },
       { "Name": "Sugar" }
     ]
     ```

5. **Left Join + Enrich (Add Quantities)**  
   - Modify Dataset B to include quantities:  
     ```jsonc
     [
       { "Name": "Wheat",   "Quantity": "100g" },
       { "Name": "Eggs",    "Quantity": 2       },
       { "Name": "Salt",    "Quantity": "50g"  },
       { "Name": "Lemon",   "Quantity": 1       },
       { "Name": "Sugar",   "Quantity": "6tbsp"}
     ]
     ```
   - Node: `Merge`  
   - Mode: `enrichInput1`  
   - Keys: `["Name"]`  
   - Result:  
     ```jsonc
     [
       { "Name": "Wheat", "Quantity": "100g" },
       { "Name": "Eggs",  "Quantity": 2       },
       { "Name": "Milk"                },
       { "Name": "Lemon", "Quantity": 1       },
       { "Name": "Sugar", "Quantity": "6tbsp"}
     ]
     ```

6. **Union All (Append A and B)**  
   - Node: `Merge`  
   - Mode: `unionAll`  
   - Directly concatenates A + B:  
     ```jsonc
     [
       { "Name": "Wheat" },
       { "Name": "Eggs" },
       { "Name": "Milk" },
       { "Name": "Lemon" },
       { "Name": "Sugar" },
       { "Name": "Eggs" },
       { "Name": "Lemon" },
       { "Name": "Sugar" }
     ]
     ```

7. **Left Join + Filter (Keep Only New Items)**  
   - Node: `Merge`  
   - Mode: `leftJoin`  
   - Keys: `["Name"]`  
   - Result (with B data keyed by `null` when no match):  
     ```jsonc
     [
       { "Name": "Wheat", "fromB": null },
       { "Name": "Eggs", "fromB": { "Name": "Eggs" } },
       { "Name": "Milk", "fromB": null },
       { "Name": "Lemon","fromB": { "Name": "Lemon" } },
       { "Name": "Sugar","fromB": { "Name": "Sugar" } }
     ]
     ```
   - Follow with a **Filter** node to keep only items where `fromB` is `null`:  
     ```jsonc
     [
       { "Name": "Wheat" },
       { "Name": "Milk" }
     ]
     ```

8. **Save or Output**  
   - Node: `Google Sheets` / `Write to CSV` / `Console Log`  
   - Persist the merged dataset to your preferred destination.

---

## 🔢 Equivalent SQL Queries

Below are SQL examples that achieve the same results as each Merge mode:

1. **Union All (Append A and B)**  
   ```sql
   SELECT Name FROM A
   UNION ALL
   SELECT Name FROM B;
