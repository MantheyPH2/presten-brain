---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-28
status: completed
completed: 2026-04-28
priority: high
due: 2026-04-30
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/Non-GotSport Source Implementation Priority List.md"
---

# Task: Non-GotSport Source Implementation Priority List

## Context

FORGE has completed extensive research on non-GotSport sources: SnapSoccer, SincSports, Affinity Soccer, Elite64, NAL, USL Academy, EDP. The research documents exist. What does not exist: a decision-ready ranked list that tells SENTINEL and Presten which source to implement first, in what order, and why.

Right now, SENTINEL cannot authorize Stage 2 non-GotSport expansion without knowing the priority sequence. The `task-2026-04-26-non-gotsport-source-research-synthesis.md` covers the landscape. This task converts that synthesis into an actionable priority list with implementation order, effort estimate, and go/no-go criteria for each source.

The DSS deadline (May 16) and the May 9 readiness check constrain how many new sources can be added safely before DSS. FORGE must account for this.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/Non-GotSport Source Implementation Priority List.md`

---

### Section 1: Prioritization Criteria

State the criteria FORGE used to rank sources. Suggested criteria:
- Game volume estimate (how many additional unique games does this add?)
- Parse complexity (can an existing parser be reused or adapted?)
- Data quality confidence (structured API vs. scraped HTML vs. tournament export)
- Authorization risk (public data vs. login-required vs. partner agreement needed)
- Time-to-first-game estimate (days from implementation start to first successful ingestion)

ELO's calibration needs should inform this: sources that fill gaps in leagues with known coverage holes rank higher.

---

### Section 2: Ranked Priority Table

| Rank | Source | Parser Reuse | Game Volume Est. | Parse Complexity | Time-to-First-Game | Pre-DSS Safe? | Notes |
|------|--------|-------------|-----------------|-----------------|-------------------|--------------|-------|
| 1 | | | | | | | |
| 2 | | | | | | | |
| 3 | | | | | | | |
| 4 | | | | | | | |
| 5 | | | | | | | |

**Pre-DSS Safe = YES** means: implementation can complete, first run validated, and any rating impact assessed before May 9 readiness check.
**Pre-DSS Safe = NO** means: implementation starts after DSS.

---

### Section 3: Implementation Sequence

For the top 2 sources (those prioritized for pre-DSS implementation):

**Source 1: [Name]**
- Implementation start: [date]
- Steps: [1-line each]
- Expected first successful run: [date]
- Validation criteria: [what defines success]
- SENTINEL authorization needed before: [specific step]

**Source 2: [Name]**
- Implementation start: [date, must follow Source 1 validation]
- Steps: [1-line each]
- Expected first successful run: [date]
- Validation criteria: [what defines success]
- SENTINEL authorization needed before: [specific step]

---

### Section 4: Sources Deferred to Post-DSS

List sources ranked 3+ with the specific reason for deferral:
- Parse complexity too high for safe pre-DSS window
- Data quality unclear without deeper testing
- Authorization required (partner agreement)
- Game volume estimate too low to justify pre-DSS risk

---

### Section 5: Coverage Gap Impact

Which leagues or states get meaningful coverage improvement from the top-priority source(s)?

Cross-reference with `Infrastructure/GotSport Coverage Map by State.md` and `Rankings/Competitive Coverage Gap Inventory.md`. If Source 1 fills a gap ELO has flagged as calibration-critical, note that explicitly.

---

## Definition of Done

- Document filed at deliverable path by April 30
- All sources from the research synthesis are ranked (none omitted)
- Top 2 sources have complete implementation sequences with dates
- Pre-DSS safety determination made for each source
- FORGE briefing April 30 references this document and states which source FORGE recommends implementing first

## Why This Matters

SENTINEL cannot authorize non-GotSport expansion without a priority order. Every week of delay on the next source is a week of missing game data — which means ELO is calibrating against an incomplete game set, and Presten is demoing a platform with known coverage gaps. The research exists. The implementation plan does not. This closes that gap.

## References

- `task-2026-04-26-non-gotsport-source-research-synthesis.md` — landscape research to convert
- `task-2026-04-26-non-gotsport-top-source-ingestion-design.md` — design doc for top candidate
- `task-2026-04-27-snapsoccer-test-extraction-plan.md` — SnapSoccer feasibility
- `task-2026-04-27-sincsports-parser-reuse-check.md` — SincSports parser evaluation
- `task-2026-04-25-competitive-coverage-gap-inventory.md` — gaps this must address
- `Infrastructure/GotSport Coverage Map by State.md` — state coverage context
