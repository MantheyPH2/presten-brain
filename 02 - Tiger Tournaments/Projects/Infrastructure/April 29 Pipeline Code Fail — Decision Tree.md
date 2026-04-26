---
title: April 29 Pipeline Code Fail — Decision Tree
tags: [infrastructure, pipeline, decision, ga-aspire, ecnl, deployment, evo-draw, forge]
created: 2026-04-25
updated: 2026-04-25
author: FORGE
status: active — reference before April 28 code review
task: task-2026-04-25-pipeline-code-fail-decision-tree
---

# April 29 Pipeline Code Fail — Decision Tree

> **Purpose:** Presten runs the 5-item code review checklist in the Appendix of `Infrastructure/April 29 Pipeline Validation Spec.md`. If any item returns FAIL, this document defines what to do next. No improvisation under time pressure.

> **When to use:** After the checklist is completed on April 27. For each FAIL result, go directly to the relevant section below. Record the path letter (A/B/C) in a message to FORGE.

---

## How to Use This Document

1. Presten completes the 5-item checklist in `Infrastructure/April 29 Pipeline Validation Spec.md` Appendix.
2. For each item: PASS → no action needed. FAIL → find the relevant section below.
3. Note the path letter (A/B/C) for each FAIL and send to FORGE.
4. FORGE records the path in its next briefing.
5. **If Failure Mode 1 returns Path B or C:** SENTINEL must confirm before April 28 runs. Do not start April 28 DB session until confirmed.

---

## Failure Mode 1: Event Classifier — `ga_aspire` Not Recognized

**What it means:** The pipeline's event classifier determines `event_tier` for every new game ingested. If `ga_aspire` is absent from the classifier's lookup map, new games from GA ASPIRE tournaments will be written as `event_tier = 'ga'` — silently undoing the April 28 DB UPDATE on the first May 1 pipeline run.

**Check:** Search for `ga_aspire` in the codebase. If not found → FAIL. If found → confirm it has the correct calibration value.

| Path | Condition | Action | Impact on April 28 |
|------|-----------|--------|--------------------|
| **A — Safe** | `ga_aspire` is present in the classifier map with the correct calibration value | Proceed with April 28 as planned. May 1 pipeline run is safe. | No change |
| **B — Fixable** | `ga_aspire` is absent; adding it is a single entry in an existing lookup dict/table (≤ 5 lines of code) | **Do NOT start April 28 DB session until this code change is committed and confirmed.** Sequence: (1) Presten adds `ga_aspire` entry to classifier, (2) FORGE or Presten confirms the change is present, (3) April 28 DB session runs. Adds ~1–2 hours to the April 28 prep timeline. | Delay April 28 start by the code fix duration |
| **C — Structural** | `ga_aspire` handling requires changes beyond a single lookup entry (e.g., classifier architecture, new pipeline stage, conditional branching) | April 28 DB session proceeds on schedule — the DB fix is still correct and safe. Code fix is scoped as a separate task before May 1. FORGE files a task immediately on learning of this outcome and flags it to SENTINEL. May 1 deployment is CONDITIONAL: pipeline launches only after the code fix is confirmed deployed. | April 28 runs; May 1 is conditional on code fix |

**SENTINEL authorization rule:**
- Path A → auto-approved. No SENTINEL action needed.
- Path B → SENTINEL must confirm before April 28 DB session starts. FORGE escalates within 2 hours of learning Path B.
- Path C → SENTINEL must confirm before May 1 deployment. April 28 proceeds. FORGE opens a new task immediately.

---

## Failure Mode 2: TGS Scraper — `ecnl_verified` Not Written

**What it means:** After the April 28 `ALTER TABLE` adds the `ecnl_verified` column and the Step 2b backfill marks historical ECNL games `TRUE`, every new ECNL game ingested after April 28 must also be written as `ecnl_verified = TRUE`. If the `tgs_scraper` has not been updated to write this field, new ECNL games accumulate as `ecnl_verified = FALSE`, progressively degrading the June 2 migration verification baseline.

**Check:** Search for `ecnl_verified` in `tgs_scraper`. If found as `ecnl_verified: true` for ECNL games → PASS. If absent or set to `FALSE` → FAIL.

| Path | Condition | Action | Impact on April 28 |
|------|-----------|--------|--------------------|
| **A — Safe** | `tgs_scraper` already writes `ecnl_verified = TRUE` for games where `source IN ('tgs', 'tgs_athleteone') AND event_tier = 'ecnl'` | Proceed. June 2 baseline will populate correctly from May 1 onward. | No change |
| **B — Missing, simple** | `tgs_scraper` does not write `ecnl_verified`; adding `ecnl_verified: true` to the ECNL insert is a ≤ 5-line change | April 28 proceeds on schedule. This is a May 1 blocker (June 2 deadline, not April 28). FORGE adds one condition to `Infrastructure/May 1 Deployment Pre-Authorization Checklist.md`: *"`ecnl_verified` scraper write confirmed deployed."* Code fix must be in place before May 1 pipeline launch. | April 28 proceeds; May 1 Pre-Auth Checklist gains one condition |
| **C — Structural** | Scraper architecture prevents a simple field addition (e.g., INSERT payload is generated dynamically without direct field mapping) | April 28 proceeds. FORGE files a separate task scoping the scraper change immediately. SENTINEL adds *"`ecnl_verified` scraper update confirmed"* as a hard May 1 Pre-Auth condition AND as a Stage 2 authorization prerequisite. Stage 2 cannot be authorized until this is resolved. | April 28 proceeds; Stage 2 gated on this fix |

**SENTINEL authorization rule:**
- Path A → auto-approved.
- Path B → no SENTINEL action needed for April 28. FORGE updates May 1 Pre-Auth Checklist; SENTINEL reviews at May 1 gate.
- Path C → SENTINEL holds Stage 2 authorization until the scraper fix is confirmed deployed. FORGE tasks this immediately.

---

## Combined Scenario: Both Checks FAIL

If both Failure Mode 1 AND Failure Mode 2 return FAIL:

| Failure Mode 1 Result | Failure Mode 2 Result | April 28 Decision |
|-----------------------|-----------------------|-------------------|
| Path A | Any | April 28 proceeds |
| Path B | Any | April 28 delayed until Failure Mode 1 code fix is committed. Failure Mode 2 path handled per its own rules above. |
| Path C | Path A or B | April 28 proceeds. Both code fixes tracked as separate pre-May-1 tasks. |
| Path C | Path C | April 28 proceeds. Both are separate pre-May-1 / pre-Stage-2 conditions. FORGE escalates both to SENTINEL within 2 hours. |

**Rule for Path B + any combined failure:** If Failure Mode 1 is Path B (classifier fix required before April 28) and the code fix cannot be confirmed before April 28's scheduled start time, SENTINEL recommends delaying April 28 by one day. The April 28 DB UPDATE creates a window during which the DB has corrected `event_tier` values that the next pipeline run will immediately overwrite. A one-day delay costs little relative to avoiding a silent reversion.

**Escalation protocol for combined failures:** FORGE notifies SENTINEL within 2 hours of discovering any combined FAIL scenario. Do not wait for the next briefing cycle.

---

## Quick Reference Card

| Result | Failure Mode 1 Path | Failure Mode 2 Path | April 28 Status | May 1 Status |
|--------|--------------------|--------------------|-----------------|--------------|
| Both PASS | — | — | GO | GO |
| FM1 FAIL, Path A | A (it passed) | — | GO | GO |
| FM1 FAIL, Path B | B | — | HOLD until fix committed | GO after fix |
| FM1 FAIL, Path C | C | — | GO | CONDITIONAL |
| FM2 FAIL, Path B | — | B | GO | CONDITIONAL |
| FM2 FAIL, Path C | — | C | GO | CONDITIONAL + Stage 2 gated |
| Both FAIL, FM1=B | B | B or C | HOLD until FM1 fix | CONDITIONAL |

---

## References

- `Infrastructure/April 29 Pipeline Validation Spec.md` — the 5-item checklist this tree responds to; Sections 1 and 2 define the checks in detail
- `Infrastructure/April 28 Reference Card.md` — April 28 session affected by Path B delays
- `Infrastructure/May 1 Deployment Pre-Authorization Checklist.md` — Path B/C outputs add conditions here
- `Infrastructure/Stage 2 Expansion — SENTINEL Authorization Gate.md` — Failure Mode 2 Path C adds a gate condition here

*FORGE — 2026-04-25*
