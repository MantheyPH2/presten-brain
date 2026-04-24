---
title: Rank Bands — Presten Decision Brief
type: decision-brief
author: ELO
date: 2026-04-24
status: ready
related: "[[Rank Bands Threshold Validation — 2026-04-24]]", "[[Rank Bands — Implementation Plan]]"
tags: [rank-bands, decision, dss, evo-draw]
---

# Rank Bands — Presten Decision Brief

> **Read time:** Under 5 minutes.
> **Purpose:** Tells you what to check before starting the 5.5-hour implementation session, and whether any decision is needed first.

---

## Section 1: Implementation Readiness

The threshold validation queries are written and ready to run. The planned thresholds (from [[Rank Bands — Implementation Plan]]) are:

| Band   | Threshold     | Validation Status              | Ready?                   |
|--------|---------------|-------------------------------|--------------------------|
| Gold   | ≥ 1500        | Pre-flight queries ready       | Yes — run pre-flight first |
| Silver | 1350–1499     | Pre-flight queries ready       | Yes — run pre-flight first |
| Bronze | 1200–1349     | Pre-flight queries ready       | Yes — run pre-flight first |
| Red    | 1050–1199     | Auto-passes if above 3 pass   | Yes                      |
| Blue   | 900–1049      | Auto-passes if above 3 pass   | Yes                      |
| Green  | < 900         | Auto-passes if above 3 pass   | Yes                      |

The thresholds have not yet been validated against live DB data — that validation is a 15-minute pre-flight you run at the start of the session. The [[Rank Bands Threshold Validation — 2026-04-24]] document contains all queries needed.

**If all three upper bands pass:** proceed to implementation immediately.

**If any upper band fails:** run the adjustment algorithm in the validation doc (Step 5). This takes 10 minutes and produces corrected thresholds before you write a single line of code.

---

## Section 2: Pre-Session Decisions Required

**Decision 1: Run the pre-flight validation before starting implementation.**
- ELO recommendation: Run Steps 2, 4, and 6 from [[Rank Bands Threshold Validation — 2026-04-24]] before opening the implementation plan.
- Time: 15 minutes.
- Outcome: Either "proceed with planned thresholds" or "use adjusted thresholds." Both outcomes are handled in the validation doc.

**Decision 2: Football Academy NJ must be Gold.**
- If Football Academy NJ is Silver at the planned thresholds (rating 1450–1499), lower Gold to ≥ 1450 before proceeding.
- If Football Academy NJ appears twice (split record), run the merge queries from [[Pre-DSS Team Merge Audit — U13G]] before starting implementation. This is a demo-blocking issue.

No other decisions are required before starting.

---

## Section 3: Implementation Session Scope

The 5.5-hour session implements Rank Bands end-to-end: database view (`rankings_with_bands`), API update, and frontend badge rendering. Full scope is in [[Rank Bands — Implementation Plan]]. Session is self-contained — no external dependencies once the pre-flight passes.

---

## Section 4: Post-Implementation Verification

After the session completes, ELO will confirm:

1. **Band distribution check** — re-run the Step 2 query from the threshold validation doc. Gold should be 0.5–2% of U13G; Silver 2–8%; Bronze 12–28%.
2. **Football Academy NJ band confirmation** — re-run the Step 6 demo team spot check. Must return Gold.
3. **Top-10 U13G** — all 10 teams should have a non-null band. Any null = view logic issue.

If any check fails, ELO will diagnose and file a follow-up task before the DSS data validation run (May 14). You do not need to stay online for this — ELO will confirm via briefing.

---

## Related

- [[Rank Bands Threshold Validation — 2026-04-24]] — run this first (15-minute pre-flight)
- [[Rank Bands — Implementation Plan]] — the 5.5-hour session
- [[Pre-DSS Team Merge Audit — U13G]] — Football Academy NJ merge investigation (run if team appears twice)
- [[DSS Demo Spec — May 2026]] — Football Academy NJ as Gold anchor requirement
