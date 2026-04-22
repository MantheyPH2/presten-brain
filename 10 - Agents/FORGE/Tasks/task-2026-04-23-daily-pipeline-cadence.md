---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-23
status: completed
completed: 2026-04-23
priority: high
---

# Task: Scope Moving from Overnight Batch to Daily/Hourly Pipeline

## Context

USA Rank updates rankings daily. Evo Draw runs an overnight batch (`run-overnight.sh`) on a single machine. This is one of the clearest feature gaps identified in our competitive intel. During peak showcase season (March–June), stale rankings are a real product liability — coaches and parents are making decisions on data that's 12–24 hours old.

The current pipeline (12-hour full crawl of 107,528 GotSport team IDs at 1200ms delay, 3 workers) cannot simply be run more frequently without a redesign.

Read before starting:
- `02 - Tiger Tournaments/Projects/Knowledge/USA Rank Competitive Intel.md` — "How They Rank Teams" section, update frequency
- `02 - Tiger Tournaments/Projects/Pipeline/Data Pipeline.md` — all 9 steps, current architecture
- `02 - Tiger Tournaments/Projects/Scrapers/GotSport API Scraper.md` — crawl mechanics, 107,528 IDs, 1200ms delay, 3 workers
- `02 - Tiger Tournaments/Projects/Infrastructure/Database Schema.md` — schema-adaptive insertion
- `02 - Tiger Tournaments/Projects/Infrastructure/Server and Frontend.md` — Node.js server, current endpoints
- `02 - Tiger Tournaments/Projects/Knowledge/USA Rank Competitive Intel.md` — Strategic Insights #12 on pipeline bottleneck

## What You Need to Scope

### 1. Crawl Time Analysis
- Current: 107,528 team IDs × 1200ms ÷ 3 workers = ~12 hours. Verify this math.
- What is the bottleneck — rate limit (1200ms delay), worker count, or DB write speed?
- Is the 1200ms delay self-imposed or required to avoid blocks? What happens at 600ms, 300ms?
- What fraction of the 107,528 IDs had new games in the last 30 days? (Active vs inactive teams)

### 2. Incremental Crawl Design
- Instead of crawling all 107,528 IDs every run, identify which teams have had recent activity.
- Design an "active team" filter: teams with games in the last 90 days (or with upcoming events).
- Estimate: if only 20% of teams are active in any 30-day window, a daily crawl needs ~21,500 IDs → ~2.5 hours at current settings.
- Design the query to identify active team IDs: `SELECT DISTINCT gotsport_id FROM games WHERE game_date > NOW() - INTERVAL '90 days'`

### 3. Pipeline Step Dependency Analysis
- Which of the 9 pipeline steps must run in full every time?
- Which steps can be run incrementally (only on new/changed data)?
- Which steps are CPU-bound vs IO-bound?
- Can ranking recomputation (Steps 6–9) run independently of data ingestion (Steps 1–5)?

### 4. Scheduling Architecture Options
Evaluate three options:

**Option A: Smarter Overnight Batch**
- Filter to active teams only, reduce crawl time to 2–3 hours
- Run at 2 AM daily
- Pros: Minimal code change. Cons: Still 24-hour lag.

**Option B: Segmented Daily Runs**
- Split 107,528 IDs into 7 segments, crawl one segment per day (rotating)
- Each team gets updated weekly, rankings recomputed daily
- Pros: Even load. Cons: Some teams are stale for up to 7 days.

**Option C: Event-Triggered Incremental Updates**
- Monitor GotSport for newly completed events (via the events endpoint)
- When a new event is detected, crawl only its participating teams
- Rankings recompute after each event batch
- Pros: Near-real-time for active events. Cons: Requires event monitoring script.

For each option: estimate implementation effort (hours), latency improvement, and infrastructure risk.

### 5. Infrastructure Requirements
- Does the current single-machine setup support daily runs? (cron job feasibility)
- What monitoring is needed to detect failed pipeline runs?
- What does a failed incremental run do to data integrity? (Partial updates, stale rankings)
- Is there a risk of DB locks or write conflicts if pipeline runs overlap?

## What Done Looks Like

Produce a scoping document at:
`02 - Tiger Tournaments/Projects/Product & Planning/Daily Pipeline Cadence Scope.md`

The document must include:
1. Current crawl time breakdown (verified math, bottleneck identified)
2. Active team analysis — estimated count of teams active in last 90 days
3. Incremental crawl design — SQL query + logic for active ID selection
4. Comparison table of all 3 options (latency, effort, risk)
5. Recommended option with justification
6. Implementation plan for recommended option — step by step, in order
7. Cron schedule for recommended option (crontab entry)
8. Rollback plan — how to revert to overnight batch if incremental has issues

## Success Criteria

- Scoping document written with all 8 sections
- A clear recommendation is made (not "it depends")
- Presten can implement the recommended option in a single focused work session
- The plan does not require new infrastructure — works on existing machine and PostgreSQL setup
- Timeline estimate: daily updates achievable before DSS (May 16, 2026)
