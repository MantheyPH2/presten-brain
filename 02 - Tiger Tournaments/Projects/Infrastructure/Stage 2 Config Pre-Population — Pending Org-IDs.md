---
title: Stage 2 Config Pre-Population — Pending Org-IDs
tags: [infrastructure, gotsport, org-ids, stage2, config, pipeline, evo-draw]
created: 2026-04-25
updated: 2026-04-25
author: FORGE
status: active — entries ready; org-IDs pending browser session
companion: "Infrastructure/GotSport Org-ID Config Edit Reference.md"
---

# Stage 2 Config Pre-Population — Pending Org-IDs

> **Purpose:** Pre-written config entries for all 11 entities pending browser lookup. When Presten sends confirmed org IDs, FORGE replaces `[TBD-ORG-ID]` with the actual value and pastes into `crawl-gotsport-api-parallel.js`. Target: one config update per entity in under 60 seconds.

---

## Section 1: Config Entry Format Reference

Canonical format from `Infrastructure/GotSport Org-ID Config Edit Reference.md`. Every entry in Section 2 follows this exactly.

**File:** `crawl-gotsport-api-parallel.js` (project root)
**Language:** JavaScript — bare integer array entries with inline comments

```javascript
// [LEAGUE / ASSOCIATION NAME] — added [YYYY-MM-DD] by Presten
REPLACE_WITH_ORG_ID, // [Entity Name] — [region description]
```

- Entry is a bare integer (no quotes, no semicolons mid-array)
- Trailing comma required unless it is the last element in the array
- Inline comment is for human reference only — not parsed by the crawler
- After adding: verify syntax with `node -c crawl-gotsport-api-parallel.js && echo "Syntax OK"`

---

## Section 2: Pre-Written Config Entries — All 11 Pending Entities

---

### Group A: USYS State Associations

---

--- ENTITY: Cal North Soccer (USYS CA-NorCal) ---
Status: pending-browser-lookup
Config section: `// USYS State Associations`
Pre-written entry:

```javascript
// Cal North Soccer (USYS CA — NorCal) — added [YYYY-MM-DD] by Presten
[TBD-ORG-ID], // Cal North Soccer — California NorCal statewide
```

When confirmed: Replace `[TBD-ORG-ID]` with actual org-ID. Remove this block header. Paste into config under `// USYS State Associations`.

---

--- ENTITY: Cal South Soccer (USYS CA-SoCal) ---
Status: pending-browser-lookup
Config section: `// USYS State Associations`
Pre-written entry:

```javascript
// Cal South Soccer (USYS CA — SoCal) — added [YYYY-MM-DD] by Presten
[TBD-ORG-ID], // Cal South Soccer — California SoCal statewide
```

When confirmed: Replace `[TBD-ORG-ID]` with actual org-ID. Remove this block header. Paste into config under `// USYS State Associations`.

> [!note] CA note: Two separate entries
> California requires two entries — one for Cal North (NorCal) and one for Cal South (SoCal). These are distinct USYS affiliates with separate GotSport org IDs. Add both.

---

--- ENTITY: Florida Youth Soccer Association (FYSA) ---
Status: pending-browser-lookup
Config section: `// USYS State Associations`
Pre-written entry:

```javascript
// Florida Youth Soccer Association (USYS FL) — added [YYYY-MM-DD] by Presten
[TBD-ORG-ID], // FYSA — Florida statewide
```

When confirmed: Replace `[TBD-ORG-ID]` with actual org-ID. Remove this block header. Paste into config under `// USYS State Associations`.

---

--- ENTITY: Eastern NY Youth Soccer Association (ENYYSA) ---
Status: pending-browser-lookup
Config section: `// USYS State Associations`
Pre-written entry:

```javascript
// Eastern NY Youth Soccer Association (USYS NY) — added [YYYY-MM-DD] by Presten
[TBD-ORG-ID], // ENYYSA — New York statewide
```

When confirmed: Replace `[TBD-ORG-ID]` with actual org-ID. Remove this block header. Paste into config under `// USYS State Associations`.

---

--- ENTITY: NJ Youth Soccer Association (NJYSA) ---
Status: pending-browser-lookup
Config section: `// USYS State Associations`
Pre-written entry:

```javascript
// NJ Youth Soccer Association (USYS NJ) — added [YYYY-MM-DD] by Presten
[TBD-ORG-ID], // NJYSA — New Jersey statewide
```

When confirmed: Replace `[TBD-ORG-ID]` with actual org-ID. Remove this block header. Paste into config under `// USYS State Associations`.

---

--- ENTITY: Washington Youth Soccer Association (WSYSA) ---
Status: pending-browser-lookup
Config section: `// USYS State Associations`
Pre-written entry:

```javascript
// Washington Youth Soccer Association (USYS WA) — added [YYYY-MM-DD] by Presten
[TBD-ORG-ID], // WSYSA — Washington statewide
```

When confirmed: Replace `[TBD-ORG-ID]` with actual org-ID. Remove this block header. Paste into config under `// USYS State Associations`.

---

--- ENTITY: Eastern PA Youth Soccer Association (EPYSA) ---
Status: pending-browser-lookup
Config section: `// USYS State Associations`
Pre-written entry:

```javascript
// Eastern PA Youth Soccer Association (USYS PA — East) — added [YYYY-MM-DD] by Presten
[TBD-ORG-ID], // EPYSA — Eastern Pennsylvania (Philadelphia metro + East PA)
```

When confirmed: Replace `[TBD-ORG-ID]` with actual org-ID. Remove this block header. Paste into config under `// USYS State Associations`.

> [!note] PA note: East only for Stage 2
> EPYSA covers Eastern PA (Philadelphia metro). Western PA (WPYSA/Pittsburgh) is secondary priority — add separately if browser session returns that org-ID. Do not combine into one entry.

---

### Group B: League Entities

---

--- ENTITY: USL Academy ---
Status: pending-browser-lookup
Config section: `// USL Academy`
Pre-written entry:

```javascript
// USL Academy — added [YYYY-MM-DD] by Presten
[TBD-ORG-ID], // USL Academy — national
```

When confirmed: Replace `[TBD-ORG-ID]` with actual org-ID. Remove this block header. Create `// USL Academy` section in config if it does not exist. Paste entry beneath.

> [!note] USL Academy is a national league
> Single org_id covers all USL Academy clubs nationally (90+ clubs, U13–U19, professional pathway). Do not attempt to add per-club or per-state sub-orgs unless Presten confirms they are separate GotSport entities.

---

--- ENTITY: EDP Soccer (Eastern Development Program) ---
Status: pending-browser-lookup
Config section: `// EDP Soccer`
Pre-written entry:

```javascript
// EDP Soccer (Eastern Development Program) — added [YYYY-MM-DD] by Presten
[TBD-ORG-ID], // EDP Soccer — Northeast regional (NJ, NY, PA, CT, MA, MD, RI, DE, VA, VT)
```

When confirmed: Replace `[TBD-ORG-ID]` with actual org-ID. Remove this block header. Create `// EDP Soccer` section in config if it does not exist. Paste entry beneath.

> [!note] EDP dedup check required
> EDP has ~150+ clubs in the Northeast. Some EDP games may already be in the DB via ranked-team discovery. Run the pre-flight dedup query (Config Edit Reference Section 4A) before adding EDP to config. If existing_games > 0: flag to FORGE before proceeding. Expected: some EDP games already present; the org-level crawl will fill unranked divisions not yet captured.

---

--- ENTITY: Elite 64 ---
Status: pending-browser-lookup (conditional GO)
Config section: `// Elite 64`
Pre-written entry (use if confirmed as dedicated GotSport org):

```javascript
// Elite 64 — added [YYYY-MM-DD] by Presten
[TBD-ORG-ID], // Elite 64 — national (US Club Soccer affiliated)
```

Pre-written entry (use if confirmed as sub-org under US Club Soccer):

```javascript
// Elite 64 (via US Club Soccer) — added [YYYY-MM-DD] by Presten
[TBD-ORG-ID], // Elite 64 — national showcase/championship events
```

Use whichever variant the browser session confirms. If Elite 64 events appear under a parent US Club Soccer org rather than a dedicated org, use the "via US Club Soccer" variant.

When confirmed: Replace `[TBD-ORG-ID]` with actual org-ID. Remove all block headers and the unused variant. Create `// Elite 64` section in config. Paste entry beneath.

> [!warning] Elite 64 is conditional GO
> Only add Elite 64 if the browser session confirms a GotSport org_id. If Elite 64 is not on GotSport, move to Section 3 of the Org-ID Master Reference and flag in next briefing. Do not add a placeholder entry to the config.

---

--- ENTITY: NAL (North American League) ---
Status: pending-browser-lookup (deferred — platform unconfirmed)
Config section: N/A until platform confirmed

**No config entry pre-written.** NAL's result platform is unknown as of 2026-04-25. If the browser session finds NAL on GotSport, FORGE will write a config entry using the League Entity template from the Config Edit Reference Section 2 at that time.

**If browser session finds NAL on GotSport:** FORGE drafts the entry and adds it here before the next briefing.

**If browser session confirms NAL is NOT on GotSport:** Move NAL to Section 3 of the Org-ID Master Reference with the identified platform. Flag in next briefing. No config entry required.

---

## Section 3: Entities Confirmed Not-on-GotSport

| Entity | Status | Alternative Platform (if known) |
|--------|--------|---------------------------------|
| None confirmed yet — pending browser session. | | |

> Once the browser session runs, FORGE updates this section for any entity where GotSport search returns no results after 2–3 attempts.

---

## Section 4: Activation Protocol

When Presten sends browser session results:

1. FORGE opens this document
2. Locates the entity row in Section 2
3. Replaces `[TBD-ORG-ID]` with the confirmed numeric org-ID
4. Removes the "--- ENTITY: ---" block header and all notes from that entry
5. Pastes the cleaned entry into the appropriate section of `crawl-gotsport-api-parallel.js`
6. Verifies syntax: `node -c crawl-gotsport-api-parallel.js && echo "Syntax OK"`
7. Updates the Org-ID Master Reference (`Infrastructure/GotSport Org-ID Master Reference — April 2026.md`):
   - Moves entity from Section 2 to Section 1 with confirmed org-ID value and date added
   - Updates Section 4 Status Summary counts
8. Records the update in Section 5 (Update Log) below
9. Notes all confirmed entities in the next FORGE briefing

**Target speed:** One entity from confirmed org-ID to config update in under 60 seconds.

**Deadline:** All entities confirmed during a browser session must be processed and reflected in the Org-ID Master Reference before FORGE files the next briefing after that session. No entity confirmed during the browser session should remain in `pending-browser-lookup` status after the next FORGE briefing.

---

## Section 5: Update Log

| Date | Entity | Org-ID Confirmed | Config Section Updated | Master Reference Updated |
|------|--------|-----------------|------------------------|--------------------------|
| | | | | |

*Start empty. FORGE populates as confirmations arrive.*

---

## References

- `Infrastructure/GotSport Org-ID Config Edit Reference.md` — canonical config format and step-by-step add procedure
- `Infrastructure/GotSport Browser Lookup Session Prep Package.md` — the 11 entities and their lookup details
- `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` — Section 2 is the source of truth for pending entities; Section 1 receives confirmed entities
- `Infrastructure/GotSport Org-ID Expansion Stage 2 — Pre-Authorization Criteria.md` — Stage 2 gate conditions unlocked by these confirmations

*FORGE — 2026-04-25*
