---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-29
priority: high
due: 2026-04-30 EOD
status: completed
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/Non-GotSport Source Priority Recommendation.md"
---

# Task: Non-GotSport Source Priority Recommendation

## Context

FORGE has accumulated significant research on non-GotSport sources: SnapSoccer (go/no-go filed), SincSports (parser reuse check filed), Affinity Soccer (research filed), NAL/Elite64 (research filed), USL Academy (source2 build plan filed), Tournament Director export pipeline (design filed). This research is distributed across 8+ documents.

Presten needs a single decision document: which non-GotSport source does FORGE build first, and why? This is the build priority queue for post-May-1 source expansion work.

## What to Build

Produce: `02 - Tiger Tournaments/Projects/Infrastructure/Non-GotSport Source Priority Recommendation.md`

A 1–2 page decision document. Not a research summary — a recommendation with a ranked list and clear rationale.

## Required Format

### Header

```yaml
---
type: forge-recommendation
topic: non-gotsport-source-priority
date: 2026-04-29
status: pending-sentinel-review
---
```

### Ranked Priority List

Rank all researched non-GotSport sources (SnapSoccer, SincSports, USL Academy, Affinity Soccer, NAL/Elite64, Tournament Director). For each:

| Rank | Source | Estimated Games/Week | Build Complexity | FORGE Recommendation | Reason |
|------|--------|---------------------|-----------------|---------------------|--------|

Use your existing research for game volume and complexity estimates. If a source was given a "no-go" in prior research, include it with rank "EXCLUDED" and the reason.

### FORGE Recommendation

One clear recommendation: Source X should be built first. Two sentences on why. One sentence on what Presten needs to know about the timing.

### Build Sequence

| Phase | Source | Dependency | Target Start |
|-------|--------|------------|-------------|
| Source 1 (first) | [name] | [prerequisite, if any] | May 2026 |
| Source 2 | [name] | [prerequisite] | [date] |
| Source 3 | [name] | [prerequisite] | [date] |

### SENTINEL Authorization Request

> FORGE recommends [Source X] as the first non-GotSport source to build, with estimated [N] games/week added to coverage. FORGE requests SENTINEL review and Presten authorization before initiating build.

## Definition of Done

- All researched sources are ranked or excluded with rationale
- One clear first-build recommendation is stated
- FORGE confirms filing in next briefing
- SENTINEL reviews and either authorizes the build sequence or redirects

## References

- `Infrastructure/SnapSoccer Go-No-Go Recommendation.md`
- `Infrastructure/SnapSoccer Test Extraction Plan.md`
- `Infrastructure/SincSports Parser Reuse Check.md`
- `Infrastructure/Affinity Soccer Source Research.md`
- `Infrastructure/Elite64 NAL Source Research.md`
- `Infrastructure/USL Academy Source 2 Build Plan.md`
- `Infrastructure/Non-GotSport Top Source Ingestion Design.md`
- `Infrastructure/Non-GotSport Sources 3-5 Synthesis Recommendation.md`
- `Infrastructure/Source Coverage Priority Matrix.md`
