---
title: April 28 Escalation Reference Card
tags: [execution, escalation, p0, sentinel, evo-draw]
created: 2026-04-24
updated: 2026-04-24 (Section 3 integrated from ELO's ga-aspire-tier-fix-2026-04-24.md at 15:46)
author: FORGE
status: ready — use on April 28, 2026
task: task-2026-04-24-april28-escalation-reference-card
---

# April 28 Escalation Reference Card

---

## Section 1: Deadline Summary

```
Date: April 28, 2026
Time required: ~75–90 minutes total (assuming GA ASPIRE SQL is staged — see Section 3 flag)
Items: 2 (both must be done today to avoid SENTINEL escalation)
```

| Item | Consequence if not done by April 28 |
|---|---|
| ECNL dedup P0 spec review | June 1 ECNL migration risk — if `ecnl-transition-dedup.js` is written without `verified = true`, 500–2,000 ECNL teams lose game history overnight. SENTINEL posts queue alert. |
| GA ASPIRE tier fix | DSS demo shows rankings with ~40pt overcalibration on all GA ASPIRE-level teams (350–1,050 teams). SENTINEL posts queue alert. |

---

## Section 2: Item 1 — ECNL Dedup P0 Spec Confirmation (~15 min)

> [!note] Status as of April 24
> `scripts/ecnl-transition-dedup.js` does **not yet exist** — it is scheduled to be written on **May 1, 2026**. The April 28 action is not a code deploy. It is a spec confirmation: read the mandatory INSERT pattern, confirm it is understood, and confirm May 1 is on the calendar.

**The mandatory INSERT pattern** (from `Reports/ecnl-dedup-verified-fix-2026-04-24.md`):

```javascript
// Every INSERT INTO team_merges in ecnl-transition-dedup.js MUST use this pattern:
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

**Steps for April 28:**

1. Open `Reports/ecnl-dedup-verified-fix-2026-04-24.md` — read the full INSERT spec (2 minutes).
2. Confirm May 1 is blocked for writing `scripts/ecnl-transition-dedup.js`.
3. When the script is written (May ~10), run this verification query before running `compute-rankings.js`:

```sql
-- Confirm all ECNL_TRANSITION_2026 merges have verified = true
SELECT COUNT(*) FROM team_merges 
WHERE merge_type = 'ecnl_transition' 
  AND (verified IS NULL OR verified = false);
-- Expected: 0. If > 0: do NOT run compute-rankings.js.
```

**Done state for April 28:** May 1 script-writing session confirmed on calendar. INSERT spec reviewed. ✅

---

## Section 3: Item 2 — GA ASPIRE Tier Fix (~45–60 min)

> Source: `Reports/ga-aspire-tier-fix-2026-04-24.md` (ELO, filed 2026-04-24). Copy-paste all SQL below verbatim.

**Pre-condition — check Boys GA ASPIRE exposure (run first):**

```sql
SELECT g.gender, COUNT(DISTINCT g.id) AS game_count
FROM games g
JOIN events e ON e.id = g.event_id
WHERE e.event_name ILIKE '%ASPIRE%'
  AND e.event_tier = 'ga'
GROUP BY g.gender;
```

If Boys game_count > 0: this is safe to proceed — Boys GA calibration is already 100 (same as ga_aspire), so reclassification has no net rating impact for Boys. Note the count in your session log.

**Step 1 — Scope verification:**

```sql
SELECT COUNT(*) AS events_to_reclassify, event_tier
FROM events
WHERE event_name ILIKE '%ASPIRE%'
  AND event_tier = 'ga'
GROUP BY event_tier;
```

Expected: some rows with `event_tier = 'ga'`. If **0 rows**: STOP — events may already be reclassified or the name pattern is wrong. Do not proceed.

**Step 2 — Run the UPDATE:**

```sql
UPDATE events
SET event_tier = 'ga_aspire'
WHERE event_name ILIKE '%ASPIRE%'
  AND event_tier = 'ga';
```

Note rows updated: ___. Must match `events_to_reclassify` from Step 1.

**Step 3 — Verify:**

```sql
-- Confirm no GA ASPIRE events remain tagged as 'ga'
SELECT COUNT(*) AS still_tagged_ga
FROM events
WHERE event_name ILIKE '%ASPIRE%'
  AND event_tier = 'ga';
-- Expected: 0

-- Confirm ga_aspire tier is populated
SELECT COUNT(*) AS now_tagged_aspire, MIN(event_name) AS sample_event
FROM events
WHERE event_tier = 'ga_aspire';
-- Expected: > 0, matching rows updated in Step 2
```

**Step 4 — Recompute rankings.**

The calibration change (140 → 100 for Girls GA ASPIRE) does not take effect until rankings are recomputed. Trigger: `node scripts/compute-rankings.js`. Post-recompute verification: see `Reports/ga-aspire-post-fix-verification-2026-04-28.md` (ELO spec — confirms avg rating shift <15 pts for U13/U14).

**Rollback (if needed within 48 hours):**

```sql
UPDATE events
SET event_tier = 'ga'
WHERE event_name ILIKE '%ASPIRE%'
  AND event_tier = 'ga_aspire';
```

Then recompute rankings to restore prior calibration.

---

## Section 4: Done State

After completing both items:

```
□ ECNL P0 spec: May 1 calendar confirmed, INSERT spec reviewed
□ GA ASPIRE fix: UPDATE executed, Step 3 verification queries return (0, >0), post-recompute check passes per `Reports/ga-aspire-post-fix-verification-2026-04-28.md`
□ Note completion time in Presten Session Plans log (Product & Planning/Presten Session Plans — April 2026.md)
```

When both boxes are checked, SENTINEL will mark these items resolved and remove them from the escalation queue.

---

## Source Documents

| Item | Source |
|---|---|
| ECNL P0 spec | `Reports/ecnl-dedup-verified-fix-2026-04-24.md` |
| GA ASPIRE approval | `10 - Agents/ELO/Queue/pending-2026-04-23-ga-calibration-review.md` |
| GA calibration analysis | `Reports/ga-calibration-validation-2026-04-23.md` |
| U12/U13/U14/U19 calibration (separate) | `Rankings/calibration-fix-pre-deploy-2026-04-23.md` — NOT the GA ASPIRE fix |
| Broader session context | `Product & Planning/Presten Session Plans — April 2026.md` |

---

*FORGE — 2026-04-24*
