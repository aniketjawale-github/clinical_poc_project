# Azure Clinical Data Engineering POC (Free Tier)

**End‑to‑end Azure data engineering project** that integrates 3 sources — **Database, API, and ADLS files** — and delivers analytics via a modern **Medallion Architecture (Bronze → Silver → Gold)** using **ADF, Databricks, Synapse (Serverless), and Power BI**. Built to run on the **Azure Free Tier** with minimal cost.

---

## 🚀 Quick Intro

This POC simulates a pharma/clinical scenario (e.g., Abbott) by ingesting:

* **Azure SQL DB** (trial subjects),
* **HTTP API** (Adverse Events via Azure Functions), and
* **ADLS files** (LOINC‑coded Lab Results),
  then transforming in **Azure Databricks**, storing curated datasets in **ADLS**, exposing them via **Synapse Serverless**, and visualizing in **Power BI**.

**Highlights**

* 3 sources → ADF orchestration with **Self‑Hosted IR**
* Databricks transformations + KPIs
* Synapse Serverless external tables over ADLS **Gold**
* Git integration across **ADF, Databricks, Synapse**
* Designed for **₹0 using Free Tier credits**

---

## 🧱 Reference Architecture

```
Database (Azure SQL) ─┐
API (Azure Function) ─┼─> Azure Data Factory (Self‑Hosted IR + Auto‑Resolve)
ADLS (CSV/JSON) ──────┘            │
                                   ▼
                         ADLS Gen2 (Bronze)
                                   ▼
                           Databricks (ETL)
                                   ▼
                         ADLS Gen2 (Silver/Gold)
                                   ▼
                        Synapse (Serverless SQL)
                                 
```



---

## 🧰 Azure Services Used

* **Azure SQL Database (Basic/Free Tier)** – transactional source (trial subjects)
* **Azure Functions (HTTP Trigger)** – simulated Adverse Events API
* **Azure Data Lake Storage Gen2** – Raw/Silver/Gold zones
* **Azure Data Factory** – ingestion, scheduling, Databricks notebook runs
* **Integration Runtime** – **Self‑Hosted IR** (hybrid) + **Auto‑Resolve IR**
* **Azure Databricks** – transformations, joins, KPIs (Delta/Parquet)
* **Azure Synapse Analytics (Serverless SQL Pool)** – external tables on Gold
* **Power BI** – dashboards
* **GitHub** – Git integration for ADF, Databricks, Synapse

---

## 🗂️ Repo Structure

```
/README.md
/docs/architecture.png               # (optional) exported diagram                       # exported notebooks (ipynb/py)
/adf/
  linkedServices/                    # ARM/JSON
  datasets/
  pipelines/
/databricks/
  notebooks/                          
synapse/
  sqlscripts/                               # external table & view scripts
                        
```

---

## 📚 Domain & Datasets (Samples)

**1) Trial Subjects (Database/CSV)**

* Columns: `subject_id, trial_id, site_id, enrollment_date, arm, sex, dob, country`

**2) Adverse Events (API/JSON)**

* Columns: `event_id, subject_id, trial_id, onset_date, resolved_date, preferred_term, severity, serious, relatedness, outcome`

**3) Lab Results (Files/CSV)**

* Columns: `lab_result_id, subject_id, encounter_id, order_datetime_utc, collected_datetime_utc, result_datetime_utc, loinc_code, test_name, result_value, unit, abnormal_flag, status, lab_vendor`

> Place sample files under `/data/raw/...` and register them in ADF.

---

## 🧪 Medallion Layers

**Bronze (Raw)**

* As‑is copies of sources.
* Example paths: `/raw/trial_subjects/`, `/raw/adverse_events/`, `/raw/lab_results/` (partition by `ingest_date=`)

**Silver (Clean & Conform)**

* Standardized schemas, types, UTC timestamps, basic quality rules.
* Example outputs: `/silver/trial_subjects/`, `/silver/adverse_events/`, `/silver/lab_results/`

**Gold (Curated & KPIs)**

* Joined subject‑level and trial‑level tables.
* Example outputs: `/gold/subject_summary/`, `/gold/trial_kpi/`, optional `/gold/ae_detail/`, `/gold/lab_detail/`

**Dataset Counts** *(example for POC)*

* Bronze: **3** datasets
* Silver: **3** datasets
* Gold: **2–4** datasets

---

## 🔢 KPIs (Examples)

* **Serious AE Rate (%)** = serious AEs / total AEs
* **Avg AE Duration (days)**
* **Abnormal Lab %** = abnormal tests / total tests
* **Mean Lab Result (per subject/trial)**
* **Subjects Enrolled (trial/site)**

---

## ⚙️ Setup & Run

### Prerequisites

* Azure subscription (Free Trial / credits)
* Resource Group + Region
* Contributor rights on RG

### 1) Provision Services (minimal cost)

* **ADLS Gen2** (Hierarchical Namespace = On)
* **Azure SQL Database** (Basic tier) + table for subjects
* **Azure Function** (Consumption plan) for AE API (HTTP trigger)
* **Databricks** (trial/small cluster; stop when idle)
* **Synapse Workspace** (use **Serverless SQL** only)
* **ADF** (Data Factory v2)

### 2) Integration Runtime

* Install & register **Self‑Hosted IR** on your machine (for local/secured sources)
* Use **Auto‑Resolve IR** for cloud‑to‑cloud moves

### 3) Git Integration

* **ADF** → Manage → Git configuration → connect GitHub repo/branch
* **Databricks** → Repos → connect Git (GitHub) for notebooks
* **Synapse** → Manage → Git configuration → link repo for SQL scripts

### 4) Ingestion (ADF)

* Linked Services: Azure SQL, HTTP (Function), ADLS, Databricks, Synapse
* Datasets: CSV/JSON for files, REST for API, SQL for DB
* Pipelines:

  1. **Copy_SQL_to_ADLS** → `/raw/trial_subjects/`
  2. **Copy_API_to_ADLS** → `/raw/adverse_events/`
  3. **Copy_Files_to_ADLS** → `/raw/lab_results/`
  4. **Run_Databricks_ETL** (notebook activity)
  5. **Refresh_Synapse_Metadata** (optional: SQL script)

### 5) Transformations (Databricks)

* **Silver:** type casting, UTC normalization, dedupe, data quality rules
* **Gold:** joins (subjects ⟷ labs ⟷ AEs), KPI calculations, write Parquet/Delta to `/gold`

### 6) Synapse (Serverless)

* Create **EXTERNAL DATA SOURCE** to ADLS Gold
* Define **EXTERNAL TABLES / VIEWS** over Parquet/Delta folders
* Validate with `SELECT TOP 100 * FROM dbo.subject_summary;`

### 7) Power BI

* Connect to Synapse (Serverless) or directly to ADLS Parquet
* Publish dashboards

---

## 🔐 Security & Secrets

* Use **Key Vault** for connection strings & keys (optional in POC)
* RBAC on ADLS containers; Private Endpoints if required

---

## 💸 Cost Tips

* Prefer **Serverless Synapse** (pay‑per‑query)
* Use **Consumption plan** for Functions
* **Stop/terminate** Databricks clusters when idle
* Limit ADF triggers and pipeline runs

---

## ✅ Deliverables

* Automated pipelines (ADF)
* Cleaned Silver and curated Gold datasets in ADLS
* Synapse Serverless external tables
* Power BI dashboard(s)
* Git history for ADF, Databricks, and Synapse assets

---

## 📄 License

MIT (or your preferred license)

---

## 🙋 Support / Session

Interested in **building or learning** this kind of Azure project? I’m happy to help or run a short walkthrough session. Connect with me on LinkedIn or open an issue in this repo.
