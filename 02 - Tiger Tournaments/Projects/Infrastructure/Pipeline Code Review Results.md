---
title: Pipeline Code Review Results
tags: [infrastructure, pipeline, code-review, ga-aspire, ecnl, evo-draw, forge]
created: 2026-04-25
updated: 2026-04-25
author: FORGE
status: ready — Presten completes before April 28
reference: "Infrastructure/April 29 Pipeline Validation Spec.md"
decision-tree: "Infrastructure/April 29 Pipeline Code Fail — Decision Tree.md"
---

# Pipeline Code Review Results

> **Open this document alongside the Appendix of `Infrastructure/April 29 Pipeline Validation Spec.md`. Complete both in one sitting. Target time: 15–20 minutes. Fill in before April 28 DB session starts.**

---

## Header

```
Review Date: ___________
Reviewer: Presten
Reference: Infrastructure/April 29 Pipeline Validation Spec.md — Appendix
Decision Tree: Infrastructure/April 29 Pipeline Code Fail — Decision Tree.md
```

---

## FM1: GA ASPIRE Event Tier — Classifier Check

*Items below are verbatim from Validation Spec Appendix, "GA ASPIRE" section.*

| Check | Item | Result |
|-------|------|--------|
| FM1-1 | Search for `ga_aspire` in the codebase. If found: note file:line. If not found: this is a FAIL — the classifier does not know about `ga_aspire`. | ✅ PASS / ❌ FAIL |
| FM1-2 | Find the `event_tier` classification logic. Confirm it handles GA ASPIRE events distinctly from standard GA. | ✅ PASS / ❌ FAIL |
| FM1-3 | Find the game INSERT/upsert function. Confirm whether `event_tier` is included in an `ON CONFLICT DO UPDATE` clause. If yes: April 28 fix will revert on re-scrape of existing games. | ✅ PASS / ❌ FAIL |

**Notes (Presten fills):**

```
FM1-1 — `ga_aspire` found at (file:line): ___________
FM1-2 — Classification logic location: ___________
FM1-3 — ON CONFLICT DO UPDATE includes event_tier? YES / NO
```

**FM1 Path Letter (circle one): A / B / C**

| Path | Condition |
|------|-----------|
| **A — Safe** | `ga_aspire` is present in the classifier map with the correct calibration value |
| **B — Fixable** | `ga_aspire` is absent; adding it is a single entry in an existing lookup dict/table (≤ 5 lines of code) |
| **C — Structural** | `ga_aspire` handling requires changes beyond a single lookup entry |

---

## FM2: ecnl_verified Column — Scraper Write Check

*Items below are verbatim from Validation Spec Appendix, "ecnl_verified" section.*

| Check | Item | Result |
|-------|------|--------|
| FM2-1 | Search for `ecnl_verified` in `tgs_scraper`. If found: note current value assignment. If not found: column is not yet written by the scraper. | ✅ PASS / ❌ FAIL |
| FM2-2 | Confirm whether the tgs_scraper code change from the Population Plan has been applied (search for `ecnl_verified: true` in the scraper). | ✅ PASS / ❌ FAIL |
| FM2-3 | If not applied: confirm this is on the May 1 session plan before the pipeline launches. | ✅ PASS / ❌ FAIL / N/A |

**Notes (Presten fills):**

```
FM2-1 — `ecnl_verified` found in tgs_scraper at (file:line): ___________ (or: not found)
FM2-2 — Code change (ecnl_verified: true for ECNL games) applied? YES / NO
FM2-3 — If not applied: on May 1 session plan? YES / NO / N/A
```

**FM2 Path Letter (circle one): A / B / C**

| Path | Condition |
|------|-----------|
| **A — Safe** | `tgs_scraper` already writes `ecnl_verified = TRUE` for games where `source IN ('tgs', 'tgs_athleteone') AND event_tier = 'ecnl'` |
| **B — Missing, simple** | `tgs_scraper` does not write `ecnl_verified`; adding `ecnl_verified: true` to the ECNL insert is a ≤ 5-line change |
| **C — Structural** | Scraper architecture prevents a simple field addition (e.g., INSERT payload generated dynamically without direct field mapping) |

---

## Decision Tree Routing

```
FM1 Path: ___   FM2 Path: ___
```

| FM1 | FM2 | April 28 Status | May 1 Status |
|-----|-----|-----------------|--------------|
| A | A | **GO** | **GO** |
| A | B | **GO** | **CONDITIONAL** — `ecnl_verified` scraper fix required before May 1 launch |
| A | C | **GO** | **CONDITIONAL + Stage 2 gated** — scraper fix scoped as separate task; Stage 2 cannot authorize until fix deployed |
| B | A | **HOLD** — do not start April 28 DB session until FM1 code fix is committed | **GO** after fix |
| B | B | **HOLD** — FM1 fix first; FM2 path handled per its own rules after April 28 | **CONDITIONAL** on FM2 fix before May 1 |
| B | C | **HOLD** — FM1 fix first | **CONDITIONAL + Stage 2 gated** |
| C | A | **GO** — FM1 code fix scoped as separate pre-May-1 task | **CONDITIONAL** on FM1 code fix before May 1 |
| C | B | **GO** — April 28 proceeds | **CONDITIONAL** on both FM1 + FM2 code fixes before May 1 |
| C | C | **GO** — April 28 proceeds; FORGE escalates both to SENTINEL within 2 hours | **CONDITIONAL + Stage 2 gated** — both fixes required before May 1; Stage 2 gated on FM2 fix |

**SENTINEL authorization rules:**
- FM1 Path A → auto-approved
- FM1 Path B → SENTINEL must confirm before April 28 DB session starts; FORGE escalates within 2 hours of learning Path B
- FM1 Path C → SENTINEL must confirm before May 1 deployment; April 28 proceeds; FORGE opens new task immediately
- FM2 Path A → auto-approved
- FM2 Path B → no SENTINEL action for April 28; FORGE adds one condition to May 1 Pre-Auth Checklist
- FM2 Path C → SENTINEL holds Stage 2 authorization until scraper fix is confirmed deployed

---

## SENTINEL Disposition

> **SENTINEL fills — do not pre-fill.**

```
Results reviewed: [ ]
FM1: ___   FM2: ___
Decision: PROCEED / HOLD / REMEDIATE
If HOLD or REMEDIATE: ___________
SENTINEL signature: ___________
Review date/time: ___________
```

---

## References

- `Infrastructure/April 29 Pipeline Validation Spec.md` — Appendix: verbatim source of all checklist items above
- `Infrastructure/April 29 Pipeline Code Fail — Decision Tree.md` — full routing logic and escalation rules
- `Infrastructure/April 28 Execution Log.md` — DB session proceeds only after FM1A + FM2A (or SENTINEL authorization under B/C path)

*FORGE — 2026-04-25*
