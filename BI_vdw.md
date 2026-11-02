# BI_vdw — WARS Data Warehouse

**GitHub:** [glg/BI_vdw](https://github.com/glg/BI_vdw)

## Overview

`BI_vdw` is GLG's primary Business Intelligence data warehouse repository, housing the **WARS** (Warehouse and Analytics Reporting System) SQL ETL codebase. All analytical tables that power Looker dashboards, executive reporting, and downstream data products originate here.

## Tech Stack

| Layer | Technology |
|---|---|
| Data Warehouse | Snowflake (`kma51753.us-east-1`) |
| Query Language | Snowflake SQL |
| Downstream BI | Looker |
| Source Systems | SQL Server (GLGLIVE, FUSION), Aurora PostgreSQL, Azure SQL |

## Repository Structure

```
BI_vdw/
├── README.md
├── CODEOWNERS                  ← @glg/dre owns all files
├── PULL_REQUEST_TEMPLATE.md
└── WARS/
    ├── ETL_SF/                 ← Core ETL SQL scripts (50+ table folders)
    │   ├── F_PROJECT/          ← Fact tables (F_ prefix)
    │   ├── D_COUNCIL_MEMBER/   ← Dimension tables (D_ prefix)
    │   ├── B_*/                ← Bridge tables (B_ prefix)
    │   ├── A_EMPLOYEE_SUMMARY/ ← Aggregate tables (A_ prefix)
    │   └── ... (50+ more)
    ├── ETL_LIBRARY/            ← Library/events ETL
    ├── ETL_ENRICH/             ← Data enrichment ETL
    ├── ETL_VENDORGRAPHIC/      ← VendorGraphic ETL
    ├── STAGING_SF/             ← Staging layer SQL
    ├── DDL/                    ← Table DDL definitions
    ├── LOADS_POST_PROCESS/     ← Post-processing for Loads DB
    ├── COMPANY_CLEANUP/        ← Company data cleanup scripts
    ├── COPY/                   ← COPY command scripts
    └── EXTERNAL/               ← External data source loading
```

## Table Naming Conventions

| Prefix | Meaning | Example |
|---|---|---|
| `F_` | Fact table | `F_PROJECT`, `F_CALL` |
| `D_` | Dimension table | `D_COUNCIL_MEMBER`, `D_CLIENT` |
| `B_` | Bridge table | `B_PROJECT_EXPERT` |
| `A_` | Aggregate table | `A_EMPLOYEE_SUMMARY` |

## ETL Architecture

Each table folder under `ETL_SF/` contains SQL scripts that the `bi_etl` engine executes. The engine resolves foreign-key dependency order automatically — Level 0 tables load first (no dependencies), then Level 1 (depends on Level 0), and so on.

### Sync Modes

- **Replace** — Truncate and full reload
- **Append** — Insert new rows only
- **Merge** — Upsert based on primary key

## Data Flow

```
Source Systems (GLGLIVE SQL Server, FUSION, Aurora PG, Azure SQL)
        │
        ▼
    STAGING_SF  (raw replicated data)
        │
        ▼
    ETL_SF      (transformed WARS fact/dim/bridge/agg tables)
        │
        ▼
    Looker      (dashboards and reports)
```

## Governance

- **CODEOWNERS:** `@glg/dre` — all PRs require DRE team review
- **PR Template:** Standardized change checklist via `PULL_REQUEST_TEMPLATE.md`
- All schema changes must also go through **SchemaTron** for ISO 27001 audit compliance

## Related Repositories

| Repo | Role |
|---|---|
| `bi_etl` | Python ETL engine that executes these SQL scripts |
| `bi_airflow_dags` | Airflow DAGs that schedule ETL runs (5×/day) |
| `SchemaTron` | DDL change management and audit trail |
| `bi_md` | Master Data tables that feed into WARS |
