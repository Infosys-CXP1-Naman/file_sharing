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
| Document Title | GEN_SAC_INVOICE - Business Requirements Document |
| Job / Script Name | GEN_SAC_INVOICE.sh |
| Document Version | 1.0 |
| Date Created | October 26, 2026 |
| Author/Prepared By | BRD Agent |
| Source Artifacts | `GEN_SAC_INVOICE.sh`; `dms_cron_profile`; `PKG_ADDON_SALE.pck`; `PKG_COMMUNICATION.pck`; `PKG_LOYALTY.pck`; `PKG_SERV_PROCESS_DATA.pck`; `FNC_VERIF_MOBILE.fnc`; `SF_PRINCIPAL_MAP_CD.fnc`; `PKG_EINV.pck`; `PKG_FDI_SERV.pck`; `PKG_FDI.pck` |

### 1.2 Revision History

| Version | Date | Author | Description |
| --- | --- | --- | --- |
| 1.0 | October 26, 2026 | BRD Agent | Initial as-is draft from ZIP extraction |

## 2. Executive Summary

*High-level overview of the **GEN_SAC_INVOICE** batch job capability and the business context evidenced in the source. Fill the placeholder with one short business-outcome paragraph plus at most one “how it runs” sentence — not a table/SQL dump.*

The `GEN_SAC_INVOICE` job automates the end-to-end generation of Service Add-on Contract (SAC) invoices at dealerships, ensuring that every qualifying SAC order is converted into a committed, GST-compliant invoice, loyalty transactions are credited to the customer's account, and a post-sale notification is sent — all without manual intervention. The job runs every five minutes: the shell script `GEN_SAC_INVOICE.sh` sources the Oracle environment profile, connects via `sqlplus`, and executes `PKG_ADDON_SALE.SP_GEN_SAC_INVOICE(1)`, which orchestrates order processing, e-invoice creation, CCP/FDI data staging, and SMS dispatch before writing the result to the operational spool log.

### Key Stakeholders

*The source identifies batch execution context (shell, SQL\*Plus, Oracle credentials), but it does not name a business role, job title, approver, or process owner. No role is inferred for this baseline.*

| Stakeholder | Role |
| --- | --- |
| N/A | No named business actor or role appears in the source. The batch job runs under the OS/Oracle credentials supplied by the sourced profile; the person or team that owns the business outcome is not identified in the ZIP. |

### In scope / out of scope

**In scope**

- This job’s shell `GEN_SAC_INVOICE.sh` and the database work it starts
- Entry routine `PKG_ADDON_SALE.SP_GEN_SAC_INVOICE` and helpers on this job’s call chain (not other package members)
- Write the operational spool log at `/home/dmsdb/logs/GEN_SAC_INVOICE.log`

**Out of scope**

- Other procedures in the same `.pck` that this job does not call
- UI / Oracle Forms modules
- Other regional script variants unless listed in this ZIP
- Time zone, blackout windows, Control-M predecessors/successors, business owners, and SLAs not present in source

### Open gaps

- routine: `SF_GET_MODEL_CHANNEL`
- package: `PKG_SFDC_SMS`
- routine: `SEND_SMS`
- package: `PKG_ENQ_POC_DETAIL`
- package: `PKG_SPR_EINV`
- routine: `SF_OLD_VAR_DESC`
- routine: `SP_PUSH_EWC_FDI_DATA`
- routine: `SP_PUSH_EWC_FDI_DATA_SM`
- routine: `SP_PUSH_EWU_FDI_DATA`
- routine: `SP_PUSH_EWU_FDI_DATA_SM`
- routine: `SP_PUSH_EW_FDI_DATA`
- routine: `SP_PUSH_EW_FDI_DATA_SM`
- routine: `SP_PUSH_MCPC_FDI_DATA`
- routine: `SP_PUSH_MCP_FDI_DATA`
- package: `PKG_SFDC_SMS`
- routine: `SEND_SMS`
- routine: `SF_GET_BONUS_AGE_SLAB`
- routine: `SF_GET_GRN_DISCOUNT`
- routine: `SF_GET_GST_CHARGES_PR`
- routine: `SF_GET_HSN`
- routine: `SF_GET_TOTAL_BONUS`
- routine: `SP_PUSH_COUNTERSALE_INVOICE`
- routine: `SP_PUSH_COUNTERSALE_RETURN`
- routine: `SP_PUSH_PART_RECIPT`
- Business rules not signed off by SME
- Tables not delivered as DDL files (schema objects appear via static SQL scan only)

## 3. System Context

*Describes the **GEN_SAC_INVOICE** job's verified scope, process flow, and data interactions. System purpose / Process Objective = business outcome; technical path stays in Process Flow STEPs below.*

This batch job automates the generation of Service Add-on Contract (SAC) invoices for dealer locations. Running every five minutes, it picks up SAC orders that are ready to be invoiced, applies loyalty programme logic, produces the required e-invoice records, and triggers post-invoice customer notifications. The job runs end-to-end without human intervention and is the primary mechanism by which SAC sale events are converted into committed financial and loyalty records in the database.

**System purpose:** To automatically generate SAC invoices, record associated add-on and loyalty transactions, produce GST-compliant e-invoice entries, and dispatch customer notifications — ensuring that each qualifying SAC order placed at a dealer is invoiced, compliance-stamped, and communicated to the customer in near real time.

**In scope:**

- Run this batch job (`GEN_SAC_INVOICE.sh`) under the sourced profile
- Invoke entry `PKG_ADDON_SALE.SP_GEN_SAC_INVOICE` with the parameters observed in the shell
- Persist / update business data as implemented on the entry path
- Write the operational spool log when a path is present in source

**Process Name:** `PKG_ADDON_SALE.SP_GEN_SAC_INVOICE` (business process label: SAC Invoice Generation)

**Process Owner:** N/A — not identified in the available source artifacts

**Process Objective:** To process all pending SAC orders at dealer locations, issue the corresponding invoices (including GST-compliant e-invoices and QR codes), credit loyalty points to eligible customers, and deliver post-sale SMS notifications — so that each SAC sale is fully invoiced and recorded within the next scheduled run cycle.

**Process Trigger:** The job is scheduled to run **every 5 minutes** (server local time). Source: cron execution register.

**Process Flow:**

The verified batch process follows this sequence. Each step is mapped to its Section 5 requirement.

```
STEP 1 ── JOB START [BR-001]
         Scheduler invokes `GEN_SAC_INVOICE.sh` on the batch host (scheduler identity: N/A).

STEP 2 ── SOURCE PROFILE [BR-001]
         The shell sources the cron profile (`/dmsdb/cron/dms_cron_profile`) to establish
         Oracle environment and credentials (`$ORACLE_HOME`, `$PD`, `$UD`).

STEP 3 ── CONNECT AND INVOKE ENTRY [BR-001]
         SQL\*Plus connects and executes `PKG_ADDON_SALE.SP_GEN_SAC_INVOICE(1)`.
         Package source: `PKG_ADDON_SALE.pck`.

STEP 4 ── PROCEDURE WORK [BR-002]
         The entry procedure performs the evidenced data read/write path
         (see §5 / §7 / §11). Sibling procedures in the same package that are
         not called by this job remain out of scope.

STEP 5 ── SPOOL AND EXIT [BR-003]
         SQL\*Plus writes the spool log (`/home/dmsdb/logs/GEN_SAC_INVOICE.log`) and exits.
```

**Major process interactions:**

- **Oracle database** — entry package/procedure `PKG_ADDON_SALE.SP_GEN_SAC_INVOICE` (file `PKG_ADDON_SALE.pck`)
- **Credential / profile** — `/dmsdb/cron/dms_cron_profile`
- **Operational log** — `/home/dmsdb/logs/GEN_SAC_INVOICE.log`
- **Downstream consumers** — N/A — no downstream system or process is named in the source; SMS and FDI payloads are staged to database tables for consumption by external processes not identified in this ZIP.

### Systems involved

| System | Evidence |
| --- | --- |
| OS / shell | Main script `GEN_SAC_INVOICE.sh` |
| Oracle database | SQL*Plus / EXEC / anonymous PL/SQL block in source |
| Credential / profile | Sourced scripts (`/dmsdb/cron/dms_cron_profile`) |
| Downstream consumers | N/A — not identified in source artifacts |

### Runtime environment

Environment variables observed: `$ORACLE_HOME`, `$PD`, `$UD`

Log path(s): `/home/dmsdb/logs/GEN_SAC_INVOICE.log`

Profile contents: N/A until profile file contents are reviewed from the ZIP (presence is recorded in §4.5).

## 4. Job Identity and Operations

*Batch identity, parameters, schedule posture, and operational controls evidenced for **GEN_SAC_INVOICE**.*

### 4.1 Batch identity

| Field | Value |
| --- | --- |
| Script | GEN_SAC_INVOICE.sh |
| Log | /home/dmsdb/logs/GEN_SAC_INVOICE.log |
| Entry call | PKG_ADDON_SALE.SP_GEN_SAC_INVOICE(1) |
| Package file | PKG_ADDON_SALE.pck |

### 4.2 Parameters

| # | Technical value | Business meaning | Evidence |
| --- | --- | --- | --- |
| 1 | 1 | Execution mode flag passed to `SP_GEN_SAC_INVOICE`; exact business meaning (e.g., production run, all-dealer scope, or a specific mode selector) is N/A — SME confirmation required. | Observed from source — client confirms business meaning |

### 4.3 Trigger and schedule

The job is scheduled to run **every 5 minutes** (server local time).

| Field | Value | Meaning |
| --- | --- | --- |
| Minute | 00,05,10,15,20,25,30,35,40,45,50,55 | at minutes 00, 05, 10, 15, 20, 25, 30, 35, 40, 45, 50, and 55 |
| Hour | * | every hour (`*`) |
| Day of month | * | every day of the month (`*`) |
| Month | * | every month (`*`) |
| Day of week | * | every day of the week (`*`) |
| Cron path | /dmsdb/cron/GEN_SAC_INVOICE.sh | Matched from cron execution register |
| Crontab | `00,05,10,15,20,25,30,35,40,45,50,55 * * * *` | Five-field expression as recorded |

`*` means every value of that field (every minute / hour / day / month / weekday).
Day of week: `0` = Sunday, `1` = Monday, `2` = Tuesday, `3` = Wednesday, `4` = Thursday, `5` = Friday, `6` = Saturday.
Time is server local time. Time zone is not in the register.
Blackout windows, predecessors, and successor jobs are still N/A.

### 4.4 Success, failure, rerun

N/A — How success is judged, who is alerted, and same-day rerun policy require ops/client confirmation. Observed spool log path: `/home/dmsdb/logs/GEN_SAC_INVOICE.log`.

### 4.5 Profile and security

| Profile / sourced file | Present in ZIP? | Notes |
| --- | --- | --- |
| /dmsdb/cron/dms_cron_profile | Yes | Observed from source |

Credential variables observed: `$ORACLE_HOME`, `$PD`, `$UD`.

## 5. Functional / Business Requirements

*Numbered, testable requirements for job `GEN_SAC_INVOICE.sh`. Business-readable cards; technical names in backticks. Evidence sourced from the Cron JSON-LD entry-path walk and `PKG_ADDON_SALE.pck` procedure body.*

---

#### Requirement ID: BR-001

**Requirement Title:** Invoke the SAC invoice generation entry procedure on schedule

**Business Description:**
Every 5 minutes the job shall source the batch environment profile and open a database session to call `PKG_ADDON_SALE.SP_GEN_SAC_INVOICE(1)`, ensuring that pending SAC (Service Add-on Contract) orders are converted into invoices on a near-real-time cadence. The job is scheduled to run **every 5 minutes** (server local time), as recorded in the cron execution register. The single numeric argument `1` is the `principal_map_cd` scope passed to the entry procedure.

**Actors:** N/A — no named business role in source

**Preconditions:**
- Batch profile `/dmsdb/cron/dms_cron_profile` is present and sets `ORACLE_HOME`, `UD`, and `PD` environment variables
- Oracle database session can be opened with the credentials provided by the profile

**Main Flow:**
1. Source the batch environment profile to load database credentials
2. Open a SQL\*Plus session and connect with the resolved credentials
3. Invoke `PKG_ADDON_SALE.SP_GEN_SAC_INVOICE(1)` to process pending SAC orders for `principal_map_cd = 1`
4. Capture start and end timestamps in the run log
5. Spool output to the operational log and exit the SQL\*Plus session

**Technical evidence:**
- Entry call: `PKG_ADDON_SALE.SP_GEN_SAC_INVOICE(1)`
- Profile sourced: `/dmsdb/cron/dms_cron_profile`
- Environment variables used: `ORACLE_HOME`, `UD`, `PD`
- Crontab: `00,05,10,15,20,25,30,35,40,45,50,55 * * * *`

---

#### Requirement ID: BR-002

**Requirement Title:** Convert qualifying SAC orders into add-on invoice records

**Business Description:**
For each pending SAC order in `vt_sac_order` that has not yet been invoiced (`txn_status = 'N'`), is not cancelled, was placed within the last day, and has an associated service job card in a completed state (`ro_status = '40'` with SAC CIC integration enabled), the job shall generate a unique add-on certificate number and invoice number via `SP_GEN_SEQ_NUM`, and insert a fully populated invoice record into `vt_addon`. Upon successful insertion, the originating order row in `vt_sac_order` is updated to `txn_status = 'S'` to mark it as processed. Orders for which generation fails are rolled back; successfully processed orders are committed individually.

**Actors:** N/A — no named business role in source

**Preconditions:**
- Unprocessed SAC orders (`txn_status = 'N'`, `cancel_yn = 'N'`, `order_date >= TRUNC(SYSDATE) - 1`) exist in `vt_sac_order`
- The associated service job card (`vh_ro`) has `ro_status = '40'` and `sac_cic_int_yn = 'Y'`
- Sequence generation helper `SP_GEN_SEQ_NUM` is available

**Main Flow:**
1. Read pending SAC orders from `vt_sac_order` for the target `principal_map_cd`, ordered by `order_no`
2. For each order, confirm the linked service job card is in a completed, CIC-enabled state via `vh_ro`
3. Resolve the customer's city description from `gm_city`
4. Generate a unique add-on certificate number (type `SAC`) and invoice number (type `SCI`) using `SP_GEN_SEQ_NUM`
5. Insert the invoice record — including GST rates, loyalty amounts, and service type flag — into `vt_addon`
6. Update the source order row in `vt_sac_order` with the generated numbers and set `txn_status = 'S'`
7. Commit on success; roll back on any error for that order

**Technical evidence:**
- Source cursor: `vt_sac_order` filtered by `principal_map_cd`, `order_date >= TRUNC(SYSDATE) - 1`, `txn_status = 'N'`, `cancel_yn = 'N'`
- Job card check: `vh_ro` where `sac_cic_int_yn = 'Y'` and `ro_status = '40'`
- Sequence generator: `PKG_ADDON_SALE.SP_GEN_SEQ_NUM` (types `'SAC'`, `'SCI'`)
- Invoice insert target: `vt_addon`
- Order status update: `vt_sac_order.txn_status = 'S'`

---

#### Requirement ID: BR-003

**Requirement Title:** Write an operational run log

**Business Description:**
The job shall spool all SQL\*Plus output — including start time, end time, and any procedure output — to the operational log file at `/home/dmsdb/logs/GEN_SAC_INVOICE.log`, so that operations teams can confirm each run and investigate any failures.

**Actors:** N/A — no named business role in source

**Preconditions:**
- Log directory `/home/dmsdb/logs/` is writable on the batch host

**Main Flow:**
1. Before invoking the entry procedure, open the spool to `/home/dmsdb/logs/GEN_SAC_INVOICE.log`
2. Record the start timestamp (`to_char(sysdate, 'dd-mon-yy hh24:mi:ss')`)
3. Capture all SQL\*Plus server output and timing during the procedure run
4. Record the end timestamp
5. Close the spool and exit the session

**Technical evidence:**
- Spool path: `/home/dmsdb/logs/GEN_SAC_INVOICE.log`
- Shell directives: `set serveroutput on`, `set time on timing on`

---

#### Requirement ID: BR-004

**Requirement Title:** Scope invoice generation to the configured principal map code

**Business Description:**
The job shall restrict SAC invoice generation to the dealer principal scope identified by the argument passed at invocation — currently `1`, representing `principal_map_cd = 1`. Only SAC orders and supporting reference data matching this principal scope shall be selected for processing. This ensures the batch run does not cross principal boundaries.

**Actors:** N/A — no named business role in source

**Preconditions:**
- The numeric argument is present in the shell entry call
- Corresponding orders exist in `vt_sac_order` for that `principal_map_cd`

**Main Flow:**
1. Accept the `principal_map_cd` argument from the shell call
2. Pass the value as the `pn_pmc` parameter into `PKG_ADDON_SALE.SP_GEN_SAC_INVOICE`
3. Apply the value as a filter on `vt_sac_order` and all supporting lookups (e.g., `gm_city`, `gd_loyalty_enrol`)
4. All inserts and updates are scoped to that principal

**Technical evidence:**
- Entry: `PKG_ADDON_SALE.SP_GEN_SAC_INVOICE(1)`
- Parameter declared as `pn_pmc IN NUMBER` in `PKG_ADDON_SALE.pck`
- Filter applied: `WHERE x.principal_map_cd = pn_pmc` in the `vt_sac_order` cursor

---

#### Requirement ID: BR-005

**Requirement Title:** Trigger customer notifications and loyalty award transactions on successful invoice creation

**Business Description:**
After a SAC add-on invoice is successfully inserted, the job shall, where applicable, send an SMS notification to the customer and record a loyalty award transaction. Mobile number validity is checked using `FNC_VERIF_MOBILE` before any SMS dispatch; if valid, `SP_PUSH_CCPSMS` sends the purchase confirmation SMS and `PKG_ADDON_SALE.SP_SEND_SMS_SRV_CCP_SALE` sends the service CCP sale notification. Where the customer has a loyalty card linked to their vehicle, `SP_GET_VIN_LOYALTY_DTL` retrieves loyalty details, `SF_CALCULATE_RUPEES_TO_POINTS` computes the award points, `SP_INS_LOYALTY_TRANSACTIONS` records the transaction (type `CCP`), and `SP_SEND_SMS` dispatches the award notification.

**Actors:** N/A — no named business role in source

**Preconditions:**
- Invoice insert into `vt_addon` succeeded with no error (`ln_err_cd = 0`)
- Customer mobile is non-null and passes `FNC_VERIF_MOBILE` check for SMS path
- Customer VIN is linked to a loyalty card in `gd_loyalty_card_vin_map` / `gd_loyalty_enrol` for loyalty path

**Main Flow:**
1. After successful invoice insert, check customer mobile validity via `FNC_VERIF_MOBILE`
2. If valid, dispatch purchase SMS via `SP_PUSH_CCPSMS` (type `CCP_PURCHASE`) and service sale notification via `SP_SEND_SMS_SRV_CCP_SALE`
3. Retrieve loyalty details for the customer's VIN via `SP_GET_VIN_LOYALTY_DTL`
4. If a loyalty card is found, compute award points via `SF_CALCULATE_RUPEES_TO_POINTS` using transaction amount, loyalty type, channel, and tier
5. Insert the loyalty award transaction (description `'Customer convenience package'`) via `SP_INS_LOYALTY_TRANSACTIONS`
6. If loyalty insert succeeds, dispatch the award SMS via `SP_SEND_SMS` (type `'AWARD'`)

**Technical evidence:**
- Mobile validation: `FNC_VERIF_MOBILE` (standalone function, `FNC_VERIF_MOBILE.fnc`)
- SMS push: `SP_PUSH_CCPSMS` (in `PKG_ADDON_SALE.pck`), `PKG_ADDON_SALE.SP_SEND_SMS_SRV_CCP_SALE`
- Loyalty detail retrieval: `SP_GET_VIN_LOYALTY_DTL` (in `PKG_ADDON_SALE.pck`, delegates to `PKG_LOYALTY`)
- Points calculation: `SF_CALCULATE_RUPEES_TO_POINTS` (in `PKG_ADDON_SALE.pck`)
- Loyalty transaction insert: `SP_INS_LOYALTY_TRANSACTIONS` (in `PKG_ADDON_SALE.pck`)
- Award SMS: `SP_SEND_SMS` (in `PKG_ADDON_SALE.pck`)

---

#### Requirement ID: BR-006

**Requirement Title:** Generate QR code for unregistered GST customers

**Business Description:**
Where the customer's GST number is `'GSTUNREGISTERED'`, the job shall trigger the B2C e-invoice QR code generation process via `VAL_INS_INVOICES_B2C_CCP` (which delegates to `PKG_SERV_PROCESS_DATA`), ensuring that unregistered customers receive the appropriate QR-tagged invoice as required for B2C GST compliance.

**Actors:** N/A — no named business role in source

**Preconditions:**
- Invoice insert into `vt_addon` succeeded with no error
- Customer `cust_gst_num` equals `'GSTUNREGISTERED'`

**Main Flow:**
1. After successful invoice insert, check the customer's `cust_gst_num`
2. If equal to `'GSTUNREGISTERED'`, call `VAL_INS_INVOICES_B2C_CCP` with document type `'ECS'` and invoice type `'ECS'`
3. The helper invokes `PKG_SERV_PROCESS_DATA` to stage and produce the B2C QR code record

**Technical evidence:**
- Condition: `cur_r.cust_gst_num = 'GSTUNREGISTERED'`
- Helper called: `PKG_ADDON_SALE.VAL_INS_INVOICES_B2C_CCP` (p_inv_type `'ECS'`, p_doc_type `'ECS'`, p_type `'S'`)
- Downstream: `PKG_SERV_PROCESS_DATA` → `SP_B2C_QR_GENERATOR` / `PKG_EINV`

---

#### Requirement ID: BR-007

**Requirement Title:** Failure visibility via operational log and per-order rollback

**Business Description:**
Any order that cannot be processed — due to sequence generation failure or insert error — shall be rolled back individually without affecting other orders in the same run. The operational log at `/home/dmsdb/logs/GEN_SAC_INVOICE.log` records all SQL\*Plus output, providing the primary visibility channel for operations in the absence of a dedicated alerting mechanism coded in this script.

**Actors:** N/A — no named business role in source

**Preconditions:** N/A

**Main Flow:**
1. On sequence generation failure (`ln_err_cd <> 0`), skip the insert and issue a `ROLLBACK` for that order
2. On insert exception, set `ln_err_cd := 1` and issue a `ROLLBACK`
3. On success, issue a `COMMIT` per order
4. All output is captured in the spool log at `/home/dmsdb/logs/GEN_SAC_INVOICE.log`

**Technical evidence:**
- Per-order `COMMIT` / `ROLLBACK` pattern in `PKG_ADDON_SALE.SP_GEN_SAC_INVOICE`
- No dedicated alerting or notification procedure coded in the shell

## 6. Business Rules Catalogue

*One business rule sentence per row. Mechanism detail, DECODE logic, and predicate specifics in the Source column. Rules derived exclusively from the entry-path procedure body (`PKG_ADDON_SALE.SP_GEN_SAC_INVOICE`) and its on-chain helpers.*

| Rule ID | Description | Source | Related BR |
| --- | --- | --- | --- |
| BRC-001 | Only SAC orders that are not yet invoiced, not cancelled, and placed within the last calendar day are eligible for invoice generation in a given run. | `PKG_ADDON_SALE.SP_GEN_SAC_INVOICE` — cursor filter on `vt_sac_order`: `txn_status = 'N'` (or NULL), `cancel_yn = 'N'`, `order_date >= TRUNC(SYSDATE) - 1`; `PKG_ADDON_SALE.pck` line 9099–9101 | BR-002 |
| BRC-002 | An SAC order can only be invoiced when the corresponding service job card is fully completed and has SAC CIC integration enabled. | `PKG_ADDON_SALE.SP_GEN_SAC_INVOICE` — check on `vh_ro`: `sac_cic_int_yn = 'Y'` and `ro_status = '40'`; `PKG_ADDON_SALE.pck` line 9134–9135 | BR-002 |
| BRC-003 | Each successfully processed SAC order must be individually committed; any order that fails sequence generation or invoice insert must be rolled back without affecting other orders in the same batch run. | `PKG_ADDON_SALE.SP_GEN_SAC_INVOICE` — per-order `COMMIT` / `ROLLBACK` pattern; `ln_err_cd` checked after `SP_GEN_SEQ_NUM` and after the `vt_addon` INSERT; `PKG_ADDON_SALE.pck` lines 9433–9460 | BR-002, BR-007 |
| BRC-004 | An SMS notification may only be sent to a customer mobile number that is non-null and passes mobile number validation. | `PKG_ADDON_SALE.SP_GEN_SAC_INVOICE` — condition `cur_r.mobile IS NOT NULL AND FNC_VERIF_MOBILE(cur_r.mobile) != 0` before calling `SP_PUSH_CCPSMS`; `PKG_ADDON_SALE.pck` line 9374 | BR-005 |
| BRC-005 | Loyalty award points may only be recorded and notified when the customer's VIN is linked to a loyalty card and award points were successfully calculated for the transaction amount. | `PKG_ADDON_SALE.SP_GEN_SAC_INVOICE` — conditions `lv_loy_card IS NOT NULL AND lv_ccp_num IS NOT NULL` and `ln_loy_awd_pts IS NOT NULL` before calling `SP_INS_LOYALTY_TRANSACTIONS` and `SP_SEND_SMS`; `PKG_ADDON_SALE.pck` lines 9398–9401 | BR-005 |
| BRC-006 | When the customer is unregistered for GST, the invoice must be processed through the B2C e-invoice QR code generation path rather than the standard B2B path. | `PKG_ADDON_SALE.SP_GEN_SAC_INVOICE` — condition `cur_r.cust_gst_num = 'GSTUNREGISTERED'` triggers call to `VAL_INS_INVOICES_B2C_CCP` with `p_inv_type = 'ECS'`, `p_doc_type = 'ECS'`; `PKG_ADDON_SALE.pck` line 9463 | BR-006 |
| BRC-007 | The service type flag stored on the invoice record must reflect whether the associated service was a periodic maintenance service or a customer pay service. | `PKG_ADDON_SALE.SP_GEN_SAC_INVOICE` — `IF cur_r.srv_type = 'PMS' THEN lv_srv_flag := 'S'; ELSE lv_srv_flag := 'C'; END IF;`; flag written to `vt_addon.with_srv_flag`; `PKG_ADDON_SALE.pck` line 9191 | BR-002 |

## 7. Data Requirements

### 7.1 Business entities

The following logical business entities are touched by this job on its entry path. Each is shown with the primary table(s) that holds it.

| Business Entity | Primary Table(s) |
| --- | --- |
| SAC order (pending add-on contract order awaiting invoice) | `vt_sac_order` |
| Add-on / SAC invoice (generated certificate and invoice) | `vt_addon` |
| Service job card (vehicle repair order) | `vh_ro` |
| Dealer / location master | `am_dealer_loc`, `am_dealer_loc_param`, `am_dealer_loc_company_master` |
| Principal / organisation | `am_organisation`, `am_zone` |
| Vehicle master | `gm_vin`, `gm_vin_ext` |
| Customer / party | `gm_party` |
| City / state / geography | `gm_city`, `gm_state`, `gm_district`, `gm_tehsil`, `gm_village` |
| Loyalty enrolment | `gd_loyalty_enrol`, `gd_loyalty_card_vin_map` |
| Loyalty transaction | `gd_loyalty_trans` |
| GST / tax configuration | `gst_charge`, `gst_cess_rate` |
| SMS status / notification log | `gm_sms_status`, `gm_email_status` |
| e-Invoice staging | `gbl_einv_stg_tab`, `gd_einv_temp`, `sh_einvoice`, `sh_einvoice_dtl` |
| Document / sequence reference | `document_no`, `docdtls_no` |

### 7.2 Data read / data written

| Object | Type | Access (scan) | Notes |
| --- | --- | --- | --- |
| AM_CHARGE | TABLE | READ | Entry-path static SQL scan (this job) |
| AM_DEALER_LOC | TABLE | READ | Entry-path static SQL scan (this job) |
| AM_DEALER_LOC_COMPANY_MASTER | TABLE | READ | Entry-path static SQL scan (this job) |
| AM_DEALER_LOC_PARAM | TABLE | READ/WRITE | Entry-path static SQL scan (this job) |
| AM_LIST | TABLE | READ | Entry-path static SQL scan (this job) |
| AM_LIST_RANGE | TABLE | READ | Entry-path static SQL scan (this job) |
| AM_MUL_DOCS | TABLE | READ/WRITE | Entry-path static SQL scan (this job) |
| AM_ORGANISATION | TABLE | READ | Entry-path static SQL scan (this job) |
| AM_RECPT_BANK | TABLE | READ | Entry-path static SQL scan (this job) |
| AM_ZONE | TABLE | READ | Entry-path static SQL scan (this job) |
| BUSINESS_TEST_TABLE_TEMP | TABLE | WRITE | Entry-path static SQL scan (this job) |
| DMSD_DIRF | TABLE | READ | Entry-path static SQL scan (this job) |
| DMSD_EW_SALE | TABLE | READ | Entry-path static SQL scan (this job) |
| DMSD_EW_SALE_SM | TABLE | READ | Entry-path static SQL scan (this job) |
| DOCDTLS_NO | TABLE | READ | Entry-path static SQL scan (this job) |
| DOCUMENT_NO | TABLE | READ | Entry-path static SQL scan (this job) |
| GBL_EINV_STG_TAB | TABLE | READ/WRITE | Entry-path static SQL scan (this job) |
| GBL_SERV_INV_DETL | TABLE | READ/WRITE | Entry-path static SQL scan (this job) |
| GBL_SERV_STG_TAB | TABLE | READ/WRITE | Entry-path static SQL scan (this job) |
| GD_EINV_TEMP | TABLE | READ/WRITE | Entry-path static SQL scan (this job) |
| GD_FDI_TRANS | TABLE | READ/WRITE | Entry-path static SQL scan (this job) |
| GD_FDI_TRANS_CHARGES | TABLE | READ/WRITE | Entry-path static SQL scan (this job) |
| GD_FDI_TRANS_CUSTOMER | TABLE | READ/WRITE | Entry-path static SQL scan (this job) |
| GD_FDI_USER_DEALERS | TABLE | READ | Entry-path static SQL scan (this job) |
| GD_LOYALTY_CARD_VIN_MAP | TABLE | READ/WRITE | Entry-path static SQL scan (this job) |
| GD_LOYALTY_ENROL | TABLE | READ/WRITE | Entry-path static SQL scan (this job) |
| GD_LOYALTY_PRE_ENROLL | TABLE | WRITE | Entry-path static SQL scan (this job) |
| GD_LOYALTY_TRANS | TABLE | READ/WRITE | Entry-path static SQL scan (this job) |
| GD_LOY_BADGE_UTL | TABLE | READ/WRITE | Entry-path static SQL scan (this job) |
| GD_LOY_OTP_LOG | TABLE | READ/WRITE | Entry-path static SQL scan (this job) |
| GD_LOY_RECON | TABLE | READ/WRITE | Entry-path static SQL scan (this job) |
| GLB_LOY_DETAILS | TABLE | READ/WRITE | Entry-path static SQL scan (this job) |
| GLB_TEMP_GD_LOY_PRINT | TABLE | READ/WRITE | Entry-path static SQL scan (this job) |
| GLB_TEMP_GD_LOY_PRINT1 | TABLE | READ/WRITE | Entry-path static SQL scan (this job) |
| GLB_TEMP_LOY_INV | TABLE | READ/WRITE | Entry-path static SQL scan (this job) |
| GLB_TEMP_LOY_POCORD | TABLE | READ/WRITE | Entry-path static SQL scan (this job) |
| GM_CIN | TABLE | READ | Entry-path static SQL scan (this job) |
| GM_CITY | TABLE | READ | Entry-path static SQL scan (this job) |
| GM_DISTRICT | TABLE | READ | Entry-path static SQL scan (this job) |
| GM_DMS_DATE_CODES | TABLE | READ | Entry-path static SQL scan (this job) |
| GM_DMS_STATS_CONFIG | TABLE | READ | Entry-path static SQL scan (this job) |
| GM_ECLR | TABLE | READ | Entry-path static SQL scan (this job) |
| GM_EMAIL_STATUS | TABLE | WRITE | Entry-path static SQL scan (this job) |
| GM_EMP | TABLE | READ | Entry-path static SQL scan (this job) |
| GM_FDI_USERS | TABLE | READ | Entry-path static SQL scan (this job) |
| GM_FOR | TABLE | READ | Entry-path static SQL scan (this job) |
| GM_LOYALTY_MASTER | TABLE | READ | Entry-path static SQL scan (this job) |
| GM_LOY_BADGE_CONF | TABLE | READ | Entry-path static SQL scan (this job) |
| GM_LOY_TIER_MASTER | TABLE | READ | Entry-path static SQL scan (this job) |
| GM_LOY_TRANS_TYPES | TABLE | READ | Entry-path static SQL scan (this job) |
| GM_MOD | TABLE | READ | Entry-path static SQL scan (this job) |
| GM_PARTY | TABLE | READ | Entry-path static SQL scan (this job) |
| GM_PROCESS_MASTER | TABLE | READ | Entry-path static SQL scan (this job) |
| GM_SMS_STATUS | TABLE | READ/WRITE | Entry-path static SQL scan (this job) |
| GM_STATE | TABLE | READ | Entry-path static SQL scan (this job) |
| GM_TEHSIL | TABLE | READ | Entry-path static SQL scan (this job) |
| GM_VAR | TABLE | READ | Entry-path static SQL scan (this job) |
| GM_VILLAGE | TABLE | READ | Entry-path static SQL scan (this job) |
| GM_VIN | TABLE | READ | Entry-path static SQL scan (this job) |
| GM_VIN_EXT | TABLE | READ | Entry-path static SQL scan (this job) |
| GST_CESS_RATE | TABLE | READ | Entry-path static SQL scan (this job) |
| GST_CHARGE | TABLE | READ | Entry-path static SQL scan (this job) |
| MARUTI | TABLE | READ | Entry-path static SQL scan (this job) |
| MWAR_EXTE | TABLE | READ/WRITE | Entry-path static SQL scan (this job) |
| NIC | TABLE | READ | Entry-path static SQL scan (this job) |
| NI_NEW_MINS | TABLE | READ | Entry-path static SQL scan (this job) |
| NOWAIT | TABLE | WRITE | Entry-path static SQL scan (this job) |
| PD_ISSUE | TABLE | READ | Entry-path static SQL scan (this job) |
| PD_ISSUE_CHARGES | TABLE | READ | Entry-path static SQL scan (this job) |
| PH_ENQ | TABLE | READ | Entry-path static SQL scan (this job) |
| PH_ISSUE | TABLE | READ/WRITE | Entry-path static SQL scan (this job) |
| PH_RECEIPTS | TABLE | READ | Entry-path static SQL scan (this job) |
| PH_RETURNS | TABLE | READ/WRITE | Entry-path static SQL scan (this job) |
| PM_PART | TABLE | READ | Entry-path static SQL scan (this job) |
| QR | TABLE | WRITE | Entry-path static SQL scan (this job) |
| RM_ETYPES | TABLE | READ | Entry-path static SQL scan (this job) |
| RSA_REG_DTL | TABLE | READ/WRITE | Entry-path static SQL scan (this job) |
| SD_APPORTION | TABLE | READ | Entry-path static SQL scan (this job) |
| SD_GRN | TABLE | READ | Entry-path static SQL scan (this job) |
| SD_GRN_CHARGES | TABLE | READ | Entry-path static SQL scan (this job) |
| SD_INV_CHARGES | TABLE | READ | Entry-path static SQL scan (this job) |
| SD_MGA_BOOKING | TABLE | READ | Entry-path static SQL scan (this job) |
| SD_POC_INV_CHARGES | TABLE | READ | Entry-path static SQL scan (this job) |
| SD_VOUCHER | TABLE | READ | Entry-path static SQL scan (this job) |
| SH_EINVOICE | TABLE | READ/WRITE | Entry-path static SQL scan (this job) |
| SH_EINVOICE_DTL | TABLE | READ/WRITE | Entry-path static SQL scan (this job) |
| SH_GRN | TABLE | READ | Entry-path static SQL scan (this job) |
| SH_INVOICE | TABLE | READ/WRITE | Entry-path static SQL scan (this job) |
| SH_INV_SCHEME | TABLE | READ | Entry-path static SQL scan (this job) |
| SH_ORDBOOK | TABLE | READ | Entry-path static SQL scan (this job) |
| SH_ORD_CHARGES | TABLE | READ | Entry-path static SQL scan (this job) |
| SH_POC_INVOICE | TABLE | READ/WRITE | Entry-path static SQL scan (this job) |
| SH_POC_ORDBOOK | TABLE | READ | Entry-path static SQL scan (this job) |
| SH_RECEIPTS | TABLE | READ | Entry-path static SQL scan (this job) |
| SH_TV_EVAL | TABLE | READ | Entry-path static SQL scan (this job) |
| SH_VOUCHER | TABLE | READ/WRITE | Entry-path static SQL scan (this job) |
| SI_INCD | TABLE | READ | Entry-path static SQL scan (this job) |
| SI_INVD | TABLE | READ | Entry-path static SQL scan (this job) |
| SI_INVH | TABLE | READ | Entry-path static SQL scan (this job) |
| SM_DMSPOC_MAKE | TABLE | READ | Entry-path static SQL scan (this job) |
| SM_SPRICE_FOR | TABLE | READ | Entry-path static SQL scan (this job) |
| SM_TEAM | TABLE | READ | Entry-path static SQL scan (this job) |
| ST_APPORTION | TABLE | READ | Entry-path static SQL scan (this job) |
| ST_STKTR_CHARGES | TABLE | READ | Entry-path static SQL scan (this job) |
| ST_STKTR_ISS | TABLE | READ/WRITE | Entry-path static SQL scan (this job) |
| VD_AMC_ENROL | TABLE | READ | Entry-path static SQL scan (this job) |
| VD_AMC_ENROLCHARGES | TABLE | READ | Entry-path static SQL scan (this job) |
| VD_CCP_COMM_DTL | TABLE | READ/WRITE | Entry-path static SQL scan (this job) |
| VD_EW_COMM_DTL | TABLE | READ/WRITE | Entry-path static SQL scan (this job) |
| VD_RO_OPR | TABLE | READ | Entry-path static SQL scan (this job) |
| VD_RO_OPRCHARGES | TABLE | READ | Entry-path static SQL scan (this job) |
| VH_AMC_ENROL | TABLE | READ/WRITE | Entry-path static SQL scan (this job) |
| VH_AMC_SM | TABLE | READ/WRITE | Entry-path static SQL scan (this job) |
| VH_PSF | TABLE | READ | Entry-path static SQL scan (this job) |
| VH_RO | TABLE | READ | Entry-path static SQL scan (this job) |
| VH_SMP_DEBITINV | TABLE | WRITE | Entry-path static SQL scan (this job) |
| VH_TCU_ENROL | TABLE | WRITE | Entry-path static SQL scan (this job) |
| VH_WC | TABLE | READ | Entry-path static SQL scan (this job) |
| VIEW | TABLE | READ | Entry-path static SQL scan (this job) |
| VM_CLAIMLOAD | TABLE | READ | Entry-path static SQL scan (this job) |
| VM_EMAIL_DTL | TABLE | READ | Entry-path static SQL scan (this job) |
| VM_OPR | TABLE | READ | Entry-path static SQL scan (this job) |
| VM_SVAR | TABLE | READ | Entry-path static SQL scan (this job) |
| VT_ADDON | TABLE | READ/WRITE | Entry-path static SQL scan (this job) |
| VT_CRI_INVDTL | TABLE | READ/WRITE | Entry-path static SQL scan (this job) |
| VT_EXTSAWA | TABLE | READ/WRITE | Entry-path static SQL scan (this job) |
| VT_FSC_LOT | TABLE | READ/WRITE | Entry-path static SQL scan (this job) |
| VT_FSC_LOT_CHARGES | TABLE | READ | Entry-path static SQL scan (this job) |
| VT_FTIR_NEW | TABLE | READ | Entry-path static SQL scan (this job) |
| VT_IQIC_INVDTL | TABLE | READ/WRITE | Entry-path static SQL scan (this job) |
| VT_MASS_BUSS_ZONE | TABLE | WRITE | Entry-path static SQL scan (this job) |
| VT_MPSI_INVDTL | TABLE | READ/WRITE | Entry-path static SQL scan (this job) |
| VT_SAC_ORDER | TABLE | READ/WRITE | Entry-path static SQL scan (this job) |
| VT_SRS | TABLE | READ/WRITE | Entry-path static SQL scan (this job) |
| VT_STR | TABLE | READ/WRITE | Entry-path static SQL scan (this job) |
| VT_TCU_COMM_DTL | TABLE | READ/WRITE | Entry-path static SQL scan (this job) |

Labelled **as-is from code** on the entry-path walk; not a full ERD. Static scan — not live `ALL_DEPENDENCIES`. `LV_*` scan names omitted here (see §10).


### 7.2.1 Package-file objects not on this job's entry path

The following object name(s) appear in the same `.pck` file (or ZIP package inventory) but were **not** evidenced on the shell entry routine body: `DATE`, `DMSD_EW_FIN`, `GBL_ADDON`, `GBL_CCP_PREM`, `MOBILE`, `ST_ALLOT`, `VD_RO_SAC_CHECKSHEET`, `VD_RO_VHC_DTL`, `VD_RO_VHC_SUMM`, `VM_ADDON_PREM`, `VM_CCPPREM_UPLOAD`, `VM_CCP_DISC`…. Treated as sibling/scan inventory — not requirements for this job unless SME confirms.

### 7.3 Reference / configuration data

The following configuration and master-data tables are read by the entry path to support lookups during invoice generation:

| Table | Role |
| --- | --- |
| `am_list` / `am_list_range` | Code-list lookups used within the on-chain packages |
| `gm_loyalty_master` | Loyalty programme configuration (tier/point rules) |
| `gm_loy_tier_master` | Loyalty tier definitions used by `SF_CALCULATE_RUPEES_TO_POINTS` |
| `gm_loy_trans_types` | Loyalty transaction type codes |
| `gm_loy_badge_conf` | Loyalty badge configuration |
| `gst_charge` / `gst_cess_rate` | GST rate and cess rate configuration for invoice tax computation |
| `gm_process_master` | Process control flags referenced by on-chain helpers |
| `gm_dms_stats_config` | DMS statistics configuration |
| `gm_dms_date_codes` | Date-code lookups |

### 7.4 Data lifecycle

- **Source / input:** Pending SAC orders are read from `vt_sac_order` (written upstream by the order-creation flow; filtered to yesterday and today).
- **Processing output:** Invoice records are inserted into `vt_addon`; processed orders are updated (`txn_status = 'S'`) in `vt_sac_order`. Loyalty transactions are inserted into `gd_loyalty_trans` (via `SP_INS_LOYALTY_TRANSACTIONS`). For B2C GST customers, e-invoice staging data is written to `gbl_einv_stg_tab` / `gd_einv_temp` (via `VAL_INS_INVOICES_B2C_CCP` → `PKG_SERV_PROCESS_DATA` → `PKG_EINV`).
- **Retention / archival:** N/A — no retention or archival logic coded in this entry path; downstream table lifecycle is governed by the DMS schema owners.

### 7.5 Volumes and timing (optional)

N/A — no client-supplied volume estimates in source.

## 8. Glossary

### 8.1 Business terms

| Term | Definition | Aliases | Related |
| --- | --- | --- | --- |
| SAC | Service Add-on Contract — a dealer-sold service product attached to a vehicle repair order that provides the customer with defined add-on service benefits; the subject of this job's invoice generation. | Service Add-on | `VT_SAC_ORDER`, `VT_ADDON`, `FN_GET_LOY_SAC_CODE` |
| CCP | Comprehensive Care Plan — a vehicle service plan product sold at the dealership; this job generates B2C e-invoices for CCP transactions and dispatches post-sale SMS notifications. | Comprehensive Care Plan | `VT_CCP_ORDER`, `VAL_INS_INVOICES_B2C_CCP`, `SP_SEND_SMS_SRV_CCP_SALE` |
| E-Invoice | A GST-compliant electronic invoice record generated by the system (via `PKG_EINV`) and stored in `SH_EINVOICE` / `SH_EINVOICE_DTL`; includes a QR code for verification. | Electronic Invoice | `PKG_EINV`, `PKG_SERV_PROCESS_DATA`, `SH_EINVOICE` |
| FDI | Franchisee Data Integration — an integration channel through which dealer transaction data (add-on sales, extended warranty, etc.) is pushed to a central platform; this job stages FDI payloads via `GD_FDI_TRANS` and related tables. | Franchisee Data Integration | `PKG_FDI_SERV`, `PKG_FDI`, `GD_FDI_TRANS` |
| VIN | Vehicle Identification Number — the unique identifier for a vehicle, used throughout the job to look up repair-order, loyalty, and invoice data. | N/A | `GM_VIN`, `VT_EW_ONLINE_VIN`, `VT_ONLINE_VIN` |
| Loyalty | Customer loyalty programme under which points are accrued or redeemed against qualifying service events; managed by `PKG_LOYALTY` and recorded in `GD_LOYALTY_TRANS`. | Loyalty Programme | `PKG_LOYALTY`, `GD_LOYALTY_ENROL`, `GD_LOYALTY_TRANS` |

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
| PKG_ADDON_SALE.SP_GEN_SAC_INVOICE | PKG_ADDON_SALE.SP_GEN_SAC_INVOICE(1) | Entry call |

## 9. Technical Inventory

### 9.1 Call chain

| Level | What | Name | Meaning | Source file |
| --- | --- | --- | --- | --- |
| 0 | SCAN L1 | GEN_SAC_INVOICE.sh | Start — main shell script | GEN_SAC_INVOICE.sh |
| 1 | FOUND L1 | PKG_ADDON_SALE.SP_GEN_SAC_INVOICE | Named in the shell EXEC/CALL |  |
| 1 | FOUND L1 | PKG_ADDON_SALE | Package source in ZIP | PKG_ADDON_SALE.pck |
| 2 | CASCADE L2 | PKG_COMMUNICATION | Referenced from entry path of PKG_ADDON_SALE |  |
| 2 | CASCADE L2 | PKG_LOYALTY | Referenced from entry path of PKG_ADDON_SALE |  |
| 2 | CASCADE L2 | PKG_SERV_PROCESS_DATA | Referenced from entry path of PKG_ADDON_SALE |  |
| 2 | FOUND L2 | PKG_COMMUNICATION | Package source in ZIP | PKG_COMMUNICATION.pck |
| 2 | FOUND L2 | PKG_LOYALTY | Package source in ZIP | PKG_LOYALTY.pck |
| 3 | CASCADE L3 | PKG_SFDC_SMS | Referenced from entry path of PKG_LOYALTY |  |
| 2 | FOUND L2 | PKG_SERV_PROCESS_DATA | Package source in ZIP | PKG_SERV_PROCESS_DATA.pck |
| 3 | CASCADE L3 | PKG_EINV | Referenced from entry path of PKG_SERV_PROCESS_DATA |  |
| 2 | FOUND L2 | FNC_VERIF_MOBILE | Standalone routine file in ZIP (called from entry path) | FNC_VERIF_MOBILE.fnc |
| 2 | FOUND L2 | SP_GEN_SEQ_NUM | Same-package helper defined in owning .pck (on entry path) | PKG_ADDON_SALE.pck |
| 2 | FOUND L2 | SP_PUSH_CCPSMS | Same-package helper defined in owning .pck (on entry path) | PKG_ADDON_SALE.pck |
| 3 | MISSING L3 | SF_GET_MODEL_CHANNEL | Called from entry path; source file not in ZIP |  |
| 3 | MISSING L3 | PKG_SFDC_SMS | Package file not in ZIP |  |
| 3 | MISSING L3 | SEND_SMS | Called from entry path; source file not in ZIP |  |
| 3 | FOUND L3 | SF_PRINCIPAL_MAP_CD | Standalone routine file in ZIP (called from entry path) | SF_PRINCIPAL_MAP_CD.fnc |
| 3 | FOUND L3 | PKG_EINV | Package source in ZIP | PKG_EINV.pck |
| 4 | CASCADE L4 | PKG_ENQ_POC_DETAIL | Referenced from entry path of PKG_EINV |  |
| 4 | CASCADE L4 | PKG_LOYALTY | Referenced from entry path of PKG_EINV |  |
| 4 | CASCADE L4 | PKG_SPR_EINV | Referenced from entry path of PKG_EINV |  |
| 2 | FOUND L2 | SF_CALCULATE_RUPEES_TO_POINTS | Same-package helper defined in owning .pck (on entry path) | PKG_ADDON_SALE.pck |
| 2 | FOUND L2 | SP_GET_VIN_LOYALTY_DTL | Same-package helper defined in owning .pck (on entry path) | PKG_ADDON_SALE.pck |
| 2 | FOUND L2 | SP_INS_LOYALTY_TRANSACTIONS | Same-package helper defined in owning .pck (on entry path) | PKG_ADDON_SALE.pck |
| 2 | FOUND L2 | SP_SEND_SMS | Same-package helper defined in owning .pck (on entry path) | PKG_ADDON_SALE.pck |
| 2 | FOUND L2 | SP_SEND_SMS_SRV_CCP_SALE | Same-package helper defined in owning .pck (on entry path) | PKG_ADDON_SALE.pck |
| 2 | FOUND L2 | VAL_INS_INVOICES_B2C_CCP | Same-package helper defined in owning .pck (on entry path) | PKG_ADDON_SALE.pck |
| 3 | FOUND L3 | PKG_FDI_SERV | Package source in ZIP | PKG_FDI_SERV.pck |
| 4 | CASCADE L4 | PKG_FDI | Referenced from entry path of PKG_FDI_SERV |  |
| 3 | FOUND L3 | FNC_VERIF_MOBILE | Standalone routine file in ZIP (called from entry path) | FNC_VERIF_MOBILE.fnc |
| 4 | MISSING L4 | PKG_ENQ_POC_DETAIL | Package file not in ZIP |  |
| 4 | FOUND L4 | PKG_LOYALTY | Package source in ZIP | PKG_LOYALTY.pck |
| 5 | CASCADE L5 | PKG_SFDC_SMS | Referenced from entry path of PKG_LOYALTY |  |
| 4 | MISSING L4 | PKG_SPR_EINV | Package file not in ZIP |  |
| 4 | MISSING L4 | SF_OLD_VAR_DESC | Called from entry path; source file not in ZIP |  |
| 3 | FOUND L3 | SP_B2C_QR_GENERATOR | Same-package helper defined in owning .pck (on entry path) | PKG_ADDON_SALE.pck |
| 3 | FOUND L3 | SP_QR_GENERATOR | Same-package helper defined in owning .pck (on entry path) | PKG_ADDON_SALE.pck |
| 4 | FOUND L4 | PKG_FDI | Package source in ZIP | PKG_FDI.pck |
| 4 | MISSING L4 | SP_PUSH_EWC_FDI_DATA | Called from entry path; source file not in ZIP |  |
| 4 | MISSING L4 | SP_PUSH_EWC_FDI_DATA_SM | Called from entry path; source file not in ZIP |  |
| 4 | MISSING L4 | SP_PUSH_EWU_FDI_DATA | Called from entry path; source file not in ZIP |  |
| 4 | MISSING L4 | SP_PUSH_EWU_FDI_DATA_SM | Called from entry path; source file not in ZIP |  |
| 4 | MISSING L4 | SP_PUSH_EW_FDI_DATA | Called from entry path; source file not in ZIP |  |
| 4 | MISSING L4 | SP_PUSH_EW_FDI_DATA_SM | Called from entry path; source file not in ZIP |  |
| 4 | MISSING L4 | SP_PUSH_MCPC_FDI_DATA | Called from entry path; source file not in ZIP |  |
| 4 | MISSING L4 | SP_PUSH_MCP_FDI_DATA | Called from entry path; source file not in ZIP |  |
| 3 | FOUND L3 | SP_PUSH_CCPC_FDI_DATA | Same-package helper defined in owning .pck (on entry path) | PKG_ADDON_SALE.pck |
| 5 | MISSING L5 | PKG_SFDC_SMS | Package file not in ZIP |  |
| 5 | MISSING L5 | SEND_SMS | Called from entry path; source file not in ZIP |  |
| 5 | FOUND L5 | SF_PRINCIPAL_MAP_CD | Standalone routine file in ZIP (called from entry path) | SF_PRINCIPAL_MAP_CD.fnc |
| 4 | FOUND L4 | FN_GET_LOY_SAC_CODE | Same-package helper defined in owning .pck (on entry path) | PKG_ADDON_SALE.pck |
| 4 | FOUND L4 | VAL_INS_INVOICES_B2C | Same-package helper defined in owning .pck (on entry path) | PKG_ADDON_SALE.pck |
| 5 | MISSING L5 | SF_GET_BONUS_AGE_SLAB | Called from entry path; source file not in ZIP |  |
| 5 | MISSING L5 | SF_GET_GRN_DISCOUNT | Called from entry path; source file not in ZIP |  |
| 5 | MISSING L5 | SF_GET_GST_CHARGES_PR | Called from entry path; source file not in ZIP |  |
| 5 | MISSING L5 | SF_GET_HSN | Called from entry path; source file not in ZIP |  |
| 5 | MISSING L5 | SF_GET_TOTAL_BONUS | Called from entry path; source file not in ZIP |  |
| 5 | MISSING L5 | SP_PUSH_COUNTERSALE_INVOICE | Called from entry path; source file not in ZIP |  |
| 5 | MISSING L5 | SP_PUSH_COUNTERSALE_RETURN | Called from entry path; source file not in ZIP |  |
| 5 | MISSING L5 | SP_PUSH_PART_RECIPT | Called from entry path; source file not in ZIP |  |
| 4 | FOUND L4 | SP_PUSH_FDI_TRANSACTIONS | Same-package helper defined in owning .pck (on entry path) | PKG_ADDON_SALE.pck |

### 9.2 Found source files (ZIP)

| File | Status |
| --- | --- |
| FNC_VERIF_MOBILE.fnc | In ZIP |
| GEN_SAC_INVOICE.sh | In ZIP |
| PKG_ADDON_SALE.pck | In ZIP |
| PKG_COMMUNICATION.pck | In ZIP |
| PKG_EINV.pck | In ZIP |
| PKG_FDI.pck | In ZIP |
| PKG_FDI_SERV.pck | In ZIP |
| PKG_LOYALTY.pck | In ZIP |
| PKG_SERV_PROCESS_DATA.pck | In ZIP |
| SF_PRINCIPAL_MAP_CD.fnc | In ZIP |

### 9.3 Missing / blocking

| Kind | Reference | Level |
| --- | --- | --- |
| routine | SF_GET_MODEL_CHANNEL | 3 |
| package | PKG_SFDC_SMS | 3 |
| routine | SEND_SMS | 3 |
| package | PKG_ENQ_POC_DETAIL | 4 |
| package | PKG_SPR_EINV | 4 |
| routine | SF_OLD_VAR_DESC | 4 |
| routine | SP_PUSH_EWC_FDI_DATA | 4 |
| routine | SP_PUSH_EWC_FDI_DATA_SM | 4 |
| routine | SP_PUSH_EWU_FDI_DATA | 4 |
| routine | SP_PUSH_EWU_FDI_DATA_SM | 4 |
| routine | SP_PUSH_EW_FDI_DATA | 4 |
| routine | SP_PUSH_EW_FDI_DATA_SM | 4 |
| routine | SP_PUSH_MCPC_FDI_DATA | 4 |
| routine | SP_PUSH_MCP_FDI_DATA | 4 |
| package | PKG_SFDC_SMS | 5 |
| routine | SEND_SMS | 5 |
| routine | SF_GET_BONUS_AGE_SLAB | 5 |
| routine | SF_GET_GRN_DISCOUNT | 5 |
| routine | SF_GET_GST_CHARGES_PR | 5 |
| routine | SF_GET_HSN | 5 |
| routine | SF_GET_TOTAL_BONUS | 5 |
| routine | SP_PUSH_COUNTERSALE_INVOICE | 5 |
| routine | SP_PUSH_COUNTERSALE_RETURN | 5 |
| routine | SP_PUSH_PART_RECIPT | 5 |

### 9.4 Explicitly not in this inventory

- Runtime executables such as `sqlplus` (listed informationally under executables)
- Environment variables as files
- Database tables as deliverable files
- Procedures in a `.pck` that are **not** on this call chain

### 9.5 Excel / JSON pointer

Companion artefacts for this run: `GEN_SAC_INVOICE_TraceabilityReport.md` (traceability) and `_cron_metrics.json` (deterministic counts).

## 10. Referenced Database Objects

### 10.1 Tables / views

| Name | Type | Access (scan) | One-line purpose |
| --- | --- | --- | --- |
| AM_CHARGE | TABLE | READ | N/A |
| AM_DEALER_LOC | TABLE | READ | N/A |
| AM_DEALER_LOC_COMPANY_MASTER | TABLE | READ | N/A |
| AM_DEALER_LOC_PARAM | TABLE | READ/WRITE | N/A |
| AM_LIST | TABLE | READ | N/A |
| AM_LIST_RANGE | TABLE | READ | N/A |
| AM_MUL_DOCS | TABLE | READ/WRITE | N/A |
| AM_ORGANISATION | TABLE | READ | N/A |
| AM_RECPT_BANK | TABLE | READ | N/A |
| AM_ZONE | TABLE | READ | N/A |
| BUSINESS_TEST_TABLE_TEMP | TABLE | WRITE | N/A |
| DMSD_DIRF | TABLE | READ | N/A |
| DMSD_EW_SALE | TABLE | READ | N/A |
| DMSD_EW_SALE_SM | TABLE | READ | N/A |
| DOCDTLS_NO | TABLE | READ | N/A |
| DOCUMENT_NO | TABLE | READ | N/A |
| GBL_EINV_STG_TAB | TABLE | READ/WRITE | N/A |
| GBL_SERV_INV_DETL | TABLE | READ/WRITE | N/A |
| GBL_SERV_STG_TAB | TABLE | READ/WRITE | N/A |
| GD_EINV_TEMP | TABLE | READ/WRITE | N/A |
| GD_FDI_TRANS | TABLE | READ/WRITE | N/A |
| GD_FDI_TRANS_CHARGES | TABLE | READ/WRITE | N/A |
| GD_FDI_TRANS_CUSTOMER | TABLE | READ/WRITE | N/A |
| GD_FDI_USER_DEALERS | TABLE | READ | N/A |
| GD_LOYALTY_CARD_VIN_MAP | TABLE | READ/WRITE | N/A |
| GD_LOYALTY_ENROL | TABLE | READ/WRITE | N/A |
| GD_LOYALTY_PRE_ENROLL | TABLE | WRITE | N/A |
| GD_LOYALTY_TRANS | TABLE | READ/WRITE | N/A |
| GD_LOY_BADGE_UTL | TABLE | READ/WRITE | N/A |
| GD_LOY_OTP_LOG | TABLE | READ/WRITE | N/A |
| GD_LOY_RECON | TABLE | READ/WRITE | N/A |
| GLB_LOY_DETAILS | TABLE | READ/WRITE | N/A |
| GLB_TEMP_GD_LOY_PRINT | TABLE | READ/WRITE | N/A |
| GLB_TEMP_GD_LOY_PRINT1 | TABLE | READ/WRITE | N/A |
| GLB_TEMP_LOY_INV | TABLE | READ/WRITE | N/A |
| GLB_TEMP_LOY_POCORD | TABLE | READ/WRITE | N/A |
| GM_CIN | TABLE | READ | N/A |
| GM_CITY | TABLE | READ | N/A |
| GM_DISTRICT | TABLE | READ | N/A |
| GM_DMS_DATE_CODES | TABLE | READ | N/A |
| GM_DMS_STATS_CONFIG | TABLE | READ | N/A |
| GM_ECLR | TABLE | READ | N/A |
| GM_EMAIL_STATUS | TABLE | WRITE | N/A |
| GM_EMP | TABLE | READ | N/A |
| GM_FDI_USERS | TABLE | READ | N/A |
| GM_FOR | TABLE | READ | N/A |
| GM_LOYALTY_MASTER | TABLE | READ | N/A |
| GM_LOY_BADGE_CONF | TABLE | READ | N/A |
| GM_LOY_TIER_MASTER | TABLE | READ | N/A |
| GM_LOY_TRANS_TYPES | TABLE | READ | N/A |
| GM_MOD | TABLE | READ | N/A |
| GM_PARTY | TABLE | READ | N/A |
| GM_PROCESS_MASTER | TABLE | READ | N/A |
| GM_SMS_STATUS | TABLE | READ/WRITE | N/A |
| GM_STATE | TABLE | READ | N/A |
| GM_TEHSIL | TABLE | READ | N/A |
| GM_VAR | TABLE | READ | N/A |
| GM_VILLAGE | TABLE | READ | N/A |
| GM_VIN | TABLE | READ | N/A |
| GM_VIN_EXT | TABLE | READ | N/A |
| GST_CESS_RATE | TABLE | READ | N/A |
| GST_CHARGE | TABLE | READ | N/A |
| MARUTI | TABLE | READ | N/A |
| MWAR_EXTE | TABLE | READ/WRITE | N/A |
| NIC | TABLE | READ | N/A |
| NI_NEW_MINS | TABLE | READ | N/A |
| NOWAIT | TABLE | WRITE | N/A |
| PD_ISSUE | TABLE | READ | N/A |
| PD_ISSUE_CHARGES | TABLE | READ | N/A |
| PH_ENQ | TABLE | READ | N/A |
| PH_ISSUE | TABLE | READ/WRITE | N/A |
| PH_RECEIPTS | TABLE | READ | N/A |
| PH_RETURNS | TABLE | READ/WRITE | N/A |
| PM_PART | TABLE | READ | N/A |
| QR | TABLE | WRITE | N/A |
| RM_ETYPES | TABLE | READ | N/A |
| RSA_REG_DTL | TABLE | READ/WRITE | N/A |
| SD_APPORTION | TABLE | READ | N/A |
| SD_GRN | TABLE | READ | N/A |
| SD_GRN_CHARGES | TABLE | READ | N/A |
| SD_INV_CHARGES | TABLE | READ | N/A |
| SD_MGA_BOOKING | TABLE | READ | N/A |
| SD_POC_INV_CHARGES | TABLE | READ | N/A |
| SD_VOUCHER | TABLE | READ | N/A |
| SH_EINVOICE | TABLE | READ/WRITE | N/A |
| SH_EINVOICE_DTL | TABLE | READ/WRITE | N/A |
| SH_GRN | TABLE | READ | N/A |
| SH_INVOICE | TABLE | READ/WRITE | N/A |
| SH_INV_SCHEME | TABLE | READ | N/A |
| SH_ORDBOOK | TABLE | READ | N/A |
| SH_ORD_CHARGES | TABLE | READ | N/A |
| SH_POC_INVOICE | TABLE | READ/WRITE | N/A |
| SH_POC_ORDBOOK | TABLE | READ | N/A |
| SH_RECEIPTS | TABLE | READ | N/A |
| SH_TV_EVAL | TABLE | READ | N/A |
| SH_VOUCHER | TABLE | READ/WRITE | N/A |
| SI_INCD | TABLE | READ | N/A |
| SI_INVD | TABLE | READ | N/A |
| SI_INVH | TABLE | READ | N/A |
| SM_DMSPOC_MAKE | TABLE | READ | N/A |
| SM_SPRICE_FOR | TABLE | READ | N/A |
| SM_TEAM | TABLE | READ | N/A |
| ST_APPORTION | TABLE | READ | N/A |
| ST_STKTR_CHARGES | TABLE | READ | N/A |
| ST_STKTR_ISS | TABLE | READ/WRITE | N/A |
| VD_AMC_ENROL | TABLE | READ | N/A |
| VD_AMC_ENROLCHARGES | TABLE | READ | N/A |
| VD_CCP_COMM_DTL | TABLE | READ/WRITE | N/A |
| VD_EW_COMM_DTL | TABLE | READ/WRITE | N/A |
| VD_RO_OPR | TABLE | READ | N/A |
| VD_RO_OPRCHARGES | TABLE | READ | N/A |
| VH_AMC_ENROL | TABLE | READ/WRITE | N/A |
| VH_AMC_SM | TABLE | READ/WRITE | N/A |
| VH_PSF | TABLE | READ | N/A |
| VH_RO | TABLE | READ | N/A |
| VH_SMP_DEBITINV | TABLE | WRITE | N/A |
| VH_TCU_ENROL | TABLE | WRITE | N/A |
| VH_WC | TABLE | READ | N/A |
| VIEW | TABLE | READ | N/A |
| VM_CLAIMLOAD | TABLE | READ | N/A |
| VM_EMAIL_DTL | TABLE | READ | N/A |
| VM_OPR | TABLE | READ | N/A |
| VM_SVAR | TABLE | READ | N/A |
| VT_ADDON | TABLE | READ/WRITE | N/A |
| VT_CRI_INVDTL | TABLE | READ/WRITE | N/A |
| VT_EXTSAWA | TABLE | READ/WRITE | N/A |
| VT_FSC_LOT | TABLE | READ/WRITE | N/A |
| VT_FSC_LOT_CHARGES | TABLE | READ | N/A |
| VT_FTIR_NEW | TABLE | READ | N/A |
| VT_IQIC_INVDTL | TABLE | READ/WRITE | N/A |
| VT_MASS_BUSS_ZONE | TABLE | WRITE | N/A |
| VT_MPSI_INVDTL | TABLE | READ/WRITE | N/A |
| VT_SAC_ORDER | TABLE | READ/WRITE | N/A |
| VT_SRS | TABLE | READ/WRITE | N/A |
| VT_STR | TABLE | READ/WRITE | N/A |
| VT_TCU_COMM_DTL | TABLE | READ/WRITE | N/A |


### 10.1.2 Package sibling objects (not on entry path)

`DATE`, `DMSD_EW_FIN`, `GBL_ADDON`, `GBL_CCP_PREM`, `MOBILE`, `ST_ALLOT`, `VD_RO_SAC_CHECKSHEET`, `VD_RO_VHC_DTL`, `VD_RO_VHC_SUMM`, `VM_ADDON_PREM`, `VM_CCPPREM_UPLOAD`, `VM_CCP_DISC`, `VM_CCP_RATE_MASTER`, `VM_EW_PARAM`, `VM_RESTRICTED_VIN` (+3 more). Present in package-file scan but not verified in the shell entry routine body — see also §7.2.1.

### 10.2 Other DB objects

Sequences / synonyms — only if seen in walked code (included above when classified).

**Note:** Static scan of supplied sources on the entry path where available, **not** a live `ALL_DEPENDENCIES` dump. Purpose column is `N/A` unless SME fills later.

## 11. Backend Packages and Procedures

### 11.1 Main flow index

shell → GEN_SAC_INVOICE.sh → L1: PKG_ADDON_SALE (PKG_ADDON_SALE.pck) → L2: FNC_VERIF_MOBILE (FNC_VERIF_MOBILE.fnc) → L2: PKG_COMMUNICATION (PKG_COMMUNICATION.pck) → L2: PKG_LOYALTY (PKG_LOYALTY.pck) → L2: PKG_SERV_PROCESS_DATA (PKG_SERV_PROCESS_DATA.pck) → L2: SF_CALCULATE_RUPEES_TO_POINTS (PKG_ADDON_SALE.pck) → L2: SP_GEN_SEQ_NUM (PKG_ADDON_SALE.pck) → L2: SP_GET_VIN_LOYALTY_DTL (PKG_ADDON_SALE.pck) → L2: SP_INS_LOYALTY_TRANSACTIONS (PKG_ADDON_SALE.pck) → L2: SP_PUSH_CCPSMS (PKG_ADDON_SALE.pck) → L2: SP_SEND_SMS (PKG_ADDON_SALE.pck) → L2: SP_SEND_SMS_SRV_CCP_SALE (PKG_ADDON_SALE.pck) → L2: VAL_INS_INVOICES_B2C_CCP (PKG_ADDON_SALE.pck) → L3: FNC_VERIF_MOBILE (FNC_VERIF_MOBILE.fnc) → L3: PKG_EINV (PKG_EINV.pck) → L3: PKG_FDI_SERV (PKG_FDI_SERV.pck) → L3: SF_PRINCIPAL_MAP_CD (SF_PRINCIPAL_MAP_CD.fnc) → L3: SP_B2C_QR_GENERATOR (PKG_ADDON_SALE.pck) → L3: SP_PUSH_CCPC_FDI_DATA (PKG_ADDON_SALE.pck) → L3: SP_QR_GENERATOR (PKG_ADDON_SALE.pck) → L4: FN_GET_LOY_SAC_CODE (PKG_ADDON_SALE.pck) → L4: PKG_FDI (PKG_FDI.pck) → L4: PKG_LOYALTY (PKG_LOYALTY.pck) → L4: SP_PUSH_FDI_TRANSACTIONS (PKG_ADDON_SALE.pck) → L4: VAL_INS_INVOICES_B2C (PKG_ADDON_SALE.pck) → L5: SF_PRINCIPAL_MAP_CD (SF_PRINCIPAL_MAP_CD.fnc)

### 11.2 Catalogue

| Name | File | Type | Parent (found_from) | Role |
| --- | --- | --- | --- | --- |
| PKG_ADDON_SALE | PKG_ADDON_SALE.pck | package | GEN_SAC_INVOICE.sh | Entry-level package that orchestrates SAC invoice generation, CCP/add-on order processing, loyalty points recording, e-invoice creation, and post-invoice SMS notifications. |
| PKG_COMMUNICATION | PKG_COMMUNICATION.pck | package | PKG_ADDON_SALE | Handles outbound SMS and email dispatch for add-on sale events, reading dealer, zone, and channel configuration to build and queue communication messages. |
| PKG_LOYALTY | PKG_LOYALTY.pck | package | PKG_ADDON_SALE | Manages customer loyalty programme transactions — enrolment checks, points accrual, tier evaluation, OTP logging, and reconciliation — triggered by service invoice events. |
| PKG_SERV_PROCESS_DATA | PKG_SERV_PROCESS_DATA.pck | package | PKG_ADDON_SALE | Compiles service invoice detail (parts, operations, charges) into staging tables and initiates e-invoice generation for service transactions. |
| FNC_VERIF_MOBILE | FNC_VERIF_MOBILE.fnc | function | PKG_ADDON_SALE | Validates a mobile number string; returns a boolean or corrected value to the calling procedure before SMS dispatch. |
| SP_GEN_SEQ_NUM | PKG_ADDON_SALE.pck | procedure | PKG_ADDON_SALE | Generates the next document sequence number by locking and updating `AM_MUL_DOCS`, ensuring uniqueness across concurrent calls. |
| SP_PUSH_CCPSMS | PKG_ADDON_SALE.pck | procedure | PKG_ADDON_SALE | Validates mobile numbers, dispatches post-sale SMS notifications for CCP orders, and initiates FDI data push via `PKG_FDI_SERV`. |
| SF_PRINCIPAL_MAP_CD | SF_PRINCIPAL_MAP_CD.fnc | function | PKG_LOYALTY | Looks up and returns the principal mapping code for a given dealer location from `AM_DEALER_LOC`. |
| PKG_EINV | PKG_EINV.pck | package | PKG_SERV_PROCESS_DATA | Generates e-invoice records (B2B and B2C), produces QR codes, posts commission detail rows, and updates enrolment tables for multiple transaction types. |
| SF_CALCULATE_RUPEES_TO_POINTS | PKG_ADDON_SALE.pck | function | PKG_ADDON_SALE | Converts a monetary amount in rupees to an equivalent loyalty points value according to the programme conversion rate. |
| SP_GET_VIN_LOYALTY_DTL | PKG_ADDON_SALE.pck | procedure | PKG_ADDON_SALE | Retrieves loyalty enrolment and status details for a given vehicle identification number (VIN). |
| SP_INS_LOYALTY_TRANSACTIONS | PKG_ADDON_SALE.pck | procedure | PKG_ADDON_SALE | Records a loyalty points credit or debit transaction in the loyalty transaction tables for a completed service event. |
| SP_SEND_SMS | PKG_ADDON_SALE.pck | procedure | PKG_ADDON_SALE | Queues an SMS notification record into `GM_SMS_STATUS` for outbound delivery by the messaging layer. |
| SP_SEND_SMS_SRV_CCP_SALE | PKG_ADDON_SALE.pck | procedure | PKG_ADDON_SALE | Sends a CCP-sale-specific SMS to the customer following a successful service add-on invoice transaction. |
| VAL_INS_INVOICES_B2C_CCP | PKG_ADDON_SALE.pck | function | PKG_ADDON_SALE | Validates and inserts B2C e-invoice records for CCP (Comprehensive Care Plan) transactions, returning a success/failure indicator. |
| PKG_FDI_SERV | PKG_FDI_SERV.pck | package | SP_PUSH_CCPSMS | Coordinates service-related FDI (Franchisee Data Integration) data pushes — extended warranty, AMC, and MCP transaction records — to the FDI platform via `PKG_FDI`. |
| FNC_VERIF_MOBILE | FNC_VERIF_MOBILE.fnc | function | SP_PUSH_CCPSMS | Validates a mobile number string before CCP SMS dispatch (same function, called again at this level). |
| PKG_LOYALTY | PKG_LOYALTY.pck | package | PKG_EINV | Provides loyalty-related support to the e-invoice flow — points accrual, tier updates, and reconciliation triggered at invoice commit. |
| SP_B2C_QR_GENERATOR | PKG_ADDON_SALE.pck | procedure | PKG_SERV_PROCESS_DATA | Generates a B2C e-invoice QR code string and writes it to the target e-invoice staging record. |
| SP_QR_GENERATOR | PKG_ADDON_SALE.pck | procedure | PKG_SERV_PROCESS_DATA | Generates a standard (B2B) e-invoice QR code string and writes it to the target e-invoice staging record. |
| PKG_FDI | PKG_FDI.pck | package | PKG_FDI_SERV | Core FDI integration package that constructs and pushes financial transaction payloads (GRN, invoices, receipts, stock transfers) to the FDI platform. |
| SP_PUSH_CCPC_FDI_DATA | PKG_ADDON_SALE.pck | procedure | SP_PUSH_CCPSMS | Pushes CCP cancellation (CCPC) transaction data to the FDI integration layer for reporting. |
| SF_PRINCIPAL_MAP_CD | SF_PRINCIPAL_MAP_CD.fnc | function | PKG_LOYALTY | Looks up and returns the principal mapping code for a given dealer location (same function, called at this deeper level). |
| FN_GET_LOY_SAC_CODE | PKG_ADDON_SALE.pck | function | PKG_EINV | Returns the SAC (Service Add-on Contract) loyalty programme code to be applied when generating an e-invoice for a loyalty-eligible transaction. |
| VAL_INS_INVOICES_B2C | PKG_ADDON_SALE.pck | function | PKG_EINV | Validates and inserts a B2C (business-to-consumer) e-invoice record; returns a status indicator consumed by the e-invoice flow. |
| SP_PUSH_FDI_TRANSACTIONS | PKG_ADDON_SALE.pck | procedure | PKG_FDI_SERV | Constructs and submits FDI transaction records to `GD_FDI_TRANS` and related charge/customer tables for downstream financial reporting. |

### 11.3 Package procedure specification (entry SP)

| Field | Value |
| --- | --- |
| Qualified name | `PKG_ADDON_SALE.SP_GEN_SAC_INVOICE` |
| Raw call | `PKG_ADDON_SALE.SP_GEN_SAC_INVOICE(1)` |
| High-level steps | 1. Reads pending SAC order records from `VT_SAC_ORDER` and associated repair-order data from `VH_RO`. 2. Checks loyalty enrolment status for each vehicle/customer via `GD_LOYALTY_ENROL` and resolves city data from `GM_CITY`. 3. Generates unique document sequence numbers by calling the local helper `SP_GEN_SEQ_NUM` (updating `AM_MUL_DOCS`). 4. Creates or updates add-on transaction records in `VT_ADDON`. 5. Delegates service invoice data compilation to `PKG_SERV_PROCESS_DATA`, which populates staging tables and triggers e-invoice generation via `PKG_EINV`. 6. Dispatches post-invoice SMS notifications via `SP_PUSH_CCPSMS` (which validates the mobile number, queues messages through `GM_SMS_STATUS`, and calls `PKG_FDI_SERV` for FDI data push). 7. Records loyalty points transactions for eligible customers through local helpers `SP_GET_VIN_LOYALTY_DTL`, `SF_CALCULATE_RUPEES_TO_POINTS`, and `SP_INS_LOYALTY_TRANSACTIONS`. 8. Commits all changes; spool log is written by the calling SQL\*Plus session. |

## 12. Shared Functions

| Function | File | Level | Found from |
| --- | --- | --- | --- |
| FNC_VERIF_MOBILE | FNC_VERIF_MOBILE.fnc | 2 | PKG_ADDON_SALE |
| SP_GEN_SEQ_NUM | PKG_ADDON_SALE.pck | 2 | PKG_ADDON_SALE |
| SP_PUSH_CCPSMS | PKG_ADDON_SALE.pck | 2 | PKG_ADDON_SALE |
| SF_PRINCIPAL_MAP_CD | SF_PRINCIPAL_MAP_CD.fnc | 3 | PKG_LOYALTY |
| SF_CALCULATE_RUPEES_TO_POINTS | PKG_ADDON_SALE.pck | 2 | PKG_ADDON_SALE |
| SP_GET_VIN_LOYALTY_DTL | PKG_ADDON_SALE.pck | 2 | PKG_ADDON_SALE |
| SP_INS_LOYALTY_TRANSACTIONS | PKG_ADDON_SALE.pck | 2 | PKG_ADDON_SALE |
| SP_SEND_SMS | PKG_ADDON_SALE.pck | 2 | PKG_ADDON_SALE |
| SP_SEND_SMS_SRV_CCP_SALE | PKG_ADDON_SALE.pck | 2 | PKG_ADDON_SALE |
| VAL_INS_INVOICES_B2C_CCP | PKG_ADDON_SALE.pck | 2 | PKG_ADDON_SALE |
| FNC_VERIF_MOBILE | FNC_VERIF_MOBILE.fnc | 3 | SP_PUSH_CCPSMS |
| SP_B2C_QR_GENERATOR | PKG_ADDON_SALE.pck | 3 | PKG_SERV_PROCESS_DATA |
| SP_QR_GENERATOR | PKG_ADDON_SALE.pck | 3 | PKG_SERV_PROCESS_DATA |
| SP_PUSH_CCPC_FDI_DATA | PKG_ADDON_SALE.pck | 3 | SP_PUSH_CCPSMS |
| SF_PRINCIPAL_MAP_CD | SF_PRINCIPAL_MAP_CD.fnc | 5 | PKG_LOYALTY |
| FN_GET_LOY_SAC_CODE | PKG_ADDON_SALE.pck | 4 | PKG_EINV |
| VAL_INS_INVOICES_B2C | PKG_ADDON_SALE.pck | 4 | PKG_EINV |
| SP_PUSH_FDI_TRANSACTIONS | PKG_ADDON_SALE.pck | 4 | PKG_FDI_SERV |

Only helpers **called from the entry path** are listed. ZIP `.fnc` files not reached from the entry routine are inventory-only (see §9 / unclassified). Do **not** paste full function source here.

## 13. Interfaces

### 13.1 Database

This job talks to Oracle via `sqlplus` as observed in source.

### 13.2 External systems

N/A — No external file feeds, REST APIs, FTP targets, or named third-party systems are directly declared in the shell script or the scanned package source. Downstream SMS delivery and FDI platform interactions are mediated through internal database tables (`GM_SMS_STATUS`, `GD_FDI_TRANS`, `GD_FDI_TRANS_CHARGES`, `GD_FDI_TRANS_CUSTOMER`); the actual transport layer that reads those tables is outside this job's scope and is not identified in the source artifacts.

- Log file path(s): `/home/dmsdb/logs/GEN_SAC_INVOICE.log` — operational, not a user report.
- Tables/views written by this job (entry-path static scan, WRITE access): `AM_DEALER_LOC_PARAM`, `AM_MUL_DOCS`, `BUSINESS_TEST_TABLE_TEMP`, `GBL_EINV_STG_TAB`, `GBL_SERV_INV_DETL`, `GBL_SERV_STG_TAB`, `GD_EINV_TEMP`, `GD_FDI_TRANS`, `GD_FDI_TRANS_CHARGES`, `GD_FDI_TRANS_CUSTOMER`, `GD_LOYALTY_CARD_VIN_MAP`, `GD_LOYALTY_ENROL`, `GD_LOYALTY_PRE_ENROLL`, `GD_LOYALTY_TRANS`, `GD_LOY_BADGE_UTL`, `GD_LOY_OTP_LOG`, `GD_LOY_RECON`, `GLB_LOY_DETAILS`, `GLB_TEMP_GD_LOY_PRINT`, `GLB_TEMP_GD_LOY_PRINT1`, `GLB_TEMP_LOY_INV`, `GLB_TEMP_LOY_POCORD`, `GM_EMAIL_STATUS`, `GM_SMS_STATUS`, `MWAR_EXTE`, `NOWAIT`, `PH_ISSUE`, `PH_RETURNS`, `QR`, `RSA_REG_DTL` — SME confirms downstream consumers.
- Read-only masters used as inputs are **not** listed here (see §7.2 / §10).

## 14. Non-Functional Requirements

| Topic | Requirement | Source |
| --- | --- | --- |
| Availability window | The job is scheduled to run **every 5 minutes**, 24 × 7. The database and batch host must be available to support this cadence. No blackout window is specified in the cron register or source code. | Cron execution register (row 688); `GEN_SAC_INVOICE.sh` |
| Max runtime | N/A — no maximum runtime threshold is defined in the source or register. | Ops/client |
| Rerun | N/A — no rerun or recovery procedure is coded in the shell script. Failed orders are individually rolled back within the same run; reprocessing relies on the order remaining with `txn_status = 'N'` for the next scheduled invocation. | `PKG_ADDON_SALE.SP_GEN_SAC_INVOICE` (per-order rollback pattern) |
| Audit of who ran | N/A — the batch session credentials are supplied via environment variables (`UD` / `PD`) from the batch profile; no named user or audit trail beyond the spool log is coded. | `GEN_SAC_INVOICE.sh`; `dms_cron_profile` |
| PII in logs | Customer mobile numbers and names are present in the `vt_sac_order` source cursor and in the procedure variables processed; the spool log at `/home/dmsdb/logs/GEN_SAC_INVOICE.log` may capture SQL\*Plus output that includes this data. PII handling and log retention controls are subject to ops/client policy. | `PKG_ADDON_SALE.SP_GEN_SAC_INVOICE` — `cur_r.mobile`, `cur_r.cust_name`; spool path `GEN_SAC_INVOICE.sh` |

## 15. Assumptions, Constraints, N/A Register

| Item | Classification | Notes |
| --- | --- | --- |
| Not an Oracle Report | N/A | No layout, SRW, RDF parameters, attached PLL, format triggers |
| Profile / sourced files outstanding | Constraint | routine:SF_GET_MODEL_CHANNEL; package:PKG_SFDC_SMS; routine:SEND_SMS; package:PKG_ENQ_POC_DETAIL; package:PKG_SPR_EINV; routine:SF_OLD_VAR_DESC; routine:SP_PUSH_EWC_FDI_DATA; routine:SP_PUSH_EWC_FDI_DATA_SM; routine:SP_PUSH_EWU_FDI_DATA; routine:SP_PUSH_EWU_FDI_DATA_SM; routine:SP_PUSH_EW_FDI_DATA; routine:SP_PUSH_EW_FDI_DATA_SM; routine:SP_PUSH_MCPC_FDI_DATA; routine:SP_PUSH_MCP_FDI_DATA; package:PKG_SFDC_SMS; routine:SEND_SMS; routine:SF_GET_BONUS_AGE_SLAB; routine:SF_GET_GRN_DISCOUNT; routine:SF_GET_GST_CHARGES_PR; routine:SF_GET_HSN; routine:SF_GET_TOTAL_BONUS; routine:SP_PUSH_COUNTERSALE_INVOICE; routine:SP_PUSH_COUNTERSALE_RETURN; routine:SP_PUSH_PART_RECIPT |
| BRD status | Assumption | As-is technical + business draft until SME sign-off |
| Schedule | Fact | Sourced from cron execution register — see §4.3 |
| Owners | Assumption | N/A |
| Static dependency scan | Constraint | Not live database dependency metadata |

## 16. Traceability Matrix

| BR-ID | Job step / package.sp | File | Test idea |
| --- | --- | --- | --- |
| BR-001 | Invoke PKG_ADDON_SALE.SP_GEN_SAC_INVOICE | GEN_SAC_INVOICE.sh | Run job; confirm EXEC/entry call in spool log |
| BR-002 | Entry procedure data path → PKG_ADDON_SALE.SP_GEN_SAC_INVOICE | PKG_ADDON_SALE.pck | Verify writes to entry-path WRITE tables (AM_DEALER_LOC_PARAM, AM_MUL_DOCS, BUSINESS_TEST_TABLE_TEMP, GBL_EINV_STG_TAB, GBL_SERV_INV_DETL) |
| BR-003 | Spool log written | GEN_SAC_INVOICE.sh | Confirm file exists: /home/dmsdb/logs/GEN_SAC_INVOICE.log |
| BR-004 | Parameters on PKG_ADDON_SALE.SP_GEN_SAC_INVOICE | GEN_SAC_INVOICE.sh | Confirm shell args match EXEC parameters |
| BR-005 | Helpers on entry path: FNC_VERIF_MOBILE, SP_GEN_SEQ_NUM, SP_PUSH_CCPSMS, SF_PRINCIPAL_MAP_CD, SF_CALCULATE_RUPEES_TO_POINTS, SP_GET_VIN_LOYALTY_DTL | §12 | Trace helper call sites in entry procedure body |
| BR-006 | B2C QR code generation → PKG_ADDON_SALE.VAL_INS_INVOICES_B2C_CCP → PKG_SERV_PROCESS_DATA | PKG_ADDON_SALE.pck | Verify `VAL_INS_INVOICES_B2C_CCP` is called when `cust_gst_num = 'GSTUNREGISTERED'`; confirm QR staging record created in `GD_EINV_TEMP` |
| BR-007 | Per-order rollback on failure → PKG_ADDON_SALE.SP_GEN_SAC_INVOICE | PKG_ADDON_SALE.pck | Introduce a sequence-generation error; confirm individual order rolls back while others commit; check `/home/dmsdb/logs/GEN_SAC_INVOICE.log` for error output |
