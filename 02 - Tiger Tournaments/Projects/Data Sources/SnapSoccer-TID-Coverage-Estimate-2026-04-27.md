---
title: SnapSoccer TID Coverage Estimate
tags: [data-sources, snapsoccer, sincsports, coverage, evo-draw, forge]
created: 2026-04-27
updated: 2026-04-27
author: FORGE
status: estimate — pre-browser-check
related_task: task-2026-04-27-snapsoccer-tid-coverage-estimate
---

# SnapSoccer TID Coverage Estimate — 2026-04-27

> **Purpose:** Estimate total TID count, expected new game volume, and discovery runtime for SnapSoccer ingestion. Needed before the May 17 build session so SENTINEL and Presten can set realistic expectations. This estimate is independent of the browser accessibility check.

---

## Section 1: TID Inventory Estimate

### Source of SnapSoccer TIDs

SnapSoccer competitive events are hosted on SincSports (`soccer.sincsports.com`). TIDs are discoverable by crawling `snapsoccer.com/events/` and extracting `tid=` parameters from links to SincSports schedule pages. The source research memo (`Data Sources/SnapSoccer Source Research.md`) confirms the pattern.

Known TIDs from prior research:

| TID | Event Type | Scores Available |
|-----|-----------|-----------------|
| SNAPLEAG | SnapSoccer Spring Playdates | NO — development only |
| SNAPLEAF | SnapSoccer Fall Playdates | NO — development only |
| SSSPF | SnapSoccer Spring Playdates (variant) | NO — development only |
| COASTALINV | Coastal Soccer Invitational | YES — competitive |
| ACADCUP | Coastal Academy Cup | YES — competitive |

The Playdate TIDs (3 known) are out of scope — no scores recorded. Only competitive tournament TIDs produce game result data.

### TID Count Estimate

**Basis for estimate:** SnapSoccer runs 25+ soccer weekends per year (per source research). Not all weekends produce distinct TIDs — some events recur annually under the same TID family (e.g., Coastal Soccer Invitational may have one TID per year: COASTALINV2024, COASTALINV2025). Assuming one distinct TID per event per year, and 10–15 scoreable competitive events per year, with a 5–8 year history on SincSports:

| Scenario | Annual Competitive Events | Years of History | Total Estimated TIDs |
|----------|--------------------------|-----------------|---------------------|
| Low | 10 | 5 | 50 |
| Mid | 13 | 6 | 78 |
| High | 15 | 8 | 120 |

**Point estimate: ~75 unique competitive TIDs** (mid scenario).

**Breakdown estimate:**
- Competitive tournaments (scored): ~75 TIDs
- Playdate/development events (no scores, skip): 3 known TIDs + ~10 variants over history = ~13 TIDs
- Total TIDs in SnapSoccer universe: ~85–90

**Confidence: LOW-MEDIUM**

The `snapsoccer.com/events/` page lists current and recent events but may not enumerate full historical event archives. Historical TID discovery may require a separate search strategy (e.g., Google indexing of SincSports URLs with SnapSoccer event names). Until the browser check confirms the events page structure, the estimate could be off by ±50%. A full crawl of the events page will produce a definitive count within minutes.

---

## Section 2: Expected New Game Volume

### Cross-Reference with Existing SincSports Dataset

The existing SincSports dataset contains **34,961 games** imported in bulk (not live-scraped). This bulk import covered SincSports games known to Evo Draw at the time of import.

Key question: are SnapSoccer TIDs already in that bulk import?

**Assessment: Partially — but the overlap is smaller than the total SincSports bulk count suggests.**

The SincSports bulk import was a one-time historical pull. SnapSoccer events are a specific subset of SincSports tournaments (Gulf Coast / Southeast region). The bulk import likely captured some SnapSoccer TIDs that were in the SincSports database at import time. However:

1. **Temporal gap:** The bulk import was a point-in-time snapshot. SnapSoccer events from 2024–2026 seasons are almost certainly not in the bulk dataset.
2. **Discovery gap:** The bulk import may not have specifically targeted SnapSoccer tournament IDs. If TID discovery was not exhaustive, some historical SnapSoccer TIDs may be missing.
3. **Playdate exclusion:** The 3 Playdate TIDs in the bulk import add schedule data but no scores — those records have no game result value regardless.

**Net new game volume scenarios:**

| Scenario | Total SnapSoccer Competitive Games | Estimated % Not in DB | Net New Games |
|----------|-----------------------------------|-----------------------|---------------|
| Low (bulk import was comprehensive, only recent games new) | 12,000 | 20% | ~2,400 |
| Mid (bulk import captured older events, misses 2 years + any undiscovered TIDs) | 15,000 | 40% | ~6,000 |
| High (bulk import had minimal overlap with SnapSoccer TIDs) | 20,000 | 65% | ~13,000 |

**Recommendation before committing to a volume figure:** Cross-check the existing SincSports games in the DB against known SnapSoccer event names (e.g., `event_name ILIKE '%coastal%soccer%'` or `event_name ILIKE '%snapleag%'`). This query can be run in under 2 minutes and will reveal how many SnapSoccer games are already present. FORGE recommends running this query before the May 17 session.

**Point estimate for planning: ~5,000–7,000 net new games** (mid scenario). This is meaningful new volume — comparable to roughly 2 months of active GotSport daily pipeline output from a single state association.

---

## Section 3: Discovery Runtime Estimate

### Rate and Volume

- Rate limit (per source research): 800–1000ms between requests (safe conservative approach)
- Estimated competitive TIDs to crawl: ~75
- Pages per TID: typically 1 schedule page (`schedule.aspx?tid={TID}`) — some may have division-level sub-pages

**Full TID discovery crawl:**
- Crawl `snapsoccer.com/events/` once: ~2–5 seconds (single page)
- Parse events listing for TID references: < 1 second per request
- Total TID discovery phase: **< 30 seconds** if all TIDs are discoverable from the events page

**Full schedule crawl (after TID list is built):**
- 75 TIDs × 1.0 second per request = **~75 seconds** (1.25 minutes)
- If some TIDs have multiple division sub-pages: add 20–30% overhead → **~90–100 seconds** maximum

**Total estimated runtime per full crawl: ~2 minutes**

### Caching Recommendation

**Strongly recommend: one-time full discovery + incremental delta (nightly).**

Rationale: SnapSoccer runs 10–15 competitive events per year. New events appear infrequently (roughly monthly at peak season). A full TID re-crawl every night is wasteful. The correct architecture:

1. **One-time full discovery** (run during May 17 build session): crawl `snapsoccer.com/events/` exhaustively + check SincSports archives for historical TIDs. Cache the full TID list.
2. **Nightly delta** (production operation): check `snapsoccer.com/events/` for new TIDs not in the cached list. Expected: 0–2 new TIDs per week during active season, 0 during off-season.
3. **Weekly full re-crawl** (optional): re-crawl all known TIDs for updated scores on in-progress events. Most TIDs are historical (complete); only ~2–5 active TIDs at any time.

This approach keeps nightly runtime under 30 seconds in steady state.

---

## Section 4: Pre-Build Session Recommendation

**Recommendation: GO — include SnapSoccer in the May 17 build session.**

Rationale:
- TID count is tractable (~75 active competitive TIDs) and discovery runtime is negligible (~2 minutes)
- Expected net new volume (~5,000–7,000 games) is meaningful for Southeast regional coverage
- Parser architecture is straightforward: HTML scrape of `schedule.aspx`, reusing SincSports parser logic, rate limit is not a practical constraint at this volume
- The one blocking condition — `soccer.sincsports.com` accessibility — is confirmed as a pending browser check, NOT a confirmed blocker. The source research notes the site uses standard HTTP GET with no known auth wall.

**Conditional GO — one condition must be confirmed first:**

> `soccer.sincsports.com` must be confirmed ACCESSIBLE during the SincSports browser check (per `Data Sources/SincSports-Browser-Check-Protocol-2026-04-27.md`) before the build session is scheduled. If the site is BLOCKED or requires authentication, the May 17 session scope changes entirely.

**If the browser check returns ACCESSIBLE:** May 17 build session proceeds as planned.
**If the browser check returns BLOCKED:** Defer SnapSoccer to post-DSS Phase 2. Revisit in September 2026 with a new access path investigation.

---

## Notes

- This estimate is advisory. The browser check must confirm `soccer.sincsports.com` is ACCESSIBLE before the build session begins.
- The 34,961 SincSports bulk import game count should be cross-referenced against known SnapSoccer event names before the May 17 session to sharpen the net new game estimate.
- If the `snapsoccer.com/events/` page is behind a login or returns a sparse listing (< 20 events), FORGE should flag this to SENTINEL — the TID count estimate may need revision and the one-time discovery may require a different approach (e.g., Google search enumeration).

---

## References

- `Data Sources/SnapSoccer Source Research.md` — platform research, TID examples, game volume estimates
- `Data Sources/SincSports-Browser-Check-Protocol-2026-04-27.md` — accessibility gate for this build
- `Data Sources/SnapSoccer-Minimum-Viable-Parser-Spec-2026-04-27.md` — parser design for May 17 session
- `Infrastructure/SnapSoccer — SincSports Parser Reuse Assessment.md` — build approach
- `task-2026-04-27-snapsoccer-tid-coverage-estimate.md` — originating task

*FORGE — 2026-04-27*
