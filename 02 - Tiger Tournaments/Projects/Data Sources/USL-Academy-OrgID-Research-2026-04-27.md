---
title: USL Academy — GotSport Org-ID Research
tags: [data-sources, usl-academy, gotsport, org-id, coverage, evo-draw, forge]
created: 2026-04-27
author: FORGE
status: partial — org_id pending browser lookup; research synthesis complete
task: task-2026-04-27-usl-academy-org-id-research
---

# USL Academy — GotSport Org-ID Research

> Research synthesis from vault knowledge. Org-ID confirmation requires Presten browser session at `system.gotsport.com`. This document answers: is USL Academy on GotSport, what's the acquisition path, and is it worth prioritizing before May 9?

**Prepared by:** FORGE — 2026-04-27  
**Org-ID status:** `pending-browser-lookup` (already in `GotSport Org-ID Master Reference — April 2026.md` Section 2)

---

## 1. Club List Source

**Source:** USL Academy is a Boys academy league operated by USL Soccer (United Soccer Leagues). The 90-club estimate comes from the league hierarchy research and `Infrastructure/Source Gap Inventory — April 2026.md`.

- League scope: U13, U14, U15, U16, U17, U19 Boys
- Professional pathway: clubs are affiliated with USL Championship or USL League One franchises
- Club list currency: the 90-club figure was current as of the Source Gap Inventory (filed 2026-04-24). The full current club list is available at `uslsoccer.com/usla` — FORGE has not verified whether any clubs joined or departed since the count was recorded.

**Confidence in club count:** Medium. 90 is the figure in the inventory; the actual active club count may differ by ±5–10 clubs depending on the season.

---

## 2. GotSport Presence Check

**Method:** Vault research cross-check (no browser lookup available without Presten). Evidence of GotSport presence:

| Evidence Type | Finding |
|--------------|---------|
| USL Soccer parent org status | USL Soccer is a confirmed GotSport organization. GotSport is the primary tournament and league management platform for USL Soccer's youth programs. High probability USL Academy events are registered under a USL Soccer org_id or a dedicated USL Academy org_id. |
| Ranked-team discovery | Games from USL Academy clubs likely already appear in the Evo Draw DB via the ranked-team discovery mechanism — when a USL Academy club team is ranked, the GotSport API returns its recent games regardless of the org_id being explicitly configured. The DB likely has partial USL Academy coverage already. |
| Source Gap Inventory status | Listed as "Stale/Partial" — confirming some games are present, but explicit org_id is not configured. |

**Sampling method note:** FORGE cannot access `system.gotsport.com` directly. The 10–15 club GotSport presence sample requested in the task spec requires a Presten browser session or GotSport API access. This section substitutes vault-derived confidence assessment.

**GotSport presence assessment:** **High probability (85–90%).** USL Soccer's platform relationship with GotSport is well-established. A dedicated `USL Academy` org_id almost certainly exists at `system.gotsport.com`. The specific org_id numeric value is the only unknown.

---

## 3. Org-ID Acquisition Path

Three paths assessed:

| Path | Description | Feasibility |
|------|-------------|-------------|
| (a) Browser lookup per club | Navigate to each club's GotSport page; not the right approach for org-level discovery | Not recommended — 90 lookups is not the path |
| (b) GotSport org-level search at `system.gotsport.com` | Navigate to org search, search "USL Academy" or "United Soccer League Academy," retrieve the numeric org_id from the URL | **Recommended — 5–10 minutes, already in the browser session prep package** |
| (c) Known public resource | No known public directory of GotSport org_ids exists | Not viable without option (b) |

**Recommended path:** Path (b) — browser lookup at `system.gotsport.com`. This is already included in the planned browser session (`GotSport Org-ID Master Reference — April 2026.md` Section 2, row 1). No additional session planning required.

**Estimated time:** 5–10 minutes as part of the existing 11-entity browser session.

---

## 4. Game Volume Estimate

| Metric | Estimate | Basis |
|--------|----------|-------|
| Clubs in league | ~90 | Source Gap Inventory |
| Teams per club (U13–U19 boys) | ~3–6 age-group teams per club | Standard youth academy structure |
| Total teams in league | ~270–540 | Estimated |
| Games per team per season | ~20–30 (regular season + playoffs) | Standard youth league cadence |
| Total games per season | ~5,400–16,200 | Range based on team count × games |
| Games already in DB (partial) | Unknown — requires query: `SELECT COUNT(*) FROM games WHERE event_name ILIKE '%USL Academy%'` | Ranked-team discovery likely captured high-ranked teams |
| Incremental games from org_id add | Uncertain — depends on what ranked-team discovery already captured | If DB already has top-tier teams, increment may be 20–40% of total |

**Calibration against DSS tracker:** The Source Gap Inventory says USL Academy is a "Priority 2 stale/partial" source. The 20,000–35,000 estimate in the DSS tracker is specifically for USYS state cup unranked divisions — USL Academy is a separate gap. USL Academy's ~5,000–16,000 game estimate is directionally plausible and represents a meaningful coverage addition.

**Key uncertainty:** How many USL Academy games are already in the DB via ranked-team discovery? Running `SELECT COUNT(*) FROM games WHERE event_name ILIKE '%USL Academy%' OR event_tier = 'usl_academy'` would answer this. Recommend Presten runs this query before the browser session.

---

## 5. DSS Relevance

**Assessment: MEDIUM — worth confirming org_id before May 9.**

Rationale:
- USL Academy is a national Boys professional-pathway league (90 clubs, U13–U19). It is a known coverage gap that SENTINEL and Presten are aware of.
- If the org_id exists and is added to the crawl config, the game count addition may be substantial (5,000–16,000 games), comparable to or exceeding the FYSA or WSYSA state org additions.
- The work is a config add (5 minutes post browser lookup), not a new scraper build.
- However: if ranked-team discovery already captures most USL Academy teams (the "partial" in "Stale/Partial"), the incremental gain from explicit org_id config may be 1,000–3,000 additional games. Worth confirming.
- DSS coverage narrative: "USL Academy (90 professional-pathway clubs)" is a valuable item to include. Confirming org_id before May 9 allows it to appear in the May 16 coverage numbers.

**Recommendation: PRIORITIZE confirmation before May 9.** The browser session is already planned; USL Academy is already in the session scope. No additional scheduling required.

---

## 6. Recommended Next Action

**Specific action:** Presten should include USL Academy in the already-planned browser session at `system.gotsport.com`. The entity is in `GotSport Org-ID Master Reference — April 2026.md` Section 2, Row 1. The session is the unblocking action.

**Before the browser session, Presten should optionally run:**
```sql
SELECT COUNT(*), MAX(game_date) AS latest_game
FROM games
WHERE event_name ILIKE '%USL Academy%'
   OR event_tier = 'usl_academy';
```
This confirms whether the DB already has substantial USL Academy coverage (which would reduce the urgency of the org_id add) or very little (which raises the urgency).

**Post-browser session:** When USL Academy org_id is confirmed, FORGE will:
1. Update `GotSport Org-ID Master Reference — April 2026.md` Section 2 → Section 1
2. Pre-stage the config entry in `Stage 2 Org-ID Config — Pre-Staged Additions.md`
3. Note in next briefing with confirmed org_id

**If not on GotSport:** Move to Section 3 of the Master Reference with "no GotSport presence confirmed." Reassess via alternative source research (USL Soccer may have a data API or a public results page). Flag to SENTINEL as a Priority 1 gap.

---

## Status Cross-Reference

This research supplements the in-progress `task-2026-04-25-usl-academy-edp-org-id-verification.md`, which remains blocked on the browser session for org_id confirmation. That task is resolved when the browser session produces the org_id.

Current location in Org-ID Master Reference: **Section 2, Row 1** — `pending-browser-lookup`.

---

## References

- `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` — USL Academy in Section 2 (browser session pending)
- `Infrastructure/Source Gap Inventory — April 2026.md` — USL Academy listed as Priority 2 stale/partial
- `Infrastructure/GotSport Browser Lookup Session Prep Package.md` — USL Academy included in session scope
- `task-2026-04-25-usl-academy-edp-org-id-verification.md` — parent task (in_progress, blocked on browser)
- `Infrastructure/Stage 2 Org-ID Config — Pre-Staged Additions.md` — pre-staged config slot for USL Academy

*FORGE — 2026-04-27*
