---
title: Boys GA ASPIRE Calibration Decision — April 2026
type: calibration-decision
author: ELO
date: 2026-04-24
status: decided
related: "[[League Hierarchy]]", "[[ga-aspire-tier-fix-2026-04-24]]"
tags: [boys, ga-aspire, calibration, decision, april-28]
---

# Boys GA ASPIRE Calibration Decision — April 2026

> **Decision date:** 2026-04-24
> **Decision:** Option A — Boys GA ASPIRE calibration = 100 (same as Boys GA). The April 28 UPDATE is safe to run without Boys-specific modification.
> **Action required on April 28:** Run the Boys existence check query (Section 1) and note the result. No additional handling needed.

---

## Context

The April 28 GA ASPIRE fix reclassifies events from `event_tier = 'ga'` to `event_tier = 'ga_aspire'` using a name-pattern UPDATE (`event_name ILIKE '%ASPIRE%'`). This UPDATE is gender-agnostic — it affects all events with "ASPIRE" in the name regardless of which genders played in them.

ELO's GA ASPIRE Fix SQL (`ga-aspire-tier-fix-2026-04-24.md`, Section 3) flagged an ambiguity: if Boys GA ASPIRE events exist and `compute-rankings.js` has no Boys-specific calibration for `ga_aspire`, the fix could silently miscalibrate Boys rankings. This document resolves that ambiguity.

---

## Section 1: Boys GA ASPIRE Existence Check

Run this on April 28 before executing the UPDATE (already included in `ga-aspire-tier-fix-2026-04-24.md` Section 3, but restated here for the Boys-specific record):

```sql
SELECT g.gender, COUNT(DISTINCT g.id) AS game_count, COUNT(DISTINCT e.id) AS event_count
FROM games g
JOIN events e ON e.id = g.event_id
WHERE e.event_name ILIKE '%ASPIRE%'
  AND e.event_tier = 'ga'
GROUP BY g.gender;
```

**Document the result here after running on April 28:**
- Boys game_count: ___________
- Boys event_count: ___________

If Boys game_count = 0: Boys GA ASPIRE events do not exist in the current DB. The UPDATE is Girls-only. Skip Sections 2–3 and note "Boys not affected."

If Boys game_count > 0: Boys GA ASPIRE events exist and will be reclassified. Read Sections 2–3 to confirm this is safe.

---

## Section 2: The Three Options and ELO's Decision

### Option A: Boys GA ASPIRE = 100 (same as Boys GA) ✅ SELECTED

**Calibration value:** 100

**Rationale:**

The Girls GA ASPIRE fix was needed because Girls GA ASPIRE events were tagged as `ga` (cal=140) when the correct value for GA ASPIRE-level competition is 100 — a 40-point overcalibration. The structural cause: GA ASPIRE launched in February 2025 as a lower-tier subset of GA, but the event classifier didn't yet distinguish it.

For Boys, the situation is structurally different:
- Boys GA calibration = **100** (already)
- Girls GA calibration = **140**

Boys GA sits 40 points below Girls GA because Boys GA is a secondary national circuit — MLS NEXT is Boys #1, and GA sits below it in the Boys competitive hierarchy. The calibration value of 100 for Boys GA already reflects a lower competitive standard than Girls GA (140).

When the April 28 UPDATE reclassifies Boys GA ASPIRE events from `event_tier = 'ga'` to `event_tier = 'ga_aspire'`, `compute-rankings.js` Phase 8 will look up the calibration value for `ga_aspire` for Boys games. The Boys GA ASPIRE calibration entry is 100 — the same as Boys GA.

**Net rating impact on Boys teams: zero.** The event tier tag changes but the calibration weight used in the Elo computation is unchanged (100 → 100).

**Risk:** If Boys GA ASPIRE competition is genuinely weaker than Boys GA (analogous to the Girls relationship), the correct Boys GA ASPIRE value could be ~70 (applying the same proportional reduction: 140 → 100 for Girls corresponds to 100 → ~70 for Boys). However:
1. No Boys-specific Brier analysis has been run to validate this
2. Boys GA game volume may be insufficient for reliable calibration at that level of detail
3. The overcalibration risk is smaller — if Boys GA ASPIRE teams are currently at cal=100 (same as Boys GA) and should be at ~70, we're potentially overcalibrating them by ~30 points, not 40 points
4. The conservative approach is to leave Boys GA ASPIRE at 100 until Option B (full Boys Brier analysis, scheduled post-DSS) produces data to justify a different value

---

### Option B: Boys GA ASPIRE = ~70

**Not selected for April 28.**

**Rationale for deferral:** This value is structurally plausible (same proportional reduction as Girls) but empirically unvalidated. Applying a new, unvalidated calibration change in the same session as the Girls fix introduces unnecessary risk. If the value is wrong (e.g., Boys GA ASPIRE and Boys GA are actually at the same level), it would understate Boys GA ASPIRE teams. Schedule Option B analysis for post-DSS June 2026.

---

### Option C: Exclude Boys from April 28 UPDATE

**Not selected.**

**Rationale:** Unnecessary. Option A establishes that the reclassification is safe for Boys (cal=100 → cal=100, no change). Adding a gender filter to the UPDATE complicates the SQL without any benefit. The simple UPDATE from Section 4 of the fix SQL is correct.

---

## Section 3: What Happens in compute-rankings.js

Phase 8 (League Calibration) of the ranking engine maps `event_tier` values to calibration weights. The relevant Boys entries:

| event_tier | Boys cal value |
|------------|---------------|
| ga | 100 |
| ga_aspire | 100 |

Both values are 100. After the April 28 reclassification, Boys games that were in `ga` events get moved to `ga_aspire` events. During the next recompute, Phase 8 looks up `ga_aspire` → 100. The rating computation is identical to before.

No code changes to `compute-rankings.js` are needed for Boys GA ASPIRE as part of the April 28 session.

---

## Section 4: League Hierarchy Update

The following row reflects this decision in [[League Hierarchy]]:

| Circuit | Gender | Cal Value | Validation Status | Note |
|---------|--------|-----------|-------------------|------|
| GA ASPIRE | Boys | 100 | assumed (safe for April 28) | Same as Boys GA. No net rating impact from April 28 reclassification. Full Boys GA ASPIRE validation deferred to post-DSS Brier analysis (June 2026). |

The League Hierarchy document has already been updated with this row and the associated Boys GA ASPIRE flag as of 2026-04-24.

---

## Section 5: April 28 Pre-Fix Pre-Condition (for FORGE Reference Card Section 3)

Add the following pre-condition to Section 3 of the April 28 Escalation Reference Card:

```
Pre-condition (Boys GA ASPIRE check):
  Run: SELECT g.gender, COUNT(DISTINCT g.id) AS game_count FROM games g
       JOIN events e ON e.id = g.event_id
       WHERE e.event_name ILIKE '%ASPIRE%' AND e.event_tier = 'ga'
       GROUP BY g.gender;

  If Boys game_count = 0: proceed — Boys not affected.
  If Boys game_count > 0: proceed — Boys GA ASPIRE cal = 100 (same as Boys GA).
                          No rating impact for Boys. Note count in session log.
```

This pre-condition confirms the decision from this document at runtime without requiring Presten to remember the reasoning during the session.

---

## Summary

**Boys GA ASPIRE calibration decision: 100 (Option A). Same as Boys GA. No Boys-specific modification to the April 28 UPDATE or to `compute-rankings.js` is needed.**

The April 28 GA ASPIRE fix is safe to run as written. The Boys existence check should be run and logged for the record, but regardless of whether Boys GA ASPIRE events exist, the reclassification has no net effect on Boys ratings.

Full Boys GA ASPIRE calibration validation (including whether ~70 is more appropriate) is deferred to the post-DSS Option B Brier analysis.

---

## References

- [[League Hierarchy]] — calibration values; Boys GA ASPIRE row updated 2026-04-24
- [[Ranking Engine]] — Phase 8 calibration map
- [[ga-aspire-tier-fix-2026-04-24]] — companion fix SQL (Section 3 references this decision)
- [[Boys Calibration Gap Analysis — April 2026]] — broader Boys calibration validation context
- `pending-2026-04-23-ga-calibration-review.md` — Presten approval for Girls GA ASPIRE fix
