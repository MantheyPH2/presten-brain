---
title: ECNL Migration — Option B Technical Risk Deep Dive
type: risk-analysis
author: ELO
date: 2026-04-27
status: ready — informs April 29 option selection gate
related: "[[ECNL Migration — Option Comparison Matrix]]", "[[ECNL Migration — Option Selection Decision Gate]]", "[[ECNL Rating Continuity Spec — June 2026]]"
tags: [ecnl, migration, option-b, risk, rating-continuity, evo-draw]
---

# ECNL Migration — Option B Technical Risk Deep Dive

> **Purpose:** Pre-gate stress test of Option B (re-tag historical `ecnl` games to `ecnl_rl` + full recompute) before ELO selects it at the April 29 CP1/CP2 gate. This document does not replace the Option Comparison Matrix — it extends it by fully characterizing Option B's failure modes so the gate decision is made with complete information.
>
> **Written:** 2026-04-27. **Gate decision:** 2026-04-29 (if CP1 passes, Option B is the likely selection). **Migration date:** June 1, 2026.

---

## Section 1: Option B Summary (Reference Only)

Option B re-tags all historical `ecnl` game records for Path A teams (teams migrating from TGS to GotSport on June 1) to `event_tier = 'ecnl_rl'`, then runs a full `compute-rankings.js` recompute. This eliminates the calibration seam between pre- and post-migration games: both old and new games are processed at the `ecnl_rl` cal value (approximately 55–65 Boys, 65–75 Girls, based on the 40–80 pt cliff description in the Option Comparison Matrix). FORGE executes the mass UPDATE query against `team_merges` and `event_tiers`; ELO validates post-recompute ratings. Full technical definition in `Rankings/ECNL Rating Continuity Spec — June 2026.md`. Option B is ELO's second-preference path; Option 1 (no re-tag) is preferred if CP1 passes.

---

## Section 2: Compounding Risk — Simultaneous ID Change + Age Group Transition

**This is the highest-priority failure mode for Option B.**

### The Scenario

On June 1, some ECNL Path A teams will simultaneously experience two identity changes:
1. **Platform ID change:** TGS ID → GotSport ID (the ECNL migration event)
2. **Age group transition:** U13 → U14, U14 → U15, etc. (the annual school-year advance)

For these teams, Option B must:
- Map the old TGS ID to the new GotSport ID (via `ecnl_transition` merge record)
- Re-tag the old `ecnl` game records to `ecnl_rl`
- Assign the resulting rating to the *new* age group entry, not the *old* age group entry

### Can the re-mapping algorithm handle dual-change teams correctly?

**Short answer: Yes, but only if the merge record and the re-tag are executed in the correct order.**

The Elo computation resolves merges in Phase 1, before any game processing. If the `ecnl_transition` merge record maps `merged_team_id = old_TGS_id → canonical_team_id = new_GotSport_id`, Phase 1 correctly routes all game history (including the freshly re-tagged `ecnl_rl` games) to the GotSport canonical ID.

The age group transition is handled separately by `age_group_advance()`, which runs after `compute-rankings.js` assigns the post-recompute rating to the team's current age group. The age group field on the team record (not the game record) determines the bucket — so the sequence that works is:

```
Step 1: Write ecnl_transition merge records (verified = true)
Step 2: Run re-tag UPDATE (ecnl → ecnl_rl for Path A teams)
Step 3: Confirm age_group_advance() will fire on the team's NEW age group
Step 4: Run compute-rankings.js (Phase 1 loads merge; Phase 3 processes ecnl_rl games for GotSport canonical ID)
Step 5: age_group_advance() runs; team is bucketed into new age group with its correctly re-tagged, continuity-preserved rating
```

**Failure mode:** If `age_group_advance()` fires BEFORE `compute-rankings.js` completes the recompute, the team's age group label changes mid-computation. Phase 3 would attribute the team's games to, say, U14 while the re-tag just ran on U13 game records. The resulting rating would be assigned to the U14 bucket with U13 game history — not wrong in isolation, but it creates attribution ambiguity in post-migration validation queries that filter by age group.

**Mitigation:** FORGE's June 1 session script must confirm that `age_group_advance()` runs AFTER the full Option B recompute cycle completes. This is the same ordering constraint in the June 1 Risk Register (R-series). Add an explicit step to the ECNL Migration Pre-Flight Checklist: "Confirm `age_group_advance()` is scheduled AFTER Option B recompute confirms 0 rating orphans via Check 6."

**Visibility to a demo user:** If the ordering is inverted and a U14 team is shown with a U13-range rating because its game history was mis-bucketed, it would appear as an outlier in the Club Rankings output (a top ECNL club showing in the bottom 25% of U14G). This would be visible at DSS.

---

## Section 3: Incomplete Re-Mapping Coverage Risk

### What Happens to Unmapped Teams (Coverage < 100%)?

If FORGE's `ecnl-transition-dedup.js` produces ID mappings for only 80% of Path A teams, the unmapped 20% are NOT covered by the `ecnl_transition` merge records. For these teams:

- The Phase 1 merge map has no entry for their old TGS ID → new GotSport ID
- Their old game records (now re-tagged to `ecnl_rl` by the mass UPDATE) are attributed to the OLD TGS ID
- The NEW GotSport canonical ID has no game history — it initializes at ~1000

**This is a rating reset, not a rating cliff.** Option B's 40–80 pt cliff assumes the re-tagging works and the rating is continuous — just at a lower level. An unmapped team gets a 300–500 pt drop (from a rated Silver/Gold team to provisional). This is far more visible than the intended cliff.

The re-tag UPDATE applies to games regardless of whether the team has a mapping. So if a team is unmapped, the result is: old games re-tagged (now orphaned under the dead TGS ID) + new GotSport ID initializes at 1000. Option B's incomplete coverage failure is structurally worse than Option 1's incomplete coverage failure.

### Minimum Viable Coverage

**ELO's minimum viable coverage threshold for Option B: ≥ 95% of the top-500 ECNL teams must have confirmed ID mappings before the re-tag UPDATE is authorized.**

Rationale: The top-500 ECNL teams are the teams most likely to appear in the Club Rankings demo and in the rank distributions SENTINEL shows Presten. An unmapped team at rank 1,450 (mid-Silver) is noise; an unmapped team at rank 1,650 (Gold) is a demo liability.

If FORGE can confirm:
- Mapping coverage ≥ 95% for all ECNL teams ranked above 1,400 in any age group
- Mapping coverage ≥ 90% overall across all ECNL Path A teams

...then Option B is viable from a coverage standpoint.

**If coverage is 80–94%:** ELO recommends deferring Option B. Run Option 1 (no re-tag) instead, even if CP1 condition is borderline. A 6–20% rating-reset rate on ECNL teams is unacceptable for DSS.

### Detection Query for Incomplete Re-Mapping (Post-June-1)

```sql
-- Detect ECNL teams with ecnl_transition merges showing near-initialization ratings
-- (Adapted from ECNL Rating Continuity Spec Check 6)
SELECT
  t.name,
  r.rating::int AS current_rating,
  r.age_group,
  r.gender,
  COUNT(tm.id) AS transition_merges
FROM rankings r
JOIN teams t ON t.id = r.team_id
JOIN team_merges tm ON (tm.canonical_team_id = r.team_id OR tm.merged_team_id = r.team_id)
WHERE tm.method = 'ecnl_transition'
  AND r.rating BETWEEN 900 AND 1100
GROUP BY t.name, r.rating, r.age_group, r.gender
ORDER BY r.rating;
-- Pass: 0 rows (or only teams that were provisionally rated pre-migration)
-- Fail: any team with pre-migration rating > 1300 now showing < 1100
```

This query should return 0 rows if Option B was executed with ≥ 95% coverage. Any result is an incomplete-mapping failure.

---

## Section 4: Rating Continuity Risk Under Option B

### Expected Rating Shift with Perfect ID Re-Mapping

Even with perfect ID re-mapping, Option B introduces a deliberate rating cliff. The magnitude depends on:
- How many of the team's historical games are `ecnl` (now re-tagged to `ecnl_rl`)
- The calibration delta between `ecnl` and `ecnl_rl` values

From the Option Comparison Matrix: "a one-time 40–80 pt rating cliff." This is not a bug — it is the intended behavior of Option B. Teams that played in ECNL (high cal) will have their historical game contributions recomputed at the lower `ecnl_rl` cal value. Teams with many ECNL games will shift more; teams with few ECNL games will shift less.

**Expected shift by exposure:**
| ECNL Game Share | Expected Rating Shift (Option B) |
|-----------------|----------------------------------|
| ≥ 75% of games ECNL | 60–80 pts downward |
| 50–74% of games ECNL | 35–60 pts downward |
| 25–49% of games ECNL | 15–35 pts downward |
| < 25% of games ECNL | < 15 pts — minimal |

**Glicko RD decay risk:** A team that played its last TGS game 4+ months before June 1 will have RD expansion applied by the recompute (Glicko parameter: RD expands over time when no new games are processed). After Option B re-tag, this team's rating is computed with expanded RD — meaning its historical ECNL games carry less weight in the computation than they would have when the games were fresh. The net effect: the Option B cliff is partially offset by RD-driven attenuation for teams with stale game histories. This is expected and correct behavior, not a failure mode.

**Monitoring Signal Post-June-1:**

```sql
-- Confirm rating continuity for re-mapped Option B teams
-- Compare post-migration ratings against pre-migration snapshot (from May 31 backup)
-- Expected: shift consistent with ECNL game share (40-80 pts for high-exposure teams)
-- Unexpected: any team with ECNL exposure > 50% showing < 10 pt shift (re-tag may not have applied)
-- OR any team showing > 120 pt shift (re-tag scope exceeded expected ECNL game count)

SELECT
  t.name,
  r_pre.rating AS pre_migration_rating,
  r_post.rating AS post_migration_rating,
  (r_post.rating - r_pre.rating)::int AS delta,
  r_post.age_group
FROM rankings r_post
JOIN teams t ON t.id = r_post.team_id
JOIN rankings_snapshot_20260531 r_pre ON r_pre.team_id = r_post.team_id
JOIN team_merges tm ON tm.canonical_team_id = r_post.team_id
WHERE tm.method = 'ecnl_transition'
ORDER BY ABS(r_post.rating - r_pre.rating) DESC
LIMIT 50;
-- Expected: deltas cluster in -40 to -80 range for high-ECNL-exposure teams
-- Flag: any delta outside -120 to +10 range requires investigation
```

---

## Section 5: Option B Recommendation

**PROCEED with Option B + Conditions**

Option B's risks are manageable, provided the following conditions are confirmed before the June 1 re-tag UPDATE is authorized:

| Condition | Threshold | Check |
|-----------|-----------|-------|
| FORGE ID mapping coverage — top 500 ECNL teams | ≥ 95% confirmed | FORGE delivers mapping report before Option B authorized |
| FORGE ID mapping coverage — all ECNL Path A teams | ≥ 90% overall | Same mapping report |
| `age_group_advance()` ordering | Runs AFTER Option B recompute | FORGE confirms in ECNL Migration Pre-Flight Checklist |
| April 29 G2 gate | PASS (calibration stable) | G2 gate result — if FAIL, Option B is held per Option Comparison Matrix disqualification rule |
| Presten written approval of the rating cliff | Written sign-off on 40–80 pt cliff for ECNL teams | Per Option Comparison Matrix: required if Path A team count > 1,000 |

**If all conditions are met:** Option B can be authorized at the CP7 gate (May 10) for June 1 execution.

**If FORGE mapping coverage is 80–94%:** Escalate to SENTINEL. Coverage below 95% for top-500 teams means high-visibility ECNL teams may reset to ~1000, which is a DSS-level liability. Consider Option 1 if CP1 passes; or extend FORGE mapping effort to push coverage above 95% threshold by May 28.

**If April 29 G2 FAIL (calibration unstable):** Option B is automatically held per the Option Comparison Matrix. Do not proceed until G2 clears.

**Note for the Option Selection Decision Gate:** After filing this document, ELO will update `Rankings/ECNL Migration — Option Selection Decision Gate.md` to reference this deep dive in the Option B row, specifically adding the 95% coverage condition and the `age_group_advance()` ordering requirement as named gate conditions.

---

## References

- `Rankings/ECNL Migration — Option Comparison Matrix.md` — Option B definition, disqualification rules, CP matrix
- `Rankings/ECNL Migration — Option Selection Decision Gate.md` — gate document this analysis feeds into
- `Rankings/ECNL Rating Continuity Spec — June 2026.md` — `verified` flag mechanics, Phase 1 merge loading, Check 6 query
- `Rankings/June 2 ECNL Migration Verification Spec.md` — Check 2B and Check 6 as post-migration validators
- `Rankings/June 1 Age Transition — Risk Register.md` — `age_group_advance()` ordering constraints

*ELO — 2026-04-27*
