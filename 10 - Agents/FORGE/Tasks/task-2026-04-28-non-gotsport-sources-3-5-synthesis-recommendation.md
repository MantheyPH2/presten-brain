---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-28
status: completed
completed: 2026-04-28
completion_note: Document filed at deliverable path. All 5 candidates evaluated (SnapSoccer, Affinity Soccer, NAL, Elite 64, SincSports). Elite 64 and SincSports excluded from ranking with rationale. Matrix scored: SnapSoccer #3 (18), Affinity Soccer #4 (16), NAL #5 (10 conditional). SENTINEL action items documented with May 10 deadline for Source #3 authorization.
priority: high
due: 2026-04-30
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/Non-GotSport Sources #3–5 — Candidate Synthesis and Recommendation.md"
---

# Task: Non-GotSport Sources #3–5 — Candidate Synthesis and Recommendation

## Context

SENTINEL's current sprint plan has Sources #1 (EDP) and #2 (USL Academy) fully scoped with build plans. SENTINEL flagged today that Sources #3–5 are undefined. FORGE has now completed individual research tasks on SnapSoccer, SincSports, Affinity Soccer, NAL, and Elite64. Those are separate documents. No synthesis exists.

Without a ranked Sources #3–5 recommendation, the post-DSS sprint (May 17+) cannot be planned. Presten cannot make go/no-go decisions on individual sources without a comparative view. SENTINEL cannot authorize anything.

This task consolidates all prior source research into a single recommendation document — FORGE's top-3 picks for Sources #3, #4, and #5, with rationale, access path, estimated build effort, and coverage impact.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/Non-GotSport Sources #3–5 — Candidate Synthesis and Recommendation.md`

---

### Section 1: Candidate Sources Evaluated

List all non-GotSport sources FORGE has researched, with their research document reference:

| Source | Research Document | Last Updated |
|--------|------------------|--------------|
| SnapSoccer | `task-2026-04-23-snapsoccer-source-research.md`, `task-2026-04-27-snapsoccer-*.md` | 2026-04-27 |
| SincSports | `task-2026-04-27-sincsports-parser-reuse-check.md`, `sincsports-go-no-go-decision-package.md` | 2026-04-27 |
| Affinity Soccer | `task-2026-04-24-affinity-soccer-source-research.md`, `task-2026-04-27-affinity-soccer-access-path-research.md` | 2026-04-27 |
| NAL | `task-2026-04-27-nal-platform-identification-research.md` | 2026-04-27 |
| Elite64 | `task-2026-04-25-elite64-nal-source-research.md` | 2026-04-25 |

---

### Section 2: Evaluation Matrix

Rate each source across five dimensions (1–5 scale, 5 = best):

| Source | Coverage Impact | Access Feasibility | Build Effort (inverted) | Parser Reuse | Urgency for Rankings |
|--------|----------------|-------------------|------------------------|--------------|---------------------|
| SnapSoccer | | | | | |
| SincSports | | | | | |
| Affinity Soccer | | | | | |
| NAL | | | | | |
| Elite64 | | | | | |
| **Weighted Score** | | | | | |

**Weights:** Coverage Impact 35%, Access Feasibility 25%, Build Effort 20%, Parser Reuse 10%, Urgency 10%

FORGE fills scores from existing research documents. Do not research new sources — synthesize from what exists.

---

### Section 3: Ranked Recommendation

**Source #3 (build immediately post-DSS):** [Name]

- Coverage impact: [leagues/regions added]
- Access path: [confirmed / browser auth required / API key required / blocked]
- Build effort: [estimate in days]
- Parser reuse from existing code: [yes/no — specify]
- Key risk: [one sentence]

**Source #4 (build May 17–31):** [Name]

Same fields as Source #3.

**Source #5 (build June+, lower priority):** [Name]

Same fields as Source #3.

---

### Section 4: Sources Not Recommended (with Reason)

For each candidate not making top-3, one-line explanation:

| Source | Reason Not Recommended |
|--------|----------------------|
| | |

---

### Section 5: Decision Required from SENTINEL/Presten

**SENTINEL needs to confirm Source #3 selection by May 10** to allow FORGE to begin build planning in the post-DSS sprint window. If Source #3 requires Presten browser access for credentials/API keys, flag this explicitly with the earliest date needed.

List any pre-authorization items FORGE needs before May 17:

| Item | Needed From | By Date |
|------|-------------|---------|
| | | |

---

## Definition of Done

- Evaluation matrix fully populated from existing research (no new research required)
- Ranked recommendation for Sources #3, #4, #5 with rationale
- Non-recommended sources explained
- SENTINEL action items clearly stated
- Filed at deliverable path by April 30

## Why This Matters

The post-DSS sprint is 2.5 weeks away. Without a Sources #3–5 priority decision, FORGE enters that sprint without a build plan — exactly what happened with Sources #1 and #2 (EDP was only planned April 28 even though it was known as a candidate since April 22). This document prevents that repeat. SENTINEL reviews by May 5 and issues Source #3 authorization before May 17.

## References

All listed research tasks in Section 1. Prioritize reading:
- `task-2026-04-27-sincsports-go-no-go-decision-package.md` — most complete single-source analysis
- `task-2026-04-27-non-gotsport-source-priority-matrix.md` — existing scoring attempt (check if it covers all 5 sources; if incomplete, this task completes it)
- `task-2026-04-27-non-gotsport-source-rerank-post-sincsports.md` — updated ranking post-SincSports assessment
