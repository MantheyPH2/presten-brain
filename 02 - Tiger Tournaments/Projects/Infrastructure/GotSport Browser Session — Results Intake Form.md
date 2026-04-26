---
title: GotSport Browser Session — Results Intake Form
tags: [infrastructure, gotsport, org-ids, browser-session, evo-draw, forge]
created: 2026-04-26
updated: 2026-04-26
author: FORGE
status: ready — Presten completes during browser session
---

# GotSport Browser Session — Results Intake Form

---

## Usage Instructions

```
PRESTEN: Open this file before starting the browser session. For each entity:
1. Go to system.gotsport.com
2. Search the entity name from the table below
3. If found: copy the org_id from the URL (?org_id=XXXX or ?OrgId=XXXX)
4. If not found: write "NOT FOUND"
5. Note any ambiguity (multiple results, name variation, alternate platform) in the Notes column
6. File this document when done — FORGE will update all dependent documents within the same session

Reference for search tips: Infrastructure/GotSport Browser Lookup Session Prep Package.md
```

---

## Results Table

| # | Entity Name | Type | Found? (Y/N) | Org-ID (if found) | Notes / Ambiguity |
|---|-------------|------|-------------|-------------------|-------------------|
| 1 | USL Academy | League entity | | | USL Soccer is a known GotSport org; Academy may be sub-org |
| 2 | EDP Soccer | League entity | | | Try "Eastern Development Program" or "EDP Soccer" |
| 3 | Elite 64 | League entity | | | Try "Elite 64", "Elite64", or "E64"; may appear under US Club Soccer |
| 4 | NAL | League entity | | | Try "North American League", "NAL Soccer", "NAL Youth"; platform uncertain |
| 5 | Cal North Soccer | USYS State Assoc (CA-NorCal) | | | California North/NorCal affiliate |
| 6 | Cal South Soccer | USYS State Assoc (CA-SoCal) | | | California South/SoCal affiliate — separate org from Cal North |
| 7 | Florida Youth Soccer (FYSA) | USYS State Assoc (FL) | | | Try "Florida Youth Soccer" |
| 8 | Eastern NY Youth Soccer (ENYYSA) | USYS State Assoc (NY) | | | Try "Eastern NY Youth Soccer" or "ENYYSA" |
| 9 | NJ Youth Soccer (NJYSA) | USYS State Assoc (NJ) | | | Try "New Jersey Youth Soccer" |
| 10 | Washington Youth Soccer (WSYSA) | USYS State Assoc (WA) | | | Try "Washington Youth Soccer" |
| 11 | Eastern PA Youth Soccer (EPYSA) | USYS State Assoc (PA) | | | Try "Eastern Pennsylvania Youth Soccer"; covers Philadelphia metro |

---

## Post-Session Summary (Presten fills)

```
Session Date: ___________
Total entities confirmed: ___/11
Total org-IDs captured: ___
Total not-found: ___
Session duration: ___
Any entities requiring follow-up (ambiguous results, multiple matches): ___
```

---

## FORGE Processing Protocol (FORGE fills after receiving intake)

When this form is filed, FORGE executes in this order:

### 1. Update GotSport Org-ID Master Reference
File: `Infrastructure/GotSport Org-ID Master Reference — April 2026.md`
- Move confirmed rows from `pending-browser-lookup` → `confirmed-[org_id]` in Section 2
- Move confirmed rows to Section 1 (Active Org-IDs) with stage = "Stage 2 candidate" and date = session date
- Move not-found rows to Section 3 (Deferred / Not-on-GotSport) with reason = "Not found in browser session [date]"
- Update Section 4 Status Summary counts

### 2. Update Stage 2 Authorization Trigger Document
File: `Infrastructure/Stage 2 Crawl Expansion — Authorization Trigger Document.md`
- Section 3: Update each entity row from `pending-browser-lookup` to `confirmed-[org_id]` or `not-on-gotsport`
- Add note: "Browser session completed [date]. [N] entities confirmed, [N] not on GotSport."

### 3. Update GotSport Org-ID Config Edit Reference
File: `Infrastructure/GotSport Org-ID Config Edit Reference.md`
- For each confirmed Tier A source (EDP, USL Academy, Elite 64): note org_id is confirmed and ready to add to crawl config
- Presten must authorize before config edit

### 4. Notify SENTINEL
In the next FORGE briefing, note:
- "Browser session complete. [N] confirmed, [N] not found."
- "Tier A sources confirmed: [list]. Config update ready for Presten authorization."
- "Any entities requiring follow-up: [list or 'none']."

---

## References

- `Infrastructure/GotSport Browser Lookup Session Prep Package.md` — session guide, search tips, alternate search terms
- `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` — primary update target
- `Infrastructure/Stage 2 Crawl Expansion — Authorization Trigger Document.md` — Section 3 update
- `Infrastructure/GotSport Org-ID Config Edit Reference.md` — config update guidance
- `Infrastructure/Non-GotSport Source Research — April 2026 Synthesis.md` — Tier A source context

*FORGE — 2026-04-26*
