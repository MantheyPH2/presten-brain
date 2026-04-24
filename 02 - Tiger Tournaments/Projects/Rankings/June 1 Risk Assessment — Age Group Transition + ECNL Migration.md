---
title: June 1 Risk Assessment — Age Group Transition + ECNL Migration
aliases: [June 1 Risk, Transition Risk Assessment, ECNL Migration Risk]
tags: [rankings, migration, risk, ecnl, age-groups, evo-draw]
created: 2026-04-23
updated: 2026-04-23
author: ELO Agent
status: final — ready for Presten review
due: 2026-04-28
---

# June 1 Risk Assessment — Age Group Transition + ECNL Migration

> **Purpose:** Two significant changes converge on June 1, 2026: the ECNL platform migration (TGS → GotSport) and the age group transition (birth-year to school-year). Their simultaneous occurrence creates a compounding risk to rating continuity that FORGE's ECNL validation script does not address. This document assesses the rating system implications and delivers a minimum viable patch list prioritized for Presten.

---

## Section 1: Transition Migration Spec Audit

The existing spec `[[Age Group Transition Migration Spec 2026-06-01]]` was read in full. Assessment against the six required coverage items:

| Coverage Item | Status | Notes |
|---|---|---|
| Birth year to age group mapping logic | ✅ Complete | Section 1 defines `age_group_advance()` with school-year vs birth-year tables |
| Rating carryover formula | ✅ Complete | Section 1: match_count ≥ 10 + active within 60 days = carry; otherwise reset |
| RD reset values post-transition | ✅ Complete | Section 2 table: per-age-group formula `max(end_RD × 1.15/1.2, floor)` |
| `birth_year_cohort` as source of truth | ✅ Complete | Section 3: SQL DDL + backfill logic + mapping rule for birth-year vs school-year |
| Dual-system teams handling | ✅ Complete | Section 4: primary-league rule for mixed ECNL/MLSNEXT teams; `cross_system_age_group` flag |
| Age group transition order of operations | ⚠️ Partial | Spec describes the migration steps but does not explicitly state whether age group advancement runs BEFORE or AFTER the nightly Glicko compute in the operational pipeline. **This is the one gap.** See Section 2, Risk C. |

**Conclusion:** The transition migration spec is substantially complete. One operational sequencing ambiguity identified (Risk C below). No full rewrite required — a single paragraph addition closes the gap.

### Gap Fix: Order of Operations (add to spec Section 5)

The age group transition migration must run in this sequence relative to the nightly pipeline:

```
1. Stop the nightly pipeline (or run migration in a maintenance window)
2. Apply calibration parameter updates to calibration.json (if approved)
3. Run birth_year_cohort backfill (ALTER TABLE + UPDATE games/rankings)
4. Run age group advancement for all carry-forward teams
5. Apply season-start RD expansion
6. Run full ranking recomputation (compute-rankings.js)
7. Validate rating distribution (Section 5 monitoring table in spec)
8. Re-enable nightly pipeline
```

The critical ordering rule: **age group advancement must run before ranking recomputation**, not after. If recomputation runs first, teams are computed under their 2025-26 age group bucket and then advanced — resulting in U14 ratings for teams in the U13 slot for one pipeline cycle. This causes one cycle of incorrect rankings that the next run corrects, but it could appear as a ranking spike to users.

---

## Section 2: ECNL Migration — Rating System Impact Analysis

FORGE's validation script (`validate-ecnl-migration.js`) handles data continuity: team ID bridging, game record matching, rating deviation thresholds. It does NOT address:
- What happens to a team's Glicko-2 parameters when its source changes
- How source-change timing interacts with age group transition timing
- Whether the calibration parameter pipeline runs before or after the platform dedup pass

The following three risks are ELO's domain.

---

### Risk A: ECNL Team ID Discontinuity → Orphaned Rating History

**Mechanism:** When ECNL teams move from TGS to GotSport, they receive new GotSport team IDs. If the dedup logic fails to create a `team_merges` record linking the TGS ID to the GotSport ID, the team's Glicko-2 history stays on the TGS record (which is now `is_hidden = true` and no longer active). The new GotSport record starts at the default provisional rating (~1000). A previously Gold-band team becomes unranked overnight.

**Scale at risk:** The pre-migration snapshot captured all TGS-primary ECNL teams. FORGE's validation script (Check 1) flags teams without confirmed GotSport links as `no_gotsport_match`. **The intervention threshold is 5% of snapshot teams.** If this threshold is breached, Rating System impact is severe — potentially hundreds of top-100 teams reset.

**Detection query** (complements FORGE's Check 1 — run after post-migration validation):

```sql
-- Find ECNL teams whose TGS records are now hidden but have no matching rankings entry
-- from a GotSport record — these teams lost their rating history
SELECT
  t.team_id,
  t.team_name,
  r_old.rating AS last_known_rating,
  r_old.rank   AS last_known_rank,
  r_old.rd     AS last_known_rd,
  r_old.last_game_date,
  'ORPHANED_HISTORY' AS risk_flag
FROM teams t
JOIN rankings r_old ON r_old.team_id = t.team_id
WHERE
  -- Teams that had TGS as their primary source
  (SELECT COUNT(*) FROM games g
   WHERE (g.home_team_id = t.team_id OR g.away_team_id = t.team_id)
     AND g.source IN ('tgs', 'tgs_athleteone')
     AND g.is_hidden = false) = 0  -- no longer has visible TGS games
  AND
  -- But still have a rankings entry suggesting they haven't been re-rated under GotSport
  r_old.last_game_date < '2026-06-01'  -- no post-migration game date
  AND r_old.age_group IN ('U13','U14','U15','U16','U17')  -- active demo age groups
ORDER BY r_old.rating DESC;
```

**Mitigation:** FORGE's pre-migration snapshot + post-migration validation script is the primary defense. ELO's role: **ensure the rating recompute for affected age groups runs AFTER FORGE's transition dedup pass completes**, not before. If the dedup runs after recompute, orphaned history remains orphaned for one additional ranking cycle.

---

### Risk B: Simultaneous ID Change + Age Group Transition → Double Reset

**This is the highest-severity compounding risk.**

**Scenario:**
- A U12G ECNL team (TGS team ID `tgs_12345`) ends 2025-26 with Glicko-2 rating 1380 (Silver band), RD 72, match_count 28.
- June 1, 2026: this team becomes U13G ECNL on GotSport (new team ID `gs_api_99999`).
- The ECNL transition dedup pass runs and creates a team_merges record: canonical = `gs_api_99999`, merged = `tgs_12345`.
- The age group transition migration runs and advances the canonical team from U12 → U13 in the rankings table.
- The ranking recompute runs.

**The risk:** If the team_merges record is created with `verified = false`, the Ranking Engine Phase 1 only loads **verified** merges. The new GotSport record starts Phase 3 (Glicko-2 computation) as a brand-new team with no game history. Rating: ~1000. Age group: U13. Previous rating: gone.

**Detection query:**

```sql
-- Find ECNL teams that have both an active GotSport record and a TGS record
-- where the team_merges link is unverified (will not be loaded by ranking engine)
SELECT
  t_gs.team_id  AS gotsport_team_id,
  t_gs.team_name AS gotsport_name,
  t_tgs.team_id  AS tgs_team_id,
  t_tgs.team_name AS tgs_name,
  tm.verified,
  r_gs.rating   AS current_gs_rating,
  r_tgs.rating  AS historical_tgs_rating,
  'UNVERIFIED_CROSSPLATFORM_MERGE' AS risk_flag
FROM team_merges tm
JOIN teams t_gs  ON tm.canonical_team_id = t_gs.team_id
JOIN teams t_tgs ON tm.merged_team_id    = t_tgs.team_id
LEFT JOIN rankings r_gs  ON r_gs.team_id = t_gs.team_id
LEFT JOIN rankings r_tgs ON r_tgs.team_id = t_tgs.team_id
WHERE
  tm.verified = false
  AND (
    t_gs.team_name ILIKE '%ECNL%'
    OR r_gs.age_group IN ('U13','U14','U15','U16','U17')
  )
  AND t_tgs.team_id IN (
    SELECT DISTINCT home_team_id FROM games WHERE source IN ('tgs','tgs_athleteone')
    UNION
    SELECT DISTINCT away_team_id FROM games WHERE source IN ('tgs','tgs_athleteone')
  )
ORDER BY r_tgs.rating DESC NULLS LAST;
```

**Mitigation:** All team_merges records created by the ECNL transition dedup pass (`scripts/ecnl-transition-dedup.js`) must be inserted with `verified = true`. This is a one-line code fix in the transition dedup script — the INSERT statement must set `verified = true, verified_by = 'ECNL_TRANSITION_2026'`. Without this, every ECNL team's rating resets on June 1.

**This is the #1 item on the Minimum Viable Patch List (Section 4).**

---

### Risk C: Calibration Parameter Ordering Bug

**Mechanism:** The calibration parameter updates (U12 RD floor → 60, U13 K=24/RD floor → 75, U14 K=28/RD floor → 65, U19 K=24) are applied by replacing `calibration.json` and re-running `compute-rankings.js`. If the age group transition migration ALSO triggers a full recompute, and the calibration.json replacement happens at a different time than the migration recompute, there is a window where:

- Migration recompute runs with old calibration parameters
- Calibration replacement happens afterward
- Second recompute runs with new parameters

Result: two full recomputes in sequence, the first with wrong calibration. The final state is correct, but during the first recompute, the system briefly has miscalibrated ratings. This is operationally acceptable — it does not corrupt data permanently.

**More serious ordering risk:** If calibration.json is updated BEFORE the migration and the migration includes a full recompute, but the transition migration code assumes specific parameter values (e.g., hardcodes expected RD floor in its carryover logic), a mismatch could occur.

**Mitigation:** Per the Calibration Production Runbook, calibration changes are applied in the May 18–20 window. The age group transition runs June 1. These are 12+ days apart — the ordering bug only manifests if both happen in the same maintenance window. **Apply calibration first (May 18–20), verify stability for 2 weeks, then run migration (June 1).** Never in the same session.

---

## Section 3: Risk Matrix

| Risk | Likelihood | Impact | Detection Method | Mitigation |
|---|---|---|---|---|
| ECNL ID discontinuity → orphaned rating history (Risk A) | Medium | **High** | FORGE Check 1 in validate-ecnl-migration.js; ELO detection query in Section 2A | FORGE runs validate-ecnl-migration --mode=post first; dedup pass runs ONLY after validation clears |
| Simultaneous ID + age group change → double reset (Risk B) | **High** | **High** | Detection query in Section 2B; any team with unverified cross-platform merge | transition-dedup.js must set verified=true; ranking engine loads verified merges before recompute |
| Age group transition order-of-operations bug | Medium | Medium | Review migration script before June 1 | Always run age group advancement BEFORE full recompute (sequence in Section 1 gap fix) |
| Calibration ordering: migration + calibration in same window | Low | Medium | Run migration and calibration change in separate windows | Calibration: May 18–20; Migration: June 1 — never simultaneous |
| Teams at 10-match threshold crossing final week of season | Low | Low | Already handled in spec (Edge Case 2) | No action needed — high RD self-corrects |
| U19 teams advancing to non-existent U20 bracket | Medium | Medium | Check spec Section 1 Edge Case 3 | U19 ratings archived, not advanced — confirm archive logic is implemented in migration script |
| School-year offset calculation wrong for 2026-27 ECNL games | Low | High | `birth_year_cohort` spot-check query after first week of 2026-27 season | Verify `backfill-age-gender.js` changes from `Reports/backfill-age-gender-changes-spec-2026-04-23.md` are deployed before June 1 |

---

## Section 4: Minimum Viable Patch List

Ordered by: if Presten can only do one thing, it's #1.

### Patch 1 (P0 — Must ship before June 1): `ecnl-transition-dedup.js` — Insert `verified = true`

**What breaks without it:** Every ECNL team that migrated from TGS to GotSport in June 2026 resets to ~1000 provisional rating on the next ranking recompute. Potentially 500–2,000 teams lose their rating history. Top-100 U13–U17 ECNL teams disappear from Gold/Silver bands.

**The patch:** In `scripts/ecnl-transition-dedup.js` (to be written by FORGE as per the ECNL migration plan), the INSERT statement for `team_merges` must include:

```javascript
// In ecnl-transition-dedup.js, when inserting cross-platform merge records:
await client.query(`
  INSERT INTO team_merges (canonical_team_id, merged_team_id, merge_method, verified, verified_by, created_at)
  VALUES ($1, $2, 'ecnl_transition', true, 'ECNL_TRANSITION_2026', NOW())
  ON CONFLICT DO NOTHING
`, [gotsportTeamId, tgsTeamId]);
// verified = true is MANDATORY — Ranking Engine Phase 1 only loads verified merges
```

**Owner:** FORGE (write the script); ELO (verify the insert includes verified=true before it runs)  
**Effort:** 5-minute code change — one line in the INSERT statement

---

### Patch 2 (P1 — Should ship before June 1): Migration sequencing validation

**What breaks without it:** If `compute-rankings.js` runs before age group advancement completes, teams appear in wrong age group brackets for one pipeline cycle.

**The patch:** Add a validation check to the migration runbook (Section 5 of the transition spec):

```bash
# Before running compute-rankings.js, confirm age group advancement is complete:
psql -U p -d youth_soccer -c "
  SELECT COUNT(*) as teams_still_on_old_age_group
  FROM rankings
  WHERE season = '2025-2026'
    AND age_group IS NOT NULL;
"
# Expected: 0 (all 2025-26 records have been archived; 2026-27 records with new age groups are in place)
# If > 0: age group advancement did not complete. DO NOT run compute-rankings.js yet.
```

**Owner:** ELO (adds this to migration runbook); Presten (executes)  
**Effort:** 30 minutes to add validation queries to the migration checklist

---

### Patch 3 (P2 — Can defer until post-June 1 if needed): `backfill-age-gender.js` school-year offset

**What breaks without it:** Games in the 2026-27 season from school-year leagues (ECNL, ECNL RL) get incorrect `birth_year_cohort` values. Cross-league analysis and cohort comparisons are wrong. Rankings are not directly affected in the short term (the offset only matters for the `birth_year_cohort` column, not the rating computation).

**The patch:** Deploy the `backfill-age-gender.js` changes specified in `Reports/backfill-age-gender-changes-spec-2026-04-23.md` before any 2026-27 games are ingested. The pipeline should not ingest 2026-27 games until this change is deployed.

**Owner:** FORGE (implements the code change); ELO (spec is complete)  
**Effort:** 3–4 hours implementation per the spec

---

### Patch 4 (P2 — Can defer): U19 archive logic confirmation

**What breaks without it:** U19 teams advance to a "U20" bracket that doesn't exist in the ranking engine, or worse, their records remain in the U19 bracket accumulating games that should belong to a new cohort.

**The patch:** Confirm that the migration script contains the Edge Case 3 logic from the transition spec:

```sql
-- Verify U19 teams are archived, not advanced
-- This query should return 0 after migration completes
SELECT COUNT(*) FROM rankings WHERE age_group = 'U19' AND season = '2026-2027';
-- Should be 0 — U19 teams are archived to season_archive table, not carried to 2026-27
```

**Owner:** Presten (verify during post-migration validation)  
**Effort:** 15-minute query verification

---

## Pre-June-1 Critical Path

Working backwards from June 1 migration date:

| Date | Action | Owner |
|---|---|---|
| April 28 | Presten reviews and approves this risk assessment | Presten |
| May 1 | FORGE begins writing ecnl-transition-dedup.js | FORGE |
| May 10 | ecnl-transition-dedup.js complete with verified=true fix (Patch 1) | FORGE |
| May 14–16 | DSS concludes; rankings frozen for DSS post-processing | All |
| May 18–20 | Calibration changes applied (U12/U13/U14/U19 parameter fixes) | Presten |
| May 25 | FORGE delivers backfill-age-gender.js school-year offset changes | FORGE |
| May 28 | Presten reviews migration runbook with Patch 2 sequencing check | Presten |
| May 31 | Take pre-migration backup: `pg_dump youth_soccer --table=rankings > backup_pre_migration_2026-05-31.dump` | Presten |
| June 1 | Run migration in sequence: (1) calibration confirmed, (2) age group advancement, (3) recompute | Presten |
| June 1–7 | ECNL goes live on GotSport; run validate-ecnl-migration.js --mode=post | FORGE |
| June 7–14 | Run ecnl-transition-dedup.js; re-run recompute | FORGE/Presten |

**If Presten can only do one thing before June 1:** Ensure Patch 1 is in `ecnl-transition-dedup.js` before FORGE runs it. A five-second code review to confirm `verified = true` in the INSERT statement prevents catastrophic rating reset for all ECNL teams.

---

## Related Notes

- [[Age Group Transition Migration Spec 2026-06-01]] — the spec this document audits (Section 1)
- [[Calibration Production Runbook — May 2026]] — calibration application (May 18–20 window)
- [[Ranking Engine]] — Phase 1 loads only verified merges; Phase 3 Glicko computation
- `Reports/ecnl-migration-validation-plan.md` — FORGE's data continuity validation (companion to this risk assessment)
- `Reports/backfill-age-gender-changes-spec-2026-04-23.md` — Patch 3 spec
- [[Dedup Strategy]] — team_merges verified flag behavior
