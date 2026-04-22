---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-22
status: completed
priority: medium
---

# Task: Scope the Webhook Fallback — v2 Spec Item

## Context

Your pipeline review noted that the webhook fallback listed in the v2 spec was never implemented. You characterized this as "a longer-term resilience concern" that is "not an immediate blocker."

SENTINEL disagrees with that urgency characterization. GotSport just silently cut their rate limit in half with no changelog notice. The HTML scraper is a partial fallback but it is event-centric, not team-centric — it cannot replicate the team discovery step that found 107,528 team IDs from the rankings endpoint. If GotSport adds authentication requirements or fully blocks the public API (as Sinc Sports was blocked by Cloudflare), Evo Draw loses 90% of its data pipeline with no fallback path.

The webhook fallback needs to be scoped so Presten can evaluate it as a roadmap item. This is not an ask to build it now — it is an ask to define what "building it" would mean so it can be prioritized correctly.

## Required Work

### Step 1: Re-read the v2 Spec Webhook Specification
Pull the v2 spec section that describes the webhook fallback. Document:
- What trigger events it was designed to receive (game completion? score update? roster change?)
- What GotSport-side action would be required to activate it (do they support webhooks, or was this theoretical?)
- What endpoint on Evo Draw's side would receive the data

### Step 2: Distinguish From the HTML Scraper Fallback
The GotSport HTML scraper (`crawl-gotsport-html.js`) is an existing partial fallback. Clarify:
- What the HTML scraper covers that the API scraper does NOT cover and vice versa
- Why the HTML scraper cannot replace the API scraper's team discovery function (107,528 team IDs from the rankings endpoint)
- What the data quality difference is between HTML-scraped and API-scraped game records (structured vs parsed)

### Step 3: Define Three Fallback Scenarios
Document what Evo Draw's data pipeline looks like under each scenario:

**Scenario A: GotSport API remains public but rate-limited**
(Current situation — handle with backoff)

**Scenario B: GotSport API moves to authenticated access only**
(Requires: a GotSport developer account / partnership, or a shift to HTML scraping only)
- What percentage of current data volume would be retained via HTML scraper alone?
- What team discovery capability is lost?

**Scenario C: GotSport fully blocks scraping (Cloudflare/auth wall)**
(Sinc Sports situation — permanent static dataset)
- What is Evo Draw's position if this happens overnight?
- What is the maximum data age that would be in the database (i.e., how stale would the 1.49M games become and at what rate)?

### Step 4: Estimate Implementation Scope
For the webhook fallback specifically, estimate:
- Whether GotSport's platform supports outbound webhooks for their tournament operators (research their API docs / support pages)
- If yes: what would a webhook receiver endpoint look like, what schema would it need to handle, and how would it integrate with the existing pipeline?
- If no: propose an alternative "push" mechanism (e.g., tournament directors manually exporting and uploading GotSport event files)

## Deliverable

Write a scoping document to `Pipeline/Webhook Fallback Scope.md` covering all four steps above. The document should conclude with a clear RECOMMENDATION — not just options — on what the highest-value resilience investment is given the rate limit situation already underway.

This feeds into a queue item SENTINEL will post for Presten: the GotSport API dependency is the platform's single largest risk and needs a documented mitigation roadmap.

## References

- [[GotSport API Scraper]] — current primary scraper
- [[GotSport HTML Scraper]] — existing partial fallback
- [[GotSport]] — source overview and risk context
- [[Strategic Insights]] — Risk 1: GotSport API Dependency (Critical/Medium severity)
- [[Sinc Sports]] — example of what permanent lockdown looks like
- [[Data Sources Overview]] — source volume breakdown
