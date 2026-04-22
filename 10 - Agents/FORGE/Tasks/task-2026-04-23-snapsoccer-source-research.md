---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-23
status: completed
completed: 2026-04-23
priority: high
---

# Task: Research Adding SnapSoccer as a Data Source

## Context

USA Rank (our primary competitor) ingests from 400+ sources and updates daily. We have 7. SnapSoccer is a confirmed USA Rank data source — their event pages repeatedly reference "SnapSoccer Fall Playdates" and "Spring Playdates," indicating SnapSoccer runs structured youth soccer events with published results.

Read before starting:
- `02 - Tiger Tournaments/Projects/Knowledge/USA Rank Competitive Intel.md` — source breakdown section
- `02 - Tiger Tournaments/Projects/Data Sources/Data Sources Overview.md` — current source architecture
- `02 - Tiger Tournaments/Projects/Pipeline/Data Pipeline.md` — step 0 scraper requirements

## What You Need to Research

### 1. SnapSoccer Public Footprint
- What is SnapSoccer's website/domain?
- Do they publish game results publicly? What format (HTML, JSON, API)?
- What age groups and events do they run? What regions?
- Do they require auth to view results, or are results public-facing?
- Can event pages be crawled without credentials?

### 2. Data Availability Assessment
- Are scores, teams, dates, and event names parseable from their pages?
- Does SnapSoccer assign stable team IDs or event IDs?
- Are results paginated? How many events/games appear to exist?
- Do their team names follow any standard format that would help with our `team_merges` matching?

### 3. Scraper Feasibility
- Is the site protected by Cloudflare or similar (like Sinc Sports)? Test with simple curl/fetch.
- What HTTP structure does the results page use — server-rendered HTML or client-side JSON?
- Estimate pages/requests needed for a full crawl. What rate limit is safe?
- Which existing scraper is the best template: GotSport HTML Scraper or MLS NEXT crawler?

### 4. Dedup Considerations
- Are SnapSoccer events likely to overlap with GotSport events (same games already in our DB)?
- What source priority should SnapSoccer get? (Current range: 1–8, GotSport is 1/2)
- What `source_id` format should be used? (Pattern: `snapsoccer_{eventId}_{gameId}`)

## What Done Looks Like

Produce a research memo at:
`02 - Tiger Tournaments/Projects/Data Sources/SnapSoccer Source Research.md`

The memo must answer:
1. Go / No-Go recommendation with justification
2. Estimated game volume (order of magnitude)
3. Scraper approach (HTML vs API, auth required or not)
4. Dedup risk level (Low / Medium / High overlap with GotSport)
5. Estimated dev effort to build the scraper (hours)
6. Proposed source priority number
7. Draft `source_id` format string

If Go: include a skeleton scraper outline — key URLs to hit, data fields to extract, and insertion logic stub.

## Success Criteria

- Research memo is written and contains a clear Go/No-Go
- If Go: Presten has enough detail to start building the scraper in one session
- If No-Go: the reason is documented so we don't re-investigate later
