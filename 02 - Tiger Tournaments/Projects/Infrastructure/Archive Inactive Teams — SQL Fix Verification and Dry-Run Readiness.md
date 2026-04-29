---
title: Archive Inactive Teams — SQL Fix Verification and Dry-Run Readiness
type: infrastructure-doc
agent: FORGE
filed: 2026-04-28
status: filed — awaiting psql verification at April 29 session
due: 2026-04-29
---

# Archive Inactive Teams — SQL Fix Verification and Dry-Run Readiness

> FORGE-authored readiness document. Sections 1–5 populated from vault documents. Section 1 "Expected candidate count" requires psql confirmation at April 29 session open — noted explicitly where verification is needed.

---

## Section 1: SQL Fix Verification

| Check | Result |
|-------|--------|
| GROUP BY bug: original symptom | `g.event_name` included in both `SELECT` and `GROUP BY` in `archive-inactive-teams.js` candidate query. One row per team-event pair produced instead of one row per team_id. An inactive team with 3 historical events generated 3 archive candidate rows — causing inflated candidate count and misleading dry-run output. |
| Fix applied: what changed | Remove `g.event_name` from the `GROUP BY` clause and from the `SELECT` column list. Replace with a scalar subquery: `(SELECT g2.event_name FROM games g2 WHERE g2.team_id = t.id ORDER BY g2.game_date DESC LIMIT 1) AS last_event_name` — returns the most recent event name without expanding the GROUP BY. |
| Fix specified in | `task-2026-04-26-archive-sql-fix-and-dry-run.md` — exact SQL fix, rationale, and one-line change documentation |
| Fix reviewed in | `Infrastructure/Archive Workflow.md` — authoritative spec (GROUP BY bug corrected April 27) |
| Expected output with fix | **UNCONFIRMED from vault alone — requires psql or code inspection at April 29 session** (see note below) |
| Expected candidate count range | 155–165 teams (from `task-2026-04-28-april28-session-outcome-report.md` Section 5, and `task-2026-04-28-archive-sql-fix-ready-state-verification.md`) |
| Other known bugs or risks | NONE documented in vault. The GROUP BY fix is the only flagged SQL issue. `archived_at` schema migration confirmed complete as of 2026-04-22. |
| SQL fix source file | `archive-inactive-teams.js` (project codebase — exact path to be confirmed with Presten at April 29 session) |

**VERIFICATION NOTE:** FORGE cannot confirm from vault documents alone that the fix has been applied to the production codebase. At the April 29 session open, Presten should run:

```bash
# Confirm fix is applied — should NOT see "event_name" in GROUP BY
grep -A 20 "GROUP BY" archive-inactive-teams.js
```

Expected output: GROUP BY clause contains only `t.id` (and possibly `t.name`, `t.last_game_date`) — `g.event_name` must NOT appear.

If `g.event_name` is still in the GROUP BY: **STOP — the fix was not applied. Do not run --dry-run. File SENTINEL queue item: `task-archive-fix-not-applied-YYYY-MM-DD.md` before proceeding.**

---

## Section 2: Exact Dry-Run Command

```bash
# Copy-paste exactly. No modifications needed.
node archive-inactive-teams.js --dry-run
```

**Expected output characteristics:**
- Lists teams meeting archive criteria (last game date > 180 days ago, no active org-ID in current crawl config)
- Shows count of candidate teams as single-row result per team_id (not per team-event pair)
- Does NOT modify any database records
- Script exits with code 0 (no SQL error)
- Run time: approximately 5–30 seconds (depends on query plan; first run may be slower)

**Threshold Reference:**

| Result | Candidate Count | FORGE Response |
|--------|----------------|---------------|
| **GREEN** | 155–165 teams | File dry-run authorization request within 30 min |
| **YELLOW** | 130–190 (outside GREEN band) | Log and share full output with FORGE before proceeding |
| **RED** | < 100 or > 250, or any SQL error | Do NOT proceed to authorization; share output with FORGE immediately |

**Threshold discrepancy note:** The dry-run authorization request template (`task-2026-04-28-archive-dry-run-authorization-request.md`) specifies GREEN = 80–150 teams. The verification task and session outcome report specify 155–165. This discrepancy exists across two documents. FORGE flags this for SENTINEL reconciliation. FORGE will use 130–190 as the YELLOW band and flag any result outside 155–165 for review — but will not block authorization for counts in the 130–190 range pending SENTINEL clarification.

---

## Section 3: Post-Dry-Run FORGE Response Protocol

| Scenario | FORGE Response | Time Budget |
|----------|---------------|------------|
| GREEN (155–165 candidates) | Files `Infrastructure/Archive Inactive Teams — Dry-Run Authorization Request.md` (filled template) to SENTINEL | < 30 min |
| YELLOW (130–190, outside GREEN band) | Analyzes candidate list for anomalies (active clubs, unexpected names); files authorization request with anomaly note; SENTINEL decides | < 1 hour |
| RED (< 100, > 250, or SQL error) | Files defect report; identifies whether GROUP BY fix was applied correctly or whether a new error was introduced; does NOT file authorization request | < 2 hours |

---

## Section 4: Presten Execution Instructions

**What to do (5 minutes):**

1. Navigate to the project directory containing `archive-inactive-teams.js`
2. Confirm GROUP BY fix is applied (grep check in Section 1 above)
3. Run the exact command in Section 2: `node archive-inactive-teams.js --dry-run`
4. Copy the full terminal output (including candidate count line and any SQL output)
5. Share output with FORGE: paste in session or file at `Infrastructure/Archive Dry-Run Output — 2026-04-[DATE].md`

**What NOT to do:**
- Do not run without `--dry-run` flag — that executes the production archive (irreversible without rollback)
- Do not filter or truncate the output before sharing — FORGE needs the full text to verify the candidate list
- Do not proceed to authorization if any SQL error appears in the output

---

## Section 5: Authorization Sequence After Dry-Run

1. Presten shares `--dry-run` output → FORGE reviews (< 30 min)
2. FORGE fills `Infrastructure/Archive Inactive Teams — Dry-Run Authorization Request.md`:
   - Candidate count from dry-run
   - GREEN/YELLOW/RED verdict
   - Spot-check of candidate team names (any active/known clubs? flag or "none")
   - SENTINEL authorization block (Section 4 of that template)
3. SENTINEL reviews and issues GO / DEFER
4. Only after SENTINEL GO: Presten runs `node archive-inactive-teams.js` (no `--dry-run` flag)
5. FORGE files post-execution confirmation within 30 minutes of production execution

**Production execution note:** Production run archives teams by setting `archived = true` and `archived_at = NOW()` for all candidates. This does NOT delete data. Rollback: `UPDATE teams SET archived = false, archived_at = NULL WHERE archived_at >= '[run timestamp]'`

---

## Why This Matters

The archive has been overdue since April 26. Stale inactive teams accumulate in every pipeline run (Steps 6–8), inflating Glicko-2 RD for active teams that happen to share state/league segments with inactive teams. May 1 is the daily pipeline launch date — running the first week of production pipeline with 155+ stale inactive teams in the active set degrades data quality from Day 1. The single remaining blocker is the dry-run output. The SQL fix is specified. The authorization template is ready. The production execution template is filed. This document ensures Presten's 5-minute dry-run produces immediate FORGE authorization action.

---

## References

- `task-2026-04-26-archive-sql-fix-and-dry-run.md` — fix specification (GROUP BY bug exact description)
- `Infrastructure/Archive Workflow.md` — authoritative spec (corrected post April 27 fix)
- `task-2026-04-28-archive-dry-run-authorization-request.md` — dry-run authorization request template (FORGE fills after dry-run)
- `Infrastructure/Archive Inactive Teams — Production Execution Authorization Request.md` — production authorization template (FORGE fills after dry-run authorized)
- `task-2026-04-22-archive-workflow.md` — original archive workflow spec

*FORGE — 2026-04-28 @ 17:25*
