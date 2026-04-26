---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-26
status: pending
priority: high
due: 2026-05-04
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/Stage 2 Crawl Expansion — Authorization Trigger Document.md (Section 1 populated)"
---

# Task: Stage 2 Authorization Trigger Document — Section 1 Population

## Context

FORGE filed `Infrastructure/Stage 2 Crawl Expansion — Authorization Trigger Document.md` on April 26. The document was filed with Section 1 empty — the 6-metric performance table has thresholds pre-populated from the Validation Spec and related documents, but the actual results column is blank. FORGE's briefing (2026-04-26 06:39) noted: "FORGE fills Section 1 after May 1 + 3-day monitoring window."

This task formalizes that execution commitment. The 3-day monitoring window ends May 4 (May 1 launch + 3 runs: May 1, 2, 3). FORGE has until end of day May 4 to fill Section 1 and submit the trigger document to SENTINEL.

SENTINEL uses this document — and only this document — to make the Stage 2 authorization decision. If Section 1 is incomplete, SENTINEL cannot authorize Stage 2. Stage 2 authorization is the gate for expanding the org-ID crawl scope beyond the initial Stage 1 entities. The browser session entities in Section 3 are also pending — FORGE should update Section 3 at the same time if the browser session has been completed.

## What to Do

**Target file:** `02 - Tiger Tournaments/Projects/Infrastructure/Stage 2 Crawl Expansion — Authorization Trigger Document.md`

Populate Section 1 only (and Section 3 if browser session complete). Do not modify the SENTINEL Disposition block.

---

### Section 1: First-Week Performance Evidence

After 3 pipeline runs (May 1, 2, 3), fill in each row of the 6-metric table:

| Metric | Threshold | Actual (3-run avg) | PASS / FAIL | Source |
|--------|-----------|-------------------|-------------|--------|
| New games per run | ≥ [threshold from Validation Spec] | | | First-Week Monitoring Log |
| Run completion rate | 100% (no crashes) | | | First-Week Monitoring Log |
| Source error rate | ≤ [threshold] | | | First-Week Monitoring Log |
| Game dedup rate | ≤ [threshold] | | | First-Week Monitoring Log |
| GotSport org-ID hit rate | ≥ [threshold] | | | First-Week Monitoring Log |
| Crawl duration | ≤ [threshold] | | | First-Week Monitoring Log |

Pull thresholds from the Stage 2 Authorization Trigger Document (already pre-populated from the Validation Spec).
Pull actuals from the First-Week Pipeline Monitoring Log daily entries (May 1–3 rows).

---

### Section 2: Blocking Issues Log

State one of:
- "No blocking issues detected in May 1–3 runs."
- List any blocking issues: issue, severity (1/2/3), current status, resolution plan.

A Severity 3 issue (crash, data corruption, zero-game run) from any of the 3 runs constitutes an automatic Stage 2 HOLD. Note this explicitly if it applies.

---

### Section 3: Browser Session Org-ID Status (update if applicable)

If Presten has run the browser session by May 4:
- Update each of the 11 entity rows from `pending-browser-lookup` to either `confirmed-[org_id]` or `not-on-gotsport`
- Add note: "Browser session completed [date]. [N] entities confirmed, [N] not on GotSport."

If browser session has not run by May 4:
- Leave Section 3 as-is with `pending-browser-lookup` status
- Add note: "Browser session not yet completed as of May 4. Section 3 update deferred."

---

### Section 4: FORGE Authorization Request

Fill the authorization request template at the bottom of Section 4:

```
FORGE requests Stage 2 authorization based on the following:
- Section 1: [N/6] metrics PASS
- Section 2: [No blocking issues / N blocking issues — status]
- Section 3: [N/11 entities confirmed / browser session pending]
- FORGE assessment: [Stage 2 is ready to proceed / Stage 2 should be held — reason]
- Requested Stage 2 start date: [date]
```

---

## Quality Standard

- Pull all numbers from the First-Week Monitoring Log. Do not estimate or use prior projections.
- If any metric FAILS, do not request authorization. State "FORGE does not recommend Stage 2 at this time" and provide the specific metric that failed and the reason.
- The authorization request must be a definitive recommendation (PROCEED or HOLD), not hedged.
- File a note in the FORGE briefing on May 4 that the trigger document has been submitted to SENTINEL.

## Timing

| Milestone | Date |
|-----------|------|
| May 1 pipeline launch | May 1 |
| May 1 monitoring log entry | May 1 (same day) |
| May 2 monitoring log entry | May 2 |
| May 3 monitoring log entry | May 3 |
| **This task: Section 1 fill + SENTINEL submission** | **May 4** |
| SENTINEL Stage 2 authorization decision | May 5 (target) |

## Why This Matters

Stage 2 expansion significantly increases game coverage — it adds all browser-confirmed org-IDs beyond the initial Stage 1 set. The expansion is only safe if Stage 1 is demonstrably stable. If FORGE does not fill this document by May 4, Stage 2 authorization slips to the May 9 session at the earliest — compressing the pre-DSS data collection window. Three days of first-week monitoring is the minimum SENTINEL requires before signing Stage 2.

## References

- `Infrastructure/Stage 2 Crawl Expansion — Authorization Trigger Document.md` — the file to update
- `Infrastructure/First-Week Pipeline Monitoring Log — May 1–7.md` — data source for Section 1
- `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` — Section 3 source for browser session status
- `Infrastructure/GotSport Org-ID Expansion Stage 2 — Pre-Authorization Criteria.md` — G1–G5 gate definitions
