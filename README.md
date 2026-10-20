# 💳 Cashier Core Analytics — End-to-End Fintech BI Solution
*Auditoría Transaccional, Embudos de Conversión y SLAs Operativos*

[![Power BI](https://img.shields.io/badge/Power_BI-Desktop-F2C811?style=flat&logo=powerbi&logoColor=black)](https://powerbi.microsoft.com/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-AWS_RDS-336791?style=flat&logo=postgresql&logoColor=white)](https://www.postgresql.org/)
[![DAX](https://img.shields.io/badge/DAX-Advanced_Modeling-orange?style=flat)](https://learn.microsoft.com/dax/)
[![SQL](https://img.shields.io/badge/SQL-Window_Functions-blue?style=flat)](https://en.wikipedia.org/wiki/SQL)
[![License: Proprietary](https://img.shields.io/badge/License-All_Rights_Reserved-red.svg)](LICENSE.md)

[English Version](#english-documentation) | [Versión en Español](#documentación-en-español)

---

<a name="english-documentation"></a>

## 🌐 English Documentation

### 📌 Project Overview
A production-grade Business Intelligence platform developed to monitor transactional health, conversion funnels, operational SLAs, and customer retention metrics for a centralized fintech checkout platform (*Cashier Core*) processing high-volume instant payments (PIX) and automated withdrawals.

### 🎯 Core Business Objectives
- **Transactional Reliability:** Monitor approval, rejection, and drop-off rates across architectural layers: Gateway/Bank (`PSP`), checkout interface validation rules (`Cajero`), and core platform communication (`Omega`).
- **Conversion Funnel Auditing:** Pinpoint exact user and technical drop-offs across 4 milestones (Session Checkout -> Order Initialization -> QR Code Issuance -> Bank Settlement).
- **Operational SLA Tracking:** Measure manual withdrawal turnaround buckets (`< 2h`, `2-24h`, `24-48h`, `> 48h`) and calculate automation ratios.
- **Payment Recovery & Cohorts:** Map cross-payment retry behavior after transaction failures and measure retention trajectories for First Time Depositors (FTD) across 24h, 7d, 30d, 60d, and 90d intervals.

### 🏛️ Data Architecture: Constellation Schema (Galaxy Schema)
To accommodate disparate event granularities without generating ambiguous circular relationships, the semantic model implements a **Constellation Schema** sharing conformed single-direction dimensions:

    [Dim_MetodosPago]     [Dim_Dispositivos]     [Dim_Estados]
             \                   |                   /
              \                  |                  /
       [Fact_Transacciones]  [Fact_SLA_Retiros]  [Fact_Embudo_Depositos]
                 \              /       \              /
                  \            /         \            /
            [Calendario] (Role-Playing)   [Brands] (Conformed)
                   |                            |
           [Fact_Redepositos]          [Fact_User_Journeys]

#### Conformed Dimensions
- **`Calendario`:** Formally validated date table supporting continuous time-series continuity (`Mark as Date Table`), ensuring non-broken calculations across sparse activity data.
- **`Brands`:** Master dimension standardizing internal identifiers (`brand_id`) into operational business brands.
- **`Dim_MetodosPago` / `Dim_Dispositivos`:** Normalized lookup entities for payment methods and hardware categories.
- **`Dim_Estados`:** Homogenized transactional lifecycle states (`SUCCESS`, `FAILED`, `REJECTED`, `CANCELLED`).

#### Fact Tables
- **`Fact_Transacciones`:** Aggregated volumetric table capturing daily transactional totals, amounts, and layer failure classifications.
- **`Fact_Embudo_Depositos`:** Ordered milestone events tracking stage-by-stage checkout conversion.
- **`Fact_SLA_Retiros`:** Execution records tracking automated vs. manual approval runtimes categorized by resolution buckets.
- **`Fact_User_Journeys`:** Session-level user retry sequences mapping payment method switches upon technical failure.
- **`Fact_Redepositos`:** Cohort recurrence metrics evaluating re-deposit intervals from initial user activation (FTD).

### ⚙️ ETL & Pushdown Computation Strategy
To preserve VertiPaq memory and optimize analytical responsiveness, transformations and aggregations are pushed down directly to the PostgreSQL instance via `Value.NativeQuery`:
- **Server-Side Aggregations:** Raw event logs are aggregated by date, brand, device, and status at the database layer, decreasing row footprint by over 80%.
- **Window Functions:** Implemented `ROW_NUMBER() OVER(PARTITION BY user_id ORDER BY created_at ASC)` to detect FTD transactions directly in SQL.
- **Time Deltas in Epochs:** Executed `EXTRACT(EPOCH FROM (event_end - event_start))` in SQL to generate clean latency metrics in seconds and hours.
- **Error Taxonomy Engine:** Mapped unstructured JSON log payloads (`log_info->>'errorMessage'`) into definitive operational layers (`OMEGA`, `CAJERO`, `PSP`, `ABANDONO / TIMEOUT`).

### 📊 Dashboard Modules & Analytical Views
1. **Executive Overview:** High-level executive KPI scorecard displaying Net Cashflow, overall success rates, W/D Ratio (Withdrawals to Deposits), and operational SLA gauges.
2. **Deposit Funnel:** 4-step conversion flow identifying friction between checkout entry, order creation, QR code generation, and bank payment confirmation.
3. **Retry & Recovery Matrix:** Cross-analysis tracing user behavior after payment rejections, identifying which secondary payment methods achieve the highest recovery rates.
4. **Retention Cohorts (FTD):** Lifecycle tracking measuring re-deposit velocity across 24h, 7d, 30d, 60d, and 90d intervals.

### 📂 Repository File Structure

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

### 🔒 Confidentiality & Data Sanitization
All metrics, brand entities, and database infrastructure endpoints presented in this repository have been sanitized:
- Sensitive brand names and server connection strings have been replaced with anonymous placeholders.
- Visual dashboards exhibit aggregated metrics with normalized numerical values strictly for technical portfolio display.

---

<a name="documentación-en-español"></a>

## 🇪🇸 Documentación en Español

### 📌 Resumen del Proyecto
Solución de Business Intelligence desarrollada para supervisar la salud transaccional, embudos de conversión, acuerdos de nivel de servicio (SLA) y retención en una plataforma transaccional de pagos (*Cashier Core*) que procesa transferencias instantáneas masivas (PIX) y retiros automatizados.

### 🎯 Objetivos de Negocio Implementados
- **Confiabilidad Transaccional:** Monitoreo de tasas de aprobación, rechazo y abandono distribuidas por capa de falla: Pasarela/Banco (`PSP`), motor de interfaz/reglas (`Cajero`) y plataforma central (`Omega`).
- **Embudo de Conversión (4 Pasos):** Identificación de caídas entre hitos (Apertura de Checkout -> Generación de Orden -> Emisión de Código QR -> Liquidación Bancaria).
- **Supervisión de SLAs Operativos:** Medición de tiempos de resolución de retiros manuales clasificados por franjas (`< 2h`, `2-24h`, `24-48h`, `> 48h`) y cálculo de tasas de automatización.
- **Recuperación y Retención:** Análisis del comportamiento del usuario al reintentar tras un rechazo (conmutación de método de pago) y curvas de cohortes para usuarios nuevos (FTD) a 24h, 7d, 30d, 60d y 90d.

### 🏛️ Arquitectura de Datos: Esquema en Constelación (Galaxy Schema)
Para resolver los distintos niveles de granularidad sin generar ambigüedad en los filtros, el modelo semántico implementa un **Esquema en Constelación** con dimensiones conformadas de filtro unidireccional:

    [Dim_MetodosPago]     [Dim_Dispositivos]     [Dim_Estados]
             \                   |                   /
              \                  |                  /
       [Fact_Transacciones]  [Fact_SLA_Retiros]  [Fact_Embudo_Depositos]
                 \              /       \              /
                  \            /         \            /
            [Calendario] (Role-Playing)   [Brands] (Conformed)
                   |                            |
           [Fact_Redepositos]          [Fact_User_Journeys]

#### Dimensiones Conformadas
- **`Calendario`:** Tabla de fechas formalmente validada (`Mark as Date Table`) para garantizar la continuidad temporal de funciones *Time Intelligence* ante días sin operaciones.
- **`Brands`:** Dimensión maestra que homologa identificadores técnicos (`brand_id`) con marcas comerciales.
- **`Dim_MetodosPago` / `Dim_Dispositivos`:** Entidades de normalización para métodos de pago y tipos de dispositivos.
- **`Dim_Estados`:** Estados homologados del ciclo de vida de la transacción (`SUCCESS`, `FAILED`, `REJECTED`, `CANCELLED`).

#### Tablas de Hechos (Fact Tables)
- **`Fact_Transacciones`:** Tabla pre-agrupada que consolida recuentos, importes totales y clasificación de capas de error a nivel diario.
- **`Fact_Embudo_Depositos`:** Registros de progresión secuencial del embudo de depósitos.
- **`Fact_SLA_Retiros`:** Auditoría de tiempos de aprobación operativa para retiros manuales y automáticos.
- **`Fact_User_Journeys`:** Registro de sesiones de reintento para medir resiliencia de pago tras un error técnico o financiero.
- **`Fact_Redepositos`:** Cohortes de recurrencia de depósitos a partir del primer depósito exitoso (FTD).

### ⚙️ Ingeniería de Datos y Estrategia ETL Pushdown
Para maximizar la eficiencia de memoria en VertiPaq, las transformaciones pesadas se delegan al motor relacional PostgreSQL mediante `Value.NativeQuery`:
- **Pre-agregación en Origen:** Agrupación server-side por fecha, marca, dispositivo y estado, reduciendo el volumen de filas en más de un 80%.
- **Funciones de Ventana:** Uso de `ROW_NUMBER() OVER(PARTITION BY user_id ORDER BY created_at ASC)` para aislar el hito de FTD directamente en SQL.
- **Cálculo de Tiempos en Segundos:** Uso de `EXTRACT(EPOCH FROM ...)` para calcular diferencias temporales sin sobrecargar el modelo en Power BI.
- **Taxonomía de Errores:** Sentencias `CASE` dinámicas para clasificar logs JSON no estructurados (`log_info->>'errorMessage'`) en capas de error (`OMEGA`, `CAJERO`, `PSP`, `ABANDONO / TIMEOUT`).

### 📊 Módulos del Tablero y Solapas Analíticas
1. **Visión Ejecutiva (Overview):** Tarjetas KPI consolidadas de Net Cashflow, tasa global de éxito, ratio Retiros/Depósitos (W/D Ratio) y medidores de SLA en retiros manuales.
2. **Embudo de Depósitos:** Flujo de conversión de 4 etapas que detecta puntos de fuga entre el inicio de sesión, creación de la orden, entrega del QR y confirmación bancaria.
3. **Matriz de Reintentos y Resiliencia:** Análisis cruzado de caminos de pago alternativos que adoptan los usuarios ante rechazos bancarios o caídas de pasarela.
4. **Cohortes de Retención (FTD):** Seguimiento del ciclo de vida del usuario midiendo la velocidad de redepósito a las 24 horas, 7 días, 30 días, 60 días y 90 días.

### 📂 Estructura de Carpetas del Repositorio

    ├── LICENSE.md                     # Cláusula de derechos de autor y licencia propietaria
    ├── README.md                      # Documentación arquitectónica y operativa
    ├── docs/                          # Exportaciones de diagramas y capturas visuales
    │   ├── data-model.png             # Captura del modelo en constelación
    │   ├── overview-tab.png           # Captura de la vista ejecutiva
    │   ├── funnel-analysis.png        # Embudo de conversión de 4 pasos
    │   └── retries-recovery.png       # Matriz de rutas de reintento y recuperación
    ├── dax/                           # Medidas DAX documentadas línea por línea
    │   ├── 01_calculo_moda_dinamica.dax
    │   ├── 02_sla_distribucion_retiros.dax
    │   ├── 03_time_intelligence_mom.dax
    │   └── 04_embudo_conversion_drops.dax
    └── sql/                           # Consultas SQL nativas aplicadas en el ETL
        ├── etl_fact_transacciones.sql
        ├── etl_fact_embudo_depositos.sql
        ├── etl_fact_redepositos.sql
        └── etl_fact_sla_retiros.sql

### 🔒 Confidencialidad y Sanitización de Datos
Todos los datos, marcas e infraestructura expuestos en este repositorio han sido sometidos a un proceso de sanitización:
- Los nombres de marcas y cadenas de conexión a bases de datos han sido anonimizados mediante pseudónimos.
- Las capturas de pantalla presentan datos agregados con cifras normalizadas con propósitos estrictamente demostrativos de portafolio.
