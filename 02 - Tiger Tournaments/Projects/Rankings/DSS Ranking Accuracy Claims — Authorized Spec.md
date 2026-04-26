---
title: DSS Ranking Accuracy Claims — Authorized Spec
type: authorized-claims
author: ELO
date: 2026-04-25
status: ready — SENTINEL review required before DSS
related: "[[USA Rank Ranking Accuracy Audit — April 2026]]", "[[Boys Calibration Status Summary — April 2026]]", "[[Boys Calibration Option A — Interpretation Guide]]", "[[DSS Demo Spec — May 2026]]"
tags: [dss, accuracy, claims, girls, boys, calibration, evo-draw]
---

# DSS Ranking Accuracy Claims — Authorized Spec

> **Purpose:** ELO's signed authorization of specific accuracy claims Presten may use at DSS on May 16. Every claim in Sections 1–4 is backed by cited evidence. Section 5 lists claims that are NOT authorized. SENTINEL must review before DSS.

---

## Section 1: Authorized Claims — Girls Rankings

### Claim G1: Methodology Alignment with USA Rank (Top-Tier Teams)

> **Claim text:** "For elite U13 and U14 Girls teams — the top 100 teams per age group — Evo Draw's rankings broadly agree with USA Rank. ECNL-affiliated teams like Football Academy NJ and Maryland Rush Azzurri appear in the top positions on both systems."

**Evidence:** USA Rank Ranking Accuracy Audit — April 2026, Section 3 — Predicted Agreement Zones. Zone A predicts "Strong alignment expected" for USA Rank #1–100, based on shared ECNL source coverage. The audit SQL is ready and pending DB execution by Presten or FORGE.

**Confidence:** Medium. The alignment prediction is structural (both systems see ECNL games via GotSport). Actual confirmation requires running the audit SQL. If the Football Academy NJ merge is confirmed by May 14, confidence upgrades to High.

**Authorized for DSS:** ⚠️ With caveat — confirm alignment by running audit SQL before May 14. If Football Academy NJ appears as two split records in Evo Draw, do not claim top-tier alignment for U13G until merge is resolved.

---

### Claim G2: Deeper Coverage than USA Rank

> **Claim text:** "We actively rank more teams than USA Rank in every age group. In U13 Girls alone, we rank [X] active teams — teams with games in the last 7 months. USA Rank's visible list covers fewer because they exclude teams without sufficient recent data on their platform."

**Evidence:** USA Rank Ranking Accuracy Audit — April 2026, Section 4 (Missing Team Investigation). USA Rank's #500–2000 range includes teams that likely have minimal GotSport presence — the inverse is also true. Evo Draw's coverage of ECNL and GotSport-sourced events is comprehensive. The active team count query (games in last 7 months) is in `DSS Data Readiness Validation Plan — May 14-15.md`, Step 4A.

**Confidence:** Medium. The coverage claim directionally holds (we rank all GotSport-sourced teams regardless of recent-game minimums for USA Rank's algorithm). The specific count requires running Step 4A on May 14. Do not use a placeholder number — fill in the actual count before the demo.

**Authorized for DSS:** ⚠️ With caveat — run Step 4A before May 14 and replace [X] with the actual active team count. Do not state this claim with a placeholder number at DSS.

---

### Claim G3: Full Game History (Not Last-20 Recency Window)

> **Claim text:** "Our rankings use a team's full game history, not just the last 20 games. A team that has been consistently elite for two seasons — not just hot the last month — is reflected in our ratings. That's a more complete picture of a program's quality."

**Evidence:** USA Rank Ranking Accuracy Audit — April 2026, Section 5 (Methodology Divergence Assessment) — "USA Rank's Last-20-Games Recency Window." Evo Draw uses Glicko-2 with a recency decay weighting — all games contribute, with recent games weighted higher, but no hard cutoff at 20 games.

**Confidence:** High. This is a design fact, not a DB-dependent claim.

**Authorized for DSS:** ✅ Yes. This claim requires no additional data validation.

---

### Claim G4: Calibration Corrected Post-April 28 (Girls GA ASPIRE)

> **Claim text:** "We identified and corrected an overcalibration issue in our Girls rankings that was suppressing 350–1,000 Girls teams by 20–40 rating points. Our rankings are more accurate after that fix."

**Evidence:** Post-April-28 Expected Rating Shift — Pre-Run Model (ELO, 2026-04-25). GA Coverage Audit Results — 2026-04-23. The fix corrects `event_tier` for Girls GA ASPIRE events from `ga` (cal=140) to `ga_aspire` (cal=100).

**Confidence:** High for the claim's direction and mechanism. Medium for the specific magnitude (350–1,000 teams) — confirm via April 29 analysis.

**Authorized for DSS:** ⚠️ Conditional on April 29 analysis confirming the fix ran correctly (Gate G1 pass). If G1 fails, do not mention this claim at DSS.

---

## Section 2: Authorized Claims — Boys Rankings

### Pre-Option-A Claims (available regardless of Boys spot check outcome)

These claims are backed by structural design and cross-league data — not dependent on Option A results:

**Claim B1:**

> **Claim text:** "Boys MLS NEXT rankings are our most rigorously validated — we use ~5,800 cross-league games to validate the calibration values for MLS NEXT Homegrown and Academy tiers."

**Evidence:** Boys Calibration Status Summary — April 2026, Section 1 (MLS NEXT Boys) — "empirically validated by two independent methods from ~5,800 games." League Hierarchy — MLS NEXT Homegrown cal=160, Academy cal=135.

**Confidence:** High.

**Authorized for DSS:** ✅ Yes.

---

**Claim B2:**

> **Claim text:** "Boys rankings use the same Glicko-2 engine as Girls — full game history, calibrated by league tier. MLS NEXT and ECNL Boys teams are the best-validated cohorts."

**Evidence:** Boys Calibration Status Summary — April 2026, Section 4 (DSS Risk Assessment) — "Structural calibration design for Boys is sound."

**Confidence:** High for the structural claim. Do not extend to "Boys rankings are comprehensively validated" — the Brier analysis has not been run for Boys separately.

**Authorized for DSS:** ✅ Yes, as stated above.

---

### Option-A-Dependent Claims (authorize only if Boys Option A PASSES)

**Claim B3 (Option-A dependent):**

> **Claim text:** "Boys U13 and U14 rankings are spot-checked and consistent — Boys and Girls ratings align within expected ranges for each age group, and Boys GA and MLS NEXT game volumes are sufficient to produce stable rankings."

**Evidence:** Boys Option A Spot Check — Presten Execution Package (ELO, 2026-04-25). Requires all three queries to PASS by May 9.

**Confidence:** Conditional. Cannot authorize until Boys Option A results are in hand.

**Authorized for DSS:** ⚠️ Conditional — authorized only if Boys Option A returns PASS on all three queries. If any query FAILs, remove this claim from the demo script and replace Boys club rankings with Girls-only per the fallback spec.

---

## Section 3: Predictive Accuracy Claim

**Status: SOFTENED CLAIM AUTHORIZED. Full specific-Brier-score claim remains pending post-fix computation.**

The Brier Score Completion document (`Rankings/Brier Score Completion.md`) was filed April 25, 2026. See that document for full pre-fix Brier values and post-fix projections.

What is known from existing analysis:
- U13 blended Brier: 0.312 (current, pre-calibration fix) — above acceptable threshold of ≤0.23
- U14 blended Brier: 0.273 — above threshold
- Post-calibration-fix target: U13 ≤0.24, U14 ≤0.24 (per Calibration Validation 2026-04-22)
- These are blended Boys+Girls values; gender-separated Brier not yet run

**Authorized predictive accuracy claim for DSS:**

> **Claim text:** "Our ranking engine generates a probability prediction for every game. We track prediction accuracy, and we're actively improving it — our U13 and U14 calibration changes are specifically designed to improve predictive accuracy by bringing Brier scores below our internal threshold."

This claim is authorized because: (a) the pipeline does produce predictions, (b) Brier scores have been computed, (c) the calibration work is aimed at improving them. It avoids quoting a specific number ELO cannot currently stand behind.

**What is NOT authorized:** Quoting the current Brier score (0.312, 0.273) as a positive accuracy metric at DSS. These are pre-fix values above threshold — do not present them as evidence of accuracy.

**Authorized for DSS:** ✅ Yes — use the softened claim text above only. Do not quote raw Brier numbers.

---

## Section 4: Coverage Claims

**Claim C1: Active ranking minimum game threshold**

> **Claim text:** "A team needs at least 6 games in our system to receive a ranking. Teams with fewer games are tracked but not yet ranked — we show them as 'in progress' rather than guessing at an inaccurate number."

**Evidence:** USA Rank Ranking Accuracy Audit — April 2026, Section 4 (Missing Team Investigation) — threshold referenced as "< 6 games" for ranking eligibility.

**Confidence:** High.

**Authorized for DSS:** ✅ Yes.

---

**Claim C2: Trustworthy ranking threshold**

> **Claim text:** "A team with 15+ games has a stable ranking. Below that, we're more uncertain — the rating is preliminary."

**Evidence:** Glicko-2 design — RD (Rating Deviation) decreases with game count. ELO judgment: 15 games is where RD typically stabilizes below the threshold for confident ranking. This is consistent with Calibration Validation 2026-04-22 calibration analysis.

**Confidence:** Medium — the 15-game threshold is ELO's judgment, not a formally validated cutoff.

**Authorized for DSS:** ⚠️ Use "15+ games" as the stable threshold guidance when discussing individual teams. If the specific team shown in the demo has fewer than 15 games, note it on the team card rather than presenting the rating as final.

---

**Claim C3: USYS state league coverage**

> **Claim text:** "We're expanding our coverage to USYS state leagues, which USA Rank doesn't rank. That means more teams — including clubs that don't play ECNL or MLS NEXT — get fair rankings."

**Evidence:** USA Rank Ranking Accuracy Audit — April 2026, Section 4 (Missing Team Investigation) — 5 of 16 reference teams are at high risk of being absent from Evo Draw due to SnapSoccer/SincSports sources. FORGE's GotSport org ID expansion task was assigned April 24.

**Confidence:** Low — conditional on FORGE completing the org ID expansion and USYS state league games appearing in rankings by May 14.

**Authorized for DSS:** ⚠️ Conditional — only if FORGE confirms state league expansion is complete and Step 4C of the DSS Data Readiness Validation Plan returns a non-zero count. If Step 4C returns 0, do NOT claim state league coverage at DSS.

---

## Section 5: What NOT to Claim

**DO NOT claim these at DSS:**

1. **"Our rankings are the most accurate in youth soccer."** — Not authorized. ELO has not run a head-to-head Brier comparison against USA Rank's algorithm. "More accurate" requires a comparison baseline we do not have.

2. **"Our U13 and U14 Girls Brier scores are below 0.23."** — Not authorized. Pre-fix Brier values are 0.312 and 0.273, both above threshold. Post-fix values have not been measured. Quoting pre-fix numbers as accuracy proof would be misleading.

3. **"Boys rankings are fully calibrated and validated."** — Not authorized unless Boys Option A passes. As of April 25, Boys have not been separately Brier-validated. Boys calibration design is sound but unvalidated by gender-specific analysis.

4. **"We rank all youth soccer teams."** — Not authorized. EDP Soccer, SincSports, and SnapSoccer events are currently not covered (or are partially covered). Approximately 5 of the 16 USA Rank reference teams for U13G/U14G are likely absent from Evo Draw due to northeast corridor source gaps.

5. **Any claim about league-specific accuracy for leagues without cross-validated calibration values.** — Not authorized for leagues outside the 575K-game cross-league study. USYS state leagues are the primary example — calibration values for those tiers have not been empirically validated.

---

## Section 6: SENTINEL Sign-Off Block

```
ELO authorization: I confirm the claims in Sections 1–4 are backed by the cited evidence
                   and are accurate as of the date of this document. Claims marked ⚠️
                   require the stated conditions to be confirmed before use at DSS.
ELO signature: ELO — 2026-04-25

SENTINEL review: [ ] Claims reviewed. No unsupported claims found.
                 [ ] Section 5 prohibitions communicated to Presten before DSS.
                 [ ] ⚠️ conditional claims confirmed/denied by May 14 sign-off.
SENTINEL sign-off: SENTINEL — [date]
```

---

## Pre-DSS Claim Confirmation Checklist

By May 14, confirm each conditional claim:

| Claim | Condition to Confirm | Status |
|-------|---------------------|--------|
| G1 (top-tier alignment) | Audit SQL run; FA NJ merge confirmed | [ ] |
| G2 (coverage count) | Step 4A active team count filled in | [ ] |
| G4 (calibration fix) | April 29 G1 gate confirmed | [ ] |
| B3 (Boys spot check) | Boys Option A PASS by May 9 | [ ] |
| C3 (state league coverage) | Step 4C returns non-zero | [ ] |

Any unchecked ⚠️ claim on May 14 → remove from demo script before DSS.

---

## References

- `Rankings/USA Rank Ranking Accuracy Audit — April 2026.md` — primary cross-validation framework
- `Rankings/Boys Calibration Status Summary — April 2026.md` — Boys calibration context and Option A dependency
- `Rankings/Boys Option A Spot Check — Presten Execution Package.md` — Boys verdict condition for B3
- `Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md` — GA ASPIRE fix basis for G4
- `Product & Planning/DSS Data Readiness Validation Plan — May 14-15.md` — Step 4A/4C confirmations
- `Product & Planning/DSS Demo Spec — May 2026.md` — demo script this spec informs

*ELO — 2026-04-25*
