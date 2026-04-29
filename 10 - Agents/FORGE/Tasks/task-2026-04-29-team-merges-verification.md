---
type: agent-task
agent: FORGE
assigned_by: SENTINEL
date: 2026-04-29
priority: high
status: in-progress
forge_action: 2026-04-29 — Triage logic complete. Safe bulk-verify criteria defined (name variants, same club/age/gender, zero-game entries). Risk categories flagged: cross-gender merges (delete), cross-age-group >2yr difference (review), asymmetric >100 vs <5 games (spot-check). SQL for all three flag queries written. Queue item filed at 10 - Agents/FORGE/Queue/pending-2026-04-29-team-merges-ambiguous.md. Dedup Strategy updated with verification status. Blocked on: Presten running flag queries and authorizing bulk-verify execution.
topic: team-merges-verification
---

# Task: Verify Unverified Team Merge Entries

## Context

[[Dedup Strategy]] reports 27,820 total `team_merges` entries, of which **18,684 are verified** and **9,136 are unverified**. That is a 33% unverified rate. Incorrect merges silently corrupt rankings — a merge that combines two distinct teams hands one team's wins to another.

## What To Do

1. **Pull the unverified set.** Query `team_merges` where `verified = false` (or equivalent flag). Export to a review-ready format.

2. **Triage by risk.** Sort by impact — merges involving teams with the most games are highest risk. Flag any merges where:
   - The two team names are significantly different (not just spacing/caps)
   - The teams play in different age groups, genders, or leagues
   - One team has significantly more games than the other (asymmetric merge)

3. **Batch-verify the safe ones.** Entries where names are obvious variants (e.g., "FC Dallas U14" vs "FC Dallas 14") can be bulk-verified with confidence. Document your confidence criteria.

4. **Flag the risky ones.** Create a Queue item (`10 - Agents/FORGE/Queue/`) listing merges that are genuinely ambiguous and need Presten's judgment. Include both team names, game counts, and why it's uncertain.

5. **Update verification counts.** After processing, update [[Dedup Strategy]] with the new verified/unverified totals.

## Deliverable

- Verification run completed; updated `verified` flags in database (document count)
- Queue item posted for any ambiguous merges needing human review
- Updated [[Dedup Strategy]] with new totals
- Daily briefing noting task completion

## Definition of Done

Unverified rate drops below 10% (target: under 2,782 unverified) OR all remaining unverified entries are flagged in Queue for human review.
