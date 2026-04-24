---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-24
status: completed
completed: 2026-04-24
priority: medium
due: 2026-04-26
---

# Task: SnapSoccer Go/No-Go Recommendation for DSS Sprint

## Context

FORGE delivered the SnapSoccer ingestion MVP design on 2026-04-24 (`Data Sources/SnapSoccer Ingestion MVP — Design 2026-04-24.md`). The design document describes what could be built and how. What it does not definitively state is: **should it be built in the 22-day DSS window?**

The design includes a "go/no-go recommendation" section per the task spec. SENTINEL needs to confirm whether a clear recommendation was written or whether the decision is still open. If the recommendation section was written: this task asks FORGE to verify it meets the criteria below and add the future-sprint trigger. If the section is vague or missing: write the decision now.

The goal is to remove "SnapSoccer ingestion — go/no-go pending design review" from the coverage gaps table. This item cannot stay open past April 26.

## What to Decide

Apply the following framework:

**Go criteria (ALL must be true to recommend Go):**
1. Estimated game count ≥ 10,000 games/year (worth the coverage addition before DSS)
2. MVP build estimate ≤ 8 hours of Presten + FORGE time combined
3. SnapSoccer data is accessible without per-org login gating (no CAPTCHA, no session-per-org requirement, or a documented workaround exists)
4. Team matching strategy from the design doc is implementable in the MVP without requiring a new team_merge campaign

**No-go criteria (ANY is sufficient to recommend No-Go for DSS):**
- Access requires 20+ separate authenticated sessions (e.g., one per league org)
- Team matching produces ambiguous results that require manual review at scale
- Total build estimate exceeds 8 hours when accounting for team matching and edge cases
- Any dependency that is currently unresolved and has no defined resolution path

## What to Write

Add or update the `## Go/No-Go Decision` section in the existing document `Data Sources/SnapSoccer Ingestion MVP — Design 2026-04-24.md`.

The section must state:

**If Go:**
```
Decision: GO for DSS sprint
Criteria met: [list which go criteria are satisfied]
Build estimate: [X] hours (FORGE) + [Y] hours (Presten)
Access path: [which path from the design — confirm it is stable]
Team matching: [confirm solvable in MVP, describe approach]
Presten action required: [what Presten needs to do and when]
```

**If No-Go:**
```
Decision: NO-GO for DSS sprint (May 16 deadline)
Blocking criterion: [which no-go criterion applies]
Future sprint trigger: Revisit when:
  - [specific pre-condition 1 — e.g., "browser access to SnapSoccer confirmed via test session"]
  - [specific pre-condition 2 — e.g., "team matching on a 500-team sample shows <5% ambiguity rate"]
  - [time estimate if pre-conditions are met: X hours for MVP in a post-DSS sprint]
```

Also add one line to your briefing: "SnapSoccer: [Go / No-Go]. Reason: [one sentence]."

## Definition of Done

- The existing SnapSoccer design document has a `## Go/No-Go Decision` section with a clearly stated Go or No-Go
- The decision is backed by the framework criteria (not just opinion)
- If No-Go: future sprint trigger conditions are concrete and measurable (not "when we have time")
- If Go: build time estimate and access path are confirmed
- Briefing includes one-line SnapSoccer status

## Why This Matters

A design document without a decision is a planning liability. SENTINEL cannot include SnapSoccer in coverage claims to Presten, cannot schedule it in the DSS sprint, and cannot deprioritize it cleanly — all because the decision is open. After April 26, SENTINEL needs a single word: Go or No-Go.

## References

- `Data Sources/SnapSoccer Ingestion MVP — Design 2026-04-24.md` — the design document to update
- DSS date: May 16, 2026 (22 days from task assignment)
