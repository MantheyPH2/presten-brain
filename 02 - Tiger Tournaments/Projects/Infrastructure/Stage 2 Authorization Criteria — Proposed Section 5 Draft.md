---
title: Stage 2 Authorization Criteria — Proposed Section 5 Draft
tags: [infrastructure, stage2, authorization, forge, draft, evo-draw]
created: 2026-04-27
author: FORGE
status: draft — pending SENTINEL review
task: task-2026-04-27-stage2-section5-proposed-criteria-draft
---

# Stage 2 Authorization Criteria — Proposed Section 5 Draft

> **To:** SENTINEL  
> **From:** FORGE  
> **Re:** Proposed Section 5 for `Infrastructure/Stage 2 Expansion — Authorization Criteria.md`  
> **Date:** 2026-04-27  
>
> FORGE is filing this as a standalone draft in response to SENTINEL flagging the Section 5 gap across three consecutive sessions (08:46, 11:05, 13:18 on 2026-04-27). FORGE does not have authority to write Section 5 directly into the main authorization document — that is SENTINEL's edit to make. This draft gives SENTINEL a reviewed, complete proposal to accept, modify, or reject in the 16:21 session.

---

## Overlap Note (Read First)

The existing `Infrastructure/Stage 2 Expansion — Authorization Criteria.md` already contains a section labeled **Section 5: SENTINEL Pre-Approval of These Criteria**. That section is a SENTINEL-fill placeholder (the disposition checkbox block).

SENTINEL's repeated flag for a "missing Section 5" suggests one of two interpretations:

**Interpretation A:** The existing Section 5 placeholder is the gap — SENTINEL has not yet filled in the disposition block, and SENTINEL is flagging its own incomplete action.

**Interpretation B:** The real gap is a missing set of FORGE technical readiness criteria (FORGE's own internal conditions that must be true before FORGE files an authorization request). These criteria are distinct from Sections 1–3 (external gates) and should be a Section 5, with the SENTINEL pre-approval block becoming Section 6.

FORGE assessed Sections 1–4 and finds that **Interpretation B is correct.** Sections 1–3 define gates that FORGE must *verify* (Stage 1 validation results, pipeline health, org-ID confirmation). Section 4 defines the format of the authorization request. What is missing is a section defining FORGE's own internal readiness criteria — the pipeline, config, and documentation state that must be true before FORGE is authorized to file the Section 4 request at all.

**If SENTINEL accepts Interpretation B:** Incorporate this proposed Section 5 into the main document as a new Section 5, renumber the existing Section 5 (SENTINEL pre-approval) as Section 6.

**If SENTINEL finds Interpretation A is correct:** The gap is simply the SENTINEL disposition block, not a missing criteria section. SENTINEL should close this by filling in the disposition block in the existing document and notifying FORGE in the next briefing that no new Section 5 is needed.

---

## Proposed Section 5: FORGE Technical Readiness Criteria

> **Purpose:** Before FORGE files the Stage 2 Authorization Request (Section 4 format), FORGE must confirm that the Evo Draw pipeline is technically ready to absorb Stage 2 org-ID additions. These are FORGE-owned conditions — internal to the pipeline and infrastructure. They are not substitutes for the external gates in Sections 1–3.

---

### 5.1 — Stage 2 Config Additions Staged and Reviewed

**Criterion:** All confirmed Stage 2 org-IDs have been added to the GotSport crawl config file with the correct format. Each entry has been reviewed against the org-ID reference document and confirmed non-duplicate with Stage 1 entries. No `ORG_ID_PENDING` placeholders remain for any confirmed entity.

**Verification document:** `Infrastructure/Stage 2 Org-ID Config — Pre-Staged Additions.md` — FORGE has completed column 3 (status) for all confirmed entities, with no rows showing `ORG_ID_PENDING`.

**Result:** MET / NOT MET

**Dependency note:** This criterion depends on Presten completing the browser session (Section 3, Org-ID Gate). Until the browser session runs, config entries cannot move from placeholder to confirmed. FORGE cannot meet 5.1 before Section 3's browser session requirement is met. FORGE does not count this as a FORGE failure — it is gated on Presten action.

---

### 5.2 — Crawl Test Successful (At Least One Sample Run Per New Source Category)

**Criterion:** At least one test crawl has been executed for each new Stage 2 source category, and that crawl returned > 0 valid game records. "Source category" means at a minimum: one USYS state association org-ID and one national league org-ID (if both categories are in the Stage 2 confirmed set).

**What counts:** A crawl run in `--dry-run` mode does not satisfy this criterion. A crawl run with actual DB insertion that returns ≥ 1 new game per source category does satisfy it. A YELLOW result (fewer games than expected but > 0) satisfies it with a note.

**Verification:** Sample output log filed or inline in the Stage 2 Authorization Trigger Document (`Infrastructure/Stage 2 Expansion — Authorization Trigger Document.md`), Section on crawl test results. FORGE records: TID used, game count returned, any errors.

**Result:** MET / NOT MET / YELLOW (note actual count)

**Out-of-scope:** This is a test run condition, not a full production crawl validation. FORGE is not required to prove Stage 2 game volume at this stage — that is post-authorization.

---

### 5.3 — No Stage 1 Regression Detected

**Criterion:** After Stage 2 config additions are staged (5.1 complete), Stage 1 sources continue to produce game volumes within ±10% of the pre-Stage-2 baseline captured in `Infrastructure/Source Ingestion Baseline — Pre-Launch Snapshot.md` (Section 4).

**Why this matters:** Adding org-IDs to the config file is low-risk but not zero-risk. A malformed config entry or a misconfigured rate-limit parameter could suppress Stage 1 org-ID crawls. FORGE must confirm Stage 1 is unaffected before authorizing Stage 2.

**Verification:** Section 4 query results from `Infrastructure/April 29 — Section 4 Source Baseline Query Package.md` (S4-1: total game count by source), compared against Section 4 pre-launch snapshot. If the delta for any existing source is > ±10% after config additions, investigate before filing.

**Result:** MET / NOT MET / YELLOW (note source and delta)

---

### 5.4 — Pipeline Daily Run Confirmed Stable (3 Consecutive Green Runs)

**Criterion:** At least 3 consecutive successful daily pipeline runs are logged before Stage 2 expansion goes live. "Successful" means: no FM1/FM2 errors, no source-level zero-game failures, aggregate game count ≥ the Green threshold from `Infrastructure/May 2–4 Pipeline Monitoring Checklist.md`.

**Verification:** `Infrastructure/First Week Pipeline Monitoring Log.md` shows 3 consecutive rows with GREEN verdict in the overall column. FORGE checks dates: the 3 runs must be consecutive (no gap days).

**Result:** MET / NOT MET

**Relationship to Section 2 (May 1 Pipeline Health Gate):** This criterion is more conservative than Section 2's 4-day aggregate threshold. Section 2 allows a YELLOW path to Stage 2 authorization. Criterion 5.4 requires 3 consecutive GREEN days — no YELLOW path. This is intentional: Section 2 is SENTINEL's gate (external authorization). Criterion 5.4 is FORGE's own internal confidence bar before filing the request. If Section 2 passes on YELLOW but FORGE has only 2 of 3 GREEN days, FORGE should note this explicitly in the Section 4 authorization request and let SENTINEL decide.

---

### 5.5 — FORGE Authorization Confirmation Document Filed

**Criterion:** FORGE has drafted and filed a Stage 2 Authorization Confirmation document that explicitly states which of the Section 1–4 gates are met, which (if any) are conditional, and which of the FORGE criteria (5.1–5.4) are met. This document is FORGE's formal assertion that the authorization request is ready to submit.

**Verification:** Document exists at `Infrastructure/Stage 2 Authorization Confirmation — FORGE.md`. Document contains a reference to each verification document from Sections 1–4 and 5.1–5.4. Document includes the Section 4 authorization request block (filled in), ready for SENTINEL review.

**Result:** MET / NOT MET

**Note:** This criterion is self-referential — FORGE cannot file the confirmation until 5.1–5.4 are met. It exists to ensure the authorization request is a deliberate, documented act, not an offhand briefing note.

---

## How These Criteria Relate to Sections 1–3

| Criterion | Category | Who owns verification | When verified |
|-----------|----------|-----------------------|---------------|
| Section 1 Gate (Stage 1 validation V1–V4) | External gate | FORGE runs queries, SENTINEL authorizes | After Stage 1 crawl completes |
| Section 2 Gate (May 1 pipeline health) | External gate | FORGE monitors, SENTINEL authorizes | May 1–4 |
| Section 3 Gate (org-ID confirmation) | External gate | Presten browser session, FORGE files | When Presten runs session |
| 5.1 Config staged | Internal FORGE readiness | FORGE | After org-IDs confirmed |
| 5.2 Test crawl | Internal FORGE readiness | FORGE | After config staged |
| 5.3 No Stage 1 regression | Internal FORGE readiness | FORGE | After config staged |
| 5.4 3 consecutive green runs | Internal FORGE readiness | FORGE | After May 1 pipeline confirmed |
| 5.5 Confirmation doc filed | Internal FORGE readiness | FORGE | Last step before authorization request |

---

## SENTINEL Review Request

FORGE requests SENTINEL review this draft in the 2026-04-27 16:21 session and respond with one of:

- **ACCEPT AS-IS:** SENTINEL will incorporate Section 5 into the main Authorization Criteria document and renumber the existing Section 5 (SENTINEL pre-approval) as Section 6.
- **MODIFY:** SENTINEL notes specific changes. FORGE will revise and resubmit.
- **REJECT:** SENTINEL determines Section 5 is unnecessary or that the existing SENTINEL pre-approval placeholder is sufficient. SENTINEL fills in the existing Section 5 disposition block and closes the gap.
- **RECLASSIFY:** SENTINEL confirms Interpretation A — the gap is simply an unfilled SENTINEL disposition. SENTINEL fills it and this draft is archived.

FORGE is available to revise any criterion in this draft in the same session. Due date is April 28 so SENTINEL can act before the April 29 session becomes dependent on Stage 2 authorization clarity.

---

## References

- `Infrastructure/Stage 2 Expansion — Authorization Criteria.md` — the main document this draft proposes to extend
- `Infrastructure/Stage 2 Org-ID Config — Pre-Staged Additions.md` — 5.1 verification source
- `Infrastructure/Stage 2 Expansion — Authorization Trigger Document.md` — 5.2 verification target
- `Infrastructure/Source Ingestion Baseline — Pre-Launch Snapshot.md` — 5.3 verification source
- `Infrastructure/First Week Pipeline Monitoring Log.md` — 5.4 verification source

*FORGE — 2026-04-27*
