---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-25
status: pending
priority: high
due: 2026-04-27
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/Source Coverage Priority Matrix — April 2026.md"
---

# Task: Source Coverage Priority Matrix

## Context

The Source Gap Inventory (filed 2026-04-24) lists known coverage gaps — leagues with no source or stale ingestion. SENTINEL has no triage tool. When Presten has 30–60 minutes for source work, the answer to "which gap closes first?" should not depend on whoever is in the room. It should be pre-computed.

This task: build the decision-support matrix. Every gap gets a score across three dimensions — estimated game volume, ranking impact, and effort to close. Output is a ranked list so SENTINEL can answer the triage question immediately.

## What to Do

### Step 1: Enumerate All Known Gaps

Pull every source gap from `Infrastructure/Source Gap Inventory — April 2026.md`. Include both Priority 1 (zero coverage) and Priority 2 (stale/partial) gaps. Each becomes one row.

### Step 2: Score Each Gap — Three Dimensions

**A. Estimated Game Volume (1–3)**
- 3 = High: >5,000 games/season estimated
- 2 = Medium: 1,000–5,000 games/season
- 1 = Low: <1,000 games/season

Use the Source Gap Inventory's existing estimates where available. Where missing: use league tier from the League Hierarchy × typical event count for that tier.

**B. Ranking Impact (1–3)**
- 3 = High: Absence creates visible ranking distortions — affects top-50 teams; ECNL/GA/MLS NEXT players will notice
- 2 = Medium: Affects ranking depth below top-50 or regional cohorts
- 1 = Low: Unlikely to affect any ranked team's position materially

Use League Hierarchy calibration values — high-cal leagues have higher ranking impact.

**C. Effort to Close (1–3, inverted: 1 = hard, 3 = easy)**
- 3 = Easy: Already on GotSport — just add org_id to crawl config (5 min Presten time)
- 2 = Medium: Requires browser session or small script change (15–60 min Presten time)
- 1 = Hard: Requires new scraper, API integration, or legal/access review (weeks of FORGE work)

### Step 3: Compute Priority Score

Priority Score = Volume + Impact + Effort

Sort by Priority Score descending. For ties, rank by Impact first (higher Impact wins).

### Step 4: Write the Matrix

Deliverable: `02 - Tiger Tournaments/Projects/Infrastructure/Source Coverage Priority Matrix — April 2026.md`

Format:

```
| Rank | League | Status | Volume (1–3) | Impact (1–3) | Effort (1–3) | Score | Path to Close |
```

For the "Path to Close" column, pull the one-line go/no-go recommendation from the Source Gap Inventory.

### Section 2: Top-3 Recommendations

After the matrix, write a short paragraph on the top 3 gaps to close. Each: why it ranks where it does and what Presten does to close it (one concrete action).

### Section 3: Deferral List

Any gap scoring ≤3: mark as "Deferred — insufficient leverage or closing effort too high." SENTINEL will not schedule work for these until higher-priority gaps are resolved.

## Definition of Done

- All gaps from Source Gap Inventory are scored
- Matrix is sorted by Priority Score
- Top-3 recommendation paragraph is written with specific actions
- Deferral list identifies lower-priority gaps
- Summary in briefing: "Source Coverage Priority Matrix filed. Top gap: [name] (score: [N]). Top 3 all closable via [method]."

## Why This Matters

Right now, every source triage conversation starts from scratch. The Priority Matrix converts the Source Gap Inventory from a reference doc into a decision tool. SENTINEL can hand Presten the top-3 list at the start of any source work session.

## References

- `02 - Tiger Tournaments/Projects/Infrastructure/Source Gap Inventory — April 2026.md`
- `02 - Tiger Tournaments/Projects/Leagues/League Hierarchy.md`
