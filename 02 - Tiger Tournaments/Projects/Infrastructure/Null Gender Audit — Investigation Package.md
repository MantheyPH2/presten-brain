---
title: Null Gender Audit — Investigation Package
type: forge-investigation
author: FORGE
date: 2026-04-29
status: pending-db-execution
topic: null-gender-audit
tags: [data-quality, gender, parsing, audit, evo-draw]
---

# Null Gender Audit — Investigation Package

**Current state:** ~12,000 null gender values (3.3% of ~1.64M visible games). Target: below 1.5% (≤24,600 null; need to eliminate ~7,400+ nulls).

[[Data Quality]] identifies GotSport API as the primary source. [[Parsing Rules]] documents 6 gender inference methods. This package contains the SQL to locate root causes and proposed parsing improvements to resolve the majority.

---

## Section 1 — Source Breakdown Queries (Run First)

Run these in order to identify where nulls concentrate.

### Query 1A — Nulls by Source
```sql
SELECT 
  source, 
  COUNT(*) AS null_count,
  ROUND(COUNT(*)::numeric / SUM(COUNT(*)) OVER () * 100, 1) AS pct_of_all_nulls
FROM games 
WHERE gender IS NULL AND is_hidden = false
GROUP BY source 
ORDER BY null_count DESC;
```

**Expected result:** gotsport_api likely accounts for 70–80% of nulls based on Data Quality notes.

### Query 1B — Nulls by Age Group
```sql
SELECT 
  COALESCE(age_group, 'NULL') AS age_group, 
  COUNT(*) AS null_count
FROM games 
WHERE gender IS NULL AND is_hidden = false
GROUP BY age_group 
ORDER BY null_count DESC;
```

### Query 1C — Top 50 Events with Null Gender (Volume Weighted)
```sql
SELECT 
  event_name, 
  source,
  COUNT(*) FILTER (WHERE gender IS NULL) AS null_gender,
  COUNT(*) AS total_games,
  ROUND(COUNT(*) FILTER (WHERE gender IS NULL)::numeric / COUNT(*) * 100, 1) AS null_pct
FROM games
WHERE is_hidden = false
GROUP BY event_name, source
HAVING COUNT(*) FILTER (WHERE gender IS NULL) >= 20
ORDER BY null_gender DESC
LIMIT 50;
```

**Purpose:** Identifies whether nulls are spread across thousands of events or concentrated in a small number of high-volume events. If top 10 events account for 50%+ of nulls, targeted event-level overrides will be more effective than pattern changes.

### Query 1D — Sibling Inference Coverage Gap
```sql
-- Events where sibling inference SHOULD work but doesn't
-- (division has games, some have gender, but < 95% threshold not met)
SELECT 
  event_name,
  division,
  COUNT(*) AS total,
  COUNT(*) FILTER (WHERE gender = 'Boys') AS boys_count,
  COUNT(*) FILTER (WHERE gender = 'Girls') AS girls_count,
  COUNT(*) FILTER (WHERE gender IS NULL) AS null_count,
  ROUND(GREATEST(
    COUNT(*) FILTER (WHERE gender = 'Boys'),
    COUNT(*) FILTER (WHERE gender = 'Girls')
  )::numeric / COUNT(*) * 100, 1) AS dominant_pct
FROM games
WHERE is_hidden = false
GROUP BY event_name, division
HAVING COUNT(*) FILTER (WHERE gender IS NULL) > 0
  AND COUNT(*) > 5
  AND GREATEST(
    COUNT(*) FILTER (WHERE gender = 'Boys'),
    COUNT(*) FILTER (WHERE gender = 'Girls')
  )::numeric / COUNT(*) >= 0.80
  AND GREATEST(
    COUNT(*) FILTER (WHERE gender = 'Boys'),
    COUNT(*) FILTER (WHERE gender = 'Girls')
  )::numeric / COUNT(*) < 0.95
ORDER BY null_count DESC
LIMIT 30;
```

**Purpose:** Finds divisions currently below the 95% sibling inference threshold but above 80%. Lowering threshold to 90% would catch these.

---

## Section 2 — Proposed Parsing Improvements

Based on known Parsing Rules gaps and GotSport API data patterns.

### Fix A — Lower Sibling Inference Threshold (90%)

**Current:** 95% of a division must have the same gender to infer the rest.  
**Proposed:** Lower to 90%.  
**Rationale:** At 90% confidence, the risk of misclassification is low and the gain in coverage is significant. A division where 45 of 50 games are "Boys" and 5 are null — those 5 are almost certainly also Boys.

**Implementation location:** `backfill-all-games.js`, sibling inference pass.

```javascript
// Change:
const SIBLING_INFERENCE_THRESHOLD = 0.95;
// To:
const SIBLING_INFERENCE_THRESHOLD = 0.90;
```

**Estimated impact:** Query 1D above will quantify this. Rough estimate: 2,000–4,000 nulls resolved.

### Fix B — Event-Level Gender Inference

**Proposed logic:** If 100% of a named event's games for a given age group across ALL divisions have the same gender, apply that gender to any remaining nulls in that event+age group.

**Use case:** An event named "2026 TX State Cup U14" where every single game is Boys — a few null-gender records in that event are overwhelmingly likely to also be Boys.

```sql
-- Identify events where ALL non-null games share one gender per age group
SELECT event_name, age_group, MAX(gender) AS inferred_gender
FROM games
WHERE gender IS NOT NULL AND is_hidden = false
GROUP BY event_name, age_group
HAVING COUNT(DISTINCT gender) = 1;
-- Use this result set to UPDATE games WHERE gender IS NULL for matching (event_name, age_group)
```

**Implementation location:** Add as a new pass in `final-backfill.js` or `backfill-all-games.js`.

**Estimated impact:** 1,000–3,000 nulls resolved (events where all divisions have uniform gender).

### Fix C — Team Name Pattern Expansion

**Current patterns:** "FC Girls U14", "Boys" keyword.  
**Proposed additions:**

```javascript
// Additional team name gender patterns (case-insensitive)
const TEAM_NAME_GENDER_PATTERNS = [
  { pattern: /\bboys?\b/i, gender: 'Boys' },
  { pattern: /\bgirls?\b/i, gender: 'Girls' },
  { pattern: /\bmale\b/i, gender: 'Boys' },
  { pattern: /\bfemale\b/i, gender: 'Girls' },
  // New additions:
  { pattern: /\bB(\d{2})\b/i, gender: 'Boys' },      // "B14 Division"
  { pattern: /\bG(\d{2})\b/i, gender: 'Girls' },     // "G14 Division"
  { pattern: /\b\d{2}B\b/i, gender: 'Boys' },         // "14B Elite"
  { pattern: /\b\d{2}G\b/i, gender: 'Girls' },        // "14G Elite"
  { pattern: /^(boys|girls)\s/i, gender: (m) => m[1].charAt(0).toUpperCase() + m[1].slice(1) },
  { pattern: /\s(boys|girls)$/i, gender: (m) => m[1].charAt(0).toUpperCase() + m[1].slice(1) },
];
```

**Estimated impact:** 500–1,500 nulls resolved.

### Fix D — Event Name Pattern Expansion

Add to Step 4 (`backfill-all-games.js`) event-name scanning for gender tokens:

```javascript
const EVENT_GENDER_PATTERNS = [
  /\bgirls?\s+(cup|league|classic|showcase|invitational|open|state|national)/i,
  /\bboys?\s+(cup|league|classic|showcase|invitational|open|state|national)/i,
  /(cup|league|classic|showcase|invitational)\s+girls?\b/i,
  /(cup|league|classic|showcase|invitational)\s+boys?\b/i,
  /\bwomen'?s?\b/i,          // "Women's League"
  /\bmen'?s?\b/i,            // not in U7-U19 scope but safe to skip
  /\becnl\s+girls?\b/i,      // ECNL Girls
  /\becnl\s+boys?\b/i,       // ECNL Boys (rare but exists)
  /\bnpsl\s+girls?\b/i,
  /\bnpl\s+girls?\b/i,
  /\bnpl\s+boys?\b/i,
];
```

---

## Section 3 — Backfill SQL (Apply After Code Fix)

After deploying parsing improvements, run this backfill to catch existing records:

```sql
-- Verify counts before/after
SELECT 
  COUNT(*) FILTER (WHERE gender IS NULL) AS null_before,
  COUNT(*) AS total,
  ROUND(COUNT(*) FILTER (WHERE gender IS NULL)::numeric / COUNT(*) * 100, 2) AS null_pct_before
FROM games WHERE is_hidden = false;

-- After running updated backfill scripts, re-run above to measure improvement
```

---

## Section 4 — Execution Plan

| Step | Action | Who | When |
|------|--------|-----|------|
| 1 | Run Section 1 queries — identify concentration | Presten | Next session |
| 2 | Share query results with FORGE | Presten | Same session |
| 3 | FORGE implements Fix A (lower threshold) | FORGE | On receipt |
| 4 | FORGE implements Fix B (event-level inference) | FORGE | On receipt |
| 5 | FORGE implements Fix C + D (name patterns) | FORGE | On receipt |
| 6 | Run pipeline Step 2–5 on existing data | Presten | After code deploy |
| 7 | Run Step 9 audit to measure new null rate | Presten | After backfill |
| 8 | FORGE updates Data Quality with actuals | FORGE | On receipt |

**Target outcome:** Null gender rate below 1.5% (from 3.3%). Fixes A+B alone should achieve ~2.0–2.5%; all four fixes together should reach the 1.5% target.

---

## Section 5 — What FORGE Needs from Presten

> **Blocker:** FORGE cannot measure actual null counts or test pattern changes without database and codebase access. Section 1 queries must be run by Presten. Results should be pasted to FORGE in the next session.

Priority order:
1. Run Query 1C (event concentration) — this determines whether event-level overrides are worth building
2. Run Query 1D (sibling inference gap) — quantifies Fix A impact
3. Run Query 1A (source breakdown) — confirms GotSport API is dominant source

---

## Related Notes
- [[Parsing Rules]] — current gender inference logic
- [[Data Quality]] — baseline metrics (3.3% null gender)
- [[Data Pipeline]] — Steps 2, 3, 4, 5 implement parsing
- `Infrastructure/FM2 Active — Code Change Specification.md` — related gender fix (GA ASPIRE)
