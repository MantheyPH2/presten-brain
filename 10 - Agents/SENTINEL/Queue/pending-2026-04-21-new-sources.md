---
type: agent-queue
agent: SENTINEL
category: update
priority: medium
date: 2026-04-21
status: resolved
answer: ""
---

# Update: 3 New Potential Game Data Sources Identified

## Overview

During today's system health and data quality scan, SENTINEL identified three external data sources that could supplement GotSport's game and roster data. Given the active rate limit issues with GotSport (see FORGE's queue item), having at least one secondary data source is strategically valuable — both for redundancy and for filling gaps when the primary sync fails.

No action is required immediately, but Presten should be aware of these options for roadmap planning.

---

## Source 1: US Youth Soccer Club Finder API

**URL:** National platform, public developer API  
**Coverage:** Roster affiliation, club registration data, coach licensing records  
**Format:** JSON REST API  
**Rate limits:** 200 req/min (documented and stable)  
**Cost:** Free tier available; enterprise tier ~$150/mo for unlimited access

**Assessment:** This source doesn't provide game results, but it's excellent for roster verification and club affiliation data — exactly the kind of baseline that GotSport misses when teams are newly registered or transferring clubs. Could be used to fill in the 127 teams with unexplained stale data that FORGE couldn't root-cause today. Medium integration effort.

---

## Source 2: Blue Star Sports Regional Results Feeds

**URL:** Blue Star Sports developer portal (BSS manages tournaments in 12 states primarily in the Southeast and Midwest)  
**Coverage:** Game results, bracket outcomes, tournament standings  
**Format:** XML feeds (updated every 4 hours during active tournament windows)  
**Rate limits:** Not explicitly documented; appears generous based on public feed inspection  
**Cost:** Partnership agreement required — likely low-cost or free with co-marketing arrangement

**Assessment:** High value if Evo Draw has a significant user base in the 12-state coverage area. Blue Star runs a large number of regional showcases and tournaments that GotSport doesn't always cover comprehensively. XML format requires a parser layer, but it's well-structured. Worth a partnership conversation if the regional overlap is confirmed.

---

## Source 3: Stack Sports Game Reporting Module

**URL:** Stack Sports API (requires approved developer account)  
**Coverage:** Game results, referee reports, disciplinary records  
**Format:** JSON REST API  
**Rate limits:** 100 req/min  
**Cost:** API access is a paid add-on; pricing requires a sales conversation

**Assessment:** Stack Sports is GotSport's primary competitor and serves a different regional club network with some overlap. For Evo Draw, the most valuable unique data from Stack Sports would be the disciplinary records (yellow/red cards, suspensions) which GotSport's API does not currently expose. Could enhance team profile depth. Medium-high integration effort, higher cost.

---

## Recommendation

If Presten wants to pursue one of these in the near term, the priority order would be:

1. **US Youth Soccer Club Finder API** — free, fast to integrate, fills a known gap (stale roster/affiliation data)
2. **Blue Star Sports** — high game data value if the regional coverage aligns; requires a partnership conversation first
3. **Stack Sports** — valuable for disciplinary data but higher cost and effort; better suited for a future phase

No engineering work has been started on any of these. This is purely an awareness and options update.
