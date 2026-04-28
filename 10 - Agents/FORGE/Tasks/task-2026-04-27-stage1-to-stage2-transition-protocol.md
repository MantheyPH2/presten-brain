---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-27
status: open
priority: high
due: 2026-04-29
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/Stage 1 to Stage 2 — Transition Protocol.md"
---

# Task: Stage 1 to Stage 2 — Transition Protocol

## Context

Stage 1 (TYSA TX crawl) could run any time from tonight through April 30. When it does, FORGE has 30 minutes to file the Stage 1 Post-Run Verification Statement and open the Stage 2 Section 1 gate for SENTINEL review. What happens after SENTINEL approves Stage 1 is not documented.

The Stage 2 Authorization Criteria document (6 sections) defines what must be true before Stage 2 is authorized. The Stage 2 Config Pre-Staged Additions document lists the new org-IDs to activate. The Org-ID Master Reference tracks pending vs. confirmed entities. But nowhere is the answer to: **what does FORGE actually do, step by step, in the session immediately after SENTINEL says "Stage 2 authorized"?**

This creates a gap: SENTINEL authorizes, then FORGE needs time to figure out what to change. Under active pipeline conditions (May 1+), FORGE's interpretation latency adds a day of Stage 2 delay. Writing this protocol now means FORGE executes Stage 2 activation within one session of authorization.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/Stage 1 to Stage 2 — Transition Protocol.md`

This is a linear protocol. FORGE follows it from top to bottom the moment SENTINEL issues Stage 2 authorization.

---

### Pre-Transition Gate

FORGE does not begin this protocol until SENTINEL files Stage 2 authorization. FORGE monitors for one of two authorization signals:

1. SENTINEL briefing contains: "Stage 2 authorized — proceed with org-ID config activation"
2. `Infrastructure/Stage 2 Expansion — Authorization Criteria.md` Section 6 (SENTINEL disposition) is filled with verdict: AUTHORIZED

If neither signal is present: do not modify any config. Do not activate any new org-ID.

---

### Step 1: Confirm Stage 2 Scope (< 5 minutes)

Open `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` Section 2 (Pending Confirmation).

For each entity in Section 2:
- Status `confirmed-[org_id]` → proceed to activation in Step 2
- Status `pending-browser-lookup` → **do not activate** — wait for browser session results
- Status `not-on-gotsport` → skip (no action)

Build a list: "Stage 2 entities ready to activate: [N] confirmed, [M] pending." File this list in the briefing under "Stage 2 Transition — Activation Scope."

If 0 entities are confirmed: file a "Stage 2 authorized but no confirmed org-IDs ready — awaiting browser session results" note in SENTINEL queue and stop.

---

### Step 2: Config Activation (< 15 minutes)

Using `Infrastructure/GotSport Org-ID Config Edit Reference.md` as the procedure guide:

For each confirmed org-ID from Step 1:
```
[ ] Open run-daily.sh (or equivalent config file per the Config Edit Reference)
[ ] Add org_id entry under the correct config section (state association / league / club)
[ ] Confirm no duplicate entry (reference Stage 2 Org-ID Duplicate Cross-Check)
[ ] Verify syntax matches existing entries — no trailing commas, correct array format
```

Do not add any org-ID not confirmed (status `pending-browser-lookup`). Note in briefing: "N org-IDs activated. M org-IDs remain pending browser session confirmation and will be activated in Stage 2 Phase 2."

---

### Step 3: Org-ID Master Reference Update (< 10 minutes)

```
[ ] Open: Infrastructure/GotSport Org-ID Master Reference — April 2026.md
[ ] For each org-ID activated in Step 2:
    - Move row from Section 2 to Section 1
    - Set Stage Added = "Stage 2"
    - Set Date Added = [today's date]
[ ] Update Section 4 Status Summary block:
    - Active org-IDs in crawl config: [updated count]
    - Pending browser confirmation: [updated count]
    - Last updated: [date/time]
```

---

### Step 4: Stage 2 Authorization Criteria — Section 6 Update

```
[ ] Open: Infrastructure/Stage 2 Expansion — Authorization Criteria.md
[ ] If Section 6 (SENTINEL disposition) is not yet filled by SENTINEL: leave it
[ ] If SENTINEL has filled Section 6 with AUTHORIZED verdict:
    - Add a FORGE acknowledgment line below SENTINEL's entry:
      "FORGE acknowledged Stage 2 authorization: [date/time]. Config activation completed: [date/time]. Org-IDs activated: [count]. Pending browser session: [count]."
```

---

### Step 5: Verify Next Pipeline Run Will Include Stage 2 Org-IDs (< 5 minutes)

```
[ ] Confirm the config file edit will be picked up by the next scheduled run
[ ] If pipeline runs daily at a set time: confirm the run has not already started today; if so, Stage 2 org-IDs activate on the following run
[ ] Note in briefing: "Stage 2 org-IDs will be active starting: [next scheduled run date/time]"
```

---

### Step 6: Briefing Entry

File a structured Stage 2 Transition entry in the next FORGE briefing:

```
## Stage 2 Transition — Execution Log

Authorization received: [date/time]
Config activation completed: [date/time]
Org-IDs activated: [N] (list: [org-id: entity name, ...])
Org-IDs deferred (pending browser session): [M] (list: [entity names])
Org-ID Master Reference updated: YES
First pipeline run including Stage 2 org-IDs: [scheduled date/time]
Any issues: [none / description]
SENTINEL action required: [none / "Phase 2 browser session results needed to activate remaining M entities"]
```

---

### Phase 2 Extension (If Browser Session Results Arrive After Stage 2 Activation)

When Presten provides the remaining pending org-IDs (entities currently `pending-browser-lookup`):

Repeat Steps 1–6 for the newly confirmed entities. Phase 2 activation follows the same protocol; it is not a separate authorization event. SENTINEL does not need to re-authorize — the Stage 2 authorization covers all confirmed org-IDs including late-arriving browser session results.

---

## Quality Standard

- The Pre-Transition Gate is non-optional. FORGE does not touch the config without explicit SENTINEL authorization.
- Zero pending-browser-lookup entities may be activated. Only `confirmed-[org_id]` status entities.
- Every step ends with a ✅ in the briefing entry or a documented reason for incomplete execution.
- The briefing entry (Step 6) must include the list of activated org-IDs. SENTINEL needs entity-level confirmation, not just a count.

## Why This Matters

Stage 2 authorization from SENTINEL is the most time-sensitive action in the current pipeline expansion plan. Once SENTINEL authorizes, every day FORGE takes to interpret what to do is a day without Stage 2 data. The TYSA TX crawl (Stage 1) could run as early as tonight; if it passes, SENTINEL could authorize Stage 2 the same session. Without this protocol, FORGE's first action post-authorization is to read 5 documents and synthesize an action plan. With it, FORGE opens one document and works through a 35-minute checklist.

## References

- `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` — primary status tracker
- `Infrastructure/Stage 2 Expansion — Authorization Criteria.md` — Section 6 authorization signal
- `Infrastructure/GotSport Org-ID Config Edit Reference.md` — config edit procedure
- `Infrastructure/Stage 2 Org-ID Config — Pre-Staged Additions.md` — org-IDs ready for activation
- `Infrastructure/Stage 2 Org-ID Duplicate Cross-Check.md` — confirms no conflicts
- `task-2026-04-27-stage1-post-run-verification-protocol.md` — Stage 1 verification that precedes Stage 2 authorization
