---
title: Event Strength Phase 1 — Post-Authorization Runbook
type: execution-runbook
author: ELO
date: 2026-04-26
status: awaiting-authorization
tags: [event-strength, phase-1, runbook, execution, evo-draw]
related: "[[Event Strength Phase 1 — Full Execution Package]]", "[[April 29 Decision Tree — Verdict Filing Document]]", "[[Event Strength Phase 1 — Rankings Impact Preview]]", "[[Event Strength Phase 2 — Prerequisites and Scope]]"
---

# Event Strength Phase 1 — Post-Authorization Runbook

> **What this document is:** Step-by-step execution guide for ELO and Presten to run on the day SENTINEL authorizes Event Strength Phase 1. Authorization is granted via the April 29 Decision Tree — Verdict Filing Document (SENTINEL Disposition section). Do not begin this runbook until SENTINEL's signature is recorded there.

---

## Header

```
Status: AWAITING AUTHORIZATION
Authorization Date: ___________ (fill April 29 after Decision Tree)
Authorizing agent: SENTINEL
ELO Execution Lead: ELO
Target completion: Within 48 hours of authorization
Actual start time: ___________
Actual completion time: ___________
```

---

## Pre-Condition Checklist (read before starting)

Before running any Phase 1 steps, confirm all four:

- [ ] April 29 Verdict Filing Document — all 4 gates (G1–G4) recorded as PASS
- [ ] SENTINEL Disposition section signed and dated
- [ ] Girls GA ASPIRE post-fix rating distribution confirmed stable (G2 PASS in Verdict Filing Document)
- [ ] No active pipeline issues flagged by FORGE (check latest FORGE briefing)

If any pre-condition is unmet: **do not proceed.** File status in next ELO briefing and notify SENTINEL. Event Strength Phase 1 is a write operation — running it over unstable calibration produces event strength ratings that will need to be recomputed once calibration settles.

---

## Step 1: Confirm Event Tier Coverage

**What:** Before touching `event_strength`, verify that all `event_tier` values in the current game dataset have a known strength classification. An unmapped tier means those games are silently excluded from Phase 1 output.

**Action:** Run this query in psql:

```sql
SELECT event_tier, COUNT(*) AS game_count
FROM games g
JOIN events e ON e.id = g.event_id
WHERE e.season = '2025-2026'
  AND e.event_tier IS NOT NULL
GROUP BY e.event_tier
ORDER BY game_count DESC;
```

Cross-check each returned `event_tier` against the tier mapping in `Rankings/Event Strength Phase 1 — Full Execution Package.md` (Section 1 remediation list and League Hierarchy).

**Gate:** Every `event_tier` value returned by this query must have a known tier assignment. Any tier appearing in the query output that is NOT in the mapping table → **STOP.** Add the tier assignment (using the Section 1 remediation format in the Execution Package) before continuing. An unmapped tier means those games compute with no event strength weight — this is a silent accuracy gap.

Record confirmed tiers here:

| event_tier | game_count | Status |
|-----------|------------|--------|
| | | confirmed / unconfirmed |

---

## Step 2: Snapshot Pre-Phase-1 Ratings

**What:** Capture current team ratings by age group and gender before Phase 1 runs. This is the comparison baseline for the Step 4 impact analysis. Must be captured **before** Step 3 — not reconstructed afterward.

**Action:** Run in psql and paste output into the table below:

```sql
SELECT age_group, gender,
       ROUND(AVG(rating), 1) AS avg_rating,
       COUNT(*) AS team_count
FROM rankings
WHERE rating IS NOT NULL
GROUP BY age_group, gender
ORDER BY gender, age_group;
```

**Pre-Phase-1 Baseline (paste output here):**

| Age Group | Gender | Avg Rating (Pre-Phase 1) | Team Count |
|-----------|--------|--------------------------|------------|
| | | | |

> **Note:** This table is read-only after Step 3 begins. Do not update it with post-Phase-1 numbers.

---

## Step 3: Run Phase 1 Recompute

**What:** Execute the Event Strength computation INSERT. This populates the `event_strength` table and is the primary Phase 1 deliverable.

**Prerequisite:** Gate A in the Execution Package must return 0 before this step (confirms GA ASPIRE fix has run). If running Phase 1 in the same session as April 29 verification, Gate A should already be confirmed.

**Action:** Open `Rankings/Event Strength Phase 1 — Full Execution Package.md`. Run in order:
1. Section 0 — Pre-Flight Gate A (confirm GA ASPIRE fix)
2. Section 0 — Pre-Flight Gate B (confirm null event_tier count)
3. Section 1 — Remediation SQL if Gate B > 0 (only if needed)
4. Section 2 — Schema: `CREATE TABLE IF NOT EXISTS event_strength`
5. Section 3 — Computation INSERT (the main CTE)

Log times:

```
Step 3 start: ___________
Step 3 end: ___________
Runtime: ___________
```

**Expected behavior:** All team ratings unchanged (Phase 1 does not write to `rankings`). Total row count for `event_strength` should be > 0 after INSERT completes.

**If the INSERT runs more than 15 minutes:** Add a season filter per the Execution Package note:
```sql
-- Add to game_context CTE WHERE clause:
AND e.season IN ('2024-2025', '2025-2026')
```

---

## Step 4: Validate Phase 1 Output

**What:** Confirm the computation produced clean, complete, and plausible results. Do not file Phase 1 as complete until all validation queries pass.

**Action:** Run all 4 validation queries from Section 4 of the Execution Package. Record results below.

**V1 — Row count by season:**
```sql
SELECT season, COUNT(*) AS event_count, ROUND(AVG(avg_team_rating), 1) AS avg_strength_rating
FROM event_strength
GROUP BY season
ORDER BY season;
```
Expected: ≥ 50 events for '2025-2026'. Result: ___________

**V2 — Strength class distribution:**
```sql
SELECT strength_class,
       COUNT(*) AS event_count,
       ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (), 1) AS pct_of_total,
       ROUND(AVG(avg_team_rating), 1) AS avg_rating_in_class
FROM event_strength
WHERE season = '2025-2026'
GROUP BY strength_class
ORDER BY avg_rating_in_class DESC;
```
Expected: High ~25%, Medium ~42%, Low ~33%. Result: ___________

**V3 — Null metric check:**
```sql
SELECT COUNT(*) AS rows_with_null_metrics
FROM event_strength
WHERE avg_team_rating IS NULL
   OR strength_class IS NULL
   OR strength_percentile IS NULL
   OR team_count IS NULL;
```
Expected: 0. Result: ___________

**V4 — DSS demo event check (High-strength U13G):**
```sql
SELECT e.event_name, es.avg_team_rating, es.strength_class, es.strength_percentile,
       es.competitive_spread, es.upset_rate, es.team_count
FROM event_strength es
JOIN events e ON e.id = es.event_id
WHERE es.age_group = 'U13G'
  AND es.strength_class = 'High'
  AND es.team_count >= 8
  AND e.season >= '2025-2026'
  AND e.event_name NOT ILIKE '%friendly%'
ORDER BY es.strength_percentile DESC
LIMIT 10;
```
Expected: ≥ 1 recognizable High-strength event. Selected demo event: ___________________________

**Rating Impact Comparison (Phase 1 should produce zero rating change):**

Run the pre-Phase-1 query from Step 2 again and compare against the Step 2 baseline. Average rating change should be 0.0 pts for all age groups.

| Age Group | Gender | Pre-Phase-1 Avg | Post-Phase-1 Avg | Delta |
|-----------|--------|-----------------|------------------|-------|
| | | | | |

Expected: all deltas = 0.0 pts. Any non-zero delta is from a concurrent pipeline run, not Phase 1. Check `computed_at` timestamps on recently changed rankings rows to confirm.

**Overall Pass/Fail:**
- V1: PASS / FAIL
- V2: PASS / FAIL
- V3: PASS / FAIL
- V4: PASS / FAIL (High-strength U13G event found: YES / NO)
- Rating impact: CONFIRMED ZERO / ANOMALY (see note)

If any FAIL: do not proceed to Step 5. Investigate per the Execution Package Section 5 (rollback instructions) and file the failure in the next ELO briefing to SENTINEL.

If shift magnitude in rating comparison exceeds 2 pts for any age group: flag to SENTINEL before filing Phase 1 complete. This may indicate a concurrent pipeline run — check timestamps.

---

## Step 5: File Phase 1 Completion Record

Once Step 4 passes:

1. Update this document's frontmatter:
   - `status:` → `completed`

2. Record the selected demo event (from V4) in the Execution Package Section 4 V4 field.

3. File an ELO briefing confirming Phase 1 complete. Briefing must include:
   - Pre vs. post average ratings (Step 2 and Step 4 comparison table)
   - One sentence: whether shifts matched the zero-rating-impact prediction from `Rankings/Event Strength Phase 1 — Rankings Impact Preview.md`
   - Named High-strength U13G demo event (for SENTINEL to reference in DSS demo script update)
   - Any age groups with unexpected behavior requiring follow-up

4. SENTINEL receives this briefing as the Phase 1 completion record. SENTINEL should update the DSS Demo Spec Step 3 to reference the confirmed event name.

---

## Step 6: Phase 2 Pre-Authorization Review

After Phase 1 is confirmed complete, ELO reviews `Rankings/Event Strength Phase 2 — Prerequisites and Scope.md` to determine whether Phase 2 prerequisites are now met.

Phase 2 is **not** authorized automatically with Phase 1 — it requires a separate SENTINEL sign-off. This step is a review, not an execution.

File status in the same briefing as Step 5: "Phase 2 prerequisites: MET / NOT MET. Reason if not met: ___________."

Key Phase 2 dependency: Post-Fix Calibration Stability confirmed as of May 5 (per `Rankings/Post-Fix Calibration Stability Monitoring Spec.md`). If Phase 2 prerequisite review happens before May 5, ELO notes the stability gate is still pending.

---

## Rollback

If Phase 1 produces unexpected results and must be reverted:

```sql
TRUNCATE event_strength;
```

This clears all Phase 1 output without dropping the table. Phase 1 can be re-run after investigating the failure. See Execution Package Section 5 for full rollback instructions including event_tier remediation revert.

---

## References

- [[April 29 Decision Tree — Verdict Filing Document]] — authorization trigger; SENTINEL sign-off required before this runbook starts
- [[Event Strength Phase 1 — Full Execution Package]] — Step 3 query source (authoritative)
- [[Event Strength Phase 1 — Rankings Impact Preview]] — Step 4 validation comparison; zero-rating-impact prediction
- [[Event Strength Phase 2 — Prerequisites and Scope]] — Step 6 reference
- [[Post-Fix Calibration Stability Monitoring Spec]] — calibration stability gate that precedes this runbook

*ELO — 2026-04-26*
