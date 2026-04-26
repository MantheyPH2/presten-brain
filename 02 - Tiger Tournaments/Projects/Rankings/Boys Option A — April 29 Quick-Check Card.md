---
title: Boys Option A — April 29 Quick-Check Card
type: execution-card
author: ELO
date: 2026-04-26
status: ready
related: "[[Boys Option A — Decision Document]]", "[[Boys Option A Spot Check — Presten Execution Package]]", "[[April 29 ELO Session Master Script]]"
tags: [boys, calibration, club-rankings, dss, execution, spot-check, evo-draw]
---

# Boys Option A — April 29 Quick-Check Card

```
Purpose:         Boys Option A calibration spot check
Reference:       Rankings/Boys Option A — Decision Document.md (pre-populated expected values)
Prerequisites:
  - April 28 fix confirmed: GA ASPIRE UPDATE complete + full recompute done
  - April 29 post-fix verification passed (G2 gate GREEN or YELLOW)
  - psql access to youth_soccer DB confirmed
Estimated time:  20–30 minutes
When to run:     April 29 (Step 7 of Master Script) OR any session April 29–May 5
Window closes:   May 5 — ELO needs 3 days to file verdict before May 9 go/no-go
```

---

## Step 1: Boys vs. Girls Average Rating Delta by Age Group

**Run this query:**

```sql
SELECT
  r.age_group,
  r.gender,
  COUNT(*) AS team_count,
  ROUND(AVG(r.rating)::numeric, 1) AS avg_rating,
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY r.rating) AS median_rating
FROM rankings r
WHERE r.age_group IN ('U13', 'U14', 'U15', 'U16', 'U17')
  AND r.gender IN ('M', 'F')
  AND r.rank IS NOT NULL
GROUP BY r.age_group, r.gender
ORDER BY r.age_group, r.gender;
```

**Save output:**
```
\COPY (SELECT r.age_group, r.gender, COUNT(*) AS team_count, ROUND(AVG(r.rating)::numeric, 1) AS avg_rating, PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY r.rating) AS median_rating FROM rankings r WHERE r.age_group IN ('U13', 'U14', 'U15', 'U16', 'U17') AND r.gender IN ('M', 'F') AND r.rank IS NOT NULL GROUP BY r.age_group, r.gender ORDER BY r.age_group, r.gender) TO 'reports/boys-option-a-query1-[date].txt' WITH CSV HEADER
```

**Expected output:** 10 rows — 5 age groups × 2 genders.

**Record results:**

| Age Group | Boys Avg | Girls Avg | Delta (Boys − Girls) | Result |
|-----------|----------|-----------|----------------------|--------|
| U13 | | | | PASS ≤75 / WARN 76–100 / FAIL >100 |
| U14 | | | | PASS ≤75 / WARN 76–100 / FAIL >100 |
| U15 | | | | PASS ≤75 / WARN 76–100 / FAIL >100 |
| U16 | | | | PASS ≤75 / WARN 76–100 / FAIL >100 |
| U17 | | | | PASS ≤75 / WARN 76–100 / FAIL >100 |

**Pass criterion:** All age group deltas ≤ 75 pts.
**Hard FAIL:** Any age group delta > 100 pts → Boys Club Rankings excluded from DSS. Do not continue to Step 2.

→ **If FAIL on any age group:** Note which age group(s) and the delta, then proceed to Step 2 for diagnostic context. File briefing note before ending session.
→ **If all PASS or WARN only:** Note results, skip to Step 5.

---

## Step 2 (Conditional — Only if Step 1 has any FAIL): Boys MLS NEXT Distribution

**Run this query only if Step 1 returned a FAIL:**

```sql
SELECT
  r.age_group,
  COUNT(*) AS mlsnext_team_count,
  ROUND(AVG(r.rating)::numeric, 1) AS avg_rating,
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY r.rating) AS median_rating,
  PERCENTILE_CONT(0.9) WITHIN GROUP (ORDER BY r.rating) AS top_10pct_rating,
  PERCENTILE_CONT(0.1) WITHIN GROUP (ORDER BY r.rating) AS bottom_10pct_rating,
  MAX(r.rating) AS max_rating
FROM rankings r
JOIN teams t ON t.id = r.team_id
JOIN games g ON (g.home_team_id = t.id OR g.away_team_id = t.id)
JOIN events e ON e.id = g.event_id
WHERE r.gender = 'M'
  AND r.age_group IN ('U13', 'U14', 'U15', 'U16', 'U17')
  AND e.event_tier IN ('mls_next_homegrown', 'mls_next_academy', 'mls_next')
  AND r.rank IS NOT NULL
GROUP BY r.age_group
ORDER BY r.age_group;
```

**What to look for:** Is the Boys MLS NEXT distribution compressed relative to Girls ECNL? A compressed Boys MLS NEXT distribution (top teams rated 1,400–1,500 instead of 1,600+) is a marker of Boys underweighting and explains the Step 1 divergence.

**Expected range (for age groups with ≥ 10 MLS NEXT teams):**
- Median: 1,400–1,600
- Top 10th percentile: ≥ 1,700
- Bottom 10th percentile: ≤ 1,300

This query is diagnostic context for ELO's analysis — it does not independently pass or fail. Pass/fail was determined in Step 1.

---

## Step 3: Cross-Rank Reference Team Check

Run only for age groups where Step 1 **PASSED** (or if running this card standalone without a prior FAIL):

```sql
SELECT
  t.name AS team_name,
  r.age_group,
  r.gender,
  r.rank,
  r.rating,
  r.last_game_date
FROM rankings r
JOIN teams t ON t.id = r.team_id
WHERE r.gender = 'M'
  AND r.rank IS NOT NULL
  AND t.name ILIKE ANY (ARRAY[
    '%PDA%',
    '%FC Dallas%',
    '%Real Salt Lake%',
    '%Sporting KC%',
    '%LA Galaxy%'
  ])
  AND r.age_group IN ('U14', 'U15', 'U16', 'U17')
ORDER BY r.age_group, r.rank;
```

**Pass criterion:** Known Boys elite clubs appear in the top 50 ranks for their age group.
**Concern threshold:** Any known elite Boys club ranked > 100 for their age group → flag to ELO.

---

## Step 4: File Results in Decision Document

Open `Rankings/Boys Option A — Decision Document.md`. Fill Section 1 results tables with actual values from Steps 1–3:

- Step 1: Fill all 5 age-group rows in the Query 1 table (Boys avg, Girls avg, delta, pass/fail)
- Step 2 (if run): Fill the distribution summary in the Query 3 table
- Step 3: Note any rank anomalies in the analysis section

**Do NOT write the final recommendation** — that is ELO's analysis step, not Presten's data-entry step.

---

## Step 5: File Session Note

After completing Steps 1–4, add one of the following to the next ELO briefing (or send directly to SENTINEL):

**If all PASS:** "Boys Option A Step 1 PASS. All age group deltas ≤ 75 pts. Decision Document updated. ELO analysis within 48 hours."

**If WARN (any delta 76–100 pts):** "Boys Option A Step 1 WARN. [Age group] delta = [N] pts (above 75 pt threshold but below 100 pt hard-FAIL). Decision Document updated. ELO analysis within 48 hours — conditional proceed possible."

**If FAIL (any delta > 100 pts):** "Boys Option A Step 1 FAIL. [Age group] delta = [N] pts (> 100 pt threshold). Decision Document updated with actuals. ELO analysis within 48 hours. Club Rankings Boys inclusion: SUSPENDED pending ELO verdict. Girls-only fallback spec is standing by."

---

## Quick Reference

| Query | Purpose | Pass | Fail | Automatic Action |
|-------|---------|------|------|-----------------|
| Step 1 | Boys vs. Girls avg rating delta | All deltas ≤ 75 pts | Any delta > 100 pts | FAIL → Boys excluded from DSS |
| Step 2 (conditional) | Boys MLS NEXT distribution | Median 1,400–1,600 | Median out of range > 100 pts | Diagnostic only |
| Step 3 | Reference club rank sanity | Elite clubs in top 50 | Any elite club rank > 100 | Flag to ELO |

**The only automatic action is Step 1 FAIL.** Steps 2 and 3 inform ELO's analysis — they do not independently activate or block Boys Club Rankings.

---

## References

- `Rankings/Boys Option A — Decision Document.md` — full decision record (fill Section 1 here after queries)
- `Rankings/Boys Option A Spot Check — Presten Execution Package.md` — detailed threshold derivation and save instructions
- `Rankings/Club Rankings — Girls-Only Fallback Spec.md` — activated if Step 1 FAIL
- `Rankings/April 29 ELO Session Master Script.md` — Steps 7–8 context (this card replaces the need to navigate those steps standalone)
- `02 - Tiger Tournaments/Projects/Product & Planning/DSS Feature Readiness Tracker — May 2026.md` — Boys cell this check feeds

*ELO — 2026-04-26*
