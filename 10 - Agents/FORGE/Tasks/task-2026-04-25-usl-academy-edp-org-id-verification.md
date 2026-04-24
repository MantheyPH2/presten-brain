---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-25
status: in_progress
priority: high
due: 2026-04-27
vault_research: complete-2026-04-24
org_id_confirmation: pending-presten-browser-lookup
notes: "Source Gap Inventory rows updated with provisional findings at 15:46 on 2026-04-24. USL Academy: high probability GotSport (USL Soccer is a GotSport org). EDP: known GotSport organization (~150+ clubs, Northeast). Org IDs not confirmable without browser access to system.gotsport.com. Requires Presten 15-minute browser session to confirm both org IDs."
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/Source Gap Inventory — April 2026.md (update rows)"
---

# Task: USL Academy + EDP Org-ID Verification

## Context

The Source Gap Inventory filed 2026-04-24 lists two Priority 2 gaps:

- **USL Academy** — GotSport org_id not confirmed. Status: Stale/Partial.
- **EDP (Eastern Development Program)** — GotSport org_id not confirmed. Status: Stale/Partial.

FORGE flagged both in the 13:31 briefing as a 15-minute gap check. This task formalizes it. If org IDs are confirmed and active, both leagues promote from Priority 2 (stale/partial) to Active — closing two coverage gaps without any new engineering work.

## What to Do

### Step 1: Search GotSport for USL Academy

Search GotSport's organization directory for "USL Academy" or "United Soccer League Academy." Document:
- If an org_id is found: record it, note what event types appear under it, estimate game volume from recent events.
- If no org_id is found: mark as "GotSport — not present" and note as a Priority 1 gap requiring alternative source research.

### Step 2: Search GotSport for EDP

Search GotSport's organization directory for "Eastern Development Program" or "EDP." Document:
- If an org_id is found: record it, note active events, estimate game volume.
- If no org_id is found: mark as "GotSport — not present."

### Step 3: Update Source Gap Inventory

In `Infrastructure/Source Gap Inventory — April 2026.md`:
- Update USL Academy row: fill in `Source ID / Config Key` and revise `Status` to Active, Stale, or No Source based on findings.
- Update EDP row: same.
- Update the Priority 2 gap summary — if either moves to Active, remove from the Priority 2 list or annotate as resolved.

### Step 4: If Org IDs Found — Config Action

If USL Academy or EDP has a valid GotSport org_id:
- Note the org_id in the Source Gap Inventory row.
- Add a one-line note: "Org-ID confirmed. Pending addition to crawl config — will require Presten execution during next pipeline session."
- Do NOT add the org_id to the crawl config yourself — flag for Presten.

## Definition of Done

- USL Academy and EDP rows in Source Gap Inventory are updated with confirmed Source ID or "GotSport — not present"
- Status column reflects current accurate state (Active, Stale, or No Source — not "unknown")
- If org IDs are found, flagged for Presten to add to crawl config
- Summary in briefing: "USL Academy: [result]. EDP: [result]. Source Gap Inventory updated."

## Why This Matters

Two Priority 2 gaps may be resolvable with zero engineering — just config updates. If USL Academy and EDP are on GotSport, adding their org IDs is the same 5-minute config change as the TX org-ID expansion already planned. If they're not on GotSport, SENTINEL needs to know that now to scope alternative source research.

## References

- `02 - Tiger Tournaments/Projects/Infrastructure/Source Gap Inventory — April 2026.md` — inventory to update
- `10 - Agents/FORGE/Briefings/2026-04-24-13:31.md` — "Tomorrow's plan" item 3
- `10 - Agents/FORGE/Tasks/task-2026-04-24-gotsport-org-id-expansion.md` — same pattern: find org_id → add to config
