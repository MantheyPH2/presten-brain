---
title: Team Name Matching
aliases: [Name Mismatches, DSS Name Aliases, GotSport Names]
tags: [knowledge, team-matching, names, dedup, dss, evo-draw]
created: 2026-04-21
updated: 2026-04-21
---

# Team Name Matching

A critical data quality challenge: [[GotSport]] API returns different team names than [[Dallas Spring Showcase|DSS]] registration records. This required a dedicated resolution strategy.

---

## The Problem

Teams register for DSS with one name but appear in [[GotSport]] game data with a different name. Exact string matching fails for these teams, leaving them with no game history for [[Division Calibration|division placement]] and [[Ranking Engine|ranking]].

**Examples of mismatches:**

| DSS Registration Name      | GotSport API Name              |
| -------------------------- | ------------------------------ |
| Toros FC 2017 Black        | EPPL Toros FC Black 2017       |
| Dallas Texans 2014 Red     | Texans SC 14B Red              |
| Solar SC 2015 Elite        | Solar ECNL 15B                 |

---

## The Solution

> [!tip] Match by Team ID, Not Name
> Instead of matching on exact team name, look up by **GotSport team ID** using the `home_team_id` and `away_team_id` columns in the [[Database Schema|games table]]. Team IDs are stable even when display names vary.

### Resolution Steps

1. Match DSS team names to GotSport team IDs via the GotSport event registration API
2. Look up game history using `home_team_id` / `away_team_id` instead of name matching
3. For the 24 teams that still didn't match, create manual aliases

### dss-name-aliases.json

A mapping file at `data/dss-name-aliases.json` that maps DSS registration names to their GotSport API names:

```json
{
  "Toros FC 2017 Black": "EPPL Toros FC Black 2017",
  "Dallas Texans 2014 Red": "Texans SC 14B Red"
}
```

- **24 name mismatches** identified and resolved via this file
- The pipeline loads these aliases before team history lookup

---

## Impact

| Metric                        | Before Fix | After Fix |
| ----------------------------- | ---------- | --------- |
| Teams with game history       | 271/308    | 295/308   |
| Teams with 0 games            | 37         | 13        |
| Additional games matched      | 0          | ~2,400    |

The remaining 13 teams with zero games are genuinely new teams with no competitive history in any of our [[Data Sources Overview|7 data sources]].

---

## Lessons Learned

> [!warning] Never Match on Name Alone
> Team names are unreliable across platforms. Always prefer:
> 1. Platform-specific team IDs (most reliable)
> 2. Coach name matching (see [[Dedup Strategy|match-teams-cross-platform.js]])
> 3. Fuzzy name matching (last resort)
>
> The `dss-name-aliases.json` approach is a manual override for the cases that slip through automated matching.

---

## Related Notes
- [[Dallas Spring Showcase]] — the event where this was discovered
- [[GotSport]] — source of the mismatched names
- [[Dedup Strategy]] — cross-platform team matching logic
- [[Database Schema]] — `home_team_id`/`away_team_id` columns
- [[Division Calibration]] — depends on accurate team matching
- [[Data Pipeline]] — Step 7 handles cross-platform matching
