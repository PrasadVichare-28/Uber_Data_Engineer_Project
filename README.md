<div align="center">

# 🚗 Real-Time Uber Analytics
### A Medallion Data Engineering Pipeline on Azure & Databricks

<br/>

**An end-to-end streaming data pipeline that simulates Uber ride confirmations in real time, ingests events through Azure Event Hubs (Managed Kafka), processes them through a Medallion Architecture in Databricks, and delivers a production-ready Star Schema for analytics.**

<br/>

[![Azure](https://img.shields.io/badge/Microsoft_Azure-0078D4?style=for-the-badge&logo=microsoftazure&logoColor=white)](https://azure.microsoft.com/)
[![Databricks](https://img.shields.io/badge/Databricks-FF3621?style=for-the-badge&logo=databricks&logoColor=white)](https://www.databricks.com/)
[![Apache Spark](https://img.shields.io/badge/Apache_Spark-E25A1C?style=for-the-badge&logo=apachespark&logoColor=white)](https://spark.apache.org/)
[![Apache Kafka](https://img.shields.io/badge/Apache_Kafka-231F20?style=for-the-badge&logo=apachekafka&logoColor=white)](https://kafka.apache.org/)
[![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=for-the-badge&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com/)
[![Delta Lake](https://img.shields.io/badge/Delta_Lake-003366?style=for-the-badge&logo=delta&logoColor=white)](https://delta.io/)
[![Jinja](https://img.shields.io/badge/Jinja2-B41717?style=for-the-badge&logo=jinja&logoColor=white)](https://jinja.palletsprojects.com/)

<br/>

![Stars](https://img.shields.io/github/stars/anshlambagit/Uber_Data_Engineer_Project?style=social)
![Forks](https://img.shields.io/github/forks/anshlambagit/Uber_Data_Engineer_Project?style=social)
![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)

</div>

<br/>

---

<br/>

## 📌 Table of Contents

<details open>
<summary><b>Click to expand / collapse</b></summary>

&nbsp;

- [Architecture Overview](#-architecture-overview)
- [Detailed Architecture](#-detailed-architecture)
- [Tech Stack](#%EF%B8%8F-tech-stack)
- [Pipeline Deep Dive](#-pipeline-deep-dive)
  - [Phase 1 — Real-Time Data Generation & Event Hubs](#phase-1--real-time-data-generation--event-hubs)
  - [Phase 2 — Batch Ingestion via Azure Data Factory](#phase-2--batch-ingestion-via-azure-data-factory)
  - [Phase 3 — Bronze Layer & Databricks Setup](#phase-3--bronze-layer--databricks-setup)
  - [Phase 4 — Silver Layer & One Big Table](#phase-4--silver-layer--one-big-table)
  - [Phase 5 — Gold Layer (Star Schema & SCDs)](#phase-5--gold-layer-star-schema--scds)
  - [Phase 6 — Orchestration](#phase-6--orchestration)
- [Key Engineering Decisions](#-key-engineering-decisions)
- [Getting Started](#-getting-started)
- [Project Structure](#-project-structure)
- [Contact](#-contact)

</details>

<br/>

---

<br/>

## 🏗 Architecture Overview

<div align="center">

<img src="architecture.png" alt="End-to-End Pipeline Architecture" width="100%"/>

</div>

<br/>

> **Two ingestion paths converge into a unified Medallion Lakehouse.** A FastAPI web app pushes real-time ride events to Azure Event Hubs (Managed Kafka), while Azure Data Factory pulls historical bulk data and static reference files from GitHub. Both streams land in ADLS Gen2, flow through **Bronze → Silver → Gold** layers in Databricks using Spark Declarative Pipelines, and produce a fully dimensional **Star Schema** with SCD Type 2 — ready for BI, reporting, and analytics.

<br/>

---

<br/>

## 🔎 Detailed Architecture

<div align="center">

<img src="architecture_detailed.png" alt="Uber Real-Time Data Engineering Architecture on Azure — Detailed View" width="100%"/>

</div>

<br/>

<table>
<tr>
<td width="50%">

### 🔵 Ingestion & Bronze Layer

| Data Type | Source System | Azure Service |
|:---|:---|:---|
| Real-Time Events | FastAPI Web App | Azure Event Hubs (Kafka) |
| Historical Bulk | GitHub API | Azure Data Factory (ADF) |
| Static Mapping | Internal CSV/JSON | ADLS Gen2 |

- **Metadata-Driven ADF Pipelines** — ingestion is fully parameterized; adding new data sources requires only a config file update, not code changes.
- **Bronze Layer Fidelity** — stores an exact replica of source data in Delta Lake format for full auditability and replayability.

</td>
<td width="50%">

### 🟡 Transformation & Gold Model

| Component | Technology | Purpose |
|:---|:---|:---|
| Stream Processing | Spark Declarative Pipelines (SDP) | Automated, high-performance streaming logic |
| SQL Generation | Jinja2 Templating | Dynamic, scalable OBT query construction |
| Data Model | Star Schema + SCD Type 2 | Facts & Dimensions with historical tracking |

- **Jinja2 SQL Templating** — SQL queries for the "One Big Table" (OBT) are generated dynamically, enabling scalable, modular code.
- **SCD Type 2** — organizes data into Facts and Dimensions using Slowly Changing Dimensions to track historical value changes.

</td>
</tr>
</table>

<br/>

---

<br/>

## ⚙️ Tech Stack

<div align="center">

```
  ┌─────────────────────────────────────────────────────────────────────────────────┐
  │                              TECH STACK OVERVIEW                                │
  ├──────────────┬──────────────────────────────────┬───────────────────────────────┤
  │    LAYER     │          TECHNOLOGY              │          PURPOSE              │
  ├──────────────┼──────────────────────────────────┼───────────────────────────────┤
  │  Generation  │  FastAPI (Python)                │  Real-time ride simulation    │
  │  Streaming   │  Azure Event Hubs (Kafka)        │  Pub/Sub message broker       │
  │  Batch       │  Azure Data Factory (ADF)        │  Metadata-driven ingestion    │
  │  Storage     │  ADLS Gen2 + Delta Lake          │  Centralized lakehouse        │
  │  Compute     │  Databricks + PySpark            │  Distributed transformations  │
  │  Framework   │  Spark Declarative Pipelines     │  Orchestration & lineage      │
  │  Templating  │  Jinja2                          │  Dynamic SQL generation       │
  │  Pattern     │  Medallion (Bronze/Silver/Gold)  │  Progressive data enrichment  │
  │  Modeling    │  Star Schema + SCD Type 1 & 2    │  Dimensional model with CDC   │
  └──────────────┴──────────────────────────────────┴───────────────────────────────┘
```

</div>

<br/>

---

<br/>

## 🔬 Pipeline Deep Dive

### Phase 1 — Real-Time Data Generation & Event Hubs

<table>
<tr>
<td width="55%">

```
 ┌──────────────────┐         Kafka Protocol
 │   FastAPI App     │ ═══════════════════════▶
 │   (Producer)      │       Send Policy
 │                   │
 │  POST /book-ride  │
 └──────────────────┘
                           ┌──────────────────────┐
                           │  Azure Event Hubs     │
                           │  (Managed Kafka)      │
                           │                       │
                           │  ┌─────────────────┐  │
                           │  │  uber-rides      │  │
                           │  │  (Topic)         │  │
                           │  └─────────────────┘  │
                           └──────────┬───────────┘
                                      │ Listen Policy
                                      ▼
                           ┌──────────────────────┐
                           │  Databricks Consumer   │
                           │  format = "kafka"      │
                           └──────────────────────┘
```

</td>
<td width="45%">

#### 💡 Key Highlights

- **Azure Event Hubs** operates as a managed Apache Kafka instance on the **Standard Pricing Tier** (required to enable Kafka topics).

- **Shared Access Policies** enforce security — a dedicated **Send** policy authorizes the FastAPI producer, while a **Listen** policy authorizes the Databricks consumer.

- A **FastAPI web application** simulates a ride-booking service: each booking generates a batch of simulated ride data and publishes the event to the Event Hub topic in real time.

</td>
</tr>
</table>

<br/>

---

### Phase 2 — Batch Ingestion via Azure Data Factory

<table>
<tr>
<td width="45%">

#### 💡 Key Highlights

- Performs historical/initial bulk loads and ingests **static mapping files** (cities, vehicle types, payment methods) from a GitHub repository into ADLS Gen2.

- Built as a **metadata-driven pipeline** — a `files_array.json` config drives the entire pipeline, eliminating hardcoded paths.

- **Single pipeline handles N files** — add a new source by editing JSON, not code.

</td>
<td width="55%">

```
  ┌────────────┐     HTTP Linked Service
  │   GitHub    │ ━━━━━━━━━━━━━━━━━━━━━━━━▶ ┌──────────────────────┐
  │  Repository │                            │  Azure Data Factory  │
  └────────────┘                            │                      │
                                             │  ┌────────────────┐ │
  ┌────────────┐     ADLS Linked Service     │  │ 1. Lookup      │ │
  │   ADLS     │ ◀━━━━━━━━━━━━━━━━━━━━━━━━  │  │    Activity     │ │
  │   Gen2     │      Copy Data Output       │  │    ↓           │ │
  │            │                             │  │ 2. ForEach     │ │
  │ ┌────────┐ │                             │  │    Loop        │ │
  │ │Raw Zone│ │                             │  │    ↓           │ │
  │ └────────┘ │                             │  │ 3. Copy Data   │ │
  └────────────┘                            │  │    Activity     │ │
                                             │  └────────────────┘ │
              files_array.json               │                      │
  ┌──────────────────────────┐              └──────────────────────┘
  │ [                        │
  │   {"file": "cities"},    │  ◀── Config-driven ingestion
  │   {"file": "vehicles"},  │      (Lookup reads this file)
  │   {"file": "payments"}   │
  │ ]                        │
  └──────────────────────────┘
```

</td>
</tr>
</table>

<br/>

---

### Phase 3 — Bronze Layer & Databricks Setup

<table>
<tr>
<td>

```
  ┌─────────────────────────────────────────────────────────────────────────┐
  │                         🥉  BRONZE LAYER                                │
  │                                                                         │
  │   ┌──────────────┐      ┌─────────────────┐      ┌──────────────────┐  │
  │   │  Event Hubs   │      │  SAS Token Auth  │      │  Raw Delta       │  │
  │   │  Kafka Stream  │─────▶│  ADLS Gen2       │─────▶│  Tables          │  │
  │   │  (Binary JSON) │      │  Integration     │      │  (Exact Copy)    │  │
  │   └──────────────┘      └─────────────────┘      └──────────────────┘  │
  │                                                                         │
  │   • format="kafka" with connection strings & starting offsets           │
  │   • Spark Declarative Pipelines (SDP) for orchestration & checkpoints  │
  │   • Pandas → Spark DF conversion for JSON mapping files                │
  └─────────────────────────────────────────────────────────────────────────┘
```

</td>
</tr>
</table>

- **ADLS Gen2 Integration** via SAS tokens — JSON mapping files read through Pandas, then converted to Spark DataFrames.
- **Spark Declarative Pipelines (SDP)** automate pipeline orchestration, checkpointing, and dependency management.
- Real-time Event Hubs data consumed as a **Kafka stream** (`format="kafka"`) using connection strings and starting offsets. Raw payloads arrive as binary JSON strings.

<br/>

---

### Phase 4 — Silver Layer & One Big Table

<table>
<tr>
<td>

```
  ┌─────────────────────────────────────────────────────────────────────────────────┐
  │                             🥈  SILVER LAYER                                     │
  │                                                                                  │
  │   ┌──────────────┐    from_json()    ┌──────────────┐   Jinja2 SQL Generation   │
  │   │ Raw Binary    │ ───────────────▶ │  Parsed       │ ───────────────────────▶  │
  │   │ JSON Strings  │    + Schema      │  Tabular Data │    Dynamic LEFT JOINs     │
  │   └──────────────┘                  └──────┬───────┘    + WATERMARK (3 min)     │
  │                                             │                     │              │
  │                                      dp.append_flow               ▼              │
  │                                    (Bulk + Streaming)    ┌──────────────────┐    │
  │                                                          │  Silver OBT       │    │
  │                                                          │  (One Big Table)  │    │
  │                                                          └──────────────────┘    │
  └─────────────────────────────────────────────────────────────────────────────────┘
```

</td>
</tr>
</table>

<details>
<summary><b>🔍 Click to expand — Jinja2 Templating Example</b></summary>

&nbsp;

Instead of hardcoding repetitive `LEFT JOIN` operations (which easily break when business needs change), the pipeline uses Jinja2 templating:

```python
# Python configuration dictionary
join_config = {
    "dim_city":     {"key": "city_id"},
    "dim_vehicle":  {"key": "vehicle_type_id"},
    "dim_payment":  {"key": "payment_method_id"},
}
```

```sql
-- Jinja2 dynamically generates:
SELECT s.*, 
{% for table, config in join_config.items() %}
  {{ table }}.*,
{% endfor %}
FROM streaming_table s
{% for table, config in join_config.items() %}
LEFT JOIN {{ table }} ON s.{{ config.key }} = {{ table }}.{{ config.key }}
{% endfor %}
```

This decouples join logic from table definitions — schema changes don't break SQL.

</details>

<br/>

- **Data Normalization** — PySpark `from_json()` with a defined schema converts raw binary strings into proper tabular columns.
- **Append Flow** — `dp.append_flow` safely unions the initial bulk load with continuous streaming data, preventing reprocessing of historical data.
- **Jinja2 Templating** — A Python config dictionary drives dynamic SQL generation for all LEFT JOIN operations, making the pipeline resilient to schema changes.
- **Streaming Watermarks** — The `STREAM` keyword with a `WATERMARK` (3-minute delay) on `booking_timestamp` handles late-arriving data in stateful streaming transformations.

<br/>

---

### Phase 5 — Gold Layer (Star Schema & SCDs)

<table>
<tr>
<td width="55%">

```
  ┌─────────────────────────────────────────┐
  │            🥇  GOLD LAYER                │
  │                                          │
  │          ┌──────────────────┐            │
  │          │  dim_passenger   │ SCD Type 1 │
  │          ├──────────────────┤            │
  │          │  dim_driver      │ SCD Type 1 │
  │          ├──────────────────┤            │
  │  OBT ──▶│  dim_vehicle     │ SCD Type 1 │
  │          ├──────────────────┤            │
  │          │  dim_payment     │ SCD Type 1 │
  │          ├──────────────────┤            │
  │          │  dim_booking     │ SCD Type 1 │
  │          ├──────────────────┤            │
  │          │  dim_location    │ SCD Type 2 │
  │          │  (start_at /     │            │
  │          │   end_at)        │            │
  │          ├──────────────────┤            │
  │          │  fact_rides      │ Measures   │
  │          │  (fares, dist,   │            │
  │          │   duration)      │            │
  │          └──────────────────┘            │
  └─────────────────────────────────────────┘
```

</td>
<td width="45%">

#### 💡 Dimensional Modeling

The Silver OBT is split into **6 Dimension** tables and **1 Fact** table using `dp.create_autocdc_flow`.

**SCD Type 1** (Overwrite)
> Used for Passengers, Drivers, Vehicles, Payments, and Bookings — old records are overwritten with new ones.

**SCD Type 2** (Historical Tracking)
> Implemented on `dim_location` using `sequence_by` on a timestamp column. Databricks auto-generates `start_at` / `end_at` columns to track the active lifespan of each record.

#### ⚠️ Critical Join Rule

```sql
-- REQUIRED when joining Fact → SCD Type 2
SELECT *
FROM fact_rides f
JOIN dim_location d
  ON f.location_id = d.location_id
WHERE d.end_at IS NULL  -- ← Mandatory!
```

Without this filter, joins multiply records against expired historical rows.

</td>
</tr>
</table>

<br/>

---

### Phase 6 — Orchestration

```
  ┌──────────────────────────────────────────────────────────────────────────────────────┐
  │                         ⚡ DATABRICKS JOB — SCHEDULED WORKFLOW                       │
  │                                                                                      │
  │    ┌─────────────────┐       ┌─────────────────┐       ┌──────────────────────┐     │
  │    │  Task 1          │       │  Task 2          │       │  Task 3               │     │
  │    │  Bronze ADLS     │ ────▶ │  Silver Jinja    │ ────▶ │  SDP Pipeline         │     │
  │    │  Load            │       │  Processing      │       │  (Gold Data Model)    │     │
  │    └─────────────────┘       └─────────────────┘       └──────────────────────┘     │
  │                                                                                      │
  └──────────────────────────────────────────────────────────────────────────────────────┘
```

A **Databricks Job** unifies all pipeline stages into a single scheduled workflow with dependency chaining across Bronze → Silver → Gold.

<br/>

---

<br/>

## 🧠 Key Engineering Decisions

| # | Decision | Rationale |
|:---:|:---|:---|
| 1 | **Event Hubs over raw Kafka** | Fully managed, native Azure integration, eliminates cluster ops overhead |
| 2 | **Metadata-driven ADF pipeline** | Single pipeline handles N files — add a new source by editing JSON, not code |
| 3 | **`dp.append_flow` for union** | Prevents reprocessing the bulk historical load on every micro-batch |
| 4 | **Jinja2 for SQL generation** | Decouples join logic from table definitions — schema changes don't break SQL |
| 5 | **Watermarking (3 min)** | Balances data freshness with tolerance for late-arriving streaming events |
| 6 | **SCD Type 2 on Location** | Preserves geographic change history critical for ride analytics over time |
| 7 | **`end_at IS NULL` join filter** | Mandatory guard against Cartesian explosion when joining Facts to SCD2 dims |
| 8 | **SAS tokens for ADLS access** | Granular, time-bound authentication without exposing storage account keys |

<br/>

---

<br/>

## 🚀 Getting Started

### Prerequisites

| Requirement | Details |
|:---|:---|
| **Azure Subscription** | Event Hubs (Standard tier), ADLS Gen2, Data Factory |
| **Databricks Workspace** | Free or Standard edition with SDP/DLT support |
| **Python 3.9+** | FastAPI, azure-eventhub, Jinja2, PySpark |

### Quick Start

```bash
# 1️⃣  Clone the repository
git clone https://github.com/anshlambagit/Uber_Data_Engineer_Project.git
cd Uber_Data_Engineer_Project

# 2️⃣  Install Python dependencies
pip install -r requirements.txt

# 3️⃣  Configure Azure credentials
#     → Set Event Hub connection strings (Send & Listen policies)
#     → Set ADLS Gen2 SAS tokens
#     → Update Databricks secrets / configs

# 4️⃣  Run the FastAPI data generator
uvicorn app:app --reload

# 5️⃣  Deploy ADF pipeline
#     → Import the pipeline JSON into your Azure Data Factory instance
#     → Trigger the initial batch load

# 6️⃣  Deploy Databricks notebooks
#     → Import notebooks into your Databricks workspace
#     → Configure the SDP pipeline and schedule the Databricks Job
```

<br/>

---

<br/>

## 📂 Project Structure

```
Uber_Data_Engineer_Project/
│
├── 📄 README.md
├── 🖼️ architecture.png                 # Pipeline architecture diagram
├── 🖼️ architecture_detailed.png        # Detailed architecture visual
│
├── 🐍 fastapi_app/                     # Real-time data generator
│   ├── app.py                          # FastAPI application entry point
│   └── data_generator.py              # Ride data simulation logic
│
├── 🏭 adf_pipeline/                    # Azure Data Factory configs
│   └── files_array.json               # Metadata-driven file list
│
├── 📓 databricks_notebooks/            # Databricks processing notebooks
│   ├── bronze_layer.py                # Raw ingestion & Event Hubs consumer
│   ├── silver_layer.py                # Jinja-templated joins & OBT
│   ├── gold_layer.py                  # Star schema & SCD processing
│   └── jinja_templates/               # Dynamic SQL templates
│
└── ⚙️ config/                          # Connection strings & configurations
```

> **Note:** Adjust the directory structure above to match the actual layout of your repository.

<br/>

---

<br/>

## 📬 Contact

<div align="center">

**Prasad Anshlamba**

[![GitHub](https://img.shields.io/badge/GitHub-anshlambagit-181717?style=for-the-badge&logo=github)](https://github.com/anshlambagit)

<br/>

If you found this project useful, consider giving it a ⭐ on GitHub!

---

*Built with* &nbsp; ![Azure](https://img.shields.io/badge/-Azure-0078D4?style=flat-square&logo=microsoftazure&logoColor=white) &nbsp; ![Databricks](https://img.shields.io/badge/-Databricks-FF3621?style=flat-square&logo=databricks&logoColor=white) &nbsp; ![Spark](https://img.shields.io/badge/-PySpark-E25A1C?style=flat-square&logo=apachespark&logoColor=white) &nbsp; ![FastAPI](https://img.shields.io/badge/-FastAPI-009688?style=flat-square&logo=fastapi&logoColor=white) &nbsp; ![Delta Lake](https://img.shields.io/badge/-Delta_Lake-003366?style=flat-square&logo=delta&logoColor=white)

</div>
