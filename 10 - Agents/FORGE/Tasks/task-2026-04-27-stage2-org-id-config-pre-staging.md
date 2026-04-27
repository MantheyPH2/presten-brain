---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-27
status: pending
priority: medium
due: 2026-04-30
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/Stage 2 Org-ID Config — Pre-Staged Additions.md"
---

# Task: Stage 2 Org-ID Config — Pre-Staged Additions

## Context

The GotSport browser session has 11 entities pending confirmation. When Presten completes the session, FORGE's job is to update the crawl config with the confirmed org-IDs. The process currently requires FORGE to:
1. Receive the org-IDs from Presten
2. Open the Org-ID Config Edit Reference
3. Construct the correct config entries for each entity
4. Add them to the config file

Steps 2 and 3 can be done now. The config entry structure is known (from `Infrastructure/GotSport Org-ID Config Edit Reference.md`). The 11 entities are known (from `Infrastructure/GotSport Browser Lookup Session Prep Package.md`). The only unknown is the numeric org-ID for each entity.

This task: pre-stage all 11 config entries with `ORG_ID_PENDING` placeholders. When Presten returns from the browser session, FORGE fills in the actual numbers and the config is immediately deployable. Turn-around time drops from 30–45 minutes to under 5 minutes.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/Stage 2 Org-ID Config — Pre-Staged Additions.md`

---

### Section 1: Pre-Staging Instructions

Brief note explaining how to use this document:

- Presten completes browser session → reports org-IDs to FORGE
- FORGE opens this file, replaces each `ORG_ID_PENDING` with the confirmed number
- FORGE copies the completed config block into the live crawl config
- FORGE updates the Org-ID Master Reference with confirmed status
- FORGE files in next briefing: "Stage 2 config additions applied. N entities added. Confirmed org-IDs: [list]."

---

### Section 2: Pre-Staged Config Entries

Pull the 11 entities from `Infrastructure/GotSport Browser Lookup Session Prep Package.md`. For each entity, pre-write the config addition in the correct format from `Infrastructure/GotSport Org-ID Config Edit Reference.md`:

For each of the 11 entities:

```
# [Entity Name] — Stage 2 Addition
# Type: [State Assoc / League / Club]
# Browser session date: [leave blank — fill when confirmed]
# Confirmed by: Presten
{
  "org_id": ORG_ID_PENDING,
  "name": "[Entity Name]",
  "type": "[type]",
  "stage": 2,
  "added": "2026-04-[DD]"
}
```

Leave `ORG_ID_PENDING` in place for each entry. Do not guess org-IDs.

Also pre-stage the 4 additional entities from the VA/CO/AZ research (`task-2026-04-27-va-co-az-gotsport-org-id-research.md`) and TYSA confirmation, if those are also pending browser confirmation. If they are, include them in a Section 2B: "Additional Pending Confirmations."

---

### Section 3: Not-on-GotSport Handling

For each entity where FORGE already knows (from prior research) that the entity is not on GotSport, include a pre-written record:

```
# [Entity Name] — Not on GotSport
# Confirmed: [date of FORGE research note]
# Reason: [operates on different platform / no digital game records / org_id lookup returned no results]
# Alternative: [platform name, if known]
# No config entry needed.
```

This avoids a "why isn't [X] in the config?" question later.

---

### Section 4: Post-Session Completion Checklist

After Presten reports org-IDs and FORGE fills in the placeholders:

- [ ] All `ORG_ID_PENDING` replaced with actual numbers
- [ ] Config block added to live crawl config (reference `Infrastructure/GotSport Org-ID Config Edit Reference.md` for exact file path and format)
- [ ] Org-ID Master Reference updated: all confirmed entities moved from Section 2 (Pending) to Section 1 (Active)
- [ ] Not-on-GotSport entities moved to Master Reference Section 3
- [ ] Stage 2 Authorization Trigger Document: check if session completion triggers the Stage 2 activation gate (per `Infrastructure/Stage 2 Authorization Trigger Document.md`)
- [ ] Next briefing filed with: entity count added, org-IDs confirmed, any entities not found

---

## Why This Matters

The browser session is a Presten time commitment: ≤90 minutes at system.gotsport.com. When Presten reports back, FORGE's response time matters — the config update should be immediate, not a 30-minute FORGE task. Pre-staging also catches any entity-to-config-format issues now, before the session, when there is no time pressure. If the config entry for "TYSA" requires a different structure than "Colorado Soccer Association," FORGE discovers that now, not when Presten is waiting for the config update.

## Definition of Done

- All 11 browser-session entities have pre-staged config entries with `ORG_ID_PENDING`
- Any additional VA/CO/AZ/TYSA entities are included
- Any already-confirmed not-on-GotSport entities have pre-written Section 3 records
- The post-session completion checklist is present and accurate
- The document can be used immediately when Presten reports org-IDs — no additional design needed

## References

- `Infrastructure/GotSport Browser Lookup Session Prep Package.md` — 11 entities requiring confirmation
- `Infrastructure/GotSport Org-ID Config Edit Reference.md` — config entry format
- `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` — master status document FORGE updates post-session
- `Infrastructure/Stage 2 Authorization Trigger Document.md` — gate this session may trigger
- `task-2026-04-27-va-co-az-gotsport-org-id-research.md` — additional entities to include
