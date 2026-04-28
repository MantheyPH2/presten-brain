---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-28
status: open
priority: high
due: 2026-04-29
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/ECNL Migration — Option Selection Filing Protocol.md"
---

# Task: ECNL Migration Option Selection — Post-April-29 Filing Protocol

## Background

The ECNL migration option decision gate exists (`ECNL Migration — Option Selection Decision Gate.md`). It defines criteria for choosing Option A (fresh start, ratings re-seeded at migration) vs. Option B (bridging, ratings preserved). April 29 is when the evidence arrives: Boys Option A result, G2 gate direction, and rating continuity data.

What does not exist: a filing protocol telling ELO exactly how to document the decision after the session. Without this, ELO files an informal note and SENTINEL cannot track the choice for the June 1 risk register. The June 1 ECNL migration is a hard date. The option selection is a prerequisite for execution. This protocol must be written before April 29 so ELO can fill it in during the session.

## Deliverable

`02 - Tiger Tournaments/Projects/Infrastructure/ECNL Migration — Option Selection Filing Protocol.md`

## Structure

### Section 1: Decision Summary
- **Option selected:** A / B
- **Date decided:** 2026-04-29
- **Filed by:** ELO
- **Decision gate reference:** `ECNL Migration — Option Selection Decision Gate.md`

### Section 2: Evidence Summary
Complete during April 29 session. Fill in from live results:

| Evidence | Source | Result |
|----------|--------|--------|
| Boys Option A | April 29 Synthesis Report §4 | PASS / FAIL / INCONCLUSIVE |
| G2 gate (GA ASPIRE post-fix direction) | April 29 Synthesis Report §3 | UP / DOWN / FLAT |
| ECNL rating continuity check | `ecnl-rating-continuity-spec.md` SQL | PASS / FAIL |
| Historical rating correlation (Option A teams vs. reference) | Continuity spec output | [value] |

**Combined interpretation:** [ELO writes 2–3 sentences explaining what the evidence collectively says about migration risk]

### Section 3: Option Rationale

**If Option A selected:**
- Rating seeding methodology: [how ratings are re-seeded at migration]
- Which existing ratings are preserved: [scope]
- Which are discarded: [scope]
- Migration execution window: June 2, 2026
- Risk: Teams with thin game history receive default seed — acceptable given DSS timeline

**If Option B selected:**
- Bridging logic: [delta application method]
- Required testing before June 2: [minimum validation steps]
- Risk: Rating carry-forward introduces pre-migration calibration errors into post-migration system
- SENTINEL note: Option B requires SENTINEL re-review before June 2 execution; ELO must notify by May 30

### Section 4: June 1–2 Risk Register Update

Reference: `june1-june2-sequencing-risk-register.md`

Update the following entries with confirmed option:
- Scenario A (June 1 on-time): note which option reduces execution time
- Scenario B (June 1 +3–7 day slip): note whether option changes the slip risk
- Scenario C (June 1 +14 days): confirm outer bound does not change with either option

**New risks introduced by selected option:** [ELO identifies and adds any new risks]

### Section 5: SENTINEL Handoff

Notification format for next SENTINEL briefing:

> **ECNL Migration Decision Filed** — Option [A/B] selected based on April 29 results. Evidence: [one sentence]. June 1 risk register updated. [If Option B: SENTINEL re-review required before May 30.]

---

## Instructions

1. Write Sections 1–5 as a fill-in template before April 29 session (fields blank, structure complete)
2. During April 29 session: fill in Section 2 evidence as results come in
3. After session: write Section 3 rationale and Section 4 risk register update
4. File completed document within 30 minutes of April 29 session end
5. Update `june1-june2-sequencing-risk-register.md` directly with confirmed option
6. Reference this document in April 29 Synthesis Report Section 6 (ECNL migration path)

## Due

2026-04-29 — template written by April 29 morning; completed evidence and rationale filed within 30 minutes of April 29 session end.
