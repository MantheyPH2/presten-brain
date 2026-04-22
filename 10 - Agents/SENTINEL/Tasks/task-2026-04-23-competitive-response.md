---
type: agent-task
assigned_to: SENTINEL
assigned_by: Presten
date: 2026-04-23
status: completed
priority: high
---

# PRIORITY DIRECTIVE: Competitive Response to USA Rank

Presten has provided detailed competitive intelligence on USA Rank / USA Sport Statistics — our top competitor. Read it now:

**`02 - Tiger Tournaments/Projects/Knowledge/USA Rank Competitive Intel.md`**

## Your Job

1. **Read the full intel brief** — understand what they have that we don't
2. **Create a prioritized gap analysis** — which gaps are most critical to close first
3. **Assign specific tasks to FORGE and ELO** to start closing these gaps immediately:

### For FORGE (data/pipeline gaps):
- USA Rank ingests from 400+ sources. We have 7. FORGE needs to:
  - Research and scope adding SnapSoccer as a data source
  - Research and scope adding SincSports/SoccerInCollege data
  - Design a source-submission framework (tournament directors submit missing events)
  - Scope daily pipeline runs instead of overnight batch

### For ELO (ranking/algorithm gaps):
- USA Rank uses last-20-games with recency weighting and emergent league quality. ELO needs to:
  - Compare our Glicko v7 accuracy against USA Rank's methodology on shared teams
  - Design rank bands (Gold/Silver/Bronze/Red/Blue/Green) for our rankings
  - Design club rankings (average top team ratings, U11-U17, 5+ age groups)
  - Scope event strength ratings (High/Avg/Low/Spread/Upset %)

### For yourself:
- Write a competitive response plan to `02 - Tiger Tournaments/Projects/Product & Planning/competitive-response-plan.md`
- This should be a phased roadmap: Match → Beat → Dominate
- Prioritize by impact and effort

## Success Criteria
- FORGE and ELO each have 3+ new tasks focused on closing competitive gaps
- Competitive response plan is written and actionable
- The team knows exactly what we need to build to match, then surpass USA Rank

## Why This Matters
USA Rank has 400+ sources and daily updates. We have better algorithms and more data (1.65M games), but they're ahead on source breadth and update cadence. We need to close these gaps before DSS in May.

---

## Completion Log

**Completed:** 2026-04-23 by SENTINEL

### Deliverables Produced:
- `02 - Tiger Tournaments/Projects/Product & Planning/competitive-response-plan.md` — phased roadmap (Match → Beat → Dominate) with timelines and success criteria
- `10 - Agents/FORGE/Tasks/task-2026-04-23-snapsoccer-source-research.md` — research + Go/No-Go on SnapSoccer as data source
- `10 - Agents/FORGE/Tasks/task-2026-04-23-source-submission-framework.md` — design system for tournament director source submissions
- `10 - Agents/FORGE/Tasks/task-2026-04-23-daily-pipeline-cadence.md` — scope daily/hourly pipeline (3 options evaluated)
- `10 - Agents/ELO/Tasks/task-2026-04-23-rank-bands-design.md` — design Gold/Silver/Bronze/Red/Blue/Green band system
- `10 - Agents/ELO/Tasks/task-2026-04-23-club-rankings.md` — design club ranking system (avg of top team ratings, U11–U17)
- `10 - Agents/ELO/Tasks/task-2026-04-23-event-strength-ratings.md` — design event strength rating system (High/Avg/Low/Spread/Upsets)
