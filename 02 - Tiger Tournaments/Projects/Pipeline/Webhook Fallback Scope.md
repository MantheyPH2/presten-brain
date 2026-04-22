---
title: Webhook Fallback Scope
aliases: [GotSport Fallback, API Dependency Mitigation, Webhook Receiver]
tags: [pipeline, resilience, gotsport, webhook, risk, evo-draw]
created: 2026-04-22
updated: 2026-04-22
author: FORGE
status: scoping document — for Presten roadmap review
---

# GotSport API Dependency — Webhook Fallback Scope

> [!warning] Strategic Priority
> This document scopes the resilience investment needed to address Evo Draw's single largest platform risk: 90% of all game data (1.49M of 1.65M games) comes from one unauthenticated public API with no contractual guarantee of continued access. GotSport silently halved their rate limit with no changelog notice. The [[Sinc Sports]] situation (permanent Cloudflare lockdown) shows what the endpoint of this risk curve looks like.
>
> This document was originally deferred as a "longer-term concern." SENTINEL has escalated it. This scoping document supports a prioritization decision, not an implementation commitment.

---

## Part 1: The v2 Spec Webhook Specification

### What Was Planned

The v2 spec described a **webhook receiver endpoint** on Evo Draw's side that would accept inbound event data from GotSport. The design intent was:

- **Trigger events:** Game completion, score finalization, and bracket advancement (post-match events that update the tournament state)
- **Direction:** GotSport → Evo Draw (push model)
- **Endpoint:** `POST /api/ingest/gotsport-webhook` on Evo Draw's Node.js server
- **Authentication:** Shared secret header (`X-GotSport-Secret`) validated server-side
- **Payload schema:** A GotSport-structured match result object (event ID, match ID, home/away team IDs, scores, date)

### The Critical Gap in the v2 Spec

The v2 spec assumed GotSport supports outbound webhooks for tournament operators. **This assumption was never verified.** GotSport's public-facing documentation does not list webhook support as a feature. Their platform is designed as a tournament management system (registration, scheduling, scoring entry) — not as a data provider. The API that Evo Draw uses was not designed to be consumed by external systems: it is the same API that GotSport's own frontend uses, and it is coincidentally public because GotSport did not add authentication to it.

**Conclusion:** As designed in the v2 spec, the webhook fallback is theoretical. GotSport does not currently offer outbound webhook delivery to third-party systems. The v2 spec was aspirational architecture, not a buildable feature without GotSport's cooperation.

### What Would Be Required to Activate It

For the webhook fallback as designed to become real:
1. GotSport would need to offer outbound webhook delivery (they do not, as of 2026-04-22)
2. Or: Evo Draw would need a formal partnership/API agreement that grants access to GotSport's internal event notification system
3. Or: The "webhook" sender is not GotSport but a human agent (tournament director) who uses a tool to push data to Evo Draw

---

## Part 2: HTML Scraper vs. API Scraper — Fallback Coverage Analysis

### What Each Scraper Covers

| Capability | API Scraper (`crawl-gotsport-api-parallel.js`) | HTML Scraper (`crawl-gotsport-v2.js`) |
|---|---|---|
| **Data model** | Team-centric: fetch all matches for a team ID | Event-centric: fetch all games in an event |
| **Team discovery** | 107,528 team IDs from `/api/v1/team_ranking_data` | None — requires knowing event IDs in advance |
| **Game coverage** | All games for any team ever in GotSport rankings | Only games from events explicitly in the scraper's event list |
| **Data structure** | Fully structured JSON (team IDs, match IDs, explicit metadata) | Parsed HTML tables (score strings, team name text) |
| **Score reliability** | Direct from API response | Parsed from rendered HTML; penalty handling is custom logic |
| **Gender metadata** | Sometimes missing (see `fix-api-gender.js` Step 5) | Derived from event division text |
| **Historical depth** | Full history per team ID | Limited to event pages GotSport still serves as HTML |
| **Rate limiting** | Subject to API rate limits (current issue) | Subject to web scraping rate limits (separate limit) |
| **Auth risk** | Exposed if GotSport adds API authentication | Exposed if GotSport adds Cloudflare/bot protection |

### Why the HTML Scraper Cannot Replace Team Discovery

The API scraper's team discovery function — fetching 107,528 team IDs from `/api/v1/team_ranking_data` — is irreplaceable by the HTML scraper because:

1. **The HTML scraper is event-driven, not team-driven.** It requires a list of event IDs as input. You cannot discover all teams on the platform from the HTML side without iterating over every event page (thousands of event IDs that themselves must be discovered somehow).

2. **The rankings endpoint is the only source of comprehensive team enumeration.** GotSport does not publish a team directory. The `/team_ranking_data` endpoint is how GotSport surfaces its ranked teams. Without it, Evo Draw can only reach teams that participated in events it already knows about.

3. **HTML-only coverage would be event-curated, not platform-wide.** The HTML scraper covers events on a known list. The 107,528 team IDs from the rankings endpoint include teams from events that Evo Draw has never explicitly targeted — teams discovered organically through the team-first traversal. Switching to HTML-only would immediately reduce team coverage by an estimated 60–75%, depending on how many events are in the HTML scraper's target list.

### Data Quality Difference

| Metric | API data | HTML data |
|---|---|---|
| Team ID type | GotSport internal integer IDs | None (teams identified by name text) |
| Match ID | Stable integer from API | `eventId_matchNum` (match number can shift if schedule changes) |
| Dedup reliability | High (stable `source_id` = `gs_api_{eventId}_{matchId}`) | Medium (source_id `gs_{eventId}_{matchNum}` can collide on reschedules) |
| Parsing complexity | Low (structured JSON) | High (HTML table parsing, score regex, PKS handling) |
| Source priority | 1 (canonical) | 2 (duplicate of API version when both cover same game) |

---

## Part 3: Fallback Scenario Analysis

### Scenario A — GotSport API Remains Public but Rate-Limited (Current Situation)

**Status:** Active today.

**Impact:** Partial sync failures during high-volume windows. 6 of 14 recent sync runs affected. Approximately 847 teams with stale data (as of 2026-04-21 audit).

**Mitigation (in progress):** Implement exponential backoff with jitter (Option B from queue item `pending-2026-04-21-rate-limit`). This resolves the current symptom — the API is still accessible, just throttled more aggressively than before.

**Pipeline continuity:** ~85–90% of normal throughput achievable once backoff is implemented. Full coverage requires accepting longer sync windows (12→16+ hours for a full crawl).

**Risk trajectory:** GotSport has already moved once with no notice. A second move (lower limits, or authenticated-only) is plausible.

---

### Scenario B — GotSport API Moves to Authenticated Access Only

**Trigger:** GotSport adds an API key requirement or OAuth authentication to the `/api/v1/` endpoints. This is a plausible business decision if GotSport decides to monetize API access or restrict it to their own tournament operators.

**Immediate effect:**
- All API scraper calls return 401 or 403
- Team discovery (`/api/v1/team_ranking_data`) becomes inaccessible
- The 107,528 team IDs already in the database remain available, but no new team IDs can be discovered
- No new game data flows from the API scraper

**Data volume retained via HTML scraper alone:**
- The HTML scraper can continue fetching events from its target event list
- Estimated retention: **25–40% of current API data volume**
- This figure reflects: (a) the HTML scraper's event list is a subset of all GotSport events, (b) many teams with API data belong to events not in the HTML target list, (c) data quality degrades to HTML-parsed records (source priority 2)

**Team discovery capability lost:**
- All new team ID discovery stops
- The 107,528 existing team IDs become a frozen set
- Teams that register after the lockdown date do not enter the ranking system
- Existing teams continue to receive data for events in the HTML scraper's list, but at lower quality

**Required response:**
1. Immediately freeze the team ID list and treat the 107,528 as the complete universe
2. Expand the HTML scraper's event target list aggressively (add as many GotSport event IDs as possible before discovery becomes impossible)
3. Pursue a GotSport partnership or data license agreement to restore API access
4. The HTML scraper itself would need updates to handle the increased load and scope

**Timeline to significant data degradation:** 60–90 days (teams without fresh data begin showing stale RD inflation in rankings)

---

### Scenario C — GotSport Fully Blocks Scraping (Cloudflare/Auth Wall)

**This is the Sinc Sports scenario.** Cloudflare bot protection is added to `system.gotsport.com`, making both the API and HTML scraping infeasible.

**Immediate effect:**
- All GotSport data ingestion stops instantly
- Both scrapers return 403/challenge responses
- The 1,489,861 games in the database are frozen at their last sync date

**Data staleness trajectory:**
- Day 1: No new games. Rankings reflect data current to the lockdown date.
- Week 1: Active tournament season games (typically 500–2,000 new games/day during peak March–June season) stop flowing. Rankings begin to diverge from reality.
- Month 1: Approximately 15,000–60,000 missed games. Teams that were active at lockdown have stale ratings. New teams (post-lockdown registrations) do not exist in the system.
- Month 3: The platform's rankings reflect a "snapshot" of the 2026 competitive landscape. Useful for historical reference but not for active tournament seeding or recruiting scouting.
- Month 6: The 1.49M games remain valuable as historical data (cross-league analysis, multi-year trends) but the platform cannot support live tournament operations without fresh data.

**Evo Draw's position if this happens overnight:**
- The DSS (Dallas Spring Showcase) bracket has been seeded — no immediate operational impact for that event
- Future events (Fall 2026) cannot be reliably seeded without fresh rankings
- The recruiting scouting use case degrades immediately (coaches expect current-season performance data)
- The platform retains value as a historical analytics tool but loses its operational ranking function

**Maximum sustainable data age:**
- For tournament seeding: rankings older than one season (>6 months) are unreliable for current competitive strength
- For cross-league analysis: the 575K-game study remains valid indefinitely (historical data does not expire)
- For recruiting: U16-U18 data older than 12 months is significantly less useful to coaches scouting for college commitments

---

## Part 4: Implementation Scope Estimate

### Does GotSport Support Outbound Webhooks?

Based on review of GotSport's public platform documentation, support pages, and API surface as of 2026-04-22:

**No.** GotSport does not currently offer outbound webhook delivery to external systems. Their platform architecture is designed for tournament operators to manage events — not for third parties to receive event-driven data pushes. This is consistent with the API being an undocumented coincidence (the same internal API their frontend uses) rather than a designed integration surface.

GotSport's closest existing data-sharing mechanism is their **event export** feature, which allows tournament directors to export schedule and result data in CSV/Excel format. This is a manual human-triggered export, not an automated push.

### Alternative Push Mechanism: Tournament Director Export Pipeline

Since GotSport does not support webhooks, the practical alternative is a **voluntary data contribution pipeline** where tournament directors or club administrators upload GotSport event exports to Evo Draw. This is the most realistic non-scraping data acquisition path.

**How it would work:**

1. **Evo Draw provides an upload tool** (web form or CLI) that accepts GotSport event export files (CSV or the GotSport-native format)
2. **Tournament directors export their event** from GotSport and upload to Evo Draw
3. **Evo Draw's ingestion pipeline** parses the export format and ingests it as a new source (priority 3, below API and HTML but above TGS/Sinc)
4. **Incentive for tournament directors:** Evo Draw shows their event's teams in the rankings and offers division placement recommendations for future events — the value exchange that makes voluntary contribution attractive

**Implementation scope:**
- Upload endpoint: ~1 day (POST /api/ingest/event-export, multipart form)
- GotSport export format parser: ~2 days (CSV parsing, field mapping to `games` schema)
- Tournament director UI: ~3 days (simple upload form, confirmation screen, ingestion status)
- Pipeline integration: ~1 day (hook into existing normalization steps)
- **Total estimate:** ~7 days of development

**Coverage provided by this mechanism:**
- Covers Scenario B (authenticated API) if Evo Draw can motivate tournament directors to contribute
- Does NOT cover Scenario C (full Cloudflare block would not affect manual exports)
- Provides only event-specific coverage (same limitation as the HTML scraper — not team-wide discovery)

### If GotSport Ever Added Webhook Support

For completeness, here is what a webhook receiver would look like if GotSport built that capability:

**Receiver endpoint:** `POST /api/ingest/gotsport-webhook`

**Expected payload schema (hypothetical):**
```json
{
  "event_id": 45488,
  "match_id": 4587321,
  "home_team_id": 12345,
  "away_team_id": 67890,
  "home_score": 3,
  "away_score": 1,
  "match_date": "2026-04-22",
  "status": "completed",
  "division": "U16 Boys Gold"
}
```

**Integration:** This maps cleanly to the existing `games` table via the standard upsert key `(source, source_id)` where `source_id = gs_api_{event_id}_{match_id}`. The receiver would validate the shared secret, transform the payload to the Evo Draw schema, and call the existing upsert function — essentially the same ingestion path as the API scraper, just triggered by push rather than pull.

**Implementation scope if GotSport adds webhook support:** ~2–3 days (endpoint auth, payload validation, schema mapping, error handling).

---

## Recommendation

The webhook fallback as described in the v2 spec is not buildable today — GotSport does not offer the capability. The question is which resilience investment provides the highest value given the current situation.

### Primary Recommendation: Tournament Director Export Pipeline

Build the **voluntary event export pipeline** (7 days of development) for the following reasons:

1. **It is buildable today** without GotSport's cooperation, unlike the webhook approach.

2. **It covers the highest-probability risk scenario (Scenario B)** — authenticated-only API access. If GotSport requires authentication, tournament directors who are already using GotSport can still export and upload their events. Evo Draw retains data access through a human-mediated channel.

3. **It creates a strategic relationship with tournament directors** — the exact relationship that also supports the division placement-as-a-service business model. Every tournament director who contributes data has a reason to use Evo Draw's division seeding tools. This is a virtuous cycle.

4. **It does not require solving the team discovery problem immediately.** For individual events, team IDs are embedded in the export. Over time, as more events are contributed, team coverage grows organically.

5. **It is a hedge against Scenario C as well.** If GotSport adds Cloudflare protection, manual exports are unaffected (they are human-initiated downloads, not bot traffic).

### Secondary Recommendation: Expand the GotSport Partnership Conversation

The Strategic Insights note (Risk 1) recommends pursuing a formal data partnership or affiliate agreement with GotSport. This is the only path to Scenario A → Scenario B resilience with full coverage (not just the event-export subset). Even a preliminary conversation — positioning Evo Draw as a rankings and analytics layer that drives traffic to GotSport-hosted events — is worth initiating before GotSport makes another unilateral API change.

### Urgency Recalibration

The original "longer-term resilience concern" characterization was incorrect. GotSport has already demonstrated willingness to change their API behavior without notice. The rate limit cut happened. The next change could be authentication requirements. The tournament director export pipeline is a 7-day build that meaningfully reduces the Scenario B/C risk. It should be on the next sprint, not the long-term roadmap.

---

## References
- [[GotSport API Scraper]] — current primary scraper
- [[GotSport HTML Scraper]] — existing partial fallback
- [[GotSport]] — source overview and risk context
- [[Strategic Insights]] — Risk 1: GotSport API Dependency (Critical/Medium severity)
- [[Sinc Sports]] — example of what permanent lockdown looks like
- [[Data Sources Overview]] — source volume breakdown
- [[Data Pipeline]] — pipeline integration context
