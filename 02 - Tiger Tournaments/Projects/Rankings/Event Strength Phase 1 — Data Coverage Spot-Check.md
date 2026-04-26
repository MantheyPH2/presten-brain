---
title: Event Strength Phase 1 — Data Coverage Spot-Check
type: coverage-verification
author: ELO
date: 2026-04-25
status: ready-to-run
due: 2026-04-28
related_task: task-2026-04-25-event-strength-phase1-data-coverage-check
related: "[[Event Strength Phase 1 — SENTINEL Authorization Criteria]]", "[[Post-Fix Calibration Stability Monitoring Spec]]", "[[DSS Demo — Feature Sequence Plan]]", "[[DSS Feature Readiness Tracker — May 2026]]"
tags: [event-strength, coverage, dss, evo-draw, pre-flight]
---

# Event Strength Phase 1 — Data Coverage Spot-Check

> **Purpose:** Verify that `event_strength` tier classifications exist in the database for the events most likely to appear in the DSS demo. Authorization is complete; this spot-check confirms authorization translates to real data.
>
> **Run this:** After April 29 calibration verdict is GREEN. Before May 9 readiness assessment.
>
> **Time required:** ~5 minutes in psql.

---

## Section 1: Top-Events Query

Returns the top 25 events by game count with their computed `event_strength` tier classification. Identifies any coverage gaps in the highest-volume events.

```sql
SELECT
  e.event_name,
  COUNT(g.id) AS game_count,
  es.strength_class,
  es.strength_percentile,
  es.avg_team_rating::int AS avg_team_rating,
  CASE WHEN es.strength_class IS NULL THEN '❌ UNCLASSIFIED' ELSE '✅' END AS coverage_status
FROM games g
JOIN events e ON e.id = g.event_id
LEFT JOIN event_strength es
  ON es.event_id = e.id
  AND es.season = '2025-2026'
WHERE g.is_hidden = false
  AND g.match_date >= '2025-08-01'
GROUP BY e.event_name, es.strength_class, es.strength_percentile, es.avg_team_rating
ORDER BY game_count DESC
LIMIT 25;
```

**Pass criteria:** ≥ 20 of the top 25 events have a non-NULL `strength_class`.

| Result | Action |
|--------|--------|
| ≥ 20 classified | ✅ Phase 1 top-event coverage confirmed. Proceed to Section 2. |
| 15–19 classified | ⚠️ Flag to SENTINEL. Phase 1 likely demo-usable but identify which top events are missing and assess if any are demo-critical. |
| < 15 classified | 🚨 Phase 1 needs targeted recompute before May 9. ELO files Queue item immediately with missing event list. |

**Fill after running:**

```
Top-25 events classified: ___ / 25
```

---

## Section 2: Demo-Event Spot-Check

These are the five events most likely to be named or featured in the DSS demo, derived from `DSS Demo — Feature Sequence Plan.md` Section 3 (Event Strength step). ELO's expected tier is pre-filled — compare against query results.

**Query:** Run for each event name (adjust `event_name ILIKE` pattern as needed):

```sql
SELECT
  e.event_name,
  es.strength_class,
  es.strength_percentile::int AS percentile,
  es.avg_team_rating::int AS avg_team_rating,
  es.upset_rate,
  es.team_count,
  COALESCE(es.strength_class, 'MISSING') AS coverage_status
FROM event_strength es
JOIN events e ON e.id = es.event_id
WHERE e.event_name ILIKE '%[INSERT EVENT NAME]%'
  AND es.season = '2025-2026'
ORDER BY es.strength_percentile DESC
LIMIT 5;
```

**Run for all 5 demo events using the patterns below:**

| Event Pattern | `ILIKE` Pattern | ELO Expected Tier | Actual Tier | Coverage Status |
|--------------|----------------|------------------|-------------|-----------------|
| ECNL Girls National Event (any) | `'%ecnl%national%'` OR `'%ecnl%event%'` | High | _(query result)_ | ✅ / ❌ |
| MLS NEXT Showcase or Playoffs | `'%mls next%showcase%'` OR `'%mls next%playoffs%'` | High | _(query result)_ | ✅ / ❌ |
| Girls Academy Playoffs or Cup | `'%girls academy%playoffs%'` OR `'%ga%cup%'` | Medium–High | _(query result)_ | ✅ / ❌ |
| NPL National or Regional Finals | `'%npl%national%'` OR `'%npl%regional%'` | Medium | _(query result)_ | ✅ / ❌ |
| DPL State/Regional Event | `'%dpl%'` | Low–Medium | _(query result)_ | ✅ / ❌ |

**Rationale for ELO's expected tier assignments:**
- ECNL and MLS NEXT events concentrate top-rated teams; avg_team_rating should be 75th percentile or higher → High.
- GA Playoffs similarly concentrate national-caliber Girls teams → Medium–High.
- NPL national finals attract mid-tier competitive programs → Medium.
- DPL events are regional development circuit → Low–Medium.

**If the ECNL National event is missing:** This is the single most important demo event for Event Strength Phase 1. If it has no `strength_class`, check whether the event exists in `events` at all. If it does but has no strength entry, a targeted Phase 1 recompute for that event is required before May 9.

---

## Section 3: Coverage Summary

Fill this after running Sections 1 and 2.

```
Top-25 events with strength_class: ___ / 25
Demo-critical events with strength_class: ___ / 5
Phase 1 demo-ready verdict: [ ] YES   [ ] NEEDS RECOMPUTE
```

**If verdict is NEEDS RECOMPUTE:**

- Missing events: ________
- Gap type:
  - [ ] Computation gap — event exists in `events` but has no `event_strength` row → recompute required for those events
  - [ ] Classification gap — event has too few games or teams to classify → note as non-rankable in briefing
- ELO recommended action: ________
- Estimated time to remediate before May 9: ________

---

## Section 4: Upset Rate Coverage Check

Phase 1 includes an upset rate per event. A missing `upset_rate` means Phase 1 can still show the strength class and percentile but cannot show the full three-metric display featured in the demo.

```sql
SELECT
  COUNT(*) FILTER (WHERE upset_rate IS NOT NULL) AS with_upset_rate,
  COUNT(*) AS total_classified,
  ROUND(
    COUNT(*) FILTER (WHERE upset_rate IS NOT NULL) * 100.0 / NULLIF(COUNT(*), 0),
    1
  ) AS upset_rate_coverage_pct
FROM event_strength
WHERE season = '2025-2026';
```

**Pass criteria:** ≥ 90% of classified events have a non-NULL `upset_rate`.

| Result | Action |
|--------|--------|
| ≥ 90% | ✅ Full Phase 1 display confirmed. |
| 75–89% | ⚠️ Flag to SENTINEL. Demo script should note that upset rate is not available for all events; pick a demo event where it is present. |
| < 75% | 🚨 Phase 1 demo should not feature upset rate as a primary claim. Update DSS Demo Feature Sequence Plan Step 3 to omit upset rate mention. |

```
Upset rate coverage: ___% (___/___  classified events)
```

---

## Section 5: Post-Check Action

After running all four sections, file summary in next ELO briefing using this format:

> "Phase 1 data coverage: [N]/25 top events classified. Demo events: [N]/5 have strength tiers. Upset rate coverage: [N]%. Verdict: [YES / NEEDS RECOMPUTE]. [If recompute needed: specific events and ELO recommended action.]"

If verdict is YES: update `DSS Feature Readiness Tracker — May 2026.md` Event Strength row from ⚠️ to ✅ (pending April 29 G2 gate confirmation).

If verdict is NEEDS RECOMPUTE: file Queue item `Queue/pending-2026-04-[date]-event-strength-recompute.md` with missing event list.

---

## Related

- [[Event Strength Phase 1 — SENTINEL Authorization Criteria]] — methodology authorized; this doc confirms data
- [[Post-Fix Calibration Stability Monitoring Spec]] — calibration stability is a parallel Phase 1 dependency
- [[DSS Demo — Feature Sequence Plan]] — Section 3 (Event Strength step) names the specific event to feature
- [[DSS Feature Readiness Tracker — May 2026]] — Event Strength row to update after this check

*ELO — 2026-04-25*
