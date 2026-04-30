---
type: elo-scope-decision
topic: tier2-leagues-calibration
date: 2026-04-29
due: 2026-05-02
status: filed
sentinel_review_due: 2026-05-02
presten_decision_required: true
tags: [elo, calibration, tier2, usl-academy, elite64, edp, nal, sentinel-gate]
---

# Tier 2 Leagues — Calibration Scope Decision

> **Purpose:** Four Tier 2 leagues (USL Academy, Elite 64, EDP, NAL) show calibration status as UNDEFINED in the Pre-May-9 Calibration State Dashboard. This document explains the current state, ELO's recommendation for each, the DSS demo impact, and what Presten must decide.
>
> **Audience:** SENTINEL review by May 2. Presten decision before May 9 DSS.

---

## Section 1 — Current State per League

| League | Current Cal Value in Engine | Source of Cal Value | Validation Status | Games in DB |
|--------|-----------------------------|---------------------|-------------------|-------------|
| **USL Academy** | NOT SET — events likely falling through to engine default (untiered, lowest K-factor) | Not defined in League Hierarchy or engine config | UNVALIDATED | Unknown; USL Academy has ~90 clubs nationally. Volume likely non-trivial. |
| **Elite 64** | NOT SET — engine default | Not defined in League Hierarchy or engine config | UNVALIDATED | Unknown; tournament series format limits cross-league game volume. Expected low. |
| **EDP** | NOT SET — engine default | Not defined in League Hierarchy or engine config | UNVALIDATED | Unknown; regional (Eastern) scope. Expected moderate. |
| **NAL** | NOT SET — engine default | Not defined in League Hierarchy or engine config | UNVALIDATED | Unknown; North American League, limited geographic footprint. Expected low. |

**Engine default behavior for undefined tiers:** Per `Calibration Values — League Hierarchy Reconciliation.md`, events with no defined tier code are treated as untiered. ELO's assessment: untiered events likely receive no calibration adjustment (cal=0 or a system minimum), which means teams playing only in these leagues receive no meaningful league-quality signal in their ratings. This understates the quality of teams in these leagues relative to validated Tier 2 peers.

**Source:** `Rankings/Calibration Values — League Hierarchy Reconciliation.md` Section 3 (Missing Tier Analysis); `League Hierarchy` authoritative table as of 2026-04-24.

---

## Section 2 — ELO's Recommendation

| League | ELO Recommendation | Rationale |
|--------|-------------------|-----------|
| **USL Academy** | **Set to Tier 2 default: cal = 55** | USL Academy is a national development league with ~90 clubs. Competitive level is well-characterized as Tier 2 (between ECNL RL / NPL at 55 and GA at 100/140). Setting at 55 aligns with NPL/DPL peers. No Brier analysis available, but the Tier 2 positioning is unambiguous from league structure. Revisit post-May with actual Brier data if DB volume justifies. |
| **Elite 64** | **Set to Tier 2 default: cal = 55** | Tournament series format; cross-league game volume is limited, making empirical validation impractical pre-May-9. The 55 default aligns with other Tier 2 peers and is a reasonable conservative estimate. Flag as unvalidated in League Hierarchy. |
| **EDP** | **Set to Tier 2 default: cal = 55** | Eastern Development Program; regional scope, Tier 2 characterization is consistent with its position in the competitive landscape. No cross-league data available to set a different value. 55 is the correct default pending Brier validation. |
| **NAL** | **Defer to post-May-9** | North American League has limited coverage and ELO has insufficient information to characterize its competitive level confidently. Setting it at 55 (Tier 2 default) risks overcalibrating or undercalibrating. NAL teams are unlikely to appear in the May 9 demo. Recommend deferring NAL to a post-June calibration review when more game data is available. If Presten has information about NAL's competitive positioning, override ELO's default recommendation. |

**Note on implementation:** Setting these values requires adding tier codes to the engine config and League Hierarchy. This is a low-complexity change (config entries only, no schema changes). FORGE does not need to be involved unless the tier detection logic in `compute-rankings.js` requires an update to recognize these tier codes from raw GotSport data.

---

## Section 3 — DSS Demo Impact

| League | DSS Demo Visible? | Impact of UNDEFINED Calibration |
|--------|------------------|--------------------------------|
| **USL Academy** | Unlikely — USL Academy teams are not in the primary demo set | LOW. If a USL Academy team appears in demo rankings, it will show as untiered (no quality signal). Not blocking but could cause a question if noticed. |
| **Elite 64** | No — tournament format, not a season league in demo | NONE. Elite 64 results are events, not leagues with season-long ratings. No demo exposure. |
| **EDP** | Unlikely — regional eastern league, not in demo target market | LOW. Same as USL Academy — if present, shown as untiered. |
| **NAL** | No — NAL is not a target demo league | NONE. NAL teams not expected in demo scope. |

**Overall DSS Demo Impact Assessment: LOW**

The four UNDEFINED leagues do not appear in the primary May 9 demo flow. The calibration gap is a completeness issue, not a demo-blocking risk. SENTINEL can represent this accurately at May 9 as: "Four Tier 2 leagues have pending calibration values; ELO has recommended defaults; Presten review is the only remaining step; none of these leagues appear in the demo scope."

---

## Section 4 — SENTINEL Decision Request

```
Decision needed from Presten:

[ ] OPTION A: Approve ELO's recommended defaults for all four leagues
    - USL Academy → cal 55
    - Elite 64 → cal 55
    - EDP → cal 55
    - NAL → deferred to post-May-9 (use engine default until reviewed)

[ ] OPTION B: Override ELO's recommendation for specific leagues
    (specify which leagues and what values)

[ ] OPTION C: Defer all four Tier 2 leagues to post-May-9 formal calibration review
    (no changes made before DSS; gap documented and disclosed at demo if asked)

SENTINEL can authorize ELO to apply Option A defaults: YES / NO
Presten decision required by: 2026-05-07 (before May 9 DSS gate)
```

**ELO note:** If Presten approves Option A, ELO can apply the three cal values (USL Academy, Elite 64, EDP) to the League Hierarchy documentation and flag to FORGE for engine config update. The change is low-risk and low-effort. NAL deferral requires no action. ELO does not need a DB session to document the values — FORGE applies them to the engine config in the next maintenance cycle.

---

## Related Documents

- `Rankings/Calibration Values — League Hierarchy Reconciliation.md` — Section 3 (missing tier analysis)
- `Rankings/Pre-May-9 Calibration State Dashboard.md` — Section 1 (current calibration state)
- `Rankings/May 9 DSS Gate — Risk Register.md` — Risk Register for coverage gaps

*ELO — 2026-04-29*
