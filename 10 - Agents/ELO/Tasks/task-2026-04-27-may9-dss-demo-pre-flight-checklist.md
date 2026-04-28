---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-27
status: completed
completed: 2026-04-27
priority: high
due: 2026-05-07
deliverable: "02 - Tiger Tournaments/Projects/Rankings/May 9 DSS Demo — Pre-Flight Checklist.md"
---

# Task: May 9 DSS Demo Pre-Flight Checklist

## Context

The DSS demo is scheduled for May 9. SENTINEL's job is to certify the ranking system is demo-ready before Presten walks into the demo. Right now, that certification would require SENTINEL to improvise a spot-check — reading multiple documents and querying multiple systems informally to decide "are we ready?"

That improvisation is not acceptable for a demo. If SENTINEL misses a check and the demo reveals an anomaly (a team ranked inexplicably low, a missing ECNL cluster, a Club Rankings page that shows no teams), the damage is real and traceable to SENTINEL's failure to certify properly.

ELO owns the ranking system. ELO is the right agent to design the pre-flight checklist. SENTINEL runs the checklist on May 8 (or ELO runs it and reports to SENTINEL the morning of May 8). SENTINEL reviews the results and either certifies READY or files a queue item to Presten with specific issues on May 8.

The checklist must be done by May 7 so ELO or SENTINEL can execute it on May 8 and there is still time to address any issues before May 9.

---

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/May 9 DSS Demo — Pre-Flight Checklist.md`

---

### Header

```
Written by: ELO
For use: May 8, 2026 (day before DSS demo)
Executed by: ELO (or SENTINEL with ELO SQL queries)
Certifies: Evo Draw ranking system is demo-ready for May 9
```

---

### Section 1: Ranking System Checks

Each check is one SQL query or one vault file review. ELO provides the query and the pass/fail threshold.

**Check 1 — Brier Score Within Demo-Defensible Range**

```sql
-- Pull current Brier score from whatever table/file ELO maintains it
SELECT [brier_score_query];
```

- GREEN: Brier score ≤ [ELO states the threshold — what's defensible to claim publicly?]
- YELLOW: Slightly above threshold — demo with caveat
- RED: Score indicates significant miscalibration — escalate to SENTINEL

**Check 2 — Girls GA ASPIRE Fix Confirmed in Production**

```sql
SELECT COUNT(*) FROM [leagues/events table]
WHERE event_tier = 'ga_aspire'
AND gender = 'girls';
```

- GREEN: Count > 0 (fix applied — ga_aspire tier exists in production data)
- RED: Count = 0 (fix not applied or reverted) — STOP, escalate immediately

**Check 3 — No Anomalous Rating Distributions**

```sql
-- Check for teams with ratings outside the expected range per age group
SELECT age_group, COUNT(*) as team_count, AVG(rating) as avg_rating, 
       MIN(rating) as min_rating, MAX(rating) as max_rating
FROM rankings
GROUP BY age_group
ORDER BY age_group;
```

- GREEN: All age groups have avg_rating within [ELO states expected ranges per age group]
- YELLOW: One age group has avg outside expected range — investigate
- RED: Multiple age groups anomalous — escalate before demo

**Check 4 — ECNL Teams Present and Ranked**

```sql
SELECT COUNT(DISTINCT team_id) FROM rankings r
JOIN teams t ON r.team_id = t.id
JOIN team_leagues tl ON t.id = tl.team_id
WHERE tl.league ILIKE '%ecnl%'
AND r.ranking_date = (SELECT MAX(ranking_date) FROM rankings);
```

- GREEN: Count ≥ [ELO states minimum expected ECNL team count]
- RED: Count = 0 — ECNL teams not ranking — STOP, escalate

**Check 5 — Girls Club Rankings Populated with Reference Clubs**

```sql
SELECT COUNT(DISTINCT club_id) FROM club_rankings
WHERE gender = 'girls'
AND ranking_date = (SELECT MAX(ranking_date) FROM club_rankings);
```

- GREEN: Count ≥ [ELO states minimum reference clubs]
- RED: Count = 0 — Club Rankings not populated — escalate

**Check 6 — USARank Delta Within Expected Range**

Reference: `Rankings/USARank Delta Interpretation Spec.md` (or equivalent)

Pull the most recent USARank comparison result. Confirm the top-100 rank correlation is within the expected range defined in that spec. If no recent comparison exists (> 2 weeks old), flag as STALE — ELO recommends running a spot comparison before May 9.

- GREEN: Correlation within expected range and comparison is ≤ 2 weeks old
- YELLOW: Comparison is 2–4 weeks old — acceptable for demo but note the age
- RED: Correlation outside expected range — investigate before demo

---

### Section 2: Demo Scenario Checks

These are spot-checks on specific demo scenarios Presten will likely walk through.

**Check 7 — Sample Team Lookup Returns Reasonable Result**

ELO identifies 3 high-profile teams that Presten is likely to demo (one ECNL, one MLS NEXT Girls, one GA). For each:

| Team | Expected Rank Band | Actual Rank Band | PASS/FAIL |
|------|--------------------|-----------------|-----------|
| [ECNL team] | [expected] | [run query] | |
| [MLS NEXT Girls team] | [expected] | [run query] | |
| [GA team] | [expected] | [run query] | |

Any FAIL here warrants immediate investigation — a flagship team being ranked anomalously low or high is exactly the kind of thing that surfaces in a demo and is difficult to explain in the moment.

**Check 8 — Boys Rankings Are Present But Excluded from Demo**

Confirm Boys rankings are populated in the DB (they exist — the system isn't broken for Boys) but note that the demo plan is Girls-only for Club Rankings. This ensures SENTINEL doesn't accidentally certify "Boys Club Rankings ready" when they haven't been authorized for DSS.

```sql
SELECT COUNT(*) FROM rankings WHERE gender = 'boys' 
AND ranking_date = (SELECT MAX(ranking_date) FROM rankings);
```

- GREEN: Count > 0 (Boys rankings exist; Girls-only Club Rankings is a display decision, not a data gap)
- YELLOW: Count = 0 (Boys rankings not computed — flag to SENTINEL but not a demo blocker if Boys aren't in the plan)

---

### Section 3: SENTINEL Certification Statement

When all checks are GREEN (or YELLOW with documented acceptable exceptions), ELO files this statement:

```
ELO Pre-Flight Certification — May 8, 2026

Checks run: [date/time]
Results:
  Check 1 (Brier Score): [score] — [GREEN/YELLOW/RED]
  Check 2 (GA ASPIRE fix): [count] — [GREEN/RED]
  Check 3 (Rating distributions): [summary] — [GREEN/YELLOW/RED]
  Check 4 (ECNL teams): [count] — [GREEN/RED]
  Check 5 (Club Rankings): [count] — [GREEN/RED]
  Check 6 (USARank delta): [correlation] — [GREEN/YELLOW/RED]
  Check 7 (Demo scenarios): [PASS/FAIL per team]
  Check 8 (Boys presence): [count] — [GREEN/YELLOW]

Overall certification: [READY / READY WITH CAVEATS / NOT READY]

If NOT READY: ELO has filed a SENTINEL queue item with specific issues.

Signed: ELO — 2026-05-08
```

SENTINEL reviews this statement on May 8 and either approves the demo or escalates to Presten with specific issues.

---

### Section 4: Red-Line Conditions

Any of these conditions on May 8 results in an immediate SENTINEL escalation to Presten and a recommendation to delay or limit the demo:

- Check 2 fails (GA ASPIRE fix not in production)
- Check 4 fails (ECNL teams absent from rankings)
- Check 7: any flagship demo team ranked more than 50 positions from expected band
- Overall rating distribution shows a systematic shift that ELO cannot explain within 2 hours

Red-line conditions are non-negotiable. SENTINEL does not certify demo-ready if any red-line condition is present.

---

## Quality Standard

- Every check must be a specific SQL query or a specific vault file/field reference — not "check that ECNL teams are present."
- Every threshold must be a number — "≥ 200 ECNL teams," not "a reasonable number."
- The certification statement must be fillable by ELO in 30 minutes or less. It should not require ELO to synthesize or interpret — just fill in results against pre-defined thresholds.
- Section 4 red-line conditions must be binary — either triggered or not, with no judgment required.

## Why This Matters

The DSS demo is Evo Draw's first external presentation. If SENTINEL shows up to the demo without a formal certification that the system is ready, any anomaly that surfaces during the demo becomes a quality failure that SENTINEL did not catch. With this checklist, SENTINEL can tell Presten on May 9 morning: "ELO ran the pre-flight on May 8. All checks GREEN. System is demo-ready." That is the standard SENTINEL should meet before every public-facing event.

## References

- `Rankings/DSS Demo Spec.md` — demo scope and feature list
- `Rankings/DSS Feature Readiness Tracker.md` — which features are demo-ready
- `Rankings/Club Rankings — Girls-Only vs. Full Launch Recommendation.md` — confirms Girls-only scope for Club Rankings
- `Rankings/USARank Delta Interpretation Spec.md` or equivalent — USARank comparison baseline
- `Rankings/Post-April-28 Rating Shift Analysis Spec.md` — expected distributions after GA ASPIRE fix
- `Rankings/Rank Bands — Demo Examples.md` — reference for expected rank bands per team tier
