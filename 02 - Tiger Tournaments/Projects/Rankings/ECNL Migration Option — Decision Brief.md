---
type: elo-decision-brief
topic: ecnl-migration-option
date: 2026-04-29
decision_deadline: 2026-04-30
elo_recommendation: Option 1
status: pending-sentinel-authorization
tags: [elo, ecnl, migration, decision-gate, sentinel-gate]
---

# ECNL Migration Option — Decision Brief

> **For Presten:** Read in under 3 minutes. One decision required by April 30 EOD. **ELO recommends Option 1.**
> **For SENTINEL:** Gate document for April 30 ECNL migration option authorization.

---

## Section 1: Decision Required

Presten must select Option 1 (No Re-tag) or Option 2 (Re-tag to ecnl_rl) for handling ECNL team entity migration before April 30 EOD; FORGE begins implementation May 1 on decision receipt.

---

## Section 2: Options at a Glance

| | **Option 1 — No Re-tag** | **Option 2 — Re-tag to ecnl_rl** |
|-|--------------------------|----------------------------------|
| **Approach** | Keep historical ECNL games tagged as `ecnl` (cal=110); new post-June-1 ECNL RL games tag as `ecnl_rl` (cal=55); no recompute | Retag all historical ECNL games for migrating teams to `ecnl_rl` on June 1; run full recompute |
| **Rating continuity** | < 5 pts average disruption on migration date; Path A teams carry forward earned ratings | 40–80 pt overnight rating drop for ~1,000+ Path A teams on June 1 |
| **FORGE implementation effort** | Low — event classifier update + `ecnl_transition` records in `team_merges`; no schema changes, no UPDATE SQL | High — mass UPDATE on historical games + full recompute; DDL not required but risk surface is larger |
| **Risk to May 9 rankings** | Low — no recompute triggered by migration itself | Medium — full recompute could surface anomalies in the May 9 readiness window |
| **ELO preference** | **RECOMMENDED** | Not recommended |

Option 3 (ecnl_legacy tier) has been removed from consideration: it requires new DDL and a blended calibration value ELO has no cross-league data to support. Option 3 cannot meet the June 1 timeline.

---

## Section 3: ELO Recommendation

ELO recommends **Option 1 — No Re-tag**. Option 1 requires no schema changes and no UPDATE SQL on historical games, making it the cleanest path given the June 1 deadline. The critical tradeoff is a seam: Path A teams (ex-ECNL, now in ECNL RL) will carry ECNL-calibrated ratings into ECNL RL for 6–12 months as historical games recency-weight out. This is a defensible outcome — those teams legitimately earned those ratings in ECNL competition, and the gap narrows naturally through the ranking cycle.

Option 2's 40–80 point overnight cliff is not defensible to club directors. A team that placed well in ECNL competition seeing a sudden rating drop on June 1 will generate support inquiries and undermine trust in Evo Draw's rankings at exactly the moment DSS is launching. Option 1 avoids that cliff entirely.

---

## Section 4: What Happens After Decision

**If Option 1 selected (RECOMMENDED):**
- FORGE activates the Option 1 section in `Infrastructure/ECNL Migration — FORGE Pipeline Impact Spec.md`
- FORGE confirms event classifier routes new ECNL RL events to `event_tier = 'ecnl_rl'` starting June 1
- FORGE inserts `ecnl_transition` records in `team_merges` with `verified = TRUE` for Path A teams
- ELO monitors Path A vs. Path B rating gap at June 7–14; files a revised recommendation if gap exceeds 150 points
- Checkpoint sequence CP1–CP7 (April 29–May 10) validates Option 1 viability — ELO flags immediately if any checkpoint disqualifies it
- Expected completion: May 1–14 (FORGE implementation), June 1 (migration executes)

**If Option 2 selected:**
- FORGE writes mass UPDATE SQL to retag all historical ECNL games for Path A teams to `ecnl_rl`
- Full recompute required — estimated FORGE effort: significantly larger than Option 1
- ELO monitors rating continuity via CP1/CP2 checkpoints; reviews rating cliff magnitude before June 1 execution
- Expected completion: May 10–20 (longer due to UPDATE + recompute testing)
- Additional risk: full recompute in the May 9 readiness window could destabilize ratings before DSS go/no-go gate

---

## Section 5: FORGE Handoff Requirements

- [ ] **Option decision from Presten** (required before FORGE begins — April 30 EOD)
- [ ] **SENTINEL authorization** (required before FORGE activates any implementation)
- [ ] If Option 1: no additional ELO inputs required — FORGE activates per Pipeline Impact Spec
- [ ] If Option 2: ELO provides Path A team list from `rankings` + confirmation query (Continuity Spec §1) before FORGE writes UPDATE SQL
- [ ] Timing constraint: FORGE must complete implementation by May 14 to allow ELO to run June 2 verification spec before June 1 execution

---

## Section 6: SENTINEL Authorization Request

> ELO recommends Option 1 — No Re-tag for ECNL migration. Option 1 preserves rating signal, avoids a 40–80 pt cliff for 1,000+ teams, requires no schema changes, and fits the June 1 timeline cleanly. ELO requests SENTINEL authorize the Option 1 implementation path and direct FORGE to begin after April 30 decision receipt. If SENTINEL overrides to Option 2, Presten sign-off is required per Continuity Spec §5 before any UPDATE runs.

---

## References

- `Rankings/ECNL Migration — ELO Option Recommendation.md` — full recommendation with checkpoint disqualification conditions
- `Rankings/ECNL Migration — Option Comparison Matrix.md` — running verdict tracker (CP1–CP7)
- `Rankings/ECNL Migration — Option Selection Decision Gate.md` — evidence collection framework
- `Rankings/ECNL Migration Rating Continuity Spec — June 2026.md` — Option 1 rating continuity analysis
- `Rankings/ECNL Migration — ELO-FORGE Handoff Checklist.md` — FORGE implementation details

*ELO — 2026-04-29*
