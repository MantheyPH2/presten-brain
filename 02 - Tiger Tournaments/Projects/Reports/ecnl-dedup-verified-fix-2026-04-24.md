---
title: ECNL Transition Dedup — verified=true Fix Note
tags: [ecnl, dedup, june-1, migration, p0, evo-draw]
created: 2026-04-24
updated: 2026-04-24
author: FORGE
status: complete
task: task-2026-04-24-ecnl-dedup-verified-flag
---

# ECNL Dedup `verified = true` Fix Note

**P0 — Must be confirmed before `ecnl-transition-dedup.js` runs on or before June 1, 2026.**

---

## Finding: Script Does Not Yet Exist

`scripts/ecnl-transition-dedup.js` was not found in the accessible filesystem. This is expected — the June 1 Risk Assessment critical path schedules FORGE to begin writing it on **May 1, 2026**.

This document serves as the spec note that must govern the INSERT statement when the script is written. It closes ELO's P0 flag proactively: the risk is not a pre-existing bug but a spec requirement that must be enforced when the script is authored.

---

## Mandatory INSERT Pattern

When `ecnl-transition-dedup.js` is written, every `INSERT INTO team_merges` in the script **must** use this pattern:

```javascript
// ecnl-transition-dedup.js — mandatory INSERT pattern
// verified = true is NOT optional
await client.query(`
  INSERT INTO team_merges (
    canonical_team_id,
    merged_team_id,
    merge_method,
    verified,
    verified_by,
    created_at
  )
  VALUES ($1, $2, 'ecnl_transition', true, 'ECNL_TRANSITION_2026', NOW())
  ON CONFLICT DO NOTHING
`, [gotsportTeamId, tgsTeamId]);
```

**What breaks if `verified` is omitted or set to `false`:**

The Ranking Engine Phase 1 only loads `team_merges` records where `verified = true` (18,684 of 27,820 current rows are verified). If the transition dedup inserts with `verified = false` or omits the column (defaults to `false`), the Ranking Engine ignores all new merge records entirely. Every ECNL team whose platform ID changed from TGS → GotSport on June 1 starts Phase 3 (Glicko-2 computation) as a brand-new provisional team. Rating resets to ~1000. Estimated 500–2,000 ECNL teams affected.

---

## Schema Verification

From `Database Schema.md`:

| Table | Column | Status |
|---|---|---|
| `team_merges` | `verified` | **Confirmed present.** 18,684 of 27,820 rows have `verified = true`. Column exists and is in active use by the Ranking Engine. |

No DB schema query required — the column is confirmed.

---

## All INSERT Sites

The script does not yet exist, so there are no current INSERT sites to audit. When FORGE writes the script in May, the following invariant must hold:

- **Every path that creates a `team_merges` row must include `verified = true`.**
- If the script has both a batch insert path and a single-record fallback, both must include `verified = true`.
- If the script handles edge cases (e.g., a TGS team matching multiple GotSport candidates), every INSERT branch must include `verified = true`.

---

## Test File Note

No test files to review since the script doesn't exist yet. When Presten reviews the script before its June 1 run, confirm with:

```sql
-- Confirm all ECNL_TRANSITION_2026 merges are verified before running compute-rankings.js
SELECT verified, COUNT(*) as count
FROM team_merges
WHERE verified_by = 'ECNL_TRANSITION_2026'
GROUP BY verified;
-- Expected: one row only — verified=true, count = N
-- If any row shows verified=false: do NOT run compute-rankings.js
```

---

## ELO P0 Flag — Disposition

**Status: PROACTIVELY CLOSED.**

The risk was: `ecnl-transition-dedup.js` might be written with the wrong INSERT pattern. This note is now on record before the script is authored. FORGE will implement per this spec on May 1. Presten should confirm the INSERT before the script runs.

This does not require Presten action before May 1. Required actions:

| Date | Action | Owner |
|---|---|---|
| May 1 | Begin writing `ecnl-transition-dedup.js` per this spec | FORGE |
| ~May 10 | Script complete; Presten reviews INSERT statement(s) for `verified = true` | Presten |
| June 7–14 | Run `ecnl-transition-dedup.js`; confirm all inserts are verified before re-running compute-rankings | FORGE/Presten |

---

## References

- `Rankings/June 1 Risk Assessment — Age Group Transition + ECNL Migration.md` — Section 4, Patch 1 (source of this spec)
- `Infrastructure/Database Schema.md` — `team_merges` table definition
- `Reports/ecnl-migration-validation-plan.md` — FORGE's data continuity validation script
