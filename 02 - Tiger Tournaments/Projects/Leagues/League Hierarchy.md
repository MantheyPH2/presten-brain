---
title: League Hierarchy
aliases: [Tier System, League Tiers, Competitive Pathway]
tags: [leagues, hierarchy, tiers, calibration, evo-draw, critical]
created: 2026-04-21
updated: 2026-04-24
---

# League Hierarchy

> [!warning] CRITICAL REFERENCE
> This is the authoritative tier hierarchy for US youth soccer. The [[Ranking Engine]] uses these tiers for calibration. **Misclassifying a league tier will corrupt rankings.**

---

## Tier 1 — Elite

### [[MLS NEXT]]
- **Boys #1 league** in the US
- 273 clubs: 30 MLS academies + 243 Elite clubs
- Age groups: U13-U19
- Divisions: Homegrown (Tier 1) + Academy (Tier 2)
- **Calibration: 160 points**

### [[ECNL]]
- **#2 boys / #1 girls (tied with GA)**
- ~130 girls clubs, ~150 boys clubs
- Boys: 15 conferences | Girls: 10 conferences
- **Calibration: 120 boys, 130 girls**

### GA (Girls Academy)
- **#1 girls (tied with ECNL)**
- 120+ clubs
- ASPIRE tier launched Feb 2025
- **Calibration: 140 girls, 100 boys**

---

## Tier 2 — High Competitive

### [[ECNL RL]] (Regional League)
- ~700 clubs (300 girls + 400 boys)
- Feeder to [[ECNL]] — 24 clubs promoted for 2025-26
- 24 boys leagues, 23 girls leagues
- **Calibration: 55 points**

### NPL (National Premier League)
- Merit-based promotion system
- Integrating with USYS National League for 2026-27
- **Calibration: 55 points**

### DPL (Development Players League)
- Girls-focused, GA pathway
- **Calibration: 55 points**

### USL Academy
- 90 clubs with professional pathway
- Connected to USL professional system

### Others at Tier 2
- Elite 64
- EDP (Eastern Development Program)
- NAL (North American League)

---

## Tier 3 — Competitive

### [[Pre-ECNL]]
- U10-U12 developmental program
- 300+ clubs
- **Calibration: 25 points**

> [!warning] Pre-ECNL IS NOT ECNL
> Despite the name, Pre-ECNL is a Tier 3 developmental program for younger age groups. It is NOT the same competitive level as [[ECNL]]. The `resolveTeamTier()` function in [[Ranking Engine]] enforces this distinction.

### State Leagues and Cups
- Organized by state associations under [[Governing Bodies|USYS]]
- Highly variable quality

### ODP (Olympic Development Program)
- Talent identification pipeline
- State → Regional → National selection

---

## Competitive Pathway

```
Recreational → Select → Competitive → High Competitive → Elite → Professional
   (Tier 3)              (Tier 3)        (Tier 2)         (Tier 1)
```

---

## Calibration Values (used in [[Ranking Engine]])

> [!info] MLS NEXT Tier Split (pending May 18–20 implementation)
> The unified `mlsnext: 160` value is being split into `mlsnext_homegrown: 160` and `mlsnext_academy: 135` per [[MLS NEXT Tier Split — Calibration Spec 2026-04-24]]. Values below reflect both current (unified) and post-split state.

### Boys

| League | Cal Value | Post-Split Cal | Validation Status | Notes |
|--------|-----------|---------------|-------------------|-------|
| MLS NEXT (unified, pre-split) | 160 | — | validated | Legacy; will be replaced by splits below |
| MLS NEXT Homegrown | 160 | **160** | validated | 71.4% win rate vs ECNL Boys (n=3,820) |
| MLS NEXT Academy | 160 | **135** | validated | 58.2% win rate vs ECNL Boys (n=4,110) |
| MLS NEXT Unclassified | 160 | **147** | interpolated | Weighted midpoint (60% Academy / 40% Homegrown); recalculate if classifier distribution differs from assumed |
| [[ECNL]] Boys | 120 | 120 | validated | Baseline anchor for cross-league derivations |
| GA (Boys) | 100 | 100 | theoretically sound; not Brier-validated | Boys GA = secondary tier (MLS NEXT is Boys #1). Lower than Girls GA by design. See [[Boys Calibration Gap Analysis — April 2026]] |
| GA ASPIRE (Boys) | 100 (assumed) | 100 (assumed) | **assumed — unvalidated** | GA ASPIRE events for Boys currently tagged as `ga` (cal=100). If Boys GA ASPIRE should sit below Boys GA (analogous to Girls GA ASPIRE at 100 vs Girls GA at 140), the correct Boys GA ASPIRE value would be ~70. Needs Boys-specific Brier analysis to confirm. GA ASPIRE Boys fix is NOT included in the April 28 session — only Girls GA ASPIRE fix is confirmed pending. |
| NPL / DPL / [[ECNL RL]] | 55 | 55 | validated | ECNL RL beats NPL 55.1% — confirms peer-tier status |
| [[Pre-ECNL]] | 25 | 25 | validated | Tier 3 developmental; `resolveTeamTier()` enforces this |

### Girls

| League | Cal Value | Validation Status | Notes |
|--------|-----------|-------------------|-------|
| GA | 140 | validated | Girls #1 tier (tied with ECNL) |
| GA ASPIRE | 100 | validated (post-fix pending) | GA ASPIRE events overcalibrated as `ga` (140) until April 28 fix executes. Correct value: 100. See [[ga-coverage-audit-results-2026-04-23]] |
| [[ECNL]] Girls | 130 | validated | Girls #1 tier (tied with GA) |
| NPL / DPL / [[ECNL RL]] | 55 | validated | Peer tier to Boys equivalents |
| [[Pre-ECNL]] | 25 | validated | Tier 3 developmental |

---

## Per-Team Tier Overrides

> [!warning] Critical Classification Rule
> The `resolveTeamTier()` function in [[Ranking Engine]] applies per-team overrides to prevent misclassification:
> - Teams with **"ECNL RL"** or **"ECRL"** in their name → capped at `ecrl` tier, even if the event is classified as `ecnl`
> - Teams with **"Pre-ECNL"** in their name → capped at `pre_ecnl` tier
> - Without these overrides, [[ECNL RL]] teams playing in ECNL-classified events would get ECNL-level calibration (120/130 instead of 55)

---

## Cross-League Validation

From our [[Cross-League Analysis]] of 575K games:
- [[ECNL RL]] beats NPL **55.1%** → confirms Tier 2 peer status
- [[Pre-ECNL]] beats NPL **68.4%** → surprisingly strong for Tier 3
- State Cup teams beat NPL **51.4%** → very competitive
- [[MLS NEXT]] dominates all tiers

---

## Boys GA ASPIRE Flag

> [!warning] Boys GA ASPIRE — Calibration Status Uncertain
> The Girls GA ASPIRE fix (reclassifying GA ASPIRE events from `ga` tier at cal=140 to `ga_aspire` at cal=100) is scheduled for execution April 28, 2026. For Boys:
>
> - Boys GA events are calibrated at 100 (vs Girls GA at 140). Boys GA ASPIRE events, if also tagged as `ga`, receive cal=100 — the same as regular Boys GA.
> - It is unknown whether Boys GA ASPIRE competition level is materially different from Boys GA. If Boys GA ASPIRE sits below Boys regular GA in quality, the correct Boys GA ASPIRE value would be ~70 (same 30-point differential as Girls: 140 → 100).
> - **Conservative recommendation:** Do not apply a Boys GA ASPIRE fix until Option B Boys Brier analysis (scheduled post-DSS, June 2026) produces data to validate the correct value. The overcalibration risk for Boys is smaller than for Girls because Boys GA (100) is already 40 points below the Girls GA (140) it parallels.
> - **Action required:** Confirm with Presten whether the April 28 GA ASPIRE fix session includes Boys events or Girls-only. If it targets all `ga` tier events by name pattern (events with "ASPIRE" in name), Boys events will be reclassified too — which may or may not be correct depending on the Boys GA ASPIRE calibration decision above.

## Related Notes
- [[Ranking Engine]] — how calibration values are applied
- [[Cross-League Analysis]] — empirical win-rate data
- [[Recent Changes 2024-2026]] — structural shifts in the landscape
- [[MLS NEXT]], [[ECNL]], [[ECNL RL]], [[Pre-ECNL]] — individual league details
- [[Boys Calibration Gap Analysis — April 2026]] — Boys calibration validation status
- [[MLS NEXT Tier Split — Calibration Spec 2026-04-24]] — Academy/Homegrown split derivation
