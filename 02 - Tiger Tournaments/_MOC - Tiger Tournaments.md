---
type: moc
world: Tiger Tournaments
tags: [moc, tiger-tournaments, evo-draw]
---

# Tiger Tournaments

> Back to [[000 - Presten Manthey]]

Youth soccer tournament organization and the Evo Draw data platform — rankings, brackets, game aggregation, and tournament operations.

---

## Active projects

```dataview
TABLE status, priority, target
FROM "02 - Tiger Tournaments"
WHERE type = "project" AND status = "active"
SORT priority ASC
```

---

## Evo Draw Platform

The core technology. Start here: [[Evo Draw Home]]

- [[Data Sources Overview]] — all 7 sources
- [[Data Pipeline]] — 9-step processing chain
- [[Ranking Engine]] — Elo/Glicko v7
- [[Strategic Insights]] — cross-domain analysis and action items

---

## Key areas

- [[Evo Draw Home|Evo Draw / Projects]] — active builds and deliverables
- Product & Planning — roadmap, specs, strategy
- Research — market, competitors, domain knowledge
- Meetings — notes and decisions
- Ideas — future possibilities

---

## Recent meetings

```dataview
TABLE date, attendees, project
FROM "02 - Tiger Tournaments/Meetings"
WHERE type = "meeting"
SORT date DESC
LIMIT 10
```
