---
title: ECNL Migration — Option Selection Filing Protocol
tags: [ecnl, migration, option-selection, elo, infrastructure, sentinel-gate]
created: 2026-04-28
author: ELO
status: template — fill during April 29 session
---

# ECNL Migration — Option Selection Filing Protocol

> **Instructions:** Write Sections 1–5 as a fill-in template before the April 29 session (fields blank, structure complete). During the April 29 session: fill in Section 2 evidence as results come in. After session: write Section 3 rationale and Section 4 risk register update. File completed document within 30 minutes of April 29 session end.

---

## Section 1: Decision Summary

| Field | Value |
|-------|-------|
| **Option selected** | _[A / B — fill April 29]_ |
| **Date decided** | 2026-04-29 |
| **Filed by** | ELO |
| **Decision gate reference** | `Rankings/ECNL Migration — Option Selection Decision Gate.md` |

---

## Section 2: Evidence Summary

*Complete during April 29 session. Fill in from live results.*

| Evidence | Source | Result |
|----------|--------|--------|
| Boys Option A | April 29 Synthesis Report §4 | _[PASS / FAIL / INCONCLUSIVE]_ |
| G2 gate (GA ASPIRE post-fix direction) | April 29 Synthesis Report §3 | _[UP / DOWN / FLAT]_ |
| ECNL rating continuity check | `Rankings/ECNL Migration Rating Continuity Spec — June 2026.md` SQL | _[PASS / FAIL]_ |
| Historical rating correlation (Option A teams vs. reference) | Continuity spec output | _[value]_ |

**Combined interpretation:** _[ELO writes 2–3 sentences explaining what the evidence collectively says about migration risk — fill after queries]_

---

## Section 3: Option Rationale

### If Option A selected (preserve ratings, no reset)

- **Rating seeding methodology:** Carry-forward — existing ELO ratings preserved unchanged through migration
- **Which existing ratings are preserved:** All current ECNL-tagged teams with ≥ 1 game in system
- **Which are discarded:** N/A under Option A — no ratings discarded
- **Migration execution window:** June 2, 2026
- **Risk:** Teams with thin game history receive no correction — acceptable given DSS timeline

### If Option B selected (rating reset)

- **Bridging logic:** _[delta application method — fill if Option B selected]_
- **Required testing before June 2:** _[minimum validation steps]_
- **Risk:** Rating carry-forward introduces pre-migration calibration errors into post-migration system
- **SENTINEL note:** Option B requires SENTINEL re-review before June 2 execution; ELO must notify by May 30

---

## Section 4: June 1–2 Risk Register Update

**Reference:** `Rankings/June 1 Risk Assessment — Age Group Transition + ECNL Migration.md`

Update the following entries with confirmed option:

| Scenario | Option A Impact | Option B Impact |
|----------|----------------|----------------|
| Scenario A (June 1 on-time) | Lower execution time — no re-seed required | Higher execution time — bridging logic must run |
| Scenario B (June 1 +3–7 day slip) | Slip does not change option risk | Slip compresses bridging validation window |
| Scenario C (June 1 +14 days) | Outer bound unchanged | Outer bound unchanged |

**New risks introduced by selected option:** _[ELO identifies and adds — fill post-session]_

---

## Section 5: SENTINEL Handoff

*Notification format for next SENTINEL briefing:*

> **ECNL Migration Decision Filed** — Option _[A/B]_ selected based on April 29 results. Evidence: _[one sentence summarizing key evidence]_. June 1 risk register updated. _[If Option B: SENTINEL re-review required before May 30.]_

---

*ELO — template pre-staged 2026-04-28 17:25 | complete by 30 min after April 29 session end*

## References

- `Rankings/ECNL Migration — Option Selection Decision Gate.md`
- `Rankings/ECNL Migration — Option Comparison Matrix.md`
- `Rankings/ECNL Migration — ELO Option Recommendation.md` — ELO recommendation filed 2026-04-28
- `Rankings/ECNL Migration — CP1 CP2 Checkpoint Results.md` — CP1/CP2 data (April 29)
