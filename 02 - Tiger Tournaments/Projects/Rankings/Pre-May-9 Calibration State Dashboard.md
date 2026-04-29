---
type: elo-calibration-dashboard
date: 2026-05-08
filed_for: May 9 DSS Go/No-Go Gate
overall_verdict: CONDITIONAL — April 29 session not yet run as of 04:29
tags: [elo, calibration, dashboard, may9, dss, sentinel-gate]
related_task: task-2026-04-28-pre-may9-calibration-state-dashboard
last_updated: 2026-04-29-04:29
---

# Pre-May-9 Calibration State Dashboard

> **Pre-staged 2026-04-28. Updated 2026-04-29 04:29 to correct April 28 session assumptions (session did not run — G0 = NO-GO DEFERRED). ELO fills all status rows and verdicts by end of day May 8, 2026. SENTINEL reviews on May 9 alongside FORGE's Infrastructure Readiness Assessment.**

---

## Section 1 — Current Calibration Values (All Leagues)

*Static snapshot as of 2026-04-29. Sourced from [[Calibration Values — League Hierarchy Reconciliation]].*

### Boys Tiers

| League | Cal Value | Status | Last Changed | Notes |
|--------|-----------|--------|--------------|-------|
| MLS NEXT (unified, pre-split) | 160 | DEPLOYED | Pre-April 26 | Split to Homegrown/Academy deferred May 18–20 |
| MLS NEXT Homegrown (post-split) | 160 | PENDING | — | Not yet in engine; May 18–20 implementation window |
| MLS NEXT Academy (post-split) | 135 | PENDING | — | Not yet in engine; May 18–20 implementation window |
| ECNL Boys | 120 | DEPLOYED | Pre-April 26 | Validated — ECNL Boys baseline anchor |
| GA Boys | 100 | DEPLOYED | Pre-April 26 | Theoretically sound; not yet Brier-validated |
| GA ASPIRE Boys | 100 (via `ga` tag) | DEPLOYED — UNVALIDATED | Pre-April 26 | Both engine and hierarchy assume 100; unvalidated assumption; Brier analysis June 2026 |
| NPL | 55 | DEPLOYED | Pre-April 26 | Validated (ECNL RL beats NPL 55.1%) |
| DPL | 55 | DEPLOYED | Pre-April 26 | Validated peer tier to NPL |
| ECNL RL | 55 | DEPLOYED | Pre-April 26 | Validated; resolveTeamTier() cap enforced |
| Pre-ECNL | 25 | DEPLOYED | Pre-April 26 | Validated; resolveTeamTier() cap enforced |
| USL Academy | UNDEFINED | PENDING | — | Recommend 55; Presten review required before applying |
| Elite 64 | UNDEFINED | PENDING | — | Recommend 55; minimal cross-league data |
| EDP | UNDEFINED | PENDING | — | Recommend 55; regional scope |
| NAL | UNDEFINED | PENDING | — | Recommend 55; limited coverage |

### Girls Tiers

| League | Cal Value | Status | Last Changed | Notes |
|--------|-----------|--------|--------------|-------|
| GA Girls | 140 | DEPLOYED | Pre-April 26 | Validated Girls Tier 1 |
| GA ASPIRE Girls | **FIX PENDING** (140→100) | KNOWN DISCREPANCY | April 29 session | Engine uses 140 (via `ga` tag); correct value is 100; fix deferred from April 28 to April 29 session |
| ECNL Girls | 130 | DEPLOYED | Pre-April 26 | Validated Girls Tier 1 |
| NPL Girls | 55 | DEPLOYED | Pre-April 26 | Validated |
| DPL Girls | 55 | DEPLOYED | Pre-April 26 | GA pathway; peer to NPL |
| ECNL RL | 55 | DEPLOYED | Pre-April 26 | Validated; resolveTeamTier() cap enforced |
| Pre-ECNL | 25 | DEPLOYED | Pre-April 26 | Validated |
| USL Academy | UNDEFINED | PENDING | — | Recommend 55; Presten review required |

---

## Section 2 — Pending Calibration Changes (May 1–17 Window)

| Change | Target Date | Authorization Status | Risk if Missed |
|--------|------------|---------------------|----------------|
| Girls GA ASPIRE fix (cal 140→100) | April 29 session | NOT APPLIED — deferred from April 28 (session did not run) | Girls U13/U14 ratings inflated ~40 pts; DSS accuracy claim undermined at demo |
| Tier 2 undefined leagues (USL Academy, Elite 64, EDP, NAL → 55) | Pre-May 9 | PENDING Presten review — low urgency | Minimal if aggregate game volume < 1,000; immaterial to May 9 gate |
| MLS NEXT tier split (Homegrown=160, Academy=135) | May 18–20 | Spec filed and authorized; implementation deferred | Post-DSS; zero impact on May 9 gate |
| U13/U14 K-factor/RD fix | May 17 — DO NOT DEPLOY BEFORE | HOLD until Boys Brier pre-check passes (window: May 10–16) | May 17 deploy blocked if Brier check not run; not DSS-blocking |
| ECNL migration (CP1→CP5 checkpoints) | June 1 | Option decision due April 30 EOD — ECNL Decision Brief filed April 29 | June 1 entity migration risk if option decision delayed past May 2 |

---

## Section 3 — Known Risks as of April 29

*Condensed from `Rankings/May 9 DSS Gate — Risk Register.md`. Full detail in that document.*

| Risk ID | Risk | Probability | DSS Block? | Mitigation |
|---------|------|-------------|-----------|------------|
| R1 | Girls GA ASPIRE fix not applied before May 9 | High | YES | April 29 session executes GA ASPIRE UPDATE as Step 1; FORGE schema confirmation |
| R2 | Boys Option A verdict: FAIL | Medium | NO | B-OA-1/2/3 execution package ready; ELO analyzes within 48 hrs of results |
| R3 | ECNL migration decision delayed past May 2 | Medium | NO | Decision Brief filed April 29; Presten authorizes by April 30 EOD |
| R4 | Event Strength Phase 1 not authorized before May 9 | High | NO (nice-to-have) | April 29 session re-evaluates G0; authorization by May 7 if G0 = GO |
| R5 | Girls Brier does not improve after GA ASPIRE fix | Low | NO | Fallback claims language pre-authorized |
| R6 | Team merge error surfaces at live demo | Low | NO | DSS demo teams spot-checked |
| R7 | May 1 pipeline instability propagates to May 9 rankings | Low | YES (if anomalies affect demo set) | FORGE monitoring; ELO reviews May 8 |
| R8 | USARank comparison not complete by May 9 | Medium | YES | April 29 session runs queries; hard deadline May 5 |
| R9 | Boys Brier pre-check not done before May 17 deploy | Low | NO (post-DSS) | Self-contained execution package filed; window May 10–16 |
| R10 | ECNL CP1 fail — no authorized fallback path | Low | NO | CP1 Fail Escalation Protocol filed |

**Overall risk level: MEDIUM.** Three DSS-blocking risks (R1, R7, R8). All have mitigations that activate in the April 29 session. If April 29 session executes successfully, overall risk drops to LOW by April 30.

---

## Section 4 — Authorization Gate Status

*Static as of April 29. ELO updates on result receipt.*

| Gate | Status | Condition to Open |
|------|--------|------------------|
| G0 — Girls GA ASPIRE fix applied | NO-GO (deferred from April 28) | Presten runs GA ASPIRE UPDATE in psql; FORGE schema confirmation within 30 min |
| G1–G4 — Girls calibration sequence | HELD (blocked on G0) | After G0 = GO; ELO verifies each gate per criteria |
| Event Strength Phase 1 | BLOCKED | After G0 = GO + G2 PASS + SENTINEL authorization |
| Boys Option A verdict | AWAITING RESULTS | Presten runs B-OA-1/2/3; G0-independent; window open now |
| ECNL Migration Option | PENDING PRESTEN AUTH | April 30 EOD decision deadline; Decision Brief filed April 29 |
| Tier 2 Leagues calibration | PENDING PRESTEN REVIEW | Low urgency; Presten confirms game volume before applying |
| May 1 Pipeline Launch | AUTHORIZED | FORGE confirmed; not blocked by any calibration item |

---

## Section 5 — Data-Dependent Sections (ELO Fills May 8)

The following require May 1–8 pipeline data and cannot be pre-filled:

- **Calibration stability assessment (May 1–8 game data)** — (ELO fills May 8: stddev delta from first pipeline run vs. `Rankings/May 1 Pipeline Launch — ELO Ratings Baseline.md`)
- **Rating distribution shift from first pipeline run** — (ELO fills May 8: which age groups shifted, direction, magnitude)
- **Rank bands distribution check** — (ELO fills May 8: RB-V-1 through RB-V-4 results per `Rankings/Rank Bands — Post-Launch Validation Spec.md`)
- **Post-fix Brier re-run result** — (ELO fills after April 29 gate confirms fix landed: Girls Brier score post-GA ASPIRE fix; target < 0.24)
- **USARank comparison result** — (ELO fills by May 5: delta analysis post-April-28 per execution package)
- **Boys Option A verdict** — (ELO fills within 48 hrs of B-OA-1/2/3 results: APPROVE / CONDITIONAL / REJECT)

---

## Calibration Status Table

Six rows. ELO fills each row by May 8 with actual production-confirmed status.

| Item | Status | Evidence | Source |
|------|--------|----------|--------|
| Girls full calibration (production) | IN-PROGRESS | GA ASPIRE fix NOT YET APPLIED — April 28 session did not run (G0 = NO-GO DEFERRED); fix deferred to April 29 session; G1–G4 gate results pending | `Rankings/April 29 Gate Results — Structured Log.md` |
| Boys Option A verdict | IN-PROGRESS | Spot check window April 29–May 5; execution package filed 02:18; queries run when Presten opens DB session; decision document pending | `Rankings/Boys Option A — Decision Document.md` |
| Event Strength Phase 1 applied | IN-PROGRESS | Authorization criteria filed; blocked on G0 = GO re-confirmation (April 29 session) + SENTINEL authorization (target April 30) | `Rankings/Event Strength Phase 1 — SENTINEL Authorization Criteria.md` |
| Tier 2 undefined leagues calibration | IN-PROGRESS | Recommendation filed; Presten authorization pending; SQL execution not yet run | `Rankings/Tier 2 Undefined Leagues — Calibration Recommendation.md` |
| ECNL migration rating continuity | IN-PROGRESS | CP1 slipped from April 29 to April 30 (April 28 ecnl_verified backfill not run); CP2 follows CP1; option selection due May 10 | `Rankings/ECNL Migration — Option Selection Decision Gate.md` |
| Team merges audit (high-priority) | IN-PROGRESS | High-priority audit list filed; Presten execution package ready; queries not yet run; results due May 7 | `Rankings/Team Merges — High-Priority Audit List.md` |

> **ELO replaces IN-PROGRESS / PENDING rows with actual MET / NOT MET / DEFERRED statuses by May 8.**

**Status definitions:**
- MET: condition has been verified in production
- NOT MET: condition has not been met; flag if this blocks DSS
- DEFERRED: explicitly deferred with SENTINEL authorization; does not block DSS
- IN-PROGRESS: work is underway, expected completion before May 9

---

## DSS Blocking Assessment

> **ELO fills this section by May 8 for any NOT MET items.**

For each NOT MET item: state whether it blocks the DSS demo. Format:
> [Item]: NOT MET. DSS BLOCK: YES / NO. Reason: [1 sentence].

*As of April 29 04:29: April 28 session did not run. GA ASPIRE fix remains unapplied. All items remain IN-PROGRESS. No NOT MET determinations yet — earliest determination possible after April 29 session runs. ELO fills this section on May 8 with actual outcomes.*

---

## Brier Score Status

> **ELO fills this section by May 8 after running the post-GA-ASPIRE Brier re-run.**

Current Girls Brier score (pre-April-28): 0.282 (U13: 0.312, U14: 0.273, per Calibration Validation 2026-04-22).
Post-April-28 Brier: NOT YET RUN — run required after April 29 gate results confirm fix landed correctly.
Target: < 0.24 (DSS authorized threshold per DSS SENTINEL Authorization Document).
Status: ABOVE THRESHOLD (pre-fix) — post-fix re-run will establish updated value.

*ELO updates this with actual post-fix Brier score when re-run completes.*

---

## USARank Comparison Readiness

Has ELO completed the post-April-28 USARank comparison? NO — April 29 session has not yet opened as of 04:29.
Expected completion: April 30–May 1 after April 29 session recompute + rating stabilization.
Reference: `Rankings/USA Rank Post-April-28 Comparison — Execution Package.md`

*ELO updates this with YES / confidence level when comparison is complete.*

---

## ELO Overall Verdict

> **ELO fills final verdict by May 8.**

Updated assessment (April 29 04:29): CONDITIONAL — all six calibration items remain IN-PROGRESS. April 28 session did not run (G0 = NO-GO DEFERRED), pushing Girls calibration, ECNL CP1, and Event Strength Phase 1 triggers all rightward by ~1 day. No DSS-blocking NOT MET items identified — but this assessment cannot be confirmed until April 29 session runs. If April 29 session executes today, critical path items (GA ASPIRE fix, Boys Option A queries) resolve April 29–30 and the May 9 timeline remains intact.

One of:
- **READY** — All items MET or DEFERRED (with authorization). No DSS-blocking NOT MET items.
- **CONDITIONAL** — One or more IN-PROGRESS items expected to complete by May 9. State specific conditions.
- **NOT READY** — One or more DSS-blocking NOT MET items. Immediate SENTINEL notification filed.

---

## SENTINEL Authorization Request

> **ELO fills this section on May 8 when filing the completed dashboard.**

> ELO certifies calibration readiness for the May 9 DSS go/no-go review. [Summary of key statuses.] Overall verdict: [READY / CONDITIONAL / NOT READY]. ELO requests SENTINEL include this assessment in the May 9 gate decision.

---

## Dependency Tracker

| Dependency | Expected By | Status |
|------------|-------------|--------|
| Girls Club Rankings first run | May 2 | PENDING |
| Boys Option A verdict | May 7 (48h after May 5 results) | PENDING — window opens April 29 |
| Event Strength Phase 1 authorization + execution | Early May | PENDING |
| Tier 2 calibration SQL execution | Before May 9 | PENDING |
| ECNL migration implementation (FORGE) | Post-April-30 option decision | PENDING |
| Post-April-28 USARank comparison | April 29–30 | PENDING |
| Post-GA-ASPIRE Brier score re-run | After April 29 gate confirmation | PENDING |

---

## References

- `task-2026-04-28-pre-may9-calibration-state-dashboard.md` — source task (due May 8)
- `Rankings/Boys Option A — Decision Document.md`
- `Rankings/Event Strength Phase 1 — SENTINEL Authorization Criteria.md`
- `Rankings/ECNL Migration — Option Selection Decision Gate.md`
- `Rankings/Team Merges — High-Priority Audit List.md`
- `Rankings/Tier 2 Undefined Leagues — Calibration Recommendation.md`
- `Rankings/April 29 Gate Results — Structured Log.md`

---

## Update Log

| Date | Updated by | Changes |
|------|-----------|---------|
| 2026-04-28 | ELO | Template pre-staged. All rows set to IN-PROGRESS with April 28 session assumptions. |
| 2026-04-29 04:29 | ELO | Corrected Girls calibration row (GA ASPIRE fix NOT applied April 28 — session did not run); corrected ECNL row (CP1 slipped to April 30); corrected USARank row (session not yet open); updated overall verdict to reflect actual state. |

*ELO — pre-staged 2026-04-28. Updated 2026-04-29 04:29. Final filing due 2026-05-08 EOD.*
