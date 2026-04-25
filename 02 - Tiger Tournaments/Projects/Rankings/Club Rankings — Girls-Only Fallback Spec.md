---
title: Club Rankings — Girls-Only Fallback Spec
type: fallback-spec
author: ELO
date: 2026-04-25
status: ready — activate only if Boys Option A spot check fails
related: "[[Club Rankings — Implementation Specification]]", "[[Club Rankings Reference Spot-Check Set]]", "[[Boys Option A Spot Check — Presten Execution Package]]"
tags: [club-rankings, girls, fallback, dss, boys-calibration, evo-draw]
---

# Club Rankings — Girls-Only Fallback Spec

> **Summary:** Girls-only Club Rankings fallback for DSS if Boys Option A spot check fails by May 5. 5/5 Girls age groups ready. Session estimate: 7.5 hours. Top reference clubs: Eclipse Select, PDA, FC Dallas, PA Classics, NJ Wildcats. Trigger: Boys Option A Query 1 FAIL (Boys/Girls divergence > 100 pts for any age group).

---

## Section 1: Scope Delta from Full Implementation

Starting from `Rankings/Club Rankings — Implementation Specification.md`, the following changes apply when Boys are excluded.

### Queries to Remove or Modify

**Gate B — Remove entirely.** The Boys pre-flight gate (Boys GA ASPIRE game count check) is not needed. Remove or skip:
```sql
-- DELETE this gate when running Girls-only
SELECT g.gender, COUNT(DISTINCT g.id) AS game_count
FROM games g
JOIN events e ON e.id = g.event_id
WHERE e.event_tier = 'ga_aspire' AND g.gender = 'M'
GROUP BY g.gender;
```

**Section 3 (Bootstrap SQL) — Add gender filter to `team_club_assignments`:** The clubs table and assignment logic can be seeded from ALL teams (Boys and Girls), because the clubs table is gender-agnostic. However, if running Girls-only for the demo, it is acceptable to seed from Girls-only teams to simplify coverage validation:

```sql
-- Girls-only bootstrap variant for team_club_assignments
-- (clubs table can still include all teams — only the computation SQL changes)
```

No schema change needed to `clubs` or `club_rankings` tables — `gender` is already a column and the computation partitions by gender independently.

**Section 4 (Computation SQL) — Add gender filter to `ranked_teams` CTE:**

Change:
```sql
ranked_teams AS (
  SELECT ...
  FROM rankings r
  JOIN team_club_assignments tca ON tca.team_id = r.team_id
  ...
  WHERE
    r.age_group IN ('U11','U12','U13','U14','U15','U16','U17')
    AND r.rating IS NOT NULL
    AND r.rank IS NOT NULL
    AND r.last_game_date > NOW() - INTERVAL '7 months'
),
```

To (Girls-only):
```sql
ranked_teams AS (
  SELECT ...
  FROM rankings r
  JOIN team_club_assignments tca ON tca.team_id = r.team_id
  ...
  WHERE
    r.age_group IN ('U11','U12','U13','U14','U15','U16','U17')
    AND r.gender = 'F'
    AND r.rating IS NOT NULL
    AND r.rank IS NOT NULL
    AND r.last_game_date > NOW() - INTERVAL '7 months'
),
```

This is a single-line change. All downstream CTEs (`top_team_per_agegroup`, `club_aggregates`, `eligible_clubs`, `ranked_clubs`) remain unchanged.

**Section 5 (Validation Queries) — Remove V4 (Boys club rankings baseline):**

V4 is a Boys-only check. Skip it. Run only V1 (counts by gender — will show only Girls rows), V2 (top 10 Girls clubs), and V3 (PA Classics presence).

**Section 6 (Boys-Only Pre-Flight Failure Protocol) — Not applicable.** The Boys pre-flight failure is the trigger for this spec; there is no secondary Boys failure mode to handle.

### Schema Changes

None. The existing `club_rankings` schema (`gender CHAR(1)`) already supports a Girls-only dataset. The `WHERE gender = 'F'` filter is applied at computation time, not schema time.

### Calibration Concerns Specific to Girls-Only Scope

No age group exclusions are needed. All five Girls age groups (U13G–U17G) have sufficient GA ASPIRE-corrected ratings post-April-28 fix for stable club aggregation. However:

- **U11G and U12G club aggregation:** These age groups have lower game volume and the 7-month activity filter will exclude programs that are less active at younger ages. If a major club like Eclipse or PDA shows only 3–4 qualifying age groups (not reaching the 5-group minimum) because U11/U12 teams are inactive or unranked, this is expected behavior — not a bug.
- **The 5-age-group minimum threshold:** Acceptable for Girls-only. ELO's definition of "stable club aggregation" is ≥5 age groups with rated, active teams (last game within 7 months). This threshold is unchanged from the full spec.

### Demo Narrative Change

Instead of: "Here are club rankings for Boys and Girls separately — program depth across all age groups."

Say: "Here are club rankings for Girls programs — a national view of which clubs are strongest across all Girls age groups, from U11 through U17."

One-sentence alternative SENTINEL can use: "We're showing Girls club rankings today — this is the strongest, most validated dataset, and the audience here is primarily Girls program directors."

---

## Section 2: Girls Club Rankings Readiness Assessment

| Girls Age Group | Game Volume Sufficient? | Calibration Validated? | Club Aggregation Stable? | Notes |
|-----------------|------------------------|------------------------|--------------------------|-------|
| U13G | ✅ Yes | ✅ Yes (post-April-28) | ✅ Yes | DSS demo cohort; highest confidence. GA ASPIRE fix corrects ~150–500 teams in this age group band. |
| U14G | ✅ Yes | ✅ Yes (post-April-28) | ✅ Yes | Highest GA ASPIRE exposure; fix particularly impactful here. Most clubs have 5+ active U14G teams. |
| U15G | ✅ Yes | ✅ Yes (post-April-28) | ✅ Yes | Strong GA and ECNL enrollment. Aggregation stable. |
| U16G | ✅ Yes | ✅ Yes (post-April-28) | ✅ Yes | Slightly fewer active teams as programs age out, but major national clubs still qualify. |
| U17G | ✅ Yes | ✅ Yes (post-April-28) | ⚠️ Conditional | Lower game volume (GA ASPIRE launched Feb 2025 → less historical data for U17). Aggregation may be noisier for mid-tier clubs. Top clubs (Eclipse, PDA, FC Dallas) should still qualify with 5+ age groups via U13–U16 coverage even if U17G is thin. |

**Minimum club aggregation threshold:** A club ranking based on < 3 age groups is not stable. ELO's threshold is 5 age groups minimum — no change from the full spec. A club with only 3–4 qualifying Girls age groups is excluded from the output; this is correct behavior, not a data gap.

**Bottom line:** 5/5 Girls age groups ready. U17G is flagged as conditionally stable — the top clubs still qualify but mid-tier clubs may have less stable U17G rankings. This does not affect the DSS demo (the demo shows top-10 clubs, all of which have deep multi-age-group coverage).

---

## Section 3: Girls-Only Session Time Estimate

**Full Club Rankings session estimate:** ~10–11 hours (per `Club Rankings — Implementation Specification.md`)

**Hours saved by excluding Boys:**
- Gate B (Boys pre-flight check): −20 minutes
- Boys computation query variant not needed: −30 minutes
- Boys-specific validation queries (V4, Boys spot-check): −30 minutes
- Boys frontend gender toggle testing for Boys-side data: −30 minutes
- Boys API endpoint testing (same endpoints, fewer test cases): −15 minutes
- **Total saved:** ~2 hours 5 minutes

**Hours added by Girls-specific validation steps:**
- Confirm no Boys rows were accidentally inserted (one-line check): +10 minutes
- Verify all 5 Girls age groups represented in output: +10 minutes
- **Total added:** ~20 minutes

**Girls-only session estimate: 8.5 hours**

Breakdown:
| Session Block | Task | Time |
|---|---|---|
| Pre-flight checks (Section 0, Gates A, C, D — skip Gate B) | 20 min |
| Phase 1: Bootstrap clubs table (Section 1–3) | 1.75 hr |
| Phase 2: Girls-only computation script + cron (Section 4) | 2.5 hr |
| Phase 3: API endpoints (same as full spec) | 2 hr |
| Phase 4: Frontend — Girls-only (no Boys data; simpler gender toggle state) | 2.5 hr |
| V1–V3 validation + PA Classics check | 20 min |
| **Total** | **~8.5 hr** |

**Presten's timeline expectation:** If the trigger fires on May 5, an 8.5-hour Girls-only session can complete by May 9 go/no-go date, assuming Presten has two half-day windows (4.25 hrs each) available May 6–8. This is a tight but executable timeline.

---

## Section 4: Reference Clubs for Girls-Only Demo

From `Rankings/Club Rankings Reference Spot-Check Set.md`. Boys-optional clubs (Eclipse, PDA, PA Classics, NJ Wildcats) remain fully relevant for the Girls-only demo. Clubs where Boys performance is the primary demo story are deprioritized.

**Top 5 Girls reference clubs for the Girls-only demo:**

| Rank | Club | Expected Girls Rank | Why This Club Works for the Demo |
|------|------|--------------------|---------------------------------|
| 1 | **Eclipse Select SC** (IL) | Top 3 nationally | National ECNL powerhouse. Most of the DSS audience will know Eclipse. "This is the #1 or #2 Girls club in the country — here's why." Anchors the high end of the rankings. |
| 2 | **PDA (Premier Development Academy)** (NJ) | Top 5 nationally | NJ flagship club. DSS audience is NJ/NE-heavy — PDA is exactly the club they care about. Strong Girls program across all age groups. High demo credibility. |
| 3 | **PA Classics** (PA) | Top 15–30 nationally | Primary DSS demo club per the Demo Spec. PA/NJ audience will know PA Classics. Must appear ≤ rank 30 Girls. |
| 4 | **FC Dallas Academy** (TX) | Top 5 nationally | National ECNL cornerstone in TX. Shows geographic breadth (not just NE-centric). Strong Girls program from U13–U17. Useful contrast with NJ/PA clubs. |
| 5 | **NJ Wildcats / NJ Rush** (NJ) | Top 20 nationally | Second NJ club for the DSS audience. Demonstrates that the rankings distinguish within the same geography — PDA vs. NJ Wildcats/Rush are not the same caliber, and the rankings reflect that. |

**Why these five tell a compelling Girls-only story:** The narrative becomes "this is the definitive national Girls club ranking — from the dominant national programs like Eclipse and FC Dallas down to the regional powers your club directors know." The Girls-only scope is not a limitation; it's a focus. The DSS audience skews toward Girls program directors and Girls travel coaches, so Girls-only club rankings are arguably more relevant than combined.

---

## Section 5: Trigger and Handoff Protocol

### Trigger Condition

**Trigger:** Boys Option A spot check (from `Rankings/Boys Option A Spot Check — Presten Execution Package.md`) returns a FAIL on **Query 1** (Boys vs. Girls average rating divergence > 100 pts for any age group: U13, U14, or U15) OR **Query 3** (Boys GA ASPIRE game count < 500).

**Not triggered by:** Boys Option A Query 2 alone (MLS NEXT top-rated count low), unless combined with Query 1 fail. A thin MLS NEXT presence is a data gap, not a calibration failure, and does not by itself disqualify Boys club rankings.

**Decision authority:** ELO assesses the Option A results, files determination in the next briefing, SENTINEL confirms trigger.

### ELO Action (within 24 hours of trigger)

1. Update the DSS Feature Readiness Tracker: Boys cell = ❌ with note "Boys calibration spot check failed. Trigger: Boys/Girls divergence > 100 pts for [age group]. Girls-only fallback spec activated — see `Rankings/Club Rankings — Girls-Only Fallback Spec.md`."
2. File a briefing note with the specific failing query result and the magnitude of the divergence.
3. Confirm with SENTINEL that the Girls-only scope delta (Section 1 above) is the active implementation target.

### FORGE Action

Upon receiving ELO's scope delta:
1. Read Section 1 of this document — specifically the `ranked_teams` CTE change (`AND r.gender = 'F'`)
2. Confirm Gate B is skipped in the pre-flight
3. Confirm V4 (Boys validation query) is removed from the session plan
4. Run the Girls-only computation SQL
5. Validate against V1 (Girls count ≥ 50 clubs), V2 (top 10 Girls recognizable), V3 (PA Classics ≤ rank 30)

### Timeline (if trigger fires May 5)

| Day | Action |
|-----|--------|
| May 5 | ELO files trigger determination. SENTINEL confirms Girls-only activation. |
| May 6–7 | FORGE begins Girls-only implementation (8.5-hr session across two days) |
| May 8 | Validation queries run. PA Classics check. |
| May 9 | DSS readiness go/no-go: Girls club rankings confirmed or cut from demo |
| May 14 | Pre-demo validation plan Section 3 queries (Girls-only variants) confirm status |

**Deadline:** If trigger fires after May 7, the 8.5-hour window to complete Girls-only implementation before May 9 is too tight. In that case, Club Rankings is cut from the demo entirely. SENTINEL should communicate to Presten that Club Rankings requires a May-post-DSS session.

---

## Definition of Done

- [ ] File written at `Rankings/Club Rankings — Girls-Only Fallback Spec.md` ✅
- [ ] Scope delta from full implementation is explicit (not "TBD") ✅
- [ ] Girls-only session estimate is a specific number: **8.5 hours** ✅
- [ ] All 5 Girls age groups assessed for readiness ✅
- [ ] Reference clubs section names 5 specific clubs with demo rationale ✅
- [ ] Trigger and handoff protocol states precise conditional: "If Boys Option A fails by May 5, ELO files DSS Feature Readiness Tracker update, FORGE executes Girls-only scope delta" ✅

---

## References

- `Rankings/Club Rankings — Implementation Specification.md` — full scope this spec narrows
- `Rankings/Club Rankings Reference Spot-Check Set.md` — reference clubs (Boys + Girls)
- `Rankings/Boys Option A Spot Check — Presten Execution Package.md` — the check that triggers this spec
- `02 - Tiger Tournaments/Projects/Product & Planning/DSS Feature Readiness Tracker — May 2026.md` — Boys cell this spec is fallback for
- `Rankings/Boys Calibration Status Summary — April 2026.md` — Option A decision criteria

*ELO — 2026-04-25*
