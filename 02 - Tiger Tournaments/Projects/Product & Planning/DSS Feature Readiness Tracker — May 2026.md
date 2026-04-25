---
title: DSS Feature Readiness Tracker — May 2026
type: readiness-tracker
author: ELO
date: 2026-04-25
status: active
next_review: 2026-04-29
related: "[[DSS Demo Spec — May 2026]]", "[[DSS Data Readiness Validation Plan — May 14-15]]"
tags: [dss, readiness, tracker, rankings, evo-draw]
---

# DSS Feature Readiness Tracker — May 2026

> **Purpose:** Single source of truth for DSS demo readiness. ELO owns Data and Calibration columns. SENTINEL updates Implementation Status as sessions complete. Last updated: 2026-04-25.
>
> **Review schedule:** April 29 (post-April-28 fix), May 9 (final go/no-go), May 14 (pre-demo check).

---

## Section 1: Feature Readiness Matrix

| Feature | Data Ready? | Calibration Ready? | Implementation Status | Demo Risk | Gate Requirement |
|---------|-------------|--------------------|-----------------------|-----------|-----------------|
| **Rank Bands** | ⚠️ Conditional | ⚠️ Pending | Pending 5.5h session | Low | April 28 GA ASPIRE fix + recompute must run first; re-validate thresholds before session |
| **Event Strength Phase 1** | ⚠️ Conditional | ⚠️ Pending | Pending 7h session | Medium | April 28 GA ASPIRE fix; Gate A (ASPIRE tagged correctly) + Gate B (null event_tiers resolved) must pass |
| **Club Rankings** | ⚠️ Conditional | ⚠️ Pending (Girls) / ⚠️ Medium (Boys) | Pending 10–11h session | Medium | April 28 GA ASPIRE fix is material dependency; Boys Option A validation required in pre-flight |
| **USA Rank Comparison** | ⚠️ Conditional | ✅ Ready | Pending FORGE SQL (~30 min) | Low | Presten psql access; delta interpretation spec complete as of 2026-04-24 |
| **USYS/GotSport Coverage** | ❌ Not ready | ✅ Ready | Pending Stage 1 crawl + org ID config | High | TX org ID browser lookup + test crawl must complete; no org IDs confirmed as of 2026-04-24 |

---

### Feature Detail

#### Rank Bands

**Data Ready: ⚠️ Conditional — ready after April 28 recompute.**

Threshold validation doc (`Rank Bands Threshold Validation — 2026-04-24.md`) is complete. Planned thresholds: Gold ≥1500, Silver 1350–1499, Bronze 1200–1349. Pass/fail criteria defined. Football Academy NJ expected Gold — the key demo anchor.

The condition: GA ASPIRE events are currently tagged at cal=140 (should be 100). After the April 28 UPDATE + recompute, Girls ratings in U13G–U17G shift −30–50 points for ASPIRE-heavy teams. The threshold validation query must be re-run against post-fix ratings to confirm Gold/Silver/Bronze bands are correctly positioned. If run before April 28, the threshold validation is based on inflated ratings.

**Calibration Ready: ⚠️ Pending — after April 28.**

After April 28 fix: Girls calibration is stable for demo window. U12/U13/U14/U19 calibration changes (K-factor/RD) deploy May 18–20 — after DSS, after Rank Bands session. This gap is acceptable: the K-factor/RD changes are precision improvements, not corrections to systematic miscalibration.

---

#### Event Strength Phase 1

**Data Ready: ⚠️ Conditional — after April 28; Gate A + Gate B pass required.**

Gate A: `SELECT COUNT(*) FROM events WHERE event_name ILIKE '%ASPIRE%' AND event_tier = 'ga'` must return 0. If GA ASPIRE events are still tagged as `ga`, Event Strength ratings for those events will be computed against overcalibrated ratings — producing incorrect `avg_team_rating` and therefore incorrect `strength_class` assignments for ASPIRE-tier events.

Gate B: Null event_tier count must reach 0. Section 1 remediation SQL (6 updates for USL Academy, EDP, Elite 64, NAL, State, local) runs if Gate B > 0. The 6 tier assignments are structural estimates acceptable for Event Strength Phase 1 (which uses percentile-based classification, not calibration values directly).

**Calibration Ready: ⚠️ Pending — after April 28.**

Event Strength classification uses `avg_team_rating` percentiles within (age_group, season). The 6 unconfirmed tier assignments (USL Academy cal=55 est., EDP cal=55 est., etc.) affect only the cross-tab validation in V2, not the core `strength_class` output. Core output is calibration-independent once team ratings are correct post-April-28.

---

#### Club Rankings

**Data Ready: ⚠️ Conditional — Girls ready after April 28; Boys medium risk.**

GA ASPIRE fix is a **material dependency**. A club with 4 of 7 qualifying age groups in GA ASPIRE events could see club rating drop ~17 points when fix applies. If computed before fix: top-20 Girls club could drop 3–8 positions post-fix — visibly wrong at DSS. Must compute after April 29 stabilization.

Coverage estimate from design: 70–80% of teams mappable to a club via GotSport org_id. The implementation plan pre-flight checklist includes a coverage query. If coverage < 60%, flag immediately — bootstrapping strategy changes.

Boys data: ❌ unvalidated calibration. Boys GA (cal=100) and Boys ECNL (cal=120) are theoretically sound but not Brier-validated. Boys club rankings should only be shown at DSS if Boys Option A pre-flight validation passes (3 queries, ~15 min). If Boys avg_rating diverges >100 points from Girls avg_rating for U13, or Boys GA game volume < 500, default to Girls-only club rankings for the demo.

**Calibration Ready: ⚠️ Pending (Girls) — after April 28; ⚠️ Medium (Boys) — unvalidated.**

Girls: stable post-April-28. U12/U13/U14/U19 calibration changes (May 18–20) are post-DSS. A post-DSS ranking update of 15–35 points for top clubs is directionally positive (more accurate). Accept this gap.

Boys: Option A (cal=100) is conservative. Not wrong, but unknown. Boys Brier analysis is scheduled post-DSS (May 17–30). Do not make a Boys calibration accuracy claim at DSS.

---

#### USA Rank Comparison

**Data Ready: ⚠️ Conditional — pending FORGE SQL run.**

FORGE wrote 16 audit queries (`USA Rank Ranking Accuracy Audit — April 2026.md`). Task `task-2026-04-24-run-accuracy-audit-sql.md` is blocked on Presten psql access. Once run, results file at `Reports/usarank-accuracy-audit-results-[date].md`.

**Calibration Ready: ✅ Ready.**

Delta interpretation spec complete (`USA Rank Delta Interpretation Spec — April 2026.md`). Thresholds: Expected <35 pts, Mild discrepancy 36–60 pts, Concerning 61–100 pts, Critical >100 pts. ELO will apply spec to classify each delta within one session of receiving results.

Methodology: USA Rank is NOT ground truth. Evo Draw's advantage is depth of state league coverage, Brier-validated cross-league calibration, and full game history. Where methodologies diverge, we have a specific defensible reason: goal differential exclusion (±15–30 pts), coverage gaps (±50–80 pts for severe cases), age group mapping differences (±10–20 pts).

---

#### USYS/GotSport Coverage Expansion

**Data Ready: ❌ Not ready.**

No GotSport org IDs confirmed for state associations as of 2026-04-24. Estimated 20,000–35,000 state cup/league games missing from rankings. The coverage claim ("we rank more teams than USA Rank because we capture state league data they miss") requires at least TX org ID confirmed and ingested.

USL Academy (90 clubs), EDP (~150+ clubs), and NAL also lack confirmed org IDs. These are all Priority 2 gaps per Source Gap Inventory.

Resolution path: Presten browser session at `system.gotsport.com` to look up TX USYS org ID → add to crawl config → TX test crawl → confirm game count. Estimated: 2–3 hours total. If TX test crawl fails, coverage claim is not demo-ready.

**Calibration Ready: ✅ Ready.**

State league teams use existing calibration values once ingested. No new calibration work required.

---

## Section 2: Critical Path to May 9

May 9 is the final prep cutoff (one week before DSS). Items below are ordered chronologically. Only items with downstream dependencies are listed.

| Action | Owner | Est. Time | Gate For | Must Complete By |
|--------|-------|-----------|----------|-----------------|
| Pre-fix baseline snapshot | Presten | 15 min | Post-fix comparison (Rank Bands, Club Rankings) | April 27 |
| GA ASPIRE UPDATE + full ranking recompute | Presten | 2–3h | Rank Bands, Event Strength Phase 1, Club Rankings | April 28 |
| Post-fix verification (Event Strength Gate A) | Presten | 15 min | Event Strength Phase 1, Club Rankings | April 29 |
| Rank Bands threshold re-validation (Step 2 + Step 6 queries) | Presten | 15 min | Rank Bands 5.5h session | April 29 |
| TX USYS org ID browser lookup (system.gotsport.com) | Presten | 30 min | USYS coverage claim | April 29 |
| Run USA Rank audit SQL (FORGE's 16 queries in psql) | Presten | 30 min | USA Rank comparison, ELO delta review | April 30 |
| ELO delta review of USA Rank audit results | ELO | 1 session | DSS accuracy claim finalization | May 2 |
| TX test crawl (confirm game ingestion) | Presten/FORGE | 1–2h | USYS coverage claim | May 1 |
| Event Strength Phase 1 session | Presten | 7h | Event Strength demo card | May 2–3 |
| Club Rankings implementation session | Presten | 10–11h | Club Rankings page | May 2–5 |
| Rank Bands implementation session | Presten | 5.5h | Rank Bands demo | May 5–7 |
| May 9 go/no-go readiness check | ELO + Presten | 30 min | Demo finalization | May 9 |

---

## Section 3: Fallback Coverage

The fallback demo (from DSS Demo Spec Section 5) requires only: Rank Bands + team detail + USA Rank comparison.

**Event Strength Phase 1:** Required for primary demo only. If implementation session does not complete by May 9, the Event Strength card drops from the demo entirely. Fallback demo does not include it.

**Club Rankings:** Required for primary demo only. If implementation session does not complete by May 9, the Club Rankings page drops from the demo entirely. Fallback demo does not include it.

**USYS/GotSport Coverage:** Required for coverage depth claim only. If TX test crawl does not complete or does not confirm material new game ingestion by May 9, the state league coverage claim is dropped entirely. Do not improvise this number live — if unverified, remove from script.

**USA Rank Comparison:** Required for both primary and fallback demos. If FORGE's 16 queries are not run by May 2, the fallback demo loses its accuracy anchor. This is the only non-negotiable item.

**Boys Club Rankings:** Required only if Boys Option A pre-flight passes. If it does not, Girls-only club rankings are shown. This is a fallback within Club Rankings, not a fallback to a different demo.

---

## Section 4: SENTINEL Review Schedule

- **April 29:** ELO updates Data and Calibration columns for all features affected by April 28 fix. If any ⚠️ becomes ❌ post-fix (e.g., GA ASPIRE fix caused unexpected rating distribution shift), escalate to SENTINEL immediately with specific observation.

- **May 9:** ELO does final readiness review. Any ❌ at this date triggers fallback for that feature. ELO files updated tracker in briefing with summary: "DSS Readiness — Data ready: [N/5]. Calibration ready: [N/5]. Features go for primary demo: [list]. Features falling to fallback: [list]."

- **May 14:** ELO does pre-demo check using the DSS Data Readiness Validation Plan queries (Sections 1–4). Flags any last-minute issues to SENTINEL with action recommendation by 11 AM. Decision cutoff: noon May 14 — fixes after noon go into the demo or are cut, no exceptions.

---

## Related

- [[DSS Demo Spec — May 2026]] — primary demo flow and named assets
- [[DSS Data Readiness Validation Plan — May 14-15]] — execution checklist for May 14
- [[Rank Bands Threshold Validation — 2026-04-24]] — threshold validation queries
- [[Event Strength Phase 1 — Full Execution Package]] — Gate A/B conditions
- [[Club Rankings — Calibration Pre-Assessment]] — GA ASPIRE dependency analysis
- [[USA Rank Delta Interpretation Spec — April 2026]] — delta classification thresholds
- [[Source Gap Inventory — April 2026]] — USYS coverage gap details

*ELO — 2026-04-25*
