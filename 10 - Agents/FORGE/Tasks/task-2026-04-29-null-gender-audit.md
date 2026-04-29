---
type: agent-task
agent: FORGE
assigned_by: SENTINEL
date: 2026-04-29
priority: high
status: in-progress
forge_action: 2026-04-29 — Investigation package filed at Infrastructure/Null Gender Audit — Investigation Package.md. Data Quality updated. Four fixes proposed (sibling threshold, event-level inference, name patterns, event name patterns). SQL queries written for Presten to run. Blocked on DB execution — Presten must run Section 1 queries and share results before fixes can be measured.
topic: null-gender-audit
---

# Task: Audit and Fix Null Gender Values

## Context

[[Data Quality]] reports ~12,000 null gender values (3.3% of records). This is the largest known field-completeness gap in the pipeline and directly degrades ranking quality — gender is a required dimension for all Evo Draw divisions.

## What To Do

1. **Identify the source.** Query the database for games where `gender IS NULL`. Break down by data source (GotSport API, GotSport HTML, TGS, etc.) and by age group. Which sources are the worst offenders?

2. **Trace the pipeline step.** Pinpoint which step in the 9-step pipeline is dropping or failing to infer gender. Review [[Parsing Rules]] for gender inference logic. Is the regex/pattern matching failing on certain league name formats?

3. **Implement a fix.** Either:
   - Improve the parsing rule to catch more patterns, or
   - Add a backfill step that infers gender from team name, league name, or age group where possible.

4. **Measure the result.** After the fix, re-run the audit (Step 9) and report the new null rate. Target: below 1.5%.

5. **Document.** Update [[Data Quality]] with the new completeness metrics and describe what you changed.

## Deliverable

- Updated parsing/backfill code (document file path in briefing)
- Updated [[Data Quality]] note with before/after metrics
- Daily briefing noting task completion

## Definition of Done

Null gender rate drops below 1.5% and the fix is documented.
