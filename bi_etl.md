# bi_etl — Python ETL Engine

**GitHub:** [glg/bi_etl](https://github.com/glg/bi_etl)

## Overview

`bi_etl` is the **custom Python ETL engine** that powers all WARS data warehouse loads. It orchestrates the SQL scripts in `BI_vdw` by resolving foreign-key dependency order, managing batch tracking, and executing loads against Snowflake. It is the core runtime for the DRE data platform.

## Tech Stack

| Layer | Technology |
|---|---|
| Language | Python 3.13 |
| Snowflake Driver | Snowflake Connector + SQLAlchemy |
| SQL Server Driver | PyODBC |
| Auth | RSA key-pair (Snowflake), username/password (SQL Server) |
| Secret Management | AWS Secrets Manager |
| Git Integration | GitPython (captures commit info per batch) |
| Config | PyYAML |

## Repository Structure

```
bi_etl/
├── setup.py
├── requirements.txt
├── config.py                       ← Environment-specific connection config
└── etlengine/
    ├── engine.py                   ← Base class: DB connections, batch management, git capture
    ├── _version.py
    ├── etl/
    │   ├── ETL.py                  ← Main ETL orchestrator (inherits engine.py)
    │   ├── ETLTest.py              ← Test runner
    │   ├── ETLException.py
    │   └── sql/                    ← Meta SQL (batch, loads, tests tables)
    ├── loads_etl/
    │   ├── Loads_ETL.py            ← Loads database ETL subclass
    │   └── sql/
    ├── library_etl/
    │   ├── LibraryETL.py           ← Library/events ETL subclass
    │   └── sql/
    ├── ddl/
    │   └── DDL.py                  ← DDL object creation handler
    ├── copy/                       ← COPY command handling
    ├── external/                   ← External data source loading
    └── utils/
        ├── utils.py                ← AWS Secrets helper, Snowflake RSA key helper, Timer
        ├── enginelog.py            ← Structured logging
        └── utilsExceptions.py
```

## Architecture

### Class Hierarchy

```
engine.Engine          ← Base: connections, batch tracking, git info
    │
    └── ETL.ETL        ← Main WARS ETL orchestrator
    │
    └── Loads_ETL      ← Loads DB subclass
    │
    └── LibraryETL     ← Library/events subclass
```

### FK-Constraint-Based Load Ordering

Tables are loaded in dependency order, resolved at runtime:

```
Level 0 → Tables with no foreign-key dependencies (load first)
Level 1 → Tables that depend on Level 0
Level 2 → Tables that depend on Level 1
  ...
Level N → Tables that depend on Level N-1 (load last)
```

This ensures referential integrity without any hardcoded load order.

### Sync Modes

| Mode | Behavior |
|---|---|
| **Replace** | Truncate target table, then full reload from source |
| **Append** | Insert only new rows (no deletes) |
| **Merge** | Upsert — insert new, update changed, optionally delete removed |

## Batch Tracking

Every ETL run is recorded in Snowflake meta tables:

| Table | Purpose |
|---|---|
| `meta.BATCH` | One row per ETL run — start time, end time, git commit, status |
| `meta.LOADS` | One row per table per run — rows loaded, errors, duration |
| `meta.TESTS` | Test results from `ETLTest.py` runs |

Git commit SHA and branch are captured per batch, enabling exact reproducibility.

## Connection Management

### Snowflake
- Authenticates via **RSA key-pair** (private key loaded from AWS Secrets Manager)
- Uses SQLAlchemy as the connection layer
- Connection string built in `utils.py`

### SQL Server (Source)
- Connects via **PyODBC** with FreeTDS driver
- Used to read from GLGLIVE, FUSION, and other source systems

### Aurora PostgreSQL
- Connects via SQLAlchemy URL
- Used for NM/payment source data

## Key Files

### `engine.py`
Base class. Handles:
- All DB connection setup (Snowflake, SQL Server, PostgreSQL)
- Batch record lifecycle (open, update, close)
- Git metadata capture (branch, commit SHA, author)

### `ETL.py`
Main WARS ETL class. Handles:
- FK-dependency graph resolution
- Iterating tables by level
- Calling the appropriate sync mode per table
- Logging per-table load statistics

### `utils.py`
Utility functions:
- `get_snowflake_rsa_key()` — loads RSA private key from AWS Secrets
- `get_secret()` — generic AWS Secrets Manager retrieval
- `Timer` — context-manager based timing utility

## Local Development

```bash
# Install in editable mode
pip install -e .

# Run against GLGDEV (dev Snowflake environment)
python proto/refresh_wars.py

# Run a single table
python proto/refresh_one_table.py --table F_PROJECT
```

The `proto/` directory contains development scripts not used in production.

## Related Repositories

| Repo | Role |
|---|---|
| `BI_vdw` | SQL scripts that `bi_etl` executes |
| `bi_airflow_dags` | Airflow DAGs that invoke `bi_etl` |
| `bi_jobs` | Docker container that packages `bi_etl` for Airflow |
| `SchemaTron` | Manages the meta table DDL that `bi_etl` depends on |
