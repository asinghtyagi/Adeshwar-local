# Adeshwar-local

Personal knowledge base and documentation for GLG DRE (Data and Reporting Engineering) repositories.

## Repository Docs

| Repository | Description | Doc |
|---|---|---|
| `BI_vdw` | WARS data warehouse — 50+ SQL ETL scripts for Snowflake | [BI_vdw.md](./BI_vdw.md) |
| `DE_NewHireTraining` | DRE team new hire onboarding guide | [DE_NewHireTraining.md](./DE_NewHireTraining.md) |
| `SchemaTron` | Database schema change management (ISO 27001) | [SchemaTron.md](./SchemaTron.md) |
| `bi_airflow_dags` | All Airflow DAG definitions (27 DAGs) | [bi_airflow_dags.md](./bi_airflow_dags.md) |
| `bi_etl` | Custom Python ETL engine powering WARS loads | [bi_etl.md](./bi_etl.md) |
| `bi_jobs` | Airflow Docker container build & deploy | [bi_jobs.md](./bi_jobs.md) |
| `bi_md` | Master Data ETL for Workday Prism | [bi_md.md](./bi_md.md) |
| `gds.clusterconfig.j01` | ECS cron job cluster config (80+ jobs) | [gds_clusterconfig_j01.md](./gds_clusterconfig_j01.md) |
| `nm-payment-account-job` | NM bank account sync to Workday (AES decrypt + REST API) | [nm-payment-account-job.md](./nm-payment-account-job.md) |
| `PythonProject` | Personal scratch / experimentation workspace | [PythonProject.md](./PythonProject.md) |

## Platform Architecture

```
╔══════════════════════════════════════════════════════════════════════════╗
║                          SOURCE SYSTEMS                                  ║
║                                                                          ║
║  ┌─────────────┐  ┌──────────┐  ┌─────────────┐  ┌──────────────────┐  ║
║  │  GLGLIVE    │  │  FUSION  │  │  Aurora PG  │  │ Azure SQL /      │  ║
║  │ (SQL Server)│  │(SQL Svr) │  │  (AWS)      │  │ Great Plains     │  ║
║  └──────┬──────┘  └────┬─────┘  └──────┬──────┘  └────────┬─────────┘  ║
╚═════════╪══════════════╪═══════════════╪══════════════════╪═════════════╝
          │              │               │                   │
          └──────────────┴───────┬───────┘                   │
                                 │                           │
                    ┌────────────▼────────────┐              │
                    │    STAGING_SF           │              │
                    │    (Snowflake)          │              │
                    │    Raw replicated data  │              │
                    └────────────┬────────────┘              │
                                 │                           │
          ┌──────────────────────┘                           │
          │                                                  │
          │   ┌──────────────────────────────────────────┐   │
          │   │           REPO LAYER                     │   │
          │   │                                          │   │
          │   │  BI_vdw ──── SQL ETL scripts             │   │
          │   │     │                                    │   │
          │   │  bi_etl ─── Python engine (orchestrates) │   │
          │   │     │            ▲                       │   │
          │   │  SchemaTron ─── DDL change management    │   │
          │   │                                          │   │
          │   └──────────────────┬───────────────────────┘   │
          └──────────────────────┤                           │
                                 │                           │
╔════════════════════════════════╪═══════════════════════════╪═════════════╗
║                    SNOWFLAKE DATA WAREHOUSE                 │             ║
║                                 │                           │             ║
║  ┌──────────────────────────────▼──────┐  ┌────────────────▼───────────┐ ║
║  │            WARS                     │  │       MASTER_DATA          │ ║
║  │  F_ Fact  │ D_ Dim  │ A_ Aggregate  │  │  Workday-ready data        │ ║
║  │  B_ Bridge tables                   │  │  (bi_md repo manages this) │ ║
║  └──────┬──────────────────────────────┘  └──────────────┬─────────────┘ ║
╚═════════╪════════════════════════════════════════════════╪════════════════╝
          │                                                │
          ▼                                                ▼
   ┌─────────────┐                               ┌────────────────────┐
   │   Looker    │                               │   Workday Prism    │
   │  Dashboards │                               │   HR / Finance     │
   └─────────────┘                               └────────────────────┘


══════════════════════  ORCHESTRATION LAYER  ══════════════════════════════

  bi_airflow_dags ──► bi_jobs (Docker/ECR) ──► Airflow 3 on AWS ECS
        │
        ├── wars_run_etl          (5× / day — main WARS pipeline)
        ├── staging_replicate_*   (data ingestion from source systems)
        ├── revelio / library / loads / vendorgraphic refreshes
        └── API integrations      (Zoom, Kronos, Creatio, ZenHub …)

  gds.clusterconfig.j01 ──► 80+ ECS cron jobs
        ├── Invoicing             (invoicerizer-*)
        ├── Payments              (payprocess-*, cm-payment-*)
        ├── Compliance            (compliance-jobs-*)
        ├── Surveys               (surveys-*)
        ├── AI / ML               (ai-intake-*, ai-workers)
        └── Data / ETL            (datahub-*, capiq-*, revelio-*)

  nm-payment-account-job ──► Workday REST API
        └── AES-decrypt GP bank data → POST bank account updates
```
