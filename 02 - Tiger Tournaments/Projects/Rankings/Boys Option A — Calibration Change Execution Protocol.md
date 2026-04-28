---
title: Boys Option A — Calibration Change Execution Protocol
type: execution-protocol
author: ELO
date: 2026-04-27
status: conditional — executes only if Boys Option A verdict = PASS
related: "[[Boys Option A — Decision Document]]", "[[Boys Option A — Pre-Verdict Evidence Checklist]]", "[[Boys Option A — Post-Pass Implementation Timeline]]"
tags: [boys, calibration, option-a, execution, protocol, evo-draw]
---

# Boys Option A — Calibration Change Execution Protocol

> **Condition:** This protocol executes only after Boys Option A verdict = **PASS** is filed per `Rankings/Boys Option A — Decision Document.md`. If verdict = FAIL, activate `Rankings/Club Rankings — Girls-Only Fallback Spec.md` instead.
>
> **Purpose:** Make the day-after-PASS session a clean execution session, not a design session. All steps, SQL, and expected values are written here before the verdict.

---

## Section 1: Option A — What Changes

Based on `Rankings/Boys Calibration Status Summary — April 2026.md` §3 and the decision framework from `Rankings/Boys Option A — Decision Document.md`:

| Parameter | Current Value | Option A Value | Change Type | Notes |
|-----------|--------------|----------------|-------------|-------|
| Boys GA cal value | 100 | 100 (retain) | No change | Option A confirms this is correct |
| Boys GA ASPIRE cal value | 100 | 100 (retain) | No change | Set by April 2026 decision; Option A confirms Boys GA ASPIRE = Boys GA (no overcalibration found for Boys) |
| Boys K-factor (U13B) | Not separately validated | Validate via Option A Q1/Q2 | TBD | If Q1 divergence is within threshold, current K is acceptable |
| Boys MLS NEXT tier split | Undeployed (deferred May 18–20) | Remains deferred | No change (separate window) | FORGE event classifier required before this deploys |

**Key finding:** Option A is primarily a **validation check**, not a parameter change. If PASS, the primary action is confirming Boys GA cal=100 is correct and removing the uncertainty flag from Boys club rankings — not deploying new calibration values. The MLS NEXT Boys tier split (Homegrown 160 / Academy 135) is a confirmed improvement but deploys separately May 18–20.

**If the Option A spot check Q3 (Boys GA top-team sanity check) identifies a specific anomaly** — e.g., a Boys age group shows unexpected top-10 composition — ELO adds the specific correction here before filing the PASS verdict. Section 4 below accommodates that scenario.

---

## Section 2: Pre-Execution Authorization Gate

Before executing any calibration change:

- [ ] Boys Option A Decision Document filed with verdict = **PASS**
- [ ] Pre-Verdict Evidence Checklist confirmed complete (all 3 queries run, 5-day window honored, criteria written before results)
- [ ] SENTINEL notified of PASS verdict (ELO briefing)
- [ ] SENTINEL has confirmed execution authorization (can be same-day as verdict; one-line in SENTINEL response)
- [ ] No active ELO flags blocking Boys calibration: check most recent briefing for any Level 2/3 calibration instability flags
- [ ] April 29 G2 and G3 gates both PASS (confirms Girls calibration baseline is stable before adding Boys context)

---

## Section 3: Pre-Execution Baseline Snapshot

Before making any changes to Boys calibration (even if changes are minimal), capture the current Boys ratings baseline as a rollback reference:

```sql
-- Boys baseline before Option A execution
-- Save this output before any UPDATE or recompute
SELECT 
    age_group,
    COUNT(*) AS team_count,
    ROUND(AVG(rating), 1) AS avg_rating,
    ROUND(PERCENTILE_CONT(0.1) WITHIN GROUP (ORDER BY rating), 1) AS p10,
    ROUND(PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY rating), 1) AS median,
    ROUND(PERCENTILE_CONT(0.9) WITHIN GROUP (ORDER BY rating), 1) AS p90,
    ROUND(MAX(rating), 1) AS max_rating
FROM teams
WHERE gender = 'boys'
GROUP BY age_group
ORDER BY age_group;
```

File this output. It is the rollback reference and the Session 2 comparison baseline.

---

## Section 4: Implementation Steps

### Case A: No calibration value changes required (expected outcome)

If Option A PASS confirms Boys GA cal=100 and Boys GA ASPIRE cal=100 are both correct, and Q3 shows no anomalous top-team composition:

1. **No UPDATE SQL needed.** The Option A verdict is the authorization to proceed with Boys club rankings.
2. ELO documents: "Boys GA cal=100 confirmed. Boys GA ASPIRE cal=100 confirmed. No calibration value changes required."
3. ELO updates DSS Feature Readiness Tracker: Boys club rankings cell = ✅
4. ELO files Boys Option A Post-Pass Implementation Timeline, Session 1 result: "Confirmation only — no production UPDATE required."

### Case B: Targeted calibration adjustment needed (if Q3 or Q1 reveals specific anomaly)

If Option A identifies a specific Boys calibration issue (e.g., one Boys age group shows divergence > threshold):

1. **Identify the specific parameter:** ELO traces the anomaly to a specific `event_tier` + `age_group` combination
2. **Write the correction SQL:**
   ```sql
   -- Template — fill after Option A results reveal specific issue
   UPDATE league_hierarchy
   SET calibration_value = <corrected_value>
   WHERE event_tier = '<affected_boys_tier>'
     AND gender = 'boys';  -- confirm if gender filter applies in schema
   ```
3. **Run verification:** Section 2 of the Post-Pass Timeline verifies the change worked
4. **Run targeted recompute:**
   ```
   node compute-rankings.js --age-group <affected_age_group> --gender boys
   -- If partial recompute is not supported: full recompute
   -- node compute-rankings.js --full
   -- Confirm recompute command with Presten
   ```

### Recompute timing guidance

- If only confirming (Case A): no recompute needed
- If any Boys calibration value changes (Case B): targeted partial recompute for affected age groups
- Do NOT run Boys recompute within 48 hours of the U13/U14 fix deployment (scheduled May 17) — interleaved recomputes within 2 days create attribution noise

---

## Section 5: Post-Execution Verification

**Expected shifts (pre-committed before results arrive):**

For Case A (no changes): all Boys ratings should be **identical** to the Section 3 baseline (± 1 pt floating-point variance from a no-change recompute, if any).

For Case B (targeted calibration adjustment):
- The affected Boys age group's **average rating shifts in the direction predicted by Q1/Q2** in the Decision Document
- Boys MLS NEXT teams remain anchored in the **1,400–1,700 range** — empirically validated from 5,800 games; should not be affected by a Boys GA calibration change
- Girls ratings **unchanged** — Boys and Girls parameters are applied independently

**Verification queries:**

```sql
-- Post-execution: confirm Boys MLS NEXT anchor is stable
SELECT 
    age_group,
    ROUND(PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY rating), 1) AS mls_next_median,
    COUNT(*) AS team_count
FROM teams
WHERE gender = 'boys'
  AND event_tier ILIKE '%mlsnext%'
GROUP BY age_group
ORDER BY age_group;
-- Expected: median in 1,400–1,700 range for all Boys MLS NEXT age groups
```

```sql
-- Confirm Boys/Girls delta is within Option A threshold (≤ 75 pts divergence)
SELECT 
    age_group,
    gender,
    ROUND(AVG(rating), 1) AS avg_rating
FROM teams
WHERE gender IN ('boys', 'girls')
  AND age_group IN ('U13', 'U14', 'U15', 'U16', 'U17')
GROUP BY age_group, gender
ORDER BY age_group, gender;
```

```sql
-- Confirm Girls ratings unchanged (3 known Gold-band teams as sentinel)
-- Replace team_ids with actual IDs from April 29 baseline snapshot
SELECT team_id, team_name, age_group, gender, rating
FROM teams
WHERE team_id IN (<girls_gold_band_id_1>, <girls_gold_band_id_2>, <girls_gold_band_id_3>)
  AND gender = 'girls';
```

**Pass criteria:**
- Case A: all Boys ratings within ±1 pt of Section 3 baseline
- Case B: Boys/Girls delta ≤ 75 pts; Boys MLS NEXT median ≥ 1,300; Girls unchanged (≤ 1 pt)

**If any criterion fails:** Hold filing. Escalate to SENTINEL with specific query output showing the failure.

---

## Section 6: Post-Calibration Stability Monitoring

After Option A execution (Case A or B), ELO monitors Boys stability using the Post-Fix Calibration Stability Monitoring Spec queries extended to Boys:

**Monitoring window:** 3 pipeline runs after execution (approximately May 8–12)

**Stability threshold:** Average Boys rating shift between pipeline runs < 5 pts per age group

**Monitoring additions (integrated into existing Post-Fix monitoring queries):**
- Add Boys MLS NEXT avg to the daily trend query (Query 1 extension)
- Add Boys teams to the volatility count (Query 2 extension)
- Cross-gender check: Boys MLS NEXT should remain within 150 pts of Girls ECNL avg (different sports, different populations — this is a gross sanity check, not an equivalence claim)

**If instability > 5 pts/age group for 2+ consecutive runs:** File Level 2 flag in briefing. Investigate before clearing Boys for DSS demo.

---

## Section 7: Rollback Plan

If post-execution verification fails or stability check shows unexpected drift:

1. Identify the specific change that caused the issue (Case B SQL or recompute scope)
2. Reverse the calibration change:
   ```sql
   -- Use Section 3 baseline snapshot values to restore pre-execution state
   UPDATE league_hierarchy
   SET calibration_value = <original_value_from_baseline>
   WHERE event_tier = '<affected_tier>'
     AND gender = 'boys';
   ```
3. Re-run recompute (same scope as Section 4 Case B)
4. Verify return to baseline using Section 3 snapshot values (Boys ratings should match ± 1 pt)
5. File SENTINEL alert: "Boys Option A rolled back — post-execution verification failed at [specific check]. Girls-Only Fallback Spec activated."

**Rollback time estimate:** Recompute 45–90 minutes + 30 min verification. Total: ~2 hours. DSS impact: activates Girls-Only scope change with 8+ days remaining before May 16.

---

## Section 8: DSS Timing Note

| Timeline | Scenario | DSS Implication |
|----------|----------|----------------|
| Option A PASS by May 5 + execution complete May 7–8 | Nominal | Boys rankings DSS-ready by May 9 readiness assessment. Full club rankings scope. |
| Option A PASS May 5 but execution delayed to May 10 | Late | 6 days stability monitoring before DSS. Tight but acceptable — file in DSS readiness as "Boys calibration confirmed, monitoring window 6 days." |
| Post-application verification FAILS (May 8–10) | Contingency | Girls-Only fallback activates. FORGE has 6–8 days to complete Girls-only club rankings before May 16. |
| Option A verdict not filed until May 9 | Late | Boys implementation cannot complete before May 9 readiness assessment. DSS scope defaults to Girls-only unless SENTINEL grants extension. |

---

## References

- `Rankings/Boys Option A — Decision Document.md` — verdict filing document with PASS/FAIL branches
- `Rankings/Boys Option A — Pre-Verdict Evidence Checklist.md` — PASS criteria this protocol follows
- `Rankings/Boys Option A — Post-Pass Implementation Timeline.md` — session schedule and deliverables checklist
- `Rankings/Boys Option A — Query Pack Validation Note.md` — confirms query alignment for spot check queries
- `Rankings/Post-Fix Calibration Stability Monitoring Spec.md` — stability monitoring to extend to Boys
- `Rankings/Club Rankings — Girls-Only Fallback Spec.md` — contingency if execution fails
- `Rankings/Boys Calibration Status Summary — April 2026.md` — Boys calibration context

*ELO — 2026-04-27*
