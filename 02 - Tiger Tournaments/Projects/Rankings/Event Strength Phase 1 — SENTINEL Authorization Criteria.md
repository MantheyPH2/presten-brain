---
title: Event Strength Phase 1 — SENTINEL Authorization Criteria
type: authorization-spec
author: ELO
date: 2026-04-25
status: active — awaiting conditions
related: "[[Event Strength Phase 1 — Full Execution Package]]", "[[Post-Fix Calibration Stability Monitoring Spec]]", "[[April 29 Analysis — Results Filing Template]]", "[[Boys Option A Spot Check — Presten Execution Package]]"
tags: [event-strength, authorization, sentinel, calibration, dss, evo-draw]
---

# Event Strength Phase 1 — SENTINEL Authorization Criteria

> **Purpose:** Defines the exact conditions under which SENTINEL may authorize Event Strength Phase 1 to run. ELO files a Phase 1 Clearance Note (Section 4) when all conditions are met. SENTINEL does not authorize Phase 1 before receiving that note.

---

## Section 1: What Phase 1 Changes

Event Strength Phase 1 computes and inserts per-event quality ratings into the `event_strength` table using the current `rankings.rating` values as team-strength proxies. It also (if needed) applies remediation UPDATEs to `events.event_tier` for 6 unconfirmed league circuits.

**Specific changes:**
- **Schema:** Creates `event_strength` table (or truncates it if it already exists)
- **Data inserted:** `avg_team_rating`, `strength_class` (High/Medium/Low), `strength_percentile`, `competitive_spread`, `upset_rate`, `team_count` — one row per (event_id, age_group)
- **What it does NOT change:** No K-factor, no calibration value, no rating update for any team. Phase 1 is a **read from `rankings` + write to `event_strength`** operation. It does not feed back into the Glicko-2 computation for any current-season game.
- **Gender scope:** Both Girls and Boys events are classified. `event_strength` rows are computed for all age groups with sufficient team count (≥4 teams per event).
- **Reversibility:** Phase 1 is **fully reversible** without a full recompute. `TRUNCATE event_strength` undoes all Phase 1 data in seconds. The remediation UPDATEs to `events.event_tier` are also reversible (documented in Phase 1 Execution Package, Section 5 Rollback). No ranking recompute is required for either reversal.

**Why timing matters:** Phase 1 computes event strength from *current ratings*. If Phase 1 runs before the April 28 GA ASPIRE fix, Girls GA ASPIRE event strengths are computed from suppressed ratings — producing systematically low strength scores for those events. After April 28, the corrected ratings produce accurate strength scores. Running Phase 1 before April 28 requires a full Phase 1 re-run after the fix. Running Phase 1 after April 28 stability is confirmed requires only one run.

---

## Section 2: Authorization Gate Conditions

All conditions must be confirmed before ELO files the Phase 1 Clearance Note.

| # | Condition | Source of Confirmation | Pass Threshold | Current Status |
|---|-----------|----------------------|----------------|----------------|
| 1 | April 28 Execution Log: all 3 steps ✅ | `Infrastructure/April 28 Execution Log.md` — SENTINEL Disposition = AUTHORIZED | All 3 steps confirmed ✅; recompute completed | PENDING |
| 2 | April 29 G2 gate: PASS | `Rankings/April 29 Analysis — Results Filing Template.md` — G2 Status field | G2 Status = PASS | PENDING |
| 3 | Calibration stability: Girls GA ASPIRE avg change < 3 pts/run for 3 consecutive runs | `Rankings/Post-Fix Calibration Stability Monitoring Spec.md` — Criterion A | Criterion A met for 3 consecutive pipeline runs (earliest: May 2 if pipeline runs daily) | PENDING |
| 4 | Boys anomaly check: no Boys rating shift > 5 pts in April 29 analysis | `Rankings/April 29 Analysis — Results Filing Template.md` — Phase 2 Boys rows | Boys max shift ≤ 5 pts (all age groups) | PENDING |
| 5 | ECNL dedup verification complete (FORGE) | FORGE task confirmation — ECNL dedup affects game records Phase 1 classifies | FORGE confirms ECNL dedup complete OR ELO confirms dedup is not needed for Phase 1 game set | PENDING |

**Condition 3 detail:** The stability monitoring spec defines three criteria (A, B, C). Condition 3 uses **Criterion A only** as the Phase 1 gate — rate of change < 3 pts/run for 3 consecutive runs. This is the most direct signal that the GA ASPIRE correction has been absorbed by the system. ELO confirms this by citing 3 consecutive briefing stability entries showing Criterion A met.

**Condition 5 rationale:** Phase 1 classifies ECNL events using current ratings. If ECNL game records contain duplicates (FORGE task: ECNL dedup), those duplicate games inflate team_count and distort `avg_team_rating` for ECNL events. FORGE's ECNL dedup task should complete before Phase 1 to avoid re-running Phase 1 after dedup cleans the data. If FORGE confirms dedup is complete, Condition 5 is met. If FORGE confirms dedup is not needed (no duplicates found), Condition 5 is met. If FORGE's dedup task is unresolved, ELO must flag this to SENTINEL before filing the clearance note.

---

## Section 3: Authorization Window

**Earliest possible date:** May 1, 2026 — assuming:
- April 28 Execution Log is filed April 28 (Condition 1: same-day)
- April 29 G2 gate passes (Condition 2: confirmed April 29)
- Stability Criterion A first met April 30 (allowance for post-fix adjustment per monitoring schedule)
- 3 consecutive stable runs: April 30, May 1, May 2 (earliest streak start)
- In practice: authorization most likely May 2 or May 5 depending on pipeline frequency

**Latest acceptable date:** May 7, 2026.

Working backward from May 9 (DSS readiness go/no-go):
- Phase 1 must run, ratings must recompute after Phase 1, and ELO must validate Phase 1 output
- Phase 1 runtime: 2–8 minutes (computation) + 30 minutes validation
- ELO validation note: file same day or next briefing
- Required clearance before May 9: Phase 1 authorized no later than **May 7** to allow May 8 validation and May 9 readiness review to include Phase 1 results

If Phase 1 is not authorized by May 7, ELO flags the Event Strength cell in the DSS Feature Readiness Tracker as ⚠️ and updates the DSS Demo Spec to reflect Phase 1 as at-risk for May 16.

**Dead window — do not run Phase 1 during:**
1. **While Boys Option A results are being analyzed** (window: April 29 – May 5). If Boys Option A returns anomalous results, Phase 1 introduces a second calibration variable before the Boys anomaly is resolved. ELO cannot attribute post-Phase-1 rating changes to Phase 1 vs. Boys calibration if both are active simultaneously. ELO monitors Boys results; if Boys Option A is clean by May 5, the dead window lifts.
2. **During any YELLOW or RED pipeline state** (as defined in Post-Fix Calibration Stability Monitoring Spec Levels 2–3). Phase 1 computes from current ratings; if current ratings are in an unstable state, Phase 1 strength scores are unreliable.
3. **Within 2 hours of a full recompute** that has not yet been confirmed stable. Allow at least one Query 1 monitoring check (< 5 minutes) after a recompute before running Phase 1.

---

## Section 4: ELO Phase 1 Clearance Note

When all Section 2 conditions are met, ELO files this block in the next briefing to SENTINEL.

```
Phase 1 Authorization Clearance — ELO
Date: ___________

Condition 1 — April 28 Execution Log ✅: confirmed [date]
Condition 2 — April 29 G2 gate PASS: confirmed [date]
Condition 3 — Stability Criterion A met for 3 consecutive runs:
  Run 1: [date] — max avg change: [N] pts
  Run 2: [date] — max avg change: [N] pts
  Run 3: [date] — max avg change: [N] pts
  All < 3 pts: YES
Condition 4 — Boys anomaly check PASS: Boys max shift = [N] pts (≤ 5 pts)
Condition 5 — ECNL dedup status: [complete / not needed — confirmed by FORGE on [date]]

Calibration stability confirmed as of: ___________
April 29 G2 gate: PASS (confirmed: ___________)

Expected Phase 1 effects:
  - Age groups affected: U9G–U17G, U9B–U17B (all age groups with ≥4 teams per event)
  - Direction: Event strength scores will increase for events dominated by GA ASPIRE teams
    (now using corrected higher ratings vs. prior suppressed ratings)
  - Magnitude: High-strength events may see avg_team_rating increase 5–20 pts for Girls
    GA ASPIRE-heavy events; minimal change for ECNL-dominant events
  - Boys events: minimal change (Boys ratings unchanged by GA ASPIRE fix)

No dead window conditions active: YES / NO
If dead window is active: ___________

SENTINEL authorization request:
Event Strength Phase 1 is cleared to run from [date] onward.
Authorization expires: [May 7 or DSS-1 week, whichever comes first].
```

ELO does not file this clearance note until all 5 conditions are met.

---

## Section 5: SENTINEL Authorization Block

*SENTINEL fills this section after receiving ELO's clearance note.*

```
Phase 1 Authorization Clearance reviewed: [ ]
ELO clearance note date: ___________
All 5 conditions confirmed ✅: [ ]

Event Strength Phase 1: AUTHORIZED / HELD
Authorization date: ___________
If HELD, reason: ___________
Authorized window: [start date] through [end date, no later than May 7]

Notes: ___________
SENTINEL sign-off: SENTINEL — [date]
```

---

## Section 6: Post-Phase-1 Validation Requirement

After Phase 1 runs, ELO files a post-Phase-1 validation note in the next briefing. Required within **24 hours** of Phase 1 execution. Minimum contents:

- Total event_strength rows inserted (from V1 validation query)
- Strength class distribution (from V2 — confirm ~25% High, ~42% Medium, ~33% Low)
- Any rows with null metrics (from V3 — expected: 0)
- High-strength U13G event confirmed (from V4 — name the event)
- Average `avg_team_rating` shift for Girls GA ASPIRE-dominated events vs. prior phase-1 (if this is a re-run): expected to be higher post-April-28 fix
- Any anomalous event_strength values: single event with `avg_team_rating` > 1800 or < 800 → investigate (data artifact or single-team event)
- **Anomaly threshold:** Any single event where `avg_team_rating` differs from the age group median by more than **300 points** is flagged. Expected: 0 flagged events. If > 5 flagged events → Phase 1 data quality issue; notify SENTINEL.

SENTINEL reviews this note before authorizing Phase 2 (if Phase 2 is scoped).

---

## Quick-Reference: Authorization Decision Tree

```
1. Is April 28 Execution Log ✅?
   NO → do not authorize. Wait for April 28.

2. Is April 29 G2 gate PASS?
   NO → do not authorize. G2 fail means calibration is not in expected state.

3. Have 3 consecutive pipeline runs shown Criterion A (avg change < 3 pts)?
   NO → wait. Earliest: May 2.

4. Is Boys anomaly check PASS (Boys shift ≤ 5 pts)?
   NO → hold. Boys anomaly must be resolved first.

5. Is ECNL dedup confirmed complete or not needed?
   NO → hold. Request FORGE confirmation.

All YES → ELO files clearance note → SENTINEL authorizes.
```

---

## References

- `Rankings/Event Strength Phase 1 — Full Execution Package.md` — Phase 1 scope and SQL
- `Rankings/Post-Fix Calibration Stability Monitoring Spec.md` — Criterion A definition (Condition 3)
- `Rankings/April 29 Analysis — Results Filing Template.md` — G2 gate (Condition 2) and Boys check (Condition 4)
- `Infrastructure/April 28 Execution Log.md` — Condition 1 source
- `Rankings/Boys Option A Spot Check — Presten Execution Package.md` — Boys anomaly context
- `Product & Planning/DSS Feature Readiness Tracker — May 2026.md` — Event Strength cell this authorization unlocks

*ELO — 2026-04-25*
