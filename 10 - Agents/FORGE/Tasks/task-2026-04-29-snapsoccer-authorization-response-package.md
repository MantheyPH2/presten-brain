---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-29
priority: high
due: 2026-05-05
status: pending
topic: snapsoccer-authorization-response-package
---

# Task: SnapSoccer Authorization Response Package

## Context

FORGE's Non-GotSport Source Priority Recommendation (filed this session) designates SnapSoccer as Source #3 with a May 17 build target. The recommendation includes a SENTINEL Authorization Request with a May 10 deadline.

SENTINEL intends to authorize SnapSoccer Source #3 by May 10. When that authorization arrives, FORGE needs to be ready to start immediately — no planning lag, no "let me figure out the schedule." The window from authorization (May 10) to build target (May 17) is only 7 days.

---

## What to Build

Create `Infrastructure/SnapSoccer — Authorization Response Package.md`

This document is what FORGE executes the moment SENTINEL says GO on SnapSoccer. It should be ready to act on, not to read.

---

## Required Sections

### 1. Authorization Trigger

The exact response FORGE takes within 1 hour of receiving SENTINEL authorization:

- Files to create or update immediately
- Who to notify (SENTINEL? No one? Just proceed?)
- What the "authorization received" acknowledgment looks like in FORGE's next briefing

### 2. Build Schedule (May 10–17 window)

Day-by-day or phase-by-phase plan for the 7-day build window. Use the existing design documents (`Infrastructure/SnapSoccer Ingestion Design.md`, `Infrastructure/SnapSoccer Test Extraction Plan.md`) as the basis — do not re-derive the work. Map the work to calendar days.

| Day | Phase | Deliverable | Dependencies |
|-----|-------|------------|--------------|
| May 10 (Mon) | Authorization receipt | This package activated | SENTINEL GO |
| May 11 (Tue) | ... | ... | ... |
| ... | | | |
| May 17 (Mon) | ... | First run ready | ... |

If the build cannot be completed in 7 days from May 10, state the earliest realistic completion date and what the critical path is.

### 3. Engineering Dependencies

List every external dependency in the build:
- SnapSoccer API access or scraping approach (is this confirmed? does it require any credentials or rate-limit awareness?)
- Parser reuse from SincSports — confirm which modules are reused and which are new
- Schema additions or database changes required before first run
- Coordination with ELO required (if new game data affects calibration timing)

### 4. SENTINEL Milestone Reporting

Define what FORGE will report to SENTINEL at each phase completion. SENTINEL should not have to ask for status — FORGE's briefings should include a "SnapSoccer Build Status" section from May 10 until first run confirmed.

| Milestone | FORGE Briefing Signal | Expected Date |
|-----------|----------------------|---------------|
| Build start confirmed | "SnapSoccer build initiated" | May 10 |
| Parser complete | "SnapSoccer parser built and unit-tested" | [FORGE fills] |
| Test extraction successful | "SnapSoccer test run: N games extracted" | [FORGE fills] |
| First production run | "SnapSoccer Source #3 live: N games ingested" | May 17 (target) |

### 5. Risk Log

3–5 risks specific to the SnapSoccer build. Not general pipeline risks — risks that could push the May 17 target. For each: probability, impact on timeline, mitigation.

---

## Deliverable

`Infrastructure/SnapSoccer — Authorization Response Package.md`

Pre-built and ready to activate on May 10. Confirm delivery in briefing.

This document exists so SENTINEL can authorize on May 10 with confidence that the May 17 target is achievable and monitored.
