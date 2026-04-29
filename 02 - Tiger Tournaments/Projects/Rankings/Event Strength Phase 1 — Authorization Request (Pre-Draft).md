---
type: elo-authorization-request
topic: event-strength-phase1
date: 2026-04-29
status: pre-draft
sentinel_review_required: true
note: "Pre-draft written for condition where G0 = GO. When G0 is confirmed GO, ELO fills [TO FILL] cells and submits to SENTINEL."
tags: [event-strength, phase1, authorization, sentinel, elo, evo-draw]
---

# Event Strength Phase 1 — Authorization Request (Pre-Draft)

> **Pre-draft notice:** This document is written in advance of G0 confirmation. All static content is complete. Cells marked [TO FILL: trigger] require G0 = GO confirmation or session execution results before submission. When G0 is confirmed GO, ELO updates those cells and submits this document to SENTINEL.

---

## Authorization Request Header

```
Requesting:       Event Strength Phase 1 production activation
Submitted by:     ELO
Date submitted:   [TO FILL: date G0 confirmed GO]
SENTINEL reviewer: SENTINEL
Decision required by: May 7, 2026 (latest acceptable authorization date for May 9 DSS readiness)
```

---

## Section 1 — Phase 1 Scope Summary

Event Strength Phase 1 computes per-event quality ratings across all events in the database using current `rankings.rating` values as team-strength proxies, then writes those ratings to the `event_strength` table. Phase 1 covers both Girls and Boys events for all age groups with sufficient team count (≥ 4 teams per event per age group), spanning all event tiers after remediation of 6 unconfirmed circuits (USL Academy, EDP, Elite 64, NAL, State Cup, and remaining null-tier events assigned to `local`).

**What Phase 1 produces** (one row per event_id × age_group):
- `avg_team_rating` — mean of participating team ratings
- `strength_class` — High (top 25%), Medium (middle 42%), Low (bottom 33%), based on PERCENT_RANK within age_group × season
- `strength_percentile` — raw percentile position
- `competitive_spread` — standard deviation of team ratings within the event
- `upset_rate` — fraction of games where the lower-rated team won
- `team_count` — number of distinct teams in the event

**What Phase 1 does NOT change:** No K-factor, no calibration values, no `rankings.rating` update for any team. Phase 1 is a read from `rankings` + write to `event_strength`. It does not feed back into the Glicko-2 computation for any current-season game.

**Explicitly excluded from Phase 1 (deferred to Phase 2):** Historical ratings at game date (requires `rating_history` table); custom event-tier weighting in the strength algorithm; per-season strength trend analysis.

---

## Section 2 — Authorization Criteria

All 5 conditions must be confirmed before ELO submits this request to SENTINEL.

| Criterion | Required State | Actual State | Source |
|-----------|---------------|--------------|--------|
| G0 gate | GO | [TO FILL: date confirmed GO] | ELO G0 gate confirmation |
| April 28 Execution Log | All 3 steps ✅; recompute completed | [TO FILL: confirm from April 28 Execution Log] | `Infrastructure/April 28 Execution Log.md` — SENTINEL Disposition = AUTHORIZED |
| G1–G4 girls calibration gates | All PASS | [TO FILL: G1–G4 results from April 29 session] | G1–G4 gate documents, `Rankings/April 29 Analysis — Results Filing Template.md` |
| Phase 1 data coverage check | Meets coverage threshold | Pre-checkable: Event Strength League Tier Coverage Audit confirmed all major tiers covered as of 2026-04-24. 6 unconfirmed circuits assigned via remediation SQL. | `Rankings/Event Strength — League Tier Coverage Audit.md` |
| Phase 1 post-compute validation spec | Filed | YES — filed 2026-04-24 | `Rankings/Event Strength Post-Compute Validation — 2026-04-24.md` |
| Calibration stability (Criterion A) | Girls GA ASPIRE avg change < 3 pts/run for 3 consecutive runs | [TO FILL: 3 consecutive run dates and values] | `Rankings/Post-Fix Calibration Stability Monitoring Spec.md` — Criterion A |
| ECNL dedup verification | Complete or confirmed not needed by FORGE | [TO FILL: FORGE confirmation] | FORGE task confirmation |

**Pre-filled cells reasoning:**
- Data coverage check and post-compute validation spec are vault-confirmable today and are complete.
- All other cells require April 29 session execution or post-session calibration pipeline runs.

**No dead window conditions as of filing:**
- Boys Option A results have not yet arrived (dead window condition applies). Phase 1 must not run while Boys Option A is being analyzed (April 29 – May 5). Dead window lifts May 5 if Boys Option A is clean. This is the primary reason ELO cannot submit the authorization until Boys Option A is resolved.
- If Boys Option A returns anomalous results, Phase 1 stays on hold until Boys anomaly is resolved.

---

## Section 3 — Expected Impact Statement

When Event Strength Phase 1 goes live, the `event_strength` table will be populated for the first time (or repopulated if Phase 1 was previously run and truncated). No team rating in `rankings` changes. The primary visible effect is that the DSS Event Strength card becomes data-backed: the feature can now display per-event quality ratings, strength class, competitive spread, and upset rate for any event in the database.

**Expected magnitude of changes in rankings display:** For Girls GA ASPIRE-heavy events, `avg_team_rating` scores will be higher post-April-28 fix than a pre-fix Phase 1 run would have produced — GA ASPIRE team ratings were suppressed 20–40 points before the fix, which would have systematically underestimated the strength of those events. For ECNL-dominant events, minimal change is expected. For Boys events, no change is expected from the April 28 fix (Boys ratings are unchanged by the GA ASPIRE correction).

**Strength class distribution (expected):** Approximately 25% of events per age group per season will be classified High, 42% Medium, 33% Low. This distribution follows directly from the PERCENT_RANK cutoffs at the 75th and 33rd percentile and does not depend on the absolute rating scale.

**DSS demo readiness:** After Phase 1 runs, the V4 validation query (High-strength U13G event spot check) will confirm whether the DSS Event Strength demo card has a qualifying High-strength event with ≥ 8 teams — the minimum for DSS presentation. ELO files a post-Phase-1 validation note within 24 hours of execution.

---

## Section 4 — Rollback Plan

Phase 1 modifies only the `event_strength` table and (if Section 1 remediation ran) the `events.event_tier` column. Both are fully reversible without a ranking recompute.

**How to reverse:**
1. `TRUNCATE event_strength;` — clears all Phase 1 data in seconds. No recompute needed.
2. If `event_tier` remediation (Section 1) ran and caused problems, reverse with the documented per-circuit UPDATE statements (SET event_tier = NULL WHERE event_tier = '[assigned_value]' AND event_name ILIKE '%[circuit name]%'). The `local` fallback can be reset globally if needed.

**How quickly rollback completes:** TRUNCATE is immediate (< 5 seconds). The event_tier revert UPDATEs are < 1 minute.

**Who makes the rollback call:** ELO flags the anomaly to SENTINEL in the next briefing. SENTINEL authorizes rollback. Presten executes the TRUNCATE. In an emergency (clear data corruption in the rankings display), Presten may execute TRUNCATE without waiting for SENTINEL — but ELO must be notified immediately and must file a post-rollback note.

**What rollback does NOT fix:** If Phase 1 ran during a dead window (e.g., while Boys Option A was active), TRUNCATE removes the event_strength data but does not undo any calibration decisions that may have been made using unreliable Phase 1 output. Dead window violations must be documented and flagged to SENTINEL regardless of whether rollback is executed.

---

## Section 5 — Dead Window Status at Time of Submission

*ELO fills this section at time of actual submission (when G0 = GO and all [TO FILL] cells are complete).*

```
Dead window check at submission date [TO FILL: date]:

[ ] Boys Option A analysis period (April 29 – May 5): CLEAR / ACTIVE
    → If ACTIVE: do not submit. Phase 1 cannot run while Boys Option A is unresolved.
[ ] Pipeline YELLOW or RED state: CLEAR / ACTIVE
    → If ACTIVE: wait for GREEN before submitting.
[ ] Within 2 hours of an unconfirmed full recompute: CLEAR / ACTIVE
    → If ACTIVE: wait for post-recompute stability check.

All dead window conditions clear: YES / NO
```

---

## Section 6 — SENTINEL Authorization Block

*SENTINEL fills after receiving completed (non-pre-draft) submission from ELO.*

```
[ ] AUTHORIZED — Event Strength Phase 1 may proceed to production
[ ] DENIED — See conditions below:
[ ] CONDITIONAL — Proceed with the following constraints:

Authorization window (if authorized): _________________ through May 7, 2026
Dead window conditions at authorization: CLEAR

SENTINEL: _________________________ Date: ___________

Conditions or notes (if DENIED or CONDITIONAL):
___________________________________________________________
```

---

## References

- `Rankings/Event Strength Phase 1 — SENTINEL Authorization Criteria.md` — full condition definitions
- `Rankings/Event Strength Phase 1 — Full Execution Package.md` — Phase 1 scope and SQL
- `Rankings/Post-Fix Calibration Stability Monitoring Spec.md` — Criterion A definition
- `Rankings/April 29 Analysis — Results Filing Template.md` — G2 gate and Boys anomaly check source
- `Rankings/Event Strength — League Tier Coverage Audit.md` — data coverage (pre-filled)
- `Rankings/Event Strength Post-Compute Validation — 2026-04-24.md` — post-compute validation spec (pre-filled)

*ELO — 2026-04-29 (pre-draft)*
