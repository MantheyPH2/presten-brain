---
title: Recent Changes 2024–2026
type: change-log
author: ELO
created: 2026-04-29
updated: 2026-04-29
tags: [rankings, calibration, changelog, evo-draw]
---

# Recent Changes 2024–2026

This log captures significant calibration, algorithm, and league classification changes to the Evo Draw ranking engine. Listed in reverse chronological order. Each entry includes the decision date, effective date (when implemented in production), and the responsible owner.

---

## 2026

### April 28, 2026 — GA ASPIRE Tier Classification Fix (Girls)

**Decision date:** 2026-04-23
**Implementation date:** April 28, 2026 (pending session confirmation as of April 29)
**Owner:** ELO / FORGE
**Scope:** Girls, all age groups

**Change:** GA ASPIRE events (launched February 2025) were misclassified as `ga` tier (cal=140). The correct tier is `ga_aspire` (cal=100). An UPDATE reclassifies all events with "ASPIRE" in the event name and a start date ≥ 2025-02-01 from `event_tier = 'ga'` to `event_tier = 'ga_aspire'`.

**Impact:** 350–1,050 Girls teams overcalibrated by ~40 points. Fix restores correct calibration. Boys teams unaffected (Boys GA and Boys GA ASPIRE both use cal=100).

**Verification:** G1–G4 gate checks (April 29 session). See `Rankings/April 29 Gate Results — Structured Log.md`.

---

### April 24, 2026 — Boys GA ASPIRE Calibration Resolved (Option A)

**Decision date:** 2026-04-24
**Implementation date:** N/A (no code change required)
**Owner:** ELO
**Scope:** Boys, all age groups

**Change:** Boys GA ASPIRE calibration = 100 (same as Boys GA). No Boys-specific modification to the April 28 UPDATE or to `compute-rankings.js` is needed. When the gender-agnostic UPDATE reclassifies Boys GA ASPIRE events from `ga` to `ga_aspire`, the calibration lookup returns 100 in both cases — net rating impact is zero.

**Deferred:** Full Boys GA ASPIRE empirical validation (Option B, potential ~70 value) deferred to post-DSS Brier analysis, June 2026.

**Reference:** `Rankings/Boys GA ASPIRE Calibration Decision — April 2026.md`

---

### April 2026 — MLS NEXT Tier Split Decision (Deferred to May 18–20)

**Decision date:** 2026-04-28
**Implementation date:** May 18–20, 2026 (planned; post-DSS window)
**Owner:** ELO / FORGE
**Scope:** Boys, all age groups

**Change:** MLS NEXT Boys teams will be split into two calibration tiers:

| Tier | Calibration Value | Notes |
|------|-------------------|-------|
| MLS NEXT Homegrown | 160 | Unchanged from current unified value |
| MLS NEXT Academy | 135 | −25 from current; validated by cross-league win-rate data |
| MLS NEXT Unclassified (ambiguous) | 147 | Interpolated midpoint for teams unidentifiable by event-name pattern |

**Basis:** Cross-league analysis shows Homegrown teams win 71.4% vs. ECNL; Academy teams win 58.2% vs. ECNL; Homegrown vs. Academy win rate is 66.8%. 4,000+ game sample confirms the 25-point calibration gap. Event-name pattern classifier identifies ~82% of teams; remaining ~18% receive interpolated value.

**Why deferred:** DSS demo window (May 9–16) prohibits calibration changes that could shift rankings. May 18–20 is the first post-DSS safe window. Concurrent Boys calibration evaluation (Option A, April 29) must also complete before MLS NEXT tier split to ensure stable baseline.

**Implementation readiness:** Classifier SQL ready (`Rankings/MLS NEXT Event Classifier — Queries 2026-04-24.md`). Calibration application spec ready (`Rankings/MLS NEXT Tier Split — Calibration Application Spec.md`). Code change to `resolveTeamTier()` in `compute-rankings.js` documented.

**Reference:** `Rankings/MLS NEXT Boys Tier Split — April 28 Decision.md`, `Rankings/MLS NEXT Tier Split Analysis 2026-04-22.md`

---

### April 2026 — U13/U14 K-factor and RD Fix (Staged, Pending May 17)

**Decision date:** 2026-04-23
**Implementation date:** May 17, 2026 (DO NOT DEPLOY BEFORE)
**Owner:** ELO
**Scope:** Boys and Girls, U13 and U14

**Change:** Calibration parameter fixes for U13 and U14 age groups. Brier validation identified severe miscalibration: U13 Brier=0.312, U14 Brier=0.273 (threshold: ≤0.23). Root causes and parameter corrections are documented in pre-deploy checklist.

**Why deferred:** Changes affect 6,310+ teams (>25-point shift). Deploying before DSS risks disrupting seeding. May 17 is the first post-DSS safe window.

**Reference:** `task-2026-04-23-u13-u14-calibration-fix.md`, `Rankings/calibration-fix-pre-deploy-2026-04-23.md`

---

## 2025

### 2025 — GA ASPIRE League Launched (February 2025)

Girls Academy (GA) introduced the ASPIRE tier as a developmental subset below GA-level competition. Events launched February 2025. The tier was not initially classified separately in the ranking engine — events were tagged as `ga` (cal=140) instead of the correct `ga_aspire` (cal=100). This was identified in the April 2026 GA coverage audit and corrected April 28, 2026.

---

## Related Notes

- [[Ranking Engine]] — full algorithm documentation (v7)
- [[League Hierarchy]] — current calibration values by tier
- [[Division Calibration]] — Phase 9 division placement calibration
- [[Age Groups and Birth Years]] — age group format and transition schedule
