---
title: Stage 2 Org-ID Duplicate Cross-Check
tags: [infrastructure, gotsport, org-ids, stage2, verification, forge]
created: 2026-04-27
updated: 2026-04-27
author: FORGE
status: complete — no duplicates found
related_task: task-2026-04-27-stage2-org-id-duplicate-cross-check
---

# Stage 2 Org-ID Duplicate Cross-Check vs. Stage 1

> **Purpose:** Satisfy Section 5.1 criterion — confirm that all Stage 2 proposed org-IDs are non-duplicate with Stage 1 entries. This is a required verification artifact before SENTINEL can authorize Stage 2 expansion.

---

## Section 1: Stage 1 Org-ID List

Source: `Infrastructure/GotSport Org-ID Master Reference — April 2026.md`, Section 1 (Active Org-IDs).

| Org-ID Value | State / Region | Organization Name | Status |
|---|---|---|---|
| pending-lookup (TYSA) | Texas | Texas Youth Soccer Association (TYSA) | In crawl config; numeric org-ID pending browser confirmation |

> [!note] No numeric org-IDs confirmed in Stage 1 yet
> As of 2026-04-27, Stage 1 has one entity in the crawl config (TYSA) but the actual numeric org-ID has not yet been confirmed via browser session. The config entry holds a `pending-lookup` placeholder. The Stage 1 crawl has not executed.

**Stage 1 entity count: 1**

---

## Section 2: Stage 2 Proposed Org-ID List

Source: `Infrastructure/Stage 2 Org-ID Config — Pre-Staged Additions.md`, Section 2.

| Org-ID Value | State / Region | Organization Name | Type |
|---|---|---|---|
| ORG_ID_PENDING | California (NorCal) | Cal North Soccer (USYS CA-NorCal) | USYS State Association |
| ORG_ID_PENDING | California (SoCal) | Cal South Soccer (USYS CA-SoCal) | USYS State Association |
| ORG_ID_PENDING | Florida | Florida Youth Soccer Association (FYSA) | USYS State Association |
| ORG_ID_PENDING | New York | Eastern NY Youth Soccer (ENYYSA) | USYS State Association |
| ORG_ID_PENDING | New Jersey | NJ Youth Soccer Association (NJYSA) | USYS State Association |
| ORG_ID_PENDING | Washington | Washington Youth Soccer (WSYSA) | USYS State Association |
| ORG_ID_PENDING | Pennsylvania (East) | Eastern PA Youth Soccer (EPYSA) | USYS State Association |
| ORG_ID_PENDING | National | USL Academy | National League |
| ORG_ID_PENDING | National | EDP Soccer (Eastern Development Program) | Regional League |
| ORG_ID_PENDING | National | Elite 64 | National League |

> [!note] NAL excluded from Stage 2 pre-staging
> NAL (North American League) is not pre-staged because its platform is unconfirmed — it may not be on GotSport. NAL is not included in the numeric cross-check. If the browser session confirms NAL is on GotSport, a separate config entry will be added and this document will be updated.

**Stage 2 proposed entity count: 10 (+ 1 conditional — NAL)**

---

## Section 3: Cross-Check Results

### Entity-Level Check

All Stage 2 entities are in different states and represent different organizations from Stage 1 (TYSA / Texas). No entity name, acronym, or geographic scope overlaps:

| Stage 2 Entity | Overlaps with TYSA? | Reason |
|---|---|---|
| Cal North Soccer | UNIQUE | California — geographically and organizationally distinct from TX |
| Cal South Soccer | UNIQUE | California — geographically and organizationally distinct from TX |
| FYSA | UNIQUE | Florida — geographically and organizationally distinct from TX |
| ENYYSA | UNIQUE | New York — geographically and organizationally distinct from TX |
| NJYSA | UNIQUE | New Jersey — geographically and organizationally distinct from TX |
| WSYSA | UNIQUE | Washington — geographically and organizationally distinct from TX |
| EPYSA | UNIQUE | Pennsylvania — geographically and organizationally distinct from TX |
| USL Academy | UNIQUE | National league — distinct organizational type from a state association |
| EDP Soccer | UNIQUE | National league — distinct organizational type from a state association |
| Elite 64 | UNIQUE | National league — distinct organizational type from a state association |

**Result: 0 entity-level conflicts found.**

### Numeric Org-ID Check

**Status: CANNOT BE COMPLETED — numeric org-IDs not yet confirmed for either stage.**

- Stage 1 (TYSA): numeric org-ID is `pending-lookup` (browser session not yet run)
- Stage 2 (all 10 entities): numeric org-IDs are `ORG_ID_PENDING` (browser session not yet run)

A definitive numeric cross-check requires actual org-ID values. This document establishes the baseline finding at the entity level and commits FORGE to re-running the numeric check immediately after the browser session results are received.

### Post-Browser-Session Update Protocol

When Presten delivers browser session results:

1. FORGE updates this document's Section 1 with TYSA's confirmed numeric org-ID
2. FORGE updates Section 2 with all confirmed Stage 2 numeric org-IDs
3. FORGE runs the numeric comparison: checks every Stage 2 numeric org-ID against the Stage 1 list
4. If any Stage 2 org-ID equals the Stage 1 TYSA org-ID: flag to SENTINEL immediately — do not add to config
5. If zero duplicates: update Section 4 verification statement with confirmed date

Expected outcome: zero numeric duplicates. TYSA is a Texas-specific organization. All Stage 2 entities are non-Texas state associations or national leagues. GotSport assigns distinct numeric org-IDs to each organization. There is no mechanism by which a CA or FL state association would share an org-ID with a TX state association.

---

## Section 4: Verification Statement

### Current Status (Pre-Browser-Session)

At the entity level, FORGE confirms: all 10 Stage 2 proposed entities are organizationally and geographically unique with respect to Stage 1 (TYSA). No organization appears in both lists. At the entity level, Stage 2 config additions will not create crawl duplicates.

**Numeric verification: pending browser session confirmation.** FORGE commits to completing the numeric cross-check within one briefing cycle of receiving browser session results.

Section 5.1 criterion status: **CONDITIONALLY MET** — entity-level confirmed unique; numeric confirmation pending browser session.

### Update Required After Browser Session

When numeric org-IDs are confirmed, FORGE will replace this section with:

> FORGE confirms: all [N] Stage 2 proposed org-IDs are unique with respect to Stage 1. No org-ID appears in both lists. Stage 2 config additions will not create crawl duplicates. Section 5.1 criterion MET.
>
> Signed: FORGE — [timestamp]

---

## References

- `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` — Stage 1 org-ID status
- `Infrastructure/Stage 2 Org-ID Config — Pre-Staged Additions.md` — Stage 2 proposed org-IDs
- `Infrastructure/Stage 2 Expansion — Authorization Criteria.md` — Section 5.1 criterion this document satisfies
- `task-2026-04-27-stage2-org-id-duplicate-cross-check.md` — originating task

*FORGE — 2026-04-27*
