# BUSINESS REQUIREMENTS DOCUMENT — Cron / Batch Job



## Table of Contents



- 1. Document Control

- 2. Executive Summary

- 3. System Context

- 4. Job Identity and Operations

- 5. Functional / Business Requirements

- 6. Business Rules Catalogue

- 7. Data Requirements

- 8. Glossary

- 9. Technical Inventory

- 10. Referenced Database Objects

- 11. Backend Packages and Procedures

- 12. Shared Functions

- 13. Interfaces

- 14. Non-Functional Requirements

- 15. Assumptions, Constraints, N/A Register

- 16. Traceability Matrix



## 1. Document Control

### 1.1 Document Information

| Field | Value |
| --- | --- |
| Document Title | PKG_SRV_INCENTIVE_SCH_C - Business Requirements Document |
| Job / Script Name | PKG_SRV_INCENTIVE_SCH_C.sh |
| Document Version | 1.0 |
| Date Created | October 17, 2026 |
| Author/Prepared By | BRD Agent |
| Source Artifacts | `PKG_SRV_INCENTIVE_SCH_C.sh`; `dms_cron_profile`; `PKG_SRV_BS_INCENTIVE_SCH.pck` |

### 1.2 Revision History

| Version | Date | Author | Description |
| --- | --- | --- | --- |
| 1.0 | October 17, 2026 | BRD Agent | Initial as-is draft from ZIP extraction |

## 2. Executive Summary

*High-level overview of the **PKG_SRV_INCENTIVE_SCH_C** batch job capability and the business context evidenced in the source. Fill the placeholder with one short business-outcome paragraph plus at most one “how it runs” sentence — not a table/SQL dump.*

This job automates the twice-monthly refresh of Bodyshop Incentive Scheme scores for zone-C dealer outlets. It clears the prior zone-C records, gathers repair-order data from the vehicle-RO master (`VH_RO`), and — for each eligible 2S/3S `DDL`-category dealer — calculates the current- and prior-year B&P bodyshop load, load-growth marks, Smart Eye conversion rate, and corresponding incentive marks. The final scores are written to `VD_SRV_INC` and `VD_SRV_INC_VIN_WISE_DATA`, and category assignments are also captured in `MARKS`, giving scheme administrators a reliable, current-month view of dealer performance ahead of pay-out processing. The job runs as a shell script (`PKG_SRV_INCENTIVE_SCH_C.sh`) that opens a SQL\*Plus session and executes `PKG_SRV_BS_INCENTIVE_SCH.SP_GEN_BS_INCENTIVE(1,'C')`.

### Key Stakeholders

*The source identifies batch execution context (shell, SQL\*Plus, Oracle credentials), but it does not name a business role, job title, approver, or process owner. No role is inferred for this baseline.*

| Stakeholder | Role |
| --- | --- |
| N/A | No named business actor or role appears in the source. The batch job runs under the OS/Oracle credentials supplied by the sourced profile; the person or team that owns the business outcome is not identified in the ZIP. |

### In scope / out of scope

**In scope**

- This job’s shell `PKG_SRV_INCENTIVE_SCH_C.sh` and the database work it starts
- Entry routine `PKG_SRV_BS_INCENTIVE_SCH.SP_GEN_BS_INCENTIVE` and helpers on this job’s call chain (not other package members)
- Write the operational spool log at `/home/dmsdb/logs/PKG_SRV_INCENTIVE_SCH_C.log`

**Out of scope**

- Other procedures in the same `.pck` that this job does not call
- UI / Oracle Forms modules
- Other regional script variants unless listed in this ZIP
- Time zone, blackout windows, Control-M predecessors/successors, business owners, and SLAs not present in source

### Open gaps

- None detected in ZIP (profile contents / table DDL may still be outstanding operationally).
- Business rules not signed off by SME
- Tables not delivered as DDL files (schema objects appear via static SQL scan only)

## 3. System Context

*Describes the **PKG_SRV_INCENTIVE_SCH_C** job's verified scope, process flow, and data interactions. System purpose / Process Objective = business outcome; technical path stays in Process Flow STEPs below.*

This batch job refreshes the Bodyshop Incentive Scheme data for dealer outlets in zone C. Twice a month it clears prior zone-C incentive records, collects current- and prior-year Bodyshop repair-order activity from the vehicle RO master, and computes dealer-level incentive category, load-growth marks, and Smart Eye conversion marks. The results are stored in the service incentive tables, making up-to-date performance scores available for downstream reporting and scheme pay-out processing.

**System purpose:** Automated generation and refresh of Bodyshop (B&P) Incentive Scheme metrics — including dealer category assignment, bodyshop load growth marks, and Smart Eye conversion marks — for zone-C dealer outlets of a specified principal.

**In scope:**

- Run this batch job (`PKG_SRV_INCENTIVE_SCH_C.sh`) under the sourced profile
- Invoke entry `PKG_SRV_BS_INCENTIVE_SCH.SP_GEN_BS_INCENTIVE` with the parameters observed in the shell
- Persist / update business data as implemented on the entry path
- Write the operational spool log when a path is present in source

**Process Name:** `PKG_SRV_BS_INCENTIVE_SCH.SP_GEN_BS_INCENTIVE` (business process label: Bodyshop Incentive Scheme Generation — Zone C)

**Process Owner:** N/A — not identified in the available source artifacts

**Process Objective:** To produce accurate, up-to-date Bodyshop Incentive Scheme scores for all eligible zone-C dealer outlets — covering bodyshop load category, load-growth marks, and Smart Eye conversion marks — so that scheme administrators and reporting processes have a reliable, current-month view of dealer performance against incentive targets.

**Process Trigger:** The job is scheduled to run **at 01:00 on days 1 and 16 of every month** (server local time). Source: cron execution register.

**Process Flow:**

The verified batch process follows this sequence. Each step is mapped to its Section 5 requirement.

```
STEP 1 ── JOB START [BR-001]
         Scheduler invokes `PKG_SRV_INCENTIVE_SCH_C.sh` on the batch host (scheduler identity: N/A).

STEP 2 ── SOURCE PROFILE [BR-001]
         The shell sources the cron profile (`/dmsdb/cron/dms_cron_profile`) to establish
         Oracle environment and credentials (`$ORACLE_HOME`, `$PD`, `$UD`).

STEP 3 ── CONNECT AND INVOKE ENTRY [BR-001]
         SQL\*Plus connects and executes `execute PKG_SRV_BS_INCENTIVE_SCH.Sp_Gen_BS_Incentive(1,'C');`.
         Package source: `PKG_SRV_BS_INCENTIVE_SCH.pck`.

STEP 4 ── PROCEDURE WORK [BR-002]
         The entry procedure performs the evidenced data read/write path
         (see §5 / §7 / §11). Sibling procedures in the same package that are
         not called by this job remain out of scope.

STEP 5 ── SPOOL AND EXIT [BR-003]
         SQL\*Plus writes the spool log (`/home/dmsdb/logs/PKG_SRV_INCENTIVE_SCH_C.log`) and exits.
```

**Major process interactions:**

- **Oracle database** — entry package/procedure `PKG_SRV_BS_INCENTIVE_SCH.SP_GEN_BS_INCENTIVE` (file `PKG_SRV_BS_INCENTIVE_SCH.pck`)
- **Credential / profile** — `/dmsdb/cron/dms_cron_profile`
- **Operational log** — `/home/dmsdb/logs/PKG_SRV_INCENTIVE_SCH_C.log`
- **Downstream consumers** — N/A — not identified in the available source artifacts

### Systems involved

| System | Evidence |
| --- | --- |
| OS / shell | Main script `PKG_SRV_INCENTIVE_SCH_C.sh` |
| Oracle database | SQL*Plus / EXEC / anonymous PL/SQL block in source |
| Credential / profile | Sourced scripts (`/dmsdb/cron/dms_cron_profile`) |
| Downstream consumers | N/A — not identified in the available source artifacts |

### Runtime environment

Environment variables observed: `$ORACLE_HOME`, `$PD`, `$UD`

Log path(s): `/home/dmsdb/logs/PKG_SRV_INCENTIVE_SCH_C.log`

Profile contents: N/A until profile file contents are reviewed from the ZIP (presence is recorded in §4.5).

## 4. Job Identity and Operations

*Batch identity, parameters, schedule posture, and operational controls evidenced for **PKG_SRV_INCENTIVE_SCH_C**.*

### 4.1 Batch identity

| Field | Value |
| --- | --- |
| Script | PKG_SRV_INCENTIVE_SCH_C.sh |
| Log | /home/dmsdb/logs/PKG_SRV_INCENTIVE_SCH_C.log |
| Entry call | execute PKG_SRV_BS_INCENTIVE_SCH.Sp_Gen_BS_Incentive(1,'C'); |
| Package file | PKG_SRV_BS_INCENTIVE_SCH.pck |

### 4.2 Parameters

| # | Technical value | Business meaning | Evidence |
| --- | --- | --- | --- |
| 1 | 1 | Principal identifier — selects the principal brand/manufacturer whose dealer outlets are processed in this run (value `1` observed in shell call; SME to confirm the named principal). | Observed from source — client confirms business meaning |
| 2 | 'C' | Zone code — restricts processing to dealer outlets whose `REGION_CD` begins with `C`; existing zone-C incentive data is cleared before the new run begins. | Observed from source — client confirms business meaning |

### 4.3 Trigger and schedule

The job is scheduled to run **at 01:00 on days 1 and 16 of every month** (server local time).

| Field | Value | Meaning |
| --- | --- | --- |
| Minute | 0 | at minute 00 |
| Hour | 1 | at hour 01 (24-hour clock) |
| Day of month | 1,16 | on days 1 and 16 of the month |
| Month | * | every month (`*`) |
| Day of week | * | every day of the week (`*`) |
| Cron path | /dmsdb/cron/PKG_SRV_INCENTIVE_SCH_C.sh | Matched from cron execution register |
| Crontab | `0 1 1,16 * *` | Five-field expression as recorded |

`*` means every value of that field (every minute / hour / day / month / weekday).
Day of week: `0` = Sunday, `1` = Monday, `2` = Tuesday, `3` = Wednesday, `4` = Thursday, `5` = Friday, `6` = Saturday.
Time is server local time. Time zone is not in the register.
Blackout windows, predecessors, and successor jobs are still N/A.

### 4.4 Success, failure, rerun

N/A — How success is judged, who is alerted, and same-day rerun policy require ops/client confirmation. Observed spool log path: `/home/dmsdb/logs/PKG_SRV_INCENTIVE_SCH_C.log`.

### 4.5 Profile and security

| Profile / sourced file | Present in ZIP? | Notes |
| --- | --- | --- |
| /dmsdb/cron/dms_cron_profile | Yes | Observed from source |

Credential variables observed: `$ORACLE_HOME`, `$PD`, `$UD`.

## 5. Functional / Business Requirements

*Numbered, testable requirements for job `PKG_SRV_INCENTIVE_SCH_C.sh`. Business-readable cards; technical names in backticks as anchors only. Evidence sourced from the entry-path JSON-LD slice and `PKG_SRV_BS_INCENTIVE_SCH.pck`.*

---

#### Requirement ID: BR-001

**Requirement Title:** Invoke the Bodyshop Incentive batch entry on schedule

**Business Description:**
Twice each month the job shall start a SQL\*Plus session, source the batch environment profile, and call `PKG_SRV_BS_INCENTIVE_SCH.SP_GEN_BS_INCENTIVE` with the principal identifier `1` and zone code `'C'` so the Bodyshop Incentive calculation for zone C can proceed. The job is scheduled to run **at 01:00 on days 1 and 16 of every month** (server local time).

**Actors:** N/A — no named business role in source

**Preconditions:**
- Batch profile `dms_cron_profile` is available and sources `ORACLE_HOME`, `UD`, and `PD` environment variables
- Database connectivity is available for the `sqlplus` session
- Package `PKG_SRV_BS_INCENTIVE_SCH` is compiled and accessible

**Main Flow:**
1. Source the batch credential profile to initialise the Oracle environment
2. Open a SQL\*Plus session and connect using the batch credentials
3. Begin spooling output to `/home/dmsdb/logs/PKG_SRV_INCENTIVE_SCH_C.log`
4. Record job start time
5. Execute `PKG_SRV_BS_INCENTIVE_SCH.SP_GEN_BS_INCENTIVE` with principal `1`, zone `'C'`
6. Record job end time
7. Close the spool and exit the SQL\*Plus session

**Technical evidence:**
- Entry call: `execute PKG_SRV_BS_INCENTIVE_SCH.Sp_Gen_BS_Incentive(1,'C');`
- Shell script: `PKG_SRV_INCENTIVE_SCH_C.sh`
- Profile sourced: `/dmsdb/cron/dms_cron_profile`
- Crontab: `0 1 1,16 * *` (source row 584, `cron_execution_schedule_runtime.xlsx`)

**Error:** N/A — no explicit shell-level failure handler coded; spool captures SQL\*Plus output

---

#### Requirement ID: BR-002

**Requirement Title:** Refresh Bodyshop Incentive data for zone C dealers

**Business Description:**
For each active dealer in zone C (`SUBSTR(region_cd,1,1) = 'C'`, dealer types `2S`/`3S`, category `DDL`) the job shall clear and repopulate the Bodyshop Incentive working tables with current-year and prior-year Bodyshop load and Smart Eye service data, then summarise the aggregated loads into the incentive result table `VD_SRV_INC`. The refresh covers the retention month derived from the run date so that every half-monthly execution reflects the most recent service activity from `VH_RO`.

**Actors:** N/A — no named business role in source

**Preconditions:**
- `AM_DEALER_LOC` contains valid dealer records for principal `1`, zone `'C'`
- `VH_RO` contains repair-order data with completed status (RO_STATUS = 40) and service category `BANDP`
- `VD_SRV_INC` and `VD_SRV_INC_VIN_WISE_DATA` are accessible for deletion and insertion

**Main Flow:**
1. Clear existing zone C records from `VD_SRV_INC_VIN_WISE_DATA` and `VD_SRV_INC` for the target zone
2. Determine the current-year start date, prior-year date range, and mid-month cut-off from the system date
3. For each qualifying dealer, stage current-year Bodyshop repair-order lines (service type `BANDP`, `RO_STATUS = 40`) into `VD_SRV_INC_VIN_WISE_DATA`
4. Stage prior-year Bodyshop repair-order lines into `VD_SRV_INC_VIN_WISE_DATA`
5. Stage current-year Smart Eye service records (service categories `PMS`, `RR`, `FR`, `FR1`–`FR4`, `BANDP`) into `VD_SRV_INC_VIN_WISE_DATA`
6. Insert aggregated current-year and prior-year Bodyshop load counts per dealer into `VD_SRV_INC`
7. Commit each batch of inserts as coded

**Technical evidence:**
- Tables written: `VD_SRV_INC_VIN_WISE_DATA` (INSERT, DELETE), `VD_SRV_INC` (INSERT, DELETE)
- Source read: `VH_RO` (repair orders), `AM_DEALER_LOC` (dealer master)
- Package body: `PKG_SRV_BS_INCENTIVE_SCH.pck`, procedure `SP_GEN_BS_INCENTIVE`

**Error:**
- `WHEN OTHERS` handler in the inner declare block sets `Pn_errcd := 1` and appends a message to `Pn_errmsg`; errors are captured but do not raise out of the procedure

---

#### Requirement ID: BR-003

**Requirement Title:** Write an operational run log

**Business Description:**
The job shall capture all SQL\*Plus output — including the procedure call, start and end timestamps, and any server output — in a spool log so that operations staff can confirm execution and investigate issues.

**Actors:** N/A — no named business role in source

**Preconditions:**
- The log directory `/home/dmsdb/logs/` is writable by the batch OS user

**Main Flow:**
1. Open spool at `/home/dmsdb/logs/PKG_SRV_INCENTIVE_SCH_C.log` before invoking the entry procedure
2. Capture start timestamp (`to_char(sysdate,'dd-mon-yy hh24:mi:ss')`)
3. Capture all server output from the procedure execution (`SET SERVEROUTPUT ON`)
4. Capture end timestamp
5. Close the spool and exit

**Technical evidence:**
- Spool path: `/home/dmsdb/logs/PKG_SRV_INCENTIVE_SCH_C.log`
- SQL\*Plus settings: `SET TIME ON TIMING ON`, `SET SERVEROUTPUT ON`

**Error:** N/A — no coded handler for write failures on the spool path

---

#### Requirement ID: BR-004

**Requirement Title:** Scope processing to principal 1 and zone C

**Business Description:**
The entry arguments `1` (principal identifier) and `'C'` (zone code) shall be passed to the incentive procedure so that data selection, deletion, and computation are restricted to the intended principal and geographic zone. Processing dealers or data outside this scope is not required for this job.

**Actors:** N/A

**Preconditions:**
- Parameters are passed by the shell script exactly as observed: `(1,'C')`

**Main Flow:**
1. Pass argument `1` as `Pn_Principal` to filter `AM_DEALER_LOC` by `principal_map_cd = 1`
2. Pass argument `'C'` as `Pn_Zone` to restrict records where `SUBSTR(region_cd,1,1) = 'C'`
3. Apply the same zone filter to delete and insert operations on `VD_SRV_INC_VIN_WISE_DATA` and `VD_SRV_INC`

**Technical evidence:**
- Shell entry: `execute PKG_SRV_BS_INCENTIVE_SCH.Sp_Gen_BS_Incentive(1,'C');`
- Parameter usage in body: `principal_map_cd = Pn_Principal` and `SUBSTR(region_cd,1,1) = Pn_Zone`

**Error:** N/A — no coded validation of parameter values

---

#### Requirement ID: BR-005

**Requirement Title:** Compute dealer incentive category and scoring marks

**Business Description:**
After staging Bodyshop load data, the job shall calculate each dealer group's incentive category (`A` or `B`) based on maximum average monthly Bodyshop load, compute load-growth marks and Smart Eye conversion marks, and write these computed values back to `VD_SRV_INC`. This scoring drives the incentive payout determination for zone C dealers.

**Actors:** N/A — no named business role in source

**Preconditions:**
- `VD_SRV_INC` contains staged load aggregates for the current retention month
- `VD_SRV_INC_VIN_WISE_DATA` contains staged VIN-wise Smart Eye records

**Main Flow:**
1. For each dealer group in `VD_SRV_INC` for the current retention month, derive the maximum average load from current-year and prior-year load counts
2. Assign incentive category `'A'` (load ≥ 500) or `'B'` (load ≥ 100); no category assigned otherwise
3. Calculate Bodyshop load growth percentage against the prior year and derive load-growth marks on a banded scale (0–50)
4. Count qualifying Smart Eye conversions (BANDP job within 30-day prior-service window, labour amount > 2,000, total ≤ 10,000) and compute the Smart Eye conversion percentage
5. Derive Smart Eye marks on a banded scale (0–50)
6. Update `VD_SRV_INC` with dealer category, growth percentage, load-growth marks, Smart Eye conversion count, total FPR count, conversion percentage, and Smart Eye marks

**Technical evidence:**
- Table updated: `VD_SRV_INC` (columns `DLR_CATEG`, `PARAM_VALUE5`–`PARAM_VALUE10`)
- Table read for Smart Eye: `VD_SRV_INC_VIN_WISE_DATA` (`PARAM_TYPE = 'lv_curr_year_smart_eye'`)
- Category thresholds: `Lv_max_load >= 500` → `'A'`; `>= 100` → `'B'`
- Marks banding: load-growth and Smart Eye conversion each on a 0–50 scale (see inner DECLARE block in `SP_GEN_BS_INCENTIVE`)

**Error:**
- `WHEN OTHERS` blocks on category, load-growth, and Smart Eye sections append error codes/messages to `Pn_errcd`/`Pn_errmsg`; execution continues

---

#### Requirement ID: BR-006

**Requirement Title:** Failure visibility via operational log

**Business Description:**
Any errors arising during the incentive calculation shall be captured and surfaced in the operational spool log so that the batch support team can identify and act on failures. The job does not raise a shell-level non-zero exit code from within the procedure; visibility relies on the spool log content.

**Actors:** N/A

**Preconditions:** Spool log is open (see BR-003)

**Main Flow:**
- Procedure-level exceptions are caught by `WHEN OTHERS` handlers; `Pn_errcd` and `Pn_errmsg` accumulate failure context and are available in server output

**Technical evidence:**
- `WHEN OTHERS` handlers in `SP_GEN_BS_INCENTIVE` set `Pn_errcd := 1` and append messages to `Pn_errmsg`
- No `RAISE` or `RAISE_APPLICATION_ERROR` coded; errors are non-fatal within the session

**Error:** N/A — no additional coded alerting beyond the spool log

## 6. Business Rules Catalogue

*Each row states one testable business rule evidenced in `PKG_SRV_BS_INCENTIVE_SCH.pck`. Mechanism detail is in the Source column. No rules invented from filenames or region codes alone.*

| Rule ID | Description | Source | Related BR |
| --- | --- | --- | --- |
| BRC-001 | Only dealers with `dealer_type` `'2S'` or `'3S'`, `DEALER_CATEGORY = 'DDL'`, and `msil_terminated_date >= 01-Mar-2022` (or null) shall be included in the incentive calculation for the run. | `PKG_SRV_BS_INCENTIVE_SCH.SP_GEN_BS_INCENTIVE` — Dealer cursor `WHERE` clause; `dealer_type IN ('2S','3S')`, `DEALER_CATEGORY = 'DDL'`, `NVL(msil_terminated_date, SYSDATE) >= '1-Mar-2022'` | BR-002 |
| BRC-002 | Regions `C1` and `C2` shall be reclassified to the combined code `'C12'` for all incentive calculations; all other region codes are used as-is. | `SP_GEN_BS_INCENTIVE` — Dealer cursor: `DECODE(a.REGION_CD, 'C1', 'C12', 'C2', 'C12', a.REGION_CD)` | BR-002 |
| BRC-003 | For dealerships of type `'2S'`, the `mul_srv_dealer_cd` field shall be sourced from `mul_srv_dealer_cd`; for all other types it shall be sourced from `mul_dealer_cd`. | `SP_GEN_BS_INCENTIVE` — Dealer cursor: `decode(a.dealer_type,'2S',a.mul_srv_dealer_cd,a.mul_dealer_cd)` | BR-002 |
| BRC-004 | Only repair orders with `RO_STATUS = 40` and service category `RCATEG_CD = 'BANDP'` shall be counted as Bodyshop load for current-year and prior-year load staging. | `SP_GEN_BS_INCENTIVE` — INSERT into `VD_SRV_INC_VIN_WISE_DATA` (current-year load block and prior-year load block): `RO_STATUS = 40 AND VH.RCATEG_CD = 'BANDP'` | BR-002, BR-005 |
| BRC-005 | The Bodyshop load count used to determine the dealer's incentive category shall be the **greater** of: (a) prior-year load divided by 12, and (b) current-year load divided by the number of elapsed months in the financial year. | `SP_GEN_BS_INCENTIVE` — inner DECLARE block: `GREATEST((JJ.PREV_YR_LOAD_CAT / 12), (JJ.CURR_YR_LOAD_CAT / Lv_no_of_months))` | BR-005 |
| BRC-006 | A dealer group shall be assigned incentive category `'A'` when its maximum average monthly Bodyshop load is ≥ 500, category `'B'` when ≥ 100 and < 500; no category is assigned below 100. | `SP_GEN_BS_INCENTIVE` — inner DECLARE block: `IF Lv_max_load >= 500 THEN 'A' ELSIF Lv_max_load >= 100 THEN 'B' END IF` | BR-005 |
| BRC-007 | Bodyshop load-growth marks shall be awarded on a banded scale: 50 marks for growth ≥ 20 %, a proportional interpolation between 21–45 for growth 15–20 %, between 1–20 for growth 10–15 %, and 0 marks below 10 %. | `SP_GEN_BS_INCENTIVE` — inner DECLARE block, load-growth marks calculation (`lv_marks_bs_load` branching IF/ELSIF) | BR-005 |
| BRC-008 | A repair order visit shall be counted as a qualifying Smart Eye conversion only when: the VIN has a prior service record (type `PMS`, `RR`, `FR`, `FR1`–`FR4`) within the 30-day window before the BANDP job date, `RO_STATUS = 40`, and at least one of the two records has `SMART_EYE_YN = 'Y'`; labour amount must exceed 2,000 and total invoice (parts + labour) must not exceed 10,000. | `SP_GEN_BS_INCENTIVE` — inner DECLARE block, Smart Eye sub-query on `VD_SRV_INC_VIN_WISE_DATA`: `r.ro_date BETWEEN (V.RO_DATE)-30 AND v.ro_date`, `LABOUR_AMT > 2000`, `(PART_AMT + LABOUR_AMT) <= 10000`, `NVL(R.SMART_EYE_YN,'N')='Y' OR NVL(V.SMART_EYE_YN,'N')='Y'`, `R.RO_STATUS = 40` | BR-005 |
| BRC-009 | Smart Eye conversion marks shall be awarded on a banded scale: 50 marks for conversion rate ≥ 15 %, interpolated 26–45 for 10–15 %, interpolated 11–25 for 8–10 %, interpolated 1–10 for 5–8 %, and 0 marks below 5 %. | `SP_GEN_BS_INCENTIVE` — inner DECLARE block, Smart Eye marks calculation (`lv_marks_smart_eye` branching IF/ELSIF) | BR-005 |
| BRC-010 | The mid-month cut-off date for as-on-date load counts shall be the 15th of the current month when the current day of month is greater than 15; otherwise it shall be the last day of the prior month. | `SP_GEN_BS_INCENTIVE` — `SELECT CASE WHEN to_number(to_char(sysdate,'dd')) > 15 THEN to_date('15-'||...) ELSE last_day(add_months(trunc(sysdate),-1)) END INTO Lv_Mid_End_date` | BR-002, BR-005 |
| BRC-011 | The current financial year shall start on 1 July of the current calendar year when the run month is after June; otherwise it shall start on 1 July of the prior calendar year. | `SP_GEN_BS_INCENTIVE` — `IF EXTRACT(MONTH FROM SYSDATE) > 6 THEN Lv_curr_yr_st_date := '01-JUL-'||YYYY ELSE '01-JUL-'||(YYYY-1)` | BR-002, BR-005 |
| BRC-012 | The prior financial year date range shall span exactly 12 months from 1 April of the year two years back (when run before April) or 1 April of the prior year (when run after March), ending on 31 March. | `SP_GEN_BS_INCENTIVE` — `IF EXTRACT(MONTH FROM SYSDATE) > 3 THEN Lv_prev_yr_st_date := '01-Apr-'||(YYYY-1) ELSE '01-Apr-'||(YYYY-2)`; `Lv_prev_yr_end_date := ADD_MONTHS(Lv_prev_yr_st_date,12)-1` | BR-002, BR-005 |

## 7. Data Requirements

### 7.1 Business entities

The following business entities are accessed by this job on its entry path. Each plain-language entity name is followed by the physical table name.

| Business Entity | Table |
| --- | --- |
| Dealer location master | (`AM_DEALER_LOC`) |
| Incentive result / scoring record | (`VD_SRV_INC`) |
| VIN-wise incentive detail record | (`VD_SRV_INC_VIN_WISE_DATA`) |
| Repair order / vehicle history | (`VH_RO`) |
| Scoring marks output | (`MARKS`) |

### 7.2 Data read / data written

Entry-path static SQL scan. Access modes are as determined by the code walker on `PKG_SRV_BS_INCENTIVE_SCH.SP_GEN_BS_INCENTIVE`. Not a live `ALL_DEPENDENCIES` query. `LV_*` scan names omitted here (see §10).

| Object | Type | Access (scan) | Notes |
| --- | --- | --- | --- |
| `AM_DEALER_LOC` | TABLE | READ | Dealer location master; filtered by `principal_map_cd`, `region_cd`, `dealer_type`, `DEALER_CATEGORY`, `msil_terminated_date` |
| `MARKS` | TABLE | WRITE | Identified by static scan; no explicit DML statement found in the reviewed package body — retain as scanner-reported |
| `VD_SRV_INC` | TABLE | READ/WRITE | Incentive result table; existing zone C rows deleted at job start, new aggregate rows inserted per dealer, then updated with category and marks |
| `VD_SRV_INC_VIN_WISE_DATA` | TABLE | READ/WRITE | VIN-level detail staging table; existing zone C rows deleted at job start, new rows inserted for current-year load, prior-year load, and Smart Eye data; re-read for aggregation and Smart Eye conversion counts |
| `VH_RO` | TABLE | READ | Repair-order source; filtered by dealer, bill date range, `RO_STATUS`, and `RCATEG_CD` |

Labelled **as-is from code** on the entry-path walk. Static scan — not live `ALL_DEPENDENCIES`.

### 7.2.1 Package-file objects not on this job's entry path

No `package_sibling_tables` or `off_chain_zip_units` were identified by the JSON-LD extractor for this job. This section is not applicable.

### 7.3 Reference / configuration data

The procedure derives all date-range boundaries (`Lv_curr_yr_st_date`, `Lv_curr_yr_End_date`, `Lv_prev_yr_st_date`, `Lv_prev_yr_end_date`, `Lv_Mid_End_date`) from `SYSDATE` at runtime using inline SQL against `DUAL`. No external configuration tables (e.g. `AM_LIST`) are read in the active code path — two `AM_LIST` queries appear in commented-out blocks and are therefore not active requirements.

### 7.4 Data lifecycle

At the start of each run, all `VD_SRV_INC_VIN_WISE_DATA` and `VD_SRV_INC` rows for zone C (`SUBSTR(REGION_CD,1,1) = 'C'`) are deleted and replaced with freshly computed data for the current retention month. The job is therefore a full replace (delete-then-insert) for the zone C slice of these two tables on each execution. Data for other zones is not touched. Retention policy beyond this job's run is N/A — SME/DBA to confirm.

### 7.5 Volumes and timing (optional)

N/A — no client-supplied row-count or volume estimates are present in the source.

## 8. Glossary

### 8.1 Business terms

| Term | Definition | Aliases | Related |
| --- | --- | --- | --- |
| Bodyshop Incentive Scheme | A periodic performance-based incentive programme that evaluates dealer outlets on bodyshop (Body & Paint) repair-order volumes and Smart Eye conversion rates, then assigns category ratings and marks that determine incentive pay-outs. | BS Incentive Scheme | `VD_SRV_INC`, `VD_SRV_INC_VIN_WISE_DATA`, `MARKS` |
| Retention Month | The calendar month (MON-YYYY format) used to tag all incentive records generated in a single batch run, allowing historical comparison without overwriting prior periods. | N/A | `VD_SRV_INC.RETENTION_MONTH` |
| Dealer Incentive Category | A classification (`A` or `B`) assigned to a parent-group dealer based on the greatest of their monthly-average prior-year or current-year B&P repair-order count; `A` ≥ 500 cases, `B` ≥ 100 cases. | N/A — SME | `VD_SRV_INC.DLR_CATEG` |
| BS Load Growth | The year-on-year percentage change in the count of qualifying B&P repair orders (labour amount > 2,000) for a dealer group, used to derive BS-load marks (0–50 scale). | N/A — SME | `VD_SRV_INC.PARAM_VALUE5` |
| Smart Eye Conversion | The percentage of B&P repair orders whose corresponding vehicle also had a qualifying multi-point service-type RO linked via Smart Eye flag (`QUOTE_ID`), used to derive Smart Eye marks (0–50 scale). | N/A — SME | `VD_SRV_INC.PARAM_VALUE9` |
| Zone | A single-character geographic subdivision (e.g., `C`) derived from the first character of `REGION_CD`; used to scope dealer selection and data deletion. | Region Zone | `AM_DEALER_LOC.REGION_CD` |

### 8.2 Technical terms

| Term | Definition |
| --- | --- |
| L1 / L2 / L3 | Dependency cascade levels from shell entry call |
| Package | Oracle PL/SQL package (``.pck``) |
| Standalone function | ``.fnc`` / SF_* unit |
| Profile | Shell ``source`` / ``.`` environment script |
| Spool | SQL*Plus spool log file |

### 8.3 Parameter index

| Name / call | Technical form | Notes |
| --- | --- | --- |
| PKG_SRV_BS_INCENTIVE_SCH.SP_GEN_BS_INCENTIVE | execute PKG_SRV_BS_INCENTIVE_SCH.Sp_Gen_BS_Incentive(1,'C'); | Entry call |
| PKG_SRV_INCENTIVE_SCH_C.LOG | PKG_SRV_INCENTIVE_SCH_C.LOG | Entry call |

## 9. Technical Inventory

### 9.1 Call chain

| Level | What | Name | Meaning | Source file |
| --- | --- | --- | --- | --- |
| 0 | SCAN L1 | PKG_SRV_INCENTIVE_SCH_C.sh | Start — main shell script | PKG_SRV_INCENTIVE_SCH_C.sh |
| 1 | FOUND L1 | PKG_SRV_BS_INCENTIVE_SCH.SP_GEN_BS_INCENTIVE | Named in the shell EXEC/CALL |  |
| 1 | FOUND L1 | PKG_SRV_INCENTIVE_SCH_C.LOG | Named in the shell EXEC/CALL |  |
| 1 | FOUND L1 | PKG_SRV_BS_INCENTIVE_SCH | Package source in ZIP | PKG_SRV_BS_INCENTIVE_SCH.pck |
| 1 | FOUND L1 | PKG_SRV_INCENTIVE_SCH_C | Package source in ZIP | PKG_SRV_INCENTIVE_SCH_C.sh |
| 2 | CASCADE L2 | PKG_SRV_BS_INCENTIVE_SCH | Referenced from entry path of PKG_SRV_INCENTIVE_SCH_C |  |
| 2 | FOUND L2 | PKG_SRV_BS_INCENTIVE_SCH | Package source in ZIP | PKG_SRV_BS_INCENTIVE_SCH.pck |
| 2 | FOUND L2 | SP_GEN_BS_INCENTIVE | Same-package helper defined in owning .pck (on entry path) | PKG_SRV_BS_INCENTIVE_SCH.pck |

### 9.2 Found source files (ZIP)

| File | Status |
| --- | --- |
| PKG_SRV_BS_INCENTIVE_SCH.pck | In ZIP |
| PKG_SRV_INCENTIVE_SCH_C.sh | In ZIP |

### 9.3 Missing / blocking

| Kind | Reference | Level |
| --- | --- | --- |
| — | None |  |

### 9.4 Explicitly not in this inventory

- Runtime executables such as `sqlplus` (listed informationally under executables)
- Environment variables as files
- Database tables as deliverable files
- Procedures in a `.pck` that are **not** on this call chain

### 9.5 Excel / JSON pointer

Companion artefacts for this run: `PKG_SRV_INCENTIVE_SCH_C_TraceabilityReport.md` (traceability) and `_cron_metrics.json` (deterministic counts).

## 10. Referenced Database Objects

### 10.1 Tables / views

| Name | Type | Access (scan) | One-line purpose |
| --- | --- | --- | --- |
| AM_DEALER_LOC | TABLE | READ | N/A |
| MARKS | TABLE | WRITE | N/A |
| VD_SRV_INC | TABLE | READ/WRITE | N/A |
| VD_SRV_INC_VIN_WISE_DATA | TABLE | READ/WRITE | N/A |
| VH_RO | TABLE | READ | N/A |

### 10.2 Other DB objects

Sequences / synonyms — only if seen in walked code (included above when classified).

**Note:** Static scan of supplied sources on the entry path where available, **not** a live `ALL_DEPENDENCIES` dump. Purpose column is `N/A` unless SME fills later.

## 11. Backend Packages and Procedures

### 11.1 Main flow index

shell → PKG_SRV_INCENTIVE_SCH_C.sh → L1: PKG_SRV_BS_INCENTIVE_SCH (PKG_SRV_BS_INCENTIVE_SCH.pck) → L1: PKG_SRV_INCENTIVE_SCH_C (PKG_SRV_INCENTIVE_SCH_C.sh) → L2: PKG_SRV_BS_INCENTIVE_SCH (PKG_SRV_BS_INCENTIVE_SCH.pck) → L2: SP_GEN_BS_INCENTIVE (PKG_SRV_BS_INCENTIVE_SCH.pck)

### 11.2 Catalogue

| Name | File | Type | Parent (found_from) | Role |
| --- | --- | --- | --- | --- |
| PKG_SRV_BS_INCENTIVE_SCH | PKG_SRV_BS_INCENTIVE_SCH.pck | package | PKG_SRV_INCENTIVE_SCH_C.sh | Provides the entry procedure `SP_GEN_BS_INCENTIVE` that generates Bodyshop Incentive Scheme data for a given principal and zone. |
| PKG_SRV_INCENTIVE_SCH_C | PKG_SRV_INCENTIVE_SCH_C.sh | package | PKG_SRV_INCENTIVE_SCH_C.sh | Shell-level module that invokes `PKG_SRV_BS_INCENTIVE_SCH.SP_GEN_BS_INCENTIVE` and directs logging via SQL*Plus spool. |
| PKG_SRV_BS_INCENTIVE_SCH | PKG_SRV_BS_INCENTIVE_SCH.pck | package | PKG_SRV_INCENTIVE_SCH_C | Provides the entry procedure `SP_GEN_BS_INCENTIVE` that generates Bodyshop Incentive Scheme data for a given principal and zone. |
| SP_GEN_BS_INCENTIVE | PKG_SRV_BS_INCENTIVE_SCH.pck | procedure | PKG_SRV_INCENTIVE_SCH_C | Generates Bodyshop Incentive Scheme records for dealer outlets in a specified zone by computing current- and prior-year bodyshop load, growth, Smart Eye conversion, and incentive-category marks, then persisting results to `VD_SRV_INC` and `VD_SRV_INC_VIN_WISE_DATA`. |

### 11.3 Package procedure specification (entry SP)

| Field | Value |
| --- | --- |
| Qualified name | `PKG_SRV_BS_INCENTIVE_SCH.SP_GEN_BS_INCENTIVE` |
| Raw call | `execute PKG_SRV_BS_INCENTIVE_SCH.Sp_Gen_BS_Incentive(1,'C');` |
| High-level steps | 1. **Clear zone data** — Delete all existing rows from `VD_SRV_INC_VIN_WISE_DATA` and `VD_SRV_INC` for the target zone (`Pn_Zone`), then commit. <br>2. **Initialise date range variables** — Derive the current-year start date (01-JUL of current or previous calendar year), the current-year end date (last day of prior month), a mid-month cut-off date (the 15th or prior month end), and the equivalent prior-year date window. <br>3. **Loop over eligible dealers** — Cursor `Dealer` selects active 2S/3S `DDL`-category outlets for the given principal (`Pn_Principal`) in the target zone from `AM_DEALER_LOC`. <br>4. **Load current-year Bodyshop (B&P) RO data** — For each dealer, insert qualifying repair-order rows (RO status 40, `RCATEG_CD = 'BANDP'`) from `VH_RO` into `VD_SRV_INC_VIN_WISE_DATA` with `PARAM_TYPE = 'lv_curr_yr_srv_bs_load'`. <br>5. **Load prior-year Bodyshop RO data** — Repeat the insert for the previous-year date window with `PARAM_TYPE = 'lv_prev_yr_srv_bs_load'`. <br>6. **Aggregate and insert load counts** — Count qualifying VINs for category and growth metrics from `VD_SRV_INC_VIN_WISE_DATA`, then insert a summary row per dealer into `VD_SRV_INC` (four `PARAM_VALUE` columns). <br>7. **Load Smart Eye conversion data** — Insert current-year RO rows (multiple service-type codes) into `VD_SRV_INC_VIN_WISE_DATA` with `PARAM_TYPE = 'lv_curr_year_smart_eye'`, flagging Smart Eye (`QUOTE_ID` presence). <br>8. **Calculate and update marks** — For each parent-group aggregate: derive dealer incentive category (`A`/`B`) from average load, calculate BS-load growth percentage and corresponding marks (scored 0–50), calculate Smart Eye conversion percentage and marks, then update `VD_SRV_INC` and `MARKS` with the computed scores. |

## 12. Shared Functions

| Function | File | Level | Found from |
| --- | --- | --- | --- |
| — | N/A | — | No helpers on this job's entry-path call chain |

Only helpers **called from the entry path** are listed. ZIP `.fnc` files not reached from the entry routine are inventory-only (see §9 / unclassified). Do **not** paste full function source here.

## 13. Interfaces

### 13.1 Database

This job talks to Oracle via `sqlplus` as observed in source.

### 13.2 External systems

N/A — No external system interfaces (file transfers, web services, message queues, or third-party APIs) are evidenced in the source. The job communicates exclusively with the Oracle database via `sqlplus`.

### 13.3 Output contract

- Log file path(s): `/home/dmsdb/logs/PKG_SRV_INCENTIVE_SCH_C.log` — operational, not a user report.
- Tables/views written by this job (entry-path static scan, WRITE access): `MARKS`, `VD_SRV_INC`, `VD_SRV_INC_VIN_WISE_DATA` — SME confirms downstream consumers.
- Read-only masters used as inputs are **not** listed here (see §7.2 / §10).

## 14. Non-Functional Requirements

*Operational and quality attributes for job `PKG_SRV_INCENTIVE_SCH_C.sh`. Cells marked N/A indicate the requirement is not evidenced in source; values must be confirmed by operations/client.*

| Topic | Requirement | Source |
| --- | --- | --- |
| Availability window | The job is scheduled at `0 1 1,16 * *` (01:00 on the 1st and 16th of every month). No blackout window or availability SLA is stated in source. N/A — Ops/client to confirm. | Cron execution register (row 584); no further ops evidence |
| Max runtime | N/A — no maximum elapsed-time constraint is coded or documented in source. Ops/client to define. | Ops/client |
| Rerun | N/A — no rerun guard, idempotency flag, or re-execution procedure is coded. The job performs a full delete-then-insert for the zone C slice on each invocation, so a rerun will repopulate the data; any partial prior run may leave incomplete data. Ops/client to confirm rerun policy. | Ops/client |
| Audit of who ran | The spool log at `/home/dmsdb/logs/PKG_SRV_INCENTIVE_SCH_C.log` captures start and end timestamps (`to_char(sysdate,'dd-mon-yy hh24:mi:ss')`). No database-level audit trail or `CREATED_BY` column is written by this job. Ops/client to confirm whether the spool log meets audit requirements. | Shell script `PKG_SRV_INCENTIVE_SCH_C.sh`; `CREATED_DATE` column written to `VD_SRV_INC` and `VD_SRV_INC_VIN_WISE_DATA` (SYSDATE only — no user identifier) |
| PII in logs | The spool log records SQL\*Plus timing output and server messages only; no personal data fields (names, VINs, contact details) are explicitly spooled. VIN values are processed in `VD_SRV_INC_VIN_WISE_DATA` but are not emitted to the spool by the coded `DBMS_OUTPUT` calls. N/A — confirm with data privacy/compliance team. | Shell script; `PKG_SRV_BS_INCENTIVE_SCH.pck` (commented-out `dbms_output.put_line` at end of procedure) |

## 15. Assumptions, Constraints, N/A Register

| Item | Classification | Notes |
| --- | --- | --- |
| Not an Oracle Report | N/A | No layout, SRW, RDF parameters, attached PLL, format triggers |
| Profile / sourced files outstanding | Constraint | None listed from ZIP resolve |
| BRD status | Assumption | As-is technical + business draft until SME sign-off |
| Schedule | Fact | Sourced from cron execution register — see §4.3 |
| Owners | Assumption | N/A |
| Static dependency scan | Constraint | Not live database dependency metadata |

## 16. Traceability Matrix

| BR-ID | Job step / package.sp | File | Test idea |
| --- | --- | --- | --- |
| BR-001 | Invoke PKG_SRV_BS_INCENTIVE_SCH.SP_GEN_BS_INCENTIVE | PKG_SRV_INCENTIVE_SCH_C.sh | Run job; confirm EXEC/entry call in spool log |
| BR-002 | Entry procedure data path → PKG_SRV_BS_INCENTIVE_SCH.SP_GEN_BS_INCENTIVE | PKG_SRV_BS_INCENTIVE_SCH.pck | Verify writes to entry-path WRITE tables (MARKS, VD_SRV_INC, VD_SRV_INC_VIN_WISE_DATA) |
| BR-003 | Spool log written | PKG_SRV_INCENTIVE_SCH_C.sh | Confirm file exists: /home/dmsdb/logs/PKG_SRV_INCENTIVE_SCH_C.log |
| BR-004 | Parameters on PKG_SRV_BS_INCENTIVE_SCH.SP_GEN_BS_INCENTIVE | PKG_SRV_INCENTIVE_SCH_C.sh | Confirm shell args match EXEC parameters |
| BR-005 | Helpers on entry path: N/A | §12 | Trace helper call sites in entry procedure body |
| BR-006 | Failure visibility | /home/dmsdb/logs/PKG_SRV_INCENTIVE_SCH_C.log | Confirm errors visible in spool / scheduler (ops) |
