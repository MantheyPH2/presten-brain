---
title: Non-GotSport Source Implementation Priority List
tags: [infrastructure, sources, priority, coverage, dss, evo-draw, forge, sentinel]
created: 2026-04-28
updated: 2026-04-28
author: FORGE
status: filed
task: task-2026-04-28-non-gotsport-implementation-priority-list
due: 2026-04-30
---

# Non-GotSport Source Implementation Priority List

> [!info] Purpose
> Converts the April 2026 Non-GotSport Source Research Synthesis into a ranked, actionable implementation order. SENTINEL uses this document to authorize Stage 2 non-GotSport source expansion. All sources from the synthesis are ranked; none omitted.

**Prepared by:** FORGE — 2026-04-28  
**Research basis:** `Infrastructure/Non-GotSport Source Research — April 2026 Synthesis.md`  
**DSS deadline:** May 16  
**Readiness check:** May 9

---

## Section 1: Prioritization Criteria

FORGE ranked sources using five weighted criteria:

| Criterion | Weight | Description |
|-----------|--------|-------------|
| **Game volume** | High | Additional unique games/year added to the pipeline |
| **Parse complexity** | High | Can an existing parser be reused? (LOW = config add; MEDIUM = adapted scraper; HIGH = new scraper) |
| **Data quality confidence** | Medium | Structured API > semi-structured HTML > manual export |
| **Authorization risk** | Medium | Public data vs. login-required vs. partner agreement |
| **Time-to-first-game** | High | Days from implementation start to first successfully ingested game |

**Pre-DSS constraint:** The May 9 readiness check and May 16 DSS constrain implementation window to ~11 days from today (April 28). Any source that cannot complete implementation + first-run validation + rating impact assessment by May 8 is marked **Pre-DSS Safe = NO** and deferred to post-DSS.

**ELO calibration priority note:** ELO has identified Northeast Tier 2 coverage (EDP) and professional pathway coverage (USL Academy) as calibration-relevant gaps — teams from these leagues appear in the top-500 rankings but their full game histories are missing. These gaps rank higher as a result.

---

## Section 2: Ranked Priority Table

| Rank | Source | Parser Reuse | Game Volume Est. | Parse Complexity | Time-to-First-Game | Pre-DSS Safe? | Notes |
|------|--------|:---:|:---:|:---:|:---:|:---:|-------|
| 1 | **EDP (Eastern Development Program)** | Full — GotSport API (identical to existing pipeline) | 3,000–8,000 games/yr | LOW | < 1 day (config add after org_id confirmed) | **YES** | Blocked only on browser session org_id confirmation. Once confirmed: 5-min config add, first run on next pipeline execution. |
| 2 | **USL Academy** | Full — GotSport API | 2,000–5,000 games/yr | LOW | < 1 day (config add after org_id confirmed) | **YES** | Same config-add path as EDP. 90 clubs, professional pathway identity. High probability GotSport org_id exists. |
| 3 | **Elite 64** | Full — GotSport API (if confirmed on GotSport) | 500–2,000 games/yr | LOW (if GotSport) / MEDIUM (if separate platform) | < 1 day if GotSport | **CONDITIONAL** | Pre-DSS Safe = YES if browser session confirms GotSport org_id. Pre-DSS Safe = NO if not on GotSport (defers to platform assessment). |
| 4 | **NAL (North American League)** | Unknown | < 1,000 games/yr (estimated) | UNKNOWN | Unknown until platform confirmed | **NO** | Platform not confirmed. Requires 15-min GotSport search before any engineering decision. If GotSport: reassess — may become Pre-DSS Safe on same config-add path. |
| 5 | **SnapSoccer (SincSports)** | Partial — SincSports HTML parser adaptable | 1,500–2,500 games/yr (ongoing); 10,000–20,000 historical | MEDIUM | 5–9 hrs (test) / 8–12 hrs (production) | **NO** | Explicit NO-GO decision filed 2026-04-24. Volume insufficient to justify DSS sprint risk vs. Org-ID expansion ROI. Post-DSS trigger checklist in design doc. |
| 6 | **Affinity Soccer (IL + Midwest)** | None — new scraper required | 5,000–15,000 games/yr (IL alone) | HIGH | Weeks (new scraper + access path + dedup design) | **NO** | Hard engineering blocker. No existing scraper. No confirmed public access path. Largest long-term volume opportunity but not actionable before June 2026. |

---

## Section 3: Implementation Sequence

### Source 1: EDP (Eastern Development Program)

**Implementation start:** Within 1 hour of browser session org_id confirmation (target: April 28–29)

**Steps:**
1. Receive EDP org_id from browser session results
2. Open `Infrastructure/Stage 2 Config Pre-Population — Pending Org-IDs.md` → find EDP entry
3. Replace `[TBD-ORG-ID]` with confirmed org_id
4. Paste cleaned entry into crawl config (GotSport org-level section)
5. Update `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` → move EDP from Section 2 to Section 1, status → `confirmed-[org_id]`
6. Run baseline DB count: `SELECT COUNT(*) FROM games WHERE event_name ILIKE '%EDP%' OR event_name ILIKE '%Eastern Development%'` (pre-config baseline)
7. After next pipeline run: re-run count, confirm delta > 0

**Expected first successful run:** May 1 pipeline launch (if browser session completes by April 30); otherwise, next pipeline execution after config update

**Validation criteria:** New games ingested from EDP org_id; no duplicate entries (dedup key `(source, source_id)` handles this natively); `event_tier` classification assigns correct tier to EDP games

**SENTINEL authorization needed before:** Config addition (Stage 2 gate — SENTINEL must authorize before FORGE adds any new org-IDs beyond the initial Stage 1 set)

---

### Source 2: USL Academy

**Implementation start:** Same session as EDP if EDP config add runs cleanly; otherwise day after EDP first-run validation

**Steps:**
1. Receive USL Academy org_id from browser session
2. Follow same config-add process as EDP (Config Pre-Population document → config paste → Master Reference update)
3. Run baseline DB count: `SELECT COUNT(*) FROM games WHERE event_name ILIKE '%USL Academy%' OR event_name ILIKE '%United Soccer League Academy%'`
4. After next pipeline run: confirm delta > 0

**Expected first successful run:** May 1 pipeline launch (if browser session and EDP add complete by April 30)

**Validation criteria:** New games ingested; `event_tier` classification is correct for USL Academy games; no unexpected dedup collisions with existing GotSport data

**SENTINEL authorization needed before:** Config addition (same Stage 2 gate)

---

## Section 4: Sources Deferred to Post-DSS

| Source | Deferral Reason | Revisit Condition | Estimated Post-DSS Start |
|--------|----------------|-------------------|--------------------------|
| **Elite 64** (if not on GotSport) | Platform not confirmed; if not GotSport, requires platform assessment before engineering scoping | Browser session result: if not on GotSport, conduct platform assessment in a post-DSS session | May 17–24 (platform assessment sprint) |
| **NAL** | Platform unconfirmed; volume likely < 1,000 games/yr. Not worth engineering time before platform is known. | Run 15-min GotSport search + website check. If GotSport confirmed: reassess as Tier A config add. If non-GotSport: defer indefinitely pending volume confirmation. | May 17+ (platform check, then re-score) |
| **SnapSoccer** | NO-GO explicit decision filed. Annual ongoing volume (1,500–2,500 games) does not justify 8–12 hr engineering sprint during DSS prep. | (1) USYS org-ID expansion confirmed stable in production. (2) Presten has 6–8 hour session available. (3) Browser test confirms HTML structure matches parsing spec. (4) SincSports parser confirmed reusable. | May 17+ (earliest) |
| **Affinity Soccer** | New scraper required. No confirmed access path. Weeks of engineering. Cannot safely execute before DSS. | Post-DSS: confirm Illinois access path, scope engineering effort, assign as standalone sprint. | June 2026+ |

---

## Section 5: Coverage Gap Impact

### EDP adds:
- **Leagues/states covered:** Northeast U.S. — NJ, NY, PA, CT, MA, MD, RI, DE, VA, VT
- **Teams:** ~150+ clubs, U8–U19
- **Competitive gap closed:** EDP is indexed by GotSport Rankings; Evo Draw may have partial coverage via ranked-team discovery but full org-level ingestion confirms unranked divisions. Closes a gap that DSS audiences coaching Northeast teams will notice.
- **Calibration relevance:** Northeast teams appear in top-500 rankings; incomplete EDP history means their Glicko-2 ratings have inflated RD. Adding full EDP org coverage stabilizes those ratings.

### USL Academy adds:
- **Leagues/states covered:** National — professional pathway clubs with MLS/USL affiliations, 90 clubs
- **Teams:** Boys focus (U13–U19); Girls equivalent not confirmed
- **Competitive gap closed:** GotSport Rankings indexes USL Academy; Evo Draw currently does not. Professional pathway identity (similar status to MLS NEXT in DSS context) strengthens competitive coverage narrative.
- **Calibration relevance:** USL Academy teams are calibrated between NPL and GA tiers (calibration ~70). Adding these teams fills a mid-tier gap where H2H outcomes currently have thin data.

### Combined pre-DSS impact (EDP + USL Academy, if browser session completes by April 30):
- Estimated additional games: **5,000–13,000 games/year**
- Configuration time: **~10 minutes**
- DSS coverage statement: "Evo Draw now covers EDP and USL Academy alongside ECNL, MLS NEXT, GA, ECNL RL, NPL, and DPL — the complete competitive spectrum from professional pathway academies through Tier 2 regional leagues."

**Cross-reference:** Competitive Coverage Gap Inventory Section 2 lists EDP (Medium severity) and USL Academy (Medium severity) as explicitly closeable pre-DSS with browser lookup. Section 4 ranks them #2 and #3 in priority additions (after USYS state unranked divisions, which are handled by the Stage 1/2 org-ID expansion track).

---

## FORGE Recommendation

**Implement Source 1 (EDP) first.** EDP has the highest Priority Matrix score (7) of all non-GotSport sources, covers a geographic gap where ranked teams already appear in Evo Draw rankings with incomplete history, and carries zero engineering risk (config add). The implementation is gated only on the browser session producing an org_id — which should be the first action taken.

**Implement Source 2 (USL Academy) in the same session** if EDP config add produces no anomalies. The process is identical, the risk is identical, and the combined coverage impact justifies the 5-minute second add.

**FORGE does not recommend implementing any Source 3+ before May 9.** The remaining sources either have confirmed NO-GO status (SnapSoccer), unknown platforms (NAL), or engineering blockers that make pre-DSS completion impossible (Affinity Soccer, Elite 64 if non-GotSport).

---

## References

- `Infrastructure/Non-GotSport Source Research — April 2026 Synthesis.md` — research basis for all rankings and estimates
- `Infrastructure/Competitive Coverage Gap Inventory — April 2026.md` — gap severity ratings used in Section 5
- `Infrastructure/Stage 2 Config Pre-Population — Pending Org-IDs.md` — pre-written config entries for all pending entities
- `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` — org-ID tracking; Section 2 contains pending browser-lookup entities
- `Infrastructure/GotSport Org-ID Expansion Stage 2 — Pre-Authorization Criteria.md` — Stage 2 gate conditions
- `task-2026-04-27-snapsoccer-test-extraction-plan.md` — SnapSoccer feasibility
- `task-2026-04-27-sincsports-parser-reuse-check.md` — SincSports parser evaluation

*FORGE — 2026-04-28*
