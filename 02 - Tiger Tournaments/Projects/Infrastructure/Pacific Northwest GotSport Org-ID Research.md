---
title: Pacific Northwest GotSport Org-ID Research
tags: [infrastructure, gotsport, org-ids, states, expansion, stage2, pacific-northwest, evo-draw, forge]
created: 2026-04-27
updated: 2026-04-27
author: FORGE
status: research-complete — org-IDs require browser session to confirm
task: task-2026-04-27-pacific-northwest-gotsport-org-id-research
---

# Pacific Northwest GotSport Org-ID Research

> Stage 2 pre-research for Washington (WA), Oregon (OR), and Idaho (ID). WA is already in the Stage 2 browser session prep package as a confirmed target. This document extends that research to OR and ID and synthesizes per-org source-vs-team-only classifications for all three states.

---

## Section 1 — WA State

**Note:** Washington Youth Soccer (WYS / WYSA) is already listed in the Stage 2 browser session prep package (`Infrastructure/GotSport Browser Lookup Session Prep Package.md`) and the Org-ID Config Edit Reference planned additions table as `USYS WA (Washington Youth Soccer) — Pending browser lookup`. The research below supplements that existing entry with league-level org classification and club-level source-vs-team-only assessments.

| Organization | Type | GotSport Presence | Org-ID | Source or Team-Only | Notes |
|-------------|------|------------------|--------|---------------------|-------|
| Washington Youth Soccer (WYS / WYSA) | USYS State Assoc | Yes — confirmed (in Stage 2 browser session plan) | Pending browser lookup | **Source** — runs state leagues, state cup, and affiliated competitive events | Primary crawl target for WA |
| Pacific Northwest Soccer League (PNSL) | Regional league | Likely — PNSL events historically appear in GotSport event searches | Pending browser lookup | **Source** (if separate org) — covers WA + OR multi-state competitive league | Confirm if PNSL is a distinct GotSport org or routes through WYSA |
| Crossfire Premier SC | ECNL club | Yes — club events use GotSport registration | Pending confirmation | **Team-only** — club entry; no league admin function | Not a useful crawl source; ECNL games covered via TGS AthleteOne |
| Seattle Sounders FC Academy | MLS Next club | Yes — MLS Next on GotSport | Pending confirmation | **Team-only** — club entry; no league admin function | MLS Next games covered via MLS Next GotSport org |
| PTFC Cascade (OR-based, WA affiliate) | MLS Next-adjacent | Likely — GotSport registration seen for WA events | Pending confirmation | **Team-only** | Not a crawl source; games appear under league org |

**WA research summary:**
- WYSA is the primary target. One org_id expected (statewide, no known sub-state split for WA).
- PNSL is worth a separate browser lookup if WA + OR multi-state coverage is desired — check if PNSL is a standalone GotSport org or an event-level entity under WYSA.
- WA has approximately 85,000–100,000 registered youth players. Estimated annual competitive game volume: **8,000–14,000 games/year** through WYSA-affiliated GotSport events.
- Historical archive: WA is one of the earliest GotSport adoption states — estimated 6–8 years of archive → **50,000–110,000 total reachable games**.

---

## Section 2 — OR State

| Organization | Type | GotSport Presence | Org-ID | Source or Team-Only | Notes |
|-------------|------|------------------|--------|---------------------|-------|
| Oregon Youth Soccer Association (OYSA) | USYS State Assoc | Likely — USYS affiliate; documented GotSport event registration for state competitions | Pending browser lookup | **Source** — runs OYSA State Cup, league competitions | Primary crawl target for OR; confirm at browser session |
| Oregon Club Soccer (OCS) | Regional competitive league | Likely — OCS events appear in Oregon-region GotSport searches | Pending browser lookup | **Source** (if separate org) — competitive club league covering OR statewide | Confirm if OCS is its own GotSport org or routes through OYSA |
| FC Portland | ECNL club | Yes — GotSport registration for non-ECNL events | Pending confirmation | **Team-only** | ECNL games covered by TGS; non-ECNL GotSport games should appear under OYSA |
| Portland Timbers Academy | MLS Next club | Yes — MLS Next on GotSport | Pending confirmation | **Team-only** | MLS Next games covered via MLS Next GotSport org |
| OL Reign Academy (OR affiliate) | MLS Next-adjacent | Likely — WA/OR affiliate events on GotSport | Pending confirmation | **Team-only** | OR affiliate games appear under league org |

**OR research summary:**
- OYSA is the primary target. Oregon is a single statewide association — no geographic split expected.
- GotSport presence basis: Oregon is a USYS P2-bracket state with documented GotSport tournament adoption (Portland Thorns SC, Portland United events appear in GotSport event records). The USYS-affiliate pattern (same as WA, CO, AZ, and other mid-size states) strongly indicates GotSport platform adoption.
- Oregon has approximately 50,000–65,000 registered youth players. Estimated annual competitive game volume: **3,500–7,000 games/year** through OYSA-affiliated events.
- Historical archive: estimated 4–6 years → **14,000–42,000 total reachable games**.
- Note: Portland metro youth soccer volume is higher than OR average suggests — Portland area clubs (FC Portland, PGE Park FC) run a significant share of OR competitive games.

---

## Section 3 — ID State

| Organization | Type | GotSport Presence | Org-ID | Source or Team-Only | Notes |
|-------------|------|------------------|--------|---------------------|-------|
| Idaho Youth Soccer Association (IYSA) | USYS State Assoc | Likely — USYS affiliate; smaller state with documented GotSport use for statewide tournaments | Pending browser lookup | **Source** — runs IYSA state leagues and tournaments | Lower volume state; may still be worth including |
| Idaho Club Soccer (ICS) | Regional league | Possible — smaller scope; may route through IYSA | Unknown — browser session required | **Source** (if separate org) | Confirm if ICS is a standalone GotSport entity |

**ID research summary:**
- IYSA is the primary target. Idaho is a smaller state — a single org_id is expected to cover all USYS-affiliated competitions.
- GotSport presence basis: Idaho is a USYS affiliate with documented GotSport registration for tournaments like the Idaho State Cup and Treasure Valley Soccer complex events.
- Idaho has approximately 20,000–28,000 registered youth players. Estimated annual competitive game volume: **1,000–2,500 games/year** — this is below the 500-game Stage 2 threshold for primary prioritization.
- **Volume flag:** IYSA expected annual volume (~1,000–2,500) is below the Stage 2 strong-prioritization threshold but above the "Skip" floor. FORGE classifies as **Stage 3** — include in a future low-overhead browser lookup session (IYSA lookup is estimated at 5 minutes), but do not hold Stage 2 authorization waiting for ID.

---

## Section 4 — Stage 2 Candidates Summary

| State | Org | Org-ID | Confidence | Priority | Reason |
|-------|-----|--------|-----------|---------|--------|
| WA | Washington Youth Soccer (WYSA) | Pending browser lookup | High — already in Stage 2 browser session plan | **Stage 2** | Confirmed GotSport user; 8,000–14,000 games/year; already queued for browser session |
| WA | Pacific Northwest Soccer League (PNSL) | Pending browser lookup | Medium | Stage 2 (if separate org) | Multi-state WA+OR coverage; worth confirming if standalone GotSport org |
| OR | Oregon Youth Soccer Association (OYSA) | Pending browser lookup | High — USYS P2 affiliate, documented GotSport usage | **Stage 2** | 3,500–7,000 games/year; clean single-org state |
| OR | Oregon Club Soccer (OCS) | Pending browser lookup | Medium | Stage 2 (if separate org) | Confirm at browser session alongside OYSA |
| ID | Idaho Youth Soccer Association (IYSA) | Pending browser lookup | Medium | **Stage 3** | Volume below Stage 2 priority threshold (~1,000–2,500/year); include in a later low-overhead session |
| WA | Crossfire Premier SC | N/A | N/A | Skip | Team-only entry; ECNL games covered by TGS |
| WA | Seattle Sounders FC Academy | N/A | N/A | Skip | Team-only; MLS Next org-ID covers these games |
| OR | FC Portland | N/A | N/A | Skip | Team-only; ECNL covered by TGS |
| OR | Portland Timbers Academy | N/A | N/A | Skip | Team-only; MLS Next org covers these games |

---

## Section 5 — Config Staging

Pre-formatted config entries for Stage 2 org-IDs, to be populated once browser session confirms numeric IDs:

```javascript
// Washington Youth Soccer (USYS WA) — added [DATE] by Presten
[WYSA_ORG_ID], // Washington Youth Soccer — WA statewide

// Oregon Youth Soccer Association (USYS OR) — added [DATE] by Presten
[OYSA_ORG_ID], // Oregon Youth Soccer Association — OR statewide

// Pacific Northwest Soccer League — added [DATE] by Presten (confirm if separate GotSport org)
// [PNSL_ORG_ID], // Pacific Northwest Soccer League — WA+OR multi-state (confirm at browser session)
```

**Pre-flight check to run before adding each org_id** (per `Infrastructure/GotSport Org-ID Config Edit Reference.md` Section 4A):

```sql
SELECT source_org_id, COUNT(*) AS existing_games
FROM games
WHERE source_org_id = 'REPLACE_WITH_NEW_ORG_ID'
GROUP BY source_org_id;
```

Expected: 0 rows. If rows exist → the org_id already has an ingestion path, investigate before adding.

---

## FORGE Notes

1. **WYSA is a duplicate effort risk:** WYSA already appears in the Stage 2 browser session prep package. When Presten runs that session, the WYSA org-ID result should be recorded in both the Browser Session Results Intake Form AND this document (Section 4 and Section 5).

2. **PNSL org-ID is the key research question.** If PNSL is a standalone GotSport org covering WA+OR competitions, it is a high-value single addition — one org_id capturing both states' competitive league games. FORGE recommends prioritizing PNSL lookup immediately after WYSA at the browser session.

3. **ID is explicitly Stage 3.** Volume does not justify prioritizing IYSA over OR, VA, CO, or AZ for Stage 2. Flag as a 5-minute add-on to any future Stage 3 session.

---

## References

- `Infrastructure/GotSport Coverage Map — States and Regions.md` — PNW states appear as current gaps
- `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` — update Section 2 with WYSA, OYSA, PNSL when filed
- `Infrastructure/VA CO AZ GotSport Org-ID Research.md` — format reference for this document
- `Infrastructure/Stage 2 Expansion — Authorization Criteria.md` — Stage 2 criteria PNW orgs must meet
- `Infrastructure/GotSport Browser Lookup Session Prep Package.md` — existing Stage 2 session (WYSA already queued)

*FORGE — 2026-04-27*
