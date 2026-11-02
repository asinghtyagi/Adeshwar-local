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

## Platform Architecture (High Level)

```
Source Systems
(GLGLIVE SQL Server, FUSION, Aurora PG, Azure SQL, Great Plains)
        │
        ▼
   STAGING_SF (Snowflake)
        │
        ▼
   bi_etl (Python engine) ◄── BI_vdw (SQL scripts) ◄── SchemaTron (DDL changes)
        │
        ▼
   WARS / MASTER_DATA (Snowflake)
        │
        ├──► Looker (dashboards)
        └──► Workday Prism (HR/Finance)

Orchestration: bi_airflow_dags (DAGs) → bi_jobs (Docker/ECR) → Airflow on AWS ECS
Cron Jobs: gds.clusterconfig.j01 (80+ ECS scheduled tasks)
Payments: nm-payment-account-job (Workday bank account sync)
```
