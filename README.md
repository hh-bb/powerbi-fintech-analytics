# 💳 Cashier Core Analytics — End-to-End Fintech BI Solution
*Auditoría Transaccional, Embudos de Conversión y SLAs Operativos*

[![Power BI](https://img.shields.io/badge/Power_BI-Desktop-F2C811?style=flat&logo=powerbi&logoColor=black)](https://powerbi.microsoft.com/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-AWS_RDS-336791?style=flat&logo=postgresql&logoColor=white)](https://www.postgresql.org/)
[![DAX](https://img.shields.io/badge/DAX-Advanced_Modeling-orange?style=flat)](https://learn.microsoft.com/dax/)
[![SQL](https://img.shields.io/badge/SQL-Window_Functions-blue?style=flat)](https://en.wikipedia.org/wiki/SQL)
[![License: Proprietary](https://img.shields.io/badge/License-All_Rights_Reserved-red.svg)](LICENSE.md)



---

<a name="english-documentation"></a>
## 🌐 English Documentation

### 📌 Project Overview
A production-grade Business Intelligence platform developed to monitor transactional health, conversion funnels, operational SLAs, and customer retention metrics for a centralized fintech checkout platform (*Cashier Core*) processing high-volume instant payments (PIX) and automated withdrawals.

---

### 🎯 Core Business Objectives
* **Transactional Reliability:** Monitor approval, rejection, and drop-off rates across architectural layers: Gateway/Bank (`PSP`), checkout interface validation rules (`Cajero`), and core platform communication (`Omega`).
* **Conversion Funnel Auditing:** Pinpoint exact user and technical drop-offs across 4 milestones (Session Checkout -> Order Initialization -> QR Code Issuance -> Bank Settlement).
* **Operational SLA Tracking:** Measure manual withdrawal turnaround buckets (`< 2h`, `2-24h`, `24-48h`, `> 48h`) and calculate automation ratios.
* **Payment Recovery & Cohorts:** Map cross-payment retry behavior after transaction failures and measure retention trajectories for First Time Depositors (FTD) across 24h, 7d, 30d, 60d, and 90d intervals.

---

### 🏛️ Data Architecture: Constellation Schema (Galaxy Schema)
To accommodate disparate event granularities without generating ambiguous circular relationships, the semantic model implements a **Constellation Schema** sharing conformed single-direction dimensions:


       [Dim_MetodosPago]     [Dim_Dispositivos]     [Dim_Estados]
                \                   |                   /
                 \                  |                  /
  [Fact_Transacciones]     [Fact_SLA_Retiros]     [Fact_Embudo_Depositos]
            \                 /            \               /
             \               /              \             /
              [Calendario] (Role-Playing)    [Brands] (Conformed)
                     |                             |
             [Fact_Redepositos]           [Fact_User_Journeys]

Conformed Dimensions
Calendario: Formally validated date table supporting continuous time-series continuity (Mark as Date Table), ensuring non-broken calculations across sparse activity data.
Brands: Master dimension standardizing internal identifiers (brand_id) into operational business brands.
Dim_MetodosPago / Dim_Dispositivos: Normalized lookup entities for payment methods and hardware categories.
Dim_Estados: Homogenized transactional lifecycle states (SUCCESS, FAILED, REJECTED, CANCELLED).

Fact Tables
Fact_Transacciones: Aggregated volumetric table capturing daily transactional totals, amounts, and layer failure classifications.
Fact_Embudo_Depositos: Ordered milestone events tracking stage-by-stage checkout conversion.
Fact_SLA_Retiros: Execution records tracking automated vs. manual approval runtimes categorized by resolution buckets.
Fact_User_Journeys: Session-level user retry sequences mapping payment method switches upon technical failure.
Fact_Redepositos: Cohort recurrence metrics evaluating re-deposit intervals from initial user activation (FTD).

⚙️ ETL & Pushdown Computation Strategy
To preserve VertiPaq memory and optimize analytical responsiveness, transformations and aggregations are pushed down directly to the PostgreSQL instance via Value.NativeQuery:

Server-Side Aggregations: Raw event logs are aggregated by date, brand, device, and status at the database layer, decreasing row footprint by over 80%.

Window Functions: Implemented ROW_NUMBER() OVER(PARTITION BY user_id ORDER BY created_at ASC) to detect FTD transactions directly in SQL.

Time Deltas in Epochs: Executed EXTRACT(EPOCH FROM (event_end - event_start)) in SQL to generate clean latency metrics in seconds and hours.

Error Taxonomy Engine: Mapped unstructured JSON log payloads (log_info->>'errorMessage') into definitive operational layers (OMEGA, CAJERO, PSP, ABANDONO / TIMEOUT).

📊 Dashboard Modules & Analytical Views
Executive Overview: High-level executive KPI scorecard displaying Net Cashflow, overall success rates, W/D Ratio (Withdrawals to Deposits), and operational SLA gauges.

Deposit Funnel: 4-step conversion flow identifying friction between checkout entry, order creation, QR code generation, and bank payment confirmation.

Retry & Recovery Matrix: Cross-analysis tracing user behavior after payment rejections, identifying which secondary payment methods achieve the highest recovery rates.

Retention Cohorts (FTD): Lifecycle tracking measuring re-deposit velocity across 24h, 7d, 30d, 60d, and 90d intervals.

📂 Repository File Structure
├── LICENSE.md                     # Proprietary copyright notice
├── README.md                      # Architecture and operational documentation
├── docs/                          # Diagram exports and dashboard visual assets
│   ├── data-model.png             # Constellation schema diagram
│   ├── overview-tab.png           # Executive overview view
│   ├── funnel-analysis.png        # 4-Step conversion funnel
│   └── retries-recovery.png       # Retry routes and user journey matrix
├── dax/                           # Documented DAX measures
│   ├── 01_calculo_moda_dinamica.dax
│   ├── 02_sla_distribucion_retiros.dax
│   ├── 03_time_intelligence_mom.dax
│   └── 04_embudo_conversion_drops.dax
└── sql/                           # Native SQL queries powering the ETL
    ├── etl_fact_transacciones.sql
    ├── etl_fact_embudo_depositos.sql
    ├── etl_fact_redepositos.sql
    └── etl_fact_sla_retiros.sql

🔒 Confidentiality & Data Sanitization
All metrics, brand entities, and database infrastructure endpoints presented in this repository have been sanitized:

Sensitive brand names and server connection strings have been replaced with anonymous placeholders.

Visual dashboards exhibit aggregated metrics with normalized numerical values strictly for technical portfolio display.

