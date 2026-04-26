---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-25
status: completed
completed: 2026-04-25
priority: high
due: 2026-04-28
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Event Strength Phase 1 — Data Coverage Spot-Check.md"
---

# Task: Event Strength Phase 1 — Data Coverage Spot-Check

## Context

Event Strength Phase 1 has been authorized conditionally (pending April 29 G2 gate and calibration stability clearance). The authorization criteria spec, stability monitoring spec, and execution package are all filed. However: authorization means SENTINEL has cleared the *criteria and methodology* — it does not confirm that the event_strength tier data actually exists in the database for the events most likely to appear in the DSS demo.

Phase 1 delivers: a per-event High/Medium/Low classification, a percentile ranking, and an upset rate. If the top events by game volume don't have computed tier values, the DSS demo has an empty feature at its top slot (DSS Demo Value: 4/5). Discovering this on May 9 readiness check — or worse, during the demo — is unacceptable.

This task: ELO writes a targeted SQL spot-check that tells SENTINEL how many of the top events actually have event_strength tier values. SENTINEL uses this to determine whether Phase 1 is demo-ready in practice (not just in theory).

## What to Produce

### File: `Rankings/Event Strength Phase 1 — Data Coverage Spot-Check.md`

---

### Section 1: Top-Events Query

SQL to return the top 25 events by game count with their event_strength tier classification:

```sql
-- ELO writes the actual query
-- Should return: event_name, game_count, event_strength_tier (or NULL if unclassified)
-- ORDER BY game_count DESC
-- LIMIT 25
```

**Pass criteria:** ≥ 20 of the top 25 events have a non-NULL event_strength_tier. If < 20 have tiers, ELO flags to SENTINEL immediately — Phase 1 may need a targeted recompute before May 9.

---

### Section 2: Demo-Event Spot-Check

Beyond the volume-top-25, these specific events are likely to appear in the DSS demo by name. ELO checks tier coverage for each:

| Event Name | Expected Tier | Actual Tier | Coverage Status |
|-----------|--------------|-------------|-----------------|
| [ECNL National Event — specify name] | High | [query result] | ✅ / ❌ |
| [MLS NEXT Showcase or Playoffs — specify] | High | [query result] | ✅ / ❌ |
| [GA Cup or equivalent] | Medium | [query result] | ✅ / ❌ |
| [NPL National — specify] | Medium | [query result] | ✅ / ❌ |
| [Regional DPL event — specify] | Low | [query result] | ✅ / ❌ |

ELO should select the 5 most demo-relevant events based on the DSS Demo Feature Sequence Plan (already filed). The "Expected Tier" column is ELO's pre-run prediction — fill it before running the query, then compare.

---

### Section 3: Coverage Summary

```
Top-25 events with tier value: [N] / 25
Demo-critical events with tier value: [N] / 5
Phase 1 demo-ready verdict: YES / NEEDS RECOMPUTE
```

If verdict is NEEDS RECOMPUTE:
- Which events are missing tier values?
- Is the missing data a computation gap (event_strength recompute needed) or a classification gap (events with 0 games, not rankable)?
- ELO's recommended action before May 9?

---

### Section 4: Upset Rate Coverage Check

Phase 1 also includes an upset rate per event. Run a parallel check:

```sql
-- ELO writes query
-- Should return: count of events WITH a computed upset_rate vs. total events with tier values
-- Pass criteria: ≥ 90% of classified events also have an upset_rate
```

If upset_rate coverage < 90%, flag to SENTINEL: Phase 1 may need to present the feature without upset rate for low-coverage events.

---

## Definition of Done

- File written at `Rankings/Event Strength Phase 1 — Data Coverage Spot-Check.md`
- All queries are copy-paste ready
- Section 2 demo-event table has ELO's expected tier pre-filled before running queries
- Section 3 verdict is definitive: YES or NEEDS RECOMPUTE
- If NEEDS RECOMPUTE: specific events identified and recommended action stated
- Summary in briefing: "Phase 1 data coverage: [N]/25 top events classified. Demo events: [N]/5 have tiers. Verdict: [YES/NEEDS RECOMPUTE]."

## Why This Matters

Authorization without data is a paper guarantee. SENTINEL needs to know before May 9 whether Phase 1 can actually be demonstrated — not just whether it has been authorized. This spot-check is the reality check. It takes ELO 30 minutes to write and Presten 5 minutes to run. The result either confirms that Phase 1 is ready (and SENTINEL removes this risk from the DSS Readiness Tracker), or it surfaces a gap with enough time to fix it before May 9.

## References

- `Rankings/Event Strength Phase 1 — Authorization Criteria.md` — authorization criteria (methodology)
- `Rankings/Post-Fix Calibration Stability Monitoring Spec.md` — stability criteria Phase 1 also depends on
- `Product & Planning/DSS Demo Feature Sequence Plan — May 2026.md` — which events to use as Section 2 demo examples
- `Product & Planning/DSS Feature Readiness Tracker — May 2026.md` — Event Strength row to update after this check
