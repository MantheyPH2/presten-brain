---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-25
status: pending
priority: medium
due: 2026-04-28
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/Stage 2 Config Pre-Population — Pending Org-IDs.md"
---

# Task: Stage 2 Config Pre-Population Document

## Context

The GotSport Org-ID Master Reference tracks the status of 11 entities pending browser session confirmation. The Config Edit Reference explains how to add an org-ID to the crawl config. What does not exist: a document with the complete config entries for all 11 pending entities already pre-written, with `[TBD]` placeholders in the org-ID field.

The current flow when the browser session produces an org-ID:
1. Presten sends FORGE the org-ID for entity X
2. FORGE opens the Config Edit Reference to recall the format
3. FORGE writes the config entry from scratch
4. FORGE confirms format against existing entries
5. FORGE adds to config and confirms with Presten

The desired flow:
1. Presten sends FORGE "Entity X = org-ID 98765"
2. FORGE opens this document, finds Entity X's pre-written entry, replaces `[TBD-ORG-ID]` with 98765
3. FORGE pastes into config in under 60 seconds

The browser session may produce 11 org-IDs at once. Under that load, writing config entries from scratch increases error risk. Pre-writing entries now, when there is no time pressure, reduces that risk to zero.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/Stage 2 Config Pre-Population — Pending Org-IDs.md`

---

### Section 1: Config Entry Format Reference

One short block at the top of the document. Pull the exact config entry format from `Infrastructure/GotSport Org-ID Config Edit Reference.md`. This is the canonical format; every entry in Section 2 follows it exactly.

```
[Example format from Config Edit Reference — FORGE copies the exact template here]
```

---

### Section 2: Pre-Written Config Entries — All 11 Pending Entities

For every entity in Section 2 of the Org-ID Master Reference (`status: pending-browser-lookup`), write a complete config entry in the canonical format. Use `[TBD-ORG-ID]` wherever the org-ID field would go. All other fields (name, type, label, description) fill from known data in the prep package.

Layout for each entity:

```
--- ENTITY: [Entity Name] ---
Status: pending-browser-lookup
Config section: [which section of the crawl config this belongs in]
Pre-written entry:

[complete config entry with [TBD-ORG-ID] placeholder]

When confirmed: Replace [TBD-ORG-ID] with actual org-ID. Remove this block header. Paste into config.
```

If an entity's config section depends on what type the browser session confirms (e.g., if a club could be either a state association entry or a league entry), write both variants and label them "Use if confirmed as [type A]" / "Use if confirmed as [type B]."

---

### Section 3: Entities Confirmed Not-on-GotSport

Entities from the prep package that FORGE has already determined are not on GotSport. These do not get config entries — but they are listed here so the document is a complete accounting of all 11 entities.

| Entity | Status | Alternative Platform (if known) |
|--------|--------|---------------------------------|
| | not-on-gotsport | |

If none have been confirmed as not-on-GotSport yet, leave this section with a single row: "None confirmed yet — pending browser session."

---

### Section 4: Activation Protocol

Short paragraph. When Presten sends browser session results:
1. FORGE opens this document
2. Finds the entity row
3. Replaces `[TBD-ORG-ID]` with the confirmed org-ID
4. Removes the "Pre-written entry" block header
5. Pastes the cleaned entry into the appropriate crawl config section
6. Updates the Org-ID Master Reference: moves entity from Section 2 to Section 1, status → `confirmed-[org_id]`
7. Notes the update in next FORGE briefing

FORGE completes this process for all confirmed entities before filing the next briefing after the browser session. No entity confirmed during the browser session should remain in `pending-browser-lookup` status after the next FORGE briefing.

---

### Section 5: Update Log

Running table. Each time an entity is confirmed and its entry is activated:

| Date | Entity | Org-ID Confirmed | Config Section Updated | Master Reference Updated |
|------|--------|-----------------|------------------------|--------------------------|

Start empty. Populate as confirmations arrive.

---

## Quality Standard

- All 11 entities from the Org-ID Master Reference Section 2 must appear in Section 2 of this document at creation time. Do not leave any pending entity out.
- The config entry format must exactly match the canonical format from the Config Edit Reference. FORGE should copy one confirmed-active entry from the Master Reference Section 1 and use it as the template, not write from memory.
- Every entry must be complete except for `[TBD-ORG-ID]`. No other field should be `[TBD]` — if a field is unknown (e.g., a league's sub-tier), note "unknown — confirm during browser session" but still include the field.
- The activation protocol in Section 4 must be one sequence FORGE can follow in under 2 minutes per entity.

## Why This Matters

The browser session is the most overdue Presten action item (2+ days blocked, Stage 2 gate 0/6). When it finally runs, FORGE should be able to translate every confirmed org-ID into a config update in under 60 seconds per entity. The only way to guarantee that speed is to write the config entries now, before the session. Post-session, FORGE's attention will be split between updating the Master Reference, updating the Stage 2 gate, and briefing SENTINEL — all in a compressed window. This document makes the config update step trivial.

## References

- `Infrastructure/GotSport Org-ID Config Edit Reference.md` — canonical config format
- `Infrastructure/GotSport Browser Lookup Session Prep Package.md` — the 11 entities and their known details
- `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` — Section 2 source of truth for pending entities; Section 1 receives confirmed entities
- `Infrastructure/GotSport Org-ID Expansion Stage 2 — Pre-Authorization Criteria.md` — Stage 2 gate conditions that these confirmations unlock
