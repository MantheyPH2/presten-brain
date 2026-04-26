---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-26
status: pending
priority: urgent
due: 2026-04-28
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/GA ASPIRE Pipeline Classifier Audit — April 2026.md"
---

# Task: GA ASPIRE Pipeline Classifier Audit

## Context

The April 28 DB session applies a one-time SQL UPDATE: `UPDATE games SET event_tier = 'ga_aspire' WHERE ...`. This fixes all historical games currently in the DB. But it is a one-time fix.

The daily pipeline (deploying May 1) ingests NEW games every day. If the GotSport crawler's event classification logic still maps GA ASPIRE event names to `event_tier = 'ga'`, then every new GA ASPIRE game ingested after April 28 will land with `event_tier = 'ga'`. Within days, the GA ASPIRE fix will begin to erode as new games accumulate at the wrong tier. Within two weeks, if there is active GA ASPIRE game play in late April/early May, the calibration distortion SENTINEL just fixed will partially return.

This is a forward-looking code audit. The April 28 Execution Log and Reference Card cover the DB fix. This document audits whether the pipeline will sustain the fix going forward.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/GA ASPIRE Pipeline Classifier Audit — April 2026.md`

---

### Section 1: Event Classification Logic Location

- Where in the pipeline code does `event_tier` get assigned to a new game?
- Which file(s) contain the event classification logic? (e.g., `crawlers/gotsport/classify-event.js` or equivalent)
- Is classification done at ingest time (when game is first written) or post-ingest (via a separate classification pass)?

---

### Section 2: GA ASPIRE Pattern Coverage

Review the classification logic for how it distinguishes GA ASPIRE from GA:

- Does the classifier have a separate rule for GA ASPIRE events? (Check for string patterns like `'GA ASPIRE'`, `'GA-ASPIRE'`, `'ASPIRE'` in event name matching)
- If yes: show the exact rule and confirm it assigns `event_tier = 'ga_aspire'`
- If no: the fix is incomplete. Document the gap and specify what pattern addition is needed

---

### Section 3: Audit Result — One of Three Outcomes

**Outcome A: Classifier correctly handles GA ASPIRE**
- Quote the specific rule that assigns `ga_aspire`
- Confirm the classification runs before games are written to DB
- No action needed. Note: SENTINEL clears this risk.

**Outcome B: Classifier assigns all GA-branded events to `event_tier = 'ga'`**
- This is the gap. Document the exact code path.
- Propose the minimum code change: a pattern match added before the generic GA fallback
- Estimate the impact: if no GA ASPIRE games have been played since last DB session, impact is zero now but grows daily after May 1
- SENTINEL will use this to authorize a code fix before pipeline deployment

**Outcome C: Classifier is unknown / code not accessible**
- If FORGE cannot access or locate the classifier code, flag immediately in this document
- Do not leave SENTINEL without an answer — at minimum write: "Classifier code not found at expected path. SENTINEL must request Presten confirm event_tier assignment behavior before May 1 deployment."

---

### Section 4: ecnl_verified Parallel Check

While auditing the classifier, check whether the ingestion write path sets `ecnl_verified` for new games (separate from the Step 2b backfill). Note: this is also covered by `task-2026-04-26-ecnl-verified-ingestion-path-audit.md`. If you complete both audits in one code review session, combine findings into this document's Section 4 and note in the other task that the results are here.

---

## Quality Standard

- Section 3 must have a single clear verdict (A, B, or C). Do not end in "may be an issue."
- If the outcome is B: the code change must be specific enough for Presten to implement in one terminal session.
- This document must be filed BEFORE April 28. If the classifier is broken (Outcome B), SENTINEL needs to flag this to Presten before the pipeline deploys May 1.

## Why This Matters

The April 28 DB fix without a corresponding pipeline fix is a temporary fix. New GA ASPIRE games are actively being played in the spring season. If GA ASPIRE game classification is wrong in the pipeline, ELO's calibration stability monitoring will detect unexplained post-fix rating drift on Girls GA teams — exactly the symptom the fix was intended to eliminate. Identifying this now costs 30 minutes of code review; discovering it on May 5 or May 9 costs the DSS readiness timeline.

## References

- `Infrastructure/April 28 Reference Card.md` — the DB fix this audit complements
- `Infrastructure/April 28 Execution Log.md` — the session this audit's findings should be appended to (Section: Pre-condition notes)
- `task-2026-04-26-ecnl-verified-ingestion-path-audit.md` — parallel audit task for ecnl_verified
- `Rankings/Post-Fix Calibration Stability Monitoring Spec.md` — the monitoring that will detect this failure if it isn't caught now
