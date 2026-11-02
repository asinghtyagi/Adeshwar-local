# bi_jobs — Airflow Docker Container Build & Deploy

**GitHub:** [glg/bi_jobs](https://github.com/glg/bi_jobs) &nbsp;·&nbsp; **Last updated:** 2026-11-02

## Contents

- [Overview](#overview)
- [Tech Stack](#tech-stack)
- [Repository Structure](#repository-structure)
- [CI/CD Flow](#cicd-flow)
- [Production Docker Image](#production-docker-image)
- [Local Development Setup](#local-development-setup)
- [Deployment Scripts](#deployment-scripts)
- [requirements.txt (Key Dependencies)](#requirementstxt-key-dependencies)
- [Related Repositories](#related-repositories)

## Overview

`bi_jobs` is the **infrastructure repository** for GLG's Airflow 3 deployment. It builds the production Docker container image, pushes it to AWS ECR, and deploys it via AWS CodeDeploy. It also contains a full local development environment so engineers can run Airflow on their laptop.

## Tech Stack

| Layer | Technology |
|---|---|
| Container Runtime | Docker |
| Orchestration | Apache Airflow 3 |
| Registry | AWS ECR |
| Deployment | AWS CodeDeploy (`appspec.yml`) |
| CI/CD | GitHub Actions |
| Local Dev | Docker Compose |

## Repository Structure

```
bi_jobs/
├── Dockerfile                          ← Production Airflow image definition
├── docker-compose-prod.yml             ← Production compose config
├── requirements.txt                    ← Python packages installed into the image
├── appspec.yml                         ← AWS CodeDeploy deployment specification
├── CODEOWNERS
├── continuous-deployment-scripts/
│   ├── start-airflow-server            ← Shell: start Airflow services
│   ├── stop-airflow-server             ← Shell: gracefully stop Airflow
│   └── get-latest-source               ← Shell: pull latest code before deploy
├── af3-local/                          ← Local development environment
│   ├── Dockerfile-local                ← Local dev image (lighter than prod)
│   ├── docker-compose-local.yml        ← Local compose (scheduler + webserver + DB)
│   ├── requirements.txt                ← Local-only Python deps
│   ├── dags/                           ← Sample/test DAGs for local runs
│   │   ├── wars_run_etl 1.py
│   │   └── AST_TestDAG.py
│   ├── config/
│   │   └── airflow.cfg                 ← Local Airflow configuration
│   ├── plugins/
│   ├── repos/                          ← Mount point for checked-out repos (bi_etl, etc.)
│   ├── qa_repos/
│   └── README.md                       ← Local setup instructions
└── .github/workflows/
    └── build-and-push-to-ecr.yml       ← CI pipeline
```

## CI/CD Flow

```
PR merged to main
        │
        ▼
GitHub Actions: build-and-push-to-ecr.yml
        │  - docker build -f Dockerfile .
        │  - docker tag + push to AWS ECR
        ▼
AWS CodeDeploy (appspec.yml)
        │  - BeforeInstall: stop-airflow-server
        │  - Install:       get-latest-source
        │  - AfterInstall:  start-airflow-server
        ▼
Airflow running with new image on EC2/ECS
```

> A manual approval gate is required before CodeDeploy proceeds to production.

## Production Docker Image

The `Dockerfile` builds an Airflow 3 image that:
1. Installs all Python dependencies from `requirements.txt` (includes `bi_etl`, Snowflake connector, PyODBC, etc.)
2. Configures the Airflow executor and metadata database
3. Mounts the `bi_airflow_dags` repo so DAGs are available to the scheduler

## Local Development Setup

The `af3-local/` directory provides a complete local Airflow 3 environment using Docker Compose.

### Quick Start

```bash
cd af3-local/

# Start local Airflow (webserver + scheduler + postgres metadata DB)
docker compose -f docker-compose-local.yml up

# Access the UI
open http://localhost:8080
# Default credentials: airflow / airflow
```

### Connecting Local repos
Mount your local `bi_etl` and `bi_airflow_dags` checkouts into `repos/` so the local scheduler picks them up — see `af3-local/README.md` for details.

## Deployment Scripts

| Script | What It Does |
|---|---|
| `start-airflow-server` | Starts Airflow webserver and scheduler services |
| `stop-airflow-server` | Gracefully drains and stops running tasks, then shuts down services |
| `get-latest-source` | Pulls the latest `bi_airflow_dags` code to the deploy host |

## `requirements.txt` (Key Dependencies)

The production image includes:
- `apache-airflow` (pinned to Airflow 3.x)
- `bi_etl` (installed from local wheel or git)
- `snowflake-connector-python`
- `snowflake-sqlalchemy`
- `pyodbc`
- `pandas`
- `boto3` (AWS SDK — Secrets Manager, ECR auth)

## Related Repositories

| Repo | Role |
|---|---|
| `bi_airflow_dags` | DAG files that this container serves |
| `bi_etl` | Python engine installed into this container |
| `gds.clusterconfig.j01` | ECS-based cron jobs (separate from this Airflow deployment) |
