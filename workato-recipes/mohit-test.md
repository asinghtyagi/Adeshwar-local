# Mohit-Test

| Field | Value |
|-------|-------|
| **Recipe ID** | `72018777` |
| **Status** | 🔴 Stopped |
| **Trigger** | Workato Recipe Function |
| **Applications** | Workato Recipe Function, Snowflake, Marketo |

## Overview

A test recipe by Mohit that queries Snowflake and upserts each result row as a Marketo lead in a repeat loop, then returns the result count.

**Purpose:** Developer test harness for validating Snowflake-to-Marketo lead upsert logic.

## Trigger

Called manually via Recipe Function

## Data Flow Summary

| | Detail |
|---|---|
| **Source** | Snowflake |
| **Transformations** | • Search Snowflake rows
• Repeat: upsert each row as a Marketo lead |
| **Destination** | Marketo (leads) |

## Data Flow Diagram

```mermaid
flowchart LR
    A([Recipe Function Trigger]) --> B[Snowflake\nSearch rows]
    B --> C{Repeat for each row}
    C --> D[Marketo\nUpsert Lead]
    D --> C
    C --> E[Return Result]
    style A fill:#4a90d9,color:#fff
    style E fill:#27ae60,color:#fff
    style B fill:#29b5e8,color:#fff
    style D fill:#e74c3c,color:#fff
```

## Step-by-Step Breakdown

- **[Trigger] Workato Recipe Function: Recipe Function trigger**
- **Snowflake: Search rows**
- **Repeat / While loop**
  - **Marketo: Upsert lead**
  - **While condition check**
- **Workato Recipe Function: Return result**

---
*Generated from Workato recipe ID 72018777*
