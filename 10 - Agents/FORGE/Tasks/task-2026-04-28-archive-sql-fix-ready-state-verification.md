---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-28
status: completed
completed: 2026-04-28
completion_note: Document filed at deliverable path. Section 1 GROUP BY fix described from spec documents — expected candidate count 155-165 documented but marked UNCONFIRMED pending psql/grep verification at April 29 session open. Exact dry-run command, GREEN/YELLOW/RED thresholds, and Presten execution instructions all populated. Threshold discrepancy between this doc (155-165) and dry-run authorization request template (80-150) flagged for SENTINEL reconciliation.
priority: high
due: 2026-04-29
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/Archive Inactive Teams — SQL Fix Verification and Dry-Run Readiness.md"
---

# Task: Archive SQL Fix — Ready-State Verification and Dry-Run Readiness

## Context

The `archive-inactive-teams.js` GROUP BY bug was identified April 26 and marked as fixed April 27. The archive dry-run has been overdue since April 26. FORGE has been waiting on Presten to run `--dry-run` — but SENTINEL does not have confirmation that the SQL fix is actually complete, verified, and produces the correct candidate count range (155–165 teams).

The risk: Presten runs `--dry-run` tonight or April 29, the output is wrong (GROUP BY bug still present, or new error introduced), and FORGE discovers the issue reactively during the April 29 session rather than before. This costs an hour of session time.

This task has FORGE verify the fix is correct against the spec, document the exact ready state, and produce a one-page dry-run execution guide Presten can follow without FORGE assistance.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/Archive Inactive Teams — SQL Fix Verification and Dry-Run Readiness.md`

---

### Section 1: SQL Fix Verification

FORGE reads `task-2026-04-26-archive-sql-fix-and-dry-run.md` and `Infrastructure/Archive Workflow.md` and confirms:

| Check | Result |
|-------|--------|
| GROUP BY bug: original symptom | [describe] |
| Fix applied: what changed in the SQL | [describe exact change] |
| Fix verified against: [test or spec document] | |
| Expected output with fix: candidate teams 155–165 | CONFIRMED / UNCONFIRMED |
| Any other known bugs or risks in `archive-inactive-teams.js` | NONE / [describe] |
| SQL fix source file location | [path] |

If FORGE cannot confirm the fix from existing documents alone: note "UNCONFIRMED — requires psql verification at April 29 session open" and flag to SENTINEL.

---

### Section 2: Exact Dry-Run Command

```bash
# Copy-paste exactly. No modifications needed.
node archive-inactive-teams.js --dry-run
```

Expected output characteristics:
- Lists teams meeting archive criteria (inactive + no recent games)
- Shows count of candidates
- Does NOT modify any database records
- Run time: approximately [X] seconds

**GREEN result:** Candidate count 155–165, no SQL errors, script exits 0
**YELLOW result:** Candidate count 130–190 — log and share with FORGE before proceeding
**RED result:** Candidate count < 100 or > 250, or SQL error — do NOT proceed; share output with FORGE immediately

---

### Section 3: Post-Dry-Run FORGE Response Protocol

When Presten shares `--dry-run` output:

| Scenario | FORGE Response | Time |
|----------|---------------|------|
| GREEN (155–165 candidates) | Files `Archive Production Execution Authorization Request.md` immediately | < 30 min |
| YELLOW (130–190) | Analyzes candidate list for anomalies; files authorization with anomaly note | < 1 hour |
| RED (error or out of range) | Files defect report; identifies whether GROUP BY fix was applied correctly | < 2 hours |

---

### Section 4: Presten Execution Instructions

**What to do (5 minutes):**

1. Navigate to the project directory containing `archive-inactive-teams.js`
2. Run the exact command in Section 2
3. Copy the full terminal output (including candidate count line)
4. Share output with FORGE (paste in session or file as `Infrastructure/Archive Dry-Run Output — 2026-04-[DATE].md`)

**What NOT to do:**
- Do not run without `--dry-run` flag — that would execute production archive
- Do not filter the output before sharing — FORGE needs the full text

---

### Section 5: Authorization Sequence After Dry-Run

1. Presten shares output → FORGE reviews (< 30 min)
2. FORGE files `Archive Production Execution Authorization Request.md` with:
   - Candidate count from dry-run
   - GREEN/YELLOW/RED verdict
   - SENTINEL authorization block (Section 5 of that template)
3. SENTINEL reviews authorization request and issues GO / DEFER
4. Only after SENTINEL GO: Presten runs `node archive-inactive-teams.js` (no `--dry-run`)

---

## Definition of Done

- Section 1 SQL fix verification fully populated from existing documents
- Section 2 exact command confirmed correct (matches `Archive Workflow.md` spec)
- Sections 3–5 populated
- Filed at deliverable path by April 29 before Presten's session

## Why This Matters

The archive has been overdue since April 26. Every day of delay accumulates stale teams in the database that depress rankings accuracy for active teams. The single blocker is now the dry-run output — not the SQL fix, not FORGE's readiness. This document ensures that when Presten runs `--dry-run` (5 minutes of work), FORGE's 30-minute response is immediate and SENTINEL can authorize production execution the same day.

## References

- `task-2026-04-26-archive-sql-fix-and-dry-run.md` — fix specification
- `Infrastructure/Archive Workflow.md` — authoritative spec (GROUP BY bug corrected)
- `task-2026-04-28-archive-production-execution-template.md` — production authorization template (ready to file)
- `task-2026-04-22-archive-workflow.md` — original archive workflow spec
