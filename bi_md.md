# bi_md — Master Data ETL

**GitHub:** [glg/bi_md](https://github.com/glg/bi_md)

## Overview

`bi_md` is the **Master Data (MD) ETL repository** for GLG. It contains SQL ETL scripts and DDL for a Snowflake master data database that serves as the authoritative source for Workday Prism integrations, WARS reporting, and other downstream consumers. Data sourced here feeds finance, HR, and operational reporting systems.

## Tech Stack

| Layer | Technology |
|---|---|
| Data Warehouse | Snowflake (MASTER_DATA database) |
| Query Language | Snowflake SQL |
| Downstream | Workday Prism, WARS |

## Repository Structure

```
bi_md/
├── README.md
├── CODEOWNERS
├── PULL_REQUEST_TEMPLATE.md
└── MD/
    ├── CORE/
    │   ├── etl/                        ← Core ETL SQL scripts
    │   └── ddl/                        ← Core DDL definitions
    └── WORKDAY/
        ├── USAGE/                      ← Usage data for Workday Prism
        ├── CLIENT_CONTRACT/            ← Active client contract data
        ├── CLIENT_CONTRACT_DISABLED/   ← Disabled/expired contracts
        ├── THIN_LEDGER/                ← Thin ledger financial data
        └── NM_PAYMENT/                 ← Network Member payment data
```

## Data Domains

### CORE
Foundation tables and ETL scripts that other Workday integrations depend on. Includes base-level entity definitions (companies, employees, cost centers, etc.) and their DDL.

### WORKDAY Integrations

| Module | Description |
|---|---|
| `USAGE` | Usage data prepared for Workday Prism reporting — tracks product and service consumption |
| `CLIENT_CONTRACT` | Active client contract information — feeds Workday financial workflows |
| `CLIENT_CONTRACT_DISABLED` | Historical/disabled contracts — preserved for audit and historical reporting |
| `THIN_LEDGER` | Thin ledger financial line-item data — supports Workday accounting integrations |
| `NM_PAYMENT` | Network Member payment records — tracks payment status, amounts, and bank details |

## Data Flow

```
Source Systems (GLGLIVE SQL Server, Aurora PostgreSQL, WARS)
        │
        ▼
    bi_md ETL SQL Scripts
        │
        ▼
    Snowflake MASTER_DATA
        │
        ├──► Workday Prism (HR / Finance reports)
        └──► WARS (analytical reporting via WARS ETL)
```

## ETL Pattern

Each module under `WORKDAY/` contains:
- **`etl/`** — SQL scripts that read from source systems and load into the MASTER_DATA Snowflake tables
- **`ddl/`** — `CREATE OR REPLACE TABLE` definitions for each target table

Scripts follow the same naming and sync-mode conventions as `BI_vdw` and are executed by the `bi_etl` engine.

## Governance

- **CODEOWNERS:** `@glg/dre` — all PRs require DRE team review
- **PR Template:** Standardized checklist to ensure downstream Workday impact is assessed before merging
- DDL changes must also go through **SchemaTron** for ISO 27001 audit compliance

## Related Repositories

| Repo | Role |
|---|---|
| `BI_vdw` | Consumes MASTER_DATA tables for WARS analytical reporting |
| `bi_etl` | Python engine that runs the ETL SQL scripts |
| `bi_airflow_dags` | Schedules the MD ETL runs |
| `SchemaTron` | Manages DDL changes to the MASTER_DATA database |
| `nm-payment-account-job` | Downstream consumer — syncs NM payment data to Workday |
