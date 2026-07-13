# 🥃 Iowa Liquor Sales: Competitor Intelligence & Market Share Recovery

<p>
  <img src="https://img.shields.io/badge/DuckDB-FFF000?style=flat&logo=duckdb&logoColor=black" alt="DuckDB">
  <img src="https://img.shields.io/badge/Apache_Parquet-00A3A3?style=flat&logo=apacheparquet&logoColor=white" alt="Parquet">
  <img src="https://img.shields.io/badge/Google_Colab-F9AB00?style=flat&logo=googlecolab&logoColor=white" alt="Colab">
  <img src="https://img.shields.io/badge/SQL-4479A1?style=flat&logo=postgresql&logoColor=white" alt="SQL">
  <img src="https://img.shields.io/badge/Power_BI_%2B_DAX-F2C811?style=flat&logo=powerbi&logoColor=black" alt="Power BI">
</p>

*An enterprise-grade business intelligence case study utilizing DuckDB and Parquet for high-speed in-memory analytics, focused on market share recovery, competitor distribution gaps, and tactical field sales optimization.*

---

## 🎯 The Business Problem: The Market Share Bleed

Imagine a mid-sized liquor vendor operating in the highly regulated Iowa market. Looking at their quarterly financial reports, the reality is brutal: they are losing money, losing market share, and losing physical shelf space to direct competitors.

The core problem isn't a lack of data; the state of Iowa provides a massive, public dataset of every single drop of liquor sold. The problem is a lack of *actionable intelligence*. Right now, the vendor's regional managers are flying blind. They know they are losing, but they don't know *where* or *why*. Their field sales representatives are driving county to county, guessing which stores to visit and which products to pitch. Meanwhile, their competitors are systematically poaching their highest-value accounts.

**The Objective:**
Our brand is bleeding market share and physical shelf space, but our sales and executive teams lack the localized, tactical data required to pinpoint exactly where we are losing, which specific competitors are beating us, and which individual stores we must target to recover lost revenue. The business requires an automated intelligence engine to stop the bleed and direct our field reps exactly where to strike.

---

## ⚙️ Core Analytical Execution & Sub-Questions
   * *Question:1* Are we losing ground to our primary competitor in overall market share and physical store presence?
   * *Question:2* Geographically, where is our specific brand bleeding the most compared to the overall market?
   * *Question:3* Within our target counties, which specific cities have the largest distribution gaps where our competitor's products are more widely available than ours?
   * *Question:4* At the exact store level, what specific product assortment or sales velocity gaps exist that our sales reps can action immediately to win back revenue?
   * *Question:5* When are the peak buying seasons for the overall category, and are we capturing our expected share of revenue during those high-volume months compared to the major market players?

---
## ELT Architecture and Data flow
<img width="732" height="251" alt="iow_liquor" src="https://github.com/user-attachments/assets/fe063777-7c80-4b7b-8681-3f1ebc7926ed" />


--
### 🧠 Core Engineering Logic: Geographic Standardization (Demo Snippet)

*📌 Note: When tracking market share and distribution gaps at a local level, dirty geographic data is the biggest threat. In public datasets, inconsistent casing and hidden whitespace (e.g., "Des Moines" vs " DES  MOINES ") fracture data aggregations, making market share appear smaller than it is. Below is the Python/DuckDB snippet used in Google Colab to rigorously standardize this data in-memory before pushing it to Power BI.*

```python
# Snippet from: Google Colab (Data Preprocessing Layer)
# Executing DuckDB SQL directly on Parquet files for ultra-fast cleaning

q = """
CREATE OR REPLACE VIEW sales_history_v2 AS
SELECT 
    * EXCLUDE(city, county),
    
    -- 1. City Standardization: Eradicating ghost spaces, inconsistent cases, and nulls
    COALESCE(
        NULLIF(
            TRIM(
                REGEXP_REPLACE(
                    UPPER(city),
                    '\s+', -- Matches any whitespace character (spaces, tabs, newlines)
                    ' ',   -- Replaces multiple/ghost spaces with ONE standard space
                    'g'    -- Global flag: applies to the entire string
                )
            ),
        ''),
    'UNKNOWN') AS clean_city,

    -- 2. County Standardization: Ensuring accurate geographic market share roll-ups
    COALESCE(
        NULLIF(
            TRIM(
                REGEXP_REPLACE(
                    UPPER(county),
                    '\s+',
                    ' ',
                    'g'
                )
            ),
        ''),
    'UNKNOWN') AS clean_county

FROM sales_history_v1
"""

# Execute the heavy lifting entirely in-memory using DuckDB
duckdb.query(q)
print("Geographic Standardization Complete.")

```
--
   

## 📈 Final Deliverable: Business Intelligence & Actionable Insights

https://app.powerbi.com/view?r=eyJrIjoiYmE0NGE1ZGEtOTlhYy00N2ZkLTliNzYtODIzNzZhMjkyOWYwIiwidCI6IjM0YmQ4YmVkLTJhYzEtNDFhZS05ZjA4LTRlMGEzZjExNzA2YyJ9

<img width="1250" height="803" alt="image" src="https://github.com/user-attachments/assets/0491eedc-de63-472d-a6ea-9863b9798e10" />
<img width="1082" height="790" alt="image" src="https://github.com/user-attachments/assets/871ef942-b74c-4c70-a4a2-dbff9e302529" />
<img width="1328" height="795" alt="image" src="https://github.com/user-attachments/assets/75c39e88-c45c-4864-bc45-e064208c4ec0" />
<img width="1416" height="790" alt="image" src="https://github.com/user-attachments/assets/bdfccf22-5dcd-4974-b71f-68fd20756352" />
<img width="1038" height="792" alt="image" src="https://github.com/user-attachments/assets/eb54c370-3864-4ae3-9e46-1f2f8417b371" />

---
## 📁 Data Sourcing & Scale

To ensure adherence to data privacy while working with enterprise-scale volumes, this independent case study leverages the official **Iowa Liquor Sales** dataset, hosted natively within the **Google BigQuery Public Datasets** program. 

the data extraction and processing were handled programmatically to simulate a cost-optimized, modern data architecture:

* **Cloud Extraction (Google Colab):** The raw transaction logs were queried directly from the BigQuery public data registry (`bigquery-public-data.iowa_liquor_sales`) via Python inside a Google Colab environment.
* **Storage & Compute Optimization (Parquet + DuckDB):** Instead of running expensive analytical queries directly on BigQuery, the heavy partitions were extracted and converted into highly compressed **Apache Parquet** files. These files were then queried locally in-memory using **DuckDB** and SQL, demonstrating a highly efficient, decoupled compute-and-storage architecture before pushing the final aggregates to Power BI.
