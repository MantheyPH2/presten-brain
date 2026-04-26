---
title: June 1 Age Transition — Risk Register
type: risk-register
author: ELO
date: 2026-04-25
status: active
due: 2026-05-07
related:
  - "[[June 1 Risk Assessment — Age Group Transition + ECNL Migration]]"
  - "[[June 1 Age-Group Transition — Presten Brief]]"
  - "[[June 2 ECNL Migration Verification Spec]]"
  - "[[Post-Fix Calibration Stability Monitoring Spec]]"
tags: [june-1, age-group-transition, ecnl-migration, risk-register, evo-draw]
---

# June 1 Age Transition — Risk Register

> **Purpose:** Single authoritative risk catalog for the June 1, 2026 age group transition and adjacent June 2 ECNL migration. Synthesized from prior ELO documents. SENTINEL uses this to judge June 1 go/no-go readiness. Presten uses Section 4 for DSS talking points.

---

## Section 1: Transition Summary

**What changes June 1:**

**Age group advancement.** All teams shift from birth-year to school-year age group buckets. U12 → U13, U13 → U14, etc. The `age_group_advance()` function applies the school-year mapping table. Estimated affected teams: all active ranked teams across U12–U18. Rating carryforward rule: teams with match_count ≥ 10 AND last game date within 60 days of June 1 carry their Glicko-2 parameters (rating + RD) into the new age group. All other teams reset to provisional (~1000). Expected carryforward rate: approximately 70–80% of currently active ranked teams. RD expands post-transition using `max(end_RD × 1.15, floor)` per age group to reflect increased uncertainty from the age group change.

**ECNL migration begins June 2 (independent operation).** ECNL teams shift from TGS to GotSport as their primary data source. New GotSport team IDs become canonical; TGS records are marked `is_hidden = true`. The transition dedup script (`ecnl-transition-dedup.js`) creates `team_merges` records linking old TGS IDs to new GotSport IDs. June 1 and June 2 are independent operations — each has its own verification spec and can fail independently.

**What does NOT change June 1:**

- Historical game records remain attributed to their original age group context (e.g., a game played in 2025-26 U13 stays tagged U13, even after the team advances to U14).
- Calibration values remain in effect. The May 18–20 calibration session will have applied U12/U13/U14/U19 parameter fixes 12+ days prior. No calibration changes run on June 1.
- The ranking engine algorithm (9+1 phases, Glicko-2) does not change. Only team-age-group assignments and team ID canonical records change.

---

## Section 2: Risk Register

| # | Risk | Category | Probability | Impact | Mitigation | Residual Risk | Status |
|---|------|----------|-------------|--------|------------|---------------|--------|
| R1 | Teams assigned to wrong age group post-transition due to ambiguous birth year edge cases (e.g., teams with games spanning two age groups in June) | Data quality | Low (< 20%) | Medium | `birth_year_cohort` column is the source of truth; `age_group_advance()` uses exact school-year mapping table. Post-migration completeness check: SELECT COUNT(*) with season='2025-2026' must return 0 before recompute fires. | Low — one-cycle pipeline anomaly if missed; self-corrects next run | Open |
| R2 | Rating discontinuity: teams with ECNL history (TGS source) advance age group AND receive new GotSport ID simultaneously, causing double reset — team resets to ~1000 in new age group despite having a full season of Glicko history | Rating accuracy | High (> 60%) | High | P0 patch: `ecnl-transition-dedup.js` INSERT must set `verified = true, verified_by = 'ECNL_TRANSITION_2026'`. Ranking Engine Phase 1 only loads verified merges. Without this line, every ECNL team's history is invisible to the recompute. FORGE owns implementation; ELO confirms the insert before the script runs (May 10 review). | Medium — ELO's June 3 orphan detection catches any missed teams; recovery requires re-running dedup + recompute | Open |
| R3 | Cross-league win-rate data becomes partially invalid: win rates computed over 2025-26 season reflect old age group buckets; teams now in different buckets make cross-age-group comparisons unreliable for ~4–8 weeks post-transition | Algorithm | High (> 60%) | Medium | Expected and acceptable — win-rate recalibration is scheduled post-transition (June 7, ELO). Cross-league analysis output should not be cited as current until recalibration completes. DSS demo (May 16) is before June 1; this does not affect DSS. | Low — win-rate data is a secondary signal, not the primary Glicko-2 rating input. Recalibration closes the gap by June 14. | Open |
| R4 | ECNL migration (June 2) compounds June 1 transition: if June 1 age group advancement produces unexpected data (wrong age group assignments, orphaned records) and is not caught before June 2 dedup runs, the June 2 migration may create canonical records pointing to the wrong age group records | Data quality | Medium (20–60%) | High | June 1 and June 2 are sequenced 24 hours apart intentionally. ELO runs the 3-check verification by June 3 (after both events). If any June 1 check fails, SENTINEL is notified before FORGE runs the June 2 recompute. June 2 dedup does NOT depend on age group correctness — it links team IDs regardless of age group — so a June 1 age group error and a June 2 ID merge are independent failures. | Low-Medium — the failure mode is containable: an incorrect age group assignment post-June-1 does not corrupt the team_merges record created on June 2. Both can be corrected independently. | Open |
| R5 | High-volume teams with many June 1 games (e.g., teams competing in season-ending tournaments on June 1) are re-rated mid-tournament: ratings computed with new age group context mid-event | Data quality | Low (< 20%) | Low | Already handled in Age Group Transition Migration Spec Edge Case 2: teams active on the migration date retain their match history; high RD self-corrects within 3–5 new games. No special handling required. | Low — cosmetic; any tournament-day rating anomaly resolves within 1 week of new game ingestion | Open |
| R6 | `ecnl_verified` backfill incomplete when June 2 migration runs: the post-April-28 pipeline was expected to begin populating the `ecnl_verified` column; if less than 90% of ECNL teams are backfilled by June 1, the June 2 dedup pass cannot reliably identify which teams need TGS-to-GotSport linking | Dependency | Medium (20–60%) | High | Post-Fix Calibration Stability Monitoring Spec tracks the May 9 readiness check. FORGE confirms `ecnl_verified` backfill progress by May 9 go/no-go. Threshold: ≥ 90% complete before June 2. If < 90%, FORGE extends backfill window before running dedup. | Medium — if threshold is not met and dedup runs anyway, teams without verified ECNL flags may not be linked correctly. Monitoring catches this during ELO's June 3 checks. | Open |
| R7 | Age group transition order-of-operations bug: `compute-rankings.js` fires before age group advancement completes, computing one cycle of rankings under old age group buckets | Data quality | Medium (20–60%) | Medium | P1 patch: sequencing check query must confirm 0 rows remain in old season before recompute fires. Added to the June 1 migration runbook step 6. Presten executes; 5-minute manual gate before running compute-rankings.js. | Low — one-cycle anomaly, not data corruption. Corrects automatically on next pipeline run. | Open |
| R8 | U19 teams advance to a non-existent U20 bracket, or their records remain in U19 accumulating 2026-27 games that should belong to a new cohort | Data quality | Medium (20–60%) | Medium | Age Group Transition Migration Spec Edge Case 3: U19 ratings are archived to season_archive table, not advanced. Verification: SELECT COUNT(*) FROM rankings WHERE age_group = 'U19' AND season = '2026-2027' must return 0. Presten runs this post-migration. | Low — if U19 archive logic fails, the affected teams are a single age group; easy to identify and re-archive | Open |
| R9 | School-year offset calculation wrong for 2026-27 ECNL games: `backfill-age-gender.js` not deployed before first 2026-27 game ingestion, causing incorrect `birth_year_cohort` values for ECNL teams in the new season | Data quality | Low (< 20%) | High | Patch 3: FORGE delivers `backfill-age-gender.js` school-year offset changes by May 25. Pipeline does not ingest 2026-27 games until this change is deployed. Low probability because the first 2026-27 ECNL games are unlikely before late June. | Low — pipeline gate prevents ingestion before deployment. If gate is bypassed, `birth_year_cohort` errors affect cross-league analysis only, not primary Glicko-2 ratings. | Open |
| R10 | Calibration changes and age group transition run in the same maintenance window: two recomputes in one session, the first with wrong parameters | Data quality | Low (< 20%) | Medium | Confirmed separation: calibration applied May 18–20; migration June 1. Never run both in the same session. If the May 18–20 window slips past May 28, do NOT combine it with the June 1 migration — push calibration to June 7 post-transition window instead. | Low — scheduling discipline prevents this; both runbooks explicitly prohibit combining the operations. | Open |
| R11 | ECNL ID discontinuity → orphaned rating history: FORGE's validate-ecnl-migration.js Check 1 fails for > 5% of ECNL snapshot teams (i.e., too many teams have no confirmed GotSport link), causing large numbers of teams to lose rating history | Rating accuracy | Medium (20–60%) | High | FORGE runs validate-ecnl-migration.js --mode=post BEFORE running ecnl-transition-dedup.js. If Check 1 fails (> 5% without GotSport link), the dedup pass does not run until FORGE investigates. ELO's orphan detection query (Section 2A of the Risk Assessment) catches any remaining orphans after June 3. | Medium — the 5% threshold means some teams may still be missed; these are the smallest/least-active ECNL teams and their impact on ranking integrity is Low. Top-100 teams are very likely to have GotSport links confirmed. | Open |

---

## Section 3: Pre-Transition Readiness Checklist

| # | Condition | Owner | Deadline | Status |
|---|-----------|-------|----------|--------|
| 1 | Age group mapping table verified for all affected teams (`birth_year_cohort` backfill complete) | ELO/Presten | May 24 | Pending |
| 2 | Rating carryforward rule documented, tested in staging recompute | ELO | May 24 | Pending — spec complete (see Age Group Transition Migration Spec); needs staging validation |
| 3 | Calibration changes (U12/U13/U14/U19 parameters) applied and stable for ≥ 2 weeks | Presten | June 1 | Pending — window is May 18–20; stability confirmed by May 31 |
| 4 | `ecnl-transition-dedup.js` reviewed by Presten: INSERT includes `verified = true` | Presten | May 10 | Pending — FORGE delivers script by May 10 |
| 5 | `ecnl_verified` backfill ≥ 90% complete (FORGE confirms) | FORGE | June 1 | Pending — May 9 go/no-go check includes this |
| 6 | June 2 ECNL migration verification spec reviewed and acknowledged by SENTINEL | SENTINEL | May 30 | Pending |
| 7 | `backfill-age-gender.js` school-year offset changes deployed (Patch 3) | FORGE | May 25 | Pending |
| 8 | Pre-migration DB backup: `pg_dump youth_soccer --table=rankings > backup_pre_migration_2026-05-31.dump` | Presten | May 31 | Pending |
| 9 | Post-transition cross-league win-rate recalibration scheduled | ELO | June 7 | Pending — ELO schedules after migration completes |
| 10 | ELO June 3 verification checks defined and ready to run (3-check protocol from Presten Brief Section 3) | ELO | June 1 | Complete — checks defined in `Rankings/June 1 Age-Group Transition — Presten Brief.md` Section 3 |

---

## Section 4: DSS Implications

**Safe to say at DSS (May 16):**

- "When age groups advance each year, we carry team ratings forward using a rule-based approach: teams that have played enough recent games keep their full rating history when they move up. Teams that have been inactive reset to a provisional rating and re-earn their ranking in the new group."
- "Historical game records stay attributed to the age group they were played in — a U13 game from 2025-26 always counts as U13 context, regardless of where the team competes in 2026-27."
- "Age group transitions use a school-year calendar starting June 1, 2026 — the same calendar that most leagues use for roster eligibility."
- "Rating uncertainty increases slightly at the start of each new season as teams move up, then converges quickly as new games are played. This is intentional — it makes the system honest about age-group transitions rather than assuming last year's U13 rank predicts this year's U14 rank perfectly."

**Do not commit to at DSS:**

- Exact handling of teams that have games in both the old and new age group during the transition week (edge case in tournament play; not yet operationally verified).
- Post-transition ranking distribution for any specific age group (depends on actual carryforward rate, which varies by team activity level and is not finalized until recompute runs).
- Any claim about ECNL team continuity across the June 2 migration (ECNL migration is post-DSS; the dedup script is not yet validated).

**If asked about ECNL migration specifically:** "ECNL teams are migrating to a new platform this summer, and we have a transition plan to preserve their full rating history — the process runs after DSS, so the rankings you see today reflect their complete history under the prior platform."

---

## Section 5: SENTINEL Monitoring Triggers

**Triggers that delay June 1 transition:**

- Any Risk in the register marked High probability + High impact (R2, R11) without a confirmed mitigation as of the May 31 SENTINEL review.
- `ecnl_verified` backfill < 90% complete as of June 1 (FORGE confirms; if not met, push June 2 ECNL dedup to June 7).
- June 2 ECNL migration verification spec not reviewed by SENTINEL by May 30.
- May 18–20 calibration session not completed and stable — if calibration changes have not been applied by May 28, push them to the June 7 post-transition window. Do NOT combine with June 1 migration.

**Triggers that allow June 1 to proceed with active monitoring:**

- All Medium-probability risks have confirmed mitigations (all Open risks above have mitigations documented — confirming execution is the remaining step).
- No High-impact risk is Open without a named mitigation owner and deadline as of May 31.
- Pre-migration backup confirmed (Checklist item 8).
- `ecnl-transition-dedup.js` reviewed (Checklist item 4), even if backfill is slightly below 90% threshold (SENTINEL judgment call at the 85–90% range).

**Post-June-1 monitoring (ELO files by June 3):**

ELO files a transition verification note in the June 3 briefing covering the 3 checks from `Rankings/June 1 Age-Group Transition — Presten Brief.md` Section 3:
1. Age group completeness (< 100 teams in multiple age groups simultaneously)
2. ECNL rating continuity (no previously-rated ECNL team showing < 1100 post-migration)
3. Rating orphan detection (zero rows from orphan detection query)

Any failed check is escalated to SENTINEL immediately. SENTINEL does not wait for the next scheduled briefing.

---

## References

- `Rankings/June 1 Risk Assessment — Age Group Transition + ECNL Migration.md` — source document; detailed technical risk analysis and SQL queries
- `Rankings/June 1 Age-Group Transition — Presten Brief.md` — Presten-facing context and June 1 migration sequence
- `Rankings/June 2 ECNL Migration Verification Spec.md` — June 2 adjacent event verification spec
- `Rankings/Post-Fix Calibration Stability Monitoring Spec.md` — ecnl_verified dependency tracking
- `Rankings/Age Group Transition Migration Spec 2026-06-01.md` — full technical spec for age group advancement logic
- `Rankings/ECNL Rating Continuity Spec — June 2026.md` — rating storage model and orphan detection
- `Rankings/Calibration Production Runbook — May 2026.md` — calibration timing (May 18–20 window)
