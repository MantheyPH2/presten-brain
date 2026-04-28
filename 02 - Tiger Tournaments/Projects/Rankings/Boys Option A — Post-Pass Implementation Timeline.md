---
title: Boys Option A — Post-Pass Implementation Timeline
type: implementation-timeline
author: ELO
date: 2026-04-27
status: conditional — activates on Option A PASS verdict
related: "[[Boys Option A — Pre-Verdict Evidence Checklist]]", "[[Boys Calibration Status Summary — April 2026]]", "[[Club Rankings — Girls-Only Fallback Spec]]"
tags: [boys, calibration, option-a, implementation, timeline, evo-draw]
---

# Boys Option A — Post-Pass Implementation Timeline

> **Activation condition:** This document activates when ALL of the following are true:
> - Boys Option A Pre-Verdict Evidence Checklist: **PASS** (filed by May 4–5)
> - April 29 G2 gate: Girls GA ASPIRE rating distribution confirmed healthy
> - April 29 G3 gate: Calibration stability check GREEN
> - SENTINEL has not issued a hold on Boys implementation
>
> If any trigger is unmet: file in briefing — "Boys Option A timeline not activated — [missing trigger]."
>
> If Option A is FAIL: activate `Rankings/Club Rankings — Girls-Only Fallback Spec.md` instead.

---

## Section 1 — Activation Trigger

| Condition | Source | Status (ELO fills when activating) |
|-----------|--------|-------------------------------------|
| Boys Option A verdict = PASS | `Rankings/Boys Option A — Decision Document.md` | [ ] PASS filed — [date] |
| April 29 G2: Girls GA ASPIRE distribution healthy | `Rankings/April 29 Gate Results — Structured Log.md` | [ ] G2 PASS confirmed |
| April 29 G3: Calibration stability GREEN | `Rankings/April 29 Gate Results — Structured Log.md` | [ ] G3 PASS confirmed |
| SENTINEL hold issued? | SENTINEL briefing | [ ] No hold — proceed |

When activating: ELO notes activation date in the next briefing: "Boys Option A PASS — implementation timeline activated. Session 1 target: May 5–6."

---

## Section 2 — Implementation Session Plan

### Session 1 — Boys Calibration Values Commit (target: May 5–6)

**What this session does:** Translates the Option A PASS verdict into production-ready SQL. The calibration values were defined before April 29; this session formalizes them and stages for Presten review.

**ELO's work (autonomous, ~45 minutes):**

1. Pull confirmed Boys calibration parameters from `Rankings/Boys Calibration Status Summary — April 2026.md` §3 and `Rankings/Boys Option A — Decision Document.md` §4 (PASS branch, downstream actions):
   - Confirm which Boys parameters Option A validates: Boys GA cal=100 (retain), Boys GA ASPIRE cal=100 (retain per Option A decision)
   - Confirm whether Option A spot check identified any age-group-specific adjustments needed
   - Cross-check against `Rankings/Boys Calibration Gap Analysis — April 2026.md` for any additional parameters flagged

2. Write the production UPDATE SQL for each confirmed Boys calibration change:
   ```sql
   -- Template — ELO fills after reviewing Option A Decision Document PASS branch
   -- Expected: Boys GA ASPIRE confirming cal=100 is valid (no change needed at engine level)
   -- Any age-group K-factor adjustments identified by Q3 go here
   
   -- Example structure:
   UPDATE league_hierarchy
   SET calibration_value = <Option_A_confirmed_value>
   WHERE event_tier = '<boys_tier_string>'
     AND gender = 'boys';  -- confirm if gender column exists in schema
   ```

3. File SQL at `Rankings/Boys Calibration — Production UPDATE SQL.md`
4. File with SENTINEL for review before Presten runs

**If Option A PASS means no calibration value changes are needed** (Boys GA/GA ASPIRE at 100 is confirmed correct): Session 1 documents this finding rather than writing UPDATE SQL. ELO files a one-paragraph note: "Option A PASS confirms Boys GA cal=100 is accurate. No production UPDATE required. Boys club rankings may proceed per full implementation spec."

**Presten's work (~30–60 min session):** Review Boys UPDATE SQL (or ELO's "no change needed" note), authorize, and execute if changes are required.

---

### Session 2 — Post-Application Verification (target: May 7–8)

After Presten runs the Boys calibration UPDATE (if any), ELO runs verification queries.

**Query A — Boys Rating Distribution by Tier**
```sql
-- Confirm Boys calibration applied correctly — check tier separation
SELECT 
    age_group,
    CASE 
        WHEN event_tier IN ('mlsnext_homegrown', 'mlsnext_academy') THEN 'MLS NEXT'
        WHEN event_tier IN ('ecnl_boys') THEN 'ECNL Boys'
        WHEN event_tier IN ('npl', 'dpl') THEN 'NPL/DPL'
        WHEN event_tier IN ('ga', 'ga_aspire') THEN 'GA/GA ASPIRE'
        ELSE 'Other'
    END AS tier_group,
    COUNT(*) AS team_count,
    ROUND(AVG(rating), 1) AS avg_rating,
    ROUND(MIN(rating), 1) AS min_rating,
    ROUND(MAX(rating), 1) AS max_rating
FROM teams
WHERE gender = 'boys'
GROUP BY age_group, tier_group
ORDER BY age_group, avg_rating DESC;
```

**Expected result:** MLS NEXT teams should anchor ratings in 1,400–1,700 range (validated from 5,800-game cross-league dataset). NPL/DPL should be materially below MLS NEXT. GA/GA ASPIRE should be near or below NPL. If MLS NEXT Boys fall below 1,300 after any calibration change, escalate to SENTINEL before clearing.

**Query B — Boys/Girls Cross-Gender Delta**
```sql
-- Re-run the Option A Q1 logic: confirm Boys/Girls divergence is ≤ 75 pts post-implementation
SELECT 
    age_group,
    gender,
    ROUND(AVG(rating), 1) AS avg_rating
FROM teams
WHERE gender IN ('boys', 'girls')
  AND age_group IN ('U13', 'U14', 'U15', 'U16', 'U17')
GROUP BY age_group, gender
ORDER BY age_group, gender;
-- Compare boys vs girls per age group. Delta should be ≤ 75 pts.
```

**Query C — Girls Isolation Check (confirm no cross-contamination)**
```sql
-- Spot-check 3 Girls Gold-band teams: confirm ratings unchanged after Boys recompute
-- ELO selects 3 known Girls top-band teams from the April 29 baseline snapshot
SELECT 
    team_id,
    team_name,
    age_group,
    gender,
    rating
FROM teams
WHERE team_id IN (<three_known_girls_gold_band_team_ids>)
  AND gender = 'girls';
-- Expected: ratings identical to pre-Boys-recompute values (± 1 pt floating-point variance)
```

**Pass criteria:**
- MLS NEXT Boys avg rating ≥ 1,300 across all tested age groups
- Boys/Girls delta ≤ 75 pts for all age groups tested
- Girls ratings unchanged (≤ 1 pt variance vs. April 29 baseline)

**If fail:** Escalate to SENTINEL. Boys calibration change may have been applied gender-agnostically rather than Boys-specific. Do not file "complete" until resolved.

Estimated time: 30 minutes (query execution + analysis).

---

### Session 3 — Boys Calibration Stability Baseline (target: May 9, within DSS Readiness session)

On May 9, ELO extends the existing stability monitoring queries to include Boys:

**Addition to Post-Fix Calibration Stability Monitoring Spec Query 1:**
```sql
-- Append to existing Girls GA ASPIRE trend query:
-- Add Boys MLS NEXT average to track Boys anchor stability
SELECT 
    age_group,
    gender,
    ROUND(AVG(rating), 1) AS avg_rating,
    CURRENT_DATE AS run_date
FROM teams
WHERE (gender = 'girls' AND event_tier IN ('ga', 'ga_aspire'))
   OR (gender = 'boys' AND event_tier ILIKE '%mlsnext%')
GROUP BY age_group, gender
ORDER BY gender, age_group;
```

**Addition to Query 2 (volatility check):**
```sql
-- Extend to Boys: count Boys teams with day-over-day rating change > threshold
-- [ELO integrates into existing Query 2 structure per the Stability Monitoring Spec]
```

Boys stability baseline filed alongside the Girls May 9 stability check. No standalone session needed.

**May 9 DSS readiness statement (ELO adds to readiness assessment):**
```
Boys calibration: Option A PASS confirmed [date]. 
Implementation complete [date]. 
Stability baseline established May 9.
Boys club rankings: ✅ included in DSS scope.
```

---

## Section 3 — Deliverables Checklist

| Deliverable | Owner | Target Date | Filed? |
|------------|-------|------------|--------|
| Boys Option A Decision Document — PASS verdict filed | ELO | By May 5 | [ ] |
| SENTINEL notified of PASS verdict | ELO | Same day as verdict | [ ] |
| Boys Calibration — Production UPDATE SQL (or "no change" note) | ELO | May 5–6 | [ ] |
| SENTINEL review of Boys UPDATE SQL | SENTINEL | May 6 | [ ] |
| Presten executes Boys calibration UPDATE (if changes needed) | Presten | May 7 | [ ] |
| Session 2 Verification report filed in ELO briefing | ELO | May 7–8 | [ ] |
| Boys stability baseline filed (May 9 readiness session) | ELO | May 9 | [ ] |
| DSS Feature Readiness Tracker: Boys club rankings = ✅ | ELO | May 9 | [ ] |

ELO updates this table in each briefing from May 5 onward until all items are checked.

---

## Section 4 — DSS Readiness Impact

With Boys Option A implementation complete by May 9, Evo Draw's DSS-readiness claims for Boys rankings:

| Claim | Status after implementation | Notes |
|-------|---------------------------|-------|
| Boys GA cal=100 validated | ✅ | Option A PASS confirms |
| Boys GA ASPIRE cal=100 validated | ✅ | Option A PASS confirms (per Decision; no change from Girls fix) |
| Boys MLS NEXT calibration validated | ✅ (Tier split deferred to May 18–20) | Empirical validation from 5,800-game dataset; tier split is an improvement, not a fix |
| Boys club rankings in DSS demo | ✅ | Conditional removed — proceed with full implementation spec |
| Boys/Girls divergence ≤ 75 pts threshold | ✅ (confirmed by Session 2 Query B) | |

**DSS briefing note (ELO files in May 9 assessment):** "Boys Option A PASS, implementation confirmed May [date]. Boys rankings included in DSS scope per full Club Rankings implementation spec. Boys stability monitoring active."

---

## Section 5 — Failure Contingency Reference

If, after May 4–5 Option A PASS verdict, Session 2 post-application verification FAILS:

1. ELO files incident note to SENTINEL immediately (same briefing)
2. Boys calibration change is rolled back (reverse the Session 1 UPDATE SQL)
3. Re-run recompute (same scope as Session 1)
4. Verify Girls ratings unchanged after rollback
5. Activate `Rankings/Club Rankings — Girls-Only Fallback Spec.md`
6. ELO files updated DSS Feature Readiness Tracker within 24 hours: Boys cell = ❌ (post-application verification failed)

This section is a reference pointer — the Girls-Only Fallback Spec contains the full contingency. SENTINEL activates that document, not this one, if post-application verification fails.

---

## References

- `Rankings/Boys Option A — Pre-Verdict Evidence Checklist.md` — PASS criteria that trigger this timeline
- `Rankings/Boys Option A — Decision Document.md` — PASS branch downstream actions
- `Rankings/Boys Calibration Status Summary — April 2026.md` — Option A calibration context
- `Rankings/Club Rankings — Girls-Only Fallback Spec.md` — contingency if post-application verification fails
- `Rankings/Post-Fix Calibration Stability Monitoring Spec.md` — stability monitoring to extend to Boys in Session 3
- `Product & Planning/DSS Feature Readiness Tracker — May 2026.md` — Boys calibration readiness flag

*ELO — 2026-04-27*
