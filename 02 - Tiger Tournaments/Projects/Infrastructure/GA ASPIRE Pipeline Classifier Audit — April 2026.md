---
title: GA ASPIRE Pipeline Classifier Audit — April 2026
tags: [infrastructure, pipeline, ga-aspire, classifier, audit, evo-draw, forge]
created: 2026-04-26
updated: 2026-04-26
author: FORGE
status: outcome-c — code access required
due: 2026-04-28
related: "[[April 28 Execution Log]]", "[[Infrastructure/April 28 Reference Card]]", "[[Pipeline Code Review Results]]"
---

# GA ASPIRE Pipeline Classifier Audit — April 2026

> **Audit Verdict: OUTCOME C — Classifier code not accessible from vault. Presten must complete code review before May 1 deployment. See Section 3.**

---

## Purpose

The April 28 DB session applies a one-time `UPDATE games SET event_tier = 'ga_aspire' WHERE ...` to fix all historical games. That fix is a snapshot. Starting May 1, the daily pipeline ingests new games every day. If the GotSport crawler's event classification logic still maps GA ASPIRE event names to `event_tier = 'ga'`, every new GA ASPIRE game ingested after April 28 will land at the wrong tier — and the April 28 fix will begin to erode.

This document audits whether the pipeline will sustain the fix going forward. It is a prerequisite for SENTINEL's May 1 deployment authorization.

---

## Section 1: Event Classification Logic Location

**FORGE Finding:** The pipeline codebase is not stored in the Obsidian vault. FORGE cannot directly inspect the event classification files. Based on vault documentation of the pipeline structure:

- Classification of `event_tier` is expected to occur in the GotSport crawler, likely at the point where raw event/game records are normalized before INSERT
- Expected file location (to verify): `crawlers/gotsport/classify-event.js` or equivalent — or an inline tier-mapping object within the main scraper module
- Classification is expected to happen **at ingest time**, not via a post-ingest pass, based on the pipeline architecture described in `Infrastructure/Data Pipeline.md`

**Required action for Presten:** Search codebase for `event_tier` assignment. The file containing the classification logic is the target. This is FM1-2 in `Infrastructure/Pipeline Code Review Results.md`.

---

## Section 2: GA ASPIRE Pattern Coverage

**FORGE Finding:** FORGE cannot confirm from the vault whether the classifier has a GA ASPIRE–specific rule.

**What Presten must check:**

1. Search codebase for `ga_aspire` (exact string). 
   - **If found:** Note the file and line. Confirm it appears in the event_tier assignment logic (not just a comment or test fixture). Confirm the condition that triggers it (e.g., `event_name.includes('GA ASPIRE')` or `event_name.match(/GA.?ASPIRE/i)`).
   - **If not found:** The pipeline has no `ga_aspire` rule. Every GA ASPIRE event will be classified as `event_tier = 'ga'`. This is a confirmed gap.

2. If `ga_aspire` is found in the classifier, confirm that:
   - The rule fires **before** the generic GA fallback (i.e., it's not shadowed)
   - The event_name patterns it matches are comprehensive — at minimum: `'GA ASPIRE'`, `'GA-ASPIRE'`, `'ASPIRE'` (when context is GA)

3. Check whether the game INSERT/upsert includes `event_tier` in an `ON CONFLICT DO UPDATE SET` clause. If yes, re-scraping an existing game would **overwrite** the April 28 fix — a separate risk.
   - This is FM1-3 in `Infrastructure/Pipeline Code Review Results.md`.

---

## Section 3: Audit Verdict

### OUTCOME C — Code Not Accessible

FORGE cannot access the pipeline codebase from the vault. The classifier code was not found at any vault path.

**SENTINEL must request Presten confirm GA ASPIRE event_tier assignment behavior before May 1 deployment.**

The mechanism for this confirmation already exists: `Infrastructure/Pipeline Code Review Results.md` — FM1 checks. Presten should complete FM1-1 through FM1-3 in that document and return results to FORGE before April 28.

**If FM1 Path = A (ga_aspire present, correctly assigned):** No code change needed. April 28 DB fix is sustained by the pipeline. SENTINEL clears this risk.

**If FM1 Path = B (ga_aspire absent, simple lookup addition):** FORGE recommends the following minimum code change:

Add a rule in the event tier classifier, before the generic `ga` fallback:

```javascript
// In event_tier classification logic — add before generic GA fallback
if (
  eventName.includes('GA ASPIRE') ||
  eventName.includes('GA-ASPIRE') ||
  /\bASPIRE\b/i.test(eventName)
) {
  return 'ga_aspire';
}
```

This addition is ≤ 5 lines. It should be committed and deployed **before May 1** — not after. If deployed after May 1, all GA ASPIRE games ingested between May 1 and the fix date will have `event_tier = 'ga'` and will require a supplemental UPDATE.

**If FM1 Path = C (structural change required):** FORGE escalates to SENTINEL within 2 hours of FM1 results being returned. SENTINEL will scope the code fix as a pre-May-1 requirement.

---

## Section 4: ecnl_verified Parallel Check

The parallel audit for `ecnl_verified` ingestion assignment is covered in `Infrastructure/ecnl_verified Ingestion Path Audit — April 2026.md`. FM2 in `Infrastructure/Pipeline Code Review Results.md` covers both audits in a single code review session.

**Recommendation:** Presten should complete FM1 and FM2 in the same session using the Pipeline Code Review Results checklist. Both audits can be done in 15–20 minutes if the codebase is accessible.

---

## Impact Assessment

If the GA ASPIRE classifier gap is not addressed before May 1:

| Timeline | Impact |
|----------|--------|
| May 1–7 | New GA ASPIRE spring games ingested at `event_tier = 'ga'`. Rating distortion is small (low early-week volume). |
| May 7–9 (DSS prep) | ELO's post-fix calibration stability monitoring detects unexpected Girls GA rating drift — exactly the symptom the April 28 fix was meant to eliminate. Root cause unclear at that point. |
| May 9 (DSS) | If distortion is significant, GA-adjacent Girls rankings shown at DSS may be partially incorrect. April 28 fix will have been partially undone by ongoing misclassification. |

**Cost of catching now:** 15–20 minutes of code review, ≤ 5 lines of code change.  
**Cost of catching May 5:** 1–2 hours of investigation + supplemental backfill UPDATE + explanation to SENTINEL.

---

## References

- `Infrastructure/April 28 Reference Card.md` — DB fix this audit complements
- `Infrastructure/April 28 Execution Log.md` — append Section 3 verdict here under "Pre-condition notes"
- `Infrastructure/Pipeline Code Review Results.md` — FM1 checklist (Presten fills)
- `Infrastructure/April 29 Pipeline Code Fail — Decision Tree.md` — routing logic for FM1 paths A/B/C
- `task-2026-04-26-ecnl-verified-ingestion-path-audit.md` — parallel audit for ecnl_verified
- `Rankings/Post-Fix Calibration Stability Monitoring Spec.md` — will detect this failure if not caught now

*FORGE — 2026-04-26*
