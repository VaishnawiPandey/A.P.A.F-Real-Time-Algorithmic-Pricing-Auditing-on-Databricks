# A.P.A.F-Real-Time-Algorithmic-Pricing-Auditing-on-Databricks
# A.P.A.F: Real-Time Algorithmic Pricing Auditing on Databricks

![Databricks](https://img.shields.io/badge/Databricks-FF3621?style=for-the-badge&logo=databricks&logoColor=white)
![PySpark](https://img.shields.io/badge/PySpark-E25A1C?style=for-the-badge&logo=apache-spark&logoColor=white)
![Delta Lake](https://img.shields.io/badge/Delta_Lake-0073E6?style=for-the-badge&logo=databricks&logoColor=white)
![Big Data](https://img.shields.io/badge/Big_Data_Analytics-000000?style=for-the-badge&logo=data-companyl&logoColor=white)

**A capstone project for the MBA Business Analytics at University Of Hyderabad (Big Data Analytics) program.**

---

## 1. Problem Statement

In modern e-commerce, **dynamic pricing** is common. Prices for the same product can change in real-time based on supply, demand, and user behavior. However, this can lead to **algorithmic bias** or **price discrimination**, where different users are charged different prices based on their location, device type, or browsing history.

This is a significant **ethical and business risk**. The challenge is that this bias is difficult to detect at scale, as it's hidden within millions of real-time pricing requests.

This project, **A.P.A.F. (Algorithmic Pricing Auditing for Fairness)**, is a Big Data analytics platform built to solve this problem. It ingests, processes, and analyzes pricing data in real-time to detect, quantify, and visualize algorithmic bias as it happens.

---

## 2. The 5 Implemented Use Cases

This single Big Data pipeline enables 5 distinct analytics use cases, all visualized on a central dashboard:

1.  **Use Case 1: Real-Time Bias Detection & Alerting**
    * **Description:** The core system functionality. It flags individual product windows (`product_id`, `geo_cluster`, `time_window`) that fail a fairness audit, identifying them as `HIGH_RISK_AUDIT_DETECTED`.
2.  **Use Case 2: Executive KPI Monitoring**
    * **Description:** Provides a high-level, aggregate view of pricing fairness health for business leaders.
    * **Evidence:** Dashboard scorecards showing "Total High-Risk Events" and "Average Price Gap %."
3.  **Use Case 3: Biased Cluster Identification**
    * **Description:** Identifies which specific customer segments (`geo_cluster`) are systematically being charged the highest prices, pointing to the source of the bias.
    * **Evidence:** Dashboard scorecard showing "Cluster Most Charged Max Price."
4.  **Use Case 4: Temporal Pattern Analysis**
    * **Description:** Analyzes *when* the bias is most active (e.g., time of day) to help engineers pinpoint the root cause (e.g., a specific algorithm that runs every hour).
    * **Evidence:** The "Audit Failures Over Time" bar chart.
5.  **Use Case 5: Product-Specific Deep-Dive Analysis**
    * **Description:** A "smoking gun" drill-down tool for product managers to visually confirm price discrimination for a specific, high-risk product.
    * **Evidence:** The dynamic "Price Gap Visualization" line chart.

---

## 3. Big Data Architecture: The Medallion Lakehouse

The project is built on Databricks using a **Medallion Architecture** to ensure data quality and scalability.



* **Bronze Layer (Raw Data):**
    * **Technology:** Databricks Auto Loader
    * **Process:** Ingests raw, nested JSON `price_request` events as they land in cloud storage. Data is stored in its raw form with schema evolution managed by Auto Loader.

* **Silver Layer (Cleaned & Enriched Data):**
    * **Technology:** PySpark Structured Streaming
    * **Process:**
        1.  The raw Bronze stream is cleaned, flattened, and validated.
        2.  A **Stream-Static Join** is performed to enrich the live data with the `product_catalog` dimension table (e.g., adding `category`).
        3.  The resulting clean, enriched stream is written to the Silver Delta table.

* **Gold Layer (Business Aggregates):**
    * **Technology:** PySpark Structured Streaming (with `foreachBatch`)
    * **Process:** This is the core of the project's intelligence.
        1.  A streaming aggregation groups events by `product_id`, `geo_cluster`, and a `1-hour window`.
        2.  The stream is written using **`foreachBatch`**. This allows us to run complex batch logic on each micro-batch.
        3.  Inside `foreachBatch`, a **two-step Window Function** is used to calculate the **Price Gap Ratio (PGR)**:
            * **Step A:** Find the `global_max_price` for a product *across all clusters* in that window.
            * **Step B:** Calculate the PGR for each cluster: `(global_max_price - cluster_avg_price) / global_max_price`.
        4.  This final metric, along with the `audit_risk_flag`, is upserted (merged) into the final Gold Delta table that powers the dashboard.

---

## 4. Technology Stack

* **Platform:** Databricks
* **Core Processing:** PySpark, Spark Structured Streaming
* **Storage:** Delta Lake (on Unity Catalog)
* **Ingestion:** Auto Loader (for streaming file ingestion)
* **Data Modeling:** Medallion Architecture (Bronze, Silver, Gold)
* **Analytics & Visualization:** Databricks SQL, Databricks Dashboards

---

## 5. How to Run

1.  **Set up Databricks:** Ensure you have a cluster running and have created the `ecommerce_audit` catalog and `audit_schema` schema in Unity Catalog.
2.  **Data Simulator:** Run the `data_simulator/simulate_events.py` script on your local machine to generate and upload the JSON event files to your cloud storage landing zone.
3.  **Run Notebooks (in order):**
    * `0_Bronze_Ingestion.ipynb`: This notebook sets up the Auto Loader stream to ingest raw JSON into the Bronze table.
    * `1_Silver_Enrichment.ipynb`: This notebook reads the Bronze stream, performs the stream-static join, and writes to the Silver table.
    * `2_Gold_Aggregation.ipynb`: This is the main logic notebook. It reads the Silver stream and runs the `foreachBatch` logic to compute the Price Gap Ratio and write to the final Gold table.
4.  **View Dashboard:** Open the **Databricks SQL** persona and navigate to the **"A.P.A.F. - Algorithmic Pricing Audit Dashboard"** to see the live results.

---

## 6. Final Results: The A.P.A.F. Dashboard

The final dashboard visualizes all 5 use cases, providing a 360-degree view of algorithmic bias. It includes high-level KPIs, temporal analysis, and the "smoking gun" line chart to prove price divergence.
