---
title: ECNL Migration — Option Comparison Matrix
type: decision-support
author: ELO
date: 2026-04-26
status: active
related: "[[ECNL Migration — Option Selection Decision Gate]]", "[[ECNL Migration Rating Continuity Spec — June 2026]]"
tags: [ecnl, migration, decision-matrix, rating-continuity, evo-draw]
---

# ECNL Migration — Option Comparison Matrix

> **Purpose:** Pre-computed synthesis. For each (Option × Checkpoint) cell, this matrix states what result SUPPORTS or DISQUALIFIES that option — written before any checkpoint data exists. When CP results arrive, ELO fills Section 4 (Running Verdict Tracker) mechanically. No synthesis under time pressure.
>
> **Written:** 2026-04-26. **CP1 runs:** 2026-04-29 (first item in April 29 Master Script). **Option selection due:** 2026-05-10.

---

## Section 1: Option Definitions

| Option | Name | One-Line Description | Key Assumption |
|--------|------|---------------------|----------------|
| 1 | No Re-tag | Keep historical `ecnl` games tagged as `ecnl`; new post-June-1 games tagged `ecnl_rl`; no recompute | Historical ECNL cal (120 Boys / 130 Girls) legitimately reflects higher competition; recency weighting compresses Path A vs. Path B gap over 6–12 months |
| 2 | Re-tag to ecnl_rl | On June 1, UPDATE all historical `ecnl` games for Path A teams to `event_tier = 'ecnl_rl'`; run full recompute | The cal delta (65–75 pts) creates an unfair rating advantage for Path A teams; a one-time 40–80 pt rating cliff is preferable to a 12-month rating seam |
| 3 | ecnl_legacy tier | Introduce `event_tier = 'ecnl_legacy'` for pre-migration games with a blended calibration value (approximate midpoint of ecnl and ecnl_rl cal values) | A purpose-built transition tier produces smoother continuity than either preserving ecnl cal or hard-cutting to ecnl_rl cal |

**ELO current lean:** Option 1. Option 2 creates a 40–80 pt rating cliff. Option 3 requires DDL changes and a blended cal value ELO has no cross-league data to support. Option 1 requires no schema changes and the rating seam is directionally correct (Path A teams genuinely competed at higher calibration levels).

---

## Section 2: Comparison Matrix — Expected Results by Checkpoint

| Checkpoint | What It Tests | Option 1 Expected | Option 2 Expected | Option 3 Expected |
|-----------|--------------|------------------|------------------|------------------|
| **CP1** ECNL RL staleness check (April 29) | Whether ECNL RL data is live and reliable enough to receive Path A teams | **REQUIRES:** ECNL RL ≤ 30 days stale. If ECNL RL is a dead source, Path A teams moving to it have no valid new games, and Option 1's natural convergence never happens. **Disqualifying if:** > 30 days stale. | PREFERS fresh ECNL RL data (Option 2 also wants new games to be reliable), but ECNL RL freshness is less critical — Option 2 resolves everything via re-tag, not forward accumulation. A stale ECNL RL does not disqualify Option 2. | NEUTRAL. Option 3 handles historical games only; ECNL RL freshness doesn't affect the legacy-tier computation. However, if ECNL RL is dead, the `ecnl_legacy` tier still needs a working forward ingestion path eventually. |
| **CP2** ecnl_verified column populated (April 29) | Whether Path A teams are identifiable in the DB via the verified flag | **REQUIRES:** `ecnl_verified` column exists and backfill is complete (per April 28 Execution Log Step 2b). Option 1 depends on knowing which teams are Path A so forward games are tagged correctly. Without ecnl_verified, new Path A games may receive wrong event_tier assignments. **Disqualifying if:** Column missing or backfill confirmed < 70% complete. | REQUIRES: Same dependency. Option 2's mass UPDATE query uses the `team_merges.method = 'ecnl_transition'` join to identify Path A teams — this requires the ecnl_transition method to be populated in team_merges, which depends on the ecnl_verified backfill. **Disqualifying if:** ecnl_transition records absent from team_merges. | REQUIRES: Same as Option 2 — need to identify Path A teams to assign `ecnl_legacy` tier to their historical games. Disqualifying if Path A team list cannot be constructed. |
| **CP3** First pipeline run ingests ECNL games (May 1) | Whether the daily pipeline correctly tags new post-May-1 ECNL RL game records | **REQUIRES PASS:** New games arriving post-May-1 must appear in `games` with `event_tier = 'ecnl_rl'`. Option 1's entire convergence mechanism depends on the forward pipeline tagging Path A games as `ecnl_rl` correctly from the start. **Disqualifying if:** New ECNL games arrive tagged `ecnl` instead of `ecnl_rl`, or arrive with NULL event_tier. | PREFERS PASS: Option 2 also needs forward ingestion to work, but is less sensitive — Option 2 re-tags historical games and then relies on forward correctness. A bad CP3 delays confidence, but doesn't disqualify Option 2 in isolation. | NEUTRAL TO FORWARD INGESTION: Option 3 handles historical games via `ecnl_legacy`. Forward ingestion tags are needed but don't affect the legacy tier computation. |
| **CP4** ECNL game volume after 3 pipeline runs (May 3) | Whether new ECNL RL game ingestion volume is on track | **PREFERS:** Consistent per-run game ingestion (i.e., the ECNL RL source is actively producing new games). Low volume doesn't disqualify Option 1 — it just means convergence takes longer. | PREFERS: Same. High Option 2 rating cliff + low forward game volume = Path A teams stay depressed longer. Not disqualifying but informs risk framing. | NEUTRAL: Option 3 doesn't depend on forward volume for its transition mechanism. |
| **CP5** ecnl_verified flag accuracy spot check (May 5) | Whether the ecnl_verified flag correctly identifies ECNL-sourced games in forward pipeline data | **REQUIRES:** Accuracy ≥ 90% (18/20 manually verified ECNL games have `ecnl_verified = TRUE`). Option 1's Path A identification relies on this flag for ongoing forward classification. Systematic flag errors degrade Option 1's ability to distinguish Path A vs. Path B games. **Disqualifying if:** < 80% accuracy AND FORGE cannot confirm a fix timeline before May 10. | REQUIRES: Same accuracy threshold. Option 2 uses the same flag mechanism for Path A identification in the re-tag UPDATE. **Disqualifying if:** < 80% accuracy AND fix not scheduled before June 1 re-tag execution. | REQUIRES: Same accuracy threshold for Path A identification in the `ecnl_legacy` assignment UPDATE. Same disqualifying condition as Option 2. |
| **CP6** Rating stability for ECNL teams in first-week pipeline (May 7) | Whether ECNL team ratings are behaving predictably after the April 28 fix and new game ingestion begins | **REQUIRES:** ECNL team rating day-over-day change is < 15 points average for 3+ consecutive pipeline runs. Option 1 assumes a stable baseline to observe the natural Path A/B convergence. If ratings are still oscillating post-April-28, the Option 1 seam is noise, not signal. | PREFERS: Stable baseline for the same reason. Option 2 introduces a deliberate cliff — running that cliff on top of unstable ratings risks compounding two disruptions. Not disqualifying, but increases Option 2 risk profile. | NEUTRAL TO SHORT-TERM STABILITY: Option 3's `ecnl_legacy` tier is a one-time DDL + UPDATE — it's applied once, not progressively. Short-term volatility doesn't affect the tier assignment logic. |
| **CP7** Final option selection (May 10) | Evidence complete; ELO selects option | **SELECTED** if: CP1 PASS, CP2 PASS, CP5 ≥ 80%, no active disqualifying conditions. This is the expected path. | **SELECTED** if: Option 1 is disqualified by CP1 or CP2, AND Presten has signed off on the rating cliff, AND ecnl_transition records in team_merges are complete. | **SELECTED** if: Both Option 1 and Option 2 are disqualified, AND DDL changes are authorized by SENTINEL, AND a defensible `ecnl_legacy` cal value can be derived (or SENTINEL accepts a declared estimate). This is the fallback of last resort. |

---

## Section 3: Disqualification Rules

**Option 1 is disqualified if any of the following:**
- CP1: ECNL RL staleness > 30 days (forward games won't arrive reliably; Option 1 convergence mechanism fails)
- CP2: `ecnl_verified` backfill < 70% complete OR `ecnl_transition` records absent from team_merges (Path A teams cannot be identified)
- CP5: ecnl_verified flag accuracy < 80% AND FORGE has no fix timeline before May 10

**Option 2 is disqualified if any of the following:**
- CP2: `ecnl_transition` records absent from team_merges (UPDATE query cannot identify Path A teams)
- CP5: ecnl_verified flag accuracy < 80% AND fix not scheduled before June 1 (Path A team identification is unreliable)
- Path A team count > 1,000 AND Presten has NOT approved the rating cliff in writing before May 10 (per Continuity Spec §5 authorization requirement)
- April 29 G2 FAIL (calibration unstable — do not run re-tag on unstable ratings)

**Option 3 is disqualified if any of the following:**
- No cross-league calibration data available to derive a defensible `ecnl_legacy` cal value by May 5
- DDL change not authorized by SENTINEL before May 10
- Path A team list cannot be constructed (same dependency as Options 1 and 2)

**All options disqualified if:**
- April 29 G2 FAIL — Girls calibration unstable. Entire ECNL migration plan is held until calibration is confirmed stable. ELO escalates to SENTINEL same session.

---

## Section 4: Running Verdict Tracker

ELO updates this table after each checkpoint runs. Update the corresponding row before filing the ELO briefing for that session.

| Checkpoint | Scheduled Date | Run Date | Actual Result (one line) | Option 1 Status | Option 2 Status | Option 3 Status |
|-----------|---------------|----------|--------------------------|-----------------|-----------------|-----------------|
| CP1 — ECNL RL staleness | 2026-04-29 | | | Active | Active | Active |
| CP2 — ecnl_verified backfill | 2026-04-29 | | | Active | Active | Active |
| G2 gate check | 2026-04-29 | | | Active | Active | Active |
| CP3 — First pipeline ECNL ingestion | 2026-05-01 | | | Active | Active | Active |
| CP4 — ECNL game volume (3 runs) | 2026-05-03 | | | Active | Active | Active |
| CP5 — ecnl_verified accuracy spot check | 2026-05-05 | | | Active | Active | Active |
| CP6 — ECNL team rating stability | 2026-05-07 | | | Active | Active | Active |
| **CP7 — Final option selection** | **2026-05-10** | | | **Recommended / Eliminated** | **Recommended / Eliminated** | **Recommended / Eliminated** |

**Instructions:** In each cell under Option 1/2/3 Status, write one of: `Active` / `Disqualified — [checkpoint that triggered it]` / `Recommended` (for the final selected option). Do not leave cells blank after the checkpoint date has passed.

---

## Section 5: Decision Protocol

When CP7 date arrives (May 10), apply in order:

1. **Identify disqualified options** from Section 4 Running Verdict Tracker.
2. **If exactly 1 option remains Active:** ELO recommends that option. File decision in Option Selection Decision Gate Section 5. SENTINEL authorizes. FORGE implements.
3. **If 2+ options remain Active:** Apply tiebreakers in order:
   - Prefer the option with lowest implementation risk (Option 1 < Option 2 < Option 3)
   - Prefer the option with the most COLLECTED evidence (fewest PENDING rows in Section 2)
   - If still tied: ELO selects Option 1 as default and documents why in the Option Selection Decision Gate
4. **If 0 options remain Active:** ELO escalates to SENTINEL with a Queue item. New option development required before June 1. This is a same-session escalation — do not defer to the next briefing.

**In all cases:** the decision is filed in `Rankings/ECNL Migration — Option Selection Decision Gate.md` Section 5 by May 10. The "option selected" field in that document is the authoritative record.

---

## References

- [[ECNL Migration — Option Selection Decision Gate]] — decision gate this matrix informs; Section 5 receives the final selection
- [[ECNL Migration Rating Continuity Spec — June 2026]] — option definitions, calibration delta analysis (Section 2), ELO recommendation (Section 3)
- `task-2026-04-26-ecnl-migration-evidence-collection.md` — the 7-checkpoint campaign this matrix tracks
- [[April 29 ELO Session Master Script]] — CP1 is Step 1 of the April 29 session; this matrix must exist before Step 1 runs
- `Infrastructure/ECNL RL Staleness Verification — Presten Execution Package.md` — CP1 execution source

*ELO — 2026-04-26*
