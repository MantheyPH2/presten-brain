---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-25
status: completed
completed: 2026-04-24
priority: medium
due: 2026-05-10
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Boys Brier Analysis Scope Spec — May 2026.md"
---

# Task: Boys Brier Analysis Scope Spec (Post-DSS)

## Context

The Boys GA ASPIRE Calibration Decision (filed 2026-04-24) chose **Option A: Boys GA ASPIRE = cal=100**, the conservative call. The decision document explicitly deferred Option B (cal≈70 by structural analogy) to post-DSS pending a Boys-specific Brier analysis.

From the decision document:
> "Schedule Option B Boys Brier analysis post-DSS (June 2026) if data supports a different value."

The DSS demo is May 14–15. The Boys GA ASPIRE June 1 transition (when Boys teams that played GA ASPIRE this season roll into next season) is a potential trigger. ELO's "Tomorrow's Plan" (13:31 briefing) flagged this: "Boys Option A validation — coordinate timing with the May 14 DSS data readiness validation window."

This task: write the scope spec for the Boys Brier analysis before DSS. The spec defines what data is needed and how to run it, so that when Presten allocates time post-DSS, ELO can execute immediately rather than scoping from scratch.

## What to Write

Write: `02 - Tiger Tournaments/Projects/Rankings/Boys Brier Analysis Scope Spec — May 2026.md`

---

### Section 1: Goal

One paragraph. What question does this analysis answer?

> The Girls GA ASPIRE calibration (140 → 100) was driven by Brier score analysis showing overcalibration relative to ECNL opponents. Boys GA ASPIRE is currently cal=100 (same as Boys GA) — the conservative April 28 decision. This analysis determines whether Boys GA ASPIRE teams are overcalibrated at 100, undercalibrated, or correctly set relative to their MLS NEXT and Boys GA peers.

### Section 2: Data Requirements

What data is needed to run a Boys-specific Brier analysis on GA ASPIRE events?

- Minimum game count for statistical validity: [N games — reference the Girls analysis sample size]
- Event tier filter: `event_tier = 'ga_aspire'` AND `gender = 'Boys'`
- Date range: [define — likely last 2 full seasons to match what was used for Girls]
- Cross-league games needed: Boys GA ASPIRE teams must have played against Boys GA or Boys MLS NEXT teams in the same dataset to allow comparison. Note whether cross-tier games exist in the current dataset.

Include the SQL to count the available Boys GA ASPIRE games:
```sql
SELECT COUNT(*) AS boys_ga_aspire_games,
       COUNT(DISTINCT g.home_team_id) + COUNT(DISTINCT g.away_team_id) AS teams_participating
FROM games g
JOIN events e ON e.id = g.event_id
WHERE e.event_tier = 'ga_aspire'
  AND g.gender = 'Boys';
```

Note: if Boys GA ASPIRE game count < [minimum threshold], state that the analysis is data-limited and recommend deferring until more data accumulates.

### Section 3: Analysis Method

How the analysis runs — follow the same methodology used for Girls:

1. **Baseline:** Run current rankings with Boys GA ASPIRE at cal=100 (Option A).
2. **Counterfactual:** Run rankings with Boys GA ASPIRE at cal=70 (Option B, structural analogy).
3. **Brier score comparison:** For a sample of Boys cross-tier games (GA ASPIRE vs GA, GA ASPIRE vs MLS NEXT), compute Brier scores under both calibration values. Which produces better-calibrated predictions?
4. **Interpretation:** If Option B Brier score is materially better (delta > [threshold from Girls analysis]): revise Boys GA ASPIRE to ~70. If no material difference or Option A wins: confirm Option A and close the question.

### Section 4: Timeline and Execution Window

- **Target completion:** Before June 1 Boys age-group transition
- **Prerequisite:** DSS demo must complete first (May 14–15) — ELO will not allocate analysis time during the DSS window
- **Execution estimate:** [N hours] — based on the Girls Brier analysis duration
- **Presten time required:** [N minutes] — to run SQL queries against `youth_soccer` DB

### Section 5: Decision Criteria

State the threshold for changing Boys GA ASPIRE calibration away from Option A:
- If cross-tier Brier score improvement under Option B > [threshold]: recommend revising to ~70.
- If improvement < threshold or ambiguous: retain Option A (cal=100). No further analysis needed.
- If data-limited (game count < minimum): retain Option A and re-scope after June 1 transition adds new games.

## Definition of Done

- Spec written at `Rankings/Boys Brier Analysis Scope Spec — May 2026.md`
- All five sections present
- Data requirements include the SQL to count available Boys GA ASPIRE games
- Timeline coordinates with DSS window (May 14–15) and June 1 transition
- Decision criteria explicitly define what would trigger a change from Option A to Option B
- Summary in briefing: "Boys Brier Analysis scope spec written. Target window: May 17–30. Execution prerequisite: DSS complete."

## Why This Matters

The April 28 decision was conservative by design. Conservative is correct when analysis is missing — but the analysis should be scheduled, not deferred indefinitely. If Boys GA ASPIRE is meaningfully overcalibrated at 100, it affects Boys rankings in the same way Girls GA ASPIRE was affecting Girls rankings. Writing the scope spec now means ELO can execute immediately post-DSS, rather than spending time in June scoping an analysis that should have been ready in May.

## References

- `02 - Tiger Tournaments/Projects/Rankings/Boys GA ASPIRE Calibration Decision — April 2026.md` — source of Option B deferral decision
- `10 - Agents/ELO/Tasks/task-2026-04-25-age-group-transition-june1-risk.md` — June 1 transition context
- `10 - Agents/ELO/Tasks/task-2026-04-26-dss-demo-spec.md` — DSS timing constraint
- `02 - Tiger Tournaments/Projects/Rankings/League Hierarchy.md` — Boys GA calibration value for comparison
