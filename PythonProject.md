# PythonProject — Personal Scratch / Experimentation Workspace

**GitHub:** No remote — local only

## Contents

- [Overview](#overview)
- [Tech Stack](#tech-stack)
- [Repository Structure](#repository-structure)
- [Key Files](#key-files)
- [Virtual Environments](#virtual-environments)
- [Notes](#notes)
- [Related Repositories](#related-repositories)

## Overview

`PythonProject` is a **personal scratch workspace** used for local experimentation, learning, prototyping, and one-off scripts. It is not a production repository and does not feed any automated pipeline. It serves as a sandboxed environment where new ideas, SQL tests, and tooling experiments can be developed before being promoted to production repos.

## Tech Stack

| Layer | Technology |
|---|---|
| Language | Python 3.9, Python 3.13 |
| Data Warehouse | Snowflake (connector) |
| Dependencies | NumPy, Pandas, bi_etl (local copy) |
| Virtual Envs | `env/` (Python 3.9), `.venv/` (Python 3.13) |

## Repository Structure

```
PythonProject/
├── SamplefileFor_DAG.py                ← Sample Airflow DAG template
├── WD_Submit_Supplier.py               ← Workday supplier submission script
├── test_sql.sql                        ← Ad-hoc SQL test queries
├── snowflake_learning_plan.md          ← Personal Snowflake learning notes
├── agentic_workflow_learning_plan.md   ← Notes on agentic / AI workflow patterns
├── bi_md_audit.md                      ← bi_md codebase audit notes
├── bi_etl/                             ← Local clone of bi_etl for experimentation
├── env/                                ← Python 3.9 virtual environment
└── .venv/                              ← Python 3.13 virtual environment
```

## Key Files

### `SamplefileFor_DAG.py`
A template/reference Airflow DAG file. Useful for understanding the standard DAG structure used across `bi_airflow_dags` before writing a production DAG.

### `WD_Submit_Supplier.py`
A standalone script for submitting supplier data to the Workday API. Likely an early prototype of logic later formalized in `nm-payment-account-job`.

### `test_sql.sql`
Ad-hoc SQL queries used for testing Snowflake queries, validating ETL results, or exploring data before writing production SQL.

### Learning Plans

| File | Topic |
|---|---|
| `snowflake_learning_plan.md` | Snowflake SQL features, performance tuning, clustering keys, micro-partitions |
| `agentic_workflow_learning_plan.md` | AI agent patterns, orchestration, tool use, multi-step workflows |

### `bi_md_audit.md`
Notes and findings from a manual audit of the `bi_md` codebase — useful reference when making changes to that repo.

### `bi_etl/`
A local copy of the `bi_etl` engine used for running experiments without affecting the canonical repo. Enables testing ETL changes locally against GLGDEV before PRing.

## Virtual Environments

Two Python environments are maintained:

| Directory | Python Version | Use Case |
|---|---|---|
| `env/` | Python 3.9 | Legacy scripts and compatibility testing |
| `.venv/` | Python 3.13 | Current development environment |

## Notes

- Nothing in this repo is deployed to production directly
- Scripts here may graduate to `nm-payment-account-job`, `bi_airflow_dags`, or `bi_etl` once proven
- The `bi_etl/` subdirectory should be kept in sync with the canonical `bi_etl` repo when testing local changes

## Related Repositories

| Repo | Relationship |
|---|---|
| `bi_etl` | Canonical version of the local `bi_etl/` copy |
| `bi_airflow_dags` | Production home for DAGs prototyped here |
| `nm-payment-account-job` | Production home for Workday scripts prototyped here |
| `bi_md` | Audited in `bi_md_audit.md` |
