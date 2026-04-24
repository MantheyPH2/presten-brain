---
title: April 28 Escalation Reference Card
tags: [execution, escalation, p0, sentinel, evo-draw]
created: 2026-04-24
updated: 2026-04-24
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

> [!warning] GA ASPIRE SQL NOT FOUND
> `Reports/ga-aspire-tier-fix-2026-04-[date].md` does not exist in the vault. `Rankings/calibration-fix-pre-deploy-2026-04-23.md` covers U12/U13/U14/U19 calibration — it does not contain GA ASPIRE SQL.
>
> **SENTINEL must locate or reconstruct the GA ASPIRE UPDATE SQL before this item can be executed.**
>
> Check: ELO's GA calibration validation (`Reports/ga-calibration-validation-2026-04-23.md`) and the approved queue item (`10 - Agents/ELO/Queue/pending-2026-04-23-ga-calibration-review.md`). ELO approved adding `ga_aspire` at calibration 100. The SQL must be constructed from that spec or retrieved from ELO.

**What the fix should do (from the ELO approval):**

The GA ASPIRE fix requires:
1. A new `ga_aspire` tier entry in the calibration system at value **100** (down from the current inherited GA value of 140).
2. Events tagged as GA that contain "aspire" in the event name to be re-classified as `ga_aspire`.
3. Estimated affected teams: **350–1,050** across 7 age groups.

**Once the SQL is located or provided by ELO/SENTINEL:**

```
Expected execution:
1. Open a psql session: psql -U p -h localhost -p 5432 youth_soccer
2. Run the UPDATE statement to classify ASPIRE events
3. Verify: check that affected_rows matches expected 350–1,050 range
4. If pipeline does not recompute automatically, trigger: node scripts/compute-rankings.js
```

**Expected verification query (adapt when SQL is staged):**

```sql
-- After fix: confirm ga_aspire tier exists and has a reasonable event count
SELECT event_tier, COUNT(*) as event_count
FROM event_tiers
WHERE event_tier IN ('ga', 'ga_aspire')
GROUP BY event_tier
ORDER BY event_tier;
-- Expected: ga_aspire row appears with N > 0 events
```

---

## Section 4: Done State

After completing both items:

```
□ ECNL P0 spec: May 1 calendar confirmed, INSERT spec reviewed
□ GA ASPIRE fix: SQL staged by SENTINEL/ELO, executed, verification query returns ga_aspire row
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
