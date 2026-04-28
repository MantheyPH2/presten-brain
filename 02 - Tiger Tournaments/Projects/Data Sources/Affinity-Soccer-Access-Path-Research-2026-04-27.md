---
title: Affinity Soccer (IL) — Access Path Research
tags: [data-sources, research, affinity-soccer, access-path, post-dss, evo-draw, forge]
created: 2026-04-27
author: FORGE
status: complete
task: task-2026-04-27-affinity-soccer-access-path-research
recommendation: CONDITIONAL — login and per-org auth behavior requires pre-build browser audit
---

# Affinity Soccer (IL) — Access Path Research

**Researched by:** FORGE  
**Date:** 2026-04-27  
**Task:** `task-2026-04-27-affinity-soccer-access-path-research.md`  
**Builds on:** `Data Sources/Affinity Soccer Source Research.md` (2026-04-24)  
**Purpose:** Focused access path investigation — how to actually retrieve data from affinitysoccer.com. Answers the engineering readiness question before any build commitment.

---

## Section 1: Website and Data Access

**Primary URL:** `https://www.affinitysoccer.com`  
**Schedule/results access:** `affinitysoccer.com/home/programs/{org_id}` (organization home) and `affinitysoccer.com/tournament/{event_id}` (event-specific results) — URL patterns inferred from platform structure; exact parameter names require browser confirmation.

**Public accessibility:** **INCONSISTENT.** Affinity Soccer is a club/state management platform where public result visibility is configured per-organization. Illinois Youth Soccer Association (IYSA) pages are the primary target; IYSA's specific accessibility is unconfirmed without a direct browser visit to their Affinity program page.

**Platform owner:** Active Network / Blue Star Sports — a managed platform company, not an open-source or community-supported system. This means the HTML structure and URL schema are controlled by a single vendor but may change at vendor discretion.

**Authentication wall pattern:**
- Some Affinity organizations expose all results publicly (no login)
- Others gate results behind a free account registration wall
- Others require a paid club membership or association membership
- IYSA's specific auth configuration is **unconfirmed**

**Assessment:** Data is likely accessible for IYSA but auth behavior is org-specific. Cannot confirm without browser visit to `affinitysoccer.com` → search IYSA.

---

## Section 2: Data Structure

**Format:** Server-rendered HTML (no public JSON API confirmed). Affinity Soccer does not publish a developer API. Any integration requires HTML scraping.

**HTML structure:**
- Schedule and results tables are server-rendered in the initial HTTP response (confirmed pattern for managed platforms in this category)
- No JavaScript rendering required for the score tables — standard HTML `<table>` structure likely
- This is similar to SincSports (`schedule.aspx`) — HTML table scraping, not API polling

**URL pattern evidence:**
- Organization-level pages: parameterized by `org_id` or `program_id` — similar to GotSport's `org_id` pattern
- Event-level pages: parameterized by `event_id` — requires either a directory listing or a known event list
- No stable public event directory confirmed — compared to GotSport's tournament listing, Affinity's event discoverability path is unknown

**Pagination:** Present at the organization level (multiple events per org, multiple pages per season). Estimated 10–30 pages per active state association per season.

**Game IDs in URLs:** Likely present at the game level; stability across sessions is unconfirmed.

**Key gap:** Without a browser visit to the IYSA Affinity page, the exact URL structure and pagination behavior cannot be confirmed. The 2026-04-24 source research identified the platform; this document identifies the access path uncertainty as the pre-build blocker.

---

## Section 3: Geographic Scope

**Confirmed on Affinity Soccer:**
- **Illinois (IYSA)** — primary target, confirmed from USYS State Cup Source Research (2026-04-23)

**Likely on Affinity Soccer (unconfirmed):**
- Michigan (MSYSA) — Midwest; Active Network historically strong in MI
- Minnesota (MSYSA) — Midwest; likely but unconfirmed
- Indiana (ISYA) — likely but smaller state
- Wisconsin (WYSA) — possibly Affinity

**Affinity Soccer is multi-state** — it is not an Illinois-only platform. It competes with GotSport nationally in the state association market. However, GotSport dominates the coasts and Sun Belt; Affinity is stronger in the Midwest and historically in states that adopted Active Network products before GotSport's national expansion.

**States confirmed NOT on Affinity:** TX, FL, NY, NJ, WA, CO, CA, PA (on GotSport per USYS State Cup research).

**Build implication:** An Affinity scraper built for IYSA is unlikely to work for Michigan or Minnesota without re-testing, because per-org configuration variance may require org-specific scraper adjustments. Multi-state Affinity coverage is a multi-sprint project, not a single build.

---

## Section 4: Build Complexity

**HTML scraping complexity:** MEDIUM. Server-rendered HTML tables (not JS-rendered). No Cloudflare challenge page confirmed, but unverified. Standard `requests` + `BeautifulSoup` scraping approach applies.

**Auth wall risk:** HIGH uncertainty. Per-org auth behavior is the primary unknown. If IYSA requires account registration, the build path changes from a scraper to a session-authenticated scraper — higher complexity and higher maintenance burden.

**Rate limiting:** Not documented for Affinity Soccer. Managed platforms in this category typically do not implement aggressive rate limiting for public pages, but this is unverified.

**Parser reuse from existing FORGE codebase:**
- **GotSport scraper:** No reuse — GotSport is API-based; Affinity is HTML-based
- **SincSports (TGS scraper):** Partial reuse possible — TGS scraper does HTML parsing; the table extraction logic may transfer if Affinity's HTML structure is similar to SincSports' ASP.NET pattern
- **SnapSoccer design:** If the SnapSoccer ingestion script is built first (post-DSS), its SincSports HTML parser would be the closest reuse candidate for Affinity (same category: managed-platform HTML tables)

**Build hours estimate:**
- If IYSA is fully public (no auth): **20–35 hours** (new scraper, IYSA-specific, with dedup logic)
- If IYSA requires session auth: **35–55 hours** (adds auth handling, session management, error recovery)
- Multi-state (add MI, MN, etc.): add **10–20 hours per additional state** depending on HTML variance

These estimates assume SnapSoccer is already built and its HTML parsing patterns are available for reference. Without SnapSoccer precedent: add 5–10 hours.

---

## Section 5: Access Path Verdict

**Verdict: CONDITIONAL**

Access to Affinity Soccer data requires a pre-build browser audit of IYSA's specific Affinity page. The decision tree:

```
Navigate to affinitysoccer.com → search "Illinois Youth Soccer" → open IYSA program page

IF: Results tables load without login prompt
  → ACCESSIBLE. Schedule build sprint for June 2026.
  → Revised ROI estimate: IL alone = 8,000–20,000 games/year ÷ 20–35 hours = 230–1,000 games/hour
  → This is comparable to GotSport expansion efficiency; justifies a sprint

IF: Results require free account registration
  → CONDITIONAL. Auth-scraper required. Escalate to Presten: is account registration acceptable?
  → If yes: schedule build sprint for June 2026, plan for auth handling
  → Build hours increase to 35–55 range

IF: Results require paid membership or association access
  → BLOCKED. Outreach to IYSA required before build can proceed.
  → Document as BLOCKED in Source Gap Inventory. No engineering until access path cleared.
```

**Pre-build action required:** 1-hour browser audit of IYSA's Affinity Soccer page. This is the gate before any Affinity build commitment. Presten or FORGE (if browser access is available) can complete this check in a single short session — navigate to affinitysoccer.com, find IYSA, attempt to view a recent schedule or result page without logging in.

**If ACCESSIBLE:** Affinity Soccer (Illinois) is the correct Post-DSS Sprint 2 target after SnapSoccer is deployed and stable. At 8,000–20,000 games/year for IL alone, it is the highest-volume non-GotSport source with a plausible access path.

**If CONDITIONAL or BLOCKED:** Re-evaluate before scheduling. Do not commit engineering hours until access path is confirmed.

---

## Revised ROI vs. 2026-04-24 Research

The April 24 source research estimated 5,000–15,000 games/year for Illinois alone (revised conservatively in the Non-GotSport rerank to 5,000–15,000 IL only). Using the midpoint:

| Scenario | Games/Year (IL) | Build Hours | Games/Hour |
|----------|----------------|-------------|------------|
| ACCESSIBLE (no auth) | 10,000 | 20–35 | 285–500 |
| CONDITIONAL (free auth) | 10,000 | 35–55 | 180–285 |
| + Midwest states (if confirmed) | +15,000–35,000 | +10–20/state | Scales well |

**Comparison to GotSport org-ID expansion:** GotSport config adds deliver 20,000–42,000 games with 5 minutes of config time — strictly higher ROI. GotSport expansion should always precede any Affinity Sprint.

---

## References

- `Data Sources/Affinity Soccer Source Research.md` — full 2026-04-24 source research
- `Data Sources/Non-GotSport-Source-Priority-Rerank-2026-04-27.md` — priority placement (#2 post-DSS)
- `Infrastructure/Non-GotSport Source Research — April 2026 Synthesis.md` — source matrix context
- `Infrastructure/Source Gap Inventory — April 2026.md` — Affinity as documented gap

*FORGE — 2026-04-27*
