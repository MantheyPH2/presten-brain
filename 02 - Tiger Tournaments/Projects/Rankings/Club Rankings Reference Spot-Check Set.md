---
title: Club Rankings Reference Spot-Check Set
type: validation-reference
author: ELO
date: 2026-04-25
status: complete
related: "[[Club Rankings — Implementation Specification]]", "[[Club Rankings Design]]", "[[DSS Demo Spec — May 2026]]"
tags: [club-rankings, validation, spot-check, dss, evo-draw]
---

# Club Rankings Reference Spot-Check Set

> **Purpose:** 10-club reference set for validating club rankings output during the May 2–3 implementation session. Run these checks after the Club Rankings computation SQL completes. Each club has a specific pass threshold — if a club falls outside its expected range, the diagnostic guide below tells you where to look.
>
> **When to use:** Step 8 of the implementation session (spot check 10 reference clubs). Do not proceed to nightly cron setup until at least 7 of 10 clubs pass.

---

## Reference Club Table

| # | Club | State | Primary League Affiliation | Expected Girls Rank | Girls Pass Threshold | Expected Boys Rank | Boys Pass Threshold | Notes |
|---|------|-------|--------------------------|---------------------|---------------------|-------------------|---------------------|-------|
| 1 | Eclipse Select SC | IL | ECNL (Girls flagship program) | Top 3 nationally | ≤ 10 | Not applicable (Girls-only program) | N/A | National ECNL powerhouse; multiple age group champions. If not top 10 Girls, investigate team coverage for IL (Affinity Soccer gap affects state leagues, not ECNL). |
| 2 | PDA (Premier Development Academy) | NJ | ECNL Girls + Boys | Top 5 nationally (Girls) | ≤ 10 | Top 20 nationally | ≤ 40 | NJ flagship ECNL club. Primary DSS audience geography. If PDA Girls is outside top 10, flag as high-priority investigation — this club has deep multi-age-group coverage and should appear near the top. |
| 3 | FC Dallas Academy | TX | ECNL Girls + Boys; MLS NEXT Boys | Top 5 nationally (Girls) | ≤ 10 | Top 10 nationally | ≤ 20 | National ECNL cornerstone. TX is a Priority 1 GotSport state. If outside expected range, check gotsport_org_id match rate for FC Dallas teams. |
| 4 | PA Classics | PA | ECNL Girls | Top 15 nationally (Girls) | ≤ 30 | Not well-known for Boys | N/A | Primary DSS demo club for Club Rankings. Must appear ≤ rank 30 Girls. If absent or >30: see diagnostic guide below — this is the highest-priority failure case for the demo. |
| 5 | NJ Wildcats (or NJ Rush) | NJ | ECNL Girls + ECNL RL | Top 20 nationally (Girls) | ≤ 35 | Limited Boys program | N/A | Strong NJ program well-known to DSS audience. Use whichever NJ Wildcats / NJ Rush variant has more teams in DB. |
| 6 | Real Colorado | CO | ECNL Girls + Boys | Top 20 nationally (Girls) | ≤ 35 | Top 35 nationally | ≤ 60 | Rocky Mountain ECNL flagship. CO is a Priority 1 GotSport state. Useful coverage cross-check (non-NJ/NE region). |
| 7 | FC Stars of Massachusetts | MA | ECNL Girls + Boys | Top 30 nationally (Girls) | ≤ 50 | Top 40 nationally | ≤ 70 | New England ECNL. MA is well-covered via GotSport. Serves as NE regional calibration check. |
| 8 | Michigan Hawks | MI | ECNL Girls + Boys | Top 35 nationally (Girls) | ≤ 60 | Top 35 nationally | ≤ 60 | Midwest ECNL flagship. Strong multi-age-group program. If absent: check MI GotSport org ID coverage. |
| 9 | Sporting Blue Valley | KS/MO | ECNL RL Girls; ECNL Boys (some age groups) | Top 50 nationally (Girls) | ≤ 80 | Top 30 nationally (Boys) | ≤ 50 | Midwest program with strong Boys presence. Boys rank expected higher than Girls due to Boys ECNL affiliation for some age groups. Useful Boys calibration anchor. |
| 10 | Utah Avalanche | UT | ECNL Girls | Top 60 nationally (Girls) | ≤ 100 | Limited Boys program | N/A | Western US ECNL. Useful coverage boundary test for less-dense GotSport regions. If absent entirely, check UT GotSport org ID coverage — may have fewer ranked teams than primary ECNL markets. |

---

## Pass/Fail Decision Rules

**Pass:** Club appears in `club_rankings` at or above the pass threshold for that gender.

**Fail — investigate:** Club appears in `club_rankings` but at a position worse than the pass threshold. Use the diagnostic guide to find the cause. Most likely: org_id match rate is lower than expected, fewer age groups qualified (maybe only 5 of 7 active), or calibration issue in the club's primary league.

**Fail — escalate:** Club does not appear in `club_rankings` at all. The club is either (a) not matched to teams in our DB, (b) has fewer than 5 qualifying age groups, or (c) the bootstrap incorrectly excluded their teams. Escalate to Presten immediately — a missing top-5 club is a pre-flight blocker for the demo.

### Go/No-Go Criteria

- **7+ clubs pass (including PA Classics Girls ≤30):** Proceed to nightly cron setup. Club Rankings is demo-ready.
- **PA Classics absent or >30 Girls, or PDA/Eclipse >20 Girls:** Stop. Investigate these clubs before proceeding. The demo names PA Classics explicitly — it must appear.
- **Fewer than 7 clubs pass:** Do not proceed. Run diagnostic queries on the failing clubs and identify the root cause before scheduling the demo session.

---

## Reference Queries

### Query A: Check all 10 reference clubs at once

```sql
SELECT
  c.name,
  cr.gender,
  cr.club_rank,
  ROUND(cr.club_rating, 1) AS club_rating,
  cr.age_groups_counted,
  cr.computed_at::DATE AS computed_date
FROM club_rankings cr
JOIN clubs c ON c.id = cr.club_id
WHERE c.name ILIKE ANY (ARRAY[
  '%eclipse select%',
  '%PDA%', '%premier development academy%',
  '%FC Dallas%',
  '%PA Classics%', '%Pennsylvania Classics%',
  '%NJ Wildcats%', '%NJ Rush%',
  '%Real Colorado%',
  '%FC Stars%',
  '%Michigan Hawks%',
  '%Sporting Blue Valley%',
  '%Utah Avalanche%'
])
ORDER BY cr.gender, cr.club_rank;
```

Run this immediately after the computation INSERT. Review each row against the pass thresholds above.

### Query B: Clubs that qualified (full list, top 20 Girls and top 20 Boys)

```sql
SELECT c.name, c.state, cr.club_rank, ROUND(cr.club_rating, 1) AS rating,
       cr.age_groups_counted, cr.gender
FROM club_rankings cr
JOIN clubs c ON c.id = cr.club_id
WHERE cr.club_rank <= 20
ORDER BY cr.gender, cr.club_rank;
```

Use this to orient: do you recognize the top 20 clubs? Are there any obvious surprises? A club you don't recognize in the top 5 is a signal worth investigating (possible name-parse false merge or miscalibration).

---

## Diagnostic Guide

If a well-known club does not appear in Club Rankings output or appears far outside the expected range, check in this order:

**1. Is the club's GotSport org_id matched to teams in Evo Draw?**
```sql
SELECT COUNT(*) FROM teams WHERE gotsport_org_id = '[org_id_from_clubs_table]';
```
If 0, the club was created in the `clubs` table but no teams were assigned to it. The bootstrap may have failed for this org_id. Check that the INSERT into `team_club_assignments` ran correctly.

**2. Do those teams have ratings in `rankings`?**
```sql
SELECT r.age_group, r.gender, r.rating, r.rank
FROM rankings r
JOIN team_club_assignments tca ON tca.team_id = r.team_id
JOIN clubs c ON c.id = tca.club_id
WHERE c.name ILIKE '%[club name]%'
ORDER BY r.gender, r.age_group;
```
If teams exist in `team_club_assignments` but have NULL ratings, they are unranked. Check whether teams have 6+ results against ranked opponents (the minimum for a ranked rating).

**3. Are at least 5 age groups qualifying (active, ranked teams)?**
```sql
SELECT r.age_group, r.gender, r.rating, r.last_game_date,
       CASE WHEN r.last_game_date > NOW() - INTERVAL '7 months' THEN 'active' ELSE 'inactive' END AS status
FROM rankings r
JOIN team_club_assignments tca ON tca.team_id = r.team_id
JOIN clubs c ON c.id = tca.club_id
WHERE c.name ILIKE '%[club name]%'
  AND r.age_group IN ('U11','U12','U13','U14','U15','U16','U17')
ORDER BY r.gender, r.age_group;
```
A club with strong teams in U14–U17 but no U11–U13 data (thin coverage in younger age groups) may fall below the 5-age-group minimum. If so, the issue is source coverage, not a computation bug.

**4. Is the bootstrap using Case A (GotSport org_id) or Case B (name-parse)?**
```sql
SELECT tca.assignment_method, COUNT(*) AS team_count
FROM team_club_assignments tca
JOIN clubs c ON c.id = tca.club_id
WHERE c.name ILIKE '%[club name]%'
GROUP BY tca.assignment_method;
```
If `assignment_method = 'name_parse'` for a club you expected to be on GotSport, check whether the club's teams have a `gotsport_org_id` populated in the `teams` table. Name-parse assignments have lower confidence (0.75 vs 1.00) and are more likely to have missed some teams.

**5. Check the top_teams JSONB for completeness**
```sql
SELECT cr.club_rank, cr.club_rating, cr.age_groups_counted, cr.top_teams
FROM club_rankings cr
JOIN clubs c ON c.id = cr.club_id
WHERE c.name ILIKE '%[club name]%';
```
Review the `top_teams` array. Are the age groups you expect represented? Are the team ratings plausible for each age group?

---

## Boys-Specific Note

Boys club rankings should only be computed and shown at DSS if Boys Option A pre-flight validation passes (per `Club Rankings — Calibration Pre-Assessment.md`). Run these three queries before computing Boys club rankings:

- **Query B:** `SELECT AVG(rating) FROM rankings WHERE age_group = 'U13' AND gender = 'M'` — if Boys avg U13 rating diverges >100 points from Girls avg U13 rating, Boys calibration may be systematically off
- **Boys GA game count:** `SELECT COUNT(*) FROM games g JOIN events e ON e.id = g.event_id WHERE e.event_tier = 'ga' AND g.gender = 'M'` — if <500, Boys GA is too thin to validate
- **MLS NEXT coverage check:** `SELECT COUNT(DISTINCT team_id) FROM rankings WHERE age_group = 'U13' AND gender = 'M' AND rating >= 1400` — top-rated Boys U13 teams should be MLS NEXT / ECNL; if the count is very low, Boys calibration may be compressed

If any of these returns a warning signal, default to Girls-only club rankings for the DSS demo.

---

## Related

- [[Club Rankings — Implementation Specification]] — implementation session guide (spot-check is Step 8)
- [[Club Rankings Design]] — methodology and expected data quality
- [[Club Rankings — Calibration Pre-Assessment]] — GA ASPIRE dependency and Boys risk
- [[DSS Demo Spec — May 2026]] — PA Classics as primary demo club (rank ≤ 30 Girls required)

*ELO — 2026-04-25*
