---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-27
due: 2026-04-28
priority: high
status: completed
completed: 2026-04-27
topic: stage2-section5-proposed-criteria-draft
---

# Task: Stage 2 Authorization Criteria — Proposed Section 5 Draft

## Context

SENTINEL has flagged the Stage 2 Authorization Criteria document (`Infrastructure/Stage 2 Expansion — Authorization Criteria.md`) as missing Section 5 across three consecutive FORGE briefing sessions (08:46, 11:05, 13:18 on 2026-04-27). This is a SENTINEL gap — SENTINEL has not resolved it. But FORGE has the domain knowledge to draft what Section 5 should contain.

FORGE owns the technical infrastructure side of Stage 2: which org-IDs are going in, what the config additions look like, how the pipeline ingests new sources, and what post-launch validation confirms a source is healthy. SENTINEL owns the authorization decision itself, but FORGE can draft the criteria that make that authorization technically sound.

This task: FORGE drafts a proposed Section 5 for SENTINEL to review, accept, modify, or reject. This converts a SENTINEL bottleneck into a collaborative resolution and allows SENTINEL to act in the 16:21 session rather than the next cycle.

## Task

Read `Infrastructure/Stage 2 Expansion — Authorization Criteria.md`. Review Sections 1–4. Identify what Section 5 should cover given what the existing sections establish.

Then draft a proposed Section 5: the technical readiness criteria FORGE must satisfy before Stage 2 goes live. These are FORGE's own pipeline and infrastructure conditions — not ELO calibration criteria (that is SENTINEL's and ELO's domain) and not business criteria (that is Presten's domain).

## Proposed Section 5 Structure

Draft Section 5 as a numbered list of FORGE-owned criteria. Each criterion should be:
- Verifiable (there is a specific query, log entry, or document that confirms it)
- Binary (met / not met — no partial credit)
- In FORGE's control (if it requires Presten action, note it as a dependency, not a FORGE criterion)

Suggested criteria categories to address (FORGE may revise):

**5.1 — Config Additions Staged and Reviewed**
- Criteria: All Stage 2 org-IDs have been added to the crawl config file with correct format, reviewed against the org-ID reference document, and confirmed non-duplicate with Stage 1 entries.
- Verification: FORGE config review document filed at `Infrastructure/Stage 2 Org-ID Config Pre-Staging.md`

**5.2 — Crawl Test Successful (Sample Run)**
- Criteria: At least one test crawl has been run per new source category (or per new platform type) and returned >0 valid game records.
- Verification: Sample output log filed or inline in the Stage 2 Authorization Trigger document.

**5.3 — No Stage 1 Regression**
- Criteria: Stage 1 sources continue to produce expected game volumes within ±10% of the pre-Stage-2 baseline after Stage 2 config additions are staged.
- Verification: Source Baseline Section 4 query results show Stage 1 sources within tolerance.

**5.4 — Pipeline Daily Run Confirmed Stable**
- Criteria: At least 3 consecutive successful daily pipeline runs have been logged (no FM1/FM2 errors, no source-level failures) before Stage 2 expansion goes live.
- Verification: First-Week Pipeline Monitoring Log shows 3 consecutive green runs.

**5.5 — FORGE Authorization Confirmation Filed**
- Criteria: FORGE has filed a Stage 2 Authorization Confirmation document stating that all criteria in Sections 1–5 are met, with a reference to each verification document.
- Verification: Document exists at `Infrastructure/Stage 2 Authorization Confirmation — FORGE.md`

## Format for the Draft

File the proposed Section 5 as a standalone draft document. SENTINEL will review it and either incorporate it directly into the main Authorization Criteria doc or request edits.

Do NOT modify the main `Infrastructure/Stage 2 Expansion — Authorization Criteria.md` directly. SENTINEL will make that edit after reviewing the draft.

## Deliverable

File to: `Infrastructure/Stage 2 Authorization Criteria — Proposed Section 5 Draft.md`

Update this task to `status: completed` when filed.

## Notes

This draft is advisory. SENTINEL may accept it as-is, modify it, or reject it and write Section 5 independently. FORGE's authority is to propose criteria within its own technical domain. FORGE does not have authority to define the overall authorization threshold — that is SENTINEL's decision.

If reviewing the existing Sections 1–4 reveals that Section 5 overlaps with something already covered, note the overlap explicitly. SENTINEL needs to know if the "Section 5 gap" is a labeling issue (criteria exist but aren't labeled as Section 5) vs. a genuine missing section.

Due date is April 28 so SENTINEL can act on it the same day before the April 29 session. This is the last day to resolve this before the Stage 2 authorization process becomes blocking.
