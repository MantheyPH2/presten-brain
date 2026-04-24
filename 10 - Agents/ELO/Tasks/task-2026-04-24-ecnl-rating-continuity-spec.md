---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-24
status: completed
completed: 2026-04-24
deliverable: "02 - Tiger Tournaments/Projects/Rankings/ECNL Rating Continuity Spec — June 2026.md"
priority: high
due: 2026-04-30
---

# Task: ECNL Rating Continuity Spec — How Elo Ratings Carry Through the June 1 Migration

## Context

ELO's June 2 ECNL Migration Verification Spec (delivered 2026-04-24) is well-written and correctly checks for visible rating anomalies the morning after migration. It contains one structural dependency that has never been validated: **the spec assumes Elo ratings are preserved correctly through the migration.**

That assumption may be wrong. The June 1 migration writes `ecnl_transition` records to `team_merges`, changing which `canonical_team_id` is authoritative for ECNL teams. If the Elo computation reads ratings by `team_id` and a team's `canonical_team_id` changes, the new canonical ID starts with no rating history — it initializes at 1000. On June 2, this would appear as a dramatic rating collapse for every affected team.

The June 2 verification spec catches this IF an affected team is in the 100-team spot-check list. It would miss the pattern if it's happening to teams outside that list. The fix requires a specific verification query that looks for the failure signature across all ECNL teams, not just the spot-check set.

This task: write the rating continuity spec that either confirms ratings are safe or identifies the exact failure scenario and what FORGE needs to fix.

## What to Investigate and Write

Write: `02 - Tiger Tournaments/Projects/Rankings/ECNL Rating Continuity Spec — June 2026.md`

---

### Section 1: Document the Current Rating Storage Model

From `Database Schema.md` and/or `Rankings/Ranking Engine.md`, determine and state:

1. Where are Elo ratings stored? (`rankings.team_id`, `rankings.canonical_team_id`, or another table?)
2. When two teams are merged in `team_merges`, how does the Elo computation handle the merged game history?
   - Option A: game records for the merged (non-canonical) team are re-attributed to the canonical_team_id at merge time → ratings computed from consolidated history
   - Option B: game records stay under the original team_id, and a JOIN through team_merges consolidates them at query time → ratings computed from a view
   - Option C: only the canonical team's games are used → merged team's history is discarded
3. For the `ecnl_transition` merge type specifically: is there any logic in the migration script or ranking engine that handles this merge type differently from regular merges?

Write the answer as explicit facts:
> "Ratings are stored in `[table].[column]`. When a merge is written to team_merges, game history is [re-attributed at write time / consolidated at query time via JOIN / not consolidated]. The ecnl_transition merge type [is / is not] handled differently."

If these facts cannot be determined from vault documents alone and require FORGE confirmation, state what is unknown and write the question for FORGE as a flag in your briefing.

---

### Section 2: Identify the Failure Scenario

Based on Section 1, determine whether rating continuity is preserved or at risk:

**If Option A (re-attribution at write time):**
- Migration writes new ecnl_transition merges
- Game records for affected teams are re-attributed to the new canonical IDs
- Ratings are recomputed using the consolidated history
- **Continuity: PRESERVED** — if the re-attribution step runs before the nightly pipeline
- Risk: if the re-attribution step does NOT run before the first post-migration pipeline, ratings recompute from an incomplete game set for one cycle

**If Option B (JOIN at query time):**
- Migration writes new ecnl_transition merges
- Elo computation picks up the new canonical pointer via JOIN
- **Continuity: PRESERVED** — no re-attribution needed; JOIN always reflects current canonical
- Risk: if any game records still reference old team_ids that are no longer valid pointers, they may be excluded

**If Option C (canonical only) or unknown:**
- **Continuity: AT RISK** — merged team's history is discarded; new canonical starts at 1000
- This is the failure scenario. If true, flag to SENTINEL immediately as P0.

Write explicitly:
> "Rating continuity is [PRESERVED / AT RISK / UNKNOWN] because [specific reason referencing the storage model]. The failure scenario is [description] and [does / does not] apply to the June 1 migration based on the storage model."

---

### Section 3: Add the Continuity Verification Query to the June 2 Spec

Regardless of the Section 2 determination, add a verification query to the June 2 spec that detects the specific failure signature: ECNL teams with ratings suspiciously close to initialization (1000) that were previously well above it.

Add this query as an additional check in `Rankings/June 2 ECNL Migration Verification Spec.md`, after the existing 5 checks:

**Check 6: Rating Orphan Detection (ECNL Teams)**

```sql
-- Teams with ecnl_transition merges whose post-migration rating is near initialization value
-- A high-rated pre-migration team that now shows ~1000 indicates rating history was orphaned
SELECT 
  t.name,
  r.rating AS current_rating,
  r.rank AS current_rank,
  COUNT(tm.id) AS ecnl_transition_merges
FROM rankings r
JOIN teams t ON t.id = r.team_id
JOIN team_merges tm ON (tm.canonical_team_id = r.team_id OR tm.merged_team_id = r.team_id)
WHERE tm.merge_type = 'ecnl_transition'
  AND r.age_group = 'U13' AND r.gender = 'F'
  AND r.rating BETWEEN 950 AND 1050
GROUP BY t.name, r.rating, r.rank
ORDER BY r.rank;
```

**Pass criterion:** Returns 0 rows (no ECNL-migration-involved teams near the initialization floor).
**Failure criterion:** Any rows returned warrant investigation — pull the team name, check whether it was ranked before migration, and confirm whether the merge was processed correctly.

Adapt this query for at least U13G and U14G. Note in the spec: "Run this for all age groups if time permits."

---

### Section 4: State the Migration Script Requirement

Write an explicit requirement for FORGE to confirm (or act on if continuity is at risk):

If **Option A (re-attribution):**
> "REQUIREMENT: The ECNL migration script or the post-migration pipeline step must execute game re-attribution before the first nightly ranking recompute. Confirm with FORGE that this step exists and runs in the correct order. If it does not, a gap exists between migration and the next pipeline run where ratings are based on an incomplete game set."

If **Option B (JOIN at query time):**
> "REQUIREMENT: Confirm with FORGE that the team_merges JOIN in the Elo computation correctly picks up ecnl_transition merge records. This can be verified by querying: `SELECT COUNT(*) FROM team_merges WHERE merge_type = 'ecnl_transition'` before and after the migration — the count should increase by the number of ECNL teams migrated."

If **Option C or unknown:**
> "REQUIREMENT: Rating continuity is at risk. FORGE must implement a re-attribution step before June 1. SENTINEL flagged as P0. ELO should write the re-attribution specification."

---

## Deliverable

1. `02 - Tiger Tournaments/Projects/Rankings/ECNL Rating Continuity Spec — June 2026.md` — the full spec covering Sections 1–4
2. Updated `Rankings/June 2 ECNL Migration Verification Spec.md` — add Check 6 (Rating Orphan Detection) after the existing 5 checks

## Definition of Done

- Rating storage model is documented with specific table/column references (or explicit "unknown — question for FORGE" if not determinable from vault)
- "Preserved / At Risk / Unknown" determination is stated explicitly with reasoning
- Check 6 is added to the June 2 spec with pass/fail criterion
- Migration script requirement is written for FORGE's Section 3.2 of their pre-flight checklist
- Summary in briefing: "ECNL rating continuity: [preserved / at risk / unknown]. Failure scenario: [one sentence]. June 2 spec updated with Check 6."

## Coordination Note

FORGE is simultaneously writing `Infrastructure/ECNL Migration Pre-Flight Checklist — June 2026.md`. Section 3 of that checklist currently has a placeholder noting that ELO's spec may add a requirement. When this spec is complete, the requirement from Section 4 above should be communicated to FORGE (via your briefing or directly by noting it in the spec's header).

## Why This Matters

The June 2 spec catches anomalies in the teams that are being watched. This spec closes the failure mode where 200 ECNL teams silently reinitialize to 1000 Elo and none of them happen to be in the top-10 spot-check list. That failure surfaces when a club director checks their team on Evo Draw three days after DSS.

## References

- `Rankings/June 2 ECNL Migration Verification Spec.md` — update with Check 6
- `Database Schema.md` — confirm rating storage model
- `Rankings/Ranking Engine.md` — confirm how merges are handled in Elo computation
- FORGE task `task-2026-04-24-ecnl-migration-pre-flight-checklist.md` — companion document; Section 3 awaits ELO's requirements
