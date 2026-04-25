---
title: GotSport Browser Lookup Session Prep Package
tags: [infrastructure, gotsport, org-ids, usys, session-guide, evo-draw]
created: 2026-04-25
updated: 2026-04-25
author: FORGE
status: ready — use at next browser session
---

# GotSport Browser Lookup Session Prep Package

---

## Section 1: Session Summary

**What this session accomplishes:** Confirm GotSport org IDs for 4 league entities (USL Academy, EDP, Elite 64, NAL) and 7 USYS state associations. These lookups unblock 3 pending task files and fill `null` entries in the USYS Org-ID Master List that are blocking Stage 2 planning.

| Item | Value |
|------|-------|
| **URL** | `system.gotsport.com` |
| **Estimated total time** | ~45 minutes |
| **Tasks unblocked** | `task-2026-04-25-usl-academy-edp-org-id-verification.md`, `task-2026-04-25-elite64-nal-source-research.md`, USYS Org-ID Master List Stage 2 entries |
| **After session** | Send results to FORGE. FORGE updates Source Gap Inventory and Org-ID Master List within 24 hours. |

---

## Section 2: URL and How to Extract an Org ID

**Navigation:**

1. Go to `system.gotsport.com`
2. Use the Organization search (top navigation or search bar)
3. Enter the organization name from the Lookup Table below
4. Click through to the organization's page
5. **Extract the org ID from the URL** — it appears as `?org_id=XXXX` or `?OrgId=XXXX` in the address bar. Record that number.

**If org not found on first try:**
- Try abbreviated name (e.g., "FYSA" instead of "Florida Youth Soccer Association")
- Try parent organization name
- Try state name alone (e.g., "Texas Soccer")
- If still not found after 2-3 attempts: record "not found" and move on

---

## Section 3: Lookup Table — All 11 Entities

### Group A: League Entities (~20 min)

| # | Entity | GotSport Search Term | What to Record | Notes | Est. Time |
|---|--------|----------------------|----------------|-------|-----------|
| 1 | USL Academy | "USL Academy" or "United Soccer League Academy" | org_id from URL | USL Soccer is a known GotSport org; Academy division may be a sub-org | 5 min |
| 2 | EDP Soccer | "Eastern Development Program" or "EDP Soccer" | org_id from URL | ~150+ clubs in Northeast; high-probability GotSport presence | 5 min |
| 3 | Elite 64 | "Elite 64" or "Elite64" or "E64" | org_id, OR "not found"; if not found check "US Club Soccer" for a sub-org | Provisional finding: likely on GotSport via US Club Soccer affiliation | 10 min |
| 4 | NAL | "North American League" or "NAL Soccer" or "NAL Youth" | If found: org_id. If not found: note it and check `nal.net` or similar for their result platform name | Platform unknown — GO/NO-GO deferred until platform confirmed | 10 min |

### Group B: USYS State Associations (~25 min)

These fill `null` entries in the USYS Org-ID Master List. Use the exact search terms below — they match GotSport's organization naming conventions.

| # | State | Entity Name | GotSport Search Term | What to Record | Est. Time |
|---|-------|-------------|----------------------|----------------|-----------|
| 5 | CA (NorCal) | Cal North Soccer | "Cal North Soccer" | org_id from URL | 2 min |
| 6 | CA (SoCal) | Cal South Soccer | "Cal South Soccer" | org_id from URL | 2 min |
| 7 | FL | Florida Youth Soccer Association (FYSA) | "Florida Youth Soccer" | org_id from URL | 2 min |
| 8 | NY | Eastern NY Youth Soccer (ENYYSA) | "Eastern NY Youth Soccer" or "ENYYSA" | org_id from URL | 2 min |
| 9 | NJ | NJ Youth Soccer Association (NJYSA) | "New Jersey Youth Soccer" | org_id from URL | 2 min |
| 10 | WA | Washington Youth Soccer (WSYSA) | "Washington Youth Soccer" | org_id from URL | 2 min |
| 11 | PA | Eastern PA Youth Soccer (EPYSA) | "Eastern Pennsylvania Youth Soccer" | org_id from URL | 3 min |

> [!note] CA has two orgs
> California has two separate USYS affiliates: Cal North (NorCal) and Cal South (SoCal). Both need their own org IDs — look up each separately.

> [!note] PA note
> Pennsylvania has East and West sub-associations. Lookup EPYSA (Eastern PA) first — it covers the Philadelphia metro, which is higher priority for DSS. WPYSA (Western PA / Pittsburgh) can be a secondary lookup if time permits.

---

## Section 4: Recording Template

**Fill in during the session. Leave blank if not found yet.**

| # | Entity | Found? (Y/N) | org_id | Notes |
|---|--------|-------------|--------|-------|
| 1 | USL Academy | | | |
| 2 | EDP Soccer | | | |
| 3 | Elite 64 | | | |
| 4 | NAL | | | |
| 5 | Cal North Soccer (CA-NorCal) | | | |
| 6 | Cal South Soccer (CA-SoCal) | | | |
| 7 | Florida Youth Soccer (FYSA) | | | |
| 8 | Eastern NY Youth Soccer (ENYYSA) | | | |
| 9 | NJ Youth Soccer (NJYSA) | | | |
| 10 | Washington Youth Soccer (WSYSA) | | | |
| 11 | Eastern PA Youth Soccer (EPYSA) | | | |

---

## Section 5: After the Session

Send results to FORGE (paste the completed Recording Template above into a message or queue item). FORGE will:

1. Update `Source Gap Inventory — April 2026.md` rows for USL Academy, EDP, Elite 64, NAL
2. Update `GotSport USYS Org-ID Master List.md` with confirmed org IDs for the 7 state entries
3. Mark `task-2026-04-25-usl-academy-edp-org-id-verification.md` and `task-2026-04-25-elite64-nal-source-research.md` as complete (or note what remains for further research)
4. Flag any confirmed org IDs as ready to add to crawl config — Presten executes when ready

**Turnaround:** Within 24 hours of receiving results.

---

## References

- `Data Sources/GotSport USYS Org-ID Master List.md` — master list this session populates
- `Infrastructure/Source Gap Inventory — April 2026.md` — league rows for USL Academy, EDP, Elite 64, NAL
- `10 - Agents/FORGE/Tasks/task-2026-04-25-usl-academy-edp-org-id-verification.md`
- `10 - Agents/FORGE/Tasks/task-2026-04-25-elite64-nal-source-research.md`

*FORGE — 2026-04-25*
