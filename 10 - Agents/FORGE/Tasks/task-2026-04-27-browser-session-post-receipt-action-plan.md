---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-27
status: pending
priority: high
due: 2026-04-28
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/Browser Session — Post-Receipt Org-ID Action Plan.md"
---

# Task: Browser Session Post-Receipt Org-ID Action Plan

## Context

The GotSport browser session (Presten looks up org-IDs for 11 pending entities) could run at any point during or after the April 28 session. FORGE has complete preparation:
- `Infrastructure/GotSport Browser Lookup Session Prep Package.md` — what Presten looks up
- `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` — the master tracker FORGE updates
- `Infrastructure/Stage 2 Org-ID Config — Pre-Staged Additions.md` — config entries ready for org-ID substitution
- `Infrastructure/GotSport Org-ID Config Edit Reference.md` — how to add IDs to the live config

What does not exist: a single document FORGE consults in the 60 minutes after Presten delivers org-ID results — a step-by-step action plan that prevents FORGE from missing an update or filing them in the wrong order.

The post-receipt window is high-stakes and time-pressured. Presten hands over a list of org-IDs; FORGE has multiple documents to update. Without an explicit action plan, FORGE may update the master reference but forget to substitute IDs in the pre-staged config — leaving Stage 2 in an incomplete state. Or FORGE may update the config but file an incomplete briefing note that SENTINEL cannot audit.

This document is FORGE's single reference for the 60 minutes after browser session results arrive.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/Browser Session — Post-Receipt Org-ID Action Plan.md`

---

### Step 0: Receive Results (< 5 minutes)

When Presten files browser session org-ID results (format will vary — could be a list in a message, a table in a vault file, or a direct update to the prep package):

```
[ ] Log receipt: note time in action plan header
[ ] Create a local working list: entity name + confirmed org-ID (or "not-on-gotsport")
[ ] Do NOT start updating docs until the full list is received
```

---

### Step 1: Update GotSport Org-ID Master Reference (< 15 minutes)

File: `Infrastructure/GotSport Org-ID Master Reference — April 2026.md`

For each entity in the browser session results:

```
[ ] If org-ID confirmed:
    [ ] Move row from Section 2 (Pending) to Section 1 (Active)
    [ ] Fill: Org-ID, Stage Added = "Stage 2 pending", Date Added = today, Notes = "Confirmed via browser session [date]"
    [ ] Update Status Summary block: decrement "Pending browser confirmation", increment "Active org-IDs"

[ ] If entity is NOT on GotSport:
    [ ] Move row from Section 2 to Section 3 (Deferred / Not-on-GotSport)
    [ ] Fill: Reason = "No GotSport presence confirmed via browser lookup [date]", Alternative Platform = [if FORGE knows]

[ ] Update "Last updated" timestamp in the Status Summary block
```

---

### Step 2: Substitute Org-IDs in Stage 2 Pre-Staged Config (< 15 minutes)

File: `Infrastructure/Stage 2 Org-ID Config — Pre-Staged Additions.md`

```
[ ] For each confirmed org-ID:
    [ ] Find the entity's row (search by entity name)
    [ ] Replace "ORG_ID_PENDING" with the confirmed numeric value
    [ ] Update the "Config Status" column from "pending-browser-lookup" to "confirmed — not yet in live config"

[ ] For any entity confirmed as NOT on GotSport:
    [ ] Mark the row: "REMOVED — entity not on GotSport. Confirmed [date]."

[ ] At the bottom of the doc, add an update log entry:
    "Browser session results received [date/time]. [N] org-IDs confirmed. [N] entities not-on-GotSport."
```

---

### Step 3: Flag Stage 2 Authorization Readiness (< 5 minutes)

After Steps 1–2 complete, assess Stage 2 authorization state:

```
[ ] Count: how many Stage 2 entities now have confirmed org-IDs vs. still pending?
[ ] If ALL 11 entities are resolved (confirmed or not-on-GotSport):
    → Note in briefing: "Browser session complete. Stage 2 config fully populated. [N] active org-IDs ready for Stage 2 authorization gate."
[ ] If some entities remain pending (browser session was partial):
    → Note: "Partial browser session. [N] of 11 entities resolved. Remaining: [list names]. Stage 2 config not yet fully populated."
```

---

### Step 4: File Briefing Update (< 5 minutes)

In the next FORGE briefing, include a dedicated Browser Session section:

```
## Browser Session Results — [date]

Received: [N] org-IDs confirmed, [N] entities not on GotSport.

Confirmed entities:
- [Entity Name]: org-ID [######]
- ...

Not-on-GotSport:
- [Entity Name]: removed from Stage 2 scope

Master Reference updated: ✅
Pre-Staged Config updated: ✅
Stage 2 authorization readiness: [ready / partial — [N] entities pending]
```

---

### Step 5: Notify SENTINEL (in same briefing)

SENTINEL reviews Stage 2 authorization against `Infrastructure/GotSport Org-ID Expansion Stage 2 — Pre-Authorization Criteria.md`. FORGE does not self-authorize Stage 2 — but FORGE's briefing update triggers SENTINEL's review.

```
[ ] Briefing note includes: "SENTINEL: Stage 2 Pre-Authorization Criteria checklist — please review against updated Master Reference."
```

---

## Quality Standard

- All 5 steps complete within 60 minutes of results receipt.
- FORGE does not skip Step 3 even if the update seems obvious — the count and readiness statement is what SENTINEL reads.
- If Presten delivers partial results (fewer than 11 entities), complete Steps 1–4 with whatever is received and note the outstanding count explicitly. Do not wait for the full list before updating.
- The briefing update (Step 4) must name each entity and org-ID — not "several org-IDs received." SENTINEL needs the names.

## Why This Matters

Stage 2 authorization is gated on org-IDs being confirmed and the config being staged. The browser session produces those org-IDs. The post-receipt window is the only moment where FORGE's actions directly translate browser results into Stage 2 readiness. Missing a document update in this window means Stage 2 authorization is delayed by at least one more session. For a project with a May 16 DSS deadline, that delay compounds.

## References

- `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` — Step 1 target
- `Infrastructure/Stage 2 Org-ID Config — Pre-Staged Additions.md` — Step 2 target
- `Infrastructure/GotSport Org-ID Expansion Stage 2 — Pre-Authorization Criteria.md` — Stage 2 gate
- `Infrastructure/GotSport Browser Lookup Session Prep Package.md` — 11 entities this task processes
