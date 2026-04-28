---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-28
status: open
priority: high
due: 2026-05-02
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/Non-GotSport Source #1 — Implementation Build Plan.md"
---

# Task: Non-GotSport Source #1 — Implementation Build Plan

## Context

FORGE completed the Non-GotSport Source Implementation Priority List today. That document establishes WHAT to build and in what order. What does not exist: the day-by-day build plan that tells Presten HOW to execute the #1 ranked source implementation.

The DSS window (May 16) constrains how much time is available. If Source #1 is "Pre-DSS Safe = YES," implementation must start no later than May 5 to allow validation time before May 9. This build plan gives Presten and FORGE the execution blueprint so that the first session with Source #1 is productive, not exploratory.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/Non-GotSport Source #1 — Implementation Build Plan.md`

Use the priority list deliverable (`Infrastructure/Non-GotSport Source Implementation Priority List.md`) to identify Source #1. Then produce the following.

---

### Section 1: Source Identity and Summary

| Field | Value |
|-------|-------|
| Source name | _[from priority list]_ |
| Platform / data format | _[GotSport-style API / HTML scrape / tournament export / other]_ |
| Parser reuse | YES (from: _[existing parser]_) / NO (new parser required) |
| Game volume estimate | _[from priority list]_ |
| Pre-DSS Safe | YES / NO |
| Implementation window | _[start date → expected first successful run date]_ |

---

### Section 2: Parser Architecture Decision

Two options — FORGE selects one and justifies:

**Option A: Reuse existing parser**
- Which parser to reuse: _[file path]_
- Modifications required: _[specific changes]_
- Test data format: _[where to get test fixtures]_

**Option B: New parser**
- Input format spec: _[describe data structure]_
- Parser output schema: must match existing `games` table schema
- Key parsing challenges: _[date normalization, team name matching, etc.]_

FORGE selection: _[A or B]_
Justification: _[1–2 sentences]_

---

### Section 3: Day-by-Day Build Timeline

| Day | Date | Task | Owner | Success Criteria |
|-----|------|------|-------|-----------------|
| 0 | _[start]_ | Parser scaffolding + test fixture extraction | FORGE | Parser loads without errors |
| 1 | | First game ingested to staging DB | FORGE + Presten | 1 game row in DB with correct schema |
| 2 | | Batch test run (N games) | FORGE + Presten | < 5% parse errors |
| 3 | | Team name matching validation | FORGE | Match rate > 90% against known team names |
| 4 | | Full source run (all available games) | Presten | Volume within estimated range ± 20% |
| 5 | | Rankings recompute with new data | Presten | No anomalous rating shifts (> 200 large movers) |
| 6 | | FORGE validation report filed | FORGE | First-run validation report in Infrastructure/ |

Adjust day count based on parser complexity (simple reuse = compress to 4 days; new parser = expand to 8 days).

---

### Section 4: Integration Requirements

Changes required in the existing pipeline:
- [ ] Config file update: add source identifier and credentials/endpoint
- [ ] Cron schedule: add to existing daily pipeline or create separate schedule?
- [ ] Dedup rules: does this source require new dedup logic for team matching?
- [ ] `ecnl_verified` or league classification: does this source require classifier updates?

FORGE documents specifics for each checked item.

---

### Section 5: Validation Gates

Before calling Source #1 implementation complete, three gates must pass:

**Gate 1 — Parse Quality**
- Parse error rate < 5%
- All required fields populated: team_name, opponent_name, date, score, league_name
- No duplicate game rows (same teams, same date)

**Gate 2 — Ranking Impact**
- ELO runs post-ingestion ranking comparison
- Avg rating shift from new games: < 30 pts (expected with small initial dataset)
- No team moves more than 150 pts from new source data alone
- ELO files confirmation: "Source #1 data is calibration-safe"

**Gate 3 — SENTINEL Authorization**
- FORGE files first-run validation report to `Infrastructure/`
- SENTINEL reviews and issues production authorization
- Presten confirms source is live in daily pipeline

---

### Section 6: Rollback Plan

If Gate 1 or Gate 2 fails:
- Disable source in config (do not delete data)
- FORGE files failure report to `Infrastructure/Source #1 — First Run Failure Report.md`
- SENTINEL decides: revise parser and retry, or defer to post-DSS

FORGE does not attempt a second run without SENTINEL review of the failure report.

---

## Definition of Done

- Build plan filed at deliverable path by May 2
- Section 1 identifies Source #1 by name and confirms Pre-DSS Safe status
- Section 3 has specific calendar dates (not relative day numbers)
- Section 5 gates are present with specific numeric thresholds
- FORGE 2026-05-02 briefing references this document and states which day of the build plan the session is on

## Why This Matters

The priority list tells SENTINEL which source to approve. This build plan tells everyone how to execute it. Without it, the first implementation session starts with architectural decisions instead of code. Pre-DSS time is too limited for exploratory implementation sessions.

## References

- `Infrastructure/Non-GotSport Source Implementation Priority List.md` — input (just completed by FORGE)
- `task-2026-04-27-snapsoccer-minimum-viable-parser-spec.md` — likely Source #1 parser spec
- `task-2026-04-27-sincsports-parser-reuse-check.md` — alternative source parser evaluation
- `task-2026-04-25-competitive-coverage-gap-inventory.md` — leagues this source must address
- `Infrastructure/GotSport Coverage Map by State.md` — state coverage baseline
