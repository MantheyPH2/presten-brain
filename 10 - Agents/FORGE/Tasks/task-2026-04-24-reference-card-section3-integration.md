---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-24
status: pending
priority: urgent
due: 2026-04-26
deliverable: "02 - Tiger Tournaments/Projects/Product & Planning/April 28 Escalation Reference Card.md"
---

# Task: Integrate GA ASPIRE SQL Into April 28 Reference Card Section 3

## Context

The April 28 Escalation Reference Card was filed in FORGE's 11:16 session. Section 3 (GA ASPIRE Tier Fix) was left with a flag:

> "⚠️ GA ASPIRE SQL NOT FOUND — SENTINEL must locate or reconstruct. Check ELO's calibration spec."

SENTINEL has assigned ELO a companion task (`task-2026-04-24-ga-aspire-update-sql.md`, due April 25) to write the executable SQL at:
`02 - Tiger Tournaments/Projects/Reports/ga-aspire-tier-fix-2026-04-24.md`

Once ELO files that document, FORGE must integrate its contents into Section 3 of the reference card. This task tracks that integration.

## What to Do

### Step 1: Confirm ELO's SQL File Exists

Check for: `02 - Tiger Tournaments/Projects/Reports/ga-aspire-tier-fix-2026-04-24.md`

If it does not exist by April 25: post a flag in your briefing and notify SENTINEL. Do not wait silently past the due date.

### Step 2: Read ELO's SQL File

Pull from it:
1. The pre-fix verification query (how many events will be reclassified)
2. The Boys GA ASPIRE pre-condition check (if Boys games > 0, action required)
3. The UPDATE statement
4. The expected row count statement
5. The post-fix verification queries
6. The rollback statement (for completeness in the reference card)

### Step 3: Replace Section 3 in the Reference Card

Open `02 - Tiger Tournaments/Projects/Product & Planning/April 28 Escalation Reference Card.md`.

Replace the current Section 3 flag block with:

```
### Section 3: Item 2 — GA ASPIRE Tier Fix (~45–60 min)

**Pre-condition (run first):**
[Boys GA ASPIRE pre-condition from ELO's SQL file — 2 lines]

**Step 1 — Scope verification:**
[pre-fix verification query]
Expected: [N] events to reclassify. If 0: STOP — investigate pattern.

**Step 2 — Run the UPDATE:**
[UPDATE SQL verbatim from ELO's file]
Note rows updated: ___

**Step 3 — Verify:**
[post-fix verification queries — 2 queries]
Expected: 0 remaining as 'ga', [N] now tagged 'ga_aspire'

**Step 4 — Recompute rankings.**
Rankings must recompute before the calibration change takes effect.
```

Do not paraphrase the SQL. Copy it verbatim from ELO's file.

### Step 4: Remove the Warning Flag

The current Section 3 has a `⚠️` warning block. Remove it after integration. The reference card should read clean on April 28.

### Step 5: Update the Completion Checklist

Section 4 of the reference card already has a `□ GA ASPIRE fix: calibration change confirmed in DB` checkbox. Confirm it references ELO's post-fix verification spec (`Reports/ga-aspire-post-fix-verification-2026-04-28.md`) as the authority for what "confirmed" means.

## Definition of Done

- Section 3 of the April 28 Reference Card contains copy-paste SQL (no flags, no placeholders)
- SQL is sourced verbatim from `Reports/ga-aspire-tier-fix-2026-04-24.md`
- Boys GA ASPIRE pre-condition is present as the first line of Section 3
- The `⚠️` warning flag is removed
- Summary in briefing: "Section 3 integrated from `ga-aspire-tier-fix-2026-04-24.md`. Reference card is now fully actionable for April 28."

## Why This Matters

The April 28 Reference Card is the single document Presten opens on April 28. Section 3 being flagged empty means the GA ASPIRE fix — the harder of the two April 28 items — has no copy-paste instructions. Presten would have to hunt across multiple documents under time pressure. This integration task closes the last gap in the reference card.

## Dependencies

- ELO task `task-2026-04-24-ga-aspire-update-sql.md` must complete first (due April 25)
- If ELO does not file by April 25, flag to SENTINEL immediately

## References

- `02 - Tiger Tournaments/Projects/Product & Planning/April 28 Escalation Reference Card.md` — file to update
- `02 - Tiger Tournaments/Projects/Reports/ga-aspire-tier-fix-2026-04-24.md` — ELO's SQL (source)
- `02 - Tiger Tournaments/Projects/Reports/ga-aspire-post-fix-verification-2026-04-28.md` — ELO's post-fix spec (reference in Section 4)
- `10 - Agents/FORGE/Briefings/2026-04-24-11:16.md` — FORGE's original flag on this gap
