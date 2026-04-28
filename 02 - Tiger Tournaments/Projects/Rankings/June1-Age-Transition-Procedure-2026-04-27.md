---
title: June 1 Age Transition — Vault Research and Procedure Documentation
type: procedure-research
author: ELO
date: 2026-04-27
status: complete
related: "[[Age Group Transition Migration Spec 2026-06-01]]", "[[June 1 Risk Assessment — Age Group Transition + ECNL Migration]]"
tags: [june-1, age-transition, procedure, migration, evo-draw]
---

# June 1 Age Transition — Vault Research and Procedure Documentation

> **Context:** ELO's 13:18 briefing (2026-04-27) posted this open question for Presten: "Is the June 1 age transition automatic (system-driven) or does it require a manual step?" This document resolves that question from vault sources before escalating.

---

## Section 1: What the Vault Says

### Source 1: `Rankings/Age Group Transition Migration Spec 2026-06-01.md`

**Strongest source — explicit and detailed.**

The migration spec is titled "SPECIFICATION ONLY — Do Not Implement Without Approval." Every executable action is written as a Bash or SQL command requiring a human operator. The spec includes:

- Pre-migration backup requirement: `pg_restore -d youth_soccer --table=rankings backup_pre_migration_2026-05-31.dump`
- Rollback mechanism: `git checkout v2025-26-final -- config/calibration.json` and `node compute-rankings.js`
- Schema changes: `ALTER TABLE rankings ADD COLUMN...` — requires Presten to run in psql
- Rating carryover computation: SQL UPDATE statements and `compute-rankings.js` execution

The spec states explicitly: **"Pre-migration backup is mandatory. The migration should not run unless a verified backup of the rankings table (and calibration.json) exists and has been tested for restore."**

The spec also provides an estimated run time: **"Allow a 2-hour window for the migration to complete, plus a 30-minute post-run validation window."**

**No system-driven automatic trigger is mentioned anywhere in the spec.** Every step is an explicit operator action.

### Source 2: `Rankings/June 1 Risk Assessment — Age Group Transition + ECNL Migration.md`

Title confirms the migration is a discrete event requiring coordination. The existence of a "risk assessment" document implies a planned human-executed migration, not a background automated process. Automated transitions do not require risk assessments for operator review.

### Source 3: `Rankings/Age Group Transition Migration Spec 2026-06-01.md` — Pre-Migration Checklist

The spec includes a 7-item Pre-Migration Checklist that must be confirmed before the migration runs:

- [ ] Calibration fixes applied and validated
- [ ] MLS NEXT tier split decision made
- [ ] `birth_year_cohort` schema migration run on production and backfill complete
- [ ] Pre-migration backup verified
- [ ] Rollback script tested
- [ ] `resolveTeamTier()` update plan confirmed
- [ ] **DSS 2026 concluded and division placements frozen** — explicit gate: "Do not run migration while DSS brackets are live"

This checklist requires a human operator to confirm each item. Automated systems do not have checklists.

### Source 4: `Rankings/Age Group Transition Migration Spec 2026-06-01.md` — Estimated Pipeline Run Time

| Step | Estimated Time |
|------|----------------|
| Phase 1–10 (compute-rankings.js full recompute) | ~42–60 minutes |
| `birth_year_cohort` backfill (games table) | ~15–20 minutes |
| Rankings table write + indexing | ~5 minutes |
| **Total** | **~65–90 minutes** |

This is the runtime of `node compute-rankings.js` — a script Presten runs manually. No automated scheduler is referenced.

### Source 5: Other vault sources searched

- **CLAUDE.md:** Not found in this vault location. Not a blocking gap — the migration spec is explicit.
- **Scope Rules / League Hierarchy:** No mention of automatic age transition. League Hierarchy covers calibration values, not pipeline scheduling.
- **Prior ELO briefings mentioning "June 1":** Briefings reference June 1 as a deadline and describe it as a session Presten runs, not an automated event. Example: `Rankings/June 1 Age-Group Transition — Presten Brief.md` exists — its title explicitly names Presten as the operator.

---

## Section 2: Procedure as Documented

**The June 1 age transition is MANUAL. Presten must run it.**

**Confidence level: HIGH** — explicitly documented in the migration spec with step-by-step Bash and SQL commands, pre-migration checklist, and estimated run time.

**What the procedure entails:**

1. **Pre-migration prep** (Presten, ahead of June 1):
   - Confirm all Pre-Migration Checklist items are met
   - Create and verify a backup of the `rankings` table and `calibration.json`
   - Test rollback script on staging

2. **Schema migrations** (Presten in psql):
   - `ALTER TABLE rankings ADD COLUMN birth_year_cohort INTEGER NULL;`
   - `ALTER TABLE rankings ADD COLUMN season VARCHAR(10) NULL;`
   - `ALTER TABLE games ADD COLUMN home_birth_year_cohort INTEGER NULL;`
   - `ALTER TABLE games ADD COLUMN away_birth_year_cohort INTEGER NULL;`

3. **Backfill** (Presten in psql):
   - `UPDATE rankings SET birth_year_cohort = ...` — backfill historical records
   - `UPDATE games SET birth_year_cohort = ...` — same

4. **Rating carryover and age group advancement** (Presten runs `compute-rankings.js`):
   - The migration logic determines carryover vs reset per team (match_count ≥ 10, recent game check)
   - Age groups advance one bracket per the mapping table
   - Season-start RD expansion applied per age group table in spec
   - Total runtime: **65–90 minutes**

5. **Post-migration validation** (Presten + ELO):
   - Rating distribution shape check
   - Top team rank stability spot check
   - `birth_year_cohort` backfill coverage check
   - Allow 30-minute validation window

**No system automatically triggers this.** The June 1 date is a target, not a system event. If Presten runs the migration on May 31 or June 2, the system does not self-correct. The transition must be explicitly scheduled and executed.

---

## Section 3: Risk to Post-DSS Sprint

**Post-DSS calibration plan timeline:**

| Window | Activity |
|--------|----------|
| May 17–30 | Boys Brier analysis (Boys GA ASPIRE: retain cal=100 or revise to 70) |
| May 18–20 | MLS NEXT Boys tier split deployment (Homegrown 160 / Academy 135) |
| ~May 27 | Post-calibration stability clearance (7-day window after May 18–20) |
| May 28–June 1 | Final calibration changes applied, if Boys Brier recommends revision |
| **June 1** | **Age group transition migration** |

### Does the June 1 transition interfere with post-DSS calibration?

**Yes — one potential conflict:**

If Boys Brier analysis (May 17–30) recommends revising Boys GA ASPIRE from cal=100 to cal=70, that revision must be applied AND the system must run `compute-rankings.js` to propagate it before the June 1 migration runs. If the revision is applied May 28 and Presten needs 7 days of stability monitoring before the June 1 migration, there is a direct timeline conflict: the stability window ends June 4, but the migration should run June 1.

**Resolution options:**
1. If Boys Brier completes by May 25 and revision applies May 26, the migration can proceed June 1 with only 5 days post-revision stability — shorter than ideal but workable if the revision is small (a single calibration constant change, not a K-factor redesign).
2. If Boys Brier is data-limited (< 300 games) or delta < 0.003 (retain cal=100), no revision is needed and June 1 proceeds cleanly.
3. If Boys Brier FAILS (delta > 0.008) and requires a significant revision: defer the June 1 migration to June 7–10. The migration spec says "expected rollback time ~2–3 hours" and "June 1 transition is not a hard public-facing deadline — it affects internal data quality, not a live customer event." A one-week slip is acceptable.

**Concrete ELO action:** After Boys Brier completes (target May 25), ELO assesses whether the June 1 migration date is safe or needs to slip. File assessment in the May 25 briefing.

**Other interference checks:**
- MLS NEXT Boys deployment (May 18–20) deploys BEFORE June 1 — correct sequencing per the migration spec Pre-Migration Checklist ("MLS NEXT tier split decision made" is a checklist gate).
- DSS division placements frozen — the checklist explicitly requires DSS to be concluded. DSS is May 14–15. June 1 is ~2.5 weeks post-DSS. No conflict.
- ECNL migration (June 2) — the June 1 age transition must complete before the ECNL platform migration (TGS → GotSport) on June 2. The June 1 migration includes ECNL teams (they shift to school-year cutoff). ECNL must be in their new age groups before the June 2 data migration begins.

### When must Presten schedule the June 1 migration session?

**Presten must schedule a June 1 session by May 28 at the latest.**

Planning buffer: May 29–31 for pre-migration checklist confirmation and backup verification. June 1 execution window: allow a 3-hour block (65–90 min migration + 30 min validation + 30 min buffer).

If Boys Brier is running May 17–30 and finds a required revision, Presten needs to plan two consecutive sessions in late May:
- Session 1 (May 28): Apply Boys GA ASPIRE revision, run `compute-rankings.js`
- Session 2 (June 1): Migration — schema changes, backfill, compute-rankings.js recompute

**ELO flag:** If Boys Brier finds a required revision and Presten cannot schedule May 28 + June 1 sessions, escalate to SENTINEL immediately (by May 25) for schedule coordination.

---

## Section 4: Question for Presten

**RESOLVED — no Presten question needed.**

The vault is unambiguous: the June 1 transition is a manual Presten execution session. The procedure is fully documented in the migration spec.

ELO can now update the post-DSS calibration plan with the confirmed June 1 session estimate: **~3 hours Presten time on June 1** (migration + validation). This should be blocked on Presten's calendar in advance.

**One informational note for Presten** (not a question): The post-DSS sprint assumes Boys Brier completes before May 28 to avoid a June 1 migration conflict. If Boys Brier runs long (data-limited, requiring investigation), ELO will flag a potential June 1 slip in the May 25 briefing.

---

## References

- `Rankings/Age Group Transition Migration Spec 2026-06-01.md` — primary source; full procedure
- `Rankings/June 1 Risk Assessment — Age Group Transition + ECNL Migration.md` — risk context
- `Rankings/June 1 Age-Group Transition — Presten Brief.md` — Presten-facing summary
- `Rankings/Boys Brier Analysis Scope Spec — May 2026.md` — upstream dependency (Boys GA ASPIRE verdict)
- `Rankings/Post-DSS-Calibration-Priority-Queue-2026-04-27.md` — sprint plan this document validates against

*ELO — 2026-04-27*
