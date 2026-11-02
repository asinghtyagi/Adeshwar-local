# bi_airflow_dags — Airflow DAG Definitions

**GitHub:** [glg/bi_airflow_dags](https://github.com/glg/bi_airflow_dags)

## Overview

`bi_airflow_dags` contains all **Apache Airflow DAG definitions** for the GLG data platform. Merging a PR to `main` triggers an automatic Docker build and deployment to the production Airflow environment — there is no manual deploy step.

## Tech Stack

| Layer | Technology |
|---|---|
| Orchestration | Apache Airflow 3 |
| Language | Python 3 |
| Container | Docker |
| Registry | AWS ECR |
| CI/CD | GitHub Actions |

## Repository Structure

```
bi_airflow_dags/
├── wars_run_etl.py
├── wars_glgdev_run_etl.py
├── staging_replicate_data.py
├── staging_replicate_data_daily.py
├── revelio_refresh.py
├── events_library_refresh.py
├── loads_company_refresh.py
├── loads_post_process.py
├── vendorgraphic_etl.py
├── vendorgraphic_lead_identification.py
├── looker_warehouse_scaling.py
├── looker_user_cleanup.py
├── expert_cleanup.py
├── create_ddl_objects.py
├── wars_archive.py
├── wars_cleanup_failed_batches.py
├── wars_etl_code_refresh.py
├── pricing_forecast.py
├── creatio_api_tasks.py
├── kronos_api_tasks.py
├── zoom_api_tasks.py
├── zenhub_api_tasks.py
├── mck_invoice_tasks.py
├── firm_prospects_api_tasks.py
├── refresh_dags.py
├── get_env_dag.py
└── config_sf_glgdev.py
```

## DAG Catalog

### Core WARS ETL

| DAG | Schedule | Purpose |
|---|---|---|
| `wars_run_etl` | `0 0,4,12,16,20 * * *` (5×/day) | Main WARS data warehouse ETL — the central pipeline |
| `wars_glgdev_run_etl` | As needed | WARS ETL against GLGDEV (dev/test environment) |
| `wars_archive` | Daily | Archive old WARS data |
| `wars_cleanup_failed_batches` | Daily | Remove stale failed batch records |
| `wars_etl_code_refresh` | On demand | Pull latest SQL ETL code from git to the Airflow server |

### Staging & Replication

| DAG | Purpose |
|---|---|
| `staging_replicate_data` | Replicates source system data into Snowflake STAGING |
| `staging_replicate_data_daily` | Daily variant of staging replication |

### Data Refresh

| DAG | Purpose |
|---|---|
| `revelio_refresh` | Refreshes Revelio labor market intelligence data |
| `events_library_refresh` | Library and events analytics refresh |
| `loads_company_refresh` | Loads company data refresh |
| `loads_post_process` | Post-processing for the Loads database |
| `vendorgraphic_etl` | VendorGraphic company data ETL |
| `vendorgraphic_lead_identification` | Lead identification pipeline from VendorGraphic |

### Infrastructure & Ops

| DAG | Purpose |
|---|---|
| `looker_warehouse_scaling` | Auto-scales the Snowflake warehouse used by Looker |
| `looker_user_cleanup` | Removes deprovisioned Looker user accounts |
| `expert_cleanup` | Cleans up stale council member / expert data |
| `create_ddl_objects` | Creates or refreshes DDL objects in Snowflake |
| `refresh_dags` | Utility to refresh DAG definitions in Airflow |
| `get_env_dag` | Diagnostics DAG — dumps environment info |

### External API Integrations

| DAG | Source System | Purpose |
|---|---|---|
| `creatio_api_tasks` | Creatio CRM | Pull CRM pipeline data |
| `kronos_api_tasks` | Kronos | Workforce management data |
| `zoom_api_tasks` | Zoom | Call/meeting data extraction |
| `zenhub_api_tasks` | ZenHub | Engineering project metrics |
| `mck_invoice_tasks` | McKinsey | Invoice processing pipeline |
| `firm_prospects_api_tasks` | Internal | Firm prospect data sync |

### Analytics

| DAG | Purpose |
|---|---|
| `pricing_forecast` | Pricing forecast data pipeline |

### Config

| File | Purpose |
|---|---|
| `config_sf_glgdev.py` | Snowflake GLGDEV connection configuration shared across DAGs |

## Deployment Flow

```
PR merged to main
        │
        ▼
GitHub Actions: build Docker image
        │
        ▼
Push image to AWS ECR
        │
        ▼
Airflow server auto-pulls new DAGs
        │
        ▼
Scheduler picks up updated DAG definitions
```

## Key Notes

- All DAG files live at the **root level** — no subdirectories — making it easy for Airflow to discover them
- The main WARS ETL (`wars_run_etl`) is the highest-criticality DAG; failures page on-call
- `wars_etl_code_refresh` must run before a WARS ETL run to pick up new SQL from `BI_vdw`

## Related Repositories

| Repo | Role |
|---|---|
| `bi_jobs` | Builds the Docker container that hosts these DAGs |
| `bi_etl` | Python ETL engine invoked by the WARS ETL DAGs |
| `BI_vdw` | SQL scripts executed during the WARS ETL runs |
