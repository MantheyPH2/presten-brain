---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-25
status: completed
priority: high
due: 2026-04-28
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/April 29 Pipeline Validation Spec.md"
---

# Task: April 29 Pipeline Validation Spec

## Context

The April 28 session applies two schema changes to the `games` table: GA ASPIRE event_tier fix and the new `ecnl_verified` column with backfill. After the rankings recompute confirms clean, these changes exist in the DB. What does not exist: a defined validation that confirms the **pipeline code** will correctly interact with these schema changes on May 1 and beyond.

The May 1 Day-Of Runbook and Morning-After Runbook cover deployment health (new game count, source breakdown). Neither checks that pipeline ingestion logic is writing the new schema fields correctly — specifically:
- Is the pipeline's GotSport scraper writing `event_tier = 'ga_aspire'` for GA ASPIRE games going forward, or will it revert to `'ga'`?
- Is the pipeline correctly setting `ecnl_verified = TRUE` for games ingested after the schema change?

These are implementation-level questions FORGE can answer by inspecting the pipeline code, not by running the pipeline. The spec is written before May 1; results confirm whether code changes are needed before deployment.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/April 29 Pipeline Validation Spec.md`

---

### Section 1: GA ASPIRE Event Tier — Pipeline Write Behavior

After the April 28 UPDATE, existing games have correct `event_tier` values. The question is forward-looking: when the pipeline ingests a new game from a GA ASPIRE tournament, what will it write for `event_tier`?

FORGE inspects the event classification logic in the pipeline code and answers:

| Question | Answer | Source (file:line) |
|----------|--------|-------------------|
| How does the pipeline determine `event_tier` for a new game? | | |
| Where is the `ga_aspire` string defined (or absent) in the classifier? | | |
| Will new GA ASPIRE games ingested after April 28 get `event_tier = 'ga_aspire'` or `'ga'`? | | |
| If the pipeline overwrites `event_tier` on re-ingestion, does the April 28 UPDATE survive a re-scrape of existing GA ASPIRE games? | | |

**Pass condition:** Pipeline correctly writes `ga_aspire` for new games AND does not overwrite corrected existing game rows.

**Fail condition:** Pipeline would re-classify GA ASPIRE games as `ga` on re-scrape — the April 28 fix reverts the moment the pipeline runs. If FAIL, FORGE identifies the code change required and estimates complexity.

---

### Section 2: ecnl_verified — Pipeline Write Behavior

After the April 28 ALTER TABLE + backfill, historical ECNL games are `ecnl_verified = TRUE`. For new games:

| Question | Answer | Source (file:line) |
|----------|--------|-------------------|
| Does the pipeline currently write `ecnl_verified` for new game inserts? | | |
| If not: does it default to FALSE (correct) or NULL (problem)? | | |
| Is there a code change required to set `ecnl_verified = TRUE` on ingestion for ECNL-source games? | | |
| The ECNL verified population plan identified a required tgs_scraper change — has this change been made? | | |

The population plan (filed 2026-04-25) specified that the tgs_scraper must be updated before May 1 to write `ecnl_verified = TRUE` on new ingestion. This spec confirms whether that change is in place or still pending.

**Pass condition:** Pipeline writes `ecnl_verified = TRUE` for new ECNL games (source IN ('tgs', 'tgs_athleteone') AND event_tier = 'ecnl').

**Fail condition:** Pipeline leaves `ecnl_verified` NULL or FALSE for new ECNL games. If FAIL, FORGE provides the code change required and flags as a May 1 blocker.

---

### Section 3: May 1 Deployment Go / No-Go Recommendation

Based on findings from Sections 1 and 2, FORGE provides a single table:

| Check | Status | Impact if Not Fixed Before May 1 |
|-------|--------|----------------------------------|
| GA ASPIRE forward-write correct | PASS / FAIL | April 28 fix reverts on first pipeline run |
| GA ASPIRE re-scrape safe | PASS / FAIL | Re-ingestion overwrites corrected rows |
| ecnl_verified write on new games | PASS / FAIL | New ECNL games accumulate as unverified |
| tgs_scraper code change deployed | PASS / FAIL | ECNL dedup feature non-functional post-May-1 |

If all PASS: note "Pipeline validated for May 1."

If any FAIL: note "Code change required before May 1 deployment — see Section [N]." FORGE does not implement the fix without SENTINEL authorization; FORGE identifies and documents it.

---

## Quality Standard

- Every answer in Sections 1–2 must cite a specific file and line number from the pipeline codebase.
- The recommendation in Section 3 must be deterministic — not "probably passes" but "PASS" or "FAIL" based on code inspection.
- If FORGE cannot access the pipeline code to verify, state this explicitly as a blocker, not a guess.
- Document must be completable before April 28, so results are known before the April 28 session executes.

## Why This Matters

April 28 fixes the DB. If the pipeline code is not aligned, the first run of the daily pipeline on May 1 undoes the fix. This is a class of bug that would not be visible until May 2 when ELO's monitoring queries show GA ASPIRE ratings reverting. By then, the fix and the reversion have both happened and the calibration baseline is unclear. Catching this now — before April 28 — costs FORGE a code inspection. Catching it on May 2 costs a DB session, a recompute, and a SENTINEL escalation.

## References

- `Infrastructure/ECNL ecnl_verified Column — Population Plan.md` — tgs_scraper change requirement
- `Infrastructure/April 28 Execution Log.md` — DB changes this spec validates against
- `Infrastructure/May 1 Deployment — Day-Of Runbook.md` — pipeline launch this spec prepares
- `Rankings/Post-Fix Calibration Stability Monitoring Spec.md` — monitoring that will catch forward regressions if this spec is not run
