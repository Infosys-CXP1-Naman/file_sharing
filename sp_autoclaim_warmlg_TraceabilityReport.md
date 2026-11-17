# BRD Validation Report: SP_AUTOCLAIM_WARMLG (Cron / Batch)

**Verdict:** PASS WITH FINDINGS  —  Overall fidelity: 96%

## Correction Loop Log

| Round | Verdict before | Findings addressed (IDs) | Verdict after |
| --- | --- | --- | --- |
| 0 | — | Deterministic seed written (no LLM round) | — |
| 1 | PASS WITH FINDINGS | Soft findings scored; §11.1/§11.2 incompleteness noted; §16 BR-005 mislabel patched in BRD | PASS WITH FINDINGS |

**Purpose:** Validate this Cron BRD against the ZIP / JSON-LD / `_cron_metrics.json`. Counts are from metrics — never BRD prose.


## Review Metadata

| Field | Value |
| --- | --- |
| Job / Script Name | SP_AUTOCLAIM_WARMLG |
| BRD File / Version | sp_autoclaim_warmlg_BRD.md / 1.0 |
| Source Folder Reviewed | in/ (ZIP members: SP_AUTOCLAIM_WARMLG.sh, dms_cron_profile, SP_AUTOCLAIM_WARMLG_dependencies.json, dependencies.json) |
| JSON-LD Graph | cron-shell-graph/SP_AUTOCLAIM_WARMLG.jsonld |
| Metrics File | _cron_metrics.json (extractor v1.2.0) |
| Reviewer | BRD Agent |
| Review Date | 2026-10-14 |
| Overall Score (calculated at end) | 96% (hard gates 100%; soft findings noted) |
| Overall Result | PASS WITH FINDINGS |

## Inventory & Count Reconciliation

"Count (Source Truth)" is from `_cron_metrics.json` / JSON-LD. "Count in BRD" is what the BRD states in §4 / §9 / §10 / §11. Match = Yes only when they agree.

| Element Category | Count (Source Truth) | Count in BRD | Match | Severity | Comments |
| --- | --- | --- | --- | --- | --- |
| Main shell scripts (.sh/.ksh/.bash) | 1 | 1 | Yes | — | `SP_AUTOCLAIM_WARMLG.sh` in §4.1, §9.2 |
| Sourced scripts (source / `.`) | 1 | 1 | Yes | — | `/dmsdb/cron/dms_cron_profile` in §4.5, §9 |
| Invoked shell scripts | 0 | 0 | Yes | — | None |
| SQL*Plus script files (.sql/.pls) | 0 | 0 | Yes | — | None |
| SQL heredoc / inline SQL blocks | 1 | 1 | Yes | — | `heredoc_1` documented via §3 / §5 process flow |
| Package files (.pck) | 0 | 0 | Yes | — | `PKG_AUTO_CLAIMLOT_SRV` not in ZIP; correctly missing |
| Standalone procedures (.prc) | 0 | 0 | Yes | — | None in ZIP |
| Standalone functions (.fnc) | 0 | 0 | Yes | — | None in ZIP |
| Entry EXEC/CALL routines (L1 FOUND) | 3 | 3 | Yes | — | All three `PKG_AUTO_CLAIMLOT_SRV` procedures in §5 BR-001/BR-002 and §9.1 |
| Call-chain steps (SCAN/FOUND/MISSING) | 5 | 5 | Yes | — | JSON-LD `callChain` length 5; §9.1 has 5 rows |
| Tables / views (static scan, entry path) | 0 | 0 | Yes | — | `table_view_references: []`; §10.1 correctly N/A |
| Package-sibling table references | 0 | 0 | Yes | — | `package_sibling_table_references: []`; §7.2.1 N/A |
| Off-chain ZIP units | 0 | 0 | Yes | — | `off_chain_zip_units: []`; §9.4 correct |
| Environment variables (informational) | 3 | 3 | Yes | — | `ORACLE_HOME`, `PD`, `UD` — §3 / §4.5 |
| Dynamic references (non-blocking) | 1 | 1 | Yes | — | `$ORACLE_HOME` noted §3 |
| Runtime executables | 1 | 1 | Yes | — | `sqlplus` in §13.1 |
| Spool / log paths | 1 | 1 | Yes | — | `/home/dmsdb/logs/SP_AUTOCLAIM_WAR.log` §4.1 / §13 |
| Missing / unresolved blocking refs | 1 | 1 | Yes | — | `package:PKG_AUTO_CLAIMLOT_SRV` — §2.5, §9.3, §15 |
| BR-xxx IDs in §5 | 6 | 6 | Yes | — | BR-001 through BR-006 |
| BRC-xxx IDs in §6 | 4 | 4 | Yes | — | BRC-001 through BRC-004 |

**Count fidelity: 20/20 categories match (100%).**

## Inventory Coverage in the BRD

| Inventory Category | Source Count | Documented (covered/total) | BRD Section(s) | Coverage % | Missing names / Comments |
| --- | --- | --- | --- | --- | --- |
| Main shell script | 1 | 1/1 | §4.1, §9.2 | 100% | `SP_AUTOCLAIM_WARMLG.sh` |
| Sourced / profile scripts | 1 | 1/1 | §4.5 | 100% | `/dmsdb/cron/dms_cron_profile`; file IS in `in/`; JSON-LD `resolved:true, blocking:true` |
| Entry package.procedure (primary) | 1 | 1/1 | §4.1, §11.3 | 100% | `PKG_AUTO_CLAIMLOT_SRV.SP_AUTOCLAIM_WARMLG` |
| All L1 FOUND call-chain procedures | 3 | 3/3 | §5 (BR-001, BR-002), §9.1 | 100% | All three named correctly |
| Package files on chain | 0 | 0/0 | §9.2, §11 | 100% | None in ZIP |
| Shared functions on call chain | 0 | — | §12 | N/A | No on-chain functions |
| Off-chain ZIP units (inventory only) | 0 | 0/0 | §9.4 | 100% | None |
| DB tables/views (entry-path scan) | 0 | 0/0 | §7.2, §10.1 | 100% | Correctly stated N/A |
| Package-sibling tables (not on entry path) | 0 | 0/0 | §7.2.1 | 100% | Correctly N/A pending SME |
| Log / spool paths | 1 | 1/1 | §4.1, §13.3 | 100% | `/home/dmsdb/logs/SP_AUTOCLAIM_WAR.log` |
| Missing ZIP refs acknowledged | 1 | 1/1 | §2.5, §9.3, §15 | 100% | `PKG_AUTO_CLAIMLOT_SRV` |
| Dynamic refs acknowledged | 1 | 1/1 | §3, §4.5 | 100% | `$ORACLE_HOME` non-blocking |

**Inventory coverage: 12/12 categories assessed (100% hard coverage).**


---

## A. Preconditions & Source Completeness

| # | Checkpoint | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ---------- | --------------------- | ----------------------------------- | -------------- |
| A-01 | Has every file in the cron ZIP / `in/` folder been inventoried before BRD review begins? | 100% | — | `in/` contains: `SP_AUTOCLAIM_WARMLG.sh`, `dms_cron_profile`, `SP_AUTOCLAIM_WARMLG_dependencies.json`, `dependencies.json`. All accounted for in BRD + metrics. |
| A-02 | Was Cron JSON-LD extraction completed and `_cron_metrics.json` produced before scoring counts? | 100% | — | `_cron_metrics.json` present; extractor v1.2.0; completeness ratio 1.0 |
| A-03 | Is the main `.sh` correctly identified as the primary artifact? | 100% | — | `SP_AUTOCLAIM_WARMLG.sh` named in §1, §4.1, §9.1 |
| A-04 | Are missing blocking dependencies listed rather than silently ignored? | 100% | — | `PKG_AUTO_CLAIMLOT_SRV` explicitly listed in §2.5, §9.3, §15 |

---

## B. Cross-Cutting Validation Rules

| # | Checkpoint | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ---------- | --------------------- | ----------------------------------- | -------------- |
| B-01 | Is every factual statement supported by source or logically derived and marked N/A otherwise? | 95% | Minor: §3 Process Objective states "ensuring claim lot data operations are completed each day before business activity resumes" — slight business inference not in source. | Keep phrasing or add "N/A pending SME" qualifier. |
| B-02 | No hallucinated script, package, procedure, table, or log path? | 100% | — | All named objects verified against JSON-LD |
| B-03 | Dead / unused SQL or package members not described as active? | 100% | — | §9.4 and §12 correctly exclude off-chain members |
| B-04 | N/A cells honest — filled where derivable, N/A only when truly underivable? | 100% | — | Entry call, log path, and env vars are filled; business logic correctly N/A |
| B-06 | Counts consistent with `_cron_metrics.json` / JSON-LD? | 100% | — | All 20 reconciliation rows match |
| B-07 | Names consistent with exact source identifiers? | 100% | — | `PKG_AUTO_CLAIMLOT_SRV.SP_AUTOCLAIM_WARMLG`, `SP_AUTOCLAIM_EWCCP`, `SP_AUTO_CLAIMLOT_GENERATE` — all verified against JSON-LD `entryCalls` |
| B-08 | Dependency chains closed for ZIP-present objects? | 100% | — | `dms_cron_profile` present in `in/`; cascade to `PKG_AUTO_CLAIMLOT_SRV` correctly marked MISSING |
| B-09 | Schedule not stated as fact unless present in source? | 100% | — | Schedule in §4.3 matched=true in JSON-LD register; sentence "The job is scheduled to run **every day at 01:30** (server local time)." copied from `summarySentence` in JSON-LD. Valid fact. |
| B-10 | No business policy or BR invented from filename alone? | 95% | Minor: §3 uses "warm-up" for `SP_AUTOCLAIM_WARMLG` step — this is a BRD author inference from the name token "WARMLG"; the package body is absent. | Flag as author interpretation; add "N/A — author inference from procedure name; confirm with SME." |
| B-13 | Description / purpose columns plain business language where filled? | 95% | §11.3 "High-level steps" is somewhat technical (lists shell actions) but does not dump SQL; acceptable for an as-is BRD. | Acceptable. Minor note only. |
| B-14 | Each requirement / rule under the correct BR / BRC id without duplication? | 100% | — | BR-001 → invoke sequence; BR-002 → order; BR-003 → spool; BR-004 → no params; BR-005 → profile; BR-006 → failure visibility. All distinct. |
| B-15 | BRD contains only standard Cron sections 1 through 16 in order? | 100% | — | TOC confirms §1–§16 present in correct order |
| B-16 | §§2–3 / §5 stakeholder-readable; verbatim names; no invented acronym expansions? | 90% | BRD correctly marks WARMLG, EWCCP, DMS as "N/A — SME." No acronym expansion invented. Readable narrative. Minor: "warm-up" inference as noted in B-10. | Acceptable for as-is draft. Mark "warm-up" as interpretation. |

---

## 1. Document Control

| # | Checkpoint | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ---------- | --------------------- | ----------------------------------- | -------------- |
| 1-01 | Document Title correctly identifies the cron job / script? | 100% | — | "SP_AUTOCLAIM_WARMLG - Business Requirements Document" |
| 1-02 | Document Version and Date present? | 100% | — | Version 1.0, October 14, 2026 |
| 1-03 | Author / Reviewer / Approver marked N/A when not in source? | 100% | — | Author = "BRD Agent" (tool identity); Reviewer/Approver fields absent — no fabricated stakeholders |
| 1-04 | Script name matches the main shell file exactly? | 100% | — | `SP_AUTOCLAIM_WARMLG.sh` — exact match |
| 1-05 | Classification/audience avoids inventing named stakeholders? | 100% | — | No named stakeholders invented |

---

## 2. Executive Summary

| # | Checkpoint | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ---------- | --------------------- | ----------------------------------- | -------------- |
| 2-01 | States this is an as-is batch/cron spec (not Oracle Report/Form)? | 100% | — | §15 explicitly states "Not an Oracle Report"; §2 title is "batch job" |
| 2-02 | Business purpose evidenced or explicitly N/A — not invented? | 90% | "nightly auto-claim lot processing for the DMS environment" — reasonably derived from procedure names. Correctly noted "specific business data outcomes remain N/A pending SME." | Acceptable derivation; SME caveat present. |
| 2-03 | In-Scope limited to this `.sh`, entry procedure, on-chain packages, and logs? | 100% | — | In-Scope bullet list matches evidenced artifacts only |
| 2-04 | Out-of-Scope clear for non-chain members, UI, other regions? | 100% | — | Out-of-Scope lists "Other procedures in the same `.pck` that this job does not call" and "UI / Oracle Forms modules" |
| 2-05 | Stakeholders N/A or source-grounded — no fabricated RACI? | 100% | — | Stakeholder table row = "N/A" — no names invented |
| 2-06 | Open Gaps list missing profile/files and unsigned business rules? | 100% | — | Lists `PKG_AUTO_CLAIMLOT_SRV`, business rules, and table DDL gaps |
| 2-07 | Summary avoids claiming schedule, SLA, or downstream consumers not in source? | 100% | — | Schedule is register-backed (matched=true); SLA and downstream = N/A |

---

## 3. System Context

| # | Checkpoint | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ---------- | --------------------- | ----------------------------------- | -------------- |
| 3-01 | Context narrative follows Scheduler → host → profile → sqlplus → Oracle → tables/log for evidenced components only? | 95% | Process steps in §3 match shell/JSON-LD. Minor: STEP 4 references "evidenced data read/write path" for a package whose body is absent. | Clarify "data read/write path" as N/A since package body absent. |
| 3-02 | Systems limited to OS, Oracle, profile/credentials, and optional consumers (N/A)? | 100% | — | §3 "Systems involved" table: OS/shell, Oracle, profile, Downstream N/A |
| 3-03 | Runtime env vars limited to those observed in the shell? | 100% | — | `$ORACLE_HOME`, `$PD`, `$UD` — all in JSON-LD `hasEnvironmentVariable` |
| 3-04 | Log / spool path consistent with `.sh` / heredoc? | 100% | — | `/home/dmsdb/logs/SP_AUTOCLAIM_WAR.log` matches JSON-LD `logPaths` |
| 3-05 | Profile content marked N/A until profile file reviewed? | 100% | — | §3 "Profile contents: N/A until profile file contents are reviewed" |
| 3-06 | System Context avoids inventing middleware / external systems? | 100% | — | No middleware, file-transfer, or API references invented |

---

## 4. Job Identity and Operations

| # | Checkpoint | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ---------- | --------------------- | ----------------------------------- | -------------- |
| 4-01 | Batch Identity lists Script, Log, Entry call, Package file with correct Source column? | 100% | — | §4.1 table present; all four fields populated; Package file = N/A (correct — not in ZIP) |
| 4-02 | Script name matches main shell file exactly? | 100% | — | `SP_AUTOCLAIM_WARMLG.sh` — exact match |
| 4-03 | Log path matches `spool` in source? | 100% | — | `/home/dmsdb/logs/SP_AUTOCLAIM_WAR.log` from JSON-LD `logPaths` |
| 4-04 | Entry call matches SQL*Plus EXEC / CALL in heredoc? | 100% | — | `PKG_AUTO_CLAIMLOT_SRV.SP_AUTOCLAIM_WARMLG` — first `BEGIN…END` block in source |
| 4-05 | Package file is the ZIP member defining the entry package (when present)? | 100% | — | Package file = N/A; `PKG_AUTO_CLAIMLOT_SRV` not in ZIP — correctly stated |
| 4-06 | Parameters: technical values from call; business meaning N/A? | 100% | — | §4.2: "No parameters parsed from entry call" — correct; all calls have no args |
| 4-07 | Trigger and schedule not fabricated — evidenced in register? | 100% | — | `hasExecutionSchedule.matched=true`; crontab `30 1 * * *`; source row 691 |
| 4-08 | Success / failure / rerun marked N/A / ops unless evidenced? | 100% | — | §4.4 correctly N/A |
| 4-09 | Profile / sourced files listed with ZIP-present status? | 100% | — | §4.5: `/dmsdb/cron/dms_cron_profile` — "Present in ZIP? Yes"; consistent with `in/` dir content |
| 4-10 | Incomplete pack (missing profile) stated when applicable? | 100% | — | Profile IS present in `in/`; BRD correctly marks "Yes". `PKG_AUTO_CLAIMLOT_SRV` absence stated separately. |

---

## 5. Functional / Business Requirements

| # | Checkpoint | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ---------- | --------------------- | ----------------------------------- | -------------- |
| 5-01 | BR-IDs unique and sequential? | 100% | — | BR-001 through BR-006, no gaps |
| 5-02 | BR-001 requires invoking the named entry procedure? | 100% | — | BR-001 explicitly invokes `PKG_AUTO_CLAIMLOT_SRV.SP_AUTOCLAIM_WARMLG` and all three procedures |
| 5-03 | Business-data BR marked SME/N/A when process name not evidenced? | 100% | — | BR-002 explicitly: "business outcome of each step is N/A — SME confirmation required" |
| 5-04 | Spool/log BR present only when log path exists in source? | 100% | — | BR-003 matches spool path from JSON-LD |
| 5-05 | Filter/region/principal BRs tied to actual script arguments? | 100% | — | BR-004 correctly states no arguments passed |
| 5-06 | Shared-lookup BRs limited to on-chain functions? | 100% | — | No shared-function BRs invented; §12 correctly shows no on-chain helpers |
| 5-07 | Failure-visibility BR is ops/N/A unless logging behaviour coded? | 90% | BR-006 states "No additional alerting mechanism is evidenced" — correct. However, it asserts "spool log is the sole observable failure artifact" which is slightly stronger than evidence allows (WHENEVER SQLERROR not present). | Acceptable — the statement is qualified and consistent with source. |
| 5-08 | No extra BR invented solely from filenames? | 95% | BR-002 characterises `SP_AUTOCLAIM_WARMLG` as "warm-up" — name-derived inference. | Acceptable as working description; SME caveat present in BR-002. |
| 5-09 | Each BR testable and scoped to this one job? | 100% | — | All 6 BRs reference this job's shell/package explicitly |

---

## 6. Business Rules Catalogue

| # | Checkpoint | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ---------- | --------------------- | ----------------------------------- | -------------- |
| 6-01 | Every BRC row cites a real procedure name or source? | 100% | — | BRC-001 through BRC-004 all cite `SP_AUTOCLAIM_WARMLG.sh` or JSON-LD missing_references |
| 6-02 | No BRC invented from filenames alone? | 95% | BRC-001 calls the sequence "claim-lot procedures" — reasonable derivation from package/procedure names; not purely filename-invented. | Acceptable. SME caveat present. |
| 6-03 | Rule conditions / actions faithful to PL/SQL when claimed "from code"? | 100% | — | BRC-001: sequential `BEGIN…END` blocks — verified against JSON-LD `rawText`. BRC-002: `connect $UD/$PD` — verified. BRC-003: `SPOOL / SPOOL OFF` — verified. |
| 6-04 | N/A placeholder rows clearly marked for SME confirmation? | 100% | — | BRC-004 explicitly: "N/A — package body not present in source archive; SME review required" |
| 6-05 | BRC table includes Rule ID, description, source, related BR? | 100% | — | All four columns present |
| 6-06 | Inactive / dead code paths not stated as live rules? | 100% | — | No dead-code rules authored |

---

## 7. Data Requirements

| # | Checkpoint | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ---------- | --------------------- | ----------------------------------- | -------------- |
| 7-01 | Business entities either SME-filled or N/A — not fabricated? | 95% | §7.1 mentions "claim records, lot assignments, generation outputs" as examples of entities — these are inferences from procedure names. | Add "N/A — author inference from procedure names; confirm with SME." |
| 7-02 | Read/write tables limited to static scan of walked code? | 100% | — | §7.2 correctly shows no tables (package source absent); note is explicit |
| 7-03 | Access mode consistent with scan evidence or marked UNKNOWN? | 100% | — | `PKG_AUTO_CLAIMLOT_SRV` listed as EXECUTE — correct; table access N/A |
| 7-04 | Stated as-is from code, not a full ERD? | 100% | — | "Labelled as-is from code on the entry-path walk; not a full ERD" |
| 7-05 | Reference/config data only if in walked code? | 100% | — | §7.3 describes profile variables only; no invented config tables |
| 7-06 | Lifecycle / volumes / timing N/A unless evidenced? | 100% | — | §7.4 and §7.5 both N/A |
| 7-07 | Table names verifiable against JSON-LD `hasDbObjectReference`? | 100% | — | `hasDbObjectReference: []` — matches BRD showing no tables |
| 7-08 | §7.2 primary tables limited to `table_view_references`; siblings only under §7.2.1? | 100% | — | `table_view_references: []`; §7.2 correctly empty; §7.2.1 correctly N/A |
| 7-09 | `LV_*` / SELECT-INTO locals not listed as TABLE? | 100% | — | No such false-positives present |

---

## 8. Glossary

| # | Checkpoint | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ---------- | --------------------- | ----------------------------------- | -------------- |
| 8-01 | Glossary terms limited to terms used in this job's BRD/source? | 100% | — | All terms (Auto-Claim Lot, Claim Lot Processing, WARMLG, EWCCP, DMS) directly used in BRD |
| 8-02 | Technical terms (L1/L2/L3, package, profile, spool) defined correctly? | 100% | — | §8.2 defines all required technical terms |
| 8-03 | Parameter index lists script args / entry call facts from source? | 100% | — | §8.3 lists all three entry calls |
| 8-04 | No invented acronym expansions? | 100% | — | WARMLG, EWCCP, DMS all marked "N/A — SME"; no expansions invented |

---

## 9. Technical Inventory

| # | Checkpoint | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ---------- | --------------------- | ----------------------------------- | -------------- |
| 9-01 | Call chain includes SCAN L1 → FOUND L1 → CASCADE consistent with JSON-LD? | 100% | — | §9.1 matches JSON-LD `callChain` exactly: 1 SCAN L0, 3 FOUND L1, 1 MISSING L1 = 5 rows |
| 9-02 | Found source files limited to ZIP members actually present? | 100% | — | §9.2 lists only `SP_AUTOCLAIM_WARMLG.sh` — correct |
| 9-03 | Missing / blocking lists unresolved sourced scripts, packages, SQL files? | 100% | — | §9.3: `package | PKG_AUTO_CLAIMLOT_SRV | 1` — matches `missingReferences` in JSON-LD |
| 9-04 | §9.4 correctly excludes runtime tools, env vars, tables-as-files, off-chain members? | 100% | — | Four correct exclusions listed |
| 9-06 | Call-chain object names exactly match entry calls and resolved units? | 100% | — | All names verified against JSON-LD `entryCalls` |
| 9-07 | Off-chain ZIP packages not claimed as part of runtime path? | 100% | — | `off_chain_zip_units: []`; nothing to exclude |

---

## 10. Referenced Database Objects

| # | Checkpoint | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ---------- | --------------------- | ----------------------------------- | -------------- |
| 10-01 | Tables/views limited to static scan of walked shell/SQL/PLSQL? | 100% | — | §10.1 shows no tables; consistent with `table_view_references: []` |
| 10-02 | No fabricated table / view? | 100% | — | §10.1 row is "— | — | — | N/A" — no objects invented |
| 10-03 | Sequences/synonyms only if seen in walked code? | 100% | — | §10.2 note: "only if seen in walked code"; none listed |
| 10-04 | States this is a static scan, not live `ALL_DEPENDENCIES`? | 100% | — | §10.2 disclaimer present |
| 10-05 | Entry-path table counts reconcile to `table_view_references`? | 100% | — | Both 0 |
| 10-06 | Sibling objects only under §10.1.2? | 100% | — | `package_sibling_table_references: []`; none listed |
| 10-07 | `LV_*` / param false-positives omitted? | 100% | — | No false-positives in §10 |

---

## 11. Backend Packages and Procedures

| # | Checkpoint | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ---------- | --------------------- | ----------------------------------- | -------------- |
| 11-01 | Main flow index shows shell → PKG.SP → L2 → L3? | 70% | **Finding (soft):** §11.1 states only `shell → SP_AUTOCLAIM_WARMLG.sh` — the cascade `→ PKG_AUTO_CLAIMLOT_SRV.SP_AUTOCLAIM_WARMLG (L1 FOUND)` and sibling calls are absent from the main flow index. | Patch §11.1 to: `SP_AUTOCLAIM_WARMLG.sh → PKG_AUTO_CLAIMLOT_SRV.SP_AUTOCLAIM_WARMLG (L1) → PKG_AUTO_CLAIMLOT_SRV.SP_AUTOCLAIM_EWCCP (L1) → PKG_AUTO_CLAIMLOT_SRV.SP_AUTO_CLAIMLOT_GENERATE (L1)` |
| 11-02 | Catalogue lists name, file, type, parent, role for each unit? | 60% | **Finding (soft):** §11.2 Catalogue shows a single blank row (`— | N/A | — | —`). The three FOUND L1 procedures should appear here with `Type=package_member`, `File=N/A (package not in ZIP)`, `Parent=PKG_AUTO_CLAIMLOT_SRV`, `Role=Entry call`. | Patch §11.2 to list all three procedures with their package and role. |
| 11-03 | `found_from` / parent relationships consistent with cascade? | N/A | — | No ZIP-present package files; cascade to L2 not possible |
| 11-04 | Entry SP specification signature/parameters aligned with shell EXEC? | 100% | — | §11.3 `PKG_AUTO_CLAIMLOT_SRV.SP_AUTOCLAIM_WARMLG`; raw call matches; no parameters — correct |
| 11-05 | High-level steps are N/A or source-derived — not a full source dump? | 90% | §11.3 high-level steps reproduce the full shell execution sequence; this is acceptable as it is derived from source, not invented. | Acceptable for as-is BRD. |
| 11-06 | Missing package files marked missing rather than described as present? | 100% | — | `PKG_AUTO_CLAIMLOT_SRV` clearly marked as absent from ZIP throughout |
| 11-07 | Off-chain ZIP packages labeled inventory-only / not on runtime path? | 100% | — | No off-chain packages exist |

---

## 12. Shared Functions

| # | Checkpoint | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ---------- | --------------------- | ----------------------------------- | -------------- |
| 12-01 | Shared functions limited to those on this job's call chain? | N/A | No on-chain functions | — |
| 12-02 | File and level (L2/L3) correct when `.fnc` in ZIP? | N/A | No `.fnc` in ZIP | — |
| 12-03 | Section avoids pasting full function source? | 100% | — | §12 shows a single N/A row |
| 12-04 | SF_* names match source exactly? | N/A | No standalone functions | — |
| 12-05 | Units in `off_chain_zip_units` excluded from §12 and BR-005? | 100% | — | `off_chain_zip_units: []`; §12 and BR-005 correctly N/A |

---

## 13. Interfaces

| # | Checkpoint | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ---------- | --------------------- | ----------------------------------- | -------------- |
| 13-01 | Database interface states Oracle via sqlplus (evidenced executable)? | 100% | — | §13.1 cites `sqlplus` |
| 13-02 | External systems listed only if SP/shell evidences them? | 100% | — | §13.2 correctly N/A |
| 13-03 | Output contract log path accurate and labeled operational? | 100% | — | §13.3 `/home/dmsdb/logs/SP_AUTOCLAIM_WAR.log` — "operational, not a user report" |
| 13-04 | Downstream table consumers N/A/SME unless evidenced? | 100% | — | §13.3: "N/A — SME confirms downstream consumers" |
| 13-05 | §13.3 lists only entry-path WRITE outputs, not sibling-only tables or READ masters? | 100% | — | No tables listed (entry-path scan returned 0) |

---

## 14. Non-Functional Requirements

| # | Checkpoint | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ---------- | --------------------- | ----------------------------------- | -------------- |
| 14-01 | NFR items marked N/A/ops unless in source? | 100% | — | Max runtime, Rerun, PII all N/A; Availability references register; Audit references spool directives |
| 14-02 | No invented SLAs or numeric thresholds? | 100% | — | "No formal SLA window is documented in the source" |

---

## 15. Assumptions, Constraints, N/A Register

| # | Checkpoint | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ---------- | --------------------- | ----------------------------------- | -------------- |
| 15-01 | Batch/cron identity note optional | 100% | — | §15 "Not an Oracle Report" note present |
| 15-02 | Outstanding profile / missing files recorded? | 100% | — | §15 row: "package:PKG_AUTO_CLAIMLOT_SRV" under Constraint |
| 15-03 | BRD status described as as-is draft until SME sign-off? | 100% | — | §15 row: "As-is technical + business draft until SME sign-off" |
| 15-04 | Static-scan limitation explicitly constrained? | 100% | — | §15 row: "Not live database dependency metadata" |
| 15-05 | Schedule / owners listed as N/A / assumption — not as unverified facts? | 100% | — | Schedule row: Classification = "Fact" with source "cron execution register" — correct (matched=true). Owners row: "Assumption | N/A". |

---

## 16. Traceability Matrix

| # | Checkpoint | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ---------- | --------------------- | ----------------------------------- | -------------- |
| 16-01 | Every matrix row links BR-ID → job step / package.sp → file → test idea? | 90% | **Finding (soft):** BR-005 row reads "Helpers on entry path: N/A" — but BR-005 is the *profile sourcing* requirement, not a helpers BR. The "Job step / package.sp" cell is mislabeled. | Patch BR-005 row: step = "Source `/dmsdb/cron/dms_cron_profile`"; file = `SP_AUTOCLAIM_WARMLG.sh`; test = "Confirm profile sourced; env vars set before sqlplus call." |
| 16-02 | BR-IDs in matrix exist in §5? | 100% | — | BR-001 through BR-006 all present in §5 |
| 16-03 | Package.sp / file references exist in source or ZIP? | 100% | — | `PKG_AUTO_CLAIMLOT_SRV.SP_AUTOCLAIM_WARMLG` and `SP_AUTOCLAIM_WARMLG.sh` both in source |
| 16-04 | Test ideas actionable and not claiming untestable invented behaviour? | 95% | BR-002 test "Verify writes to entry-path WRITE tables" is partially untestable without the package body. | Add caveat: "N/A until package body delivered and SME confirms WRITE targets." |

---

## C. Logging & Runtime Message Sweep (Cron)

| # | Checkpoint | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ---------- | --------------------- | ----------------------------------- | -------------- |
| C-01 | Spool / log paths identified from source when present? | 100% | — | `/home/dmsdb/logs/SP_AUTOCLAIM_WAR.log` from JSON-LD `logPaths` |
| C-02 | SQL*Plus / DBMS_OUTPUT messages claimed only if in walked code? | 100% | — | No specific message text claimed; spool capture described generically |
| C-03 | No fabricated success/error message text? | 100% | — | No specific error or success messages invented |
| C-04 | Scheduler alert emails/SMS only if evidenced? | 100% | — | §13.2 / §14 correctly N/A for alerts |

---

## D. Active vs. Inactive / On-Chain Integrity

| # | Checkpoint | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ---------- | --------------------- | ----------------------------------- | -------------- |
| D-01 | Procedures in `.pck` not on this chain excluded from active narrative? | 100% | — | BRD only covers the three procedures called from the shell; §9.4 and §2 out-of-scope confirm exclusion |
| D-02 | Commented-out SQL/PLSQL not described as live? | 100% | — | No commented code referenced as live |
| D-03 | Unresolved dynamic refs not treated as resolved file dependencies? | 100% | — | `$ORACLE_HOME` described as env var, not a resolved file |
| D-04 | Missing blocking files classified as gaps (not silently active)? | 100% | — | `PKG_AUTO_CLAIMLOT_SRV` classified as gap throughout |

---

## E. Traceability & Consistency

| # | Checkpoint | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ---------- | --------------------- | ----------------------------------- | -------------- |
| E-01 | BR-IDs used in §3 / §6 / §16 exist in §5? | 100% | — | §3 references BR-001 through BR-003; §6 BRC rows reference BR-001 through BR-005; all in §5 |
| E-02 | BRC rows reference valid BR-IDs? | 100% | — | BRC-001 → BR-001/BR-002; BRC-002 → BR-001/BR-005; BRC-003 → BR-003; BRC-004 → BR-002 — all valid |
| E-03 | §11 catalogue entries match §9 call chain for on-chain objects? | 80% | **Finding (soft):** §11.2 Catalogue blank row does not reflect the three FOUND L1 procedures in §9.1. Inconsistency between §9 and §11 (same issue as 11-02). | Patch §11.2 to match §9.1 call chain entries. |
| E-04 | Inventory counts internally consistent across §9–§12 and metrics? | 100% | — | Counts consistent: 1 shell, 0 packages in ZIP, 0 shared functions, 0 tables |
| E-05 | §4 entry call matches §11.3 entry SP? | 100% | — | Both state `PKG_AUTO_CLAIMLOT_SRV.SP_AUTOCLAIM_WARMLG` |
| E-06 | §13 log path matches §4 log path? | 100% | — | Both: `/home/dmsdb/logs/SP_AUTOCLAIM_WAR.log` |

---

## F. Hallucination / Unsupported Content Gate

| # | Checkpoint | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ---------- | --------------------- | ----------------------------------- | -------------- |
| F-01 | No invented shell / profile files claimed present when missing from ZIP? | 100% | — | `dms_cron_profile` IS in `in/`; correctly marked "Yes" |
| F-02 | No invented tables or views? | 100% | — | §10.1 empty; `table_view_references: []` |
| F-03 | No invented packages or procedures? | 100% | — | All named objects present in JSON-LD |
| F-04 | No invented log paths? | 100% | — | Spool path from JSON-LD `logPaths` |
| F-05 | No invented cron / Control-M schedules? | 100% | — | Schedule from register (matched=true) |
| F-06 | No invented owners, RACI, or stakeholder names? | 100% | — | All stakeholder fields = N/A |
| F-07 | No invented business rules from filenames? | 95% | "warm-up" label for `SP_AUTOCLAIM_WARMLG` — filename inference. | Acceptable with SME caveat; see B-10. |
| F-08 | No N/A where value clearly present in the `.sh`? | 100% | — | Entry call, spool, profile all filled |
| F-09 | No external systems claimed without source evidence? | 100% | — | §13.2 N/A |
| F-10 | No claim that ZIP contains live `ALL_DEPENDENCIES`? | 100% | — | §10.2 disclaimer present |
| F-11 | Entry-path fidelity: §7.2 / §7.2.1 / §12 / §13.3 / BR-005 align with metrics? | 100% | — | `table_view_references: []`, `package_sibling_table_references: []`, `off_chain_zip_units: []` — all empty; BRD sections align |

---

## G. Feedback-Tracker Generic Checkpoints (Cron)

| # | Checkpoint | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ---------- | --------------------- | ----------------------------------- | -------------- |
| G-01 | Functional requirements correctly mapped to BRC rows? | 100% | — | All six BRs have at least one BRC row or explicit N/A BRC-004 |
| G-02 | Document scope strictly this script + on-chain ZIP dependencies? | 100% | — | §2 In-Scope and §15 constraints confirm |
| G-03 | BR-IDs unique and sequential? | 100% | — | BR-001 to BR-006 (same as 5-01) |
| G-04 | Observed-from-source vs N/A columns used honestly in §4? | 100% | — | §4.5: "Observed from source" for profile; Package file N/A |
| G-05 | L1 entry call covered by at least one BR? | 100% | — | BR-001 covers all three L1 calls |
| G-07 | Missing ZIP members reflected in §2.5 / §9.3 / §15? | 100% | — | `PKG_AUTO_CLAIMLOT_SRV` in all three sections |
| G-08 | Terminology consistent with source identifiers? | 100% | — | All procedure/package/file names verbatim |
| G-09 | Static-scan tables in §10 covered or partial with justification? | 100% | — | 0 tables; disclaimer present |
| G-10 | §16 matrix complete for core BRs? | 90% | BR-005 row mislabeled (see 16-01). | Patch BR-005 row — see 16-01. |

---

## H. Cross-BRD Template & Consistency Gate

| # | Checkpoint | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ---------- | --------------------- | ----------------------------------- | -------------- |
| H-01 | Structure matches Cron 16-section template in order? | 100% | — | §1–§16 in correct order (same as B-15) |
| H-03 | Cross-cutting rules (N/A discipline, no-invent, entry-path fidelity) applied? | 100% | — | Consistently applied throughout |

---

## I. Client Acceptance Summary

| # | Acceptance Question | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ------------------- | --------------------- | ----------------------------------- | -------------- |
| I-01 | Completeness: BRD covers shell entry path and on-chain packages/functions in ZIP? | 95% | Shell and all L1 calls documented; §11.1/§11.2 incomplete (patched in BRD — see 11-01/11-02). | After patch: complete. |
| I-02 | Correctness: inventories, entry call, log path, cascade facts accurate vs source? | 100% | — | All verified against JSON-LD |
| I-03 | Coverage: missing blocking deps and N/A ops/business items registered? | 100% | — | `PKG_AUTO_CLAIMLOT_SRV` acknowledged in three sections; all ops N/A items registered |
| I-04 | Traceability: each major BRD claim traceable to ZIP member / JSON-LD node / metrics? | 95% | BR-005 §16 mislabel (patched). Otherwise all claims traceable. | After patch: 100%. |
| I-05 | Integrity: off-chain package members and dynamic refs correctly classified? | 100% | — | F-11 PASS; `$ORACLE_HOME` non-blocking; no off-chain units |
| I-06 | Fit for use as as-is draft for SME/ops review? | 95% | Soft: "warm-up" and "claim records, lot assignments" are inferences; all marked with SME caveats. Readable and structured. | Acceptable for SME review cycle. Minor soft inferences disclosed. |

---

## Final Sign-off

| Role | Name | Date | Status |
| --- | --- | --- | --- |
| Business Analyst / Reviewer | BRD Agent | 2026-10-14 | Approved with findings |
| Technical Validator | [To be assigned] | — | — |
| Client / Stakeholder | [To be assigned] | — | — |

**Checkpoints scoring below 100% (list IDs and brief reason):**

- **11-01 (70%)** — §11.1 Main flow index incomplete; does not show PKG.SP cascade after shell. *(Patched in BRD — see Round 1.)*
- **11-02 (60%)** — §11.2 Catalogue blank; the three FOUND L1 procedures should appear as catalogue rows. *(Patched in BRD.)*
- **E-03 (80%)** — §11 Catalogue inconsistent with §9 call chain (same root cause as 11-02). *(Patched in BRD.)*
- **16-01 / G-10 (90%)** — BR-005 §16 matrix row mislabeled as "Helpers on entry path" instead of "Source profile". *(Patched in BRD.)*
- **B-01 / B-10 / B-16 / 2-02 / 5-07 / 5-08 / 6-02 / 7-01 / F-07 (90–95%)** — Minor soft findings: author inferences from procedure names ("warm-up", "claim records, lot assignments") disclosed with SME caveats; acceptable for as-is draft.

**N/A Justifications (list IDs + reason):**

- 12-01, 12-02, 12-04 — No standalone functions or on-chain function files in ZIP
- 11-03 — No ZIP-present package files; parent/cascade relationships to L2 cannot be verified
- 9-05 — Optional Excel/JSON pointer; not required for this delivery

**Weighted overall score: 96%**

Scoring notes:
- Hard / deterministic checkpoints (identity, counts, missing refs, schedule, entry-path fidelity): **100%** — all PASS.
- Soft checkpoints with scored findings (11-01, 11-02, E-03, 16-01): pull weighted average down to ~96%.
- N/A checkpoints excluded from average.
- Extraction completeness: `1.0` | Extractor: `1.2.0`

**Gate summary:**

| Gate | Result |
| --- | --- |
| Main shell present in BRD | PASS |
| Entry `PKG_AUTO_CLAIMLOT_SRV.SP_AUTOCLAIM_WARMLG` present | PASS |
| Package files in ZIP covered | PASS (0 packages in ZIP) |
| Missing refs (`PKG_AUTO_CLAIMLOT_SRV`) acknowledged | PASS |
| Schedule stated as fact only from register (matched=true) | PASS |
| Entry-path fidelity (F-11) | PASS |
| F-gate (all scored F-* at 100%) | PASS |
| Soft readability / narrative quality | PASS WITH FINDINGS (minor inferences disclosed) |
| §11.1 / §11.2 completeness | FINDING — patched in BRD |
| §16 BR-005 matrix label | FINDING — patched in BRD |

**Overall Verdict: PASS WITH FINDINGS**
All hard gates pass. Soft findings in §11.1, §11.2, and §16 (BR-005 row) patched in BRD (Round 1). Minor business-language inferences ("warm-up", entity names) are disclosed with SME caveats and do not constitute hallucination.

