# BRD Validation Report: PKG_SRV_INCENTIVE_SCH_C (Cron / Batch)

**Verdict:** PASS WITH FINDINGS  —  Overall fidelity: 97%

## Correction Loop Log

| Round | Verdict before | Findings addressed (IDs) | Verdict after |
| --- | --- | --- | --- |
| 0 | — | Initial agent review; no prior round | PASS WITH FINDINGS |
| 1 | PASS WITH FINDINGS | Hard gates evaluated — all 7 hard gates PASS (100%). SF-01–SF-07 are soft/cosmetic (extractor artefacts, readability, generic test idea). No hard FAIL detected → no BRD patch performed. | PASS WITH FINDINGS (no patch required) |

**Purpose:** Validate the Cron BRD against the ZIP / JSON-LD / `_cron_metrics.json`. Counts from metrics only — never BRD prose.

---

## Review Metadata

| Field | Value |
| --- | --- |
| Job / Script Name | PKG_SRV_INCENTIVE_SCH_C |
| BRD File / Version | pkg_srv_incentive_sch_c_BRD.md v1.0 |
| Source Folder Reviewed | in/ (ZIP members: PKG_SRV_INCENTIVE_SCH_C.sh, PKG_SRV_BS_INCENTIVE_SCH.pck, dms_cron_profile, dependencies.json, PKG_SRV_INCENTIVE_SCH_C_dependencies.json) |

---

## Inventory & Count Reconciliation

"Count (Source Truth)" is from `_cron_metrics.json` / Cron JSON-LD. "Count in BRD" is derived from named presence and section coverage. Match = Yes only when they agree.

| Element Category | Count (Source Truth) | Count in BRD | Match | Severity | Comments |
| --- | --- | --- | --- | --- | --- |
| Main shell scripts (.sh/.ksh/.bash) | 1 | 1 | Yes | — | `PKG_SRV_INCENTIVE_SCH_C.sh` — §4.1, §9.2 |
| Sourced scripts (source / `.`) | 1 | 1 | Yes | — | `/dmsdb/cron/dms_cron_profile` — §4.5, §3 |
| Invoked shell scripts | 0 | 0 | Yes | — | none |
| SQL*Plus script files (.sql/.pls) | 0 | 0 | Yes | — | none |
| SQL heredoc / inline SQL blocks | 1 | 1 | Yes | — | heredoc_1 in shell; BRD §3 / §11.3 |
| Package files (.pck) | 1 | 1 | Yes | — | `PKG_SRV_BS_INCENTIVE_SCH.pck` — §9.2, §11 |
| Standalone procedures (.prc) | 0 | 0 | Yes | — | none |
| Standalone functions (.fnc) | 0 | 0 | Yes | — | none |
| Entry EXEC/CALL routines (L1) | 1 | 1 | Yes | — | `PKG_SRV_BS_INCENTIVE_SCH.SP_GEN_BS_INCENTIVE` — metrics `procedure_function_names` primary; `PKG_SRV_INCENTIVE_SCH_C.LOG` is an extractor artefact (spool filename parsed as entry call) — see Finding SF-01 |
| Call-chain steps (SCAN/FOUND/CASCADE) | 8 | 8 | Yes | — | JSON-LD `callChain` array has 8 entries; BRD §9.1 has 8 rows |
| Tables / views (static SQL scan, entry path) | 5 | 5 | Yes | — | `AM_DEALER_LOC`, `MARKS`, `VD_SRV_INC`, `VD_SRV_INC_VIN_WISE_DATA`, `VH_RO` — §7.2, §10.1 |
| Package-sibling table references (not on entry path) | 0 | 0 | Yes | — | `package_sibling_table_references: []` in metrics |
| Off-chain ZIP units (.fnc/.pck not on call chain) | 0 | 0 | Yes | — | `off_chain_zip_units: []` in metrics |
| Sequences (static scan) | 0 | 0 | Yes | — | none seen in JSON-LD |
| Environment variables (informational) | 3 | 3 | Yes | — | `ORACLE_HOME`, `PD`, `UD` — §3, §4.5 |
| Dynamic references (non-blocking) | 1 | 1 | Yes | — | `$ORACLE_HOME` — §3 |
| Runtime executables (sqlplus, etc.) | 1 | 1 | Yes | — | `sqlplus` — §13.1 |
| Spool / log paths | 1 | 1 | Yes | — | `/home/dmsdb/logs/PKG_SRV_INCENTIVE_SCH_C.log` — §4.1, §13.3 |
| Missing / unresolved blocking refs | 0 | 0 | Yes | — | `missing_references: []`; §9.3 correctly states "None" |
| BR-xxx IDs in §5 | 6 | 6 | Yes | — | BR-001 through BR-006 |
| BRC-xxx IDs in §6 | 12 | 12 | Yes | — | BRC-001 through BRC-012 |

**Count fidelity: 21/21 categories match (100%).**

---

## Inventory Coverage in the BRD

| Inventory Category | Source Count | Documented (covered/total) | BRD Section(s) | Coverage % | Missing names / Comments |
| --- | --- | --- | --- | --- | --- |
| Main shell script | 1 | 1/1 | §4.1, §9.2 | 100% | `PKG_SRV_INCENTIVE_SCH_C.sh` |
| Sourced / profile scripts | 1 | 1/1 | §3, §4.5, §9.3 | 100% | `/dmsdb/cron/dms_cron_profile` — present in ZIP |
| Entry package.procedure | 1 | 1/1 | §4.1, §11.3 | 100% | `PKG_SRV_BS_INCENTIVE_SCH.SP_GEN_BS_INCENTIVE` |
| L2+ call-chain objects | 3 | 3/3 | §9.1, §11.2 | 100% | `PKG_SRV_BS_INCENTIVE_SCH` (CASCADE L2 + FOUND L2), `SP_GEN_BS_INCENTIVE` (FOUND L2) |
| Package files on chain | 1 | 1/1 | §9.2, §11.2 | 100% | `PKG_SRV_BS_INCENTIVE_SCH.pck` |
| Shared functions on call chain | 0 | — | §12 | N/A | No on-chain `.fnc` |
| Off-chain ZIP units (inventory only) | 0 | 0/0 | §9.4 | N/A | None in metrics |
| DB tables/views (entry-path scan) | 5 | 5/5 | §7.2, §10.1 | 100% | All 5 tables from `table_view_references` present |
| Package-sibling tables (not on entry path) | 0 | 0/0 | §7.2.1 | N/A | None in metrics; §7.2.1 correctly states N/A |
| Log / spool paths | 1 | 1/1 | §4.1, §3, §13.3 | 100% | `/home/dmsdb/logs/PKG_SRV_INCENTIVE_SCH_C.log` |
| Missing ZIP refs acknowledged | 0 | yes | §9.3, §2 Open Gaps, §15 | 100% | No missing refs; §9.3 says "None" |
| Dynamic refs acknowledged | 1 | 1/1 | §3, §9.4 | 100% | `$ORACLE_HOME` non-blocking |

**Inventory coverage: 12/12 categories assessed.**

---

| JSON-LD Graph | cron-shell-graph/PKG_SRV_INCENTIVE_SCH_C.jsonld |
| Metrics File | _cron_metrics.json (extractor v1.2.0, completeness 1.0) |
| Reviewer | BRD Agent |
| Review Date | 2026-10-17 |
| Overall Score (calculated at end) | 97% (hard/deterministic gates 100%; soft findings reduce to 97%) |
| Overall Result (PASS / PASS WITH FINDINGS / FAIL) | PASS WITH FINDINGS |

## A. Preconditions & Source Completeness

| # | Checkpoint | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ---------- | --------------------- | ----------------------------------- | -------------- |
| A-01 | Has every file in the cron ZIP / `in/` folder been inventoried before BRD review begins? | 100% | None | ZIP members verified: `PKG_SRV_INCENTIVE_SCH_C.sh`, `PKG_SRV_BS_INCENTIVE_SCH.pck`, `dms_cron_profile`, `dependencies.json`, `PKG_SRV_INCENTIVE_SCH_C_dependencies.json` |
| A-02 | Was Cron JSON-LD extraction completed and `_cron_metrics.json` produced before scoring counts? | 100% | None | Extractor v1.2.0; completeness ratio 1.0 |
| A-03 | Is the main `.sh` correctly identified as the primary artifact? | 100% | None | `PKG_SRV_INCENTIVE_SCH_C.sh` correctly identified in §1, §4, §9 |
| A-04 | Are missing blocking dependencies listed rather than silently ignored? | 100% | None | `missing_references: []` in metrics; §9.3 states "None"; `dms_cron_profile` present in ZIP and marked "Yes" in §4.5 |

---

## B. Cross-Cutting Validation Rules

| # | Checkpoint | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ---------- | --------------------- | ----------------------------------- | -------------- |
| B-01 | Is every factual statement in the BRD directly supported by shell/SQL/PLSQL source or logically derived from it? | 95% | **SF-01:** §9.1 row 3 lists `PKG_SRV_INCENTIVE_SCH_C.LOG` as "FOUND L1 — Named in the shell EXEC/CALL". This is an extractor artefact; the raw shell line `spool /home/dmsdb/logs/PKG_SRV_INCENTIVE_SCH_C.log` is a spool command, not an Oracle package.routine entry call. BRD faithfully mirrors JSON-LD callChain but the description "Named in the shell EXEC/CALL" is misleading to a reader. | Add a parenthetical note in §9.1 clarifying that `PKG_SRV_INCENTIVE_SCH_C.LOG` is the SQL*Plus spool-log filename parsed by the extractor, not an Oracle procedure entry call. No BRD patch required for hard gate. |
| B-02 | Is there no hallucinated script, package, procedure, table, parameter, or log path? | 100% | None | All names verified against JSON-LD and metrics |
| B-03 | Are commented / dead SQL or unused package members not described as active job behaviour? | 100% | None | §7.3 explicitly notes two `AM_LIST` queries are in commented-out blocks and therefore not active. |
| B-04 | Are N/A cells honest — filled when entry call / spool / package file is in source? | 100% | None | Entry call, spool, and package file all filled. Purpose columns N/A where not derivable. |
| B-06 | Are counts in the BRD consistent with `_cron_metrics.json` / JSON-LD? | 100% | None | All 21 inventory categories match. |
| B-07 | Are names consistent with exact source identifiers (files + Oracle objects)? | 95% | **SF-02:** §11.2 Catalogue row 2 labels `PKG_SRV_INCENTIVE_SCH_C` as type "package", sourced from `.sh`. The JSON-LD classifies it as `PlsqlUnit / unitKind: package` — an extractor classification of the shell entry wrapper. Readers may confuse this with an Oracle `.pck` file. | Annotate §11.2 row for `PKG_SRV_INCENTIVE_SCH_C` to clarify it is the shell-script execution module, not an Oracle package source file. |
| B-08 | Are dependency chains closed through L1→Ln for objects present in the ZIP? | 100% | None | `PKG_SRV_BS_INCENTIVE_SCH.pck` in ZIP; L1→L2 chain walks correctly. |
| B-09 | Is schedule not stated as fact unless present in source? | 100% | None | `hasExecutionSchedule.matched: true`; schedule register sentence correctly copied verbatim in §3, §4.3, BR-001. |
| B-10 | Is business language used without inventing policy or BR/BRC from filenames alone? | 100% | None | All BRCs derived from PL/SQL body logic. |
| B-13 | Are description/purpose columns plain business language where filled? | 100% | None | §2 and §3 are outcome-led; technical names in backticks. |
| B-14 | Is each requirement / rule documented under the correct BR / BRC id without unsupported duplication? | 100% | None | BR-001–BR-006 and BRC-001–BRC-012 are distinct and non-overlapping. |
| B-15 | Does the BRD contain only the standard Cron sections 1 through 16 in order? | 100% | None | All 16 sections present and in order. |
| B-16 | Are §§2–3 and §5 stakeholder-readable; technical names verbatim; no invented acronym expansions? | 90% | **SF-03:** §6 BRC Source column embeds SQL WHERE-clause fragments (e.g. `NVL(msil_terminated_date, SYSDATE) >= '1-Mar-2022'`). These are technically grounded and traceable, but the §6 table reads more like a code excerpt than a plain-English rule catalogue for a non-technical stakeholder. No invented expansions detected; `DDL`, `2S`, `3S`, `BANDP` used verbatim as source codes. | For SME review, consider paraphrasing BRC source-column fragments from SQL excerpts into one-sentence English descriptions while retaining procedure-name evidence. |

---


## 1. Document Control

| # | Checkpoint | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ---------- | --------------------- | ----------------------------------- | -------------- |
| 1-01 | Does Document Title correctly identify the cron job / script? | 100% | None | Title: "PKG_SRV_INCENTIVE_SCH_C - Business Requirements Document" — correct. |
| 1-02 | Are Document Version and Date present? | 100% | None | Version 1.0, Date "October 17, 2026" — both present. |
| 1-03 | Is Author / Reviewer / Approver marked N/A when not in source (not invented)? | 100% | None | Author listed as "BRD Agent" — a process label, not an invented stakeholder name. |
| 1-04 | Does Script name match the main shell file exactly? | 100% | None | "PKG_SRV_INCENTIVE_SCH_C.sh" — exact match to `source_file` in metrics. |
| 1-05 | Does classification/audience avoid inventing named stakeholders? | 100% | None | No named business stakeholders invented. |

---

## 2. Executive Summary

| # | Checkpoint | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ---------- | --------------------- | ----------------------------------- | -------------- |
| 2-01 | Does §2.1 state this is an as-is batch/cron specification? | 100% | None | §1.2 Revision History: "Initial as-is draft from ZIP extraction". §2 intro paragraph frames as batch job. |
| 2-02 | Is business purpose evidenced or explicitly N/A — not invented from filename? | 90% | **SF-04 (soft):** §2 purpose states "twice-monthly refresh of Bodyshop Incentive Scheme scores for zone-C dealer outlets". The zone-C scope is evidenced by the `'C'` parameter and `SUBSTR(region_cd,1,1)='C'` filter in source. "Bodyshop Incentive Scheme" is derivable from the package name and PL/SQL logic. "Twice-monthly" is from the schedule register. The purpose is reasonably evidenced; no invented policy. Minor risk: "scores for zone-C dealer outlets" implies a complete scheme context that is not fully confirmed by the SQL logic alone. | Acceptable as-is for as-is draft. SME to confirm full business purpose at review. |
| 2-03 | Is In-Scope limited to this `.sh`, entry procedure, L1–Ln packages in ZIP, and logs? | 100% | None | In-scope block correctly identifies: shell, entry routine `SP_GEN_BS_INCENTIVE`, spool log. |
| 2-04 | Is Out-of-Scope clear? | 100% | None | Out-of-scope correctly excludes sibling package members, UI, other variants, SLAs. |
| 2-05 | Are stakeholders N/A or source-grounded — no fabricated names/RACI? | 100% | None | Stakeholder table explicitly states N/A — no actor in source. |
| 2-06 | Do Open Gaps list missing profile/files and unsigned business rules? | 100% | None | Gaps note: no missing files, business rules unsigned by SME, DDL outstanding. |
| 2-07 | Does summary avoid claiming schedule/SLA/downstream consumers not in source? | 100% | None | Schedule correctly sourced; SLA and downstream consumers marked N/A. |

---

## 3. System Context

| # | Checkpoint | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ---------- | --------------------- | ----------------------------------- | -------------- |
| 3-01 | Does context narrative follow Scheduler → host → profile → sqlplus → Oracle → tables/log for evidenced components only? | 100% | None | §3 Process Flow STEP 1–5 accurately traces: scheduler → shell → source profile → sqlplus → entry exec → spool/exit. |
| 3-02 | Are systems involved limited to OS, Oracle, profile/credentials, and optional consumers? | 100% | None | §3 Systems table: OS/shell, Oracle DB, credential/profile, downstream consumers N/A. |
| 3-03 | Are runtime env vars limited to those observed in the shell? | 100% | None | `$ORACLE_HOME`, `$PD`, `$UD` — match `environment_variables` in metrics exactly. |
| 3-04 | Is log/spool path consistent with the `.sh`? | 100% | None | `/home/dmsdb/logs/PKG_SRV_INCENTIVE_SCH_C.log` — exact match to JSON-LD `logPaths[0]`. |
| 3-05 | Is profile content marked N/A until the profile file is in the ZIP? | 100% | None | Profile IS in ZIP; §3 notes "Profile contents: N/A until profile file contents are reviewed from the ZIP (presence is recorded in §4.5)" — correctly cautious. |
| 3-06 | Does System Context avoid inventing middleware / external systems not called in source? | 100% | None | No external systems invented. |

---

## 4. Job Identity and Operations

| # | Checkpoint | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ---------- | --------------------- | ----------------------------------- | -------------- |
| 4-01 | Does Batch Identity list Script, Log, Entry call, Package file with correct Source column? | 100% | None | §4.1 table: all four fields present with exact values. |
| 4-02 | Does Script name match the main shell file exactly? | 100% | None | `PKG_SRV_INCENTIVE_SCH_C.sh` — exact. |
| 4-03 | Does Log path match `spool` in source? | 100% | None | `/home/dmsdb/logs/PKG_SRV_INCENTIVE_SCH_C.log` — exact match to JSON-LD `logPaths[0]`. |
| 4-04 | Does Entry call match SQL*Plus `EXEC`/`EXECUTE`/`CALL` in the heredoc? | 100% | None | `execute PKG_SRV_BS_INCENTIVE_SCH.Sp_Gen_BS_Incentive(1,'C');` — verbatim from JSON-LD `entryCalls[0].raw`. |
| 4-05 | Is Package file the ZIP member that defines the entry package? | 100% | None | `PKG_SRV_BS_INCENTIVE_SCH.pck` — in ZIP; defines `SP_GEN_BS_INCENTIVE`. |
| 4-06 | Are Parameters taken from the call — business meaning N/A unless SME/source explains? | 100% | None | Parameters `1` and `'C'` from shell call; business meaning annotated as "SME to confirm". |
| 4-07 | Are Trigger and schedule fields N/A unless evidenced? | 100% | None | Schedule sourced from cron execution register; `matched: true`. Correctly stated as fact. |
| 4-08 | Are Success / failure / rerun marked N/A/ops unless evidenced? | 100% | None | §4.4 marks N/A with note about spool log. |
| 4-09 | Does Profile and security list sourced files and whether each is present in the ZIP? | 100% | None | §4.5 table: `/dmsdb/cron/dms_cron_profile` — "Yes | Observed from source". |
| 4-10 | Is incomplete pack (missing profile) explicitly stated when applicable? | N/A | Not applicable — profile is present in ZIP. | — |

---

## 5. Functional / Business Requirements

| # | Checkpoint | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ---------- | --------------------- | ----------------------------------- | -------------- |
| 5-01 | Are BR-IDs unique and sequential (BR-001, BR-002, …)? | 100% | None | BR-001 through BR-006 — unique and sequential. |
| 5-02 | Does BR-001 require invoking the named entry procedure with observed parameters? | 100% | None | BR-001 explicitly calls `PKG_SRV_BS_INCENTIVE_SCH.SP_GEN_BS_INCENTIVE(1,'C')`. |
| 5-03 | Is business-data BR marked SME/N/A when process name is not evidenced? | 100% | None | Actors consistently N/A. |
| 5-04 | Is spool/log BR present only when log path exists in source? | 100% | None | BR-003 covers spool log; path evidenced in JSON-LD. |
| 5-05 | Are filter/region BRs tied to actual script arguments — not invented? | 100% | None | BR-004 is tied to observed parameters `(1,'C')`. |
| 5-06 | Are shared-lookup BRs limited to on-chain functions? | 100% | None | No shared functions on chain; §12 correctly N/A. BR-005 covers category/scoring in `SP_GEN_BS_INCENTIVE`. |
| 5-07 | Is failure-visibility BR ops/N/A unless logging behaviour is coded? | 100% | None | BR-006 cites `WHEN OTHERS` handlers in `SP_GEN_BS_INCENTIVE`. |
| 5-08 | Are there no extra BR-xxx invented solely from filenames? | 100% | None | All six BRs are derived from observable shell/PL/SQL behaviour. |
| 5-09 | Is each BR testable and scoped to this one job? | 100% | None | Each BR has a test idea in §16; all scoped to this job's entry path. |

---

## 6. Business Rules Catalogue

| # | Checkpoint | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ---------- | --------------------- | ----------------------------------- | -------------- |
| 6-01 | Does every BRC row that claims a rule cite a real procedure name? | 100% | None | All 12 BRCs cite `PKG_SRV_BS_INCENTIVE_SCH.SP_GEN_BS_INCENTIVE`. |
| 6-02 | Is there no BRC invented from filenames alone? | 100% | None | Each rule maps to a specific PL/SQL construct (cursor WHERE, DECODE, IF/ELSIF, SELECT CASE). |
| 6-03 | Are rule conditions/actions faithful to PL/SQL when claimed as "from code"? | 100% | None | Verified: BRC-001 dealer type/category filter, BRC-002 C1/C2→C12 reclassification, BRC-004 RO_STATUS=40/BANDP, BRC-005 GREATEST, BRC-006 thresholds ≥500/≥100, BRC-010 mid-month cut-off — all match JSON-LD and PL/SQL logic described. |
| 6-04 | Are N/A placeholder rows clearly marked until SME confirmation? | 100% | None | No placeholder rows present; all 12 BRCs are code-evidenced. |
| 6-05 | Does the BRC table include Rule ID, description, source, related BR? | 100% | None | All four columns present for all 12 rows. |
| 6-06 | Are inactive/dead code paths not stated as live business rules? | 100% | None | Commented-out `AM_LIST` queries excluded from §6. |

---

## 7. Data Requirements

| # | Checkpoint | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ---------- | --------------------- | ----------------------------------- | -------------- |
| 7-01 | Are business entities either SME-filled or N/A — not fabricated? | 100% | None | §7.1 entities map directly to physical tables from metrics `table_view_references`. |
| 7-02 | Are read/write tables limited to static SQL scan of walked code? | 100% | None | 5 tables match `table_view_references` exactly; access modes match JSON-LD `dbObjectReferences`. |
| 7-03 | Is access mode consistent with scan evidence? | 100% | None | `AM_DEALER_LOC` READ, `MARKS` WRITE, `VD_SRV_INC` READ/WRITE, `VD_SRV_INC_VIN_WISE_DATA` READ/WRITE, `VH_RO` READ — exact match to JSON-LD. |
| 7-04 | Is it stated that this is as-is from code, not a full ERD? | 100% | None | §7.2 footer: "Labelled as-is from code on the entry-path walk. Static scan — not live ALL_DEPENDENCIES." |
| 7-05 | Is reference/config data only listed if present in walked code? | 100% | None | §7.3 notes date-range derivation from `SYSDATE/DUAL`; no external config tables on active path. |
| 7-06 | Are lifecycle / volumes / timing N/A unless evidenced? | 100% | None | §7.5: N/A — no client-supplied estimates. |
| 7-07 | Are all table names verifiable against JSON-LD `hasDbObjectReference` / metrics? | 100% | None | All 5 tables in §7.2 present in both JSON-LD `hasDbObjectReference` and metrics `table_view_references`. |
| 7-08 | Are §7.2 primary tables limited to `table_view_references` (entry path)? | 100% | None | §7.2 lists exactly the 5 entry-path tables; §7.2.1 correctly states N/A (no siblings). |
| 7-09 | Are `LV_*` / SELECT-INTO locals and call params not listed as TABLE in §7.2 / §10.1? | 100% | None | No local variables (`Lv_max_load`, `Pn_Principal`, etc.) appear in §7.2 or §10.1. |

---

## 8. Glossary

| # | Checkpoint | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ---------- | --------------------- | ----------------------------------- | -------------- |
| 8-01 | Are glossary terms limited to terms used in this job's BRD/source? | 100% | None | Six business terms and five technical terms — all used in the BRD and derivable from source. |
| 8-02 | Are technical terms (L1/L2/L3, package, profile, spool) defined correctly? | 100% | None | §8.2 defines L1/L2/L3, Package, Standalone function, Profile, Spool — all correct. |
| 8-03 | Does Parameter index list script args / entry call facts from source? | 95% | **SF-05 (soft):** §8.3 Parameter index lists `PKG_SRV_INCENTIVE_SCH_C.LOG` as an entry call. This is the spool-log filename artefact (see SF-01); it is not an entry call and should not appear in the Parameter index alongside the real entry call. | Remove `PKG_SRV_INCENTIVE_SCH_C.LOG` from §8.3 Parameter index, or annotate it clearly as the spool-log path (not an Oracle entry call). See BRD patch note. |
| 8-04 | Are glossary definitions evidenced or N/A — never inventing acronym expansions? | 100% | None | No acronym expansions invented; "DDL", "BANDP", "2S/3S" used as source codes without fabricated expansions. "Smart Eye" and "BS Load Growth" annotated "N/A — SME" where business definition is unconfirmed. |

---


## 9. Technical Inventory

| # | Checkpoint | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ---------- | --------------------- | ----------------------------------- | -------------- |
| 9-01 | Does Call chain (§9.1) include SCAN L1 → FOUND L1 → CASCADE/FOUND L2+ consistent with JSON-LD `callChain`? | 95% | BRD §9.1 faithfully mirrors all 8 JSON-LD `callChain` entries. Row 3 (`PKG_SRV_INCENTIVE_SCH_C.LOG`) is an extractor artefact — spool filename parsed as FOUND L1 entry call. The meaning column "Named in the shell EXEC/CALL" is technically what the extractor recorded, but may mislead; no actual Oracle entry call exists for this name. See SF-01. | Annotate §9.1 row for `PKG_SRV_INCENTIVE_SCH_C.LOG` as "extractor-identified spool-filename artefact — not an Oracle procedure entry call". |
| 9-02 | Are Found source files (§9.2) limited to ZIP members actually present? | 100% | None | §9.2 lists `PKG_SRV_BS_INCENTIVE_SCH.pck` and `PKG_SRV_INCENTIVE_SCH_C.sh` — both confirmed present in `in/`. |
| 9-03 | Does Missing / blocking (§9.3) list unresolved sourced scripts, packages, SQL files? | 100% | None | `missing_references: []` in metrics; §9.3 correctly says "None". Profile present in ZIP. |
| 9-04 | Does Explicitly not in inventory (§9.4) exclude sqlplus/env/tables-as-files/off-chain members correctly? | 100% | None | §9.4 excludes runtime executables, env vars as files, DB tables as files, and off-chain package members. |
| 9-05 | Excel / JSON pointer present — N/A unless client requires. | N/A | Optional unscored. §9.5 references companion artefacts. | — |
| 9-06 | Are call-chain object names exact matches to entry calls and resolved units? | 95% | SF-01 applies — `PKG_SRV_INCENTIVE_SCH_C.LOG` is listed but is not an Oracle entry call. All other names exact. | See SF-01. |
| 9-07 | Are packages present in ZIP but off-chain not claimed as part of this job's runtime path? | 100% | None | `off_chain_zip_units: []`; no such packages exist. §9.4 states this correctly. |

---

## 10. Referenced Database Objects

| # | Checkpoint | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ---------- | --------------------- | ----------------------------------- | -------------- |
| 10-01 | Are tables/views limited to static scan of walked shell/SQL/PLSQL? | 100% | None | 5 tables; all from JSON-LD `hasDbObjectReference` / metrics `table_view_references`. |
| 10-02 | Is there no fabricated table/view? | 100% | None | No extra tables appear in §10.1 beyond the 5 in metrics. |
| 10-03 | Are sequences/synonyms only listed if seen in walked code? | 100% | None | §10.2 notes sequences only if seen; none found. |
| 10-04 | Is it stated this is a static scan, not live `ALL_DEPENDENCIES`? | 100% | None | §10.2 note: "Static scan of supplied sources on the entry path …not a live ALL_DEPENDENCIES dump." |
| 10-05 | Do entry-path table counts reconcile to `_cron_metrics.json` `table_view_references`? | 100% | None | 5/5 match. |
| 10-06 | Are sibling objects only under §10.1.2? | 100% | None | `package_sibling_table_references: []`; §10 notes no sibling references. |
| 10-07 | Are `LV_*` / param false-positives omitted (not typed as TABLE)? | 100% | None | No locals or params appear as TABLE entries in §10.1. |

---

## 11. Backend Packages and Procedures

| # | Checkpoint | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ---------- | --------------------- | ----------------------------------- | -------------- |
| 11-01 | Does Main flow index show shell → PKG.SP → L2 → L3 in resolved order? | 95% | **SF-02 applies:** §11.1 main flow index includes `PKG_SRV_INCENTIVE_SCH_C (PKG_SRV_INCENTIVE_SCH_C.sh)` as an L1 package — this is the shell script module, not an Oracle package. Chain is correct but the label "package" for a `.sh` file is an extractor classification artefact. | Annotate in §11.1 that `PKG_SRV_INCENTIVE_SCH_C` is the shell-script execution unit (not an Oracle `.pck` package file). |
| 11-02 | Does Catalogue list name, file, type, parent, role for each unit? | 100% | None | §11.2 table: all five columns present for all 4 units. |
| 11-03 | Are found_from / parent relationships consistent with cascade? | 100% | None | Parents match JSON-LD `foundFrom` fields exactly. |
| 11-04 | Is entry SP specification signature/parameters aligned with the shell EXEC call? | 100% | None | §11.3: qualified name, raw call, and high-level steps match JSON-LD and shell heredoc exactly. |
| 11-05 | Are high-level steps N/A or source-derived — not a full source dump? | 100% | None | §11.3 eight high-level steps are summarised in business language; no raw PL/SQL pasted. |
| 11-06 | Are missing package files marked missing rather than described as present? | 100% | None | No missing packages; all present. |
| 11-07 | Are off-chain ZIP packages labeled as inventory-only / not on this job's chain? | N/A | No off-chain packages in this ZIP. | — |

---

## 12. Shared Functions

| # | Checkpoint | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ---------- | --------------------- | ----------------------------------- | -------------- |
| 12-01 | Are shared functions limited to those used by this job's call chain? | N/A | No on-chain `.fnc` files in ZIP. §12 correctly states N/A. | — |
| 12-02 | Are file and level correct when `.fnc` is in ZIP? | N/A | Not applicable. | — |
| 12-03 | Does the section avoid pasting full function source? | 100% | None | §12 entry is a single-row N/A placeholder; no source pasted. |
| 12-04 | Are SF_* names exact matches to source? | N/A | Not applicable. | — |
| 12-05 | Are units in `off_chain_zip_units` excluded from §12 active helpers and from BR-005? | 100% | None | `off_chain_zip_units: []`; §12 lists no helpers. BR-005 does not reference shared functions. |

---

## 13. Interfaces

| # | Checkpoint | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ---------- | --------------------- | ----------------------------------- | -------------- |
| 13-01 | Does Database interface state Oracle via sqlplus (or evidenced executable) only? | 100% | None | §13.1: "This job talks to Oracle via `sqlplus` as observed in source." |
| 13-02 | Are external systems listed only if the SP/shell evidences them? | 100% | None | §13.2: N/A — no external systems. |
| 13-03 | Is output contract log path accurate and labeled operational (not a user report)? | 100% | None | §13.3: log path `/home/dmsdb/logs/PKG_SRV_INCENTIVE_SCH_C.log` labeled "operational, not a user report". |
| 13-04 | Are downstream table consumers N/A/SME unless evidenced? | 100% | None | §13.3: "SME confirms downstream consumers." |
| 13-05 | Does §13.3 list only entry-path WRITE / READ/WRITE outputs — not sibling-only tables and not READ-only masters? | 100% | None | §13.3 WRITE output tables: `MARKS`, `VD_SRV_INC`, `VD_SRV_INC_VIN_WISE_DATA` — exactly the WRITE/READ/WRITE tables from metrics. `AM_DEALER_LOC` and `VH_RO` (READ-only) are correctly excluded. |

---

## 14. Non-Functional Requirements

| # | Checkpoint | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ---------- | --------------------- | ----------------------------------- | -------------- |
| 14-01 | Are availability, max runtime, rerun, audit, PII marked N/A/ops unless in source? | 100% | None | All five NFR topics in §14 either cite source evidence or are marked N/A/Ops. |
| 14-02 | Are there no invented SLAs or numeric thresholds without evidence? | 100% | None | No numeric SLA thresholds invented. Availability only cites the cron register entry. |

---

## 15. Assumptions, Constraints, N/A Register

| # | Checkpoint | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ---------- | --------------------- | ----------------------------------- | -------------- |
| 15-01 | N/A register notes batch/cron BRD type (soft optional). | 100% | None | §15 item 1: "Not an Oracle Report" — correctly notes no layout/SRW/RDF parameters. |
| 15-02 | Is outstanding profile / missing files recorded when applicable? | 100% | None | §15 item 2: "Profile / sourced files outstanding — Constraint — None listed from ZIP resolve." Although the constraint row label might suggest outstanding files, the value "None listed from ZIP resolve" is correctly honest. |
| 15-03 | Is BRD status described as as-is draft until SME sign-off? | 100% | None | §15 item 3: "BRD status — Assumption — As-is technical + business draft until SME sign-off." |
| 15-04 | Is static-scan limitation explicitly constrained? | 100% | None | §15 item 6: "Static dependency scan — Constraint — Not live database dependency metadata." |
| 15-05 | Are schedule/owners listed as assumptions/N/A — not facts? | 95% | **SF-06 (soft):** §15 item 4 labels "Schedule" as "Fact — Sourced from cron execution register." This is correct (matched register entry), but the label "Fact" in the register row is unusual — the template expectation is "Assumption" or "N/A" unless the schedule is register-matched. Since it IS register-matched, marking it "Fact" is defensible. However "Sourced from cron register — verified" would be clearer. | Recommend changing §15 row 4 Classification from "Fact" to "Verified — cron register" for clarity. |

---

## 16. Traceability Matrix

| # | Checkpoint | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ---------- | --------------------- | ----------------------------------- | -------------- |
| 16-01 | Does every matrix row link BR-ID → job step / package.sp → file → test idea? | 95% | **SF-07 (soft):** BR-005 row maps to "Helpers on entry path: N/A" and §12. Since there are no shared functions on chain, the §12 reference is consistent. However the test idea "Trace helper call sites in entry procedure body" is somewhat generic — it would be more accurate to say "Verify category/marks computation in `SP_GEN_BS_INCENTIVE` body; confirm `VD_SRV_INC` update with `DLR_CATEG`, `PARAM_VALUE5`–`PARAM_VALUE10`." | Refine BR-005 test idea to reference the specific update columns in `VD_SRV_INC`. |
| 16-02 | Do BR-IDs in the matrix exist in §5? | 100% | None | BR-001–BR-006 in §16 all present in §5. |
| 16-03 | Do package.sp / file references exist in source or ZIP? | 100% | None | All package.sp and file references in §16 correspond to JSON-LD entries. |
| 16-04 | Are test ideas actionable and not claiming untestable invented behaviour? | 100% | None | All 6 test ideas are actionable: file run, spool check, table write verification, parameter match, error log check. |

---


## C. Logging & Runtime Message Sweep (Cron)

| # | Checkpoint | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ---------- | --------------------- | ----------------------------------- | -------------- |
| C-01 | Are spool/log paths identified from source when present? | 100% | None | `/home/dmsdb/logs/PKG_SRV_INCENTIVE_SCH_C.log` — identified in §4.1, §3, §13.3, BR-003. |
| C-02 | Are SQL*Plus / DBMS_OUTPUT messages claimed in BRD only if present in walked code? | 100% | None | BR-001 and BR-003 cite `SET SERVEROUTPUT ON`, `SET TIME ON TIMING ON`, and `to_char(sysdate,...)` — all in shell heredoc. |
| C-03 | Is there no fabricated success/error message text? | 100% | None | No specific error/success message text invented. WHEN OTHERS handler behaviour stated generically and correctly. |
| C-04 | Are scheduler alert emails/SMS claimed only if evidenced? | 100% | None | No scheduler alerts claimed. §4.4 N/A. |

---

## D. Active vs. Inactive / On-Chain Integrity

| # | Checkpoint | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ---------- | --------------------- | ----------------------------------- | -------------- |
| D-01 | Are procedures in a `.pck` not on this job's chain excluded from active job narrative? | 100% | None | §9.4 and §2 Out-of-Scope explicitly exclude non-called package members. |
| D-02 | Is commented-out SQL/PLSQL not described as live behaviour? | 100% | None | §7.3 explicitly states the two `AM_LIST` queries are in commented-out blocks and are not active requirements. |
| D-03 | Are unresolved dynamic refs not treated as resolved file dependencies? | 100% | None | `$ORACLE_HOME` listed as dynamic/non-blocking in §3 and §9.4; not treated as a resolved file. |
| D-04 | Are missing blocking files explicitly classified as gaps (not silently active)? | 100% | None | `missing_references: []`; no blocking files missing. §9.3 "None". |

---

## E. Traceability & Consistency

| # | Checkpoint | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ---------- | --------------------- | ----------------------------------- | -------------- |
| E-01 | Do all BR-IDs used in §3/§6/§16 exist in §5? | 100% | None | BR-001 through BR-006 referenced in §3 STEP annotations and §16 matrix all exist in §5. BRC Related-BR fields (BR-002, BR-005) exist in §5. |
| E-02 | Do all BRC rows reference valid BR-IDs when claimed? | 100% | None | All 12 BRC rows cite BR-002 or BR-005; both present in §5. |
| E-03 | Do §11 catalogue entries match §9 call chain for on-chain objects? | 100% | None | §11.2 four catalogue rows align with §9.1 call chain resolved objects. |
| E-04 | Are inventory counts internally consistent across §9–§12 and metrics? | 100% | None | 1 package file, 8 call-chain steps, 5 tables, 0 shared functions — consistent across §9, §10, §11, §12, and metrics. |
| E-05 | Does §4 entry call match §11.3 entry SP? | 100% | None | §4.1 "execute PKG_SRV_BS_INCENTIVE_SCH.Sp_Gen_BS_Incentive(1,'C');" matches §11.3 Raw call exactly. |
| E-06 | Does §13 log path match §4 log path? | 100% | None | Both sections show `/home/dmsdb/logs/PKG_SRV_INCENTIVE_SCH_C.log`. |

---

## F. Hallucination / Unsupported Content Gate

| # | Checkpoint | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ---------- | --------------------- | ----------------------------------- | -------------- |
| F-01 | No invented shell scripts or profile files claimed as present when missing from ZIP? | 100% | None | All files in §4.5 and §9.2 confirmed present in `in/`. |
| F-02 | No invented tables or views? | 100% | None | 5 tables in §7.2/§10.1 — all in `table_view_references`. |
| F-03 | No invented packages or procedures? | 100% | None | All package/procedure names in §11 match `procedure_function_names` in metrics. |
| F-04 | No invented log paths? | 100% | None | Single log path confirmed in JSON-LD `logPaths[0]`. |
| F-05 | No invented cron/Control-M schedules? | 100% | None | Schedule `0 1 1,16 * *` from register; `matched: true`. |
| F-06 | No invented owners, RACI, or stakeholder names? | 100% | None | Stakeholders N/A throughout. |
| F-07 | No invented business rules from filenames? | 100% | None | All BRs and BRCs derived from observable shell and PL/SQL behaviour, not from script name alone. |
| F-08 | No "N/A" where value is clearly present in the `.sh`? | 100% | None | Entry call, spool, and package file all populated. |
| F-09 | No external systems claimed without source evidence? | 100% | None | No external systems claimed. |
| F-10 | No claim that ZIP contains live DB dependency metadata? | 100% | None | §10.2 and §7.2 footer: "static scan…not live ALL_DEPENDENCIES". |
| F-11 | Entry-path fidelity: §7.2/§7.2.1/§12/§13.3/BR-005 align with metrics? | 100% | None | §7.2: 5 entry-path tables (matches `table_view_references`). §7.2.1: N/A (`package_sibling_table_references: []`). §12: no helpers (`off_chain_zip_units: []`). §13.3: only WRITE/READ/WRITE tables. No locals/params as TABLE. |

---

## G. Feedback-Tracker Generic Checkpoints (Cron)

| # | Checkpoint | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ---------- | --------------------- | ----------------------------------- | -------------- |
| G-01 | Are functional requirements correctly mapped to related BRC rows when both exist? | 100% | None | BR-002 and BR-005 each have multiple BRC rows; BR-001, BR-003, BR-004, BR-006 reference procedure calls and log paths (no BRC rows expected). |
| G-02 | Is document scope strictly this script + on-chain dependencies in the ZIP? | 100% | None | Scope limited to shell, entry procedure, `PKG_SRV_BS_INCENTIVE_SCH.pck`, and spool log. |
| G-03 | Are all BR-IDs unique and sequential? | 100% | None | BR-001–BR-006 sequential and unique. |
| G-04 | Are Observed-from-source vs N/A columns used honestly in §4? | 100% | None | §4.1, §4.5 "Observed from source" for all present items. |
| G-05 | Is L1 entry call covered by at least one BR? | 100% | None | BR-001 explicitly covers the entry call to `SP_GEN_BS_INCENTIVE`. |
| G-06 | Are shared functions in §12 actually referenced from the chain? | N/A | No shared functions. | — |
| G-07 | Are missing ZIP members reflected in §2.5 / §9.3 / §15? | 100% | None | No missing members; §9.3 states "None"; §2 Open Gaps and §15 consistent. |
| G-08 | Is terminology consistent with source identifiers? | 95% | SF-01/SF-02 apply — `PKG_SRV_INCENTIVE_SCH_C.LOG` and "package" type for shell script are extractor artefacts. | See SF-01, SF-02 recommendations. |
| G-09 | Are static-scan tables in §10 covered or explicitly partial with justification? | 100% | None | All 5 tables from static scan listed; §10 notes partial nature (static scan only). |
| G-10 | Is §16 matrix complete for core BRs? | 95% | SF-07 applies — BR-005 test idea is generic; could be more specific. | See SF-07. |

---

## H. Cross-BRD Template & Consistency Gate

| # | Checkpoint | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ---------- | --------------------- | ----------------------------------- | -------------- |
| H-01 | Does structure match Cron 16 sections in order? | 100% | None | ToC and section headers confirm §1–§16 in order. |
| H-03 | Are cross-cutting rules (N/A discipline, no invent, entry-path fidelity) applied identically? | 100% | None | N/A discipline consistent throughout; entry-path fidelity confirmed across §7, §10, §12, §13. |

---

## I. Client Acceptance Summary

| # | Acceptance Question | Score (0–100% or N/A) | Missing / Hallucination / Incorrect | Recommendation |
| - | ------------------- | --------------------- | ----------------------------------- | -------------- |
| I-01 | Completeness: Does the BRD cover this job's shell entry path and on-chain packages/functions in the ZIP? | 100% | None | Shell, entry procedure, `PKG_SRV_BS_INCENTIVE_SCH.pck`, spool log — all covered. No on-chain `.fnc`. |
| I-02 | Correctness: Are inventories, entry call, log path, and cascade facts accurate vs source? | 97% | SF-01/SF-02 are soft accuracy issues (extractor artefacts mirrored without qualification). All hard identity facts correct. | Add qualifying notes per SF-01 and SF-02. |
| I-03 | Coverage: Are missing blocking deps and N/A ops/business items explicitly registered? | 100% | None | No missing blocking deps; N/A items registered in §2, §4, §8.4, §15. |
| I-04 | Traceability: Can each major BRD claim be traced to a ZIP member / JSON-LD node / metrics row? | 100% | None | Every table, procedure, log path, and rule entry traces directly to JSON-LD or metrics. |
| I-05 | Integrity: Are off-chain package members and dynamic refs correctly classified? | 100% | None | `off_chain_zip_units: []`; `$ORACLE_HOME` non-blocking; both handled correctly. |
| I-06 | Fit for use: Is the BRD acceptable as an as-is draft for SME/ops review? | 95% | SF-03 (§6 BRC source citations quasi-SQL), SF-05 (§8.3 parameter index includes spool artefact). Seven soft findings total; none are hard FAIL issues. | Address SF-01 through SF-07 before client presentation for a clean PASS. |

---


## Soft Findings Summary

All hard gates pass at 100%. The following soft findings were identified during agent review:

| ID | Section | Type | Description | Severity |
| --- | --- | --- | --- | --- |
| SF-01 | §9.1, §8.3 | Accuracy / misleading | `PKG_SRV_INCENTIVE_SCH_C.LOG` listed as "FOUND L1 — Named in the shell EXEC/CALL" — this is the SQL*Plus spool-log filename parsed by the extractor, not an Oracle procedure entry call. | Low |
| SF-02 | §11.1, §11.2 | Accuracy / misleading | `PKG_SRV_INCENTIVE_SCH_C` classified as type "package" (extractor artefact from JSON-LD) when it is actually the shell-script execution module, not an Oracle `.pck` package file. | Low |
| SF-03 | §6 | Readability | BRC Source column embeds SQL WHERE-clause verbatim fragments; business readers may find it quasi-SQL rather than plain-English rules. No invented content; traceability intact. | Low |
| SF-04 | §2 | Soft purpose | "Twice-monthly refresh of Bodyshop Incentive Scheme scores for zone-C dealer outlets" — mostly evidenced; "Bodyshop Incentive Scheme" is derivable but SME confirmation recommended. | Low |
| SF-05 | §8.3 | Accuracy | Parameter index includes `PKG_SRV_INCENTIVE_SCH_C.LOG` as an entry call (extractor artefact). It is the spool-log path, not a procedure call parameter. | Low |
| SF-06 | §15 | Wording | Schedule row Classification = "Fact" — technically correct since register-matched, but "Verified — cron register" would be clearer. | Cosmetic |
| SF-07 | §16 | Completeness | BR-005 test idea is generic ("Trace helper call sites in entry procedure body") — helper references do not exist; test idea should reference `VD_SRV_INC` update columns directly. | Low |

**Hard FAIL count: 0. All hard gates at 100%.**

---

## Final Sign-off

| Role | Name | Date | Status (Approved / Approved with gaps / Rejected) |
| --- | --- | --- | --- |
| Business Analyst / Reviewer | BRD Agent | 2026-10-17 | Approved with gaps (soft findings SF-01–SF-07) |
| Technical Validator | [To be assigned] | | |
| Client / Stakeholder | [To be assigned] | | |

**Checkpoints scoring below 100% (list IDs and brief reason):**
- B-01 (95%): SF-01 — `PKG_SRV_INCENTIVE_SCH_C.LOG` presented as EXEC/CALL entry without clarification
- B-07 (95%): SF-02 — shell script classified as "package" type (extractor artefact, no qualification)
- B-16 (90%): SF-03 — §6 BRC source citations include SQL fragments; readability for non-technical stakeholders
- 8-03 (95%): SF-05 — spool artefact in Parameter index
- 9-01 (95%): SF-01 mirrors §9.1
- 9-06 (95%): SF-01 mirrors §9.6 call-chain name check
- 11-01 (95%): SF-02 mirrors §11.1 main flow index
- 15-05 (95%): SF-06 — cosmetic classification wording
- 16-01 (95%): SF-07 — BR-005 test idea imprecise
- G-08 (95%): SF-01/SF-02 aggregate
- G-10 (95%): SF-07 mirrors §16

**N/A Justifications (list IDs + reason):**
- 4-10: Profile is present in ZIP — no incomplete pack scenario
- 9-05: Optional/unscored pointer checkpoint
- 11-07: No off-chain ZIP packages in this job
- 12-01, 12-02, 12-04: No on-chain `.fnc` shared functions
- G-06: No shared functions on chain

**Weighted overall score: 97%**

Scoring notes:
- Hard/deterministic checkpoints (all scored at 0% or 100%): **100%**
- Soft checkpoints reduced by 7 findings averaging 95%–90%: weighted average **97%**
- N/A rows excluded from weighted average
- F-gate (hallucination/identity/entry-path): **PASS — all F-* scored 100%**
- Count reconciliation: **21/21 categories — 100%**

**Recommended minimum thresholds before client delivery:**

| Gate | Threshold | Status |
| ---- | --------- | ------ |
| Overall weighted score | above 95% | ✅ 97% |
| Count reconciliation match | 100% for required categories | ✅ 21/21 |
| F-gate (hallucination) | all scored F-* at 100% | ✅ PASS |
| Entry-path fidelity (F-11) | 100% | ✅ PASS |
| Soft readability (B-16) | agent judgment | ⚠ 90% — SF-03 recommendation pending SME review |

**Verdict: PASS WITH FINDINGS** — Address SF-01 through SF-07 (all low/cosmetic severity) before client presentation. No hard-gate FAIL. BRD requires minor annotation corrections only; no re-extraction needed.

