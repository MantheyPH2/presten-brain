---
title: Source Coverage Priority Matrix — April 2026
tags: [infrastructure, sources, gap-analysis, coverage, evo-draw, forge, triage]
created: 2026-04-25
updated: 2026-04-25
author: FORGE
status: complete
task: task-2026-04-25-source-coverage-priority-matrix
---

# Source Coverage Priority Matrix — April 2026

> [!info] Purpose
> Decision-support matrix for source gap triage. When Presten has 30–60 minutes for source work, this matrix answers "which gap closes first?" without requiring a research session. Every known gap is scored and ranked. Top-3 recommendations are explicit and actionable.

**Source:** [[Source Gap Inventory — April 2026]]  
**League tier data:** [[League Hierarchy]]  
**Prepared by:** FORGE — 2026-04-25

---

## Scoring Methodology

| Dimension | 1 | 2 | 3 |
|-----------|---|---|---|
| **Volume** | <1,000 games/season | 1,000–5,000 games/season | >5,000 games/season |
| **Impact** | Low — unlikely to affect any ranked team | Medium — affects depth below top-50 or regional cohorts | High — absence creates ranking distortions for top-50 teams; ECNL/GA/MLS NEXT players notice |
| **Effort** | Hard — new scraper, API integration, or legal/access review (weeks of FORGE work) | Medium — browser session or small script change (15–60 min Presten time) | Easy — already on GotSport, add org_id to config (5 min Presten time) |

**Priority Score = Volume + Impact + Effort. Ties broken by Impact (higher wins), then Volume.**

---

## Section 1: Priority Matrix

| Rank | League / Gap | Current Status | Volume (1–3) | Impact (1–3) | Effort (1–3) | Score | Path to Close |
|------|-------------|----------------|:---:|:---:|:---:|:---:|--------------|
| **1** | ECNL RL Boys/Girls — ingestion recency unverified | Active, unverified | 3 | 3 | 3 | **9** | Run verification query (Section 2 of Source Gap Inventory). If stale: check crawl logs; re-run org-level crawl if needed. 15 min. |
| **2** | GA ASPIRE — misclassified as `ga` tier (overcalibrated +40 pts) | Partial/Fix pending | 2 | 3 | 3 | **8** | April 28 SQL fix scripted in Reference Card. Presten executes UPDATE + recompute-rankings. ~60 min. |
| **3** | USYS State Cup/League (unranked divisions) | Partial — P1 gap | 3 | 2 | 2 | **7** | TX org-ID test crawl (Stage 1) per Validation Spec Section 2. After TX passes: add all P1 state org IDs from Master List. Already scoped and scheduled. |
| **4** | EDP (Eastern Development Program) | Partial — P2 gap | 2 | 2 | 3 | **7** | DB query (`SELECT COUNT(*) FROM games WHERE event_name ILIKE '%EDP%'`). If partial coverage confirmed: browser lookup at `system.gotsport.com` → "EDP Soccer" to get org_id. Add to config. ~15–30 min. |
| **5** | NPL/DPL — ingestion recency unverified | Active, unverified | 2 | 2 | 3 | **7** | Run verification query: `SELECT event_tier, MAX(created_at), COUNT(*) FROM games WHERE event_tier IN ('npl','dpl') GROUP BY event_tier`. If stale: check crawl config. 10 min. |
| **6** | USL Academy — org_id not confirmed | Stale/Partial — P2 gap | 2 | 2 | 2 | **6** | Browser lookup at `system.gotsport.com` → search "USL Academy." High probability org_id exists (USL Soccer is a GotSport organization). Add to config if confirmed. ~15–20 min. |
| **7** | Elite 64 — no org_id confirmed | No source — P1 gap | 2 | 2 | 2 | **6** | Browser lookup at `system.gotsport.com` → search "Elite 64," "US Club Soccer." If org_id found: add to config (same path as USYS states). If not found: check Elite 64 website for platform. ~15 min. |
| **8** | Illinois / Affinity Soccer states | No source — P1 gap | 3 | 2 | 1 | **6** | Requires new Affinity Soccer scraper. Research complete — see `Data Sources/Affinity Soccer Source Research.md`. Deprioritized: post-DSS. No action before June 2026. |
| **9** | SnapSoccer | No source — NO-GO | 1 | 1 | 2 | **4** | **NO-GO per existing decision.** ~1,500–2,500 games/year insufficient to justify DSS sprint. Post-DSS trigger: stable access confirmed + team match ambiguity <5%. |
| **10** | NAL (North American League) | No source — P1 gap | 1 | 1 | 1 | **3** | Platform unknown. Required step: search GotSport for "North American League." If not found: identify NAL website and result publication platform. Volume estimate needed before committing engineering time. Deferred until platform confirmed. |
| **11** | Other Affinity states (WI, MN, MO) | No source | 1 | 1 | 1 | **3** | Research incomplete. Platform not confirmed. Post-DSS. No action until Illinois gap is addressed. |

---

## Section 2: Top-3 Recommendations

### Rank 1 — ECNL RL Verification (Score 9)

ECNL RL is a ~700-club Tier 2 league (300 girls + 400 boys) with calibration 55. It is one of the highest-volume sources in the pipeline and directly affects rankings for thousands of teams. The source is marked Active in the inventory, but last ingestion date has never been verified. A single 10-minute DB query confirms whether this is actually current or silently stale. If stale, the fix is a crawl config check — no engineering required. **Presten action: run the verification query. 10 minutes.**

```sql
SELECT event_tier, MAX(created_at) AS last_game_ingested, COUNT(*) AS game_count
FROM games g
JOIN event_tiers e ON e.event_name = g.event_name
WHERE event_tier IN ('ecnl_rl', 'npl', 'dpl', 'pre_ecnl')
GROUP BY event_tier
ORDER BY event_tier;
```

### Rank 2 — GA ASPIRE Tier Fix (Score 8)

Girls GA ASPIRE is overcalibrated at 140 (the `ga` tier value) when it should be 100 (`ga_aspire`). This creates a ~40-point inflation for 350–1,050 GA ASPIRE-level teams, visible in the top rankings at DSS. The fix SQL is already written and fully staged in the April 28 Escalation Reference Card — it is not a design problem, it is a pending execution. **Presten action: April 28 session, ~60 min. Already on calendar.**

### Rank 3 — USYS State Cup/League Unranked Divisions (Score 7)

20,000–35,000 estimated missing games from unranked state cup and state league divisions — the largest single coverage gap in the pipeline by game volume. The ingestion path is confirmed (GotSport org-level discovery), the approach is validated, and the TX test crawl is the next step. Stage 1 results gate Stage 2. **Presten action: run TX test crawl per Validation Spec Section 2. ~30–60 min of active monitoring.** This one action gates the entire state org-ID expansion.

---

## Section 3: Deferral List

The following gaps score ≤3 or have explicit NO-GO decisions. SENTINEL will not schedule work for these until higher-priority gaps are resolved.

| Gap | Score | Reason for Deferral |
|-----|:-----:|---------------------|
| **NAL (North American League)** | 3 | Platform unknown. Volume likely low (<1,000 games/season). Research required before committing any engineering time. Re-evaluate post-DSS if platform is confirmed accessible. |
| **Other Affinity states (WI, MN, MO)** | 3 | Research incomplete. If these states use Affinity Soccer, they share the same hard blocker as IL: no existing Affinity scraper. Post-DSS. |
| **SnapSoccer** | 4 | Explicit NO-GO decision filed 2026-04-24. Volume insufficient. Post-DSS trigger conditions documented in `Data Sources/SnapSoccer Ingestion MVP — Design 2026-04-24.md`. |

---

## Related

- [[Source Gap Inventory — April 2026]] — full inventory with gap detail and next actions
- [[League Hierarchy]] — tier and calibration values used for Impact scoring
- `Data Sources/GotSport USYS Org-ID Master List.md` — state org IDs for USYS expansion
- `Infrastructure/Pipeline Deployment Validation Spec — April 2026.md` — TX test validation criteria
- `Product & Planning/April 28 Escalation Reference Card.md` — GA ASPIRE fix execution guide

*FORGE — 2026-04-25*
