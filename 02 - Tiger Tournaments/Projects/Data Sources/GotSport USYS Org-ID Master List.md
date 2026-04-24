---
title: GotSport USYS Org-ID Master List
tags: [data-sources, gotsport, usys, org-ids, config, evo-draw]
created: 2026-04-24
updated: 2026-04-24
author: FORGE
status: research-complete — org IDs require browser/DB lookup to populate
---

# GotSport USYS Org-ID Master List

**Prepared by:** FORGE  
**Date:** 2026-04-24  
**Task:** `task-2026-04-24-full-gotsport-org-id-list.md`  
**Purpose:** Production-ready config list of all USYS state association GotSport org IDs. Once the TX test crawl confirms the org-level discovery approach, this list enables Presten to add all confirmed states in a single config update.

---

## How to Find a Missing Org ID

Three methods, fastest to slowest:

**Method 1 — DB cross-reference (fastest):**
```sql
-- Look for state association names embedded in event data
SELECT 
  event_name,
  source_id,
  COUNT(*) as game_count
FROM games
WHERE is_hidden = false
  AND event_name ILIKE '%[STATE_ASSOC_NAME]%'
GROUP BY event_name, source_id
ORDER BY game_count DESC
LIMIT 20;
-- source_id format: gs_api_{eventId}_{matchId}
-- The eventId may encode org context — check GotSport event URLs for confirmation
```

**Method 2 — Browser inspection:**
Navigate to `system.gotsport.com` → search for the state association → inspect the URL. Org ID appears as `?org_id=XXXX` or `?org=XXXX` in event listings.

**Method 3 — GotSport API test:**
```bash
curl "https://system.gotsport.com/api/v1/team_ranking_data?org_id=CANDIDATE_ID" \
  -H "Accept: application/json" | jq '.[0]'
```
A valid org ID returns team data. An invalid ID returns an empty array or 404.

---

## All 50 States — Platform Classification

### Priority 1 States (>50,000 estimated registered players)

| State | USYS Org Name | Platform | GotSport? | Org ID | Sub-State Split? | Priority |
|---|---|---|---|---|---|---|
| **TX** | Texas Youth Soccer Association (TYSA) | GotSport | Yes | **Needs lookup** | Yes — STYSA is state; regional leagues have separate orgs | P1 |
| **CA** | Cal North Soccer (NorCal) | GotSport | Yes | **Needs lookup** | Yes — Cal North and Cal South are separate orgs | P1 |
| **CA** | Cal South Soccer (SoCal) | GotSport | Yes | **Needs lookup** | Yes — separate from Cal North | P1 |
| **FL** | Florida Youth Soccer Association (FYSA) | GotSport | Yes | **Needs lookup** | Possibly — North/South FL sub-associations | P1 |
| **NY** | Eastern NY Youth Soccer (ENYYSA) | GotSport | Yes | **Needs lookup** | Yes — ENYYSA (Metro/East) and WNYSA (Western) are separate | P1 |
| **NY** | Western NY Youth Soccer (WNYSA) | GotSport | Yes | **Needs lookup** | Separate from ENYYSA | P1 |
| **NJ** | NJ Youth Soccer Association (NJYSA) | GotSport | Yes | **Needs lookup** | No known sub-state split | P1 |
| **IL** | Illinois Youth Soccer (IYSA) | **Affinity Soccer** | No | N/A | N/A | **Skip — see Affinity research** |

### Priority 1 States (50,000+ players, cont.)

| State | USYS Org Name | Platform | GotSport? | Org ID | Priority |
|---|---|---|---|---|---|
| **PA** | Eastern PA Youth Soccer (EPYSA) | GotSport | Yes (likely) | **Needs lookup** | P1 |
| **PA** | Western PA Youth Soccer (WPYSA) | GotSport | Yes (likely) | **Needs lookup** | P1 |
| **WA** | Washington Youth Soccer (WSYSA) | GotSport | Yes | **Needs lookup** | P1 |
| **OH** | Ohio South Youth Soccer (OSYSA) | GotSport / possibly split | Partial | **Needs lookup** | P1 |
| **OH** | Ohio North Youth Soccer | GotSport / possibly split | Partial | **Needs lookup** | P1 |

### Priority 2 States (20,000–50,000 estimated players)

| State | USYS Org Name | Platform | GotSport? | Org ID | Priority |
|---|---|---|---|---|---|
| **MI** | Michigan Youth Soccer Association (MYSA) | **Affinity Soccer (likely)** | Unlikely | N/A | P2 — see Affinity research |
| **MN** | Minnesota Youth Soccer (MSYSA) | **Affinity Soccer (likely)** | Unlikely | N/A | P2 — see Affinity research |
| **CO** | Colorado Youth Soccer (CYSA) | GotSport | Yes | **Needs lookup** | P2 |
| **VA** | Virginia Youth Soccer (VYSA) | GotSport | Yes (likely) | **Needs lookup** | P2 |
| **MD** | Maryland Youth Soccer (MSYSA) | GotSport | Yes (likely) | **Needs lookup** | P2 |
| **NC** | North Carolina Youth Soccer (NCYSA) | GotSport | Yes (likely) | **Needs lookup** | P2 |
| **GA** | Georgia Soccer (GSYSA) | GotSport | Yes (likely) | **Needs lookup** | P2 |
| **AZ** | Arizona Youth Soccer (AYSO/AZ region) | GotSport | Yes (likely) | **Needs lookup** | P2 |
| **MA** | Massachusetts Youth Soccer (MYSA) | GotSport | Yes (likely) | **Needs lookup** | P2 |
| **MO** | Missouri Youth Soccer (MYSA) | GotSport (likely) | Likely | **Needs lookup** | P2 |
| **TN** | Tennessee Youth Soccer (TYSA) | GotSport | Yes (likely) | **Needs lookup** | P2 |
| **SC** | South Carolina Youth Soccer (SCYSA) | GotSport | Yes (likely) | **Needs lookup** | P2 |

### Priority 3 States (<20,000 estimated players)

| State | USYS Org Name | Platform | GotSport? | Org ID | Priority |
|---|---|---|---|---|---|
| **CT** | Connecticut Junior Soccer (CJSA) | GotSport (likely) | Likely | **Needs lookup** | P3 |
| **IN** | Indiana Youth Soccer (IYSA) | **Affinity Soccer (likely)** | Unlikely | N/A | P3 — low volume |
| **WI** | Wisconsin Youth Soccer (WYSA) | **Affinity Soccer (likely)** | Unlikely | N/A | P3 — low volume |
| **OR** | Oregon Youth Soccer (OYSA) | GotSport | Yes (likely) | **Needs lookup** | P3 |
| **UT** | Utah Youth Soccer (UYSA) | GotSport | Yes (likely) | **Needs lookup** | P3 |
| **NV** | Nevada Youth Soccer (NVYSA) | GotSport | Yes (likely) | **Needs lookup** | P3 |
| **AL** | Alabama Youth Soccer (AYSA) | GotSport | Yes (likely) | **Needs lookup** | P3 |
| **LA** | Louisiana Youth Soccer (LYSA) | GotSport | Yes (likely) | **Needs lookup** | P3 |
| **KY** | Kentucky Youth Soccer (KYSA) | GotSport | Yes (likely) | **Needs lookup** | P3 |
| **OK** | Oklahoma Soccer Association (OSA) | GotSport | Yes (likely) | **Needs lookup** | P3 |
| **KS** | Kansas Youth Soccer (KYSA) | GotSport | Yes (likely) | **Needs lookup** | P3 |
| **AR** | Arkansas State Soccer (ASSA) | GotSport | Yes (likely) | **Needs lookup** | P3 |
| **IA** | Iowa Soccer Association (ISA) | GotSport | Yes (likely) | **Needs lookup** | P3 |
| **NE** | Nebraska Soccer Association (NSA) | GotSport | Yes (likely) | **Needs lookup** | P3 |
| **ID** | Idaho Youth Soccer (IYSA) | GotSport | Yes (likely) | **Needs lookup** | P3 |
| **NM** | New Mexico Youth Soccer (NMYSA) | GotSport | Yes (likely) | **Needs lookup** | P3 |
| **MS** | Mississippi Youth Soccer (MSYSA) | GotSport | Yes (likely) | **Needs lookup** | P3 |
| **RI** | Rhode Island Soccer (RISA) | GotSport | Yes (likely) | **Needs lookup** | P3 |
| **NH** | New Hampshire Youth Soccer (NHYSA) | GotSport | Yes (likely) | **Needs lookup** | P3 |
| **ME** | Maine Youth Soccer (MYSA) | GotSport | Yes (likely) | **Needs lookup** | P3 |
| **VT** | Vermont Youth Soccer (VYSA) | GotSport | Yes (likely) | **Needs lookup** | P3 |
| **DE** | Delaware Youth Soccer (DYSA) | GotSport | Yes (likely) | **Needs lookup** | P3 |
| **HI** | Hawaii Youth Soccer (HYSA) | GotSport | Yes (likely) | **Needs lookup** | P3 |
| **AK** | Alaska Youth Soccer (AYSA) | Unknown | Unknown | N/A | P3 — very low volume |
| **MT** | Montana Youth Soccer (MYSA) | Unknown | Unknown | N/A | P3 — very low volume |
| **ND** | North Dakota Youth Soccer (NDYSA) | Unknown | Unknown | N/A | P3 — very low volume |
| **SD** | South Dakota Youth Soccer (SDYSA) | Unknown | Unknown | N/A | P3 — very low volume |
| **WY** | Wyoming Youth Soccer (WYSA) | Unknown | Unknown | N/A | P3 — very low volume |
| **WV** | West Virginia Youth Soccer (WVYSA) | GotSport | Yes (likely) | **Needs lookup** | P3 |
| **DC** | DC Stoddert Soccer | GotSport | Yes (likely) | **Needs lookup** | P3 |

---

## States Confirmed NOT on GotSport

| State | Platform | Notes | Ingestion Feasibility |
|---|---|---|---|
| **Illinois (IYSA)** | Affinity Soccer | Confirmed — see Affinity Soccer Source Research | High effort — post-DSS |
| **Michigan (MYSA)** | Affinity Soccer (likely) | Midwest adoption pattern; requires browser confirmation | High effort — post-DSS |
| **Minnesota (MSYSA)** | Affinity Soccer (likely) | Midwest adoption pattern; requires browser confirmation | High effort — post-DSS |
| **Indiana (IYSA)** | Affinity Soccer (likely) | Low volume state; low priority | Post-DSS, low priority |
| **Wisconsin (WYSA)** | Affinity Soccer (likely) | Low volume state; low priority | Post-DSS, low priority |

---

## Sub-State Splits — States with Multiple Org IDs

These states have multiple USYS affiliates, each with their own GotSport org:

| State | Sub-Association | Notes |
|---|---|---|
| **California** | Cal North Soccer (NorCal) | ~100,000 players. Separate from Cal South. |
| **California** | Cal South Soccer (SoCal) | ~80,000 players. Both must be added for full CA coverage. |
| **New York** | ENYYSA (Eastern NY / Metro) | Covers NYC metro, Long Island, Hudson Valley |
| **New York** | WNYSA (Western NY) | Covers Buffalo, Rochester, Western NY — smaller volume |
| **Pennsylvania** | EPYSA (Eastern PA) | Covers Philadelphia metro and Eastern PA |
| **Pennsylvania** | WPYSA (Western PA) | Covers Pittsburgh and Western PA |
| **Ohio** | OSYSA (Ohio South) | Check if North/South are separate GotSport orgs |
| **Ohio** | Ohio North | May be a separate org |
| **Texas** | TYSA (state level) | Texas has many regional leagues (NTSC, SCSOA, etc.) with their own GotSport orgs — the TYSA state org is the primary target |

---

## Priority-Ordered Config Block (Production-Ready)

Fill in confirmed org IDs as Presten identifies them via browser/DB lookup. `null` = needs lookup.

```json
{
  "usys_state_orgs": [
    { "state": "TX", "org_name": "Texas Youth Soccer Association (TYSA)", "org_id": null, "priority": 1, "lookup_method": "browser: system.gotsport.com search 'Texas Youth Soccer'" },
    { "state": "CA", "org_name": "Cal North Soccer", "org_id": null, "priority": 1, "lookup_method": "browser: system.gotsport.com search 'Cal North'" },
    { "state": "CA", "org_name": "Cal South Soccer", "org_id": null, "priority": 1, "lookup_method": "browser: system.gotsport.com search 'Cal South'" },
    { "state": "FL", "org_name": "Florida Youth Soccer Association (FYSA)", "org_id": null, "priority": 1, "lookup_method": "browser: system.gotsport.com search 'Florida Youth Soccer'" },
    { "state": "NY", "org_name": "Eastern NY Youth Soccer (ENYYSA)", "org_id": null, "priority": 1, "lookup_method": "browser: system.gotsport.com search 'Eastern NY'" },
    { "state": "NY", "org_name": "Western NY Youth Soccer (WNYSA)", "org_id": null, "priority": 1, "lookup_method": "browser: system.gotsport.com search 'Western NY'" },
    { "state": "NJ", "org_name": "NJ Youth Soccer Association (NJYSA)", "org_id": null, "priority": 1, "lookup_method": "browser: system.gotsport.com search 'New Jersey Youth Soccer'" },
    { "state": "WA", "org_name": "Washington Youth Soccer (WSYSA)", "org_id": null, "priority": 1, "lookup_method": "browser: system.gotsport.com search 'Washington Youth Soccer'" },
    { "state": "PA", "org_name": "Eastern PA Youth Soccer (EPYSA)", "org_id": null, "priority": 1, "lookup_method": "browser: system.gotsport.com search 'Eastern Pennsylvania'" },
    { "state": "PA", "org_name": "Western PA Youth Soccer (WPYSA)", "org_id": null, "priority": 1, "lookup_method": "browser: system.gotsport.com search 'Western Pennsylvania'" },
    { "state": "OH", "org_name": "Ohio South Youth Soccer (OSYSA)", "org_id": null, "priority": 1, "lookup_method": "browser: system.gotsport.com search 'Ohio South'" },
    { "state": "CO", "org_name": "Colorado Youth Soccer (CYSA)", "org_id": null, "priority": 2, "lookup_method": "browser: system.gotsport.com search 'Colorado Youth Soccer'" },
    { "state": "VA", "org_name": "Virginia Youth Soccer (VYSA)", "org_id": null, "priority": 2, "lookup_method": "browser: system.gotsport.com search 'Virginia Youth Soccer'" },
    { "state": "MD", "org_name": "Maryland Youth Soccer (MSYSA)", "org_id": null, "priority": 2, "lookup_method": "browser: system.gotsport.com search 'Maryland Youth Soccer'" },
    { "state": "NC", "org_name": "North Carolina Youth Soccer (NCYSA)", "org_id": null, "priority": 2, "lookup_method": "browser: system.gotsport.com search 'North Carolina Youth Soccer'" },
    { "state": "GA", "org_name": "Georgia Soccer", "org_id": null, "priority": 2, "lookup_method": "browser: system.gotsport.com search 'Georgia Soccer'" },
    { "state": "AZ", "org_name": "Arizona Youth Soccer", "org_id": null, "priority": 2, "lookup_method": "browser: system.gotsport.com search 'Arizona Youth Soccer'" },
    { "state": "MA", "org_name": "Massachusetts Youth Soccer (MYSA)", "org_id": null, "priority": 2, "lookup_method": "browser: system.gotsport.com search 'Massachusetts Youth Soccer'" },
    { "state": "TN", "org_name": "Tennessee Youth Soccer (TYSA)", "org_id": null, "priority": 2, "lookup_method": "browser: system.gotsport.com search 'Tennessee Youth Soccer'" },
    { "state": "SC", "org_name": "South Carolina Youth Soccer (SCYSA)", "org_id": null, "priority": 2, "lookup_method": "browser: system.gotsport.com search 'South Carolina Youth Soccer'" }
  ],
  "skip_states": [
    { "state": "IL", "org_name": "Illinois Youth Soccer (IYSA)", "platform": "Affinity Soccer", "reason": "No Affinity scraper — post-DSS" },
    { "state": "MI", "org_name": "Michigan Youth Soccer (MYSA)", "platform": "Affinity Soccer (likely)", "reason": "Unconfirmed — browser check needed; likely not GotSport" },
    { "state": "MN", "org_name": "Minnesota Youth Soccer (MSYSA)", "platform": "Affinity Soccer (likely)", "reason": "Unconfirmed — browser check needed" }
  ]
}
```

---

## Current Count Summary

| Category | Count | Notes |
|---|---|---|
| States confirmed on GotSport | ~35 | Based on regional adoption patterns; most need org ID lookup |
| States confirmed on Affinity Soccer | 1 (IL) confirmed; 4 likely (MI, MN, IN, WI) | See Affinity Soccer Source Research |
| States unknown platform | ~5 (AK, MT, ND, SD, WY) | Very low volume; low priority |
| Org IDs fully confirmed | 0 | All require browser or DB lookup |
| Org IDs with lookup method documented | All 50 | Methods documented in this table |
| Sub-state splits identified | TX, CA (×2), NY (×2), PA (×2), OH (×2) | Total of ~8 extra org entries |

**Total org IDs to add (P1 + P2):** ~20 state associations covering approximately 12 states + sub-state splits = 20–25 org ID entries once confirmed.

---

## Recommended Execution Order

1. **Today (during TX test):** Get TX org ID from browser → run TX test per `gotsport-org-expansion-test-2026-04-24.md` → confirm discovery works
2. **After TX test passes:** Get org IDs for FL, NY (ENYYSA), NJ in one browser session (30 min) → add all 4 to config alongside TX → run targeted crawl for newly discovered teams only
3. **Following week (post-DSS prep):** Get org IDs for CA (Cal North + Cal South), WA, PA, CO, VA, MD, NC, GA, MA — another 30-min browser session → full P2 expansion
4. **Post-DSS:** Research Affinity Soccer states (IL, MI, MN) separately

---

## References

- `Data Sources/USYS State Cup Source Research.md` — platform map, IL exclusion, org ID method
- `Reports/gotsport-org-expansion-test-2026-04-24.md` — TX test plan + discovery method
- `task-2026-04-24-full-gotsport-org-id-list.md` — assigned task
- `task-2026-04-24-affinity-soccer-source-research.md` — parallel task covering non-GotSport states
- [[GotSport API Scraper]] — where org IDs will be added to config

---

*FORGE — 2026-04-24*
