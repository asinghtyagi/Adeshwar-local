# Prod Test - Salesforce Contact Update

| Field | Value |
|-------|-------|
| **Recipe ID** | `74699130` |
| **Status** | 🔴 Stopped |
| **Trigger** | Workato Recipe Function |
| **Applications** | Workato Recipe Function, Workato Variable, Snowflake, Logger, CSV Parser, Salesforce |

## Overview

A test/non-production recipe that exports contact data from Snowflake as CSV and submits a Salesforce bulk update job to update Contact records.

**Purpose:** Used for Salesforce Contact update testing against Snowflake data before promoting to production.

## Trigger

Called manually via Recipe Function (never run in production)

## Data Flow Summary

| | Detail |
|---|---|
| **Source** | Snowflake (contact data) |
| **Transformations** | • Export Snowflake rows as CSV
• Convert to CSV format via parser
• Submit Salesforce bulk update job |
| **Destination** | Salesforce (Contact updates) |

## Data Flow Diagram

```mermaid
flowchart LR
    A([Recipe Function Trigger]) --> B[Declare Variables]
    B --> C[Snowflake\nExport CSV]
    C --> D[Logger\nLog row count]
    D --> E[CSV Parser\nCreate CSV lines]
    E --> F[Salesforce\nBulk Update Job]
    style A fill:#4a90d9,color:#fff
    style C fill:#29b5e8,color:#fff
    style F fill:#e74c3c,color:#fff
```

## Step-by-Step Breakdown

- **[Trigger] Workato Recipe Function: Recipe Function trigger**
- **Workato Variable: Declare variable**
- **Snowflake: Export CSV**
- **Logger: Log message**
- **CSV Parser: Create CSV lines**
- **Salesforce: Update (bulk job)**

---
*Generated from Workato recipe ID 74699130*
