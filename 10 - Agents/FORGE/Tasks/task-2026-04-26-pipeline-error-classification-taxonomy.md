---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-26
status: completed
completed: 2026-04-26
priority: medium
due: 2026-04-30
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/Pipeline Error Classification Taxonomy.md"
---

# Task: Pipeline Error Classification Taxonomy

## Context

May 1 launches the daily pipeline into production. From that point forward, FORGE will encounter pipeline errors in every health report: source-level fetch failures, rate limit responses, parsing errors, zero-game runs, partial runs, and computation errors. Currently there is no standard classification for these errors.

Without a taxonomy, FORGE's error reporting will be inconsistent: one briefing says "3 errors," another says "source timeout," another says "partial run." SENTINEL cannot assess severity, compare across days, or determine whether an error warrants holding Stage 2 authorization. Presten cannot judge whether an error is a pipeline health concern or routine noise.

This document establishes the error vocabulary that FORGE uses in every briefing and health report from May 1 forward.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/Pipeline Error Classification Taxonomy.md`

---

### Section 1: Error Tier Definitions

Define 3 tiers of pipeline errors:

**Tier 1 — Critical (pipeline outcome compromised)**
- Definition: errors that result in zero games ingested, corrupted data written to DB, or pipeline crash before completion
- Required action: flag to SENTINEL in same-session briefing; do not file GREEN status
- Examples: [FORGE fills based on pipeline architecture knowledge]

**Tier 2 — Degraded (partial outcome, recoverable)**
- Definition: errors that reduce game ingestion below expected threshold but do not corrupt existing data
- Required action: log in briefing with specific source and error type; escalate to SENTINEL if persists 2+ runs
- Examples: [FORGE fills]

**Tier 3 — Noise (routine, self-resolving)**
- Definition: errors that do not affect game count or data integrity; expected in any run at low frequency
- Required action: log count only; no escalation unless count exceeds [threshold] per run
- Examples: [FORGE fills]

---

### Section 2: Error Type Codes

Assign a short code to each known error type. Structure: `[SOURCE]-[TYPE]` or `[PIPELINE]-[TYPE]`.

Examples (FORGE revises based on actual pipeline architecture):

| Code | Description | Tier | Expected Frequency |
|------|-------------|------|--------------------|
| GS-TIMEOUT | GotSport org-ID fetch timeout | T2 | Low |
| GS-404 | GotSport org-ID returns no results | T3 | Medium (inactive orgs) |
| GS-RATELIMIT | GotSport rate limit response | T2 | Low (unless config misconfigured) |
| PARSE-SCORE | Score field parsing failure | T3 | Very low |
| DB-WRITE | Database write error | T1 | Should be zero |
| RUN-PARTIAL | Pipeline exited mid-run without completion | T1 | Should be zero |
| RUN-EMPTY | Pipeline completed with 0 new games | T1 | Flag if not expected |

Add any additional error types FORGE anticipates from the pipeline design documentation.

---

### Section 3: Reporting Protocol

How FORGE reports errors in briefings:

**Standard format for Tier 1:**
> "CRITICAL: [code] — [specific source/component] — [timestamp] — games affected: [N]. SENTINEL notification: [yes/no — yes if any Tier 1]."

**Standard format for Tier 2:**
> "[code] × [count] — [source] — last occurrence: [time]. Games impacted: [estimate]. Status: [monitoring / escalated]."

**Standard format for Tier 3:**
> "[code] × [count] in this run. Within expected range: [yes/no]."

FORGE includes a one-line error summary in every briefing even if the run was clean:
> "Error summary: 0 Tier 1, 0 Tier 2, [N] Tier 3 (routine). No escalation."

---

### Section 4: Escalation Thresholds

The specific thresholds that trigger SENTINEL notification:

| Condition | Action |
|-----------|--------|
| Any Tier 1 error | File to SENTINEL in briefing same session |
| Tier 2 error on 2+ consecutive runs | Escalate to SENTINEL |
| Tier 3 count > [N] in a single run | Flag as anomalous; do not escalate unless 2+ runs |
| Any source returning 0 games for 2+ runs | Escalate regardless of tier classification |

FORGE fills in the specific thresholds based on expected error volumes from the pipeline design documentation.

---

## Definition of Done

- File written at the deliverable path
- All 3 tiers defined with clear action requirements
- Minimum 5 error type codes with tier and frequency classifications
- Reporting format examples are copy-paste ready (FORGE can use them directly in briefings)
- Escalation thresholds are specific numbers, not "high" or "frequently"
- Summary in briefing: "Error taxonomy filed. [N] error codes defined. Reporting format ready for May 1."

## Why This Matters

May 1 is in 5 days. After that, FORGE files pipeline health reports every run for the foreseeable future. Without a taxonomy, those reports will be inconsistent and hard for SENTINEL to parse. With a taxonomy, SENTINEL reads one line — "0 Tier 1, 1 Tier 2 (GS-TIMEOUT × 2, source: TX State Soccer), 4 Tier 3" — and immediately knows the severity. The 30 minutes FORGE spends writing this taxonomy saves hours of ambiguity across every future health report.

## References

- `Infrastructure/Daily Pipeline Morning-After Validation Runbook.md` — health check protocol this taxonomy supports
- `Infrastructure/First-Week Pipeline Monitoring Log — May 2026.md` — first log that will use this taxonomy
- `Infrastructure/Pipeline Deployment Validation Spec — April 2026.md` — validation queries that generate error signals
- `Infrastructure/May 1 Deployment — Day-Of Runbook.md` — deployment session that begins error accumulation
