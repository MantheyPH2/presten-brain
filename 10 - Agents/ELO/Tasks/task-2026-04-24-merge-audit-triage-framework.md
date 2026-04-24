---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-24
status: completed
completed: 2026-04-24
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Merge Audit Triage Framework — May 2026.md"
priority: medium
due: 2026-05-05
---

# Task: Merge Audit Triage Framework — Process for Post-Audit Results

## Context

The merge audit SQL is written and staged. Presten will run it by May 10. When it returns results, there will be a list of potentially incorrect merges among the 27,820 `team_merges` entries. DSS is May 16 — six days after the audit results arrive.

What does not exist: a process for what happens next. "Review the output" is not a triage protocol. Without a defined framework, Presten will face a variable-length list of ambiguous merge candidates with no criteria for what to fix first, what to skip, and what the rating implications are.

This task: write the triage framework ELO will use to process merge audit results quickly and correctly within the six-day pre-DSS window.

## What to Write

Write: `02 - Tiger Tournaments/Projects/Rankings/Merge Audit Triage Framework — May 2026.md`

---

### Section 1: Merge Classification Criteria

Define three categories with explicit, executable criteria:

**CORRECT — merge should stay as-is**

A merge is CORRECT if ALL of the following are true:
1. Both team names belong to the same club (same city, same parent organization)
2. Name difference is a variation only (abbreviation, year format, color suffix, gender marker)
3. The teams do not have overlapping game dates in the same tournament
4. The canonical team ID has more total games than either component ID

Examples of correct merges: "FC Stars G2012 ECNL White" ↔ "FC Stars Girls 2012 ECNL White"

Action: No change. Mark as verified in the audit log.

---

**INCORRECT — merge should be reversed**

A merge is INCORRECT if ANY of the following are true:
1. The two teams are from different cities or different parent clubs (same name, different org — a GotSport name-collision merge)
2. The merged game histories show overlapping game dates in the same tournament (one team cannot play twice on the same date in the same event)
3. The merged record contains games from distinct geographic regions that are incompatible (e.g., CA team merged with FL team of the same name)

Verification queries for INCORRECT:
```sql
-- Check for game date overlap (same-tournament, same-date impossibility)
SELECT g1.date, g1.event_name, g1.home_team_id, g2.home_team_id
FROM games g1
JOIN games g2 ON g1.date = g2.date AND g1.event_name = g2.event_name
WHERE (g1.home_team_id = [merged_team_id] OR g1.away_team_id = [merged_team_id])
  AND (g2.home_team_id = [canonical_team_id] OR g2.away_team_id = [canonical_team_id]);
-- Any result = INCORRECT merge (same team can't play in the same event on the same day twice)
```

Action: Un-merge. Remove the entry from `team_merges`. Flag to SENTINEL with: team names, original merge date, reason for reversal.

---

**AMBIGUOUS — insufficient information to decide**

A merge is AMBIGUOUS if:
- Names are similar but from different apparent clubs
- No overlapping game dates but also no distinguishing geographic metadata
- Game histories do not provide enough signal to confirm or deny

**Conservative rule (apply to all AMBIGUOUS):** Keep merged if the canonical team ID has ≥ 2× more game history than the merged ID. Separate if the merged ID has a comparable or larger game history (this suggests it was an independent team that may have been incorrectly absorbed).

Action: Document the ambiguous merge in the audit log with the conservative rule applied. Flag to SENTINEL if the ambiguous team is in the top 500 rankings.

---

### Section 2: Rating Impact by Category

Document the Elo implication of each triage decision:

**CORRECT merges found in audit:** No rating change needed. The merged history was correctly consolidated. Mark as verified.

**INCORRECT merges reversed:**
- Both teams must have their ratings recomputed independently from their separate game histories
- Before recommending an un-merge, confirm: how many games does each team have separately? If one team has fewer than 5 games, its independent rating will be noisy and near the initialization value (1000). Document this: "Reversing this merge will result in [team A] having a rating based on [X] games (unreliable) and [team B] retaining a rating based on [Y] games (reliable)."
- Rating delta estimate: if the merged canonical had a significantly higher rating than either component alone would justify, the reversal will drop the canonical's rating. ELO should estimate this delta for any top-500 team whose merge is reversed before DSS.

**New merges identified by audit (teams that should be merged but aren't):**
- The canonical ID should be the team with more game history
- After merge: ratings recomputed from consolidated history
- Risk: if two previously separate teams had independent ratings and one was significantly higher, the merge may cause a visible rank change at DSS. Flag any such merges found in the top 200.

---

### Section 3: Three-Question Triage Protocol

For each flagged merge in the audit output, apply these questions in order:

**Q1: "Are these two teams the same organization?"**
- Check: same city AND same parent club name
- YES → Likely CORRECT. Continue to Q2.
- NO (different clubs) → INCORRECT. Un-merge.

**Q2: "Do these two teams have overlapping game dates in the same tournament?"**
- Run the game-date overlap query from Section 1
- YES (overlap exists) → INCORRECT. Un-merge.
- NO (no overlap) → Continue to Q3.

**Q3: "Does the canonical ID have substantially more game history than either component?"**
- Check: `SELECT COUNT(*) FROM games WHERE home_team_id = [id] OR away_team_id = [id]` for both IDs
- YES (canonical has ≥ 2× more games) → CORRECT. Mark as verified.
- NO (similar game counts) → AMBIGUOUS. Apply conservative rule from Section 1.

---

### Section 4: Six-Day Triage Scope (May 10–16)

Given that audit results arrive ~May 10 and DSS is May 16:

**Day 1 (May 10): Triage top-500 ranked teams first**
- Pull the list of top-500 teams for U13G and U14G from the audit results
- Apply the three-question protocol to any merge involving a top-500 team
- Reverse all INCORRECT merges in top-500 immediately — rating impact on high-ranked teams is demo-visible
- Document all reversals with rating delta estimates

**Day 2 (May 11): Triage top-500 AMBIGUOUS cases**
- For any top-500 teams still AMBIGUOUS after Day 1, apply the conservative rule
- For teams 200–500: if conservative rule says KEEP MERGED, no further action needed
- For teams 1–200 that remain AMBIGUOUS: escalate to SENTINEL for a judgment call

**Days 3–4 (May 12–13): Bottom-500 and unranked INCORRECT merges**
- Run the three-question protocol on remaining INCORRECT audit flags
- Un-merge confirmed INCORRECT cases
- Note: rating impact for unranked teams is not demo-visible — process these efficiently but don't let them consume more than half a day

**Defer until post-DSS:**
- All AMBIGUOUS merges involving unranked teams
- Any merge where rating impact research takes more than 20 minutes (document the case and defer)
- New merge candidates (teams that should be merged but aren't) — adding new merges 3 days before DSS introduces rating instability

**Day 5 (May 14): DSS data validation run**
- After triage is complete, run the data validation criteria from `Rankings/DSS Demo Validation Criteria — May 2026.md`
- If any anchor team (Football Academy NJ, Wall SC) has been affected by a merge reversal or addition, confirm they still meet the pass criteria

---

### Section 5: INCORRECT Merge Reversal Checklist

For each INCORRECT merge reversed, complete these steps in order:

1. Remove the entry from `team_merges` (`DELETE FROM team_merges WHERE canonical_team_id = [X] AND merged_team_id = [Y]`)
2. Confirm both teams now have separate entries in `teams` table
3. Run a spot-check on both teams' game counts (before and after reversal) to confirm no games were lost: `SELECT COUNT(*) FROM games WHERE home_team_id = [id] OR away_team_id = [id]`
4. Schedule or trigger a ranking recompute for both teams (note: if a daily pipeline run is scheduled for that night, no manual recompute needed)
5. Flag to SENTINEL: team names, merge date, reversal reason, estimated rating delta

---

## Definition of Done

- All three merge categories have explicit, executable criteria (not "similar names" — something that can be checked with a query or a name comparison)
- Rating impact is documented for each category with concrete implications
- Three-question triage protocol is written as a flowchart-style decision
- Six-day triage scope is defined with day-by-day priorities and defer criteria
- INCORRECT merge reversal checklist is written with SQL statements

## Why This Matters

May 10 to May 16 is six days. Without a framework, the audit produces a spreadsheet of uncertain merges and a subjective judgment process under time pressure. With this framework, it produces a triage queue that can be processed systematically: INCORRECT cases fixed on Day 1, AMBIGUOUS cases handled by Day 2, demo-safe by Day 5.

## References

- `Rankings/USA Rank Comparison 2026-04-23.md` — Football Academy NJ merge is the known case study for this framework
- `Database Schema.md` — `team_merges` table structure
- Task `task-2026-04-24-dss-demo-validation-criteria.md` — Day 5 validation criteria (companion document)
- ELO briefing 2026-04-23 23:52 — original merge audit flag (background context)
