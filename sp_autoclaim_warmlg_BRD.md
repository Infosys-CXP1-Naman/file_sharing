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
| Document Title | SP_AUTOCLAIM_WARMLG - Business Requirements Document |
| Job / Script Name | SP_AUTOCLAIM_WARMLG.sh |
| Document Version | 1.0 |
| Date Created | October 14, 2026 |
| Author/Prepared By | BRD Agent |
| Source Artifacts | `SP_AUTOCLAIM_WARMLG.sh`; `dms_cron_profile` |

### 1.2 Revision History

| Version | Date | Author | Description |
| --- | --- | --- | --- |
| 1.0 | October 14, 2026 | BRD Agent | Initial as-is draft from ZIP extraction |

## 2. Executive Summary

*High-level overview of the **SP_AUTOCLAIM_WARMLG** batch job capability and the business context evidenced in the source. Fill the placeholder with one short business-outcome paragraph plus at most one “how it runs” sentence — not a table/SQL dump.*

The **SP_AUTOCLAIM_WARMLG** batch job performs nightly auto-claim lot processing for the DMS environment. Each day at 01:30, it invokes three procedures from the `PKG_AUTO_CLAIMLOT_SRV` Oracle package — `SP_AUTOCLAIM_WARMLG`, `SP_AUTOCLAIM_EWCCP`, and `SP_AUTO_CLAIMLOT_GENERATE` — in sequence, ensuring claim lot data operations are carried out on a consistent daily cycle. The package body source was not available for review; the specific business data outcomes remain N/A pending SME confirmation. The job runs via `SP_AUTOCLAIM_WARMLG.sh`, which sources the DMS cron profile and connects to Oracle through `sqlplus`, writing execution results to an operational spool log.

### Key Stakeholders

*The source identifies batch execution context (shell, SQL\*Plus, Oracle credentials), but it does not name a business role, job title, approver, or process owner. No role is inferred for this baseline.*

| Stakeholder | Role |
| --- | --- |
| N/A | No named business actor or role appears in the source. The batch job runs under the OS/Oracle credentials supplied by the sourced profile; the person or team that owns the business outcome is not identified in the ZIP. |

### In scope / out of scope

**In scope**

- This job’s shell `SP_AUTOCLAIM_WARMLG.sh` and the database work it starts
- Entry routine `PKG_AUTO_CLAIMLOT_SRV.SP_AUTOCLAIM_WARMLG` and helpers on this job’s call chain (not other package members)
- Write the operational spool log at `/home/dmsdb/logs/SP_AUTOCLAIM_WAR.log`

**Out of scope**

- Other procedures in the same `.pck` that this job does not call
- UI / Oracle Forms modules
- Other regional script variants unless listed in this ZIP
- Time zone, blackout windows, Control-M predecessors/successors, business owners, and SLAs not present in source

### Open gaps

- package: `PKG_AUTO_CLAIMLOT_SRV`
- Business rules not signed off by SME
- Tables not delivered as DDL files (schema objects appear via static SQL scan only)

## 3. System Context

The **SP_AUTOCLAIM_WARMLG** batch job runs nightly to execute three sequential auto-claim lot processing procedures within the `PKG_AUTO_CLAIMLOT_SRV` Oracle package. Triggered at 01:30 each day, the job sources the DMS cron profile, connects to the Oracle database via `sqlplus`, and invokes `SP_AUTOCLAIM_WARMLG`, `SP_AUTOCLAIM_EWCCP`, and `SP_AUTO_CLAIMLOT_GENERATE` in sequence. Execution timestamps and output are written to an operational spool log. The package body was not available for review; internal data logic is confirmed N/A pending source delivery.

**System purpose:** Nightly automated execution of auto-claim lot processing procedures via the `PKG_AUTO_CLAIMLOT_SRV` Oracle package, invoked sequentially through `sqlplus` on a scheduled basis.

**In scope:**

- Run this batch job (`SP_AUTOCLAIM_WARMLG.sh`) under the sourced profile
- Invoke entry `PKG_AUTO_CLAIMLOT_SRV.SP_AUTOCLAIM_WARMLG` with the parameters observed in the shell
- Persist / update business data as implemented on the entry path
- Write the operational spool log when a path is present in source

**Process Name:** `PKG_AUTO_CLAIMLOT_SRV.SP_AUTOCLAIM_WARMLG` (business process label: Auto-Claim Lot Processing)

**Process Owner:** N/A — not identified in the available source artifacts

**Process Objective:** To carry out nightly auto-claim lot processing by sequentially executing `SP_AUTOCLAIM_WARMLG`, `SP_AUTOCLAIM_EWCCP`, and `SP_AUTO_CLAIMLOT_GENERATE` within the `PKG_AUTO_CLAIMLOT_SRV` package, ensuring the relevant claim lot data operations are completed each day before business activity resumes. The specific data outcomes (records updated, generated, or processed) are N/A pending delivery of the package body source.

**Process Trigger:** The job is scheduled to run **every day at 01:30** (server local time). Source: cron execution register.

**Process Flow:**

The verified batch process follows this sequence. Each step is mapped to its Section 5 requirement.

```
STEP 1 ── JOB START [BR-001]
         Scheduler invokes `SP_AUTOCLAIM_WARMLG.sh` on the batch host (scheduler identity: N/A).

STEP 2 ── SOURCE PROFILE [BR-001]
         The shell sources the cron profile (`/dmsdb/cron/dms_cron_profile`) to establish
         Oracle environment and credentials (`$ORACLE_HOME`, `$PD`, `$UD`).

STEP 3 ── CONNECT AND INVOKE ENTRY [BR-001]
         SQL\*Plus connects and executes `PKG_AUTO_CLAIMLOT_SRV.SP_AUTOCLAIM_WARMLG`.
         Package source: `N/A`.

STEP 4 ── PROCEDURE WORK [BR-002]
         The entry procedure performs the evidenced data read/write path
         (see §5 / §7 / §11). Sibling procedures in the same package that are
         not called by this job remain out of scope.

STEP 5 ── SPOOL AND EXIT [BR-003]
         SQL\*Plus writes the spool log (`/home/dmsdb/logs/SP_AUTOCLAIM_WAR.log`) and exits.
```

**Major process interactions:**

- **Oracle database** — entry package/procedure `PKG_AUTO_CLAIMLOT_SRV.SP_AUTOCLAIM_WARMLG` (file `N/A`)
- **Credential / profile** — `/dmsdb/cron/dms_cron_profile`
- **Operational log** — `/home/dmsdb/logs/SP_AUTOCLAIM_WAR.log`
- **Downstream consumers** — N/A — not identified in the available source artifacts

### Systems involved

| System | Evidence |
| --- | --- |
| OS / shell | Main script `SP_AUTOCLAIM_WARMLG.sh` |
| Oracle database | SQL*Plus / EXEC / anonymous PL/SQL block in source |
| Credential / profile | Sourced scripts (`/dmsdb/cron/dms_cron_profile`) |
| Downstream consumers | N/A — not identified in the available source artifacts |

### Runtime environment

Environment variables observed: `$ORACLE_HOME`, `$PD`, `$UD`

Log path(s): `/home/dmsdb/logs/SP_AUTOCLAIM_WAR.log`

Profile contents: N/A until profile file contents are reviewed from the ZIP (presence is recorded in §4.5).

## 4. Job Identity and Operations

*Batch identity, parameters, schedule posture, and operational controls evidenced for **SP_AUTOCLAIM_WARMLG**.*

### 4.1 Batch identity

| Field | Value |
| --- | --- |
| Script | SP_AUTOCLAIM_WARMLG.sh |
| Log | /home/dmsdb/logs/SP_AUTOCLAIM_WAR.log |
| Entry call | PKG_AUTO_CLAIMLOT_SRV.SP_AUTOCLAIM_WARMLG |
| Package file | N/A |

### 4.2 Parameters

| # | Technical value | Business meaning | Evidence |
| --- | --- | --- | --- |
| — | N/A | N/A | No parameters parsed from entry call |

### 4.3 Trigger and schedule

The job is scheduled to run **every day at 01:30** (server local time).

| Field | Value | Meaning |
| --- | --- | --- |
| Minute | 30 | at minute 30 |
| Hour | 1 | at hour 01 (24-hour clock) |
| Day of month | * | every day of the month (`*`) |
| Month | * | every month (`*`) |
| Day of week | * | every day of the week (`*`) |
| Cron path | /dmsdb/cron/SP_AUTOCLAIM_WARMLG.sh | Matched from cron execution register |
| Crontab | `30 1 * * *` | Five-field expression as recorded |

`*` means every value of that field (every minute / hour / day / month / weekday).
Day of week: `0` = Sunday, `1` = Monday, `2` = Tuesday, `3` = Wednesday, `4` = Thursday, `5` = Friday, `6` = Saturday.
Time is server local time. Time zone is not in the register.
Blackout windows, predecessors, and successor jobs are still N/A.

### 4.4 Success, failure, rerun

N/A — How success is judged, who is alerted, and same-day rerun policy require ops/client confirmation. Observed spool log path: `/home/dmsdb/logs/SP_AUTOCLAIM_WAR.log`.

### 4.5 Profile and security

| Profile / sourced file | Present in ZIP? | Notes |
| --- | --- | --- |
| /dmsdb/cron/dms_cron_profile | Yes | Observed from source |

Credential variables observed: `$ORACLE_HOME`, `$PD`, `$UD`.

## 5. Functional / Business Requirements

*Numbered, testable requirements for job `SP_AUTOCLAIM_WARMLG.sh`. Business-readable cards; technical names in backticks. Sourced from Cron JSON-LD and the shell script; N/A where SME confirmation is required.*

---

#### Requirement ID: BR-001

**Requirement Title:** Invoke the batch session and run the claim-lot entry sequence

**Business Description:**
Each night the job sources the batch environment profile, opens a database session via SQL\*Plus, and sequentially executes three procedures from `PKG_AUTO_CLAIMLOT_SRV` — `SP_AUTOCLAIM_WARMLG`, `SP_AUTOCLAIM_EWCCP`, and `SP_AUTO_CLAIMLOT_GENERATE` — within a single connected session. The job is scheduled to run **every day at 01:30** (server local time), as recorded in the cron execution register.

**Actors:** N/A — no named business role in source

**Preconditions:**
- Batch profile (`dms_cron_profile`) is present and sourced successfully
- Database credentials (`$UD` / `$PD`) are set in the environment
- SQL\*Plus is available under `$ORACLE_HOME/bin`

**Main Flow:**
1. Source the batch environment profile to set database connection variables
2. Open a SQL\*Plus session using the configured credentials
3. Record the session start timestamp
4. Execute `PKG_AUTO_CLAIMLOT_SRV.SP_AUTOCLAIM_WARMLG`
5. Execute `PKG_AUTO_CLAIMLOT_SRV.SP_AUTOCLAIM_EWCCP`
6. Execute `PKG_AUTO_CLAIMLOT_SRV.SP_AUTO_CLAIMLOT_GENERATE`
7. Record the session end timestamp and close the session

**Technical evidence:**
- Entry procedures (in call order): `PKG_AUTO_CLAIMLOT_SRV.SP_AUTOCLAIM_WARMLG`, `PKG_AUTO_CLAIMLOT_SRV.SP_AUTOCLAIM_EWCCP`, `PKG_AUTO_CLAIMLOT_SRV.SP_AUTO_CLAIMLOT_GENERATE`
- No arguments are passed to any procedure (all called with no parameters)
- Crontab entry: `30 1 * * *` (source row 691)

---

#### Requirement ID: BR-002

**Requirement Title:** Execute claim-lot processing procedures in declared order

**Business Description:**
The job shall invoke the three claim-lot procedures from `PKG_AUTO_CLAIMLOT_SRV` in the fixed order declared in the shell: warm-up (`SP_AUTOCLAIM_WARMLG`), followed by `SP_AUTOCLAIM_EWCCP`, then generation (`SP_AUTO_CLAIMLOT_GENERATE`). The business outcome of each step is N/A — SME confirmation is required because the package body is not present in the supplied source archive. Each procedure must complete without a raised exception before the session exits normally.

**Actors:** N/A

**Preconditions:**
- All three procedures exist and are executable within `PKG_AUTO_CLAIMLOT_SRV`
- The database session opened in BR-001 is active

**Main Flow:**
1. Call `PKG_AUTO_CLAIMLOT_SRV.SP_AUTOCLAIM_WARMLG` with no parameters; await completion
2. Call `PKG_AUTO_CLAIMLOT_SRV.SP_AUTOCLAIM_EWCCP` with no parameters; await completion
3. Call `PKG_AUTO_CLAIMLOT_SRV.SP_AUTO_CLAIMLOT_GENERATE` with no parameters; await completion
4. Allow the session to proceed to the end-timestamp query and clean exit

**Technical evidence:**
- All three calls are anonymous PL/SQL `BEGIN … END;` blocks executed sequentially in the same SQL\*Plus session
- Package file `PKG_AUTO_CLAIMLOT_SRV` is not present in the ZIP archive; procedure-level logic is N/A pending SME / source review

---

#### Requirement ID: BR-003

**Requirement Title:** Write an operational spool log for every run

**Business Description:**
The job shall capture all SQL\*Plus session output — including procedure server output, start/end timestamps, and any Oracle errors — into a persistent spool file so that operations teams can confirm the run completed and diagnose failures without re-executing the batch.

**Actors:** N/A — no named business role in source

**Preconditions:**
- The directory `/home/dmsdb/logs/` is writable by the batch OS user at runtime
- SQL\*Plus `SERVEROUTPUT` is enabled (`SET SERVEROUTPUT ON`)

**Main Flow:**
1. Open spool to `/home/dmsdb/logs/SP_AUTOCLAIM_WAR.log` before any procedure call
2. Enable server output and session timing (`SET TIME ON TIMING ON`)
3. Record start timestamp via `SELECT TO_CHAR(SYSDATE, …) AS STARTTIME FROM DUAL`
4. Capture all procedure output during the three calls (BR-002)
5. Record end timestamp via `SELECT TO_CHAR(SYSDATE, …) AS ENDTIME FROM DUAL`
6. Close the spool (`SPOOL OFF`) and exit SQL\*Plus

**Technical evidence:**
- Spool path: `/home/dmsdb/logs/SP_AUTOCLAIM_WAR.log`
- SQL\*Plus directives observed: `SPOOL`, `SET SERVEROUTPUT ON`, `SET TIME ON TIMING ON`

---

#### Requirement ID: BR-004

**Requirement Title:** Run with no input parameters — entry procedures take no arguments

**Business Description:**
Each of the three procedures is invoked without any arguments. No runtime filters, region codes, or date parameters are passed at the shell level; any internal scoping logic is encapsulated within `PKG_AUTO_CLAIMLOT_SRV`. The business meaning of any internal parameters is N/A pending package body review by SME.

**Actors:** N/A

**Preconditions:**
- Shell script is invoked by the cron scheduler with no additional arguments
- Procedure signatures within `PKG_AUTO_CLAIMLOT_SRV` accept zero parameters

**Main Flow:**
1. Scheduler triggers `SP_AUTOCLAIM_WARMLG.sh` with no command-line arguments
2. Shell passes no parameters to any of the three PL/SQL `BEGIN … END;` blocks
3. Each procedure determines its own processing scope internally

**Technical evidence:**
- Observed call syntax: `PKG_AUTO_CLAIMLOT_SRV.SP_AUTOCLAIM_WARMLG ;` (no arguments)
- Same pattern confirmed for `SP_AUTOCLAIM_EWCCP` and `SP_AUTO_CLAIMLOT_GENERATE`

---

#### Requirement ID: BR-005

**Requirement Title:** Source batch environment profile before opening the database session

**Business Description:**
Before any database activity, the job must load the shared batch environment profile (`dms_cron_profile`) to ensure that Oracle environment variables and database credentials are correctly set for the session. Without this step the SQL\*Plus connection cannot be established.

**Actors:** N/A

**Preconditions:**
- `/dmsdb/cron/dms_cron_profile` exists and is readable by the batch OS user

**Main Flow:**
1. Shell sources `/dmsdb/cron/dms_cron_profile` (`. /dmsdb/cron/dms_cron_profile`)
2. Environment variables `ORACLE_HOME`, `UD`, and `PD` become available to the shell
3. SQL\*Plus binary is located via `$ORACLE_HOME/bin`

**Technical evidence:**
- Sourced script: `dms_cron_profile` (resolved in ZIP; marked `blocking: true` in JSON-LD)
- Environment variables consumed: `$ORACLE_HOME`, `$UD`, `$PD`

---

#### Requirement ID: BR-006

**Requirement Title:** Failure visibility via spool log

**Business Description:**
Any Oracle exception or SQL\*Plus error raised during the session shall appear in the spool log at `/home/dmsdb/logs/SP_AUTOCLAIM_WAR.log`. No additional alerting mechanism is evidenced in the source; the spool log is the sole observable failure artifact for this job.

**Actors:** N/A

**Preconditions:** Spool log is open (BR-003)

**Main Flow:**
1. If a PL/SQL exception is raised, Oracle writes the error to server output, which is captured in the spool
2. SQL\*Plus session exits (cleanly or with error) and closes the spool
3. Operations review the spool log to determine run outcome

**Technical evidence:**
- No `WHENEVER SQLERROR EXIT` or explicit error-routing directive observed in the shell
- Failure diagnosis relies on spool content at `/home/dmsdb/logs/SP_AUTOCLAIM_WAR.log`

## 6. Business Rules Catalogue

| Rule ID | Description | Source | Related BR |
| --- | --- | --- | --- |
| BRC-001 | The three claim-lot procedures must be called in the fixed sequence `SP_AUTOCLAIM_WARMLG` → `SP_AUTOCLAIM_EWCCP` → `SP_AUTO_CLAIMLOT_GENERATE`; the shell enforces this order within a single SQL\*Plus session. | `SP_AUTOCLAIM_WARMLG.sh` — sequential `BEGIN … END;` blocks | BR-001, BR-002 |
| BRC-002 | The batch session must be established using credentials supplied entirely via environment variables (`$UD`, `$PD`); no credentials may be hard-coded in the shell script. | `SP_AUTOCLAIM_WARMLG.sh` — `connect $UD/$PD`; sourced from `dms_cron_profile` | BR-001, BR-005 |
| BRC-003 | Spool logging must be active before the first procedure call and must remain open until after the last procedure completes; the spool is closed only on `SPOOL OFF` preceding `EXIT`. | `SP_AUTOCLAIM_WARMLG.sh` — `SPOOL … / SPOOL OFF` wrapping all three `BEGIN … END;` blocks | BR-003 |
| BRC-004 | Procedure-level business rules governing claim-lot selection, eligibility, and generation are N/A — the package body (`PKG_AUTO_CLAIMLOT_SRV`) is not present in the supplied source archive; SME review is required before additional BRC rows can be authored. | Missing reference: `PKG_AUTO_CLAIMLOT_SRV` (JSON-LD `missing_references`) | BR-002 |

## 7. Data Requirements

### 7.1 Business entities

The job coordinates claim-lot processing through three procedures in the `PKG_AUTO_CLAIMLOT_SRV` package. Based on the shell script and JSON-LD evidence, the job interacts with claim-lot and auto-claim related business data managed internally by that package. The specific business entities (claim records, lot assignments, generation outputs) are N/A — the package body is absent from the source archive and must be confirmed by SME.

No database tables are statically resolvable from the entry-path walk (the package source was not available for analysis). Refer to §7.2 below.

### 7.2 Data read / data written

| Object | Type | Access (scan) | Notes |
| --- | --- | --- | --- |
| `PKG_AUTO_CLAIMLOT_SRV` | Oracle Package | EXECUTE | Package containing all three entry procedures; body not in ZIP archive — table-level access N/A |

Labelled **as-is from code** on the entry-path walk; not a full ERD. Static scan — not live `ALL_DEPENDENCIES`. No tables were resolved from the entry-path walk because `PKG_AUTO_CLAIMLOT_SRV` source is absent. `LV_*` scan names omitted here (see §10).

### 7.2.1 Package-file objects not on this job's entry path

No package sibling tables are resolvable from the supplied source archive. SME / DBA review of `PKG_AUTO_CLAIMLOT_SRV` is required to identify the complete set of tables accessed.

### 7.3 Reference / configuration data

The batch environment profile (`dms_cron_profile`) supplies Oracle connection parameters (`ORACLE_HOME`, `UD`, `PD`) used to establish the database session. No application-level reference tables or configuration keys are resolvable from the entry-path walk; N/A pending package body review.

### 7.4 Data lifecycle

N/A — data creation, update, retention, and archival rules are internal to `PKG_AUTO_CLAIMLOT_SRV` and cannot be determined without the package body source. SME confirmation required.

### 7.5 Volumes and timing (optional)

N/A

## 8. Glossary

### 8.1 Business terms

| Term | Definition | Aliases | Related |
| --- | --- | --- | --- |
| Auto-Claim Lot | A claim lot record subject to automatic processing by the `PKG_AUTO_CLAIMLOT_SRV` package procedures. Internal data definition: N/A — package body not delivered. | Claim lot | `PKG_AUTO_CLAIMLOT_SRV` |
| Claim Lot Processing | The batch operation that executes the auto-claim lot procedures (`SP_AUTOCLAIM_WARMLG`, `SP_AUTOCLAIM_EWCCP`, `SP_AUTO_CLAIMLOT_GENERATE`) in sequence each night. | Auto-claim batch | `SP_AUTOCLAIM_WARMLG.sh` |
| WARMLG | Token appearing in the entry procedure name `SP_AUTOCLAIM_WARMLG`. Business expansion: N/A — SME | N/A | `SP_AUTOCLAIM_WARMLG` |
| EWCCP | Token appearing in the sibling procedure name `SP_AUTOCLAIM_EWCCP`. Business expansion: N/A — SME | N/A | `SP_AUTOCLAIM_EWCCP` |
| DMS | Token appearing in profile path `/dmsdb/cron/dms_cron_profile`. Full expansion: N/A — SME | N/A | `dms_cron_profile` |

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
| PKG_AUTO_CLAIMLOT_SRV.SP_AUTOCLAIM_WARMLG | PKG_AUTO_CLAIMLOT_SRV.SP_AUTOCLAIM_WARMLG | Entry call |
| PKG_AUTO_CLAIMLOT_SRV.SP_AUTOCLAIM_EWCCP | PKG_AUTO_CLAIMLOT_SRV.SP_AUTOCLAIM_EWCCP | Entry call |
| PKG_AUTO_CLAIMLOT_SRV.SP_AUTO_CLAIMLOT_GENERATE | PKG_AUTO_CLAIMLOT_SRV.SP_AUTO_CLAIMLOT_GENERATE | Entry call |

## 9. Technical Inventory

### 9.1 Call chain

| Level | What | Name | Meaning | Source file |
| --- | --- | --- | --- | --- |
| 0 | SCAN L1 | SP_AUTOCLAIM_WARMLG.sh | Start — main shell script | SP_AUTOCLAIM_WARMLG.sh |
| 1 | FOUND L1 | PKG_AUTO_CLAIMLOT_SRV.SP_AUTOCLAIM_WARMLG | Named in the shell EXEC/CALL |  |
| 1 | FOUND L1 | PKG_AUTO_CLAIMLOT_SRV.SP_AUTOCLAIM_EWCCP | Named in the shell EXEC/CALL |  |
| 1 | FOUND L1 | PKG_AUTO_CLAIMLOT_SRV.SP_AUTO_CLAIMLOT_GENERATE | Named in the shell EXEC/CALL |  |
| 1 | MISSING L1 | PKG_AUTO_CLAIMLOT_SRV | Package file not in ZIP |  |

### 9.2 Found source files (ZIP)

| File | Status |
| --- | --- |
| SP_AUTOCLAIM_WARMLG.sh | In ZIP |

### 9.3 Missing / blocking

| Kind | Reference | Level |
| --- | --- | --- |
| package | PKG_AUTO_CLAIMLOT_SRV | 1 |

### 9.4 Explicitly not in this inventory

- Runtime executables such as `sqlplus` (listed informationally under executables)
- Environment variables as files
- Database tables as deliverable files
- Procedures in a `.pck` that are **not** on this call chain

### 9.5 Excel / JSON pointer

Companion artefacts for this run: `SP_AUTOCLAIM_WARMLG_TraceabilityReport.md` (traceability) and `_cron_metrics.json` (deterministic counts).

## 10. Referenced Database Objects

### 10.1 Tables / views

| Name | Type | Access (scan) | One-line purpose |
| --- | --- | --- | --- |
| — | — | — | N/A |

### 10.2 Other DB objects

Sequences / synonyms — only if seen in walked code (included above when classified).

**Note:** Static scan of supplied sources on the entry path where available, **not** a live `ALL_DEPENDENCIES` dump. Purpose column is `N/A` unless SME fills later.

## 11. Backend Packages and Procedures

### 11.1 Main flow index

`SP_AUTOCLAIM_WARMLG.sh` (shell, L0) → `PKG_AUTO_CLAIMLOT_SRV.SP_AUTOCLAIM_WARMLG` (L1, FOUND) → `PKG_AUTO_CLAIMLOT_SRV.SP_AUTOCLAIM_EWCCP` (L1, FOUND) → `PKG_AUTO_CLAIMLOT_SRV.SP_AUTO_CLAIMLOT_GENERATE` (L1, FOUND) → `PKG_AUTO_CLAIMLOT_SRV` (L1, MISSING — package body not in ZIP)

### 11.2 Catalogue

| Name | File | Type | Parent (found_from) | Role |
| --- | --- | --- | --- | --- |
| `PKG_AUTO_CLAIMLOT_SRV.SP_AUTOCLAIM_WARMLG` | N/A (package not in ZIP) | package_member | `SP_AUTOCLAIM_WARMLG.sh` | L1 entry call — first procedure invoked |
| `PKG_AUTO_CLAIMLOT_SRV.SP_AUTOCLAIM_EWCCP` | N/A (package not in ZIP) | package_member | `SP_AUTOCLAIM_WARMLG.sh` | L1 entry call — second procedure invoked |
| `PKG_AUTO_CLAIMLOT_SRV.SP_AUTO_CLAIMLOT_GENERATE` | N/A (package not in ZIP) | package_member | `SP_AUTOCLAIM_WARMLG.sh` | L1 entry call — third procedure invoked |
| `PKG_AUTO_CLAIMLOT_SRV` | N/A (not in ZIP) | package | `SP_AUTOCLAIM_WARMLG.sh` | L1 MISSING — package body not delivered in source archive |

### 11.3 Package procedure specification (entry SP)

| Field | Value |
| --- | --- |
| Qualified name | `PKG_AUTO_CLAIMLOT_SRV.SP_AUTOCLAIM_WARMLG` |
| Raw call | `PKG_AUTO_CLAIMLOT_SRV.SP_AUTOCLAIM_WARMLG` |
| High-level steps | Package source (`PKG_AUTO_CLAIMLOT_SRV`) was not delivered in the source ZIP; the internal body logic cannot be confirmed from static analysis. Based on the shell call sequence, the entry-path execution proceeds as follows: (1) The shell sources `/dmsdb/cron/dms_cron_profile` to establish Oracle runtime environment variables (`$ORACLE_HOME`, `$UD`, `$PD`). (2) `sqlplus` opens an anonymous PL/SQL block and executes `PKG_AUTO_CLAIMLOT_SRV.SP_AUTOCLAIM_WARMLG` — the primary entry procedure (internal logic: N/A — package body absent). (3) A second anonymous block executes `PKG_AUTO_CLAIMLOT_SRV.SP_AUTOCLAIM_EWCCP` (internal logic: N/A — package body absent). (4) A third anonymous block executes `PKG_AUTO_CLAIMLOT_SRV.SP_AUTO_CLAIMLOT_GENERATE` (internal logic: N/A — package body absent). (5) Start and end timestamps are selected from `DUAL` and written to the spool log at `/home/dmsdb/logs/SP_AUTOCLAIM_WAR.log`. (6) `sqlplus` spools off and exits cleanly. |

## 12. Shared Functions

| Function | File | Level | Found from |
| --- | --- | --- | --- |
| — | N/A | — | No helpers on this job's entry-path call chain |

Only helpers **called from the entry path** are listed. ZIP `.fnc` files not reached from the entry routine are inventory-only (see §9 / unclassified). Do **not** paste full function source here.

## 13. Interfaces

### 13.1 Database

This job talks to Oracle via `sqlplus` as observed in source.

### 13.2 External systems

N/A — No external system interfaces (file transfers, web services, messaging queues, or third-party APIs) are evidenced in the source artifacts. This job communicates exclusively with an Oracle database via `sqlplus`.

### 13.3 Output contract

- Log file path(s): `/home/dmsdb/logs/SP_AUTOCLAIM_WAR.log` — operational, not a user report.
- Tables/views written by this job (entry-path static scan, WRITE access): N/A — SME confirms downstream consumers.
- Read-only masters used as inputs are **not** listed here (see §7.2 / §10).

## 14. Non-Functional Requirements

| Topic | Requirement | Source |
| --- | --- | --- |
| Availability window | The job is scheduled daily at 01:30 (server local time; crontab `30 1 * * *`). The batch host and Oracle database must be available at that time. No formal SLA window is documented in the source. | Cron execution register (row 691); `SP_AUTOCLAIM_WARMLG.sh` |
| Max runtime | N/A — no maximum runtime or timeout is specified in the shell script or cron register. | Ops/client |
| Rerun | N/A — no rerun guard, idempotency marker, or manual re-trigger procedure is evidenced in the source. Rerun policy to be confirmed by Ops. | Ops/client |
| Audit of who ran | The spool log at `/home/dmsdb/logs/SP_AUTOCLAIM_WAR.log` captures start/end timestamps and all session output per run; no additional user-level audit trail is evidenced. | `SP_AUTOCLAIM_WARMLG.sh` — `SET TIME ON TIMING ON`; spool directives |
| PII in logs | N/A — the spool log captures SQL\*Plus server output; whether procedure output contains personally identifiable information cannot be determined without the `PKG_AUTO_CLAIMLOT_SRV` package body. SME review required. | Ops/client; missing `PKG_AUTO_CLAIMLOT_SRV` source |

## 15. Assumptions, Constraints, N/A Register

| Item | Classification | Notes |
| --- | --- | --- |
| Not an Oracle Report | N/A | No layout, SRW, RDF parameters, attached PLL, format triggers |
| Profile / sourced files outstanding | Constraint | package:PKG_AUTO_CLAIMLOT_SRV |
| BRD status | Assumption | As-is technical + business draft until SME sign-off |
| Schedule | Fact | Sourced from cron execution register — see §4.3 |
| Owners | Assumption | N/A |
| Static dependency scan | Constraint | Not live database dependency metadata |

## 16. Traceability Matrix

| BR-ID | Job step / package.sp | File | Test idea |
| --- | --- | --- | --- |
| BR-001 | Invoke PKG_AUTO_CLAIMLOT_SRV.SP_AUTOCLAIM_WARMLG | SP_AUTOCLAIM_WARMLG.sh | Run job; confirm EXEC/entry call in spool log |
| BR-002 | Entry procedure data path → PKG_AUTO_CLAIMLOT_SRV.SP_AUTOCLAIM_WARMLG | N/A | Verify writes to entry-path WRITE tables |
| BR-003 | Spool log written | SP_AUTOCLAIM_WARMLG.sh | Confirm file exists: /home/dmsdb/logs/SP_AUTOCLAIM_WAR.log |
| BR-004 | Parameters on PKG_AUTO_CLAIMLOT_SRV.SP_AUTOCLAIM_WARMLG | SP_AUTOCLAIM_WARMLG.sh | Confirm shell args match EXEC parameters |
| BR-005 | Source `/dmsdb/cron/dms_cron_profile` before opening database session | SP_AUTOCLAIM_WARMLG.sh | Confirm profile sourced; env vars (`$ORACLE_HOME`, `$UD`, `$PD`) set before `sqlplus` call |
| BR-006 | Failure visibility | /home/dmsdb/logs/SP_AUTOCLAIM_WAR.log | Confirm errors visible in spool / scheduler (ops) |
