---
title: Event Strength — League Tier Coverage Audit
type: pre-implementation-audit
author: ELO
date: 2026-04-24
status: complete
related: "[[Event Strength Ratings — Implementation Plan]]", "[[League Hierarchy]]"
tags: [event-strength, event-tier, leagues, implementation, pre-flight, evo-draw]
---

# Event Strength — League Tier Coverage Audit

> **Summary:** 8 leagues have confirmed `event_tier` assignments (fully covered). 5 Tier 2 leagues (USL Academy, EDP, Elite 64, NAL, ODP) lack assigned `event_tier` values. Remediation SQL provided. Phase 1 pre-flight gate written. **Critical ordering constraint: Phase 1 must run AFTER the April 28 GA ASPIRE fix** — running before will assign strength ratings to GA ASPIRE events using the wrong tier, requiring a recompute.

---

## Section 1: League Hierarchy Tier Assignment Coverage

Based on [[League Hierarchy]] as of 2026-04-24:

| League | Tier | `event_tier` in DB | Cal Value | Notes |
|--------|------|--------------------|-----------|-------|
| MLS NEXT (Homegrown) | 1 | `mlsnext_homegrown` | 160 (Boys) | Post-split; requires FORGE classifier to have run |
| MLS NEXT (Academy) | 1 | `mlsnext_academy` | 135 (Boys) | Post-split |
| MLS NEXT (Unclassified) | 1 | `mlsnext_unclassified` | 147 (Boys) | Post-split fallback |
| MLS NEXT (legacy) | 1 | `mlsnext` | 160 | Pre-split; will be 0 rows after May 18–20 |
| ECNL | 1 | `ecnl` | 120 boys / 130 girls | ✅ Fully assigned |
| GA (Girls Academy) | 1 | `ga` | 140 girls / 100 boys | ✅ Fully assigned |
| GA ASPIRE | 1 | `ga_aspire` | 100 girls / 100 boys | ✅ Post-April-28-fix |
| ECNL RL | 2 | `ecnl_rl` | 55 | ✅ Fully assigned |
| NPL | 2 | `npl` | 55 | ✅ Fully assigned |
| DPL | 2 | `dpl` | 55 | ✅ Fully assigned |
| Pre-ECNL | 3 | `pre_ecnl` | 25 | ✅ Fully assigned |
| USL Academy | 2 | **unconfirmed** | — | Likely `uslacdemy` or null; no calibration value in League Hierarchy |
| EDP | 2 | **unconfirmed** | — | EDP events may be tagged `edp` or `state` or null |
| Elite 64 | 2 | **unconfirmed** | — | No event_tier or calibration in League Hierarchy |
| NAL | 2 | **unconfirmed** | — | No event_tier or calibration in League Hierarchy |
| USYS State Cup / State Leagues | 3 | **unconfirmed** | — | May be tagged `state` or `usys` or null |
| ODP | 3 | **unconfirmed** | — | Talent identification pipeline; event_tier status unknown |

**Fully confirmed:** 11 tier entries across the 8 core competitive circuits.
**Unconfirmed:** 6 circuits (USL Academy, EDP, Elite 64, NAL, State Leagues, ODP).

---

## Section 2: Null Tier Behavior — Phase 1 Gap Analysis

### Q1: What does Phase 1 do with events from leagues that have no tier assignment?

From the Event Strength Implementation Plan Phase 1, the `event_strength` computation joins `events` (or `games.event_name`) to `rankings` for team ratings. The strength classification uses **avg_team_rating percentile** within the age group — not directly the `event_tier` calibration value.

**Good news:** The Event Strength computation does NOT require `event_tier` to compute avg_team_rating, spread, or upset rate. An event with `event_tier = NULL` can still have a strength rating computed if its teams are in the `rankings` table.

**However,** two downstream issues exist for null-tier events:

1. **Query C validation (tier vs strength cross-tab):** Post-compute validation checks whether `ga` and `mlsnext` tier events skew High/Medium and `state`/`local` events skew Low. This check fails for events with null tiers — they appear in no category, making the distribution summary incomplete.

2. **Event Strength filter endpoint:** `GET /api/events?strength=High&event_tier=ga` — filter queries that join event_strength back to event_tier would miss null-tier events. Users filtering by event tier would get incomplete results.

### Q2: How many events in the DB come from unconfirmed-tier leagues?

Run to determine actual counts:

```sql
-- Count events by event_tier to find null or unrecognized tiers
SELECT
  event_tier,
  COUNT(*) AS event_count,
  COUNT(DISTINCT CASE WHEN age_group IS NOT NULL THEN id END) AS events_with_age_group
FROM events
GROUP BY event_tier
ORDER BY event_count DESC;
```

Expected output: This will reveal any `event_tier` values that exist in the DB but aren't in the League Hierarchy (e.g., `edp`, `usla`, `state`, null). Report the counts before running Phase 1.

### Q3: For each unconfirmed league — what SHOULD the tier be?

| League | Proposed `event_tier` | Proposed Cal Value | Rationale |
|--------|----------------------|-------------------|-----------|
| USL Academy | `usl_academy` | 55 | Comparable to ECNL RL — professional pathway but below ECNL in competitive depth |
| EDP | `edp` | 55 | Regional Tier 2 — comparable to NPL/ECNL RL in Eastern states |
| Elite 64 | `elite_64` | 55 | Elite showcase; field quality similar to ECNL RL at qualifying events |
| NAL | `nal` | 25–55 | Competition level variable; conservative assignment pending cross-league analysis |
| USYS State Cup | `state` | 35 | Between Pre-ECNL (25) and NPL (55); state champion quality |
| State Leagues | `state` | 35 | General state-level competition |
| ODP | `odp` | 35 | Talent ID pipeline; game quality variable |

> [!warning] These proposed values are calibration estimates, not validated calibration values.
> USL Academy, EDP, Elite 64, and NAL have not been through cross-league Brier analysis. The 55-point assignments are conservative estimates based on structural position in the competitive pathway. Use these for Event Strength tier classification but do NOT apply these as Ranking Engine calibration values without empirical validation.

---

## Section 3: Remediation SQL

Run before Phase 1 to ensure all events have a valid `event_tier`. If `event_tier` is on the `events` table:

```sql
-- STEP 1: Assign USL Academy events
UPDATE events
SET event_tier = 'usl_academy'
WHERE event_tier IS NULL
  AND (event_name ILIKE '%USL Academy%' OR event_name ILIKE '%USLA%');

-- STEP 2: Assign EDP events
UPDATE events
SET event_tier = 'edp'
WHERE event_tier IS NULL
  AND event_name ILIKE '%EDP%';

-- STEP 3: Assign Elite 64 events
UPDATE events
SET event_tier = 'elite_64'
WHERE event_tier IS NULL
  AND (event_name ILIKE '%Elite 64%' OR event_name ILIKE '%E64%');

-- STEP 4: Assign NAL events
UPDATE events
SET event_tier = 'nal'
WHERE event_tier IS NULL
  AND event_name ILIKE '%NAL%';

-- STEP 5: Assign USYS State Cup events
UPDATE events
SET event_tier = 'state'
WHERE event_tier IS NULL
  AND (event_name ILIKE '%State Cup%' OR event_name ILIKE '%USYS%State%');

-- STEP 6: Assign remaining null-tier events to 'local' fallback
UPDATE events
SET event_tier = 'local'
WHERE event_tier IS NULL;
```

> [!note] Adapt column names to match actual `events` table schema.
> If `event_tier` is stored in `event_tiers` (a separate join table) rather than directly on `events`, adjust the UPDATE statements to target the appropriate table.

After running all remediation steps:

```sql
-- Confirm no null event_tiers remain
SELECT COUNT(*) AS null_tier_events FROM events WHERE event_tier IS NULL;
-- Expected: 0
```

---

## Section 4: Phase 1 Pre-Flight Gate

Run this query before starting the Event Strength implementation session. It must return 0 before Phase 1 can proceed:

```sql
-- Phase 1 gate: must return 0 rows before proceeding with Event Strength Phase 1
SELECT COUNT(*) AS unassigned_events
FROM events
WHERE event_tier IS NULL;
```

**If this returns > 0:** Apply the remediation SQL from Section 3. Rerun the gate until it returns 0.

**Why this matters:** Phase 1 computes `event_strength` rows grouped by `event_tier`. Events with null tiers are excluded from the strength classification distribution, which inflates the percentile scores for classified events (they're compared against a smaller universe). A clean pre-flight ensures the percentile computation is accurate across the full event dataset.

---

## Section 5: GA ASPIRE Ordering Constraint

**Phase 1 of Event Strength MUST run AFTER the April 28 GA ASPIRE fix.**

If Phase 1 runs before April 28:
- GA ASPIRE events are currently tagged `event_tier = 'ga'` (cal=140)
- Phase 1 assigns event strength ratings to these events under the `ga` tier
- April 28 fix reclassifies them to `ga_aspire` (cal=100)
- The event_strength rows now reference the wrong event_tier context
- Phase 1 must be re-run to recompute strength ratings under the correct `ga_aspire` tier

**Consequence:** Running Phase 1 before April 28 is not broken — it just requires a Phase 1 re-run afterward, wasting the 2-hour implementation window. Given that the Event Strength session is scheduled to start April 28 anyway (per the implementation plan's `start_by: 2026-04-28`), this is a natural alignment.

**Explicit ordering constraint:**
```
April 28: GA ASPIRE UPDATE + ranking recompute
April 28 (same session, after recompute): Begin Event Strength Phase 1
```

If the GA ASPIRE fix and the Event Strength Phase 1 are in separate sessions, verify the GA ASPIRE UPDATE has been executed before starting Phase 1:

```sql
-- Confirm GA ASPIRE fix has run (should return 0 if fix was applied)
SELECT COUNT(*) FROM events
WHERE event_name ILIKE '%ASPIRE%'
  AND event_tier = 'ga';
-- Expected: 0 (all ASPIRE events should now be 'ga_aspire')
```

If this returns > 0: the GA ASPIRE fix has not run. Do not start Event Strength Phase 1 until it does.

---

## FORGE Note

This constraint should be added to the April 28 Escalation Reference Card: "If Event Strength Phase 1 is scheduled for April 28, run it AFTER the GA ASPIRE UPDATE and recompute, not before."

---

## References

- [[Event Strength Ratings — Implementation Plan]] — Phase 1 design; pre-flight to add this gate
- [[League Hierarchy]] — authoritative league list and calibration values
- `02 - Tiger Tournaments/Projects/Reports/ga-aspire-tier-fix-2026-04-24.md` — GA ASPIRE UPDATE SQL (must run first)
- `10 - Agents/ELO/Tasks/task-2026-04-24-event-strength-post-compute-validation.md` — post-Phase-1 validation (Query C requires valid event_tier values)
