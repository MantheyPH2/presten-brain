---
title: May 9 Readiness — Gap Closure Action Plan
type: gap-closure-plan
author: ELO
date: 2026-04-26
status: active — living document, updated each session
last_updated: 2026-04-26
related: "[[May 9 DSS Readiness Final Assessment Spec]]", "[[DSS Feature Readiness Tracker — May 2026]]", "[[Boys Option A — Decision Document]]", "[[Post-Fix Calibration Stability Monitoring Spec]]", "[[ECNL Migration — Option Selection Decision Gate]]"
tags: [dss, may9, readiness, gap-closure, action-plan, evo-draw]
---

# May 9 Readiness — Gap Closure Action Plan

> **Purpose:** Single document showing what is open between now (April 26) and May 9, who closes each item, when it must be done, and what breaks downstream if it slips. ELO updates this document when items close or timelines shift. SENTINEL references it every briefing cycle.
>
> **May 9 gate:** Final DSS go/no-go. May 16 is DSS. No changes after May 14 noon.

---

## Section 1: May 9 Gate Criteria (from May 9 DSS Readiness Final Assessment Spec)

Five features are assessed on May 9. Each has a binary PASS/FAIL threshold:

1. **Rank Bands:** Football Academy NJ U13G rated ≥ 1500 (Gold) and found in rankings. Any anchor team not found = ❌.
2. **Event Strength Phase 1:** `event_strength` table exists; at least one ECNL event is `strength_class = 'High'`; no ECNL National event is `'Low'`. Table not existing = ❌.
3. **Club Rankings:** `club_rankings` table exists; Girls coverage ≥ 60% of ranked Girls teams. Boys included only if Boys Option A passed. Table not existing = ❌.
4. **USA Rank Comparison:** FORGE's 16 audit queries run; delta results file exists; ELO delta review complete; anchor teams found in rankings. Not run by May 9 = ❌ (non-negotiable — escalate to Presten directly).
5. **USYS/GotSport Coverage:** TX test crawl completed; ≥ 1,000 new USYS games confirmed in DB. If < 1,000 or not run → ⚠️ PARTIAL (drop specific game count claim but use coverage expansion language).

If ≥ 2 features are ❌ on May 9 → automatic fallback demo: Rank Bands + team detail + USA Rank Comparison only.

---

## Section 2: Open Items — Closure Tracking Table

*ELO updates this table as items close. Mark Status = "✅ Closed [date]" when done.*

| # | Item | Responsible | Must-Close-By | Current Status (Apr 26) | What Blocks It | Risk If Late |
|---|------|-------------|--------------|------------------------|---------------|-------------|
| 1 | April 28 GA ASPIRE fix + full recompute | Presten | **Apr 28** | Scheduled — execution log docs ready | Presten must run UPDATE + recompute session | Everything else depends on this. If April 28 slips, all dates below shift. |
| 2 | Girls GA ASPIRE calibration confirmed (G2 gate) | ELO | **Apr 29** | Pending April 28 execution | April 28 must run and recompute complete | Blocks Girls Calibration Narrative, Event Strength Phase 1 authorization. |
| 3 | Event Strength Phase 1 authorized (G4 gate) | ELO → SENTINEL | **Apr 29** (authorization); **May 5** (implementation) | Pending G4 gate Apr 29 | G4 must pass; SENTINEL review required | If G4 fails Apr 29, Phase 1 session (7h) shifts; may not complete before May 9. |
| 4 | ECNL RL staleness CP1 check | ELO | **Apr 29** | Pending — first session action | Presten must be in psql Apr 29 | If CP1 fails, Option 1 disqualified; Option 2 escalation must start Apr 29 to make May 10 decision. |
| 5 | Post-fix calibration stability baseline filed | ELO | **Apr 29** | Template ready at Reports/ | G1 gate must pass first | Stability monitoring has no anchor. All May 5 and May 9 checks lose their baseline. |
| 6 | FM1/FM2 pipeline code path confirmed | Presten | **Apr 29** (optimal) | UNKNOWN — Presten must run quick-run guide | Presten browser session + pipeline check | If unresolved entering May 1: deployment confidence LOW. May 1 first pipeline run at risk. |
| 7 | May 1 pipeline first run GREEN | FORGE + Presten | **May 1** | Pending FM1/FM2 confirmation + deployment | Item 6 must resolve first | If RED: Stage 2 crawl expansion HELD. Infrastructure track stalls. |
| 8 | TX USYS org ID confirmed + test crawl run | Presten | **May 3** | Not started — Presten browser session needed | Presten must visit system.gotsport.com | If not done by May 5: drop USYS coverage claim from DSS script entirely. |
| 9 | USA Rank comparison — FORGE 16 queries run | Presten | **May 3** | Pending Presten psql access | Presten psql session with FORGE queries | If not done by May 5: ELO cannot complete delta review; USA Rank comparison is ❌ on May 9. Non-negotiable item. |
| 10 | ELO delta review of USA Rank results | ELO | **May 5** | Pending item 9 | Item 9 (FORGE queries run) | If ELO receives results May 5 → same-day turnaround needed. Tight but doable. |
| 11 | Post-fix calibration stability confirmed | ELO | **May 7** | Monitoring begins Apr 30 | Post-fix behavior (Criteria A/B/C — 3 consecutive runs) | If unstable through May 5: Event Strength Phase 1 and Club Rankings implementation held. |
| 12 | Boys Option A spot check queries run | Presten | **May 5** | Query package ready; window Apr 29–May 5 | Presten must run 3 queries in psql | If not run by May 5: ELO cannot file Boys decision by May 7. Default to Girls-only fallback. |
| 13 | Boys Option A decision filed | ELO | **May 7** | Waiting on item 12 | Item 12 (Presten results) | If Boys FAIL or not filed by May 7: Girls-only fallback activates. FORGE has 2 days to adjust — tight. |
| 14 | Event Strength Phase 1 implementation session | Presten | **May 5** (complete) | Pending Phase 1 authorization (item 3) | G4 gate + SENTINEL authorization | If session not complete by May 7: Event Strength ❌ on May 9. |
| 15 | Club Rankings implementation session | Presten + FORGE | **May 5** (start) | Pending calibration stability (item 11) + Boys decision (item 13) | Items 11, 12, 13 | 10–11h session. Must start by May 5 to complete before May 9. Girls-only fallback reduces to ~7h. |
| 16 | Rank Bands implementation session | Presten | **May 7** (complete) | Pending calibration stability (item 11) | Item 11 | 5.5h session. Must complete by May 7 for May 9 validation. |
| 17 | ECNL Migration option selection filed | ELO | **May 10** | CP1 pending Apr 29; 7-checkpoint campaign | Checkpoints CP1–CP7 (Apr 29–May 10) | If option not selected by May 10: June 1 migration proceeds without structured plan. Rating continuity risk. |
| 18 | Girls post-fix Brier score completion | ELO | **May 7** | Pending sufficient post-fix game data | Corrected ratings need new games to accumulate | Low risk — supporting evidence, not a May 9 gate blocker. |

---

## Section 3: Critical Path

**Critical Path Item 1: April 28 GA ASPIRE Fix + Recompute (Item 1)**

Why critical: This is the root dependency for all calibration-related items (2, 3, 5, 11, 12, 13, 14, 15, 16). Every downstream session assumes corrected ratings exist. If April 28 slips by one day, the April 29 Decision Tree cannot run, Girls Calibration Narrative cannot be filed, and the monitoring baseline has no anchor point. A one-day slip on April 28 compresses every remaining window by one day — including the Club Rankings session, which has zero margin.

Slip effect: Every date in the closure table shifts right by one day. The Club Rankings session (10–11h) and Rank Bands session (5.5h) cannot both fit between May 5 and May 9 if April 28 slips more than one day.

**Critical Path Item 2: USA Rank Comparison — FORGE 16 Queries (Item 9)**

Why critical: USA Rank Comparison is the only non-negotiable DSS feature. SENTINEL's briefing language ("rankings are [X]% more accurate") has no evidence without this comparison. The item requires Presten to run FORGE's queries, ELO to review delta classification, and the results file to exist at `Reports/usarank-accuracy-audit-results-[date].md`. ELO's delta review (Item 10) cannot start until this runs. If Item 9 is not done by May 5, ELO cannot complete Item 10 by May 9.

Slip effect: USA Rank Comparison is ❌ on May 9. Per the May 9 Decision Table, this requires escalating to Presten directly — not SENTINEL first. All DSS accuracy claims are unsupported. Demo reduces to Rank Bands + team detail only.

**Critical Path Item 3: Boys Option A Decision Filed (Item 13)**

Why critical: Club Rankings implementation (Item 15) cannot start without knowing whether Boys are included. If Boys Option A queries are not run by May 5 and the decision is not filed by May 7, FORGE cannot start Boys implementation with time to complete before May 9. The Girls-only fallback spec (`Rankings/Club Rankings — Girls-Only Fallback Spec.md`) is ready, but activating it after May 7 compresses FORGE's window to < 2 days — feasible only because the Girls-only spec was pre-written.

Slip effect: If queries not run by May 5, default to Girls-only fallback immediately. ELO files Girls-only activation in the May 5 briefing. FORGE adjusts scope. Boys are not demoed at DSS regardless of calibration state.

---

## Section 4: Risk Register

| Risk | Probability | Impact | Mitigation |
|------|------------|--------|------------|
| April 28 session delayed or partial | Low | High | All docs filed; execution log captures partial completion; ELO can run partial analysis on whatever data is available |
| FM1/FM2 pipeline path unresolved entering May 1 | Medium | Medium-High | Contingency plan filed; Path B/C actions pre-written in Pipeline Contingency doc |
| Boys Option A queries not run by May 5 | Medium | Medium | Girls-only fallback spec ready (`Rankings/Club Rankings — Girls-Only Fallback Spec.md`); ELO activates same-session on May 5 if no results received |
| Post-fix calibration instability May 1–5 | Low | Medium-High | Monitoring spec filed; 3 queries running from Apr 30; Level 2 protocol holds Phase 1 until ELO clears |
| USA Rank data not collected by May 3 | Medium | High | Presten must be flagged in every briefing from Apr 29 onward — SENTINEL escalation if not done by May 2; drop comparison from DSS if not resolved by May 7 |
| ECNL Option 1 disqualified (CP1 fail Apr 29) | Low | Medium | Option 2 escalation path defined; Decision Gate ready; Presten sign-off on Option 2 is the mitigation — begin escalation same session |
| Event Strength Phase 1 session does not complete by May 7 | Medium | Medium | Feature drops from demo; fallback demo proceeds without Event Strength card |
| Club Rankings session (10–11h) does not complete before May 9 | Medium | High | Girls-only fallback reduces session to ~7h; must start by May 5; FORGE pre-briefed on scope delta |
| Rank Bands implementation session not complete by May 7 | Low | High | Rank Bands is primary demo feature; if ❌ on May 9, full fallback demo triggers; Presten must approve |

---

## Section 5: Update Protocol

**ELO updates this document:**

- **When any row closes:** Change Status to `✅ Closed [date]` and note the closing evidence in one sentence (e.g., "April 28 recompute confirmed complete per execution log ✅ filed 2026-04-28").
- **When any timeline shifts:** Update Must-Close-By and note shift reason in next briefing.
- **When a new open item is identified:** Add a row. ELO assigns it a number in sequence.
- **When Boys Option A result arrives:** Update items 13, 15 immediately. If FAIL: activate Girls-only fallback, mark both items with fallback note.

**SENTINEL uses this document:**

- SENTINEL reviews each briefing cycle. Does not add rows — that is ELO's responsibility.
- SENTINEL may annotate Must-Close-By with "SENTINEL reviewed [date]" if a slipping item needs tracking.
- If USA Rank comparison (Item 9) is not ✅ Closed by May 5: SENTINEL escalates directly to Presten with this document as the reference.

---

## Update History

| Date | Change |
|------|--------|
| 2026-04-26 | Document created. Status reflects April 26 knowledge. All items open. |

---

## References

- `Rankings/May 9 DSS Readiness Final Assessment Spec.md` — gate criteria for Section 1
- `Product & Planning/DSS Feature Readiness Tracker — May 2026.md` — per-feature readiness state
- `Rankings/Boys Option A — Decision Document.md` — Boys decision template and trigger for items 12–13
- `Rankings/Post-Fix Calibration Stability Monitoring Spec.md` — stability monitoring driving items 5, 11
- `Rankings/ECNL Migration — Option Selection Decision Gate.md` — ECNL campaign driving item 17
- `Rankings/Club Rankings — Girls-Only Fallback Spec.md` — fallback spec for item 15
- `Rankings/April 29 ELO Session Master Script.md` — April 29 session sequencing items 2–6 and CP1

*ELO — 2026-04-26*
