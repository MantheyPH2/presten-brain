---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-27
status: pending
priority: medium
due: 2026-05-05
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/SnapSoccer — Test Extraction Plan.md"
---

# Task: SnapSoccer — Test Extraction Plan

## Context

The SincSports Parser Reuse Assessment (`task-2026-04-27-sincsports-parser-reuse-check.md`) produces a REUSE / ADAPT / REBUILD verdict for the SnapSoccer ingestion build. Regardless of the verdict, the next step before committing to a full build session is a small-scale test extraction: parse one SnapSoccer event, validate the output, and confirm the approach handles real data before investing 4–12 hours in a full build.

This test extraction is the difference between a full build session that works on the first day and one that burns the first 2 hours debugging structure assumptions. FORGE should design the test now so it is ready to execute the moment the post-DSS sprint window opens (May 17+). If the test can be done before DSS (e.g., in a standalone 1-hour window), FORGE should flag that in the plan.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/SnapSoccer — Test Extraction Plan.md`

---

### Section 1: Test Scope

Define the minimum viable test:

- **Target event:** One SnapSoccer event with completed results. Choose an event that: (a) has games already in the Evo Draw DB (so output can be cross-validated), or (b) is in a league FORGE has calibration values for.
- **Target URL:** The specific SnapSoccer game listing URL FORGE will parse (or the URL pattern if exact URL requires a live lookup).
- **Expected game count:** How many games should the test extraction return? If unknown, describe how to derive the expected count.
- **Gender and age group:** Specify one gender/age group for the test. Rationale: tests one parser branch at a time.

---

### Section 2: Test Execution Steps

Step-by-step instructions for running the test extraction:

1. **Setup** — What environment, dependencies, or config does the test require? (Minimal: this is a one-event test, not a production run.)
2. **Run** — Exact command or code block to execute the extraction against the target URL.
3. **Output format** — What does a successful extraction output? (JSON, CSV, stdout?) Describe the expected fields and one example row.
4. **Validation queries** — After extraction, what SQL confirms the test data loaded correctly?

```sql
-- Example: confirm test games are present with correct fields
SELECT game_date, home_team, away_team, score, source, event_name
FROM games
WHERE source = 'snapsoccer'
  AND event_name = '[test event name]'
ORDER BY game_date;
```

---

### Section 3: Pass/Fail Criteria

| Check | Pass | Fail |
|-------|------|------|
| Game count | Extracted count matches expected ± 5% | Count differs by > 10% or is 0 |
| Field completeness | home_team, away_team, score, game_date, event_name all populated | Any required field is NULL in > 5% of rows |
| Team name format | Names match or are reconcilable with existing teams in DB | Names are garbled or truncated |
| Duplicate detection | No duplicate game_id for same home/away/date | Duplicates present |
| Error rate | 0 parse errors | Any uncaught exceptions |

If all checks pass: **test PASS** — proceed to schedule full build session.
If any check fails: **test FAIL** — document the failure, update the Reuse Assessment verdict if the failure indicates a structural incompatibility, and escalate to SENTINEL before scheduling a full build.

---

### Section 4: Pre-DSS vs. Post-DSS Recommendation

Based on the Reuse Assessment verdict and the test scope, FORGE states:

- **Can this test run before May 16 (DSS)?** Yes if: (a) it requires < 1 hour standalone, (b) the existing SincSports parser requires ≤ 30 minutes of adaptation to point at SnapSoccer, and (c) it does not touch production data.
- **Recommended window:** State the specific window (e.g., "April 29 after the ELO session, ≤ 1 hour standalone" or "May 17+ post-DSS").
- **Risk if test is skipped:** What is the risk of going straight from the Reuse Assessment to the full build session without a test extraction?

---

### Section 5: Full Build Session Prerequisites

What must be true before Presten schedules the full SnapSoccer build session?

- [ ] Test extraction PASS confirmed
- [ ] SincSports accessibility check confirmed (`soccer.sincsports.com` reachable — pending Presten 30-second check)
- [ ] Reuse Assessment verdict filed (`Infrastructure/SnapSoccer — SincSports Parser Reuse Assessment.md`)
- [ ] Post-DSS sprint window open (May 17+)
- [ ] SENTINEL has authorized SnapSoccer as the first post-DSS ingestion priority

---

## Why This Matters

The full SnapSoccer build session is the first post-DSS engineering commitment — 4–12 hours of Presten's time. Entering it without a validated test extraction means the first 1–2 hours may be spent discovering structural assumptions that turn REUSE into REBUILD. The test extraction is a 1-hour insurance policy on a 4–12 hour investment. Writing the plan now means it is ready to execute whenever Presten has an open hour, whether that is before or after DSS.

## Definition of Done

- Test scope is specific: one URL, one event, one gender/age group
- Execution steps are copy-paste ready (no pseudocode)
- Pass/fail criteria are explicit (numbers, not "looks right")
- Pre-DSS vs. post-DSS recommendation is stated with rationale
- Full build session prerequisites are listed as checkable conditions

## References

- `Infrastructure/SnapSoccer — SincSports Parser Reuse Assessment.md` — verdict that informs this plan
- `Infrastructure/Non-GotSport Top Sources — Ingestion Design.md` — SnapSoccer source context
- Existing SincSports bulk import code in the Evo Draw codebase
- `task-2026-04-27-sincsports-parser-reuse-check.md` — must be complete before this task's Section 1 can be finalized
