---
title: May 9 DSS Readiness Final Assessment Spec
aliases: [May 9 Go/No-Go Spec, DSS Final Assessment]
tags: [rankings, dss, readiness, go-no-go, evo-draw]
created: 2026-04-25
updated: 2026-04-25
author: ELO Agent
status: ready
task: task-2026-04-26-may9-readiness-final-assessment-spec
run_on: 2026-05-09
---

# May 9 DSS Readiness Final Assessment Spec

> **Purpose:** Make the May 9 final go/no-go review a 30-minute mechanical check with pre-defined pass/fail criteria. ELO runs each check, applies the decision table, and files the summary to SENTINEL. No judgment calls required — every decision is pre-specified below.
>
> **DSS date: May 16, 2026. Final go/no-go: May 9. Pre-demo check: May 14.**

---

## Section 1: Per-Feature Final Check

For each of the 5 DSS features, run the query, apply the threshold, and mark ✅ or ❌. Total time: ~30 minutes.

---

### Feature 1: Rank Bands

**Check:** Confirm threshold validation passes for the DSS anchor team after April 28 recompute.

```sql
-- Football Academy NJ — confirm Gold status
SELECT t.name, r.rating::int AS rating, r.rank,
  CASE
    WHEN r.rating >= 1500 THEN 'Gold'
    WHEN r.rating >= 1350 THEN 'Silver'
    WHEN r.rating >= 1200 THEN 'Bronze'
    WHEN r.rating >= 1050 THEN 'Red'
    WHEN r.rating >= 900  THEN 'Blue'
    ELSE 'Green'
  END AS band
FROM teams t
JOIN rankings r ON r.team_id = t.id
WHERE r.age_group = 'U13' AND r.gender = 'F'
  AND (t.name ILIKE '%football academy%'
    OR t.name ILIKE '%FA NJ%'
    OR t.name ILIKE '%FA 12G%')
ORDER BY r.rank;
```

Also check via `team_merges` if not found directly:
```sql
SELECT t1.name AS canonical_name, t2.name AS merged_name, r.rating::int, r.rank
FROM team_merges tm
JOIN teams t1 ON tm.canonical_team_id = t1.id
JOIN teams t2 ON tm.merged_team_id = t2.id
JOIN rankings r ON r.team_id = tm.canonical_team_id
WHERE r.age_group = 'U13' AND r.gender = 'F'
  AND (t2.name ILIKE '%football academy%' OR t2.name ILIKE '%FA NJ%')
ORDER BY r.rank;
```

| Result | Status |
|--------|--------|
| Football Academy NJ found, rating ≥ 1500 (Gold), rank ≤ 50 | ✅ Rank Bands PASS |
| Football Academy NJ found, rating ≥ 1500, rank 51–100 | ⚠️ Rank Bands PASS (flag rank to SENTINEL — demo proceeds) |
| Football Academy NJ found, rating < 1500 (not Gold) | ❌ Rank Bands FAIL — do not demo Rank Bands feature |
| Football Academy NJ not found in any name variation | ❌ Rank Bands FAIL — escalate to SENTINEL immediately |

---

### Feature 2: Event Strength Phase 1

**Check:** Confirm implementation session completed and ECNL events are correctly classified as High.

```sql
-- Confirm event_strength table exists and ECNL events are High
SELECT e.event_name, es.strength_class, es.avg_team_rating::int, es.team_count
FROM event_strength es
JOIN events e ON e.id = es.event_id
WHERE (e.event_name ILIKE '%ECNL%' OR e.event_tier = 'ecnl')
  AND es.age_group = 'U13' AND es.gender = 'F'
ORDER BY es.avg_team_rating DESC
LIMIT 10;
```

If the `event_strength` table does not exist, Event Strength is not implemented — mark ❌ immediately, skip to decision table.

| Result | Status |
|--------|--------|
| Table exists; at least one ECNL event is `strength_class = 'High'`; no ECNL National event is `'Low'` | ✅ Event Strength PASS |
| Table exists but ECNL events return `'Low'` or `'Medium'` for national events | ❌ Event Strength FAIL — misclassified; do not demo |
| `event_strength` table does not exist | ❌ Event Strength FAIL — not implemented |

---

### Feature 3: Club Rankings

**Check:** Confirm Club Rankings implementation session completed and club rating coverage meets minimum threshold.

```sql
-- Club Rankings coverage check
SELECT
  COUNT(DISTINCT club_id) AS clubs_ranked,
  COUNT(DISTINCT team_id) AS teams_mapped_to_club,
  (COUNT(DISTINCT team_id)::float / (SELECT COUNT(*) FROM rankings WHERE gender = 'F' AND rank IS NOT NULL)) AS coverage_pct
FROM club_rankings
WHERE gender = 'F';
```

If `club_rankings` table does not exist, feature is not implemented — mark ❌.

Also confirm Boys status from Boys Option A spot check results:

```sql
-- Boys club rankings (only if Boys Option A PASSED)
SELECT COUNT(DISTINCT club_id) AS boys_clubs_ranked
FROM club_rankings
WHERE gender = 'M';
```

| Result | Status |
|--------|--------|
| `club_rankings` table exists, Girls coverage_pct ≥ 60%, top club results look credible | ✅ Club Rankings PASS (Girls) |
| `club_rankings` table exists, Boys Option A passed, Boys clubs_ranked > 0 | ✅ Club Rankings PASS (Boys — include in demo) |
| `club_rankings` table does not exist | ❌ Club Rankings FAIL — not implemented |
| Girls coverage_pct < 60% | ❌ Club Rankings FAIL — coverage too thin for credible demo |
| Boys Option A failed or not run | ❌ Club Rankings Boys excluded — Girls-only demo |

---

### Feature 4: USA Rank Comparison

**Check:** Confirm FORGE's 16 audit queries have been run and ELO has filed the delta review.

```sql
-- Confirm USA Rank reference teams are in rankings with expected bands
SELECT t.name, r.rating::int, r.rank,
  CASE
    WHEN r.rating >= 1500 THEN 'Gold'
    WHEN r.rating >= 1350 THEN 'Silver'
    WHEN r.rating >= 1200 THEN 'Bronze'
    WHEN r.rating >= 1050 THEN 'Red'
    WHEN r.rating >= 900  THEN 'Blue'
    ELSE 'Green'
  END AS band
FROM rankings r
JOIN teams t ON t.id = r.team_id
WHERE r.age_group = 'U13' AND r.gender = 'F'
  AND (t.name ILIKE '%football academy%'
    OR t.name ILIKE '%wall soccer%'
    OR t.name ILIKE '%wall elite%')
ORDER BY r.rank;
```

Also confirm the delta results file exists at `Reports/usarank-accuracy-audit-results-[most recent date].md`.

| Result | Status |
|--------|--------|
| FORGE audit SQL run; delta results file exists; ELO delta review complete; anchor teams found in rankings within expected bands | ✅ USA Rank Comparison PASS |
| FORGE audit SQL run but delta results not yet classified by ELO | ⚠️ USA Rank Comparison PARTIAL — ELO must complete review before May 9 go/no-go |
| FORGE audit SQL NOT run by May 9 | ❌ USA Rank Comparison FAIL — non-negotiable. Contact Presten directly. Do not proceed with demo finalization without this. |
| Anchor teams not found in rankings | ❌ USA Rank Comparison FAIL — same escalation as Rank Bands anchor failure |

---

### Feature 5: USYS/GotSport Coverage

**Check:** Confirm TX org ID has been looked up and TX test crawl confirmed game ingestion.

```sql
-- Count new games ingested from USYS/GotSport sources since April 28
SELECT source, COUNT(*) AS game_count, MAX(ingested_at) AS most_recent
FROM games
WHERE source ILIKE '%usys%' OR source ILIKE '%gotsport%'
  AND ingested_at >= '2026-04-28'
GROUP BY source
ORDER BY game_count DESC;
```

(Adapt column names to match actual schema if `source` or `ingested_at` differ.)

| Result | Status |
|--------|--------|
| TX test crawl complete; ≥ 1,000 new USYS games confirmed in DB | ✅ USYS Coverage PASS — coverage claim is demo-safe |
| TX test crawl complete; < 1,000 games or crawl still running | ⚠️ USYS Coverage PARTIAL — do not cite specific game count; use "we're actively expanding state league coverage" |
| TX org ID not confirmed; test crawl not run | ❌ USYS Coverage FAIL — drop coverage claim entirely from demo script |

---

## Section 2: Decision Table

Apply this table mechanically after completing Section 1. One row per feature.

| Feature | ❌ on May 9 → Action |
|---------|---------------------|
| **Rank Bands** | Primary demo feature drops. Switch to fallback demo (rankings + team detail + USA Rank comparison only). Notify SENTINEL immediately — this is a demo-structure change Presten must approve. |
| **Event Strength Phase 1** | Drop Event Strength from demo entirely. Update DSS Demo Spec Section 5 fallback: remove Event Strength card. Note in demo script: "Event Strength labeling is in development — coming this summer." Do not attempt to show Event Strength live. |
| **Club Rankings (Girls)** | Drop Club Rankings from demo entirely. Remove from demo script and slide deck. If Boys only is passing, drop Boys too — a Boys-only Club Rankings demo creates more questions than it answers. |
| **Club Rankings (Boys only)** | Revert to Girls-only demo. Update demo script to remove Boys club rankings reference. This does not affect the Girls-only demo or other features. |
| **USA Rank Comparison** | This is non-negotiable. Contact Presten directly — not SENTINEL first. USA Rank comparison is the accuracy anchor for the entire demo. If it is ❌ on May 9, the demo is at risk for all features. Presten must run FORGE's 16 audit queries immediately or the demo scope must be reduced to Rank Bands + team detail only. |
| **USYS/GotSport Coverage** | Drop all state league coverage claims from demo script. Do not improvise. Remove any language like "we cover state leagues that USA Rank misses." Replace with: "We're adding state league sources for the 2026-27 season." |

### Fallback demo trigger

If ≥ 2 features are ❌ on May 9, automatically switch to fallback demo plan:
- **Include:** Rank Bands + team detail + USA Rank comparison (if PASS)
- **Drop:** Event Strength, Club Rankings, USYS coverage claims
- Notify SENTINEL with: "Fallback demo triggered. [N] of 5 features ❌. Primary demo requires [list]. Fallback demo sufficient for DSS. Request Presten authorization for fallback scope."

---

## Section 3: ELO Output Format for May 9

After completing Sections 1 and 2, ELO files the following in the May 9 briefing to SENTINEL:

### Required briefing content

**One-paragraph summary:**
> "May 9 DSS Readiness — Data ready: [N]/5. Calibration ready: [N]/5. Features confirmed for primary demo: [list]. Features falling to fallback: [list]. [If USA Rank is ❌: NON-NEGOTIABLE ITEM FLAGGED — USA Rank Comparison not ready. Presten must execute FORGE queries immediately.] Recommendation: [Primary demo / Fallback demo]."

**Updated Readiness Tracker columns** (update `DSS Feature Readiness Tracker — May 2026.md`):

| Feature | Data Ready? | Calibration Ready? | May 9 Status |
|---------|-------------|--------------------|----|
| Rank Bands | [✅/⚠️/❌] | [✅/⚠️/❌] | [PASS/FAIL + brief note] |
| Event Strength Phase 1 | [✅/⚠️/❌] | [✅/⚠️/❌] | [PASS/FAIL + brief note] |
| Club Rankings | [✅/⚠️/❌] | [✅/⚠️/❌] | [PASS/FAIL + Boys/Girls status] |
| USA Rank Comparison | [✅/⚠️/❌] | [✅/⚠️/❌] | [PASS/FAIL — flag if ❌] |
| USYS/GotSport Coverage | [✅/⚠️/❌] | [✅/⚠️/❌] | [PASS/FAIL + brief note] |

**Demo plan determination:** State explicitly — "Primary demo" or "Fallback demo" — and list which features are included.

**Flag line:** If USA Rank is ❌, add a separate flagged line: `⚠️ NON-NEGOTIABLE FLAG: USA Rank Comparison not ready as of May 9. Requires immediate Presten action.`

---

## Section 4: May 14 Pre-Demo Final Check

On May 14, ELO runs the full `DSS Data Readiness Validation Plan — May 14-15.md` query set. This is a separate, more detailed check from the May 9 go/no-go.

**If any result changes between May 9 and May 14:**
- Flag to SENTINEL by 11 AM May 14.
- State the change: "[Feature] was PASS on May 9. On May 14, [observation]. Recommended action: [specific action or 'no change needed']."
- Decision cutoff: **noon May 14.** Any fix that cannot be confirmed complete by noon May 14 is cut from the demo. No changes to demo plan after noon May 14.

**If all results match May 9:**
- File one-line confirmation in briefing: "May 14 pre-demo check complete. No changes from May 9 status. Demo plan unchanged: [Primary/Fallback]. All systems go."

**What ELO does NOT do on May 14:**
- Make new go/no-go decisions (that was May 9).
- Implement new features (cutoff was May 9).
- Change the demo plan without SENTINEL authorization.

---

## References

- [[DSS Demo Validation Criteria — May 2026]] — detailed per-team criteria used in Section 1 checks
- `02 - Tiger Tournaments/Projects/Product & Planning/DSS Feature Readiness Tracker — May 2026.md` — tracker to update after May 9 check
- `02 - Tiger Tournaments/Projects/Product & Planning/DSS Demo Spec — May 2026.md` — primary and fallback demo structure
- [[Boys Option A Spot Check — Presten Execution Package]] — Boys Club Rankings pre-flight; results must be received before May 9
- [[USA Rank Delta Interpretation Spec — April 2026]] — delta classification for USA Rank Comparison feature
- [[DSS Data Readiness Validation Plan — May 14-15]] — May 14 detailed check protocol

*ELO — 2026-04-25*
