# TODO

## High Priority

- [x] ~~Submit ticket to dre@glg.com for Epic Query repository access~~ ✓ Jul 9
- [x] ~~Back up `.env` and Dockerfile locally~~ ✓ Jul 9
- [x] ~~Provide list of GP-related URLs/services to Mark Huggins for UAT config~~ ✓ Jul 9
- [x] ~~Confirm INT6142 single-call fix results with Phillip~~ ✓ Jul 9
- [ ] **[Jul 18]** Follow-up call with Mark, John, Ronan on UAT cluster + S3 + Workday sandbox progress
- [x] ~~**[Due Jul 16]** Create SIT/Unit Testing cases and share with John Lawrence for review~~ ✓ Jul 16

## In Progress

- [ ] Create project tracker sheet and share with India team
- [ ] Support Ronan/Raul on update indicator + reissue logic for ingestion side (Snowflake → Workday)
- [ ] Environment setup: UAT cluster + S3 + Workday sandbox (Mark Huggins leading, target ~Jul 18)

## Follow-ups / Waiting

- [ ] Contact team/business on DAG timeout approach (Helmi's 5-min timeout fix sign-off)
- [ ] Request on-demand/manual trigger permission for GDS from SRE team (raised Jun 22)
- [ ] Clarify with Workday team whether original two-call split (supplier + address) was intentional design
- [ ] Investigate "only one active settlement bank account / no payment type selected" error on specific supplier

## Strategy

- Airflow dependency currently sits only with the US team — can leverage India team to provide support coverage during US night hours.
- Hiring on the India side should be considered due to lower cost.

## Backlog

- [ ] Review GP DB env vars (DBGP_HOST, DBGP_NAME, etc.) with colleague
- [ ] GP migration to Workday (deferred to post-phase-1)
- [ ] Docker — use `mssql-tools18` (not `mssql-tools`) in Dockerfile for ARM64/Apple Silicon; remove BIETL, Snowflake connector, extra pip lines (Epic Query handles all DB connections)
