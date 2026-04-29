---
type: elo-risk-register
topic: may9-dss-gate
date: 2026-04-29
gate_date: 2026-05-09
status: active
sentinel_review_due: 2026-05-08
tags: [elo, risk-register, may9, dss, sentinel-gate]
---

# May 9 DSS Gate — Risk Register

> **Purpose:** Forward-looking risk register for May 9 DSS go/no-go gate. SENTINEL uses this alongside the Pre-May-9 Calibration State Dashboard (filed May 8) to issue the May 9 verdict. Read alongside `Rankings/Pre-May-9 Calibration State Dashboard.md`.
>
> **Last updated:** 2026-04-29 | **Next update trigger:** Any risk probability change Low→High before May 8; otherwise updated May 8 alongside dashboard.

---

## Risk Register

| Risk ID | Risk Description | Probability | Impact | DSS Block? | Mitigation | Owner | Deadline |
|---------|-----------------|-------------|--------|-----------|------------|-------|----------|
| R1 | **Girls GA ASPIRE fix not applied before May 9** — April 28 session did not run; fix is deferred to April 29. If April 29 session also does not open, fix slips further and Girls U13/U14 rankings remain based on mis-labeled event tier data through demo day. | High (session still not run as of 04:29) | High (Girls rankings confidence claim is undermined at demo; GA ASPIRE mis-labeled data produces wrong win probabilities) | YES | April 29 session executes GA ASPIRE UPDATE as Step 1 (deferred from April 28); FORGE files Schema Confirmation within 30 min; ELO re-evaluates G0. If April 29 also fails: escalate to SENTINEL for emergency session scheduling before May 5. | Presten (session) / ELO (G0 evaluation) | April 29 session |
| R2 | **Boys Option A verdict: FAIL** — If Boys Option A spot-check (B-OA-1/2/3) returns data showing Boys GA ASPIRE is meaningfully miscalibrated at cal=100, and the fallback path (Option B, ~cal=70) cannot be analyzed and applied before May 9, boys rankings may be uncalibrated for demo. | Medium (Boys Option A data not yet available; failure probability conditional on query results) | Medium (Boys rankings still function; failure narrows the accuracy claims ELO can make at DSS) | NO (demo proceeds; accuracy claims are softened, not eliminated) | Boys Option A execution package is ready (`Rankings/Boys Option A — April 29 Execution Package.md`); Presten runs B-OA-1/2/3 in April 29 session; ELO analyzes within 48 hrs; if FAIL, Boys GA ASPIRE revision can be scoped for post-DSS only (not DSS-blocking per Boys GA ASPIRE Calibration Decision doc) | Presten (queries) / ELO (analysis) | May 5 verdict |
| R3 | **ECNL migration option decision made too late for implementation before May 9** — Option decision due April 30 EOD. If decision is delayed past May 2, FORGE may not complete implementation in time for ELO to run CP3–CP5 checkpoint validation before May 9. | Medium (decision brief filed April 29; awaiting Presten/SENTINEL authorization) | Medium (May 9 rankings are functional without ECNL migration complete; migration is June 1, not May 9; however, CP1/CP2 failures uncorrected would affect ongoing data quality) | NO (ECNL migration is June 1; May 9 gate is about current rankings accuracy, not migration completion) | ECNL Migration Option Decision Brief filed April 29 (this session); Presten authorizes by April 30 EOD; FORGE begins May 1. CP1/CP2 slip to April 30 is already planned and acceptable. | Presten (decision) / SENTINEL (authorization) | April 30 EOD |
| R4 | **Event Strength Phase 1 not authorized before May 9** — G0 = NO-GO as of April 29. If G0 is not re-confirmed GO before May 7, the Event Strength Phase 1 authorization window closes and Event Strength cannot be demonstrated at DSS on its current timeline. | High (G0 currently NO-GO; requires April 29 execution to re-evaluate) | Low (Event Strength Phase 1 is a feature enhancement, not a rankings accuracy blocker; demo can proceed without it) | NO (DSS can proceed without Event Strength Phase 1; it is a "nice to have" for the demo, not a calibration requirement) | April 29 session executes deferred April 28 steps; FORGE files Schema Confirmation; ELO re-evaluates G0. If G0 confirmed GO by May 1, ELO files Event Strength SENTINEL authorization request within 24 hrs. Hard deadline for authorization filing: May 7. | Presten (session) / FORGE (schema) / ELO (G0 eval) | May 7 (authorization filing) |
| R5 | **Brier score does not improve sufficiently after GA ASPIRE fix** — After the April 29 GA ASPIRE fix runs and rankings recompute, if Girls U13/U14 Brier does not improve by ≥ 0.010 from pre-fix baseline (0.312 / 0.273 blended), the DSS accuracy claim cannot be upgraded to the full authorized language. | Low (projections show ~0.015–0.025 improvement for Girls component alone; fix is directionally correct) | Low (softened accuracy language still authorized; demo is not blocked; impacts confidence of ranking accuracy claim only) | NO (DSS proceeds with softened Brier claim language; ELO has pre-authorized fallback language in Brier Score Completion doc) | Run post-fix Brier computation as part of April 29 analysis session; if improvement < 0.010, ELO files revised DSS claims language using softened authorized text; notifies SENTINEL same session. | ELO / Presten (Brier recompute) | April 30 (post-fix Brier run) |
| R6 | **Team merges audit unresolved for high-priority items at demo** — High-priority team merge errors (execution package ready; queries pending) could surface during DSS demo if a demo team has an incorrect merge history. | Low (merge audit queries are ready; DSS demo teams have been spot-checked in `Rankings/DSS Demo Teams — Merge Safety Verification.md`) | Medium (incorrect merge displayed live during demo damages trust; hard to explain in the moment) | NO (unlikely to block demo if demo-team spot-check is current; audit results are not required for May 9 gate) | DSS Demo Teams merge safety verification is filed; if new teams are added to demo script, run a spot-check before demo day. Full audit (May 7 session) is not May-9-gate-blocking. | ELO / Presten (demo team confirmation) | May 7 |
| R7 | **May 1 pipeline instability propagating to May 9 rankings** — The May 1 pipeline launch (FORGE confirmed; not blocked) is the first production run after the April 29 calibration changes. If the pipeline surfaces data anomalies in Week 1 (May 1–7), they could propagate into rankings shown at the May 9 gate. | Low (FORGE confirmed May 1 launch is not blocked; calibration changes are algorithmic, not pipeline-structural) | High (pipeline anomalies in the May 9 ranking set would require emergency investigation before demo; worst-case forces demo postponement) | YES (if anomalies affect the ranking set demonstrated at DSS and cannot be explained or resolved before May 14) | FORGE post-launch monitoring is documented (`Rankings/Post-Calibration Monitoring Spec — May 2026.md`); ELO reviews weekly health report May 8; any anomaly surfacing before May 8 triggers immediate ELO analysis. Pre-May-1 Girls calibration baseline snapshot is staged for comparison. | FORGE (pipeline) / ELO (anomaly detection) | May 8 review |
| R8 | **USARank comparison not complete by May 9** — USARank post-April-28 comparison execution package is filed but blocked on April 29 session execution. If April 29 session does not run, comparison slips to April 30–May 1. If it slips past May 5, ELO may not have sufficient time to analyze and file before May 9 gate. | Medium (blocked on April 29 session; if session runs April 29 the comparison can complete by May 1) | Medium (USARank comparison is ELO's primary external benchmark; without it, May 9 gate cannot confirm ranking accuracy vs. industry reference) | YES (SENTINEL's May 9 gate document requires USARank delta analysis to authorize the rankings accuracy claim for DSS) | April 29 session runs USARank comparison queries (G0-independent); ELO files analysis by April 30–May 1. If session slips to April 30, ELO files by May 2 — still within the May 8 dashboard window. Hard deadline: file by May 5. | Presten (queries) / ELO (analysis) | May 5 hard deadline |
| R9 | **Boys Brier pre-check not completed before May 17 deploy window** — Boys Brier execution package filed today (April 29) with a May 10–16 execution window. If Presten does not have a psql session available May 10–16, the U13/U14 parameter fix cannot deploy on May 17. | Low (window is 7 days; Presten typically has DB access) | Low (deploy slips to May 18–20; not DSS-blocking) | NO (U13/U14 fix is post-DSS; May 17 deploy is after the demo) | Boys Brier execution package is self-contained and filed; SENTINEL reviews package before May 10; Presten schedules session any day May 10–16. | Presten | May 16 |
| R10 | **ECNL CP1 fail — Option 1 disqualified with no approved fallback** — CP1 (ECNL RL staleness check) slipped to April 30. If CP1 fails (ECNL RL source > 30 days stale), Option 1 is disqualified and Option 2 becomes the recommended path — but Option 2 requires Presten sign-off on the rating cliff before execution. If both Option 1 is disqualified AND Presten cannot sign off on Option 2 cliff before May 10, ELO has no authorized migration path. | Low (ECNL RL staleness failure is possible but not the base case; data source is expected to be current) | High (no authorized migration path before June 1 is a material risk to the ECNL team entity migration timeline; does not block May 9 DSS but complicates June planning) | NO (ECNL migration is June 1; does not block May 9 gate) | CP1 Fail Escalation Protocol is filed (`Rankings/ECNL Migration — CP1 Fail Escalation Protocol.md`); if CP1 fails on April 30, ELO triggers escalation protocol same session, notifies SENTINEL and Presten within 2 hours, and files revised option recommendation within 24 hours. | ELO (escalation) / Presten (Option 2 sign-off if needed) | April 30 (CP1 run) |

---

## Risk Summary

There are **3 YES (DSS-blocking) risks:** R1 (Girls GA ASPIRE fix), R7 (May 1 pipeline instability), and R8 (USARank comparison). Of these: R1 and R8 both have direct mitigations that can execute in the April 29 session; R7 is low probability and FORGE has monitoring infrastructure in place.

**ELO's overall risk assessment: MEDIUM.** The April 28 session not running created a cascading deferral of 8 tasks, and the current state reflects those deferrals. If the April 29 session opens and executes successfully — GA ASPIRE fix applied, USARank comparison queries run, G0 re-evaluated — most of these risks will downgrade to LOW by April 30. If April 29 does not open, R1 and R8 escalate to HIGH probability and the May 9 gate becomes more uncertain.

**The single highest-priority mitigation Presten should focus on this week:** Open the April 29 DB session. Everything else follows from that. Step 1 (GA ASPIRE UPDATE) is the fix that directly unblocks R1, enables G0 re-evaluation (R4), and starts the rating recompute that makes USARank comparison meaningful (R8). One session resolves the three most consequential open items.

---

## SENTINEL Handoff Note

> ELO filed this risk register on 2026-04-29. It will be updated on May 8 alongside the Pre-May-9 Calibration State Dashboard. Any risk that moves from LOW to HIGH probability before May 8 will trigger an out-of-cycle SENTINEL notification via ELO Queue.

---

## References

- `Rankings/Pre-May-9 Calibration State Dashboard.md` — calibration state (updated April 29 @ 04:29)
- `Rankings/May 9 DSS Readiness Final Assessment Spec.md` — readiness assessment framework
- `Rankings/April 29 ELO Session Master Script.md` — session execution order
- `Rankings/Boys Option A — April 29 Execution Package.md` — Boys Option A queries
- `Rankings/ECNL Migration — CP1 Fail Escalation Protocol.md` — CP1 failure protocol

*ELO — 2026-04-29*
