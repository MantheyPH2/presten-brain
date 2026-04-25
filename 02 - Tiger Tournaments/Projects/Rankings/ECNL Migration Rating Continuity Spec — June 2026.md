---
title: ECNL Migration Rating Continuity Spec — June 2026
aliases: [ECNL Migration Continuity, ECNL Tier Migration Spec]
tags: [rankings, migration, ecnl, ecnl-rl, rating-continuity, calibration, evo-draw]
created: 2026-04-25
updated: 2026-04-25
author: ELO Agent
status: ready
task: task-2026-04-26-ecnl-migration-rating-continuity-spec
related: "[[ECNL Rating Continuity Spec — June 2026]]"
---

# ECNL Migration Rating Continuity Spec — June 2026

> **Scope:** This spec addresses the event-tier calibration impact of the June 1 ECNL migration — specifically, what happens to the `event_tier` tags on historical games when teams move from the ECNL platform to ECNL RL as their primary competition tier. This is a companion to [[ECNL Rating Continuity Spec — June 2026]], which covers rating continuity through the `team_merges` verified-flag mechanism. That spec is prerequisite reading.
>
> **ELO recommendation is in Section 3.** SENTINEL should read Sections 2 and 3 only for the decision. Sections 1, 4, 5 are implementation detail.

---

## Section 1: Taxonomy Clarification

Three distinct team populations are affected differently by the June 1 migration:

### Path A — ECNL teams migrating to ECNL RL
Teams that competed in ECNL events (tagged `event_tier = 'ecnl'`, cal=120 Boys / cal=130 Girls) and will compete in ECNL RL events (tagged `event_tier = 'ecnl_rl'`, cal=55) going forward. Their historical game records are tagged `ecnl`. Their new post-June-1 game records will be tagged `ecnl_rl`.

**How to identify Path A teams:** Teams whose `canonical_team_id` in `team_merges` has `method = 'ecnl_transition'` AND whose pre-migration game history contains games with `event_tier = 'ecnl'`.

Confirmation query:
```sql
SELECT DISTINCT t.name, t.id AS team_id,
  COUNT(g.id) AS ecnl_games_historical
FROM teams t
JOIN team_merges tm ON (tm.canonical_team_id = t.id OR tm.merged_team_id = t.id)
JOIN games g ON (g.home_team_id = t.id OR g.away_team_id = t.id)
JOIN events e ON e.id = g.event_id
WHERE tm.method = 'ecnl_transition'
  AND e.event_tier = 'ecnl'
GROUP BY t.name, t.id
ORDER BY ecnl_games_historical DESC;
```

### Path B — Teams already in ECNL RL (no change)
Teams that competed only in ECNL RL events before June 1. Their game history is tagged `ecnl_rl`. No migration impact. No tier change. Ratings continue without interruption.

### Path C — ECNL teams that do not continue in ECNL RL
Teams that were in ECNL but will not compete in ECNL RL post-migration (exit ranked pool, disband, or drop to a lower tier). Their historical `ecnl` games remain in the DB but no new games will arrive. Their ratings will decay via Glicko RD growth over time — this is correct behavior.

**These teams do not require any special handling.** The ranking engine naturally handles inactive teams.

---

## Section 2: Rating Impact Analysis

### Calibration value delta

| Path | Historical game tier | Historical cal (Boys) | Historical cal (Girls) | Post-June-1 game tier | Post-June-1 cal |
|------|---------------------|----------------------|----------------------|----------------------|----------------|
| Path A | `ecnl` | 120 | 130 | `ecnl_rl` | 55 |
| Path B | `ecnl_rl` | 55 | 55 | `ecnl_rl` | 55 |

The calibration gap for Path A teams is **65 points (Boys)** and **75 points (Girls)** between historical and future tiers.

### Estimated rating impact if historical games were re-tagged

If all historical `ecnl` games for Path A teams were re-tagged to `ecnl_rl` (Option 2), the rating impact would be determined by:
- **Direction:** Downward. The calibration value acts as a multiplier on outcome-driven rating updates. Lower cal → smaller per-game rating gain from wins, smaller loss from defeats. Re-tagging historical wins to cal=55 from cal=120/130 would reduce the cumulative rating earned from those games.
- **Magnitude (estimate):** An average ECNL team with 20 historical ECNL games and a 55% win rate earned approximately 65–80 rating points from those games at cal=120–130 compared to what they would have earned at cal=55. Re-tagging would reduce their rating by an estimated **40–80 points** for a typical Path A team. Top ECNL teams with 30+ games and high win rates could see drops of 100+ points.
- **DSS visibility (May 16):** Migration occurs June 1 — after DSS. Any re-tagging decision is a post-DSS implementation. No impact on DSS demo.

### Rating convergence between Path A and Path B

After June 1, Path A teams (ex-ECNL, now in ECNL RL) and Path B teams (always in ECNL RL) play in the same events. Over time:
- Path A teams enter ECNL RL with higher ratings (earned from higher-calibration historical games).
- Path B teams who have been competing at ECNL RL level have ratings calibrated purely at cal=55.
- The gap will compress naturally as Path A accumulates ECNL RL games (cal=55) under recency weighting.

**Under Option 1 (no re-tagging):** The seam is visible for approximately 6–12 months post-migration. Path A teams will appear 20–50 points higher than equivalent-performance Path B teams during this window. This reflects real performance history: Path A teams genuinely competed at a higher level historically. The seam is directionally correct, not a calibration error.

**Under Option 2 (re-tag to ecnl_rl):** Convergence is immediate but achieved by erasing the historical quality signal. Path A teams at 1,450 (Gold) before migration could drop to 1,370 (Silver) overnight — a visible and potentially unfair rating reset for teams that earned those ratings through genuine ECNL competition.

---

## Section 3: Decision and Recommendation

### ELO Recommendation: Option 1

**Keep historical `ecnl` games tagged as `ecnl`. New post-June-1 games tagged `ecnl_rl`. No re-tagging, no recompute.**

**Reasoning:**

1. **Historical games reflect real competition quality.** ECNL is cal=120–130; ECNL RL is cal=55. A Path A team's historical ECNL games were played against ECNL-level competition. Tagging those games at cal=55 misrepresents the quality of competition those games involved. The higher calibration value correctly captures that those games were more meaningful.

2. **Option 2 creates a visible, unfair rating cliff.** A 40–80 point drop on June 1 for teams that legitimately earned those ratings through ECNL competition would be indefensible to club directors. If a director asks why their team dropped from Gold to Silver overnight when they haven't changed how they play, there is no good answer under Option 2.

3. **The seam under Option 1 is defensible.** "Path A teams carry forward the ratings they earned — their ECNL history is part of who they are. As they accumulate ECNL RL results, their rating reflects the blend of both eras. By fall 2027, the ECNL history will have recency-weighted out and ratings will converge." This is accurate and explains the transition correctly.

4. **Option 3 (ecnl_legacy) is unnecessary complexity.** Introducing a new event tier for pre-migration games adds DDL changes, a new cal value to validate, and a calibration decision ELO has no cross-league data to support. Option 1 achieves the same outcome without any schema changes.

### Option 1 behavior summary

| Aspect | Behavior |
|--------|----------|
| Historical ECNL games | Stay tagged `ecnl`, continue contributing at cal=120/130 |
| New ECNL RL games (post-June 1) | Tagged `ecnl_rl`, contribute at cal=55 |
| Rating continuity | Preserved (via [[ECNL Rating Continuity Spec — June 2026]] verified-flag mechanism) |
| Path A team ratings on June 2 | Unchanged from June 1 pre-migration |
| Convergence to Path B teams | Natural — recency weighting compresses gap over 6–12 months |
| DSS impact | None — migration is post-DSS |
| Recompute required | No — only the regular post-migration recompute per the June 2 verification spec |

### Conditions under which ELO would revise this recommendation

If, after the June 2 verification run, analysis shows that Path A teams are appearing 150+ points above equivalent Path B teams in the same age group, ELO will reassess. At that magnitude, the seam becomes a calibration anomaly rather than a defensible historical signal. ELO will file a revised recommendation in the next briefing if this threshold is crossed.

---

## Section 4: SQL Requirements

Under Option 1, **no UPDATE SQL is required** for tier re-tagging. The migration is handled entirely through the `team_merges` mechanism (documented in [[ECNL Rating Continuity Spec — June 2026]]).

### Verification query (run post-migration, before recompute)

Confirm that no ecnl_transition-involved team's historical games have been inadvertently re-tagged:

```sql
-- Confirm historical ecnl games for Path A teams are still tagged ecnl
SELECT e.event_tier, COUNT(g.id) AS game_count
FROM games g
JOIN events e ON e.id = g.event_id
JOIN teams t ON (g.home_team_id = t.id OR g.away_team_id = t.id)
JOIN team_merges tm ON (tm.canonical_team_id = t.id OR tm.merged_team_id = t.id)
WHERE tm.method = 'ecnl_transition'
  AND e.event_tier IN ('ecnl', 'ecnl_rl')
GROUP BY e.event_tier;
```

**Expected result post-migration:** Two rows — one for `ecnl` (historical games) and one for `ecnl_rl` (new games). If `ecnl` row disappears, historical games were re-tagged inadvertently. Investigate before running recompute.

### New game tier verification (run 2 weeks post-migration)

Confirm that new games being ingested after June 1 for migrating teams receive `ecnl_rl` tier:

```sql
-- Post-June-1 games for Path A teams should be ecnl_rl
SELECT e.event_tier, COUNT(g.id) AS new_game_count
FROM games g
JOIN events e ON e.id = g.event_id
JOIN teams t ON (g.home_team_id = t.id OR g.away_team_id = t.id)
JOIN team_merges tm ON (tm.canonical_team_id = t.id OR tm.merged_team_id = t.id)
WHERE tm.method = 'ecnl_transition'
  AND g.match_date >= '2026-06-01'
  AND e.event_tier IN ('ecnl', 'ecnl_rl')
GROUP BY e.event_tier;
```

**Expected result:** All post-June-1 games for Path A teams should be tagged `ecnl_rl`. If any post-June-1 games are tagged `ecnl`, the event classifier has not been updated for the new ECNL RL events.

---

## Section 5: Sequencing

### Timing

| Action | When | Owner |
|--------|------|-------|
| This spec reviewed by SENTINEL | Before May 16 DSS demo | SENTINEL |
| FORGE confirms event classifier updated for ECNL RL events post-June-1 | Before June 1 | FORGE |
| Migration script runs (ecnl_transition records inserted with verified=true) | June 1 | FORGE |
| Verification query (Section 4) run before recompute | June 2 AM | Presten |
| Full ranking recompute (includes Check 6 from [[ECNL Rating Continuity Spec — June 2026]]) | June 2 | Presten |
| Post-migration distribution check (Path A vs Path B gap measurement) | June 7–14 | ELO |
| ELO recommendation revision if gap > 150 pts | June 14 | ELO → SENTINEL |

### SENTINEL authorization gate

This spec requires **SENTINEL review and acknowledgment** before June 1. The recommendation (Option 1) has no code changes, no schema changes, and no recompute outside the normal post-migration flow. SENTINEL authorization is confirmation that the "no re-tagging" decision is accepted.

### Presten sign-off

Not required for Option 1 (no schema changes, no UPDATE outside normal migration). Presten should be aware that Path A teams will carry forward their ECNL history and may appear 20–50 points above equivalent Path B teams in the transition window. This is expected behavior, not a bug.

If SENTINEL overrides to Option 2 or Option 3, Presten sign-off is required before any UPDATE runs, as those options affect 1,000+ team ratings.

---

## References

- [[ECNL Rating Continuity Spec — June 2026]] — verified-flag mechanism; prerequisite reading for this spec
- [[June 2 ECNL Migration Verification Spec]] — post-migration check protocol (Check 6 closes the residual risk)
- [[June 1 Risk Assessment — Age Group Transition + ECNL Migration]] — full migration risk context
- [[League Hierarchy]] — ECNL cal=120 (Boys) / 130 (Girls); ECNL RL cal=55
- `02 - Tiger Tournaments/Projects/Product & Planning/DSS Feature Readiness Tracker — May 2026.md` — migration items not in scope for DSS (post-DSS)

*ELO — 2026-04-25*
