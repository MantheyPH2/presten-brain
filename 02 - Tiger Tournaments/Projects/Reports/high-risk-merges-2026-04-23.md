---
title: High-Risk Merges — Top 50 for Manual Review
tags: [team-merges, dedup, data-quality, high-risk, evo-draw]
created: 2026-04-23
updated: 2026-04-23
author: ELO Agent
status: requires-db-execution
related_task: task-2026-04-23-team-merges-verification-campaign
---

# High-Risk Merges — Top 50 for Presten's Review

> [!warning] Requires Database Execution
> This document requires running the high-risk merge query from `Reports/unverified-merges-analysis-2026-04-23.md` Section 4 to populate the table below. The structure, risk categories, and review criteria are all defined here — only the specific team names and counts need to be filled in from the DB.

---

## How to Use This Document

1. Run the high-risk SQL query from the analysis doc (Section 4)
2. Paste the top 50 results into the table below
3. For each merge, do a quick spot-check: do the team names look like the same team?
4. Mark `review_verdict`: **Correct** (keep merge), **Wrong** (remove merge), or **Unsure** (escalate)
5. For any "Wrong" verdicts: run the un-merge SQL and recompute ratings for affected teams

---

## Risk Category Definitions

| Risk Flag | What it Means | How to Spot It |
|-----------|--------------|----------------|
| `fuzzy_age_mismatch` | Two teams with similar names but different age groups were merged | "FC Dallas U14 Black" merged with "FC Dallas U15 Black" — look for age numbers in names |
| `coach_name_date_gap` | Merge was based on a coach name, but the teams' active date ranges don't overlap | Teams played in different seasons with no overlap — likely different teams with the same coach |
| `large_rating_gap` | The canonical and merged teams have ratings more than 200 points apart | One team was ~Bronze, the other ~Green — different quality levels suggest different teams |
| `unknown_method` | No merge method recorded — origin is unknown | These should be manually reviewed; they may have been imported from an external source |
| `geographic_mismatch` | Teams from different states with similar names | "FC United Blue TX" merged with "FC United Blue CA" |

---

## Top 50 High-Risk Merges (Populate After DB Query)

| # | Merge ID | Canonical Name | Merged Name | Merge Method | Risk Flag | Games Affected | Rating Gap | Review Verdict | Notes |
|---|----------|---------------|-------------|--------------|-----------|----------------|------------|----------------|-------|
| 1 | [QUERY] | [QUERY] | [QUERY] | [QUERY] | [QUERY] | [QUERY] | [QUERY] | [ ] | |
| 2 | [QUERY] | [QUERY] | [QUERY] | [QUERY] | [QUERY] | [QUERY] | [QUERY] | [ ] | |
| 3 | [QUERY] | [QUERY] | [QUERY] | [QUERY] | [QUERY] | [QUERY] | [QUERY] | [ ] | |
| 4 | [QUERY] | [QUERY] | [QUERY] | [QUERY] | [QUERY] | [QUERY] | [QUERY] | [ ] | |
| 5 | [QUERY] | [QUERY] | [QUERY] | [QUERY] | [QUERY] | [QUERY] | [QUERY] | [ ] | |
| 6-50 | ... | ... | ... | ... | ... | ... | ... | [ ] | |

---

## Expected Risk Profile of the Top 50

Based on the unverified merges analysis, the top 50 highest-risk merges by game volume are expected to contain:

| Risk Category | Expected Count in Top 50 | Priority |
|---------------|--------------------------|----------|
| `fuzzy_age_mismatch` | 10-15 | HIGHEST — often wrong |
| `large_rating_gap` | 8-12 | HIGH — implies different teams |
| `unknown_method` | 5-8 | HIGH — provenance unknown |
| `coach_name_date_gap` | 5-8 | MEDIUM — coach transfer plausible |
| `geographic_mismatch` | 3-6 | MEDIUM — common club names nationwide |
| Review-flagged fuzzy matches | 8-12 | MEDIUM — review name similarity |

---

## Common Team Names to Watch For

These are the names most likely to produce false-positive merges (too common across many clubs):

- "Elite" / "Select" / "Premier" — generic names used by many unrelated clubs
- Color names: "Red", "Blue", "Black", "Gold", "White" — extremely common team suffixes
- "FC United" — several unrelated clubs use this name
- "Rush" — used by Rush Soccer (large national club) AND unrelated local clubs
- "Celtic" — common club name in multiple states
- "FC Dallas" / "North Texas" / "Dallas" — might get merged with Dallas-area MLS teams vs unrelated Dallas-named recreational clubs

For any merge involving these name patterns: extra scrutiny required. The canonical team should have GotSport ID or TGS ID to confirm source identity.

---

## Un-Merge SQL (Use When a Merge is Wrong)

```sql
-- Step 1: Remove the bad merge record
DELETE FROM team_merges 
WHERE id = [merge_id]  -- the wrong merge

-- Step 2: Mark any games incorrectly attributed to canonical team as needing re-attribution
UPDATE games
SET home_team_id = [merged_team_id]  -- restore games to the merged team
WHERE home_team_id = [canonical_team_id]
  AND id IN (
    SELECT id FROM games 
    WHERE (home_team_id = [merged_team_id] OR away_team_id = [merged_team_id])
      AND is_hidden = false
  );
-- Also update away_team_id for away games

-- Step 3: Add the merged team back as a standalone record
-- (It should already exist in the teams table; just needs is_hidden = false)
UPDATE teams
SET is_hidden = false
WHERE team_id = [merged_team_id];

-- Step 4: Flag for rankings recompute
-- Rankings will need to be recomputed for the canonical team and the restored merged team
-- Mark in a recompute queue if one exists, or just note it for the next compute-rankings.js run
```

> [!warning] IMPORTANT
> Do not run the un-merge SQL in production without running a test recomputation first in a dev environment. Confirm the canonical team's new rating looks correct before committing.

---

## Rating Impact Reference

From the full analysis in `Reports/unverified-merges-analysis-2026-04-23.md` Section 5:

- **Low-impact merge errors (< 25 point rating shift):** Fix when convenient; not urgent
- **Medium-impact merge errors (25-75 point rating shift):** Fix before the May 18-20 recalculation
- **High-impact merge errors (> 75 point rating shift):** Fix immediately — these are actively corrupting rankings for affected teams

Any merge in this top-50 list with more than 50 games affected AND a risk flag of `fuzzy_age_mismatch` or `large_rating_gap` is likely in the high-impact category.

---

## Related

- `Reports/unverified-merges-analysis-2026-04-23.md` — full analysis with all SQL
- `Reports/unverified-merges-top-2000-2026-04-23.csv` — full ranked list (requires DB)
- [[Strategic Insights]] — Insight 2, Action Item 6
- [[Ranking Engine]] — Phase 1: how merges feed into computation
