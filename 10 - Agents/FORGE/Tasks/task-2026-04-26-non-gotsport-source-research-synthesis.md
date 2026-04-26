---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-26
status: completed
completed: 2026-04-26
priority: high
due: 2026-04-29
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/Non-GotSport Source Research — April 2026 Synthesis.md"
---

# Task: Non-GotSport Source Research — April 2026 Synthesis

## Context

SENTINEL's April 26 06:39 briefing flagged a strategic gap: the current pipeline expansion is focused almost entirely on GotSport org-IDs. FORGE filed research tasks for Elite 64/NAL (`task-2026-04-25-elite64-nal-source-research`) and USL Academy/EDP (`task-2026-04-25-usl-academy-edp-org-id-verification`) on April 25. Neither research output has appeared in a FORGE briefing.

The Source Coverage Priority Matrix identified Elite 64/NAL (Score 8) and USL Academy/EDP (Score 9) as the top-priority non-GotSport research items. If Stage 1 GotSport crawl runs cleanly on May 1, the next strategic question is: which non-GotSport sources are ready to queue for Stage 2 or Stage 3? SENTINEL cannot answer that question without a synthesis of what FORGE found.

This task: produce a single document that consolidates all non-GotSport source research findings from April 2026 and produces a go/no-go recommendation for each source.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/Non-GotSport Source Research — April 2026 Synthesis.md`

---

### Section 1: Research Summary Table

For each source researched in April 2026:

| Source | Platform | Score (Priority Matrix) | Data Available | Org-ID or API Exists | FORGE Recommendation | Due Diligence Status |
|--------|----------|------------------------|----------------|---------------------|---------------------|---------------------|
| Elite 64 / NAL | | 8 | | | PROCEED / DEFER / NOT VIABLE | |
| USL Academy / EDP | | 9 | | | PROCEED / DEFER / NOT VIABLE | |
| SnapSoccer | | | | | | |
| [any others researched] | | | | | | |

**FORGE Recommendation options:**
- **PROCEED:** Source is viable, platform is accessible, FORGE can implement. Suitable for Stage 2 queue.
- **DEFER:** Source is viable but has a blocker (requires outreach, data format issue, access constraint). Document the blocker and revisit date.
- **NOT VIABLE:** Source does not have structured digital data accessible to the pipeline.

---

### Section 2: Source Profiles (one per source)

For each source, a brief profile:

**[Source Name]**
- Platform: [GotSport / own API / scrape / manual]
- Games in scope: [estimated games per season, age groups]
- Data format: [structured / semi-structured / unstructured]
- Access method: [org_id lookup / API endpoint / scrape / manual export]
- Known blockers: [any]
- Implementation complexity: LOW / MEDIUM / HIGH
- FORGE notes: [one paragraph]

Minimum: Elite 64/NAL profile, USL Academy/EDP profile, SnapSoccer profile (even if the go-no-go task was filed — synthesize the current status here).

---

### Section 3: Stage 2 Queue Recommendation

Based on all research findings, which sources should SENTINEL consider for Stage 2 expansion (concurrent with or immediately after GotSport Stage 2)?

List in priority order. Include:
- Source name
- Why now vs. later
- Prerequisites before FORGE can implement (if any)
- Estimated implementation time

If no non-GotSport source is ready for Stage 2, state that explicitly with reasoning. Do not fill in sources as "ready" if the research does not support it.

---

### Section 4: Open Research Items

Any source from the Source Coverage Priority Matrix that FORGE has not yet researched. For each:
- Source name
- Score
- Why not yet researched (deprioritized for April, blocked, etc.)
- Whether FORGE recommends assigning a research task now or deferring to post-May 1

---

## Quality Standard

- Do not re-open the individual research tasks. This is a synthesis document — pull findings from what FORGE already knows from those tasks.
- If FORGE's research on Elite 64/NAL or USL Academy/EDP produced inconclusive findings, state that explicitly. An honest "insufficient data to recommend" is better than a forced recommendation.
- Every source in Section 1 must have a recommendation. No empty rows in the recommendation column.
- Section 3 Stage 2 queue recommendation must be a ranked list, not a discussion.

## Why This Matters

After May 1 GotSport launch, the next pipeline expansion decision is: what comes after GotSport Stage 2? SENTINEL needs to brief Presten with a concrete answer, not "FORGE is researching." This synthesis document is the input to that briefing. If FORGE does not file it, SENTINEL cannot give Presten a strategic answer about source diversity — the gap SENTINEL flagged in the April 26 06:39 briefing.

## References

- `task-2026-04-25-elite64-nal-source-research.md` — Elite 64/NAL research findings
- `task-2026-04-25-usl-academy-edp-org-id-verification.md` — USL Academy/EDP research findings
- `task-2026-04-24-snapsoccer-go-no-go-recommendation.md` — SnapSoccer research findings
- `Infrastructure/Source Coverage Priority Matrix.md` — priority scores for all source candidates
- `Infrastructure/Source Gap Inventory.md` — full source gap list
