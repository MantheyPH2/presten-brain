---
type: elo-recommendation
subject: ECNL Migration Option Selection
author: ELO
date: 2026-04-28
recommended_option: Option 1 — No Re-tag
status: recommendation-filed
sentinel_authorization_required: yes
presten_authorization_required: no
tags: [ecnl, migration, recommendation, calibration, sentinel-gate, evo-draw]
---

# ECNL Migration — ELO Option Recommendation

> **Self-contained decision document.** Presten and SENTINEL can read this document and make a YES/NO authorization decision without reading any prior spec. Cross-references are provided for supporting detail; this document is complete on its own.

---

## Recommendation Statement

ELO recommends **Option 1 — No Re-tag** for the ECNL team entity migration.

Option 1 requires no schema changes, no UPDATE SQL on historical games, and no recompute beyond the regular post-migration run: it fits the current technical state cleanly and preserves the rating signal that Path A teams legitimately earned. Option 2 was not selected because it produces an estimated 40–80 point rating drop overnight for teams that earned those ratings through genuine ECNL competition — a visible cliff that cannot be explained to club directors in a way that reflects well on Evo Draw. Option 3 was not selected because it requires a new `ecnl_legacy` DDL tier, a blended calibration value ELO has no cross-league data to support, and adds schema complexity that Option 1 achieves without. The timing is appropriate now because the checkpoint sequence (CP1–CP7, completing May 10) can validate or disqualify this recommendation before June 1 execution — there is enough runway to revise if evidence changes.

---

## Key Risk Accepted

**The seam risk:** Under Option 1, Path A teams (ex-ECNL, now in ECNL RL) will carry higher ratings than equivalent-performance Path B teams (ECNL RL veterans) for an estimated 6–12 months post-June-1 migration, as the ECNL historical games recency-weight out. The mitigation: ELO monitors the Path A vs. Path B rating gap at June 7–14 and files a revised recommendation if the gap exceeds 150 points — that magnitude would indicate a calibration anomaly rather than a defensible historical signal.

---

## Rating Continuity Assessment

Under Option 1, ELO expects **MINIMAL** rating disruption for Path A teams at the point of migration: **< 5 pts average** on the migration date itself. The June 1 migration does not alter any historical game tags, so no recompute is triggered by the migration alone. Path A teams' ratings on June 2 are identical to June 1 pre-migration. The only rating movement post-June-1 comes from new ECNL RL games being ingested (cal=55), which naturally compresses the Path A/Path B gap through the normal ranking cycle.

Reference: `Rankings/ECNL Migration Rating Continuity Spec — June 2026.md` Section 3. Deviation threshold from that spec (a detectable shift > 5 pts average for Path A teams on migration date): not triggered under Option 1.

---

## FORGE Dependency

FORGE should activate **Option 1 (No Re-tag) section** of `Infrastructure/ECNL Migration — FORGE Pipeline Impact Spec.md`. Under Option 1, FORGE's implementation work is limited to: (1) confirming the event classifier correctly routes new ECNL RL events to `event_tier = 'ecnl_rl'` starting June 1, and (2) inserting `ecnl_transition` records in `team_merges` with `verified = TRUE` for Path A teams. No UPDATE SQL on historical games. No DDL changes. Expected FORGE implementation start: May 1 (concurrent with the `ecnl-transition-dedup.js` script). Completion window: May 1–14, well ahead of June 1.

---

## Checkpoint Sequence (CP1–CP7)

This recommendation is subject to disqualification through the checkpoint sequence defined in `Rankings/ECNL Migration — Option Comparison Matrix.md`. The key disqualifying conditions for Option 1 are:

| Checkpoint | Date | Disqualifies Option 1 if |
|-----------|------|--------------------------|
| CP1 — ECNL RL staleness | 2026-04-29 | ECNL RL source > 30 days stale |
| CP2 — ecnl_verified backfill | 2026-04-29 | Backfill < 70% complete OR ecnl_transition records absent |
| CP5 — ecnl_verified accuracy | 2026-05-05 | Flag accuracy < 80% AND FORGE has no fix timeline |

If Option 1 is disqualified at any checkpoint, ELO will file a revised recommendation within the same briefing session. The next-best option is Option 2, which requires Presten sign-off on the rating cliff before execution.

---

## Authorization Request

ELO requests SENTINEL and Presten authorize **Option 1 — No Re-tag** for implementation by May 1 (FORGE start date). Once authorized, FORGE activates the Option 1 section in `Infrastructure/ECNL Migration — FORGE Pipeline Impact Spec.md` and begins implementation. SENTINEL gate: the 4 conditions from Section 6 of the FORGE Pipeline Impact Spec must be met before any production schema change executes.

**Presten authorization not required for Option 1** — there are no schema changes and no UPDATE outside the normal migration flow. Presten should be aware that Path A teams will carry forward their ECNL history and may appear 20–50 points above equivalent Path B teams in the transition window. This is expected behavior, not a bug.

If SENTINEL overrides to Option 2 or Option 3, Presten sign-off is required before any UPDATE runs.

---

## What Presten Needs to Know

- **Nothing changes in the database on April 28 or April 29** as a result of this recommendation. The ECNL migration executes on June 1.
- **The May 1 pipeline run** is the first checkpoint with real data (CP3). FORGE handles the event classifier update.
- **The April 29 session** runs CP1 (ECNL RL staleness check) and CP2 (ecnl_verified backfill confirmation). Results feed back to ELO the same session.
- **Final option selection** is May 10 (CP7). If checkpoint data disqualifies Option 1, ELO flags immediately with a revised recommendation. Presten does not need to monitor the checkpoints — ELO does.

---

## References

- `Rankings/ECNL Migration — Option Comparison Matrix.md` — full checkpoint matrix; Section 4 (Running Verdict Tracker) ELO updates after each CP
- `Rankings/ECNL Migration Rating Continuity Spec — June 2026.md` — Option 1 rationale and rating impact analysis (Sections 2–3)
- `Rankings/ECNL Migration — Option Selection Decision Gate.md` — decision gate document; Section 5 receives the final confirmed option after CP7
- `Infrastructure/ECNL Migration — FORGE Pipeline Impact Spec.md` — FORGE activates Option 1 section on authorization
- `Rankings/ECNL Rating Continuity Spec — June 2026.md` — verified-flag mechanism (prerequisite for all options)
- `task-2026-04-28-ecnl-migration-option-recommendation.md` — source task

*ELO — 2026-04-28*
