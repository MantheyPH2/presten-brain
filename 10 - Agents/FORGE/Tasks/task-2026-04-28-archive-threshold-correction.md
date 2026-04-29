---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-28
status: completed
completed: 2026-04-29
completion_note: GREEN threshold corrected to 155-165, YELLOW to 140-154 or 166-185, RED to <140 or >185. Updated threshold band example in Section 1 accordingly. Metadata updated to 2026-04-29.
priority: urgent
due: 2026-04-29 (before archive dry-run executes — could be tonight if Presten provides --dry-run output)
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/Archive Inactive Teams — Production Execution Authorization Request.md (Section 2 threshold correction)"
---

# Task: Archive Candidate Count Threshold — Correct Authorization Request Template

## Context

SENTINEL ruled at the 19:41 session: **155–165 is the authoritative GREEN band for archive candidate count.** The April 28 SQL Fix Verification document (more recent, produced with specific scope analysis) is authoritative over the earlier authorization request template which states 80–150.

FORGE's authorization request template currently contains an incorrect GREEN band (80–150). If Presten runs the `--dry-run` and sees a count in the 155–165 range, the template will read that as YELLOW or RED — incorrectly. This must be corrected before the dry-run executes. The dry-run could trigger any session once Presten provides the output.

## What to Do

Open: `02 - Tiger Tournaments/Projects/Infrastructure/Archive Inactive Teams — Production Execution Authorization Request.md`

Find the candidate count threshold table (Section 2 or equivalent). Update it:

| Band | Old Value | Correct Value |
|------|-----------|---------------|
| GREEN | 80–150 | 155–165 |
| YELLOW | 130–190 | 140–154 or 166–185 (±10% buffer around GREEN) |
| RED | <80 or >190 | <140 or >185 |

**Important:** The YELLOW and RED bands should be symmetric around the GREEN range. Reconstruct from the principle: GREEN is the expected count based on scope analysis; YELLOW is ±10%; RED is anything outside ±20%.

Also update the document's "last reviewed" or metadata field if one exists to note: "Threshold corrected per SENTINEL ruling 2026-04-28."

## Definition of Done

- GREEN band reads 155–165 in the authorization request template
- YELLOW and RED bands updated consistently
- FORGE confirms correction in next briefing
- SENTINEL notified so authorization block can be completed accurately when dry-run runs

## Why This Matters

SENTINEL signs the archive authorization after reviewing the dry-run output against the candidate count threshold. If the threshold is wrong, SENTINEL may block a valid dry-run or approve an invalid one. This is a data quality gate — the threshold must be correct before the gate fires.

## References

- `Infrastructure/Archive Inactive Teams — SQL Fix Verification and Dry-Run Readiness.md` — authoritative 155–165 threshold document
- `Infrastructure/Archive Inactive Teams — Production Execution Authorization Request.md` — document to correct
- SENTINEL briefing 2026-04-28-19:41.md — ruling on authoritative threshold
