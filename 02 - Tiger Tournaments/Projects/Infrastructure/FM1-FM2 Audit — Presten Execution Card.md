---
title: FM1-FM2 Audit — Presten Execution Card
tags: [infrastructure, pipeline, fm1, fm2, ga-aspire, ecnl, audit, forge]
created: 2026-04-29
updated: 2026-04-29
author: FORGE
status: ready — Presten runs April 30; FORGE closes both audits same day
---

# FM1-FM2 Audit — Presten Execution Card

FM1 and FM2 pre-drafts are complete and waiting for two code-search results. Presten runs two grep commands (~15 min), FORGE closes both audits in under 30 min.

Open `Infrastructure/Pipeline Code Review Results.md` alongside this card. Paste results directly into it as you run each command.

---

## Section 1 — The Two Commands Presten Runs

### FM1: GA ASPIRE Classifier

Run all three commands from your project root. Paste output into FM1-1, FM1-2, FM1-3 fields in `Pipeline Code Review Results.md`.

**Command FM1-A** — Find `ga_aspire` in the codebase
```bash
grep -r "ga_aspire" . --include="*.js" --include="*.ts" --include="*.py" -l
```
What this checks: whether the `ga_aspire` string exists anywhere in the codebase (classifiers, lookup tables, inline constants).
What to paste: the list of file paths returned, or "no results" if empty.
Estimated time: 1–2 min.

**Command FM1-B** — Find event_tier assignment logic
```bash
grep -r "event_tier" . --include="*.js" --include="*.ts" --include="*.py" -n | grep -v "test" | head -30
```
What this checks: where in the code `event_tier` is assigned — this is where the GA ASPIRE rule must live.
What to paste: the matching lines (file:line format). Note any line where `event_tier` is set or mapped.
Estimated time: 1–2 min.

**Command FM1-C** — Check ON CONFLICT revert risk
```bash
grep -r "ON CONFLICT" . --include="*.js" --include="*.ts" --include="*.py" -n | grep -i "event_tier"
```
What this checks: whether re-scraping would overwrite the April 28 `event_tier` fix.
What to paste: any matching lines, or "no results" if empty.
Estimated time: 1 min.

---

### FM2: ecnl_verified Scraper Write Path

Run all three commands from your project root. Paste output into FM2-1, FM2-2, FM2-3 fields in `Pipeline Code Review Results.md`.

**Command FM2-A** — Find `ecnl_verified` in the codebase
```bash
grep -r "ecnl_verified" . --include="*.js" --include="*.ts" --include="*.py" -n
```
What this checks: whether the TGS scraper writes `ecnl_verified` to game records at ingest time.
What to paste: all matching lines (file:line format). Note whether the match is in a scraper file vs. a migration or comment.
Estimated time: 1–2 min.

**Command FM2-B** — Check if the population plan code change has been applied
```bash
grep -r "ecnl_verified.*true\|ecnl_verified: true" . --include="*.js" --include="*.ts" --include="*.py" -n
```
What this checks: whether the scraper already sets `ecnl_verified = true` for qualifying ECNL games.
What to paste: matching lines, or "no results" if not applied.
Estimated time: 1 min.

**Command FM2-C** — Check if fix is on the May 1 plan (only run if FM2-A and FM2-B return "not found")
Open `Infrastructure/May 1 Deployment — Day-Of Runbook.md`. Is `ecnl_verified` scraper fix listed?
What to paste: "YES — listed in runbook" or "NO — not in runbook."
Estimated time: 1 min.

---

## Section 2 — What FORGE Does on Receipt (the 30-minute clock)

FORGE completes both audits within 30 minutes of Presten pasting results into `Pipeline Code Review Results.md`.

### For FM1:

1. Read FM1 path (A/B/C) from `Pipeline Code Review Results.md`
2. Open `Infrastructure/FM1 Audit — Pre-Draft.md` → fill Section 3 Result Table and Section 4 Verdict with Presten's findings — 10 minutes
3. Save completed audit to `Infrastructure/FM1 Audit.md` (remove pre-draft header)
4. Update frontmatter: `status: completed`, `fm1_path: [A/B/C]`
5. If Path B: document the specific code change required (file:line from FM1-B result)
6. If Path C: file SENTINEL queue item (urgent) within 2 hours

### For FM2:

1. Read FM2 path (A/B/C) from `Pipeline Code Review Results.md`
2. Open `Infrastructure/FM2 Audit — Pre-Draft.md` → fill Section 3 Result Table and Section 4 Verdict — 10 minutes
3. Save completed audit to `Infrastructure/FM2 Audit.md` (remove pre-draft header)
4. Update frontmatter: `status: completed`, `fm2_path: [A/B/C]`
5. If Path B: document the ≤5-line code addition required for TGS scraper
6. If Path C: file SENTINEL queue item (urgent), hold Stage 2 authorization
7. Update `task-2026-04-28-fm1-fm2-audit-pre-draft.md` status to `completed`
8. Notify SENTINEL in next briefing

---

## Section 3 — Why This Matters

**FM1 is checking:** Whether the GotSport pipeline classifier will sustain the April 28 GA ASPIRE `event_tier` fix going forward — without it, every new GA ASPIRE game re-ingested after April 28 erodes the fix daily.

**FM2 is checking:** Whether the TGS scraper writes `ecnl_verified = TRUE` at ingest time for ECNL games — without it, the April 28 backfill is correct for historical records but all future ECNL games default to FALSE, degrading ELO's ECNL CP1 calibration and ECNL migration readiness.

**What depends on these audits:** FM1 determines whether a code change must be deployed before May 1. FM2 determines whether Stage 2 authorization can proceed and whether ELO's migration spec assumptions hold. Both audits have been open (Outcome C) for 6+ days — the only blocker is the two code-search runs.

---

## Footer

**Total Presten time: ~15 min. Total FORGE time post-receipt: ~30 min. No other dependencies.**

If Presten pastes partial output, FORGE will ask for the specific missing lines before completing the audit — FORGE will not guess.

Target: Presten runs commands April 30; FORGE files both audits April 30 EOD.

---

*FORGE — 2026-04-29*
