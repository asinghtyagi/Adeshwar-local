# Copy of Marketo NM Dynamic Data Ingest

| Field | Value |
|-------|-------|
| **Recipe ID** | `72460703` |
| **Status** | 🔴 Stopped |
| **Trigger** | Workato Recipe Function |
| **Applications** | Workato Recipe Function, Lookup Table, Workato Variable, Snowflake, Marketo |

## Overview

A copy of the Marketo NM Dynamic Data Ingest recipe. Reads a checkpoint from a Lookup Table, queries Snowflake for NM records since the last run, and upserts them as Marketo custom objects in batches, updating the checkpoint afterward.

**Purpose:** CDC (Change Data Capture) pattern: incrementally syncs NM data from Snowflake to Marketo custom objects, using a Lookup Table as a persistent watermark.

## Trigger

Called manually via Recipe Function (copy — not in production)

## Data Flow Summary

| | Detail |
|---|---|
| **Source** | Snowflake (NM records), Lookup Table (watermark/checkpoint) |
| **Transformations** | • Read watermark (last processed ID or timestamp) from Lookup Table
• Query Snowflake for records since watermark
• Batch upsert records to Marketo custom objects
• Update watermark in Lookup Table |
| **Destination** | Marketo (custom objects), Lookup Table (updated checkpoint) |

## Data Flow Diagram

```mermaid
flowchart LR
    A([Recipe Function Trigger]) --> B[Lookup Table\nGet checkpoint entry]
    B --> C[Declare Lists & Variables]
    C --> D[Snowflake\nSearch rows SQL\nsince last watermark]
    D --> E[Declare Variable\nbatch counter]
    E --> F{For Each Row}
    F --> G[Marketo\nUpsert Custom Objects]
    G --> H[Update Variables\ncounter]
    H --> I{Batch full?}
    I -- Yes --> J[Lookup Table\nAdd batch of entries]
    I -- No --> F
    F --> K[Lookup Table\nAdd entry - new watermark]
    style A fill:#4a90d9,color:#fff
    style D fill:#29b5e8,color:#fff
    style G fill:#e74c3c,color:#fff
    style K fill:#f39c12,color:#fff
```

## Step-by-Step Breakdown

- **[Trigger] Workato Recipe Function: Recipe Function trigger**
- **Lookup Table: Get entries**
- **Workato Variable: Declare list**
- **Workato Variable: Declare variable**
- **Snowflake: Search rows (SQL)**
- **Workato Variable: Declare variable**
- **For Each loop**
  - **Marketo: Upsert custom objects**
  - **Workato Variable: Update variables**
  - **If condition**
    - **Lookup Table: Add batch of entries**
- **Lookup Table: Add entry**

---
*Generated from Workato recipe ID 72460703*
