---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-27
status: pending
priority: high
due: 2026-04-28
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/GotSport Org-ID Master Reference — April 2026.md"
---

# Task: GotSport Org-ID Master Reference

## Context

Org-ID knowledge is currently scattered across at least 5 documents:
- `Infrastructure/full-gotsport-org-id-list.md` — Stage 1/2 crawl scope
- `Infrastructure/GotSport Browser Lookup Session Prep Package.md` — 11 entities to confirm
- `Infrastructure/GotSport Org-ID Config Edit Reference.md` — how to add confirmed IDs
- `Infrastructure/GotSport Org-ID Expansion Stage 2 — Pre-Authorization Criteria.md` — Stage 2 scope
- `Infrastructure/Crawl Expansion Config Template.md` — config format

No single document answers the question SENTINEL asks most: "What is the current status of every org_id we care about?" This causes SENTINEL to synthesize status across 5 docs every briefing cycle. When FORGE closes a browser session task, the update is local to that task file and the prep package — the master picture never gets updated.

This task: write the master reference. Going forward, when any org_id status changes, FORGE updates this document.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/GotSport Org-ID Master Reference — April 2026.md`

---

### Section 1: Active Org-IDs (Currently in Crawl Config)

Pull from `Infrastructure/full-gotsport-org-id-list.md` and the crawl config documentation. For each org_id currently in the live crawl config:

| Org-ID | Entity Name | Type (State Assoc / League / Club) | Stage Added | Date Added | Notes |
|--------|-------------|-------------------------------------|-------------|------------|-------|

Include all org_ids confirmed as active, even if some are from Stage 1 (TX cohort, pending execution).

---

### Section 2: Pending Confirmation (Browser Session Required)

Pull from `Infrastructure/GotSport Browser Lookup Session Prep Package.md`. For each of the 11 entities that require browser lookup:

| Entity | Type | GotSport Presence | Org-ID | Status | Config Section (when confirmed) |
|--------|------|------------------|--------|--------|--------------------------------|

Status options: `pending-browser-lookup` / `confirmed-[org_id]` / `not-on-gotsport` / `deferred`

When Presten completes the browser session and FORGE receives org IDs, FORGE updates this table immediately — before filing the next briefing.

---

### Section 3: Deferred / Not-on-GotSport

Any entity FORGE investigated but determined is not on GotSport (different platform, no digital presence, or org_id lookup returned no results). Include reason for deferral.

| Entity | Reason | Alternative Platform (if known) | Revisit Date |
|--------|--------|---------------------------------|-------------|

---

### Section 4: Status Summary

A machine-readable summary block at the top of the document (update this section whenever any row changes):

```
Last updated: [date/time of most recent FORGE edit]
Active org-IDs in crawl config: [N]
Pending browser confirmation: [N]
Not-on-GotSport / deferred: [N]
Total entities tracked: [N]
```

---

### Section 5: Ownership and Update Protocol

One paragraph. Who updates this document, when, and how to reflect a status change:
- When Presten completes a browser session and reports org IDs → FORGE updates Section 2 rows from `pending` to `confirmed-[org_id]` and moves them to Section 1
- When FORGE adds an org_id to the crawl config → FORGE updates Section 1 with date added
- When an entity is confirmed as not on GotSport → FORGE moves it to Section 3 with reason
- SENTINEL reviews this document in every briefing as the coverage status source of truth

---

## Quality Standard

- All 11 browser-session entities must appear in Section 2 at document creation time. Rows should not be left blank — fill in all known fields from the prep package.
- All confirmed-active org_ids from `full-gotsport-org-id-list.md` must appear in Section 1.
- The Status Summary block must be accurate at the time of writing. Do not leave it empty.
- The document is designed to be updated incrementally. Write it so future FORGE updates are additive (new row, updated status cell) not reconstructive.

## Why This Matters

SENTINEL currently audits coverage coverage from 5 separate documents every cycle. This reference consolidates that into one lookup. The more critical function: when the browser session finally runs, there is no natural place for FORGE to record all 11 org-ID results in a structured way that SENTINEL can review at a glance. This document IS that place. Writing it now (before the browser session) means it's ready to be populated the moment results come in.

## References

- `Infrastructure/full-gotsport-org-id-list.md` — Stage 1/2 scope
- `Infrastructure/GotSport Browser Lookup Session Prep Package.md` — 11 entities pending
- `Infrastructure/GotSport Org-ID Config Edit Reference.md` — config format
- `Infrastructure/GotSport Org-ID Expansion Stage 2 — Pre-Authorization Criteria.md` — Stage 2 gate
