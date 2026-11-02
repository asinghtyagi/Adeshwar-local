# DE_NewHireTraining — DRE New Hire Onboarding Guide

**GitHub:** [asinghtyagi/DE_NewHireTraining](https://github.com/asinghtyagi/DE_NewHireTraining) &nbsp;·&nbsp; **Last updated:** 2026-11-02

## Contents

- [Overview](#overview)
- [Tech Stack](#tech-stack)
- [Repository Structure](#repository-structure)
- [Guide Contents](#guide-contents)
- [Who Should Read This](#who-should-read-this)
- [Related Repositories](#related-repositories)

## Overview

`DE_NewHireTraining` is the official onboarding documentation repository for the GLG **Data and Reporting Engineering (DRE)** team. It contains a comprehensive guide that walks new team members through everything they need to know: team structure, tooling, access provisioning, workflows, and coding standards.

## Tech Stack

| Layer | Technology |
|---|---|
| Format | Markdown |
| Source Control | Git / GitHub |

## Repository Structure

```
DE_NewHireTraining/
└── NEW_HIRE_TRAINING.md    ← ~400-line comprehensive onboarding guide
```

## Guide Contents

### 1. Team Overview
- DRE team mission and responsibilities within GLG
- Org structure and key stakeholders

### 2. Access & Roles
- **Sailpoint** — identity governance (request access here)
- **GitHub** — repo access via `@glg/dre` team membership
- Required tool accounts: Snowflake, Airflow, DataGrip/IDE, AWS Console

### 3. Tech Stack Overview
| Tool | Purpose |
|---|---|
| Snowflake | Cloud data warehouse (WARS, STAGING, FIVETRAN, LOADS, etc.) |
| Apache Airflow 3 | Workflow orchestration (hosted on AWS ECS) |
| SQL Server | Source system (GLGLIVE, FUSION) |
| Aurora PostgreSQL | Source system for NM/payment data |
| Azure SQL | Source system for various GLG services |
| Docker / AWS ECR | Container build and deploy |
| DataGrip | Primary SQL IDE |

### 4. Key Repositories
| Repo | Purpose |
|---|---|
| `BI_vdw` | WARS data warehouse SQL ETL |
| `bi_etl` | Python ETL engine |
| `bi_airflow_dags` | Airflow DAG definitions |
| `bi_jobs` | Airflow Docker container build/deploy |
| `SchemaTron` | DB schema change management |
| `bi_md` | Master Data ETL |
| `gds.clusterconfig.j01` | ECS cron job configurations |
| `nm-payment-account-job` | NM payment Workday sync |

### 5. Local Dev Setup
- Snowflake key-pair authentication setup
- DataGrip connection profiles
- Python virtual environment setup (`bi_etl`)
- Docker local Airflow (`bi_jobs/af3-local/`)

### 6. ETL / Airflow Workflows
- How the WARS ETL pipeline runs (5×/day schedule)
- DAG anatomy and how to add a new DAG
- How ETL_SF scripts are picked up and executed by `bi_etl`

### 7. SchemaTron Process
- How to raise a schema change PR
- Required reviewers and approval gates
- ISO 27001 compliance requirements

### 8. Coding Standards
- SQL formatting conventions
- Python style guidelines
- Commit message format
- PR description requirements

### 9. Incident Response
- On-call rotation details
- How to investigate a failed ETL batch
- Airflow failure triage checklist

### 10. Quick-Start Checklist
- Day 1 access requests
- Day 1–5 walkthrough tasks
- First PR guidance

## Who Should Read This

Every new DRE team member should work through `NEW_HIRE_TRAINING.md` during their first week. It is also a useful reference for anyone rejoining after a long absence or transitioning to a new area of the platform.

## Related Repositories

| Repo | Why Relevant |
|---|---|
| `SchemaTron` | Contains `CLAUDE.md` with additional coding-standard guidance |
| `bi_airflow_dags` | First hands-on repo for most new hires |
| `bi_etl` | Core engine all DRE engineers should understand |
