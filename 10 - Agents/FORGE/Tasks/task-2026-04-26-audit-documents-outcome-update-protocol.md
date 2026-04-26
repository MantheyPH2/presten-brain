---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-26
status: pending
priority: high
due: 2026-04-28
deliverable: "Both audit documents updated with Presten's FM1/FM2 results"
---

# Task: Audit Documents — Outcome Update Protocol (Post-FM1/FM2 Results)

## Context

The GA ASPIRE Pipeline Classifier Audit and ecnl_verified Ingestion Path Audit both landed at Outcome C because the pipeline codebase is not in the vault. Both documents are structured with Outcome A/B/C sections, but the Outcome sections are currently written as conditional ("if this is the case, then…"). They contain no actual verdict.

When Presten completes the FM1/FM2 code review (via `Infrastructure/FM1-FM2 Code Search — Presten Quick-Run Guide.md`) and files results in `Pipeline Code Review Results.md`, FORGE must update both audit documents to reflect the actual outcome — within 2 hours of receiving Presten's results. SENTINEL cannot close the FM1/FM2 risk loop until both documents show a final, filled-in verdict.

This task pre-builds the update protocol so FORGE knows exactly what to change in each document for each outcome path, without re-reading the full audit document.

## What to Do

This task has no new deliverable file. FORGE updates the two existing audit documents in place:

1. `02 - Tiger Tournaments/Projects/Infrastructure/GA ASPIRE Pipeline Classifier Audit — April 2026.md`
2. `02 - Tiger Tournaments/Projects/Infrastructure/ecnl_verified Ingestion Path Audit — April 2026.md`

And updates the frontmatter status of both from `outcome: C — pending Presten code review` (or equivalent) to the actual path letter.

---

## Update Protocol

### Step 1 — Read Presten's FM1/FM2 results

Open `Infrastructure/Pipeline Code Review Results.md`. Read:
- FM1 Path Letter (A, B, or C) and FM1-1/FM1-2/FM1-3 notes
- FM2 Path Letter (A, B, or C) and FM2-1/FM2-2/FM2-3 notes

### Step 2 — Update the GA ASPIRE Classifier Audit

Open `Infrastructure/GA ASPIRE Pipeline Classifier Audit — April 2026.md`.

In the **frontmatter**, update:
- `status:` from current value to one of: `outcome-A-cleared`, `outcome-B-fix-required`, `outcome-C-confirmed`
- Add `outcome_confirmed: YYYY-MM-DD` with the date of Presten's code review
- Add `fm1_path: A | B | C`

In **Section 3 (Audit Result)**:
- Locate the sub-section matching FM1's actual path (A, B, or C)
- Add a line at the top of that sub-section: `**CONFIRMED [date] — Presten code review. See Pipeline Code Review Results.md FM1.**`
- If Path B: fill in the exact code change Presten identified (file:line from FM1-2, pattern from FM1-1). Do not leave the code change template blank.
- If Path A: add one sentence confirming the specific rule Presten found (from FM1-1 and FM1-2 notes). No action needed line is already there.

In **Section 4 (ecnl_verified Parallel Check)**:
- Note: "FM2 results are in ecnl_verified Ingestion Path Audit. See FM2 path in Pipeline Code Review Results.md."

### Step 3 — Update the ecnl_verified Ingestion Path Audit

Open `Infrastructure/ecnl_verified Ingestion Path Audit — April 2026.md`.

In the **frontmatter**, update:
- `status:` to one of: `outcome-A-cleared`, `outcome-B-fix-required`, `outcome-C-confirmed`
- Add `outcome_confirmed: YYYY-MM-DD`
- Add `fm2_path: A | B | C`

In **Section 3 (Audit Verdict)**:
- Locate the sub-section matching FM2's actual path (A, B, or C)
- Add: `**CONFIRMED [date] — Presten code review. See Pipeline Code Review Results.md FM2.**`
- If Path B: fill in the specific code addition Presten identified (from FM2-1 and FM2-2 notes). The code addition template is already in the document — fill in the file:line.
- If Path A: add one confirming sentence.

In **Section 4 (Interaction with ECNL Migration)**:
- Add one sentence at the bottom noting whether the migration spec's assumptions are affected, based on the actual FM2 outcome:
  - Path A: "ecnl_verified accuracy confirmed for forward ingestion. Migration spec assumptions intact."
  - Path B: "ecnl_verified not set at write time. Migration spec Option 2 (re-tag historical) retains validity for pre-April-28 games; forward ingestion gap requires May 1 scraper fix before migration spec assumptions fully apply."
  - Path C: "Structural gap. SENTINEL has flagged to Presten. Migration spec caveated pending resolution."

### Step 4 — File update confirmation in next FORGE briefing

In the next FORGE briefing, note:
- FM1 Path confirmed: [A/B/C] — Audit document updated
- FM2 Path confirmed: [A/B/C] — Audit document updated
- SENTINEL Disposition in Pipeline Code Review Results.md: awaiting SENTINEL review

---

## Quality Standard

- Both audit documents must be updated within 2 hours of Presten filing FM1/FM2 results
- Frontmatter must reflect the actual outcome, not "pending"
- No conditional language ("if the code change is needed…") in updated sections — replace with confirmed statements
- If FM1 is Path B: the specific code change Presten identified must appear in the document, not the generic template

## Why This Matters

Both audits are Outcome C right now — open, unresolved. The moment Presten fills FM1/FM2, the audits should close to definitive verdicts. If FORGE does not update the documents, SENTINEL's risk log still shows two open Outcome C items heading into April 28 — even after the code review is complete. This update protocol ensures the documents stay authoritative.

## References

- `Infrastructure/Pipeline Code Review Results.md` — source of FM1/FM2 paths
- `Infrastructure/GA ASPIRE Pipeline Classifier Audit — April 2026.md` — update target 1
- `Infrastructure/ecnl_verified Ingestion Path Audit — April 2026.md` — update target 2
- `task-2026-04-26-fm1-fm2-code-search-quick-run-guide.md` — the task that precedes this one
