# User Metrics - CDC ETL

| Field | Value |
|-------|-------|
| **Recipe ID** | `66806430` |
| **Status** | 🟢 Running |
| **Trigger** | Workato Recipe Function |
| **Applications** | Workato Recipe Function, Lookup Table, Snowflake |

## Overview

A CDC ETL sub-recipe that truncates a Lookup Table and repopulates it with fresh user metrics data from Snowflake using multiple SQL queries, acting as a full refresh of the in-memory state.

**Purpose:** Resets and rebuilds the Lookup Table used as a fast-access cache/checkpoint store for user metrics, pulling the latest data from Snowflake.

## Trigger

Called by other recipes via Recipe Function

## Data Flow Summary

| | Detail |
|---|---|
| **Source** | Snowflake (multiple SQL queries for user metrics) |
| **Transformations** | • Truncate Lookup Table (full reset)
• Run SQL 1: base metrics → write to Lookup Table
• Run SQL 2, 3, 4: supplemental metrics → append to Lookup Table |
| **Destination** | Workato Lookup Table (rebuilt with fresh data) |

## Data Flow Diagram

```mermaid
flowchart LR
    A([Recipe Function Trigger]) --> B[Lookup Table\nTruncate all entries]
    B --> C[Snowflake\nSQL 1 - base metrics]
    C --> D[Lookup Table\nAdd entry - base]
    D --> E[Snowflake\nSQL 2 - supplemental]
    E --> F[Snowflake\nSQL 3 - supplemental]
    F --> G[Snowflake\nSQL 4 - supplemental]
    G --> H[Return Result]
    style A fill:#4a90d9,color:#fff
    style H fill:#27ae60,color:#fff
    style C fill:#29b5e8,color:#fff
    style E fill:#29b5e8,color:#fff
    style F fill:#29b5e8,color:#fff
    style G fill:#29b5e8,color:#fff
    style B fill:#f39c12,color:#fff
```

## Step-by-Step Breakdown

- **[Trigger] Workato Recipe Function: Recipe Function trigger**
- **Lookup Table: Truncate table**
- **Snowflake: Run custom SQL**
- **Lookup Table: Add entry**
- **Snowflake: Run custom SQL**
- **Snowflake: Run custom SQL**
- **Snowflake: Run custom SQL**
- **Workato Recipe Function: Return result**

---
*Generated from Workato recipe ID 66806430*
