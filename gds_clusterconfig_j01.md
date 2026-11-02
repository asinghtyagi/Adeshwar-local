# gds.clusterconfig.j01 — ECS Cron Job Cluster Configuration

**GitHub:** [glg/gds.clusterconfig.j01](https://github.com/glg/gds.clusterconfig.j01)

## Contents

- [Overview](#overview)
- [Tech Stack](#tech-stack)
- [Repository Structure](#repository-structure)
- [Job Catalog](#job-catalog)
- [Config File Roles](#config-file-roles)
- [Cluster Properties](#cluster-properties)
- [Related Repositories](#related-repositories)

## Overview

`gds.clusterconfig.j01` contains the **GDS (GLG Deployment System) ECS cluster configuration** for the `j01` internal-only jobs cluster. This cluster hosts all scheduled cron jobs and background processing tasks across the GLG platform. Unlike application clusters, `j01` has **no ALB, no DNS, and receives no inbound traffic** — it is exclusively for outbound/scheduled work.

## Tech Stack

| Layer | Technology |
|---|---|
| Container Orchestration | AWS ECS (Elastic Container Service) |
| Job Configuration | JSON (`orders/`, `policy.json`, `secrets.json`) |
| Logging | SumoLogic |
| Health Checks | `hcall.sh` |

## Repository Structure

```
gds.clusterconfig.j01/
├── zero-trust-map.json             ← Security/network access mapping
├── hcall.sh                        ← Health check utility script
├── CODEOWNERS
└── <service-name>/                 ← One directory per deployed job/service
    ├── orders/                     ← ECS task definition and scheduling config
    ├── policy.json                 ← IAM policy for this job
    └── secrets.json                ← Secret references (AWS Secrets Manager ARNs)
```

Every subdirectory is an independently deployed cron job or background service. There are **80+ jobs** in this cluster.

## Job Catalog

### Invoicing & Billing

| Job | Purpose |
|---|---|
| `invoicerizer-auto-invoice` | Automatically generate invoices |
| `invoicerizer-bundle-reload` | Reload bundled invoice data |
| `invoicerizer-minimum-spend` | Process minimum spend commitments |
| `invoicerizer-auto-inv-oos` | Auto-invoice out-of-scope items |
| `invoicerizer-auto-sales` | Auto-process sales invoices |
| `invoicerizer-fx-rate` | Update FX rates for invoice calculations |
| `invoicerizer-cm-expenses` | Process council member expenses |
| `invoicerizer-update-tiers` | Update client billing tiers |

### Payments & Network Members

| Job | Purpose |
|---|---|
| `payprocess-upload` | Upload payment batches to banking system |
| `payprocess-tmf` | Process TMF (Transfer Money Flow) payments |
| `payprocess-tmf-jpy` | Process TMF payments in JPY |
| `payprocess-reap-tmf-download` | Download/reconcile TMF payment results |
| `cm-pay-invoices` | Pay council member invoices |
| `cm-payment-nra-followup` | Follow up on NRA (Non-Resident Alien) payment issues |
| `cm-payment-referral-paid` | Mark referral payments as paid |
| `payment-monitors-*` | Various payment monitoring and alerting jobs |

### Compliance

| Job | Purpose |
|---|---|
| `compliance-jobs-gdpr-clean` | GDPR data erasure and anonymization |
| `compliance-jobs-q2c-ai` | AI-assisted Q2C compliance checks |
| `compliance-jobs-q2c-daily` | Daily Q2C (Quote-to-Cash) compliance runs |
| `compliance-jobs-scmp-daily` | Daily SCMP compliance processing |
| `compliance-jobs-expiry` | Handle expiring compliance records |
| `compliance-jobs-pay-auto` | Auto-process compliance-related payments |
| `compliance-jobs-dnc-notifier` | Do-not-contact list notifications |
| `compliance-jobs-hp-*` | HP-specific compliance workflows |
| `compliance-jobs-c-trials` | Client trial compliance tracking |

### Surveys

| Job | Purpose |
|---|---|
| `surveys-autoclose` | Auto-close surveys past their end date |
| `surveys-decipher-sync-more` | Incremental sync with Decipher survey platform |
| `surveys-decipher-survey-sync` | Full sync with Decipher |
| `surveys-send2pb` | Send survey results to PitchBook |
| `surveys-remind-*` | Various survey reminder email jobs |

### Data / ETL (Non-Airflow)

| Job | Purpose |
|---|---|
| `datahub-workday-etl` | Workday data ETL to DataHub |
| `datahub-great-plains-etl` | Great Plains financial data ETL |
| `capiq-refresh` | Capital IQ data refresh |
| `revelio-*` | Revelio labor market data jobs |
| `ci-snowflake-sa-*` | Snowflake service account CI/validation jobs |
| `detailed-usage-weekly` | Weekly usage reporting |
| `detailed-usage-monthly` | Monthly usage reporting |
| `detailed-usage-quarterly` | Quarterly usage reporting |

### AI / ML

| Job | Purpose |
|---|---|
| `ai-intake-abandoned-prod` | Process abandoned AI intake requests |
| `ai-intake-*` | Various AI intake pipeline jobs |
| `ai-workers` | Background AI worker processes |

### Events & Library

| Job | Purpose |
|---|---|
| `events-rec-main-*` | Main event recommendation jobs |
| `event-rec-jobs-*` | Event recommendation sub-jobs |
| `event-transcript-reminder` | Reminder emails for outstanding event transcripts |

### Salesforce / CRM

| Job | Purpose |
|---|---|
| `sfdc-idx-*` | Salesforce indexing jobs |

### Workday

| Job | Purpose |
|---|---|
| `workday-*` | Various Workday HR/Finance sync jobs |

### Reconciliation & Finance

| Job | Purpose |
|---|---|
| `reconciler-auto-approve` | Auto-approve reconciliation items meeting criteria |
| `fraud-api-credit-check` | Credit check calls via fraud API |
| `change-tracker-fanout-1-min` | 1-minute change event fanout |
| `mcm-*` | MCM (client management) background jobs |

## Config File Roles

### `orders/`
ECS task definition and scheduling configuration — container image, CPU/memory allocation, environment variables, cron schedule expression.

### `policy.json`
IAM policy that grants this job only the AWS permissions it needs (least-privilege). Typically includes access to specific S3 buckets, Secrets Manager paths, SQS queues, and other AWS resources the job reads/writes.

### `secrets.json`
References to AWS Secrets Manager ARNs that are injected as environment variables at runtime. Actual secret values are never stored in the repo.

### `zero-trust-map.json`
Maps each service to its allowed network destinations — enforces zero-trust networking within the cluster.

## Cluster Properties

- **No inbound traffic** — no load balancer, no DNS record, no open ports
- **Outbound-only** — jobs make API calls, write to databases, and publish events
- **SumoLogic** — all container stdout/stderr forwarded for log aggregation and alerting
- **80+ active jobs** — highest job density of any GLG ECS cluster

## Related Repositories

| Repo | Role |
|---|---|
| `bi_airflow_dags` | Airflow-based ETL jobs (separate from this ECS cluster) |
| `bi_jobs` | Airflow container deployment (separate infrastructure) |
| `nm-payment-account-job` | NM payment job — has its own ECR image, may run here |
