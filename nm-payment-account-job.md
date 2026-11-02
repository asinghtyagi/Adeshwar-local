# nm-payment-account-job — Network Member Payment Account Sync

**GitHub:** [glg/nm-payment-account-job](https://github.com/glg/nm-payment-account-job) &nbsp;·&nbsp; **Last updated:** 2026-11-02

## Contents

- [Overview](#overview)
- [Tech Stack](#tech-stack)
- [Repository Structure](#repository-structure)
- [Main Script Flow](#main-script-flow)
- [Environment Variables](#environment-variables)
- [Workday XML Templates](#workday-xml-templates-wd)
- [Security Notes](#security-notes)
- [CI/CD Flow](#cicd-flow)
- [Related Repositories](#related-repositories)

## Overview

`nm-payment-account-job` is an automated job that syncs **Network Member (NM) bank account and payment data** from GLG's internal systems (GLGLIVE SQL Server + Great Plains) into **Workday** via the Workday REST API. It handles AES-CBC decryption of encrypted bank credentials before submission and logs all results back to Snowflake.

## Tech Stack

| Layer | Technology |
|---|---|
| Language | Python 3.12 |
| Data Processing | Pandas |
| SQL Server Driver | SQLAlchemy + PyODBC |
| HTTP Client | Requests (OAuth2 token refresh) |
| Encryption | PyCryptodome (AES-CBC decryption) |
| Secret Management | AWS Secrets Manager + python-dotenv |
| Email | smtplib |
| Container | Docker |
| Registry | AWS ECR |
| CI/CD | GitHub Actions |
| Orchestration | Airflow 3 (local dev config included) |

## Repository Structure

```
nm-payment-account-job/
├── WD_xml_dataset_from_gp_one_by_one.py   ← Main script (~560 lines)
├── Dockerfile                               ← Production container image
├── requirements.txt
├── .env.example                             ← Environment variable template
├── .env                                     ← Actual secrets (git-ignored)
├── CODEOWNERS
├── dags/                                    ← Airflow DAG definitions
├── config/
│   └── airflow.cfg                          ← Local Airflow config
├── plugins/
├── wd/                                      ← Workday XML payload templates
│   ├── WD_NM_Supplier_1st_part.txt
│   ├── WD_NM_Supplier_2nd_part.txt
│   ├── WD_NM_Supplier_3rd_part.txt
│   ├── WD_Remit_To_ACC.txt
│   ├── WD_Remit_to_NO_ACC.txt
│   └── GLG - Bank_Account_Validation 1.xlsx
└── .github/workflows/
    └── build-and-push-to-ecr.yml
```

## Main Script Flow

`WD_xml_dataset_from_gp_one_by_one.py` executes the following pipeline for each run:

```
Step 1: Fetch supplier list from Snowflake (via Epiquery)
        └─ Suppliers that are ready/pending Workday bank account update
        │
        ▼
Step 2: Fetch NM payment records from GLGLIVE SQL Server (via Epiquery)
        │
        ▼
Step 3: Bulk-query Great Plains (GP) SQL Server for payment account details
        └─ Account numbers, IBANs, SWIFT codes, routing numbers (all encrypted)
        │
        ▼
Step 4: Join three DataFrames on supplier_id
        │
        ▼
Step 5: For each record — decrypt AES-CBC encrypted bank fields
        ├─ Bank account number  (key/IV from AWS Secrets)
        ├─ IBAN
        ├─ SWIFT code
        └─ Routing number
        │
        ▼
Step 6: Build Workday XML payloads from wd/ template files
        ├─ WD_NM_Supplier parts (1, 2, 3) → combined supplier XML
        ├─ WD_Remit_To_ACC.txt   → if account exists
        └─ WD_Remit_to_NO_ACC.txt → if no account
        │
        ▼
Step 7: POST to Workday REST API (OAuth2 with refresh token)
        ├─ Bank account update endpoint
        └─ Remit-to update endpoint
        │
        ▼
Step 8: Log results to Snowflake (via Epiquery)
        │
        ▼
Step 9: Email error report via SMTP (if any failures)
```

## Environment Variables

### Workday API

| Variable | Purpose |
|---|---|
| `WD_REFRESH_TOKEN` | OAuth2 refresh token for Workday API auth |
| `WD_BASE_URL` | Workday tenant base URL |
| `WD_TOKEN_URL` | OAuth2 token endpoint |
| `WD_CLIENT_ID` | OAuth2 client ID |
| `WD_CLIENT_SECRET` | OAuth2 client secret |

### Encryption Keys (AES-CBC)

| Variable | Purpose |
|---|---|
| `CRYPTO_BANKACCOUNTNUMBER_KEY` | AES key for bank account number |
| `CRYPTO_BANKACCOUNTNUMBER_IV` | AES IV for bank account number |
| `CRYPTO_BANKSWIFTNUMBER_KEY` | AES key for SWIFT/BIC code |
| `CRYPTO_BANKSWIFTNUMBER_IV` | AES IV for SWIFT/BIC code |

### Database Connections

| Variable | Purpose |
|---|---|
| `DB_GP_HOST` | Great Plains SQL Server hostname |
| `DB_GP_NAME` | Great Plains database name |
| `DB_GP_USER` | Great Plains SQL Server username |
| `DB_GP_PASSWORD` | Great Plains SQL Server password |
| `EPIQUERY_GLGLIVE` | Epiquery endpoint for GLGLIVE SQL Server |
| `EPIQUERY_SNOWFLAKE` | Epiquery endpoint for Snowflake |

### Email

| Variable | Purpose |
|---|---|
| `SMTP_SERVER` | SMTP server for error report emails |

### AWS Secrets Fallback

Encryption keys can also be loaded from AWS Secrets Manager at runtime:
- `prod/NEW_CRYPTO_BANKACCOUNTNUMBER`
- `prod/NEW_CRYPTO_BANKSWIFTNUMBER`

## Workday XML Templates (`wd/`)

| File | Purpose |
|---|---|
| `WD_NM_Supplier_1st_part.txt` | Opening XML for Workday supplier payload |
| `WD_NM_Supplier_2nd_part.txt` | Middle section with supplier fields |
| `WD_NM_Supplier_3rd_part.txt` | Closing XML for supplier payload |
| `WD_Remit_To_ACC.txt` | Remit-to XML when bank account is present |
| `WD_Remit_to_NO_ACC.txt` | Remit-to XML when no bank account |
| `GLG - Bank_Account_Validation 1.xlsx` | Validation reference for bank account formats |

## Security Notes

- Bank credentials are **never stored in plaintext** — all values from Great Plains are AES-CBC encrypted at rest
- Decryption happens in-memory only during runtime
- Workday API uses OAuth2 with short-lived access tokens (refreshed from `WD_REFRESH_TOKEN`)
- `.env` is git-ignored; use `.env.example` as a template
- AWS Secrets Manager is the production secret store

## CI/CD Flow

```
PR merged to main
        │
        ▼
GitHub Actions: build Docker image
        │
        ▼
Push to AWS ECR
        │
        ▼
ECS task or Airflow DAG pulls latest image and runs on schedule
```

## Related Repositories

| Repo | Role |
|---|---|
| `bi_md` | Source for NM payment data in Snowflake (`NM_PAYMENT` module) |
| `SchemaTron` | Manages any SQL Server schema changes in GLGLIVE that this job depends on |
| `gds.clusterconfig.j01` | ECS cluster where this job may be deployed as a cron |
| `bi_airflow_dags` | Alternative orchestration if triggered via Airflow |
