---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-25
status: pending
priority: high
due: 2026-04-27
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/April 29 Pipeline Code Fail — Decision Tree.md"
---

# Task: Pipeline Code Fail — Decision Tree

## Context

The April 29 Pipeline Validation Spec (filed 2026-04-25) includes a 5-item code review checklist Presten can complete in ~15 minutes. The spec identifies two failure modes: (1) event classifier doesn't know about `ga_aspire` → first May 1 run reverts GA ASPIRE fix; (2) tgs_scraper hasn't been updated → new ECNL games never set `ecnl_verified = TRUE`.

The spec defines what to check. It does not define what to do if the check fails.

If Presten runs the code review on April 27 and finds a FAIL, there is currently no decision guide. Without one, SENTINEL is forced to improvise on April 27 or April 28 under time pressure, with April 28 execution already scheduled. That is a bad decision environment.

This document provides the decision tree before it is needed.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/April 29 Pipeline Code Fail — Decision Tree.md`

This is a conditional decision document. Two sections: one per failure mode from the Pipeline Validation Spec. Each section has three decision paths with specific conditions for each.

---

### Failure Mode 1: Event Classifier — `ga_aspire` not recognized

Reference: Section 2, Check A of the Pipeline Validation Spec.

**Description:** The event classifier uses `event_tier` values to assign calibration multipliers. If `ga_aspire` is not in the classifier's event_tier map, new games with `event_tier = 'ga_aspire'` (written by April 28 UPDATE) will either throw a classification error or default back to `ga` — silently undoing the April 28 fix on first pipeline run.

Decision paths based on what Presten finds in the code:

| Path | Condition | Recommendation | Impact on April 28 |
|------|-----------|----------------|-------------------|
| A — Safe | `ga_aspire` is in the classifier map with correct cal value | Proceed with April 28 as planned. May 1 is safe. | No change |
| B — Fixable | `ga_aspire` is missing from the map; code change is simple (add one entry to a lookup dict/table) | Do NOT proceed with April 28 DB fix until code is committed. Delay April 28 OR scope the code fix first: (1) Presten adds `ga_aspire` to classifier, (2) SENTINEL confirms, (3) April 28 runs. Adds 1–2 hours. | Delay April 28 by the code fix duration |
| C — Complex | `ga_aspire` handling requires structural pipeline change (not a one-line addition) | April 28 proceeds on the DB only. Document as: "April 28 applies the DB fix; pipeline code fix is a separate session before May 1." FORGE scopes the code change and flags to SENTINEL. May 1 deployment is conditional on code fix confirmed complete. | April 28 runs, May 1 conditional |

**SENTINEL decision rule:** SENTINEL authorizes Path A automatically. Paths B and C require SENTINEL review before April 28 proceeds. FORGE flags Path B or C in briefing immediately.

---

### Failure Mode 2: TGS Scraper — `ecnl_verified` not written

Reference: Section 2, Check B of the Pipeline Validation Spec.

**Description:** The tgs_scraper ingests ECNL games from TGS. After the April 28 ALTER TABLE adds `ecnl_verified`, the scraper must write `ecnl_verified = TRUE` for ECNL games. If the scraper hasn't been updated, new ECNL games after May 1 will have `ecnl_verified = NULL` or `FALSE`, degrading the June 2 migration baseline.

Decision paths:

| Path | Condition | Recommendation | Impact on April 28 |
|------|-----------|----------------|-------------------|
| A — Safe | tgs_scraper already writes `ecnl_verified = TRUE` for ECNL games | Proceed. June 2 baseline will populate correctly from May 1 onward. | No change |
| B — Missing, simple | Scraper doesn't write `ecnl_verified`; adding `ecnl_verified = TRUE` to the ECNL insert is straightforward | This is lower urgency than Failure Mode 1 (June 2 deadline, not May 1). April 28 can proceed. Code fix should be deployed before May 1. FORGE scopes the change and adds to the May 1 Pre-Auth Checklist as a required condition. | April 28 proceeds; May 1 Pre-Auth Checklist gains one condition |
| C — Complex | Scraper architecture prevents a simple field addition | April 28 proceeds. FORGE files a separate task scoping the scraper change. SENTINEL adds `ecnl_verified scraper update confirmed` as a hard May 1 Pre-Auth condition. Stage 2 authorization is not granted until this is resolved. | April 28 proceeds; Stage 2 gated |

**SENTINEL decision rule:** Path A is approved automatically. Path B: FORGE adds condition to May 1 Pre-Auth Checklist. Path C: SENTINEL holds Stage 2 authorization until resolved. Either way, April 28 proceeds.

---

### Combined Scenario: Both Checks Fail

If both failure modes return FAIL:

- April 28 proceeds only if Failure Mode 1 is Path B (fixable before April 28) or Path C (code fix deferred to before May 1).
- If Failure Mode 1 is Path B but code fix cannot be completed before April 28, SENTINEL recommends delaying April 28 by one day to allow the code fix to be confirmed first. The DB fix without the code fix creates a window where the April 28 improvement is immediately overwritten by the first pipeline run.
- FORGE escalates to SENTINEL within 2 hours of discovering a combined fail scenario.

---

### How to Use This Document

1. Presten completes the 5-item code review checklist (Appendix of `Infrastructure/April 29 Pipeline Validation Spec.md`)
2. For each check: PASS → no action. FAIL → refer to the relevant section above.
3. FORGE records the path letter (A/B/C) in its next briefing.
4. SENTINEL reviews briefing and confirms the path taken.
5. If Path B or C for Failure Mode 1: SENTINEL confirmation required before April 28 runs.

---

## Quality Standard

- Each decision path must have explicit conditions — not "if the fix is simple." "Simple" means: added as a single value to an existing lookup structure in ≤ 5 lines of code.
- SENTINEL's decision rule for each path must be stated explicitly. SENTINEL does not re-derive the decision from the description.
- The combined scenario section must recommend a specific action, not "consult SENTINEL." SENTINEL consultation is a last resort, not a first step.

## Why This Matters

The Pipeline Validation Spec is a detection document: it finds problems. This document is the response document: it defines what to do when problems are found. Without it, a FAIL result on April 27 triggers a real-time improvised decision under time pressure. With it, SENTINEL's response is a 2-minute reference check. The cost of writing this now is low; the cost of not having it on April 27 evening is high.

## References

- `Infrastructure/April 29 Pipeline Validation Spec.md` — the checklist this decision tree responds to
- `Infrastructure/April 28 Reference Card.md` — the April 28 session this tree may affect
- `Infrastructure/May 1 Deployment Pre-Authorization Checklist.md` — Path B/C outputs may add conditions here
- `Infrastructure/Stage 2 Expansion — SENTINEL Authorization Gate.md` — Path C may add conditions here
