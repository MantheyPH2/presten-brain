---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-25
status: pending
priority: medium
due: 2026-05-01
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Club Rankings Reference Spot-Check Set.md"
---

# Task: Club Rankings Reference Spot-Check Set

## Context

The Club Rankings implementation plan (`Rankings/Club Rankings — Implementation Specification.md`) includes a spot-check step: verify known clubs appear at plausible positions. The plan names PA Classics (must appear ≤ rank 30 Girls), Eclipse Select, and FC Dallas as examples.

Three clubs is not a validation set — it's a sanity check. When Presten runs the May 2–3 session and sees club rankings output for the first time, a 10-club reference set with expected positions for both Boys and Girls provides a real validation layer. Without it, Presten is judging "does this look right?" from intuition rather than a pre-defined spec.

This task: compile that reference set before the implementation session runs.

## What to Do

### Step 1: Select 10 Reference Clubs

Choose clubs that meet all of these criteria:
- Teams in 5+ age groups (U11–U17) — the minimum to qualify for club rankings
- Nationally or regionally recognized by coaches and club directors
- Geographic spread across at least 4 states (not all NJ/PA)

Structure the selection:
- Girls top 10 expected: 3 clubs
- Girls rank 11–30 expected: 3 clubs
- Girls rank 31–60 expected: 2 clubs
- Girls regionally significant but not top-30 nationally: 2 clubs

Boys: same structure if Boys club rankings are expected to compute (note if Boys Option A validation requirement affects Boys availability — if <300 cross-tier games, Boys rankings may not be produced at May 2–3).

### Step 2: Estimate Expected Position

For each club and gender, estimate the expected club ranking position using:
- Known ECNL or GA affiliation (ECNL clubs should rank above comparable non-ECNL clubs)
- USA Rank club-level data if available (reference `Rankings/USA Rank Ranking Accuracy Audit — April 2026.md` and [[Competitors]])
- ELO's understanding of which clubs have deep multi-age-group programs

The position estimate does not need to be exact. Ranges are sufficient:
- "Expected top 5"
- "Expected top 10–20"
- "Expected top 30–50"
- "Expected top 50–75"

What matters: if a club ends up far outside its expected range, ELO catches it during the implementation session.

### Step 3: Define Pass/Fail Criteria Per Club

For each club:
- **Pass:** "Club appears in rankings at position ≤ [threshold] for [gender]"
- **Fail — investigate:** "Club appears but at position > [threshold] — check: [specific cause, e.g., low org_id match rate, too few qualifying age groups, calibration outlier in the relevant leagues]"
- **Fail — escalate:** "Club does not appear at all — check: org_id match, team-to-club assignment coverage, whether club name parsing selected the wrong club name"

The fail threshold for Girls top-tier clubs (PA Classics, Eclipse, etc.) must be ≤ 30. For the mid-tier clubs, set thresholds consistent with their expected ranges.

### Step 4: Write the Reference Document

Deliverable: `02 - Tiger Tournaments/Projects/Rankings/Club Rankings Reference Spot-Check Set.md`

Format:

```
| Club | State | League Affiliation | Expected Girls Rank | Girls Pass Threshold | Expected Boys Rank | Boys Pass Threshold |
```

After the table, write a one-paragraph diagnostic guide:

"If a well-known club does not appear in Club Rankings output or appears far outside the expected range, check in order: (1) Is the club's GotSport org_id matched to teams in Evo Draw? Run: `SELECT COUNT(*) FROM teams WHERE gotsport_org_id = '[org_id]'` (2) Do those teams have ratings? (3) Are at least 5 age groups qualifying? (4) Is the bootstrap Case A or Case B path active — and did the name-parse capture this club correctly?"

## Definition of Done

- Reference set at `Rankings/Club Rankings Reference Spot-Check Set.md`
- 10 clubs selected with expected positions for both Boys and Girls
- Pass/fail thresholds are specific numbers, not "approximately correct"
- Diagnostic guide is present and actionable (specific SQL, not "investigate")
- Summary in briefing: "Club Rankings spot-check reference set compiled. [N] clubs. Girls: PA Classics ≤ rank [N], Eclipse Select ≤ rank [N]. Boys: [N] clubs if option A validated."

## Why This Matters

Club Rankings will be in the DSS demo. If the bootstrap runs correctly, club positions should match well-known expectations from the soccer community. If a club that everyone in the DSS room knows to be elite appears at rank 45, that's a problem — but only if someone has pre-defined what elite clubs should appear at. Writing the reference set now means Presten can validate club rankings in-session rather than sending the output back to ELO for review.

## References

- `Rankings/Club Rankings — Implementation Specification.md` — spot-check requirements
- `Rankings/Club Rankings Design.md` — methodology and expected data quality
- `Rankings/USA Rank Ranking Accuracy Audit — April 2026.md` — USA Rank reference data
- [[Competitors]] — USA Rank club rankings page for cross-reference
