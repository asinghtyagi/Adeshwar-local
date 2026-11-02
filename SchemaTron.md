# SchemaTron — Database Schema Change Management

## Overview

`SchemaTron` is GLG's **central change management system** for all database schema, code, and data changes. Every DDL statement, DML script, stored procedure, view, and function change across the entire GLG data platform must go through a pull request here. This process is mandatory for **ISO 27001 audit compliance**.

## Tech Stack

| Layer | Technology |
|---|---|
| SQL Dialects | T-SQL (SQL Server), Snowflake SQL, PostgreSQL, MySQL |
| Container | Docker |
| CI/CD | GitHub Actions |
| AI Integration | Claude Code + custom slash commands |

## Repository Structure

```
SchemaTron/
├── CLAUDE.md                       ← Claude Code guidance (file naming, PR rules)
├── Dockerfile
├── entrypoint.sh
├── freetds.conf                    ← FreeTDS config for SQL Server connections
├── Snowflake/                      ← 40+ Snowflake databases
│   ├── WARS/
│   ├── FIVETRAN/
│   ├── STAGING/
│   ├── SEARCH_ANALYSIS/
│   ├── REVELIO/
│   ├── DATA_DICTIONARY/
│   ├── MASTER_DATA/
│   ├── ANALYTICS/
│   └── ... (35+ more databases)
├── SQLServer/                      ← SQL Server databases
│   ├── FUSION/
│   ├── GLGLIVE/
│   ├── FIVETRAN/
│   ├── TRANSCRIPTS/
│   ├── CAPIQ/
│   └── ACTIVITY_MATRIX/
├── Azure/                          ← Azure SQL databases
│   ├── timeline-events/
│   ├── client-media/
│   ├── tracking/
│   └── cm_data_ops_queue/
├── AWS/                            ← Aurora PostgreSQL, Aurora MySQL, S3 Tables
├── Snippets/                       ← IDE code snippets
│   ├── DataGrip/
│   ├── SSMS/
│   └── AzureDataStudio/
├── lib/
├── pkg/
└── change-control/
```

## Databases Managed

| Platform | Count | Examples |
|---|---|---|
| Snowflake | 40+ | WARS, FIVETRAN, STAGING, LOADS, SEARCH_ANALYSIS, REVELIO, MASTER_DATA, ANALYTICS |
| SQL Server | 25+ | FUSION, GLGLIVE, FIVETRAN, TRANSCRIPTS, CAPIQ, ACTIVITY_MATRIX |
| Aurora PostgreSQL | 15+ | Various microservice DBs |
| Aurora MySQL | Several | Various microservice DBs |
| Azure SQL | 4+ | timeline-events, client-media, tracking, cm_data_ops_queue |

## Claude Code Integration

SchemaTron has first-class Claude Code support via a `CLAUDE.md` file and custom slash commands.

### CLAUDE.md Guidance
- File naming rules for new scripts
- Transaction template requirements (BEGIN/COMMIT/ROLLBACK wrappers)
- PR description requirements for change control

### Custom Slash Commands (`.claude/skills/`)

| Command | Purpose |
|---|---|
| `/convert-nm-to-jpy-rate` | Generate JPY FX rate conversion scripts |
| `/gdpr-redact-nm` | Generate GDPR data redaction scripts for NM records |
| `/redact-nm-bank-info` | Redact Network Member bank account information |
| `/enable-change-tracking` | Generate SQL Server change-tracking enablement scripts |
| `/schematron_pr` | Automate PR creation with correct change-control metadata |

## Change Control Process

```
1. Engineer writes DDL/DML script
        │
        ▼
2. PR opened in SchemaTron (correct folder per DB platform)
        │
        ▼
3. PR reviewed by @glg/dre + DB owner
        │
        ▼
4. Approved PR merged → script applied to target database
        │
        ▼
5. Change recorded in audit log (ISO 27001 evidence)
```

## Script Organization Convention

Each change script lives in a path that reflects:
```
SchemaTron/<Platform>/<Database>/<Schema>/<ScriptName>.sql
```

Example:
```
SchemaTron/Snowflake/WARS/META/add_batch_retry_column.sql
SchemaTron/SQLServer/GLGLIVE/dbo/create_expert_audit_table.sql
```

## IDE Snippets

The `Snippets/` directory contains pre-built templates for common patterns (transaction wrappers, index creation, view updates) for DataGrip, SSMS, and Azure Data Studio — reducing boilerplate and enforcing consistent style.

## Related Repositories

| Repo | Relationship |
|---|---|
| `BI_vdw` | WARS SQL changes require a matching SchemaTron DDL PR |
| `bi_md` | Master Data DDL changes go through SchemaTron |
| `bi_etl` | ETL engine schema (meta tables) managed here |
