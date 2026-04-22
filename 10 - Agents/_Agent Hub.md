---
type: hub
tags: [agents, hub]
---

# Agent Hub

> Back to [[000 - Presten Manthey]]

Six autonomous agents working daily across your three worlds, managed by CHIEF.

---

## Management

| Agent | Role | Reports To | Manages |
|-------|------|-----------|---------|
| [[CHIEF Config\|CHIEF]] | Chief of Staff | Presten | SENTINEL, ORACLE, FLUX |

## Evo Draw Wing

| Agent | Role | Reports To |
|-------|------|-----------|
| [[SENTINEL Config\|SENTINEL]] | Quality & Completeness Manager | CHIEF |
| [[FORGE Config\|FORGE]] | Pipeline & Data Engineer | SENTINEL |
| [[ELO Config\|ELO]] | Ranking & Algorithm Engineer | SENTINEL |

## Personal Wing

| Agent | Role | Reports To |
|-------|------|-----------|
| [[ORACLE Config\|ORACLE]] | Personal Intelligence & Life Advisor | CHIEF |

## Exel Labs Wing

| Agent | Role | Reports To |
|-------|------|-----------|
| [[FLUX Config\|FLUX]] | Brand Builder & Product Developer | CHIEF |

---

## Chain of Command

```
Presten
  |
  CHIEF (Chief of Staff — handles everything, only escalates decisions)
  |
  +-- SENTINEL → FORGE, ELO (Evo Draw)
  +-- ORACLE (Personal + Markets + Twitter)
  +-- FLUX (Exel Labs — standby)
```

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
LIMIT 12
```
