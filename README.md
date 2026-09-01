# Marketplace Analytics Platform

> End-to-end analytics platform unifying Wildberries, Ozon and Yandex Market data into a centralized PostgreSQL DWH and Qlik Sense analytical environment.

## Executive Summary

I designed and developed the analytical layer of a marketplace data platform for a multi-billion-ruble e-commerce business operating across Wildberries, Ozon and Yandex Market.

Before the project, analytics relied on marketplace dashboards, 1C reports, Excel exports and ad-hoc data extraction. Historical data was fragmented, metric definitions differed between sources, and many cross-marketplace analyses required days of manual preparation.

The platform established a reusable analytical foundation across **6 seller accounts, 2M+ products, 5M+ marketplace listings and 350M+ analytical records**.

My role covered analytical architecture, data modeling, API specifications, data validation, marketplace normalization, Qlik transformation layers and BI development. Production API ingestion was implemented by a Python developer based on my specifications.

The platform now supports unified sales analytics, SKU scoring and assortment management, pricing analytics, warehouse performance management, and cohort/unit economics analysis.

---

## At a Glance

| | |
|---|---|
| **Role** | Data Analyst / Analytics Platform Owner |
| **Scale** | 2M+ products · 5M+ listings · 350M+ records |
| **Sources** | Wildberries · Ozon · Yandex Market · 1C |
| **Stack** | PostgreSQL · Python · SQL · Qlik Sense |
| **Core contribution** | Architecture · Data modeling · Marketplace normalization · BI |
| **Users** | CEO · Marketplace managers · Warehouse team |

---

## Business Problem

The project originally started as a request to segment and analyze a large marketplace assortment. It quickly became clear that sustainable product analytics required solving a more fundamental data problem.

The company had no centralized analytical platform. Marketplace managers primarily worked with native marketplace dashboards, 1C reports and manual Excel exports.

This created four major limitations:

- **Fragmented data** — Wildberries, Ozon and Yandex Market use different entities, identifiers, statuses and business logic.
- **Missing history** — important historical data such as stock levels, funnel metrics and search positions was not consistently retained.
- **Inconsistent metrics** — revenue, buyouts, cancellations and returns could be calculated differently across reports and teams.
- **Limited scalability** — manual analysis could answer individual questions but could not support millions of products and recurring analytical workflows.

A new cross-marketplace business question could require from several hours to several days of data preparation, while some historical analyses were impossible because the required data had never been stored.

---

## My Role

**Data Analyst / Analytics Platform Owner**

I owned the analytical side of the solution from business requirements to production BI.

My responsibilities included:

- gathering requirements from business stakeholders;
- identifying and evaluating marketplace API methods;
- defining datasets, data grain, business keys and loading strategies;
- specifying pagination, incremental loading and duplicate-handling requirements;
- validating ingested data against APIs and source systems;
- designing the analytical architecture and data models;
- writing SQL for validation and analysis;
- developing Qlik transformation and QVD layers;
- normalizing marketplace-specific business logic;
- developing BI applications and business metrics;
- administering the Qlik Sense analytical environment;
- prioritizing further development of the analytics platform.

Production API ingestion was implemented by a Python developer based on my analytical specifications.

Infrastructure and virtual machine administration were handled by the system administrator.

---

## Scale

| Metric | Scale |
|---|---:|
| Physical product assortment | **2M+ products** |
| Marketplace listings | **5M+ listings** |
| Seller accounts | **6** |
| Daily orders | **10K+** |
| Analytical data | **350M+ records** |
| PostgreSQL DWH | **150GB+** |
| Marketplace API methods | **~30** |
| BI users | **10+** |
| Marketplaces | **3** |

---

# Technical Deep Dive

## Solution Architecture

![Marketplace Analytics Platform Architecture](docs/architecture.png)

Marketplace data is collected through an internal API provider responsible for request queues, rate limits and retries.

For each required dataset, I defined the analytical specification: API method, required fields, expected grain, business keys, pagination, full or incremental loading strategy, duplicate handling and validation requirements.

Production ingestion was then implemented in Python based on these specifications.

Source-aligned data is stored in PostgreSQL and extracted into QVD files. The analytical transformation layer follows the general pattern:

**RAW → STG → DIM / MART**

Individual ingestion processes are independent, preventing a failure in one marketplace source from stopping the entire daily load.

The resulting analytical models are used by multiple Qlik Sense applications rather than being built separately for each dashboard.

---

## Unified Analytical Model

One of the main modeling challenges was creating consistent analytical entities across marketplaces with fundamentally different data structures.

![Unified Marketplace Data Model](docs/data-model.png)

### Product Identity

A single physical product can have multiple marketplace listings with different identifiers across Wildberries, Ozon and Yandex Market.

The internal `code_1c` is used as the shared business key connecting these listings.

The analytical model therefore separates two grains:

- **`dim_product_master`** — one physical product per `code_1c`;
- **`dim_marketplace_product`** — one marketplace listing.

This enables cross-marketplace product analytics without losing marketplace-level granularity.

### Unified Sales Fact

Marketplace-specific order and sales structures are transformed into a common analytical fact containing standardized concepts such as:

- order and sale dates;
- quantities;
- revenue;
- cancellations;
- returns;
- buyouts;
- fulfillment model;
- marketplace product identity.

The goal is not to assume that marketplace entities are identical, but to explicitly map their different semantics into a shared analytical model.

---

## Marketplace Normalization

Each marketplace has its own order lifecycle, identifiers and status model. A significant part of the analytical work was converting these source-specific structures into consistent business semantics.

### Wildberries

Wildberries orders and sales are transformed into a common staging structure with standardized product keys, lifecycle dates, quantities, revenue and status flags.

[`transform_wb.qvs`](qlik/transform_wb.qvs)

### Ozon

Ozon FBO and FBS data are normalized into the same analytical structure, including fulfillment-specific logic, financial fields, cancellations, returns and currency normalization.

[`transform_ozon.qvs`](qlik/transform_ozon.qvs)

### Yandex Market

Yandex Market provides order status history rather than the same sales structure used by the other marketplaces.

The transformation reconstructs the analytical order lifecycle and derives standardized event dates and status flags.

[`transform_yandex.qvs`](qlik/transform_yandex.qvs)

The resulting staging datasets are combined into a unified marketplace sales fact:

[`build_unified_sales_mart.qvs`](qlik/build_unified_sales_mart.qvs)

The product identity layer is built separately:

[`build_product_model.qvs`](qlik/build_product_model.qvs)

---

## BI & Analytics

The platform provides reusable analytical models for multiple business processes rather than isolated dashboards.

### Unified Sales Analytics

![Unified Sales Dashboard](docs/unified-sales-dashboard.png)

The application provides a single analytical view across marketplaces and seller accounts.

Business users can move from company-level performance to marketplace, account, category, brand, geography and individual product analysis without manually combining reports from different sources.

The model includes common business metrics such as revenue, buyout rate, cancellation rate and return rate while preserving marketplace-specific dimensions where required.

<details>
<summary><strong>View underlying Qlik data model</strong></summary>

<br>

![Unified Sales Qlik Data Model](docs/qlik-sales-data-model.png)

</details>

---

### Warehouse Operations Analytics

![Warehouse Operations Dashboard](docs/warehouse-dashboard.png)

The warehouse analytical model combines operational events with warehouse, product, employee, role and operation attributes.

It supports analysis of processing volume, processing time, workload distribution and operational performance across warehouses and operation types.

The same data foundation is also used for warehouse KPI and performance-management analytics.

<details>
<summary><strong>View underlying Qlik data model</strong></summary>

<br>

![Warehouse Qlik Data Model](docs/qlik-warehouse-data-model.png)

</details>

---

## Business Impact

The primary result of the project is not a single dashboard but a reusable analytical foundation.

Before the platform, cross-marketplace analysis often required manual exports and repeated data preparation. Historical analysis was limited by whichever data happened to be available at the time.

The platform established persistent historical data and standardized analytical entities that can be reused across projects.

It enabled several downstream analytical initiatives:

- **SKU Scoring & Assortment Management** — combining historical sales, funnel, stock and product data to analyze a multi-million-product assortment.
- **Pricing Analytics** — analyzing prices, marketplace discounts, logistics costs and product economics.
- **Warehouse Performance Management** — analyzing operational performance by warehouse, operation, employee and product.
- **Cohort & Unit Economics Analysis** — evaluating order lifecycle, buyouts, cancellations and returns.
- **Ad-hoc Business Analysis** — answering many recurring questions directly from reusable models instead of rebuilding datasets from source exports.

Analyses that previously required days of manual data preparation can now often be performed in minutes through existing analytical models and BI applications.

Most importantly, new analytical projects no longer need to rebuild the marketplace data foundation from scratch.

---

## Technical Decisions & Trade-offs

### PostgreSQL as the DWH

**Decision:** use PostgreSQL rather than introduce an additional analytical database.

**Why:** PostgreSQL was already supported within the company and was sufficient for the primarily scheduled daily analytical workload. Introducing another database would have increased operational complexity without a clear immediate business benefit.

**Trade-off:** as data volume, concurrency or near-real-time requirements grow, a specialized analytical database could become worth evaluating.

---

### Source-Aligned RAW Layer

**Decision:** store marketplace data close to its original source structure before semantic normalization.

**Why:** marketplace APIs have different structures and business semantics. Preserving source-level data reduces ingestion complexity and makes source-specific behavior easier to investigate.

**Trade-off:** downstream transformations must explicitly handle these differences.

---

### Transformation Logic in Qlik

**Decision:** build a significant part of the STG and MART transformation layer using Qlik Script and QVD.

**Why:** this allowed rapid development of analytical models and BI applications with limited development resources.

**Trade-off:** reusable transformation logic becomes more tightly coupled to the BI environment.

As the platform grows, I would move more reusable STG and MART transformations into PostgreSQL and keep Qlik focused primarily on the semantic and visualization layer.

---

### Scheduled Pipeline Execution

**Decision:** use scheduled loads with time buffers rather than introduce a dedicated orchestration platform during the initial implementation.

**Why:** the platform primarily operates as a daily batch workflow, and the small team prioritized delivering the analytical foundation over adding infrastructure complexity.

**Trade-off:** dependencies become harder to manage as the number of pipelines grows.

---

### Data Quality

Data validation currently includes:

- schema and data type checks;
- row count validation;
- missing-date checks;
- reconciliation of key aggregates;
- selective comparison with marketplace APIs and 1C;
- reference data validation.

A significant part of these checks is still manual.

This was sufficient during the initial development stage but becomes a limitation as the number of datasets and consumers grows.

---

## What I Would Improve

The next iteration of the platform would focus less on adding new BI functionality and more on strengthening the analytical engineering layer.

### 1. Move reusable transformations into PostgreSQL

Shift common STG and MART logic from Qlik into the DWH to improve reuse and reduce BI-layer coupling.

### 2. Automate data quality checks

Introduce automated freshness, completeness, uniqueness and reconciliation tests for critical datasets.

### 3. Add proactive monitoring and alerts

Failed or delayed loads should be detected automatically rather than discovered by downstream users.

### 4. Improve orchestration

Move from time-buffer-based scheduling toward explicit pipeline dependency management as complexity increases.

### 5. Strengthen version control and testing

Treat analytical transformations as production code with reviewable changes and automated tests.

These improvements were not necessary to establish the initial business value of the platform, but they become increasingly important as the analytical environment scales.

---

## Tech Stack

| Area | Technology |
|---|---|
| Data Sources | Wildberries API, Ozon API, Yandex Market API, 1C |
| API Integration | REST API |
| Ingestion | Python |
| Data Warehouse | PostgreSQL |
| Transformation | SQL, Qlik Script |
| Analytical Storage | QVD |
| BI | Qlik Sense |
| Data Modeling | Fact / Dimension analytical models |
| Version Control | Git / GitHub |

---

## Implementation Examples

This repository contains simplified and anonymized examples of the transformation logic used to build the analytical layer.

| File | Purpose |
|---|---|
| [`transform_wb.qvs`](qlik/transform_wb.qvs) | Wildberries normalization |
| [`transform_ozon.qvs`](qlik/transform_ozon.qvs) | Ozon FBO/FBS normalization |
| [`transform_yandex.qvs`](qlik/transform_yandex.qvs) | Yandex Market order lifecycle normalization |
| [`build_unified_sales_mart.qvs`](qlik/build_unified_sales_mart.qvs) | Unified marketplace sales fact |
| [`build_product_model.qvs`](qlik/build_product_model.qvs) | Marketplace and master product dimensions |

The examples demonstrate the analytical approach rather than reproduce the complete production implementation.

---

## Repository Structure

```text
marketplace-analytics-platform/
│
├── README.md
│
├── docs/
│   ├── architecture.png
│   ├── architecture.drawio
│   ├── data-model.png
│   ├── data-model.drawio
│   ├── unified-sales-dashboard.png
│   ├── warehouse-dashboard.png
│   ├── qlik-sales-data-model.png
│   └── qlik-warehouse-data-model.png
│
└── qlik/
    ├── transform_wb.qvs
    ├── transform_ozon.qvs
    ├── transform_yandex.qvs
    ├── build_unified_sales_mart.qvs
    └── build_product_model.qvs
```

---

## Confidentiality Notice

This case study is based on a production analytics platform.

Company-specific identifiers, brands, seller account names, financial values and selected implementation details have been anonymized or modified.

The code included in this repository is a simplified portfolio representation of the production solution and does not contain proprietary source code, credentials or confidential business data.
