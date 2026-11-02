# bi_md — Changes & New Requirements (Last 2 Weeks)

> Period: Oct 19 – Nov 2, 2026  
> All changes by: **Samyak Bahuguna** (`sbahuguna@glgroup.com`)  
> Repo: `glg/bi_md`

---

## Summary Table

| Date | PR | Type | File(s) Changed | One-line Summary |
|---|---|---|---|---|
| Oct 21 | #89 | `feat` | `PROJECT_STAGE.sql` | Swap survey owner column from Research Manager to Programmer |
| Oct 23 | #90 | `chore` | 5 files | Revert UAT-only changes back to production sources |
| Oct 27 | #92 | `chore` | `VW_SUPPLIER_INVOICE.sql` | Hardcode Employee ID for parallel testing |
| Oct 29 | #93 | `fix` | `NM_PAYMENT_STAGE.sql` | Fix Cost Centre data quality via VEGA_ID_C join |

---

## PR #89 — Oct 21 — Column Change for Surveys (New Requirement)

**Commit:** `f103123`  
**Type:** Feature / New Requirement  
**File:** `MD/WORKDAY/NM_PAYMENT/SUPPLIER_INVOICE/etl/PROJECT_STAGE/PROJECT_STAGE.sql`

### Requirement
For survey-type projects in Workday, the **Primary RM** field should reflect the **Programmer** (the person running the survey operationally), not the **Research Manager** who owns the client relationship. This was a business-side request to fix attribution on Workday supplier invoices for surveys.

### What Changed

**Before:**
```sql
-- Qualtrics/Decipher survey branch in PROJECT_STAGE
, S.ResearchManagerPersonID   -- Research Manager assigned to the survey
```

**After:**
```sql
-- Qualtrics/Decipher survey branch in PROJECT_STAGE
, S.PROGRAMMERPERSONID        -- Programmer (operational owner) of the survey
```

### Impact
The `PRIMARY_RM_PERSON_ID` column fed from `PROJECT_STAGE` into `NM_PAYMENT_STAGE` now resolves to the Programmer's Fusion ID instead of the Research Manager's. This flows through to `APPROVER_FUSION_ID` / `EMPLOYEE_ID` on the Workday supplier invoice.

---

## PR #90 — Oct 23 — Revert UAT Changes (5 Files)

**Commit:** `8459c30`  
**Type:** Chore / Revert  
**Files Changed:** `NM_ADDRESS_STAGE.sql`, `NM_DETAIL_STAGE.sql`, `NM_PAYMENT_STAGE.sql`, `PRODUCT_STAGE.sql`, `PROJECT_STAGE.sql`

### Requirement
UAT (User Acceptance Testing) had required temporary changes to point the pipeline at a dev Fivetran source (`DEV_FIVETRAN.GLGLIVE_DEV_DBO`) and apply special character transformations on address fields. Once UAT completed, these needed to be reverted back to production sources before going live.

### What Changed

#### 1. `NM_ADDRESS_STAGE.sql` — Source + Special Character Handling

**Before (UAT):**
```sql
-- Reading from DEV Fivetran + applying special char replacements
, REPLACE(REPLACE(REPLACE(PA.ADDRESS1, '\\', '\\\\'), 'æ', 'ae'), 'ß', 'ss') AS ADDRESS1
, REPLACE(REPLACE(PA.ADDRESS2, 'æ', 'ae'), 'ß', 'ss')                        AS ADDRESS2
, REPLACE(REPLACE(PA.ADDRESS3, 'æ', 'ae'), 'ß', 'ss')                        AS ADDRESS3
, REPLACE(REPLACE(PA.CITY,     'æ', 'ae'), 'ß', 'ss')                        AS CITY

FROM DEV_FIVETRAN.GLGLIVE_DEV_DBO.PAYMENT_ACCOUNT PA
JOIN DEV_FIVETRAN.GLGLIVE_DEV_DBO.COUNTRY C ...
```

**After (Production):**
```sql
-- Reading from production Staging/Fivetran, raw columns
, PA.ADDRESS1
, PA.ADDRESS2
, PA.ADDRESS3
, PA.CITY

FROM STAGING.GLGLIVE_DBO.PAYMENT_ACCOUNT PA
JOIN FIVETRAN.GLGLIVE_DBO.COUNTRY C ...
```

> The special character replacements (`æ→ae`, `ß→ss`) were a workaround to replicate a WARS copy-job transformation that was being lost when reading from the dev source. With prod sources these transformations are no longer needed.

#### 2. `NM_PAYMENT_STAGE.sql`, `NM_DETAIL_STAGE.sql`, `PROJECT_STAGE.sql`, `PRODUCT_STAGE.sql`
- All were similarly reverted from `DEV_FIVETRAN.*` back to `STAGING.*` / `FIVETRAN.*` production databases
- Approximately 120 lines changed across these files — primarily source database references, no logic changes

---

## PR #92 — Oct 27 — Hardcoded Employee ID for Parallel Testing

**Commit:** `a355821`  
**Type:** Chore / Temporary  
**File:** `MD/WORKDAY/NM_PAYMENT/SUPPLIER_INVOICE/etl/VW_SUPPLIER_INVOICE/VW_SUPPLIER_INVOICE.sql`

### Requirement
To validate the pipeline during parallel testing (running the new and old pipeline side-by-side and diffing outputs), the `EMPLOYEE_ID` field needed to be deterministic. `APPROVER_FUSION_ID` varies per payment based on who approved it, making it impossible to diff outputs cleanly.

### What Changed

**Before:**
```sql
, APPROVER_FUSION_ID AS EMPLOYEE_ID
-- Dynamic — varies per payment based on the approver's Fusion ID
```

**After (Temporary):**
```sql
, CAST(206101 AS NUMBER(38,0)) AS EMPLOYEE_ID
-- Hardcoded constant — same value for every row during testing
```

### Status
**Temporary** — this is intended to be reverted once parallel testing completes. The constant `206101` is a known valid Fusion employee ID used as a stable placeholder.

---

## PR #93 — Oct 29 — Bug Fix: Cost Centre Data Quality

**Commit:** `5604c00`  
**Type:** Bug Fix  
**File:** `MD/WORKDAY/NM_PAYMENT/SUPPLIER_INVOICE/etl/NM_PAYMENT_STAGE/NM_PAYMENT_STAGE.sql`

### Problem
Cost Centre and Region worktags on Workday supplier invoices were showing blank or incorrect values for some Network Members. The `SEG1_WORKTAGS` CTE was joining Salesforce `CONTACT` records on `NUMERIC_VEGA_ID_C` — a pre-computed numeric column in Salesforce. This column had data quality issues (missing or stale values) for a subset of contacts, causing the join to fail silently and leaving `COST_CENTER_CODE` and `CLIENT_REGION_WT` as `'CC_Default_Blank'` / `'RE_Default_Blank'` on the invoice.

### What Changed

**Before:**
```sql
, SEG1_WORKTAGS AS (
    SELECT
        CAST(CON.NUMERIC_VEGA_ID_C AS NUMBER(38,0)) AS USER_CONTACT_ID  -- pre-computed, unreliable
      , M.REGION_ID
      , M.COST_CENTER_ID
    FROM FIVETRAN.SALESFORCE.CONTACT CON
    ...
    WHERE
          CON._FIVETRAN_DELETED = FALSE
      AND CON.IS_DELETED = FALSE
      AND CON.NUMERIC_VEGA_ID_C IS NOT NULL  -- filters on the bad column
)
```

**After:**
```sql
, SEG1_WORKTAGS AS (
    SELECT
        TRY_TO_NUMBER(CON.VEGA_ID_C) AS USER_CONTACT_ID  -- derived fresh from raw string field
      , M.REGION_ID
      , M.COST_CENTER_ID
    FROM FIVETRAN.SALESFORCE.CONTACT CON
    ...
    WHERE
          CON._FIVETRAN_DELETED = FALSE
      AND CON.IS_DELETED = FALSE
      AND TRY_TO_NUMBER(CON.VEGA_ID_C) IS NOT NULL  -- filters on the freshly derived value
)
```

### Why `TRY_TO_NUMBER()` instead of `CAST()`
- `CAST(... AS NUMBER)` throws an error on non-numeric strings, which would fail the entire ETL run
- `TRY_TO_NUMBER()` returns `NULL` gracefully on bad values — the `IS NOT NULL` filter then excludes those rows cleanly
- Reading from `VEGA_ID_C` (the raw source field) bypasses the stale pre-computed column entirely

### Impact
Cost Centre and Region worktags on Workday supplier invoices now correctly populate for the affected contacts, improving GL coding accuracy in Workday.

---

## Files Changed (Full List)

| File | PRs |
|---|---|
| `MD/WORKDAY/NM_PAYMENT/SUPPLIER_INVOICE/etl/NM_PAYMENT_STAGE/NM_PAYMENT_STAGE.sql` | #90, #93 |
| `MD/WORKDAY/NM_PAYMENT/SUPPLIER_INVOICE/etl/VW_SUPPLIER_INVOICE/VW_SUPPLIER_INVOICE.sql` | #92 |
| `MD/WORKDAY/NM_PAYMENT/SUPPLIER_INVOICE/etl/PROJECT_STAGE/PROJECT_STAGE.sql` | #89, #90 |
| `MD/WORKDAY/NM_PAYMENT/SUPPLIER_INVOICE/etl/PRODUCT_STAGE/PRODUCT_STAGE.sql` | #90 |
| `MD/WORKDAY/NM_PAYMENT/SUPPLIER/etl/NM_ADDRESS_STAGE/NM_ADDRESS_STAGE.sql` | #90 |
| `MD/WORKDAY/NM_PAYMENT/SUPPLIER/etl/NM_DETAIL_STAGE/NM_DETAIL_STAGE.sql` | #90 |
