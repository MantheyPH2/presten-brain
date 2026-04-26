---
title: FM1-FM2 Code Search — Presten Quick-Run Guide
tags: [infrastructure, pipeline, code-review, ga-aspire, ecnl, forge]
created: 2026-04-26
updated: 2026-04-26
author: FORGE
status: ready — Presten completes before April 28
open-alongside: "[[Pipeline Code Review Results]]"
---

# FM1-FM2 Code Search — Presten Quick-Run Guide

```
Target time:    ≤ 5 minutes in terminal
Open alongside: Infrastructure/Pipeline Code Review Results.md
Complete before: April 28 DB session start
```

> Run each command from your project root directory. Paste output into the Notes fields of `Pipeline Code Review Results.md` as you go.

---

## FM1: GA ASPIRE Classifier (fills FM1-1, FM1-2, FM1-3)

### Step 1 — Find `ga_aspire` in the codebase

```bash
grep -r "ga_aspire" . --include="*.js" --include="*.ts" --include="*.py" -l
```

**Expected:** one or more files returned.

- **If files found →** note the file path(s) in FM1-1. Proceed to Step 2.
- **If zero results →** FM1 is Path B or C. Write "not found" in FM1-1. Skip to FM1 Path mapping below.

---

### Step 2 — Find event tier classification logic

```bash
grep -r "event_tier" . --include="*.js" --include="*.ts" --include="*.py" -n | grep -v "test" | head -30
```

Look for lines where `event_tier` is assigned or mapped (e.g., `event_tier = ...` or `event_tier:` in an object). Note the `file:line` in FM1-2.

---

### Step 3 — Check for ON CONFLICT revert risk

```bash
grep -r "ON CONFLICT" . --include="*.js" --include="*.ts" --include="*.py" -n | grep -i "event_tier"
```

- **If results include `event_tier` →** FM1-3 is a **FAIL**. The April 28 fix will revert on re-scrape.
- **If empty →** FM1-3 is a **PASS**. Write "NO" in FM1-3 notes.

---

### FM1 Path Mapping

| What you found | Path |
|----------------|------|
| `ga_aspire` found in classifier map with correct assignment value | **A — Safe** |
| `ga_aspire` not found; classifier uses a simple lookup dict/table | **B — Fixable** (≤ 5-line addition) |
| Classifier logic is complex or structural | **C — Structural** |

Circle the path letter in `Pipeline Code Review Results.md` → FM1 section.

---

## FM2: ecnl_verified Scraper Write (fills FM2-1, FM2-2, FM2-3)

### Step 1 — Find `ecnl_verified` in TGS scraper

```bash
grep -r "ecnl_verified" . --include="*.js" --include="*.ts" --include="*.py" -n
```

- **If found →** note `file:line` in FM2-1. Proceed to Step 2.
- **If not found →** FM2-1 is a **FAIL** (column not written by scraper). Write "not found" in FM2-1. Skip to Step 3.

---

### Step 2 — Check if the Population Plan code change has been applied

```bash
grep -r "ecnl_verified.*true\|ecnl_verified: true" . --include="*.js" --include="*.ts" --include="*.py" -n
```

- **If found →** FM2-2 **PASS**. Write "YES — applied" in FM2-2.
- **If not found →** FM2-2 **FAIL**. Write "NO — not applied" in FM2-2. Proceed to Step 3.

---

### Step 3 — If not applied, confirm it is on the May 1 plan

Open `Infrastructure/May 1 Deployment — Day-Of Runbook.md`. Is `ecnl_verified` scraper fix listed?

- **YES →** FM2-3 **PASS**
- **NO →** FM2-3 **FAIL** — add it to the runbook before closing this session

---

### FM2 Path Mapping

| What you found | Path |
|----------------|------|
| Scraper writes `ecnl_verified = TRUE` for ECNL games (source + event_tier conditions match) | **A — Safe** |
| Scraper does not write `ecnl_verified`; adding it is a ≤ 5-line change | **B — Missing, simple** |
| Scraper architecture prevents simple field addition | **C — Structural** |

Circle the path letter in `Pipeline Code Review Results.md` → FM2 section.

---

## Results Paste Template

Copy-paste the block below into the Notes sections of `Pipeline Code Review Results.md`:

```
FM1-1 — `ga_aspire` found at: [paste grep output line, or "not found"]
FM1-2 — Classification logic at: [paste file:line]
FM1-3 — ON CONFLICT includes event_tier? YES / NO

FM2-1 — `ecnl_verified` found at: [paste grep output line, or "not found"]
FM2-2 — Code change applied? YES / NO
FM2-3 — On May 1 plan? YES / NO / N/A

FM1 Path: A / B / C
FM2 Path: A / B / C
```

---

## After Completing

1. Fill in the **SENTINEL Disposition** section of `Pipeline Code Review Results.md` header (leave the SENTINEL block blank — SENTINEL fills that)
2. Reference the Decision Tree routing table in `Pipeline Code Review Results.md` to confirm April 28 and May 1 status
3. FORGE will update both audit documents within 2 hours of receiving your results:
   - `Infrastructure/GA ASPIRE Pipeline Classifier Audit — April 2026.md`
   - `Infrastructure/ecnl_verified Ingestion Path Audit — April 2026.md`

---

## References

- `[[Pipeline Code Review Results]]` — the form this guide populates
- `[[GA ASPIRE Pipeline Classifier Audit — April 2026]]` — FM1 source and context
- `[[ecnl_verified Ingestion Path Audit — April 2026]]` — FM2 source and context
- `[[April 29 Pipeline Code Fail — Decision Tree]]` — routing table after FM1/FM2 paths known

*FORGE — 2026-04-26*
