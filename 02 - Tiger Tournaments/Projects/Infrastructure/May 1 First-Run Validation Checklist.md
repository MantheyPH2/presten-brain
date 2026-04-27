---
title: May 1 First-Run Validation Checklist
tags: [infrastructure, pipeline, validation, may1, first-run, evo-draw, forge]
created: 2026-04-26
updated: 2026-04-26
author: FORGE
status: ready — FORGE completes morning of May 2
task: task-2026-04-27-post-may1-first-run-validation-checklist
---

# May 1 First-Run Validation Checklist

> **Run on:** May 2, 2026 morning — after the first May 1 pipeline run completes.  
> **Purpose:** Verify the pipeline produced correct output — not just that the process ran. A GREEN runbook exit does not confirm data integrity. This checklist does.  
> **Target time:** ≤ 30 minutes in psql + log review.

---

## Pre-Checklist: Reference Numbers

Fill in before running any checks. All comparison checks use these as the baseline.

```
Pre-pipeline total games in DB (from Source Ingestion Baseline — Pre-Launch Snapshot, Section 2): ___________
Expected new games from May 1 run: [TBD — fill from Stage 1 TX test crawl results when available; see Infrastructure/Stage 1 Backfill Results Report.md]
Expected active org-IDs in Stage 1 crawl config: 1 (TYSA — Texas Youth Soccer Association)
Confirmed org-ID value for TYSA: [TBD — fill after browser session confirms org_id]
Expected pipeline runtime (from May 1 Deployment Day-Of Runbook): ___________
Actual May 1 run start time (from log): ___________
Actual May 1 run end time (from log): ___________
```

---

## Check 1: Game Count Delta

**Query to run:**

```sql
SELECT COUNT(*) AS new_games_ingested
FROM games
WHERE created_at >= '[paste May 1 pipeline run start timestamp]';
```

| Metric | Expected | Actual | Status |
|--------|----------|--------|--------|
| New games ingested | [N from Stage 1 estimate — TBD] | | PASS / FLAG |
| New games > 0 (minimum bar) | > 0 | | PASS / FLAG |
| Total games in DB after run | pre-pipeline total + new | | PASS / FLAG |

**PASS criteria:** Actual new games ≥ 50% of expected estimate, AND at least 1 new game ingested.  
**FLAG criteria:** 0 new games despite GREEN pipeline exit, OR actual count < 50% of expected.  
**If FLAG:** Query which org_id produced results. If TYSA org_id returned empty, verify the org_id value is correct and the TX configuration is pointing to the right endpoint. Escalate to SENTINEL.

---

## Check 2: Org-ID Coverage Verification

Verify each Stage 1 org-ID actually contributed games:

```sql
SELECT org_id, COUNT(*) AS games_ingested
FROM games
WHERE created_at >= '[May 1 pipeline run start timestamp]'
  AND source IN ('gotsport_api', 'gotsport')
GROUP BY org_id;
```

| Org-ID | Entity | Games Ingested | Status |
|--------|--------|---------------|--------|
| [TYSA org_id — confirm after browser session] | Texas Youth Soccer Association (TYSA) | | PASS / ZERO |

**PASS criteria:** TYSA org-ID contributed ≥ 1 game.  
**ZERO criteria:** TYSA org-ID returned 0 games — this is a problem if games exist in the TX spring season calendar. Check:
1. Is the org_id value correct? Compare to browser-session-confirmed value.
2. Are TX games currently in season? (Spring season runs March–June — yes, games expected.)
3. Check DLQ for TYSA-related errors.
If ZERO and no DLQ explanation: flag to SENTINEL before May 2 cron run fires.

---

## Check 3: Team Identity Integrity

Spot-check that newly ingested games are assigned to the correct team records. Prevents phantom teams or misassignment from going undetected.

```sql
SELECT g.id AS game_id, 
       ht.name AS home_team, ht.age_group AS home_age, ht.gender AS home_gender,
       at.name AS away_team, at.age_group AS away_age, at.gender AS away_gender,
       g.match_date, g.event_name
FROM games g
JOIN teams ht ON ht.id = g.home_team_id
JOIN teams at ON at.id = g.away_team_id
WHERE g.created_at >= '[May 1 pipeline run start timestamp]'
ORDER BY RANDOM()
LIMIT 5;
```

For each of the 5 results:

| Game ID | Home Team | Away Team | Home/Away Match? | Age/Gender Consistent? | Notes |
|---------|-----------|-----------|-----------------|----------------------|-------|
| | | | YES / FLAG | YES / FLAG | |
| | | | YES / FLAG | YES / FLAG | |
| | | | YES / FLAG | YES / FLAG | |
| | | | YES / FLAG | YES / FLAG | |
| | | | YES / FLAG | YES / FLAG | |

**"Home/Away Match?"** — team names look like real youth soccer club names (not test data, generic names, or duplicates).  
**"Age/Gender Consistent?"** — home team and away team share the same age group and gender (both U13 Girls, both U16 Boys, etc.).

**FLAG criteria:** Any game where home/away teams have different age groups or genders — indicates misassignment. Any game where team names look like test data or contain null/placeholder values.

---

## Check 4: ELO Update Confirmation

Confirm ELO recompute ran for the newly ingested games. A pipeline that ingests games but skips ELO is a silent failure.

```sql
SELECT COUNT(DISTINCT team_id) AS teams_with_updated_elo
FROM elo_history
WHERE updated_at >= '[May 1 pipeline run start timestamp]';
```

| Metric | Expected | Actual | Status |
|--------|----------|--------|--------|
| Teams with updated ELO ratings | > 0 (if any games ingested) | | PASS / FAIL |

**PASS criteria:** At least 1 team ELO updated, assuming new games were ingested in Check 1.  
**FAIL criteria:** 0 teams updated despite Check 1 showing new games. The ELO/Glicko-2 computation step may have silently failed. Check pipeline log for `STEP 6` or `STEP 7` output (ranking computation). Escalate to SENTINEL immediately — do NOT run May 2 cron until resolved.

> [!note] If Check 1 was ZERO
> If no new games were ingested (Check 1 FLAG), a Check 4 result of 0 is expected. Do not flag Check 4 independently in that case — the root issue is Check 1.

---

## Check 5: Dead Letter Queue Review

Check the Dead Letter Queue for May 1 processing failures.

```sql
SELECT COUNT(*) AS dlq_entries
FROM dead_letter_queue
WHERE created_at >= '[May 1 pipeline run start timestamp]';
```

**DLQ entry count from May 1 run:** ___________

```sql
-- If count >= 6, list all entries
SELECT team_id, error_type, error_message, attempt_count, created_at
FROM dead_letter_queue
WHERE created_at >= '[May 1 pipeline run start timestamp]'
ORDER BY created_at ASC;
```

| DLQ Count | Interpretation | Action |
|-----------|---------------|--------|
| 0 | Clean run | No action |
| 1–5 | Expected noise | Log in briefing, no escalation |
| 6–15 | Elevated — review entries | File DLQ entries in FORGE briefing; classify using `Infrastructure/Pipeline Error Classification Taxonomy.md` |
| 16+ | Abnormal | SENTINEL flag required. Classify every entry. Do not proceed to Stage 3 backfill until root cause identified. |

List all DLQ entries if count ≥ 6 (use Tier 1/2/3 classification from Pipeline Error Classification Taxonomy):

| Team ID | Error Type Code | Error Message | Attempts | Tier |
|---------|----------------|---------------|----------|------|
| | | | | |

---

## Check 6: Archive State Confirmation

Complete this check ONLY if the archive dry-run ran and Presten authorized a production archive pass before May 1.

**Was the archive production pass run before May 1?** YES / NO

If YES:

```sql
SELECT COUNT(*) AS active_teams_in_run
FROM teams
WHERE archived_at IS NULL
  AND last_seen_at >= NOW() - INTERVAL '90 days';
```

| Metric | Expected | Actual | Status |
|--------|----------|--------|--------|
| De-prioritized teams excluded from pipeline run | ~155–165 (of 180 de-prioritized) | | PASS / FLAG |
| Active team count processed in this run | [total active - ~160 archived] | | |

If NO (archive did not run before May 1): note this. The 180 de-prioritized teams ran through the pipeline and likely returned empty payloads. This is expected noise — do NOT flag as errors. These teams will generate DLQ entries if GotSport returns 404 for them, which is normal.

---

## Final Verdict

After completing all checks:

```
Check 1 PASS and all checks 2–5 PASS:  [ ] → GREEN — pipeline running correctly
                                              → File GREEN status in FORGE briefing
                                              → Stage 3 backfill authorized per Morning-After Runbook

Any Check 2–5 FAIL while Check 1 PASS: [ ] → YELLOW — partial success, specific issue
                                              → Describe the specific failure in FORGE briefing
                                              → Do NOT proceed to Stage 3 until SENTINEL reviews
                                              → Run May 2 cron as scheduled (issue does not block)

Check 1 at <10% of expected or ZERO:   [ ] → RED — pipeline ran but produced near-nothing
                                              → File RED status in FORGE briefing immediately
                                              → Do NOT run Stage 3 backfill
                                              → SENTINEL review required before May 2 cron
```

---

## Validation Summary

Fill after completing all checks:

```
Validation date: May 2, 2026
Validator: FORGE
Check 1 (Game Count): PASS / FLAG — [N] new games ingested
Check 2 (Org-ID Coverage): PASS / ZERO — TYSA: [N] games
Check 3 (Team Identity): PASS / FLAG — [N/5] games spot-checked OK
Check 4 (ELO Update): PASS / FAIL — [N] teams updated
Check 5 (DLQ): PASS / ELEVATED — [N] DLQ entries
Check 6 (Archive): PASS / FLAG / N/A

FINAL VERDICT: GREEN / YELLOW / RED
Stage 3 backfill authorized: YES / NO / PENDING SENTINEL
Notes for FORGE briefing: ___________
```

---

## References

- `Infrastructure/May 1 Deployment — Day-Of Runbook.md` — launch execution this validates
- `Infrastructure/Pipeline Error Classification Taxonomy.md` — error codes for DLQ classification
- `Infrastructure/Source Ingestion Baseline — Pre-Launch Snapshot.md` — baseline game count (Section 2)
- `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` — Stage 1 org-ID list (TYSA)
- `Infrastructure/Daily Pipeline Morning-After Validation Runbook.md` — parallel validation document for cron run

*FORGE — 2026-04-26*
