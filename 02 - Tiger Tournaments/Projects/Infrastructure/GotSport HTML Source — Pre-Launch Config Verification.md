---
title: GotSport HTML Source — Pre-Launch Config Verification
tags: [infrastructure, gotsport, html, pipeline, verification, may1, forge]
created: 2026-04-29
updated: 2026-04-29
author: FORGE
status: filed — PARTIAL (deployment inclusion requires Presten config check)
task: task-2026-04-29-gotsport-html-config-pre-launch-verify
---

# GotSport HTML Source — Pre-Launch Config Verification

```
Filed by:     FORGE
Date:         2026-04-29
Due:          2026-04-30 EOD
Assessment:   PARTIAL — vault-derivable fields complete; run-daily.sh inclusion requires Presten verification
```

> [!warning] Architecture Note
> The GotSport HTML scraper (`crawl-gotsport-v2.js`) is **event-centric**, not org-ID-based. It does not use org-IDs in its config. The task's Section 1 org-ID fields are therefore not applicable; this document substitutes equivalent config fields for the HTML scraper's actual architecture.

---

## Section 1 — Config Status

| Field | Value |
|-------|-------|
| Source identifier in pipeline | `gotsport_html` |
| Script file | `crawl-gotsport-v2.js` |
| Discovery method | Event-centric HTML parsing (NOT org-ID-based) |
| Number of org-IDs configured | **N/A — HTML scraper uses event IDs, not org IDs** |
| Event ID discovery method | Runtime crawl of `system.gotsport.com` schedule pages |
| Reference test event | Dallas Cup 2026 (event 45488) — 889 games verified |
| Last vault-verified date | 2026-04-21 (GotSport HTML Scraper.md created) |
| Source status | **ACTIVE** (confirmed in Per-Source Forecast, Official Source Baseline, Pipeline Week 1 Monitoring Log — all filed April 2026) |
| Inclusion in run-daily.sh | **REQUIRES PRESTEN CHECK** — see Section 2 |

**Source ID format:** `gs_{eventId}_{matchNum}` (distinct from API's `gs_api_{eventId}_{matchId}`)

**Priority:** 2 (supplementary to API; dedup handled per `Dedup Strategy.md`)

---

## Section 2 — Known Risks

### Risk 1: run-daily.sh Inclusion Unconfirmed (MEDIUM)

FORGE cannot verify without codebase access whether `crawl-gotsport-v2.js` is explicitly included in `run-daily.sh` (the May 1 cron entry point). All planning documents assume it is active, and it appears in source forecasts with 10–50 games/day expectation. However, no vault document contains a `run-daily.sh` manifest confirming inclusion.

**Verification query for Presten (5 minutes):**
```bash
grep -i "gotsport.v2\|crawl-gotsport-v2\|gotsport_html" /path/to/run-daily.sh
```
Expected: one matching line. If not found: scraper is excluded from May 1 run.

### Risk 2: Event Discovery Scope Unknown

The HTML scraper crawls events it discovers at runtime — the event pool changes with the GotSport calendar. FORGE has no vault document specifying which event IDs are in scope at any given time. This is by design (event-centric architecture), but means the 10–50 game/day forecast has lower confidence than the API source.

- **Low end (10 games/day):** Only a small number of currently-active events in scope
- **High end (50 games/day):** Spring season in full swing with many events active
- **May 1 = Thursday (weekday):** Expect toward low end of range; spring Saturday events drive higher volume

### Risk 3: No Known Inactive or Stale Events

Since event discovery is runtime-determined, there is no static list of "stale event IDs" analogous to stale team IDs in the API scraper. The scraper either finds active events or it doesn't — staleness is not a configuration risk for this source type.

### Risk 4: 3-Layer Filter False Positives

The scraper applies HARD_SKIP keywords, page content scan, and U7–U19 youth age verification. If filter thresholds are misconfigured, events could be silently dropped. This is a pre-existing behavior with no reported issues in vault documents; flagged as low probability.

---

## Section 3 — Pre-Launch Assessment

**PARTIAL**

gotsport_html is active and present in all May 1 planning documents. Its architecture (event-centric, no org-IDs) is sound and well-documented. The scraper has been validated against a 889-game test event (Dallas Cup 2026). All forecast and monitoring documents correctly list it as a Stage 1 source.

The single unresolved item is **run-daily.sh inclusion confirmation**, which requires Presten to run a one-line grep (see Section 2, Risk 1). If the grep confirms inclusion, assessment upgrades to **VERIFIED**.

**Launch can proceed with monitoring** unless grep returns no match (in which case escalate to SENTINEL before May 1).

---

## Section 4 — May 1 Monitoring Threshold

Pre-filled for the May 1 monitoring log based on known source characteristics:

| Source | GREEN (healthy) | YELLOW (investigate) | RED (likely failed) | Flag if |
|--------|----------------|---------------------|-------------------|---------|
| gotsport_html | ≥ 5 new games | 1–4 new games | 0 new games | < 5 or > 100 |

**Lower bound rationale:** gotsport_html is supplementary and event-centric. A single active spring event could produce 5–20 new games per daily run. Zero games on May 1 (a weekday) is possible if no events have active schedules that day — but should be flagged for investigation, not treated as automatic failure.

**Note:** If run-daily.sh inclusion is unconfirmed before May 1, treat any zero-games result as a probable configuration exclusion, not just an empty-event day.

---

## Presten Action Required Before May 1

1. Run `grep -i "gotsport.v2\|crawl-gotsport-v2\|gotsport_html" /path/to/run-daily.sh`
2. Share result with FORGE
3. If found: FORGE upgrades this document to **VERIFIED** and closes task
4. If not found: FORGE escalates to SENTINEL; scraper must be added to run-daily.sh before May 1

**Time required from Presten:** < 5 minutes.

---

*FORGE — 2026-04-29*
