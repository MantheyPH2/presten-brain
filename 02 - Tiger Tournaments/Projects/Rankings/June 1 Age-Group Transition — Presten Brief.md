---
title: June 1 Age-Group Transition — Presten Brief
type: presten-brief
author: ELO
date: 2026-04-24
status: ready
related: "[[June 1 Risk Assessment — Age Group Transition + ECNL Migration]]", "[[June 2 ECNL Migration Verification Spec]]"
tags: [june-1, age-group-transition, ecnl-migration, presten-brief, evo-draw]
---

# June 1 Age-Group Transition — Presten Brief

> **Read time:** Under 5 minutes.
> **Source:** All content pulled from `Rankings/June 1 Risk Assessment — Age Group Transition + ECNL Migration.md`. Do not re-derive.

---

## Section 1: What Happens Automatically on June 1 (No Action Required)

**Age group advancement:** Teams advance from U12 → U13, U13 → U14, etc. The migration script applies the birth_year_cohort mapping and marks each team's new age group. Expected effect on rankings: all active teams appear in their new age group on the next recompute. Rank order within each new age group may shift as cohorts mix.

**Rating carryover (for qualifying teams):** Teams with match_count ≥ 10 AND last game within 60 days carry their Glicko-2 parameters (rating, RD) into the new age group. Expected effect: roughly 70-80% of active ranked teams carry their ratings. Stale teams reset to provisional (~1000).

**RD expansion:** For all teams that carry their ratings, the RD is expanded post-transition using `max(end_RD × 1.15, floor)` per age group. This correctly reflects increased uncertainty from the age group change. Expected effect: all team ratings become slightly more uncertain for the first few games of the new season, then converge as new games accumulate.

**Risk of silent failure:** The age group advancement must complete BEFORE the ranking recompute runs. If the pipeline fires before the migration finishes, teams compute under their old age group for one cycle, then correct. This looks like a rank spike to users. Not data corruption — just a one-cycle anomaly. The sequencing check in Patch 2 below prevents this.

---

## Section 2: What Requires Presten Action

| Item | Presten Action | Time Estimate | Deadline |
|------|---------------|---------------|----------|
| Review ecnl-transition-dedup.js | Confirm `verified = true` is in the INSERT before FORGE runs it | 5 minutes | May 10 |
| Pre-migration backup | `pg_dump youth_soccer --table=rankings > backup_pre_migration_2026-05-31.dump` | 15 minutes | May 31 |
| Run migration sequence | Follow the six-step sequence in Section 3 below | 1–2 hours | June 1 |
| Run post-migration sequencing check | Confirm age group advancement completed before `compute-rankings.js` fires | 5 minutes | June 1 |

**The single most critical action:** Before FORGE runs `ecnl-transition-dedup.js`, confirm the INSERT statement contains `verified = true`. A 5-second code review prevents catastrophic rating reset for every ECNL team (see Risk Register below).

---

## Section 3: ELO Monitoring Plan

After June 1, ELO runs the following checks within 48 hours (by June 3):

**Check: Age group completeness** — Confirms no team appears in both their old and new age group. Failure: >100 teams appear in multiple age groups simultaneously. Action: ELO flags to SENTINEL with count of duplicates; FORGE investigates migration script.

**Check: ECNL rating continuity** — Confirms no previously-rated ECNL team shows a rating near the initialization floor (~1000) after migration. Failure: any ECNL team previously rated >1200 now shows <1100. Action: ELO escalates to SENTINEL as P0; FORGE investigates whether verified=true was set correctly.

**Check: Rating orphan detection** — Runs the Check 6 query from `Rankings/June 2 ECNL Migration Verification Spec.md` (teams with ecnl_transition merges whose post-migration rating is near initialization). Failure: any rows returned. Action: pull team name, verify merge was processed correctly.

**ELO will report June 1 transition status by June 3.**

---

## Section 4: Risk Register

**Risk B — Simultaneous ID change + age group transition → double reset (HIGH probability / HIGH impact).**
Mitigation already in place: FORGE must set `verified = true` in `ecnl-transition-dedup.js`. This is Patch 1 (P0). Residual risk: if FORGE ships the script without this line, ELO's June 3 monitoring catches the failure — but recovery requires re-running the dedup pass and recomputing.

**Risk A — ECNL ID discontinuity → orphaned rating history (MEDIUM probability / HIGH impact).**
Mitigation: FORGE's `validate-ecnl-migration.js` Check 1 must clear before the dedup pass runs (less than 5% of ECNL teams without confirmed GotSport links). Residual risk: teams outside the 5% threshold that still have no link will lose rating history. ELO's orphan detection check catches these.

**Risk: Calibration + migration in the same window (LOW probability / MEDIUM impact).**
Mitigation confirmed: calibration changes are applied May 18–20; migration runs June 1. These are 12+ days apart. Never run both in the same session. If the May 18–20 window slips, do not combine with the June 1 migration.

---

## Section 5: Calendar Context

| Date | Event | Who Acts |
|------|-------|----------|
| April 28 | GA ASPIRE fix + ECNL dedup | Presten |
| May 1 | FORGE begins ecnl-transition-dedup.js | FORGE |
| May 10 | Presten reviews ecnl-transition-dedup.js (5-min check: verified=true?) | Presten |
| May 14–15 | DSS validation session | Presten |
| May 16 | DSS | Presten |
| May 17–30 | Boys Brier analysis window | ELO |
| May 18–20 | Calibration changes applied (U12/U13/U14/U19) | Presten |
| May 25 | FORGE delivers backfill-age-gender.js school-year offset | FORGE |
| May 31 | Pre-migration backup snapshot | Presten |
| June 1 | Age-group transition → migration sequence | Presten |
| June 2 | ECNL migration goes live on GotSport | FORGE/Presten |
| June 3 | ELO transition verification (3 checks) | ELO |
| June 7–14 | FORGE runs ecnl-transition-dedup.js + recompute | FORGE/Presten |

**June 1 and June 2 are independent operations.** The age group transition (June 1) changes which age group bucket teams sit in. The ECNL platform migration (June 2) changes which team IDs are canonical. These can fail independently. ELO monitors both. If the June 1 transition produces unexpected data, it does NOT block the June 2 ECNL migration — each has its own verification spec.

### June 1 Migration Sequence (for Presten Reference)

```
1. Stop nightly pipeline (or run in maintenance window)
2. Confirm calibration.json is already updated (May 18–20 window — done by now)
3. Run birth_year_cohort backfill
4. Run age group advancement for all carry-forward teams
5. Apply season-start RD expansion
6. Run sequencing check: SELECT COUNT(*) FROM rankings WHERE season = '2025-2026' AND age_group IS NOT NULL;
   → Must return 0 before continuing
7. Run full ranking recomputation: node scripts/compute-rankings.js
8. Re-enable nightly pipeline
```

---

## References

- `Rankings/June 1 Risk Assessment — Age Group Transition + ECNL Migration.md` — full technical risk analysis (source of all content above)
- `Rankings/June 2 ECNL Migration Verification Spec.md` — adjacent June 2 event (independent; ELO monitors both)
- `Rankings/ECNL Rating Continuity Spec — June 2026.md` — rating storage model and Check 6 orphan detection
- `Rankings/Calibration Production Runbook — May 2026.md` — calibration change timing (May 18–20)
