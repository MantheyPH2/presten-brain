---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-25
status: completed
priority: medium
due: 2026-04-28
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/GotSport Browser Lookup Session Prep Package.md"
---

# Task: GotSport Browser Lookup Session Prep Package

## Context

Four source verification tasks are blocked on a single Presten browser session at `system.gotsport.com`:

1. `task-2026-04-25-usl-academy-edp-org-id-verification.md` — USL Academy and EDP org IDs unconfirmed
2. `task-2026-04-25-elite64-nal-source-research.md` — Elite 64 GotSport presence unconfirmed, NAL platform unknown
3. USYS Stage 2 P1 state org IDs (CA, FL, NY, NJ, WA, PA, OH) listed as `null` in the Org-ID Master List

This recommendation has been in every SENTINEL briefing since April 24 13:31. The browser session keeps not happening because the lookup work is fragmented across three task files and the Org-ID Master List — Presten would need to read all four to know what to look up in a single session.

This task: consolidate everything into one document. One URL, one session, everything to look for, how to record results. Presten opens this document, completes the session in ~45 minutes, and FORGE can immediately close the three blocked tasks.

## What to Do

### Step 1: Compile All Pending Lookups

Pull from the three blocked task files and the USYS Org-ID Master List. For each entity with a pending lookup, create a row:

| # | Entity | GotSport Lookup | What to Record | Expected Time |
|---|--------|-----------------|----------------|---------------|
| 1 | USL Academy | Search by organization name | org_id (URL parameter) | 5 min |
| 2 | EDP Soccer | Search by organization name | org_id | 5 min |
| 3 | Elite 64 | Search by organization name; also check US Club Soccer org if not found directly | org_id, or "not found" | 10 min |
| 4 | NAL (National Academy League) | Check NAL website first for result publication platform; if GotSport, look up org | platform name, org_id if applicable | 10 min |
| 5–11 | USYS Stage 2 states (CA, FL, NY, NJ, WA, PA, OH) | Search each state association by name | org_id for each | 15 min (2 min each) |

Fill in accurate entity names from the existing task files. Pull the correct state association names from the USYS Org-ID Master List (not abbreviated — use the names that will match GotSport search).

### Step 2: Write the Session Procedure

**URL:** `system.gotsport.com` — Organization search

**How to find an org_id:**
1. Navigate to the organization search
2. Enter the organization name
3. Click through to the organization page
4. Extract the org_id from the URL (e.g., `?OrgId=XXXX`) — record this number

**For "not found" cases:**
- If GotSport search returns no result for the expected organization name, try: (a) abbreviated name, (b) parent organization, (c) state association affiliation
- If still not found, record "not on GotSport" — FORGE will update the Source Gap Inventory to reflect the platform as unknown and escalate to SENTINEL for alternative source research

**Recording results:**
Write a simple recording table in the document:

```
| Entity | Found? (Y/N) | org_id | Notes |
|--------|-------------|--------|-------|
| USL Academy | | | |
| EDP Soccer | | | |
| Elite 64 | | | |
| NAL | | | |
| USYS CA | | | |
| USYS FL | | | |
| USYS NY | | | |
| USYS NJ | | | |
| USYS WA | | | |
| USYS PA | | | |
| USYS OH | | | |
```

Presten fills this in during the session and shares results — FORGE updates the Source Gap Inventory and Org-ID Master List immediately.

### Step 3: Write the Delivery Document

Deliverable: `02 - Tiger Tournaments/Projects/Infrastructure/GotSport Browser Lookup Session Prep Package.md`

Sections:
1. **Session Summary** — What this session accomplishes, estimated total time (~45 min), how many blocked tasks it unblocks (3 task files + Org-ID Master List)
2. **URL and Access** — `system.gotsport.com`, how to extract org_id from URL
3. **Lookup Table** — All 11 entities with names, lookup method, and what to record
4. **Recording Template** — Blank table for Presten to fill in
5. **After the Session** — "Send results to FORGE. FORGE updates Source Gap Inventory within 24 hours."

## Definition of Done

- All pending lookups consolidated in one document (no need to open any other file)
- Entity names are exact (matched to GotSport search expectations)
- Recording template is blank and ready to fill in
- Session time estimate is accurate (within ±10 minutes)
- Document is self-contained: Presten can complete the session without opening any other file
- Summary in briefing: "Browser Lookup Session Prep Package filed. [N] entities, ~[N] min total. Unblocks 3 task files and Stage 2 USYS state org IDs."

## Why This Matters

The browser lookup session has been SENTINEL's top Presten unblocking recommendation for 14+ hours. The fragmentation of the work across three task files and the Org-ID Master List is the reason it hasn't happened. One consolidated document removes that barrier. Estimated value: 3 task closures + 7 state org-IDs confirmed + Stage 2 pre-condition met.

## References

- `10 - Agents/FORGE/Tasks/task-2026-04-25-usl-academy-edp-org-id-verification.md`
- `10 - Agents/FORGE/Tasks/task-2026-04-25-elite64-nal-source-research.md`
- `02 - Tiger Tournaments/Projects/Infrastructure/GotSport Org-ID Expansion Stage 2 — Pre-Authorization Criteria.md`
- USYS Org-ID Master List (in Infrastructure/ — confirm exact filename from vault)
