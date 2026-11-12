# Marketo Update User Metrics Billable

| Field | Value |
|-------|-------|
| **Recipe ID** | `66877883` |
| **Status** | 🟢 Running |
| **Trigger** | Scheduler (Clock) |
| **Applications** | Scheduler (Clock), Workato Recipe Function, Email |

## Overview

The top-level scheduler recipe for the user metrics pipeline. Runs on a schedule, calls two sub-recipes sequentially (Job 1 and Update User Metrics), and emails an alert if either fails.

**Purpose:** Orchestrates the daily/recurring user metrics update pipeline by coordinating Job 1 and the core update recipe, with failure alerting.

## Trigger

Scheduler (Clock) — recurring cron schedule

## Data Flow Summary

| | Detail |
|---|---|
| **Source** | Triggers sub-recipes 68362761 and 66876417 |
| **Transformations** | • Call Job 1 sub-recipe synchronously
• Call Update User Metrics sub-recipe synchronously
• On any error: send failure email notification |
| **Destination** | Email (failure alerts), delegates to sub-recipes |

## Data Flow Diagram

```mermaid
flowchart TD
    A([Clock Scheduled Trigger]) --> B[Try]
    B --> C[Recipe Function\nCall Run Manual Jobs 1\n68362761]
    C --> D[Recipe Function\nCall Update User Metrics\n66876417]
    D --> E([Done])
    B -- error --> F[Email\nSend failure alert]
    F --> G([End with error])
    style A fill:#4a90d9,color:#fff
    style E fill:#27ae60,color:#fff
    style F fill:#e74c3c,color:#fff
    style C fill:#e67e22,color:#fff
    style D fill:#e67e22,color:#fff
```

## Step-by-Step Breakdown

- **[Trigger] Scheduler (Clock): Scheduled trigger**
- **Try block**
  - **Workato Recipe Function: Call sub-recipe**
  - **Workato Recipe Function: Call sub-recipe**
  - **Catch (error handler)**
    - **Email: Send email**

---
*Generated from Workato recipe ID 66877883*
