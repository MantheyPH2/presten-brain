---
title: GotSport Org-ID Master Reference — April 2026
tags: [infrastructure, gotsport, org-ids, usys, pipeline, config, evo-draw]
created: 2026-04-25
updated: 2026-04-25
author: FORGE
status: active — update whenever any org-ID status changes
---

# GotSport Org-ID Master Reference — April 2026

> **Source of truth for all GotSport org-ID status.** SENTINEL reviews this document every briefing cycle. FORGE updates it whenever any row changes — before filing the next briefing.

---

## Section 4: Status Summary

```
Last updated: 2026-04-25
Active org-IDs in crawl config: 0 (TX crawl config in place; Stage 1 run pending Presten execution)
Pending browser confirmation: 11
Not-on-GotSport / deferred: 5
Total entities tracked: 16
```

---

## Section 1: Active Org-IDs (Currently in Crawl Config)

Org-IDs confirmed in the live crawl config. TX is the Stage 1 pilot cohort — config was set up but the first run has not yet executed. All other entries will be added after the browser lookup session confirms their org IDs.

| Org-ID | Entity Name | Type | Stage Added | Date Added | Notes |
|--------|-------------|------|-------------|------------|-------|
| pending-lookup | Texas Youth Soccer Association (TYSA) | State Association | Stage 1 pilot | 2026-04-24 config prep | TX pilot crawl not yet executed. Stage 1 run pending. Config entry in `crawl-gotsport-api-parallel.js` — org_id value requires browser confirmation before first run. |

> [!warning] No org IDs are confirmed active yet
> Stage 1 (TX test crawl) has not executed as of 2026-04-25. The first confirmed org IDs will be populated here after the browser lookup session + Stage 1 run. When a run completes with confirmed org IDs, move those rows here and fill in the actual org_id values.

---

## Section 2: Pending Confirmation (Browser Session Required)

All 11 entities from the `Infrastructure/GotSport Browser Lookup Session Prep Package.md` session.

**Session URL:** `system.gotsport.com` — org_id extracted from URL parameter `?org_id=XXXX` or `?OrgId=XXXX`

| Entity | Type | GotSport Presence | Org-ID | Status | Config Section (when confirmed) |
|--------|------|------------------|--------|--------|-------------------------------|
| USL Academy | National League | High probability — USL Soccer is a GotSport org | null | pending-browser-lookup | `// USL Academy` |
| EDP Soccer (Eastern Development Program) | Regional League | High probability — ~150+ clubs, Northeast | null | pending-browser-lookup | `// EDP Soccer` |
| Elite 64 | National League | Conditional — likely via US Club Soccer affiliation | null | pending-browser-lookup | `// Elite 64` |
| NAL (North American League) | National League | Unknown platform — GotSport unconfirmed | null | pending-browser-lookup (if on GotSport) | Deferred until platform confirmed |
| Cal North Soccer (USYS CA-NorCal) | USYS State Assoc | Yes — confirmed GotSport platform | null | pending-browser-lookup | `// USYS State Associations` |
| Cal South Soccer (USYS CA-SoCal) | USYS State Assoc | Yes — confirmed GotSport platform | null | pending-browser-lookup | `// USYS State Associations` |
| Florida Youth Soccer Assoc (FYSA) | USYS State Assoc | Yes — confirmed GotSport platform | null | pending-browser-lookup | `// USYS State Associations` |
| Eastern NY Youth Soccer (ENYYSA) | USYS State Assoc | Yes — confirmed GotSport platform | null | pending-browser-lookup | `// USYS State Associations` |
| NJ Youth Soccer Assoc (NJYSA) | USYS State Assoc | Yes — confirmed GotSport platform | null | pending-browser-lookup | `// USYS State Associations` |
| Washington Youth Soccer (WSYSA) | USYS State Assoc | Yes — confirmed GotSport platform | null | pending-browser-lookup | `// USYS State Associations` |
| Eastern PA Youth Soccer (EPYSA) | USYS State Assoc | Yes (likely) — large metro, confirmed GotSport region | null | pending-browser-lookup | `// USYS State Associations` |

**How to update this table after browser session:**
- Change status from `pending-browser-lookup` to `confirmed-[org_id]` for found entities
- Change status to `not-on-gotsport` if not found after 2–3 search attempts
- Move confirmed entries to Section 1 and populate org_id in that table
- NAL: if found on GotSport → move to Section 1. If not found → move to Section 3 with platform identified.

---

## Section 3: Deferred / Not-on-GotSport

Entities FORGE investigated and confirmed are not on GotSport (different platform, no digital results presence, or GotSport org_id lookup returned no results), plus entities with unknown platform status that are out of scope pre-DSS.

| Entity | Reason | Alternative Platform (if known) | Revisit Date |
|--------|--------|---------------------------------|-------------|
| Illinois Youth Soccer (IYSA) | Confirmed not on GotSport — Midwest Affinity adoption pattern | Affinity Soccer | Post-DSS |
| Michigan Youth Soccer (MYSA) | Likely Affinity Soccer — browser confirmation pending; deprioritized | Affinity Soccer (likely) | Post-DSS |
| Minnesota Youth Soccer (MSYSA) | Likely Affinity Soccer — browser confirmation pending; deprioritized | Affinity Soccer (likely) | Post-DSS |
| Indiana Youth Soccer (IYSA) | Low volume state; likely Affinity Soccer; not worth pre-DSS investment | Affinity Soccer (likely) | Post-DSS, low priority |
| Wisconsin Youth Soccer (WYSA) | Low volume state; likely Affinity Soccer; not worth pre-DSS investment | Affinity Soccer (likely) | Post-DSS, low priority |

> [!note] NAL status
> NAL is currently in Section 2 (pending browser lookup). If the browser session confirms NAL is not on GotSport, move NAL here with the identified platform (if found) or "no accessible results data" (if none found).

---

## Section 5: Ownership and Update Protocol

**Who updates this document:** FORGE, on the following triggers:

| Trigger | Update action |
|---------|--------------|
| Presten completes browser lookup session and reports results | Update Section 2 rows from `pending-browser-lookup` to `confirmed-[org_id]` or `not-on-gotsport`. Move confirmed entries to Section 1. Move not-on-GotSport entries to Section 3. |
| FORGE adds an org_id to the crawl config | Update Section 1 with date added and confirm org_id value is no longer `null`. |
| An entity is confirmed as not on GotSport | Move from Section 2 to Section 3 with reason and alternative platform if known. |
| A previously confirmed org_id is removed from config (decommission) | Remove from Section 1, add note to Section 3 with reason and date removed. |

**SENTINEL review:** SENTINEL checks the Section 4 Status Summary at the start of each briefing review. If `Pending browser confirmation` is non-zero and the browser session has not been run: this remains the top Presten unblocking action.

**Update cadence:** Section 4 (Status Summary) must be updated every time any other section changes. The timestamp on `Last updated` must reflect the actual edit time.

---

## References

- `Data Sources/GotSport USYS Org-ID Master List.md` — full 50-state org-ID lookup table with platform classification and sub-state splits
- `Infrastructure/GotSport Browser Lookup Session Prep Package.md` — session guide for confirming Section 2 org IDs
- `Infrastructure/GotSport Org-ID Config Edit Reference.md` — how to add confirmed org IDs to `crawl-gotsport-api-parallel.js`
- `Infrastructure/GotSport Org-ID Expansion Stage 2 — Pre-Authorization Criteria.md` — Stage 2 gate conditions
- `Infrastructure/Source Gap Inventory — April 2026.md` — gap status for league-level entities (USL Academy, EDP, Elite 64, NAL)

*FORGE — 2026-04-25*
