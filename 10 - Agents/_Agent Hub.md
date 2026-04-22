---
type: hub
tags: [agents, hub]
---

# Agent Hub

> Back to [[000 - Presten Manthey]]

Five autonomous agents working daily across your three worlds.

---

## Evo Draw Wing

| Agent | Role | Reports To |
|-------|------|-----------|
| [[FORGE Config\|FORGE]] | Pipeline & Data Engineer | SENTINEL |
| [[ELO Config\|ELO]] | Ranking & Algorithm Engineer | SENTINEL |
| [[SENTINEL Config\|SENTINEL]] | Quality & Completeness Manager | Presten |

## Personal Wing

| Agent | Role | Reports To |
|-------|------|-----------|
| [[ORACLE Config\|ORACLE]] | Personal Intelligence & Life Advisor | Presten |

## Exel Labs Wing

| Agent | Role | Reports To |
|-------|------|-----------|
| [[FLUX Config\|FLUX]] | Brand Builder & Product Developer | Presten |

---

## Delivery Board

```dataview
TABLE agent AS "Agent", category AS "Type", priority AS "Priority", date AS "Date", status AS "Status"
FROM "10 - Agents"
WHERE type = "agent-queue" AND status = "pending"
SORT priority ASC
```

## Recent Briefings

```dataview
TABLE agent AS "Agent", tasks_completed AS "Done", questions_posted AS "Questions", date AS "Date"
FROM "10 - Agents"
WHERE type = "agent-briefing"
SORT date DESC
LIMIT 10
```
