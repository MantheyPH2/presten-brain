---
title: Merge Audit Triage Framework — May 2026
aliases: [Merge Triage Framework, Team Merge Audit Process]
tags: [rankings, dedup, merges, triage, dss, evo-draw]
created: 2026-04-24
updated: 2026-04-24
author: ELO Agent
status: ready
task: task-2026-04-24-merge-audit-triage-framework
---

# Merge Audit Triage Framework — May 2026

> The merge audit SQL is written and staged. Presten runs it by May 10. Results arrive 6 days before DSS (May 16). This framework processes the audit output systematically — INCORRECT merges fixed Day 1, AMBIGUOUS resolved Day 2, demo-safe by Day 5 (May 14 validation run).
>
> **27,820 total merges in `team_merges`. 9,136 unverified. Audit targets the highest-risk subset.**

---

## Section 1: Merge Classification Criteria

### CORRECT — merge stays as-is

A merge is CORRECT if ALL of the following are true:

1. Both team names belong to the same club (same city, same parent organization name)
2. Name difference is a variation only: abbreviation, year format, color suffix, gender marker, platform-specific formatting
3. The teams do NOT have overlapping game dates in the same tournament
4. The canonical team ID has more total games than the merged team ID

**Examples of correct merges:**
- "FC Stars G2012 ECNL White" ↔ "FC Stars Girls 2012 ECNL White" — same club, name format variation
- "Football Academy NJ 2012 Girls Black" ↔ "FA NJ 12G Black" — abbreviation of the same club
- "Wall SC Wall Elite Porto" ↔ "Wall Soccer Club Elite Porto G2012" — the known case study from `USA Rank Comparison 2026-04-23.md`

**Action:** No change. Mark as verified in the audit log. No rating impact.

---

### INCORRECT — merge should be reversed

A merge is INCORRECT if ANY of the following are true:

1. The two teams are from different cities or different parent clubs (GotSport name-collision — two clubs with the same common name, different organizations)
2. The merged game histories show overlapping game dates in the same tournament (a team cannot play twice on the same date in the same event — if both records show games on the same date in the same event, they were separate teams)
3. The merged record contains games from geographically incompatible regions (e.g., a California team's games appear in the same record as a Florida team's games with no cross-region explanation)

**Verification query for criterion 2 (game-date overlap):**

```sql
-- Check for same-tournament, same-date impossibility
-- Run with [merged_team_id] and [canonical_team_id] substituted
SELECT g1.match_date, g1.event_name,
  g1.home_team_id AS team_a_record,
  g2.home_team_id AS team_b_record
FROM games g1
JOIN games g2
  ON g1.match_date = g2.match_date
  AND g1.event_name = g2.event_name
WHERE (g1.home_team_id = '[merged_team_id]' OR g1.away_team_id = '[merged_team_id]')
  AND (g2.home_team_id = '[canonical_team_id]' OR g2.away_team_id = '[canonical_team_id]')
  AND is_hidden = false
LIMIT 10;
```

Any result = INCORRECT. A team cannot play in the same event on the same date under two different IDs.

**Action:** Un-merge. Remove entry from `team_merges`. Flag to SENTINEL.
**Rating impact:** Both teams must have their ratings recomputed independently from separate game histories. See Section 2.

---

### AMBIGUOUS — insufficient information to decide

A merge is AMBIGUOUS if:
- Names are similar but from different apparent clubs — not clearly the same, not clearly different
- No overlapping game dates, but no distinguishing geographic metadata to confirm or deny
- Game histories provide no clear signal (both small, or one is empty)

**Conservative rule for all AMBIGUOUS cases:**
- **KEEP merged** if the canonical team ID has ≥ 2× more game history than the merged ID. The merged record contributed a small amount of game history to a larger record — conservative to keep.
- **SEPARATE** if the merged ID has comparable or larger game history (suggests it was an independent team that may have been incorrectly absorbed into the canonical).

```sql
-- Game count comparison for ambiguous merge candidates
SELECT
  'canonical' AS role, COUNT(*) AS game_count
FROM games
WHERE (home_team_id = '[canonical_team_id]' OR away_team_id = '[canonical_team_id]')
  AND is_hidden = false
UNION ALL
SELECT
  'merged' AS role, COUNT(*) AS game_count
FROM games
WHERE (home_team_id = '[merged_team_id]' OR away_team_id = '[merged_team_id]')
  AND is_hidden = false;
```

**Action:** Document the ambiguous merge in the audit log with the conservative rule applied. Flag to SENTINEL if the ambiguous team is in the top 500 rankings (demo-visible risk).

---

## Section 2: Rating Impact by Category

### CORRECT merges found in audit

No rating change needed. The merged history was correctly consolidated. Confirm verified status and move on. If any CORRECT merges were previously unverified (in the 9,136 unverified pool), set `verified = true` — this gives the ranking engine access to game history it was previously ignoring.

```sql
-- Set verified on confirmed-correct merges that are currently unverified
UPDATE team_merges
SET verified = true
WHERE canonical_team_id = '[canonical_id]'
  AND merged_team_id = '[merged_id]'
  AND verified = false;
```

### INCORRECT merges reversed

**Before recommending an un-merge, document the rating impact:**

```sql
-- Game count for each component team (pre-reversal)
SELECT 'canonical' AS role, COUNT(*) AS game_count,
  (SELECT rating FROM rankings WHERE team_id = '[canonical_id]' AND age_group = 'U13' AND gender = 'F') AS current_rating
FROM games
WHERE (home_team_id = '[canonical_id]' OR away_team_id = '[canonical_id]') AND is_hidden = false
UNION ALL
SELECT 'merged' AS role, COUNT(*) AS game_count, NULL AS current_rating
FROM games
WHERE (home_team_id = '[merged_id]' OR away_team_id = '[merged_id]') AND is_hidden = false;
```

**Rating impact estimate:**

- If the canonical had 150 games and the merged had 20 games (87% canonical): the reversal removes ~13% of the canonical's game history. Expected rating impact: small (< 20 points for a well-established team with 150+ games).
- If the canonical had 80 games and the merged had 60 games (57% canonical): the reversal removes a significant fraction of game history. Expected rating impact: could be ±30–70 points. Flag to SENTINEL if the canonical is in the top 200.
- If the merged ID has fewer than 5 games: its independent rating post-reversal will be noisy (~1000 default). Document: "Reversing this merge will result in [merged team] having a rating based on [N] games — this is below the reliable rating threshold."

### New merges identified by audit (teams that should be merged but aren't)

**Defer all new merges until after DSS.** Adding new merges 3–6 days before DSS introduces rating instability in the next pipeline run. The newly merged canonical will have a different rating than either component had independently — this can cause visible rank shifts in the demo.

**Exception:** If a top-10 team has a clearly split record (e.g., Football Academy NJ exists as two separate unmerged records), merging them before DSS improves the demo by giving the team its full game history. Document the expected rating improvement.

---

## Section 3: Three-Question Triage Protocol

Apply in order for each flagged merge. Stop at the first definitive answer.

```
Q1: "Are these two teams the same organization?"
  Check: same city AND same parent club name (not just similar names — same org)
  ├── YES → Likely CORRECT. Continue to Q2.
  └── NO (different clubs, different cities) → INCORRECT. Un-merge. Log it.

Q2: "Do these two teams have overlapping game dates in the same tournament?"
  Run the game-date overlap query from Section 1.
  ├── YES (any overlap found) → INCORRECT. Un-merge. Log it.
  └── NO (no overlap) → Continue to Q3.

Q3: "Does the canonical ID have ≥ 2× more game history than the merged ID?"
  Run the game count comparison from Section 1 AMBIGUOUS block.
  ├── YES (canonical ≥ 2× merged games) → CORRECT. Mark as verified. Log it.
  └── NO (similar game counts) → AMBIGUOUS. Apply conservative rule. Log it.
```

Total time per merge: 2–5 minutes if the data is accessible. For the top-500 audit subset, expect 3–6 hours on Day 1 depending on number of flagged merges.

---

## Section 4: Six-Day Triage Scope (May 10–16)

### Day 1 (May 10): Top-500 ranked teams — INCORRECT merges only

Priority: fix anything that affects teams visible in the DSS demo.

1. From audit output: filter for flagged merges where either the canonical OR merged team appears in the top-500 rankings for U13G or U14G
2. Apply the three-question protocol to each
3. Reverse all INCORRECT merges in the top-500 — document rating delta estimate for each
4. Set `verified = true` on CORRECT top-500 merges that were previously unverified

**End of Day 1 target:** All INCORRECT merges involving top-500 teams are reversed. Rating recompute will run that night.

### Day 2 (May 11): Top-500 AMBIGUOUS cases + rating verification

1. For top-500 teams still AMBIGUOUS after Day 1: apply the conservative rule (≥ 2× game history → KEEP MERGED)
2. For teams ranked 1–200 still AMBIGUOUS after conservative rule: escalate to SENTINEL with a one-paragraph summary
3. Run a quick rating check on the top-10 U13G after the night's recompute: do the expected teams still appear? Did any reversals produce unexpected rank shifts?

### Days 3–4 (May 12–13): Bottom-500 and unranked INCORRECT merges

1. Apply the three-question protocol to remaining INCORRECT audit flags
2. Reverse confirmed INCORRECT cases — these are below rank 500, so rating impact is not demo-visible
3. Time box: do not spend more than half a day total on unranked teams (days 3+4 combined)

### Day 5 (May 14): Data validation run

After triage is complete, run the full validation criteria from `Rankings/DSS Demo Validation Criteria — May 2026.md`.

**Specific check after triage:** If any anchor team (Football Academy NJ, Wall SC) was directly involved in a merge reversal or new merge, re-run the anchor team lookup query from Section 1 of the validation criteria to confirm they still meet their pass criteria. A merge reversal that reduces their game history could affect their rating.

### Defer until after DSS

- All AMBIGUOUS merges involving unranked teams
- Any merge where rating impact research takes > 20 minutes (document and defer)
- All new merge candidates (teams that should be merged but aren't)
- New merges for teams outside the top 500

---

## Section 5: INCORRECT Merge Reversal Checklist

For each confirmed INCORRECT merge, complete in order:

1. **Document the case** before touching anything:
   ```
   Canonical ID: [X]  Name: [team name]  Rating: [Y]  Rank: [Z]
   Merged ID: [A]     Name: [team name]  Games: [N]
   Reason for reversal: [Q1/Q2/Q3 failure — specific reason]
   Estimated rating delta: [±X points]
   ```

2. **Remove the merge record:**
   ```sql
   DELETE FROM team_merges
   WHERE canonical_team_id = '[canonical_id]'
     AND merged_team_id = '[merged_id]';
   ```

3. **Verify both teams exist separately in `teams`:**
   ```sql
   SELECT id, name FROM teams
   WHERE id IN ('[canonical_id]', '[merged_id]');
   -- Should return 2 rows
   ```

4. **Confirm no games were lost** (spot-check, not exhaustive):
   ```sql
   SELECT COUNT(*) AS game_count
   FROM games
   WHERE (home_team_id = '[canonical_id]' OR away_team_id = '[canonical_id]')
     AND is_hidden = false;
   -- Compare against pre-reversal count
   ```

5. **Schedule recompute:** If a nightly pipeline run is scheduled for that evening, no manual recompute is needed — the pipeline will handle it. If not, note that ratings will not reflect the reversal until the next scheduled run.

6. **Flag to SENTINEL:** Team names, merge date, reversal reason, estimated rating delta. If the reversal affects a top-200 team, flag as high priority.

---

## References

- [[USA Rank Comparison 2026-04-23]] — Football Academy NJ / FA 12G Black case study (Section 4, Gap 4)
- [[Database Schema]] — `team_merges` table structure
- `Rankings/Pre-DSS Team Merge Audit — U13G.md` — the audit SQL being processed
- [[DSS Demo Validation Criteria — May 2026]] — Day 5 validation target
- ELO Briefing 2026-04-23 23:52 — original merge audit flag background context
