---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-27
status: pending
priority: medium
due: 2026-05-09
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Event Strength Phase 1 — ELO Execution Package.md"
---

# Task: Event Strength Phase 1 — ELO Execution Package

## Context

Event Strength Phase 1 authorization is gated on April 29 G1–G4 all passing and the April 28 Execution Log being clean. Once SENTINEL authorizes Phase 1, ELO's role in Phase 1 isn't passive — ELO is responsible for validating that the event strength assignments align with calibration assumptions. FORGE handles the data ingestion/assignment mechanics; ELO provides calibration validation.

What does not currently exist: an ELO execution package for Phase 1. There are FORGE-side planning documents (event strength phase 2 scope, league tier audit from April 24), but ELO's verification role has no structured document. When Phase 1 is authorized and FORGE executes the event strength assignments, ELO will be asked "does this look right calibration-wise?" This package makes that answer structured rather than ad hoc.

Writing this now — before Phase 1 is authorized — means ELO can execute its verification role within one session after authorization, rather than spending the session designing the verification approach.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/Event Strength Phase 1 — ELO Execution Package.md`

---

### Section 1: Phase 1 Scope Summary

From FORGE's phase 1 documents, summarize what Phase 1 actually changes:

- Which leagues / event tiers are getting event strength assignments in Phase 1?
- What are the expected event_strength values (or ranges) per tier?
- How many events / games are expected to be affected?
- Does Phase 1 touch Boys ratings, Girls ratings, or both?

If FORGE's event strength phase 1 documents don't specify all of this, note what is known and what ELO needs to confirm with FORGE before running validation.

---

### Section 2: ELO Calibration Assumptions for Phase 1

For each league/tier getting an event strength assignment in Phase 1, state ELO's expected effect:

| League / Event Tier | Current cal_value | Event Strength Value (Phase 1) | Expected Rating Impact | Expected Direction |
|---------------------|------------------|---------------------------------|------------------------|-------------------|
| [tier 1] | | | [minimal / moderate / significant] | [up / down / neutral] |
| [tier 2] | | | | |
| ... | | | | |

"Expected Rating Impact" should be grounded in ELO's calibration understanding, not just repeated from FORGE's spec. ELO owns the calibration interpretation.

---

### Section 3: Validation Query Pack

Three queries ELO runs after FORGE completes Phase 1 assignments:

**Query 1 — Event Strength Assignment Confirmation**
```sql
-- Confirm all Phase 1 leagues have event_strength assigned
-- Returns: event_tier, event_strength, event_count
-- Expected: no Phase 1 league shows NULL event_strength after assignment
SELECT event_tier, event_strength, COUNT(*) AS event_count
FROM events
WHERE event_tier IN ([Phase 1 tier list])
GROUP BY event_tier, event_strength
ORDER BY event_tier;
```

**Query 2 — Rating Shift After Phase 1 Recompute**
```sql
-- Compare average ratings before and after Phase 1 recompute
-- Pre-Phase-1 baseline: from April 29 gate results (G4 snapshot)
-- Returns: age_group, gender, avg_rating_before (from snapshot), avg_rating_after, delta
-- Expected: deltas consistent with ELO's Section 2 expectations
-- [ELO writes the specific query using the snapshot table or equivalent]
```

**Query 3 — Distribution Integrity Check**
```sql
-- Confirm Phase 1 did not introduce outliers inconsistent with Phase 1 scope
-- Returns: count of teams with > [threshold] pt rating change since April 29 baseline
-- Expected: count is within range predicted by Section 2 estimates
-- Alert threshold: count > [N] — flag to SENTINEL
```

---

### Section 4: Pass/Fail Criteria

After running the 3 validation queries, ELO assesses:

| Check | Pass Criterion | Fail Criterion | Action if Fail |
|-------|---------------|----------------|----------------|
| Q1: All Phase 1 tiers assigned | All Phase 1 tiers show assigned event_strength | Any Phase 1 tier shows NULL | Flag to FORGE: assignment incomplete |
| Q2: Rating deltas in expected range | All deltas consistent with Section 2 expectations | Any delta in unexpected direction | ELO investigates — potential cal logic error |
| Q3: No distribution outliers | Outlier count ≤ [threshold] | Outlier count > [threshold] | ELO identifies specific outlier teams — may indicate an event_tier was incorrectly scoped |

PASS on all 3 = Phase 1 validation complete. ELO files clearance in briefing.
FAIL on any = ELO flags in briefing with specific failure. SENTINEL holds Phase 2 authorization.

---

### Section 5: ELO Clearance Filing

When all 3 queries pass, ELO files the following in the next briefing:

```
Event Strength Phase 1 — ELO Validation Clearance
Date: [date]
Q1: PASS — [N] Phase 1 tiers assigned with event_strength
Q2: PASS — rating deltas within expected range. Largest shift: [N] pts in [age group/tier]. Within Section 2 estimate.
Q3: PASS — [N] outlier teams (threshold: [N]). No distribution anomalies.

Event Strength Phase 1 validated. ELO recommends SENTINEL authorize Event Strength Phase 2 scoping session.
```

---

### Section 6: Dependency Chain

State the authorization dependencies this package sits within:

```
April 28 completes → April 29 G1–G4 all pass → SENTINEL authorizes Phase 1
→ FORGE executes Phase 1 event strength assignments
→ ELO runs this package validation (3 queries)
→ ELO files clearance in briefing
→ SENTINEL authorizes Phase 2 scoping
```

If April 29 G2 fails (rating distribution unexpected), Phase 1 authorization is held regardless of this package.

---

## Quality Standard

- Section 2 must commit to specific expected directions for each Phase 1 tier. "TBD pending FORGE spec" is acceptable for event_strength values but not for expected calibration direction.
- Query 2 is the most complex — ELO needs to either write the actual comparison SQL or note explicitly that the April 29 snapshot table needs to exist first. If the snapshot table approach is not available, describe the alternative.
- Section 5 clearance format must be used verbatim — SENTINEL will look for this specific format in briefings.

## Why This Matters

Without this package, ELO's Phase 1 validation is a judgment call made session-by-session. The calibration risk is real: if event strength assignments are incorrect for a subset of leagues, the rating distortion compounds with each subsequent pipeline run. ELO catching this through structured validation within 48 hours of Phase 1 execution is qualitatively different from ELO noticing it 3 weeks later when DSS is approaching. The package cost is low (one session to write); the catch value is high.

## References

- `Rankings/Event Strength Phase 1 — Execution Package.md` (FORGE) — scope details this package validates
- `Rankings/Event Strength — League Tier Audit.md` — event tier / cal value source of truth for Section 2
- `Rankings/April 29 Gate Results — Structured Log.md` — G4 baseline snapshot values used in Q2
- `Rankings/Post-Fix Calibration Stability Monitoring Spec.md` — ongoing monitoring this package feeds into
- `Product & Planning/DSS Feature Readiness Tracker — May 2026.md` — Event Strength Phase 1 readiness cell
