---
title: Stage 2 Org-ID Config — Pre-Staged Additions
tags: [infrastructure, gotsport, org-ids, config, stage2, pipeline, evo-draw, forge]
created: 2026-04-26
updated: 2026-04-26
author: FORGE
status: ready — fill ORG_ID_PENDING values when Presten returns browser session results
task: task-2026-04-27-stage2-org-id-config-pre-staging
---

# Stage 2 Org-ID Config — Pre-Staged Additions

> **How to use this document:** Presten completes the browser session → reports org-IDs to FORGE → FORGE opens this file, replaces each `ORG_ID_PENDING` with the confirmed number → FORGE copies completed config blocks into `crawl-gotsport-api-parallel.js` → FORGE updates the Org-ID Master Reference → FORGE files in next briefing: "Stage 2 config additions applied. N entities added. Confirmed org-IDs: [list]."
>
> Every confirmed entity should be processed before the next briefing after the browser session. No entity confirmed during the browser session should remain `pending-browser-lookup` after the next FORGE briefing.

---

## Section 1: Config Entry Format Reference

Canonical format pulled from `Infrastructure/GotSport Org-ID Config Edit Reference.md`. Every entry in Section 2 follows this exactly.

**USYS State Association:**
```javascript
// [STATE NAME] Soccer (USYS [State]) — added YYYY-MM-DD by Presten
ORG_ID_PENDING, // [State Name] — [state abbreviation] statewide
```

**League Entity (national):**
```javascript
// [League Name] — added YYYY-MM-DD by Presten
ORG_ID_PENDING, // [League Name] — national
```

**Placement in file:** All org-IDs are added to `crawl-gotsport-api-parallel.js` in the project root, grouped by league under comment headers. See Section 3 of the Config Edit Reference for step-by-step add instructions, including the syntax check command: `node -c crawl-gotsport-api-parallel.js && echo "Syntax OK"`.

---

## Section 2: Pre-Staged Config Entries — 11 Stage 2 Browser Session Entities

### 2.1 — League Entities (Group A)

---

#### ENTITY: USL Academy
```
# USL Academy — Stage 2 Addition
# Type: National League
# Browser session date: [leave blank — fill when confirmed]
# Confirmed by: Presten
```
```javascript
// USL Academy — added [YYYY-MM-DD] by Presten
ORG_ID_PENDING, // USL Academy — national
```
**Config section:** `// USL Academy` (create header if not present)
**When confirmed:** Replace `ORG_ID_PENDING` with actual org-ID. Remove the `# Type / # Browser session / # Confirmed by` header lines. Paste into config.

---

#### ENTITY: EDP Soccer (Eastern Development Program)
```
# EDP Soccer — Stage 2 Addition
# Type: Regional League (national org, Northeast primary)
# Browser session date: [leave blank — fill when confirmed]
# Confirmed by: Presten
```
```javascript
// EDP Soccer (Eastern Development Program) — added [YYYY-MM-DD] by Presten
ORG_ID_PENDING, // EDP Soccer — national
```
**Config section:** `// EDP Soccer` (create header if not present)
**When confirmed:** Replace `ORG_ID_PENDING` with actual org-ID. Remove header block. Paste into config.

---

#### ENTITY: Elite 64
```
# Elite 64 — Stage 2 Addition
# Type: National League (US Club Soccer affiliation — conditional GO)
# Browser session date: [leave blank — fill when confirmed]
# Confirmed by: Presten
# Note: Two variants pre-staged below. Use the one matching the browser session confirmation.
```

**Variant A — If confirmed as standalone GotSport org:**
```javascript
// Elite 64 — added [YYYY-MM-DD] by Presten
ORG_ID_PENDING, // Elite 64 — national
```
**Config section:** `// Elite 64` (create header if not present)

**Variant B — If found as sub-org under US Club Soccer:**
```javascript
// Elite 64 (via US Club Soccer) — added [YYYY-MM-DD] by Presten
ORG_ID_PENDING, // Elite 64 — national (US Club Soccer sub-org)
```
**Config section:** `// US Club Soccer` (add under existing header if present, or create)

**When confirmed:** Choose the correct variant. Replace `ORG_ID_PENDING`. Remove header block. Paste into config.

---

### 2.2 — USYS State Associations (Group B)

---

#### ENTITY: Cal North Soccer (USYS CA-NorCal)
```
# Cal North Soccer — Stage 2 Addition
# Type: USYS State Association (California — Northern)
# Browser session date: [leave blank — fill when confirmed]
# Confirmed by: Presten
```
```javascript
// Cal North Soccer (USYS CA-NorCal) — added [YYYY-MM-DD] by Presten
ORG_ID_PENDING, // Cal North Soccer — CA NorCal statewide
```
**Config section:** `// USYS State Associations`
**When confirmed:** Replace `ORG_ID_PENDING`. Remove header block. Paste under the USYS State Associations comment header.

---

#### ENTITY: Cal South Soccer (USYS CA-SoCal)
```
# Cal South Soccer — Stage 2 Addition
# Type: USYS State Association (California — Southern)
# Browser session date: [leave blank — fill when confirmed]
# Confirmed by: Presten
```
```javascript
// Cal South Soccer (USYS CA-SoCal) — added [YYYY-MM-DD] by Presten
ORG_ID_PENDING, // Cal South Soccer — CA SoCal statewide
```
**Config section:** `// USYS State Associations`
**When confirmed:** Replace `ORG_ID_PENDING`. Remove header block. Paste under the USYS State Associations comment header.

> [!note] Two CA entries
> Cal North and Cal South are separate USYS affiliates with separate org-IDs. Both must be confirmed and added independently.

---

#### ENTITY: Florida Youth Soccer Association (FYSA)
```
# Florida Youth Soccer Association (FYSA) — Stage 2 Addition
# Type: USYS State Association
# Browser session date: [leave blank — fill when confirmed]
# Confirmed by: Presten
```
```javascript
// Florida Youth Soccer Association (FYSA) — added [YYYY-MM-DD] by Presten
ORG_ID_PENDING, // FYSA — FL statewide
```
**Config section:** `// USYS State Associations`
**When confirmed:** Replace `ORG_ID_PENDING`. Remove header block. Paste.

---

#### ENTITY: Eastern NY Youth Soccer (ENYYSA)
```
# Eastern NY Youth Soccer (ENYYSA) — Stage 2 Addition
# Type: USYS State Association
# Browser session date: [leave blank — fill when confirmed]
# Confirmed by: Presten
```
```javascript
// Eastern NY Youth Soccer (ENYYSA) — added [YYYY-MM-DD] by Presten
ORG_ID_PENDING, // ENYYSA — NY statewide
```
**Config section:** `// USYS State Associations`
**When confirmed:** Replace `ORG_ID_PENDING`. Remove header block. Paste.

---

#### ENTITY: NJ Youth Soccer Association (NJYSA)
```
# NJ Youth Soccer Association (NJYSA) — Stage 2 Addition
# Type: USYS State Association
# Browser session date: [leave blank — fill when confirmed]
# Confirmed by: Presten
```
```javascript
// NJ Youth Soccer Association (NJYSA) — added [YYYY-MM-DD] by Presten
ORG_ID_PENDING, // NJYSA — NJ statewide
```
**Config section:** `// USYS State Associations`
**When confirmed:** Replace `ORG_ID_PENDING`. Remove header block. Paste.

---

#### ENTITY: Washington Youth Soccer (WSYSA)
```
# Washington Youth Soccer (WSYSA) — Stage 2 Addition
# Type: USYS State Association
# Browser session date: [leave blank — fill when confirmed]
# Confirmed by: Presten
```
```javascript
// Washington Youth Soccer (WSYSA) — added [YYYY-MM-DD] by Presten
ORG_ID_PENDING, // WSYSA — WA statewide
```
**Config section:** `// USYS State Associations`
**When confirmed:** Replace `ORG_ID_PENDING`. Remove header block. Paste.

---

#### ENTITY: Eastern PA Youth Soccer (EPYSA)
```
# Eastern PA Youth Soccer (EPYSA) — Stage 2 Addition
# Type: USYS State Association (Pennsylvania — Eastern; covers Philadelphia metro)
# Browser session date: [leave blank — fill when confirmed]
# Confirmed by: Presten
```
```javascript
// Eastern PA Youth Soccer (EPYSA) — added [YYYY-MM-DD] by Presten
ORG_ID_PENDING, // EPYSA — PA Eastern statewide
```
**Config section:** `// USYS State Associations`
**When confirmed:** Replace `ORG_ID_PENDING`. Remove header block. Paste.

> [!note] PA secondary lookup
> WPYSA (Western PA / Pittsburgh) is a secondary lookup per the prep package. If Presten also confirms WPYSA, pre-stage a separate entry:
> ```javascript
> // Western PA Youth Soccer (WPYSA) — added [YYYY-MM-DD] by Presten
> ORG_ID_PENDING, // WPYSA — PA Western statewide
> ```

---

## Section 2B: Additional Pending Confirmations

These entities are in the browser session lookup but are NOT in the 11 primary Stage 2 entries. They are either Stage 1 (TYSA) or Stage 3 (VA/CO/AZ) confirmations. Include in the same browser session if time permits.

---

#### ENTITY: Texas Youth Soccer Association (TYSA) — Stage 1 Confirmation
```
# TYSA — Stage 1 Config Confirmation
# Type: USYS State Association (TX — Stage 1 pilot)
# Note: TYSA entry is already in crawl config as a placeholder. Browser session confirms the actual numeric org_id.
# Current config entry: pending-lookup placeholder
# Confirmed by: Presten
```
```javascript
// Texas Youth Soccer Association (TYSA) — added [YYYY-MM-DD] by Presten
ORG_ID_PENDING, // TYSA — TX statewide
```
**Action when confirmed:** Replace the existing `pending-lookup` placeholder value in the config with the confirmed numeric org-ID. Update Org-ID Master Reference Section 1 to replace `pending-lookup` with actual org-ID. This unblocks Stage 1 crawl execution.

---

#### ENTITY: Virginia Youth Soccer Association (VYSA) — Stage 3
```
# Virginia Youth Soccer Association (VYSA) — Stage 3 Addition
# Type: USYS State Association
# Browser session date: [leave blank — fill when confirmed]
# Confirmed by: Presten
# Stage: 3 (post-Stage 2 authorization)
```
```javascript
// Virginia Youth Soccer Association (VYSA) — added [YYYY-MM-DD] by Presten
ORG_ID_PENDING, // VYSA — VA statewide
```
**Config section:** `// USYS State Associations — Stage 3`
**Note:** Do not add to config until Stage 3 is authorized. Confirm org-ID now; hold config add until authorization.

---

#### ENTITY: Colorado Youth Soccer Association (CYSA) — Stage 3
```
# Colorado Youth Soccer Association (CYSA) — Stage 3 Addition
# Type: USYS State Association
# GotSport presence: confirmed (GotSport platform documented)
# Browser session date: [leave blank — fill when confirmed]
# Confirmed by: Presten
# Stage: 3 (post-Stage 2 authorization)
```
```javascript
// Colorado Youth Soccer Association (CYSA) — added [YYYY-MM-DD] by Presten
ORG_ID_PENDING, // CYSA — CO statewide
```
**Config section:** `// USYS State Associations — Stage 3`
**Note:** Do not add to config until Stage 3 is authorized.

---

#### ENTITY: Arizona Youth Soccer (AzSA) — Stage 3
```
# Arizona Youth Soccer (AzSA) — Stage 3 Addition
# Type: USYS State Association
# Browser session date: [leave blank — fill when confirmed]
# Confirmed by: Presten
# Stage: 3 (post-Stage 2 authorization)
```
```javascript
// Arizona Youth Soccer (AzSA) — added [YYYY-MM-DD] by Presten
ORG_ID_PENDING, // AzSA — AZ statewide
```
**Config section:** `// USYS State Associations — Stage 3`
**Note:** Do not add to config until Stage 3 is authorized.

---

## Section 3: Not-on-GotSport Handling

Entities from the prep package (and Master Reference Section 3) that are confirmed or highly likely to be not on GotSport. No config entries needed for these.

```
# NAL (North American League) — Platform Unconfirmed
# Confirmed: Not confirmed yet — pending browser session result
# Reason: Platform unknown — GotSport unconfirmed as of 2026-04-26
# Alternative: Unknown. Check nal.net or similar for platform name if not found on GotSport.
# If found on GotSport: add config entry using the League Entity template above.
# If not found: move to Org-ID Master Reference Section 3 with platform identified (if any).
# No config entry staged — too uncertain to pre-stage.
```

The following are already in Org-ID Master Reference Section 3 (confirmed not-on-GotSport / deferred):

| Entity | Reason | Alternative Platform |
|--------|--------|---------------------|
| Illinois Youth Soccer (IYSA) | Confirmed Affinity Soccer | Affinity Soccer |
| Michigan Youth Soccer (MYSA) | Likely Affinity Soccer | Affinity Soccer (likely) |
| Minnesota Youth Soccer (MSYSA) | Likely Affinity Soccer | Affinity Soccer (likely) |
| Indiana Youth Soccer | Likely Affinity Soccer; low volume | Affinity Soccer (likely) |
| Wisconsin Youth Soccer (WYSA) | Likely Affinity Soccer; low volume | Affinity Soccer (likely) |

These are already documented in the Master Reference and require no action in this document.

---

## Section 4: Post-Session Completion Checklist

After Presten reports org-IDs and FORGE fills in the placeholders:

- [ ] All `ORG_ID_PENDING` values in Section 2 replaced with actual confirmed numbers
- [ ] NAL result recorded: either added to config (if found) or moved to Master Reference Section 3 (if not found)
- [ ] TYSA `pending-lookup` placeholder in live config replaced with confirmed numeric org-ID
- [ ] Stage 3 entities (VA/CO/AZ): org-IDs confirmed and recorded, but **not yet added to config** (pending Stage 3 authorization)
- [ ] Config block for each confirmed Stage 2 entity pasted into `crawl-gotsport-api-parallel.js` under the correct league header
- [ ] Syntax check run: `node -c crawl-gotsport-api-parallel.js && echo "Syntax OK"`
- [ ] Duplicate check run (per Config Edit Reference Section 3, Step 6)
- [ ] Org-ID Master Reference updated: all confirmed entities moved from Section 2 (Pending) to Section 1 (Active)
- [ ] Not-on-GotSport entities (if any) moved to Master Reference Section 3
- [ ] Stage 2 Authorization Trigger Document: check whether the browser session completion triggers the Stage 2 activation gate (per `Infrastructure/Stage 2 Crawl Expansion — Authorization Trigger Document.md`)
- [ ] Next briefing filed with: entity count added, org-IDs confirmed, any entities not found, Stage 2 gate status

---

## Section 5: Update Log

| Date | Entity | Org-ID Confirmed | Config Section Updated | Master Reference Updated |
|------|--------|-----------------|------------------------|--------------------------|

*Start empty. Populate as confirmations arrive.*

---

## References

- `Infrastructure/GotSport Browser Lookup Session Prep Package.md` — 11 entities requiring confirmation + recording template
- `Infrastructure/GotSport Org-ID Config Edit Reference.md` — canonical config entry format + step-by-step add instructions
- `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` — master status document; FORGE updates post-session
- `Infrastructure/Stage 2 Crawl Expansion — Authorization Trigger Document.md` — gate this session may trigger
- `Infrastructure/GotSport Org-ID Research — VA, CO, AZ.md` — Stage 3 entities research detail

*FORGE — 2026-04-26*
