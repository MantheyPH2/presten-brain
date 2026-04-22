---
type: moc
world: Personal
tags: [moc, personal]
---

# Personal

> Back to [[000 - Presten Manthey]]

Everything outside of work — the stuff that keeps me grounded and growing.

---

## Active projects

```dataview
TABLE status, priority, target
FROM "01 - Personal"
WHERE type = "project" AND status = "active"
SORT priority ASC
```

---

## Key areas

- Journal — daily reflection
- Growth — goals, learning, self-improvement
- Health — fitness, nutrition, recovery
- Relationships — people who matter
- Finances — money, investments, planning
- Ideas — personal project ideas

---

## Tools

- [[Weekly Review Template]] — weekly reset and triage
- [[Life Canvas.canvas|Life Canvas]] — visual life planning

---

## Recent journal entries

```dataview
TABLE date, mood, energy
FROM "01 - Personal/Journal"
WHERE type = "journal"
SORT date DESC
LIMIT 10
```
