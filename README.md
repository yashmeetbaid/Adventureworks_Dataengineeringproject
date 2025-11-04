# 🏗️ AdventureWorks End-to-End Azure Data Engineering Project

**A Comprehensive Implementation of Dynamic Data Pipelines, Spark Transformations & Power BI Analytics**

## 📘 Overview

This repository showcases a complete, production-ready Azure Data Engineering pipeline for the AdventureWorks dataset. It integrates **Azure Data Factory (ADF)**, **Azure Data Lake Storage Gen2 (ADLS)**, **Azure Databricks (Spark)**, **Azure Synapse Analytics**, and **Power BI** — demonstrating a seamless data flow from ingestion to analytics dashboards.

The project follows the **Medallion Architecture** (Bronze → Silver → Gold) for scalability, modularity, and maintainability.

---

## 🧱 Architecture Diagram

```
GitHub Source (CSV files)
          ↓
Azure Data Factory (Dynamic Pipeline)
          ↓
Azure Data Lake Gen2 (Bronze Layer - Raw Data)
          ↓
Azure Databricks (Spark Transformation)
          ↓
Azure Data Lake Gen2 (Silver Layer - Curated Data)
          ↓
Azure Synapse Analytics (Gold Layer - Analytical Serving)
          ↓
Power BI Dashboard (Visualization & Reporting)
```

---

## 🎯 Project Objectives

-  Build an end-to-end ETL/ELT pipeline using Azure services
-  Ingest raw data dynamically from GitHub using ADF HTTP connectors
-  Store raw data in ADLS (Bronze Layer)
-  Transform & curate data with Databricks (Silver Layer)
-  Serve analytical data via Synapse (Gold Layer)
-  Visualize results with Power BI dashboards
-  Implement parameterization, ForEach looping, and JSON-driven automation
-  Follow Medallion Architecture principles for modern data engineering

---

## ⚙️ Azure Services Used

| Component | Azure Service | Description |
|-----------|--------------|-------------|
|  Storage | Azure Data Lake Gen2 | Hierarchical, scalable data storage |
|  Orchestration | Azure Data Factory | Handles dynamic ingestion pipelines |
|  Transformation | Azure Databricks | Spark-based data cleansing & enrichment |
|  Analytics | Azure Synapse Analytics | Analytical serving layer for BI |
|  Visualization | Power BI | Dashboarding and interactive reports |

---

## 🧩 Folder Structure

```
Adventureworks_Dataengineeringproject/
│
├── notebooks/
│   ├── silver_layer_refer.ipynb        # Databricks Spark transformation notebook
│
├── data/
│   ├── AdventureWorks_Products.csv
│   ├── AdventureWorks_Customers.csv
│   ├── AdventureWorks_Sales_2015.csv
│   └── ...
│
├── config/
│   └── pipeline_config.json            # JSON configuration for dynamic ADF ingestion
│
├── sql/
│   └── synapse_views.sql               # Gold layer SQL view definitions
│
└── README.md
```

---

## 🧰 Prerequisites

Before running this project, ensure you have:

- An active **Azure subscription**
- Access to:
  - Azure Data Factory
  - Azure Data Lake Gen2 (with Hierarchical Namespace)
  - Azure Databricks
  - Azure Synapse Analytics
  - Power BI Desktop
- Basic familiarity with Python, PySpark, and SQL

---

## 📦 Dependencies

| Component | Dependency | Version |
|-----------|-----------|---------|
| Databricks Runtime | Apache Spark | 3.x (Databricks 11.x LTS) |
| Python | PySpark | 3.x |
| Azure SDKs | azure-storage, azure-synapse | Latest |
| Power BI | Power BI Desktop | Latest |

---

## 🚀 Step-by-Step Execution Guide

### 1️⃣ Resource Setup

1. Create a **Resource Group** in Azure
2. Create the following services under that group:
   - Azure Data Lake Gen2 (enable Hierarchical Namespace)
   - Azure Data Factory
   - Azure Databricks
   - Azure Synapse Analytics

---

### 2️⃣ Data Lake Configuration

Create containers in your ADLS account:

```
/adventureworks/
├── bronze/
├── silver/
├── gold/
└── parameters/
```

---

### 3️⃣ Azure Data Factory (ADF) Setup

#### Linked Services

- **HTTP** → connects to GitHub raw CSV URLs
- **ADLS Gen2** → connects to target Data Lake

#### Datasets

- **Source**: HTTP → Delimited Text
- **Sink**: ADLS → Delimited Text

Add parameters:
- `p_rel_url`
- `p_sink_folder`
- `p_sink_file`

#### Upload Configuration File

Upload `pipeline_config.json` to `/parameters/` in ADLS

**Example JSON:**

```json
[
  {
    "p_rel_url": "yashmeetbaid/Adventureworks_Dataengineeringproject/main/Data/AdventureWorks_Products.csv",
    "p_sink_folder": "AdventureWorks_Products",
    "p_sink_file": "AdventureWorks_Products.csv"
  }
]
```

#### Pipeline Design

1. **Lookup Activity** → reads the JSON file
2. **ForEach Activity** → iterates through all datasets
3. Inside ForEach → add **Copy Data Activity** to move files

#### Run & Publish

1. Run the pipeline in **Debug Mode**
2. Once validated, click **Publish All**

 **Output:** Raw CSVs land in `/bronze/` for each dataset.

---

### 4️⃣ Databricks: Silver Layer Transformation

1. Launch **Azure Databricks Workspace**
2. Create a **Cluster**
   - Runtime: Databricks 11.x (Spark 3.x)
   - Node Type: Standard_DS3_v2
3. Attach the notebook `silver_layer_refer.ipynb`
4. Configure storage access via OAuth or account key
5. Run the notebook steps:
   - Read data from `/bronze/`
   - Apply schema validation, cleaning, joins, deduplication
   - Write cleaned data in Parquet format to `/silver/`

 **Output:** Cleaned, curated Parquet files stored in `/silver/`.

---

### 5️⃣ Synapse: Gold Layer Serving

1. Open **Azure Synapse Studio**
2. Link ADLS Gen2 as a data source
3. Create or use a database (e.g., `AdventureWorksGold`)
4. Run SQL view creation scripts (from `/sql/synapse_views.sql`):

```sql
CREATE VIEW gold.sales AS
SELECT * FROM OPENROWSET(
    BULK 'https://<storageaccount>.dfs.core.windows.net/silver/AdventureWorks_Sales/',
    FORMAT = 'PARQUET'
) AS rows;
```

5. Validate:

```sql
SELECT COUNT(*) FROM gold.sales;
```

 **Output:** Views and external tables available for BI tools.

---

### 6️⃣ Power BI Dashboard

1. Open **Power BI Desktop**
2. Connect to **Azure Synapse Analytics** using the SQL endpoint
3. Select the `gold` schema views (e.g., `sales`, `products`, `customers`)
4. Build dashboards to visualize:
   - Sales by Region/Year
   - Top Products by Revenue
   - Customer Insights
   - Return Trends

 **Output:** Fully interactive Power BI dashboards with live Synapse connectivity.

---

## 📊 Validation Results

| Layer | Check | Status |
|-------|-------|--------|
| Bronze | 10 CSVs ingested dynamically |  Success |
| Silver | Data cleaned & joined |  Success |
| Gold | SQL views + CTAS created |  Success |
| Power BI | Dashboards live in <5s |  Success |

---

## 🔒 Security & Governance

- **RBAC** (Role-Based Access Control) for Azure resources
- **ACLs** on ADLS folders
- **Managed Identity** authentication for Synapse & Databricks
- Optional integration with **Azure Key Vault** for secret storage

---

## 🧠 Best Practices & Learnings

- Always **parameterize pipelines** for reusability
- Use **ForEach looping** instead of static Copy Activities
- Maintain **consistent naming** across datasets and layers
- Enable **retry policies** and **parallelism** in ADF for scale
- Use **Parquet** for efficient querying and compression
- Implement **CI/CD** via GitHub Actions or Azure DevOps for production

---

## Future Enhancements

-  Implement **Delta Lake** for ACID compliance
-  Add **ADF Triggers** / Event-based orchestration
-  Metadata-driven dynamic SQL generation in Synapse
-  Integrate with **Azure DevOps CI/CD** pipelines

---

## 👨‍💻 Authors

**Yashmeet Baid, Akshata Athreya, Tanisha Savitha Naik, Kunal K**

📍 School of Computer Science and Engineering, RV University, Bengaluru

📧 yashmeetb.btech23@rvu.edu.in

---

## 🏁 Conclusion

This project demonstrates a real-world, end-to-end Azure Data Engineering pipeline integrating dynamic orchestration, Spark transformations, and Power BI analytics. It serves as both an educational reference and a production-grade blueprint for modern cloud data pipelines.

---

## 📎 References

- [Microsoft Docs – Azure Synapse Serverless SQL Reference](https://docs.microsoft.com/azure/synapse-analytics/)
- [Azure Architecture Center – Medallion Architecture Pattern](https://docs.microsoft.com/azure/architecture/)
- [Power BI Documentation – Power BI Tutorials](https://docs.microsoft.com/power-bi/)
- **YouTube Masterclass:** AdventureWorks End-to-End Azure Data Engineering Project (2025)

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## ⭐ Star this repository if you found it helpful!
