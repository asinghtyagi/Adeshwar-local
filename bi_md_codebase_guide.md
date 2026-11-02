# bi_md Codebase Guide

## What is bi_md?

`bi_md` = **Business Intelligence Master Data**. It is a Snowflake ETL repository that prepares a master data (`MD`) database used as the source for **Workday Prism integrations**, WARS (GLG's warehouse), and other downstream systems. It acts as a data transformation layer that cleans, normalizes, and structures GLG's operational data for finance and analytics.

---

## Folder Structure

```
MD/
├── CORE/           → Shared dimension tables (clients, employees, dates, products)
└── WORKDAY/        → Workday-specific pipelines
    ├── CLIENT_CONTRACT/          → Active client + contract sync to Workday
    ├── CLIENT_CONTRACT_DISABLED/ → Same pipeline, currently disabled
    ├── NM_PAYMENT/               → Network Member (expert) payment pipeline
    ├── THIN_LEDGER/              → Revenue/expense thin ledger
    └── USAGE/                    → Usage metrics
```

Each entity has two subfolders:
- **`ddl/`** — Table schema definitions (CREATE TABLE structures)
- **`etl/`** — The actual transformation SQL + a `_CONFIG.yml` that controls how it runs

---

## The Config Pattern (`_CONFIG.yml`)

Every ETL object has a companion YAML:

```yaml
target: NM_PAYMENT_STAGE    # Snowflake table/view to create
type: rawsql                 # Just run the SQL as-is
tokenize: true               # Replace tokens (e.g., env vars) before running
requires:                    # Dependencies — must run before this
  - PRODUCT_STAGE
  - PROJECT_STAGE
```

This config drives an orchestration framework that runs ETLs in dependency order.

---

## The Three Main Domain Pipelines

### 1. CORE — Shared Dimensions

These are dimension tables shared across all pipelines:

| Table | What it contains |
|---|---|
| `D_CLIENT` | SCD Type 2 client history (GLG + Salesforce) |
| `D_EMP` | Employee dimension |
| `D_DATE` | Date dimension |
| `D_PRODUCT` | Product/service types |
| `D_USER` | User accounts |
| `D_CLIENT_TYPE` | Client categorization |

**`D_CLIENT` is the most complex** — it is a full SCD Type 2 (Slowly Changing Dimension) table tracking historical changes for 6 attributes per client:

1. Global Indicator
2. Account Status
3. Primary Owner
4. Secondary Owner
5. Sector
6. PL Map Entry

It builds up 13 staging temp tables before writing the final dimension.

---

### 2. NM_PAYMENT — Network Member Payment Pipeline

This is the most active area. It handles paying GLG council members (experts) for consultations, events, and surveys.

**Data flow:**

```
Source Systems (FIVETRAN → Snowflake)
        ↓
NM_PAYMENT_STAGE   ← core staging table (~600 lines of SQL)
        ↓
NM_PAYMENT         ← persisted table with SCD tracking
        ↓
VW_SUPPLIER_INVOICE ← Workday-facing view (outbound)
        ↓
SUPPLIER_INVOICE_INTEGRATION → sent to Workday Prism
```

#### `NM_PAYMENT_STAGE` — CTE Breakdown

The staging table is built through a series of CTEs before the final SELECT:

| CTE | Purpose |
|---|---|
| `LAW_TIMESHEETS` | Joins law firm timesheets to invoices |
| `COMPLIANCE_CASE` | Links Salesforce compliance cases to consultations |
| `INVOICES` | Resolves payment → invoice relationships |
| `BRIDGE` | Flags whether a call used the conference bridge (status 52) |
| `PRODUCT_MAP` | Maps GLG product names to Workday feature/GL codes |
| `EVENT_PARTICIPANTS` | Counts attendees for events |
| `REFERRAL_INVOICE` | Identifies referral-type payments |
| `PAYMENT_REISSUE` | Tracks reissued payments via HSBC bank refs |
| `SEG1_WORKTAGS` | Maps contacts → Region + Cost Center via Salesforce PLMAP |
| `PAYMENT_BASE` | Core payment records for yesterday only (`DATEADD(DAY, -1, ...)`) |
| `PAYMENT_FINAL` | UNION of 6 payment types (see below) |

#### `PAYMENT_FINAL` — 6 Payment Type UNIONs

| Payment Type | Join Key |
|---|---|
| Consultations | `CONSULTATION_PARTICIPANT.MEETING_ID` |
| Qualtrics / Decipher Surveys | `GLGLIVE_QUALTRICS.SURVEY.SURVEYID` |
| Survey 2.0 | `GLGLIVE_SURVEY.SURVEY.SURVEY_ID` |
| Survey 1.0 | `GLGLIVE_DBO.SURVEY.SURVEY_ID` |
| Events | `EVENT_TABLE.MEETING_ID` |
| Untagged Payments | No meeting/survey ID — catch-all |

#### Final SELECT — Key Computed Columns

| Column | Logic |
|---|---|
| `NM_PAYMENT_KEY` | `HASH(PAYMENT_ID + USER_CONTACT_ID)` — surrogate key |
| `QUANTITY` | `COALESCE(BILLABLE_MINUTES, 1)` |
| `UNIT_COST` | `AMOUNT / COALESCE(BILLABLE_MINUTES, PARTICIPANT_COUNT, 1)` |
| `EXTENDED_AMOUNT` | `AMOUNT / COALESCE(PARTICIPANT_COUNT, 1)` |
| `CLIENT_REGION_WT` | Workday worktag — from `SEG1_WORKTAGS` |
| `COST_CENTER_CODE` | Workday worktag — from `SEG1_WORKTAGS` |
| `LOCATION_CODE` | `LC_` + US state or ISO3 country code |
| `SPEND_CATEGORY_ID` | `SC_1071` (honoraria), `SC_1196` (referrals), `SC_1115` (other) |
| `AUTO_REFERRAL_IND` | True if payment linked to a referral invoice |
| `REISSUE_IND` | True if payment was reissued via HSBC |

#### `VW_SUPPLIER_INVOICE`

A thin view on top of `NM_PAYMENT` that renames GLG columns to Workday integration field names:

| GLG Column | Workday Field |
|---|---|
| `PAYMENT_ID` | `SUPPLIER_INVOICE_ID` |
| `GP_VENDOR_ID` | `SUPPLIER_ID` |
| `PAYMENT_ACCOUNT_ID` | `SUPPLIER_CONNECTION_ID` |
| `NM_PAYMENT_KEY` | `PRIMARY_KEY` |
| `CLIENT_ID` | `CUSTOMER_ID` / `CUSTOMER_REFERENCE_ID` |
| `COST_CENTER_CODE` | `COST_CENTER_REFERENCE_ID` |
| `LOCATION_CODE` | `LOCATION_ID` |

> **Note:** `EMPLOYEE_ID` is currently hardcoded to `206101` for parallel testing purposes (PR #92). This is a temporary change pending revert.

---

### 3. CLIENT_CONTRACT — Client Sync to Workday

Syncs GLG client records into Workday as **Customers**:

```
CLIENT_STAGE → CLIENT → VW_WORKDAY_CUSTOMER → CUSTOMER_INTEGRATION → Workday
```

Also includes:
- `CLIENT_HIERARCHY` — resolves parent/child relationships
- `CLIENT_INCLUDED_CHILDREN` — flattens child client lists
- `CLIENT_NM_PAYMENT` — links clients to payment records
- `CLIENT_SFDC` — Salesforce-sourced client attributes
- `WORKDAY_COUNTRY_REGION` — country/region reference mapping

The `CLIENT_CONTRACT_DISABLED/` folder contains a more complex version (with `ORDER_TABLE`, `ORDER_LINE_ITEM`, `CONTRACT` tables) that is currently disabled — likely superseded by the simpler active pipeline.

---

## INBOUND / INTERNAL / OUTBOUND Pattern

Every Workday pipeline is organized into three layers:

| Layer | Role |
|---|---|
| **INBOUND** | Feedback tables where Workday writes back success/failure of import runs (e.g., `CR_SUPPLIER_IMPORT_FEEDBACK`, `CR_SUPPLIER_INVOICE_IMPORT_FEEDBACK`) |
| **INTERNAL** | Staging and transformation logic — the ETL core |
| **OUTBOUND** | Views and integration configs that push data into Workday Prism |

---

## Data Sources

| Source | What it contains |
|---|---|
| `FIVETRAN.*` | Raw replicated data — Salesforce, GLGLive (SQL Server), payment tables |
| `STAGING.*` | Partially transformed staging data (e.g., `STAGING.WORKDAY.SEG1_WORKTAG_MAP`) |
| `DEV_MASTER_DATA.*` / `WORKDAY.*` | The MD database itself (self-referential joins for dimension lookups) |

---

## Key Design Patterns

### Daily Incremental Load
`NM_PAYMENT_STAGE` only processes **yesterday's payments**:
```sql
DATE(PAY.CREATE_DATE) = DATEADD(DAY, -1, CONVERT_TIMEZONE('UTC', CURRENT_TIMESTAMP())::DATE)
```

### Deduplication with `QUALIFY ROW_NUMBER()`
Used heavily when joining to multi-row tables to pick the single most relevant record:
```sql
QUALIFY ROW_NUMBER() OVER (PARTITION BY INVOICE_ID ORDER BY CREATE_DATE DESC) = 1
```

### Safe Type Conversion with `TRY_TO_NUMBER()`
Preferred over `CAST()` for Salesforce ID fields that may contain non-numeric strings:
```sql
-- Safe — returns NULL on bad values instead of erroring
TRY_TO_NUMBER(CON.VEGA_ID_C) AS USER_CONTACT_ID
```

### SCD Type 2 via `LEAD()` Window Function
End dates are computed dynamically — not stored in source systems:
```sql
LEAD(DATEADD(NS, -100, START_DATE_UTC), 1)
    OVER (PARTITION BY CLIENT_ID ORDER BY START_DATE_UTC) AS END_DATE_UTC
```
The `-100ns` offset ensures no overlap between consecutive records.

### HASH Surrogate Keys
All dimension keys are computed hashes of business keys — no sequences:
```sql
HASH(CONCAT(CAST(PAYMENT_ID AS VARCHAR), COALESCE(CAST(USER_CONTACT_ID AS VARCHAR), 'NULL_CONTACT')))
    AS NM_PAYMENT_KEY
```

---

## Recent Changes (Last Week — as of 2026-11-02)

### PR #93 — Oct 29 — Bug Fix: Cost Centre Data Quality
**File:** `MD/WORKDAY/NM_PAYMENT/SUPPLIER_INVOICE/etl/NM_PAYMENT_STAGE/NM_PAYMENT_STAGE.sql`

**Problem:** The `SEG1_WORKTAGS` CTE was joining on `NUMERIC_VEGA_ID_C` (a pre-computed column in Salesforce) which had missing or incorrect values for some contacts, causing Cost Centre and Region worktags to be blank or wrong on Workday invoices.

**Fix:** Switched to deriving the numeric ID directly from the raw `VEGA_ID_C` string field using `TRY_TO_NUMBER()`:

```sql
-- Before
CAST(CON.NUMERIC_VEGA_ID_C AS NUMBER(38,0)) AS USER_CONTACT_ID
AND CON.NUMERIC_VEGA_ID_C IS NOT NULL

-- After
TRY_TO_NUMBER(CON.VEGA_ID_C) AS USER_CONTACT_ID
AND TRY_TO_NUMBER(CON.VEGA_ID_C) IS NOT NULL
```

### PR #92 — Oct 27 — Chore: Hardcoded Employee ID for Parallel Testing
**File:** `MD/WORKDAY/NM_PAYMENT/SUPPLIER_INVOICE/etl/VW_SUPPLIER_INVOICE/VW_SUPPLIER_INVOICE.sql`

**Change:** Replaced `APPROVER_FUSION_ID` with a constant for deterministic parallel test comparisons:

```sql
-- Before
APPROVER_FUSION_ID AS EMPLOYEE_ID

-- After (temporary)
CAST(206101 AS NUMBER(38,0)) AS EMPLOYEE_ID
```

This makes the `EMPLOYEE_ID` field identical across both pipeline runs so other columns can be diffed cleanly. Intended to be reverted after parallel testing completes.

---

## CODEOWNERS

Code ownership is defined in `CODEOWNERS` at the repo root — changes to specific paths require review from the designated owners before merging.
