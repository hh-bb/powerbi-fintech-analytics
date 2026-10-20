# Cashier Core Analytics — End-to-End Fintech BI Solution
*Auditoría Transaccional, Embudos de Conversión, Recuperación y SLAs Operativos*

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
A production-grade Business Intelligence platform developed to monitor transactional health, conversion bottlenecks, ticket sizing, operational SLAs, and customer retention for a centralized fintech checkout platform (*Cashier Core*) processing instant payment methods (PIX) and withdrawal transactions.

---

### 📑 Comprehensive System Architecture (Full Solution Scope)
The production Power BI report operates as a multi-tier analytics suite covering 7 specialized reporting domains designed for Operations, Payments, and C-Level stakeholders:

1. **Overview (Executive Health Scorecard):**
   - High-level KPIs: Net Cashflow, Total Deposit/Withdrawal volumes, overall success rates, and W/D Ratio (Withdrawals to Deposits).
   - Operational SLA monitoring: Manual withdrawal turnaround distribution across time buckets (`< 2h`, `2-24h`, `24-48h`, `> 48h`) alongside automated vs. manual processing ratios.

2. **Deposit Funnel (`Depósitos Funnel`):**
   - End-to-end 4-stage conversion tracking: Session entry -> Order creation -> PSP QR issuance -> Webhook payment settlement.
   - Stage drop-off analysis to pinpoint user abandonment vs. network/API latency drops.

3. **General Deposits (`Dep General`):**
   - Ticket sizing histogram categorized from $1.000 to +$500.000.
   - Dynamic statistical mode calculation for deposit amounts and longitudinal W/D ratio evolution.
   - Multi-period variance matrices tracking MoM (Month-over-Month) and QoQ (Quarter-over-Quarter) transaction volume and financial amount delta.

4. **Payment Method Breakdown (`Dep Detalle por PM`):**
   - Granular volumetric analysis comparing approval efficiency, processing latency, and ticket sizes across individual payment rails (PIX, Debit Cards, Credit Cards, Digital Wallets, Bank Transfers).

5. **Payment Retries & Recovery (`Dep Reintentos Recuperación`):**
   - User resilience tracking following transaction failures: retry propensity rate (77.5%), latency between attempts (<30s to >10m), and successful recovery rates.
   - Cross-Payment Method Matrix: Tracing routing switches when an original payment rail fails (e.g., Wallet -> Credit Card -> PIX).

6. **Layer Failure Diagnostic (`Dep Errores de Capa`):**
   - Root-cause architectural isolation classifying transaction failures into 4 operational tiers: `OMEGA` (core operator backend), `CAJERO` (internal cashier business rules), `PSP` (financial acquirer / bank rejection), and `ABANDONO / TIMEOUT` (expired QR codes or dropped client sessions).

7. **Retention & Lifetime Value (`AVG FTD & Tiempo`):**
   - First Time Depositor (FTD) volume, average activation amount, and MoM acquisition delta.
   - Multi-cohort recurrence tracking calculating re-deposit conversion rates across 24-hour, 7-day, 30-day, 60-day, and 90-day intervals.

---

### 🖼️ Repository Showcase Scope (Exhibited Assets)
To balance comprehensive architectural demonstration with strict intellectual property and NDA constraints, this public repository features sanitized documentation and selected visual exports from the core analytical modules:

- **Data Model Architecture (`docs/data-model.png`):** Complete constellation semantic model showing table relationships, single-direction filter propagation, and conformed dimensions.
- **Executive Overview Tab (`docs/overview.png`):** Operational scorecard, withdrawal SLA gauges, and automation ratios.
- **General Deposits Tab (`docs/dep-general.png`):** Ticket distribution histogram, statistical mode calculation, and multi-period variance tables.
- **Retries & Recovery Tab (`docs/dep-reintentos.png`):** User journey recovery flows, retry latency distribution, and cross-payment method conversion matrix.

---

### 🏛️ Data Architecture: Constellation Schema (Galaxy Schema)
To handle differing event granularities without circular relationship ambiguities, the data model implements a **Constellation Schema** connected through conformed, single-direction dimensions:

    [Dim_MetodosPago]     [Dim_Dispositivos]     [Dim_Estados]
             \                   |                   /
              \                  |                  /
       [Fact_Transacciones]  [Fact_SLA_Retiros]  [Fact_User_Journeys]
                 \              /       \              /
                  \            /         \            /
            [Calendario] (Role-Playing)   [Brands] (Conformed)
                   |                            |
           [Fact_Redepositos]          [Fact_Errores]

#### Conformed Dimensions
- **`Calendario`:** Formally validated date table (`Mark as Date Table`) ensuring continuous time-series evaluation for *Time Intelligence* measures across sparse data frames.
- **`Brands`:** Master lookup dimension resolving technical internal identifiers into commercial operational entities.
- **`Dim_MetodosPago` / `Dim_Dispositivos`:** Normalized lookup entities for payment rails and hardware classifications.
- **`Dim_Estados`:** Homogenized transactional lifecycle states (`SUCCESS`, `FAILED`, `REJECTED`, `CANCELLED`).

#### Fact Tables
- **`Fact_Transacciones`:** Aggregated volumetric table capturing daily transactional totals, amounts, ticket buckets, and layer failure classifications.
- **`Fact_SLA_Retiros`:** Execution records tracking automated vs. manual approval runtimes categorized by resolution buckets.
- **`Fact_User_Journeys`:** Session-level user retry sequences mapping payment method switches upon technical failure.
- **`Fact_Redepositos`:** Cohort recurrence metrics evaluating re-deposit intervals from initial user activation (FTD).
- **`Fact_Errores`:** Diagnostic failure logs mapping operational bottlenecks across processing layers.

---

### ⚙️ ETL & Pushdown Computation Strategy
To preserve VertiPaq memory and optimize analytical responsiveness, transformations and aggregations are pushed down directly to the PostgreSQL instance via `Value.NativeQuery`:
- **Server-Side Aggregations:** Raw event logs are aggregated by date, brand, device, and status at the database layer, decreasing row footprint by over 80%.
- **Window Functions & Sessionization:** Implemented `ROW_NUMBER()` and `LAG()` over transaction logs in SQL to detect consecutive retry journeys and time intervals without DAX row-by-row iteration.
- **Time Deltas in Epochs:** Executed `EXTRACT(EPOCH FROM (event_end - event_start))` in SQL to calculate manual operational resolution latency in hours.
- **Bucketing & Error Taxonomy:** SQL-level categorical groupings for ticket amounts and error layer classifications (`OMEGA`, `CAJERO`, `PSP`, `ABANDONO / TIMEOUT`).

---

### 📂 Repository File Structure

    ├── LICENSE.md                     # Proprietary copyright notice
    ├── README.md                      # Architecture and operational documentation
    ├── docs/                          # Diagram exports and dashboard visual assets
    │   ├── data-model.png             # Constellation schema diagram
    │   ├── overview.png               # Executive overview view
    │   ├── dep-general.png            # General deposits & ticket sizing view
    │   └── dep-reintentos.png         # Payment retry and recovery matrix view
    ├── dax/                           # Documented DAX measures
    │   ├── 01_calculo_moda_dinamica.dax
    │   ├── 02_sla_distribucion_retiros.dax
    │   ├── 03_time_intelligence_mom.dax
    │   └── 04_tasa_reintento_exitoso.dax
    └── sql/                           # Native SQL queries powering the ETL
        ├── etl_fact_transacciones.sql
        ├── etl_fact_sla_retiros.sql
        ├── etl_fact_user_journeys.sql
        └── etl_fact_redepositos.sql

---

### 🔒 Confidentiality & Data Sanitization
All metrics, brand entities, and database infrastructure endpoints presented in this repository have been sanitized:
- Sensitive brand names and server connection strings have been replaced with anonymous placeholders.
- Visual dashboards exhibit aggregated metrics with normalized numerical values strictly for technical portfolio display.

---

<a name="documentación-en-español"></a>

## 🇪🇸 Documentación en Español

### 📌 Resumen del Proyecto
Solución de Business Intelligence desarrollada para supervisar la salud transaccional, cuellos de botella en la conversión, distribución de tickets, acuerdos de nivel de servicio (SLA) y retención en una plataforma transaccional de pagos (*Cashier Core*) que procesa pagos instantáneos (PIX) y operaciones de retiro.

---

### 📑 Arquitectura Integral del Sistema (Alcance Completo de la Solución)
El reporte productivo de Power BI funciona como una suite analítica modular compuesta por 7 dominios especializados diseñados para Operaciones, Pagos y Dirección Ejecutiva:

1. **Visión Ejecutiva (`Overview`):**
   - Métricas clave de alto nivel: Net Cashflow, volúmenes totales de depósitos y retiros, tasa global de éxito y ratio Retiros/Depósitos (W/D Ratio).
   - Supervisión operativa de SLAs: Distribución de tiempos de resolución de retiros manuales clasificados por franjas (`< 2h`, `2-24h`, `24-48h`, `> 48h`) y porcentaje de automatización.

2. **Embudo de Depósitos (`Depósitos Funnel`):**
   - Monitoreo del flujo de conversión de 4 etapas: Inicio de sesión -> Creación de orden -> Emisión de QR en pasarela -> Liquidación bancaria vía Webhook.
   - Detección de fugas en cada hito para discriminar entre abandono de usuario y caídas de conectividad técnica.

3. **Depósitos General (`Dep General`):**
   - Histograma de tickets clasificado desde $1.000 hasta más de $500.000.
   - Cálculo dinámico de la moda estadística (ticket más frecuente) y evolución temporal del W/D Ratio.
   - Matriz de variaciones periódicas que audita el crecimiento mes contra mes (MoM) y trimestre contra trimestre (QoQ) en volumen e importes.

4. **Detalle por Medio de Pago (`Dep Detalle por PM`):**
   - Análisis comparativo granular de tasas de aprobación, tiempos de acreditación y montos promedio desglosados por canal financiero (PIX, Tarjetas de Débito, Tarjetas de Crédito, Billeteras Virtuales, Transferencias Bancarias).

5. **Reintentos y Recuperación (`Dep Reintentos Recuperación`):**
   - Auditoría de resiliencia del usuario ante rechazos transaccionales: propensión al reintento (77,5%), tiempo transcurrido entre intentos (<30s a >10m) y tasa de recuperación efectiva.
   - Matriz Cross-Método: Identificación de los caminos de conmutación de métodos de pago cuando la opción original falla (ej. Billetera -> Crédito -> PIX).

6. **Diagnóstico de Errores por Capa (`Dep Errores de Capa`):**
   - Aislamiento arquitectónico de causas raíz distribuyendo las fallas en 4 niveles operativos: `OMEGA` (core del operador), `CAJERO` (reglas de negocio de la interfaz), `PSP` (rechazo bancario o del adquirente) y `ABANDONO / TIMEOUT` (expiración de códigos QR o desconexión del cliente).

7. **Retención y Valor del Cliente (`AVG FTD & Tiempo`):**
   - Volumen de primeros depósitos (FTD), ticket promedio de activación y variación mensual.
   - Análisis de cohortes midiendo la velocidad y recurrencia de redepósito a las 24 horas, 7 días, 30 días, 60 días y 90 días.

---

### 🖼️ Alcance Publicado en el Portafolio (Muestra Adjunta)
Con el propósito de conciliar la demostración de competencias técnicas avanzadas con los acuerdos de confidencialidad y propiedad intelectual (NDA), este repositorio público contiene documentación sanitizada y capturas seleccionadas de los módulos neurálgicos:

- **Arquitectura del Modelo de Datos (`docs/data-model.png`):** Diagrama completo del esquema en constelación que ilustra las relaciones, el flujo unidireccional de filtros y las dimensiones conformadas.
- **Solapa Overview (`docs/overview.png`):** Tablero ejecutivo con indicadores de salud transaccional, medidores de SLA y ratios de automatización.
- **Solapa Depósitos General (`docs/dep-general.png`):** Histograma de tickets, cálculo de la moda estadística y matrices de variación temporal.
- **Solapa Reintentos y Recuperación (`docs/dep-reintentos.png`):** Flujos de reintento, distribución de latencias y matriz de conmutación de medios de pago.

---

### 🏛️ Arquitectura de Datos: Esquema en Constelación (Galaxy Schema)
Para resolver los distintos niveles de granularidad sin generar ambigüedad en los filtros, el modelo semántico implementa un **Esquema en Constelación** con dimensiones conformadas de filtro unidireccional:

    [Dim_MetodosPago]     [Dim_Dispositivos]     [Dim_Estados]
             \                   |                   /
              \                  |                  /
       [Fact_Transacciones]  [Fact_SLA_Retiros]  [Fact_User_Journeys]
                 \              /       \              /
                  \            /         \            /
            [Calendario] (Role-Playing)   [Brands] (Conformed)
                   |                            |
           [Fact_Redepositos]          [Fact_Errores]

#### Dimensiones Conformadas
- **`Calendario`:** Tabla de fechas formalmente validada (`Mark as Date Table`) para garantizar la continuidad temporal de funciones *Time Intelligence* ante días sin operaciones.
- **`Brands`:** Dimensión maestra que homologa identificadores técnicos (`brand_id`) con marcas comerciales.
- **`Dim_MetodosPago` / `Dim_Dispositivos`:** Entidades de normalización para métodos de pago y tipos de dispositivos.
- **`Dim_Estados`:** Estados homologados del ciclo de vida de la transacción (`SUCCESS`, `FAILED`, `REJECTED`, `CANCELLED`).

#### Tablas de Hechos (Fact Tables)
- **`Fact_Transacciones`:** Tabla pre-agrupada que consolida recuentos, importes totales, buckets de montos y clasificación de capas de error a nivel diario.
- **`Fact_SLA_Retiros`:** Auditoría de tiempos de aprobación operativa para retiros manuales y automáticos.
- **`Fact_User_Journeys`:** Registro de sesiones de reintento para medir resiliencia y caminos alternativos tras un fallo técnico o financiero.
- **`Fact_Redepositos`:** Cohortes de recurrencia de depósitos a partir del primer depósito exitoso (FTD).
- **`Fact_Errores`:** Registro diagnóstico de fallas agrupadas por capas operativas.

---

### ⚙️ Ingeniería de Datos y Estrategia ETL Pushdown
Para maximizar la eficiencia de memoria en VertiPaq, las transformaciones pesadas se delegan al motor relacional PostgreSQL mediante `Value.NativeQuery`:
- **Pre-agregación en Origen:** Agrupación server-side por fecha, marca, dispositivo y estado, reduciendo el volumen de filas en más de un 80%.
- **Funciones de Ventana y Sesionización:** Uso de `ROW_NUMBER()` y `LAG()` en SQL para detectar secuencias de reintento y deltas de tiempo entre intentos sin iterar en DAX.
- **Cálculo de Tiempos en Segundos:** Uso de `EXTRACT(EPOCH FROM ...)` para calcular diferencias temporales sin sobrecargar el modelo en Power BI.
- **Agrupación y Taxonomía de Errores:** Sentencias `CASE` dinámicas para categorizar buckets de importes y capas de falla (`OMEGA`, `CAJERO`, `PSP`, `ABANDONO / TIMEOUT`).

---

### 📂 Estructura de Carpetas del Repositorio

    ├── LICENSE.md                     # Cláusula de derechos de autor y licencia propietaria
    ├── README.md                      # Documentación arquitectónica y operativa
    ├── docs/                          # Exportaciones de diagramas y capturas visuales
    │   ├── data-model.png             # Captura del modelo en constelación
    │   ├── overview.png               # Captura de la vista ejecutiva
    │   ├── dep-general.png            # Captura de la vista de depósitos general
    │   └── dep-reintentos.png         # Captura de la vista de reintentos y recuperación
    ├── dax/                           # Medidas DAX documentadas línea por línea
    │   ├── 01_calculo_moda_dinamica.dax
    │   ├── 02_sla_distribucion_retiros.dax
    │   ├── 03_time_intelligence_mom.dax
    │   └── 04_tasa_reintento_exitoso.dax
    └── sql/                           # Consultas SQL nativas aplicadas en el ETL
        ├── etl_fact_transacciones.sql
        ├── etl_fact_sla_retiros.sql
        ├── etl_fact_user_journeys.sql
        └── etl_fact_redepositos.sql

---

### 🔒 Confidencialidad y Sanitización de Datos
Todos los datos, marcas e infraestructura expuestos en este repositorio han sido sometidos a un proceso de sanitización:
- Los nombres de marcas y cadenas de conexión a bases de datos han sido anonimizados mediante pseudónimos.
- Las capturas de pantalla presentan datos agregados con cifras normalizadas con propósitos estrictamente demostrativos de portafolio.

