---
title: May 9 DSS Demo — Pre-Flight Checklist
type: pre-flight-checklist
author: ELO
date: 2026-04-27
status: template-ready
related: "[[DSS Demo Spec — May 2026]]", "[[DSS Feature Readiness Tracker — May 2026]]", "[[Post-Fix Calibration Stability Monitoring Spec]]"
tags: [dss, pre-flight, demo, checklist, calibration, evo-draw]
---

# May 9 DSS Demo — Pre-Flight Checklist

```
Written by: ELO
For use: May 8, 2026 (day before DSS readiness assessment)
Executed by: ELO (running queries) + SENTINEL (reviewing results)
Certifies: Evo Draw ranking system is demo-ready for May 9 readiness assessment and May 16 DSS demo
```

> **Instructions:** Run all checks in order. Stop and escalate to SENTINEL if any Check 2 or Check 4 returns RED before proceeding. Fill the Section 3 certification statement when all checks complete. Target: complete all 8 checks in 45 minutes or less.

---

## Section 1: Ranking System Checks

### Check 1 — Girls GA ASPIRE Fix Confirmed in Production

```sql
SELECT 
    event_tier,
    COUNT(*) AS event_count
FROM events
WHERE event_name ILIKE '%aspire%'
  AND gender = 'girls'
GROUP BY event_tier;
```

Expected: row with `event_tier = 'ga_aspire'` exists and count > 0. Row with `event_tier = 'ga'` for ASPIRE-named Girls events is count = 0 (or absent).

- **GREEN:** At least 1 row with `event_tier = 'ga_aspire'` and Girls gender. Zero 'ga' rows for ASPIRE events.
- **YELLOW:** `ga_aspire` row exists but `ga` row for ASPIRE Girls events also has non-zero count. Partial fix — note the residual count.
- **RED:** `ga_aspire` row does not exist, OR count = 0. Fix has not applied or has been reverted. STOP — escalate to SENTINEL immediately. Do not proceed to Check 2 until resolved.

---

### Check 2 — No Anomalous Rating Distributions

```sql
SELECT 
    age_group,
    gender,
    COUNT(*) AS team_count,
    ROUND(AVG(rating), 1) AS avg_rating,
    ROUND(MIN(rating), 1) AS min_rating,
    ROUND(MAX(rating), 1) AS max_rating,
    ROUND(PERCENTILE_CONT(0.1) WITHIN GROUP (ORDER BY rating), 1) AS p10,
    ROUND(PERCENTILE_CONT(0.9) WITHIN GROUP (ORDER BY rating), 1) AS p90
FROM team_ratings
GROUP BY age_group, gender
ORDER BY age_group, gender;
```

Expected ranges (derived from validation work; applies to current season's active teams):

| Age Group | Gender | Expected avg_rating | Expected p10 | Expected p90 |
|-----------|--------|---------------------|-------------|-------------|
| U13 | Girls | 1,050–1,180 | 950–1,050 | 1,200–1,400 |
| U14 | Girls | 1,060–1,200 | 960–1,060 | 1,220–1,420 |
| U15 | Girls | 1,070–1,210 | 970–1,070 | 1,230–1,430 |
| U16 | Girls | 1,070–1,210 | 970–1,070 | 1,240–1,440 |
| U17 | Girls | 1,060–1,200 | 960–1,060 | 1,220–1,420 |
| U13 | Boys | 1,080–1,210 | 970–1,070 | 1,220–1,420 |
| U14 | Boys | 1,090–1,230 | 980–1,080 | 1,240–1,440 |
| U15 | Boys | 1,100–1,250 | 990–1,090 | 1,260–1,460 |
| U16 | Boys | 1,100–1,260 | 990–1,090 | 1,270–1,470 |
| U17 | Boys | 1,090–1,240 | 980–1,080 | 1,250–1,450 |

- **GREEN:** All active age groups have avg_rating within the expected range.
- **YELLOW:** One age group avg_rating is outside expected range by < 50 pts. Note which age group and why; proceed with demo with caveat.
- **RED:** Any age group avg_rating is outside expected range by ≥ 50 pts, OR any age group has team_count < 100. Investigate before demo. Escalate to SENTINEL if unexplained.

---

### Check 3 — ECNL Teams Present and Ranked

```sql
SELECT 
    COUNT(DISTINCT tr.team_id) AS ecnl_team_count,
    gender
FROM team_ratings tr
JOIN teams t ON tr.team_id = t.id
WHERE t.league ILIKE '%ecnl%'
  AND t.league NOT ILIKE '%ecnl rl%'
  AND t.league NOT ILIKE '%pre-ecnl%'
GROUP BY gender;
```

- **GREEN:** Girls ECNL count ≥ 400; Boys ECNL count ≥ 200.
- **YELLOW:** One gender is 200–400 (Girls) or 100–200 (Boys) — below expected but not zero.
- **RED:** Either gender count = 0. ECNL teams are absent from rankings — STOP, escalate to SENTINEL. This is a data gap or pipeline failure.

---

### Check 4 — Girls Club Rankings Populated

```sql
SELECT 
    COUNT(DISTINCT club_id) AS club_count,
    MAX(ranking_date) AS last_run
FROM club_rankings
WHERE gender = 'girls';
```

- **GREEN:** club_count ≥ 50 AND last_run within 14 days of check date.
- **YELLOW:** club_count ≥ 20 but < 50, OR last_run is 14–21 days old. Note age of data for SENTINEL.
- **RED:** club_count = 0 OR last_run is > 21 days old. Club Rankings not populated or stale. Escalate — this is a DSS demo blocker if Club Rankings is in the demo scope.

---

### Check 5 — Brier Score Within Demo-Defensible Range

```sql
-- Pull most recent Brier score from validation results
SELECT 
    gender,
    age_group,
    brier_score,
    computed_at
FROM brier_scores
WHERE computed_at = (SELECT MAX(computed_at) FROM brier_scores)
ORDER BY gender, age_group;
```

- **GREEN:** All age groups show Brier score ≤ 0.22. This indicates ELO predictions are meaningfully better than a naive 50% baseline.
- **YELLOW:** One or two age groups show Brier score 0.22–0.25. Demo with caveat — do not cite specific Brier numbers for these age groups.
- **RED:** Any age group Brier score > 0.25 (worse than naive baseline) OR Brier score table has no rows. Flag to SENTINEL; ELO investigates before claiming calibration accuracy at DSS.

*Note: If Brier table does not yet exist (computation deferred), mark this check YELLOW and note "Brier validation not yet computed — accuracy claim in demo should reference methodology, not numerical score."*

---

### Check 6 — USARank Comparison Current

Reference: `Rankings/USARank Comparison — Post-April-28 Results.md`

Pull the most recent comparison run date from that document's Section 0 metadata.

- **GREEN:** Comparison run date ≤ 14 days before demo date (May 16) AND any age group with Kendall's τ < 0.60 has an ELO explanation on file.
- **YELLOW:** Comparison run date is 14–28 days before demo. Acceptable — but note the data age in any DSS discussion.
- **RED:** No comparison has ever been run, OR comparison is > 28 days old, OR any age group shows τ < 0.50 with no ELO explanation. Run a spot comparison before demo or adjust the accuracy talking point accordingly.

---

## Section 2: Demo Scenario Checks

### Check 7 — Sample Teams Return Reasonable Rank Bands

ELO selects 3 reference teams confirmed in the demo plan (one ECNL Girls, one MLS NEXT Girls, one GA Girls — see `Rankings/Rank Bands — Demo Examples Document.md`):

```sql
SELECT 
    t.team_name,
    t.league,
    tr.rating,
    tr.rank_band,
    tr.age_group
FROM team_ratings tr
JOIN teams t ON tr.team_id = t.id
WHERE t.team_name IN (
    '[ECNL reference team from Demo Examples]',
    '[MLS NEXT Girls reference team from Demo Examples]',
    '[GA Girls reference team from Demo Examples]'
);
```

| Team | Expected Rank Band | Actual Rank Band | Delta | PASS/FAIL |
|------|--------------------|-----------------|-------|-----------|
| [ECNL Girls reference] | 1–3 | [run query] | | |
| [MLS NEXT Girls reference] | 1–5 | [run query] | | |
| [GA Girls reference] | 5–15 | [run query] | | |

ELO fills expected rank bands from the Demo Examples Document before May 8.

- **PASS:** Each team's actual rank band is within ± 5 positions of expected.
- **FAIL:** Any team is > 10 positions from expected rank band. Investigate before demo — this may indicate a data issue, team merge problem, or calibration shift since the demo examples were selected.

---

### Check 8 — Boys Rankings Present (Demo Scope Awareness)

```sql
SELECT 
    COUNT(*) AS boys_ranked_teams
FROM team_ratings
WHERE gender = 'boys';
```

This check confirms Boys rankings data exists and clarifies the demo scope.

- **GREEN:** boys_ranked_teams > 0. Boys data exists; Club Rankings demo is Girls-only per current plan (consistent with data state).
- **YELLOW:** boys_ranked_teams = 0. Boys pipeline has not run. This does not block a Girls-only demo, but flag to SENTINEL — Boys rankings absence should be intentional, not a pipeline failure.

---

## Section 3: SENTINEL Certification Statement

*ELO fills this after all 8 checks complete on May 8:*

```
ELO Pre-Flight Certification — May 8, 2026

Checks run: [date/time]

Results:
  Check 1 (GA ASPIRE fix): event_tier='ga_aspire' count=[N], residual 'ga' count=[N] — [GREEN/YELLOW/RED]
  Check 2 (Rating distributions): [N] age groups checked, [N] outside expected range — [GREEN/YELLOW/RED]
  Check 3 (ECNL teams): Girls=[N], Boys=[N] — [GREEN/YELLOW/RED]
  Check 4 (Club Rankings): club_count=[N], last_run=[date] — [GREEN/YELLOW/RED]
  Check 5 (Brier score): [range of scores across age groups] — [GREEN/YELLOW/RED]
  Check 6 (USARank comparison): last_run=[date], weakest τ=[N] ([age group]) — [GREEN/YELLOW/RED]
  Check 7 (Demo scenarios): [PASS/FAIL per team, with delta]
  Check 8 (Boys presence): boys_ranked_teams=[N] — [GREEN/YELLOW]

YELLOWs (acceptable exceptions):
  [list any YELLOW results with explanation]

Overall certification: [READY / READY WITH CAVEATS / NOT READY]

If READY WITH CAVEATS: demo should not reference [list specific numbers or features flagged YELLOW].
If NOT READY: ELO has filed a SENTINEL queue item with specific issues and remediation options.

Signed: ELO — 2026-05-08
```

SENTINEL reviews and signs off before the May 9 readiness assessment session.

---

## Section 4: Red-Line Conditions

Any of these on May 8 triggers immediate SENTINEL escalation and a recommendation to limit or delay the demo:

1. **Check 1 RED:** GA ASPIRE fix not confirmed in production — the most important calibration change for the demo has not applied or has reverted.
2. **Check 2 RED:** Any age group avg_rating is outside expected range by ≥ 50 pts without explanation — systematic anomaly that may be visible during the demo.
3. **Check 3 RED:** Either ECNL gender count = 0 — ECNL teams absent from rankings is a data gap visible during a team lookup.
4. **Check 7 FAIL by > 10 positions:** Any reference demo team is ranked > 10 positions from expected — this exact team will be shown in the demo and a clearly wrong ranking cannot be explained in the moment.

Red-line conditions are non-negotiable. SENTINEL does not certify READY if any red-line condition is unresolved.

---

## References

- [[DSS Demo Spec — May 2026]] — demo scope and feature list
- [[DSS Feature Readiness Tracker — May 2026]] — which features are demo-ready
- [[Club Rankings — Girls-Only vs. Full Launch Recommendation]] — confirms Girls-only Club Rankings scope
- [[USARank Comparison — Post-April-28 Results]] — USARank comparison baseline (Check 6)
- [[Rank Bands — Demo Examples Document]] — reference teams for Check 7
- [[Post-April-28 Rating Shift Analysis Spec]] — expected distributions after GA ASPIRE fix
- [[Post-Fix Calibration Stability Monitoring Spec]] — stability monitoring leading to this check

*ELO — 2026-04-27*
