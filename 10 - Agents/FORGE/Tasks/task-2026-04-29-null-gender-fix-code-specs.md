---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-29
priority: high
due: 2026-04-30 EOD
status: completed
completed: 2026-04-29
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/Null Gender Audit — Code Implementation Specs.md"
topic: null-gender-fix-code-specs
tags: [forge, task, null-gender, implementation, code-ready]
---

# Task: Null Gender Fix — Code Implementation Specs

## Context

You filed the investigation package at `Infrastructure/Null Gender Audit — Investigation Package.md` with four proposed fixes (A, B, C, D) and the SQL queries for Presten to run. The package describes WHAT each fix does. This task produces the actual code changes so that when Presten shares query results, FORGE can deploy within 30 minutes — no additional spec work required mid-session.

## What to Build

Produce: `02 - Tiger Tournaments/Projects/Infrastructure/Null Gender Audit — Code Implementation Specs.md`

A code-first document with the exact JavaScript changes for each fix. This is not a description — it is literal code diffs.

---

## Required Sections

### Section 1 — Fix A: Lower Sibling Inference Threshold

**File:** `backfill-all-games.js`

Write the exact code change: find the sibling inference block (the one that checks if ≥95% of sibling teams share a gender), and show:
- The current code (the threshold check line)
- The proposed change (95% → 90%)
- Whether this is a one-line change or requires surrounding logic

Note any risk: lowering this threshold may cause false positives if a club runs mixed-gender programs. State FORGE's assessment of that risk.

### Section 2 — Fix B: Event-Level Gender Inference

**File:** `backfill-all-games.js`

Write the new inference block that: for each `(event_id, age_group)` combination where 100% of teams with known gender share one gender, set null-gender teams in that group to match.

Show:
- The SQL or JS query to identify eligible groups
- The UPDATE logic
- Where in the execution flow this should run (before or after Fix A?)
- Whether this runs as a backfill SQL or a code-path change

### Section 3 — Fix C: Expanded Team Name Patterns

**File:** `backfill-all-games.js` (or wherever team-name gender patterns are defined)

Show the current pattern list and the proposed additions: `B14`, `G14`, `14B`, `14G`, and any others from the investigation package. If patterns are defined as a regex, show the updated regex. If they are an array, show the array additions.

### Section 4 — Fix D: Expanded Event Name Patterns

Same format as Fix C. Show current event-name patterns and additions: `ECNL Girls`, `NPL Boys`, and any others from the investigation package.

### Section 5 — Deployment Order and Backfill Plan

In what order should fixes deploy? (Recommended: C and D first — purely additive pattern expansion with zero false-positive risk; then A; then B after Presten reviews event-level override counts.)

Include the post-fix backfill SQL from the investigation package (or reference it by section number if already complete).

### Section 6 — Deployment Decision Gate

| Fix | Pre-condition | FORGE can deploy? | Presten auth needed? |
|-----|--------------|-------------------|---------------------|
| Fix A | Query 1A result (sibling coverage %) | Yes, after count review | No |
| Fix B | Query 1C result (event concentration) | Yes, if ≥50% of nulls in eligible groups | No |
| Fix C | None | Yes, immediately | No |
| Fix D | None | Yes, immediately | No |

---

## Definition of Done

- All four fixes have exact code diffs (or confirmed single-line change descriptions)
- Deployment order is specified
- Risks for each fix are stated
- Fix C and D can be deployed by FORGE independently, without waiting for query results
- Document is self-contained — FORGE can implement any fix without referencing other files

---

## Why Now

When Presten shares the 4 investigation query results, there will be 30 minutes of available execution time before context resets. If FORGE has to write the code specs mid-session, that window closes. Pre-writing the specs eliminates that risk.

Fix C and D have no query dependencies — those can be deployed immediately. Filing this document enables FORGE to act on those two fixes in the next Presten session regardless of query results.
