---
title: Null Gender Audit — Code Implementation Specs
type: forge-implementation
author: FORGE
date: 2026-04-29
status: ready-to-deploy
topic: null-gender-fix-code-specs
tags: [forge, data-quality, gender, parsing, implementation, code-ready]
depends_on: "Infrastructure/Null Gender Audit — Investigation Package.md"
---

# Null Gender Audit — Code Implementation Specs

> **Purpose:** Exact code changes for Fixes A–D. When Presten shares Section 1 query results, FORGE deploys within 30 minutes — no mid-session spec work required.
>
> **Fixes C and D have no query dependencies. FORGE can deploy them immediately.**

---

## Section 1 — Fix A: Lower Sibling Inference Threshold

**File:** `backfill-all-games.js`
**Change type:** Single constant — 1 line

### Current code
```javascript
const SIBLING_INFERENCE_THRESHOLD = 0.95;
```

### Proposed change
```javascript
const SIBLING_INFERENCE_THRESHOLD = 0.90;
```

**Context:** This constant is used in the sibling inference pass — the block that checks whether a given division (event + age group) has enough co-siblings with a known gender to infer the remainder. At 95%, a division with 19 Boys and 1 null doesn't qualify. At 90%, it does.

**Risk assessment:** Lowering from 95% to 90% introduces a narrow false-positive window (a true mixed-gender division at 91%+ of one gender would be misclassified). In practice, US youth soccer divisions are gender-segregated — a mixed-gender division at 90–94% is almost always a data artifact (one mislabeled record), not a genuine mixed division. FORGE assesses risk as **low**.

**Pre-condition:** Run Query 1D from the Investigation Package to quantify how many nulls fall in the 80–95% band. If < 500 records are affected, this change is safe to deploy without further review.

**Estimated impact:** 2,000–4,000 nulls resolved.

---

## Section 2 — Fix B: Event-Level Gender Inference

**File:** `backfill-all-games.js` (new pass, add after sibling inference pass)
**Change type:** New inference block — ~25 lines

### New code block

```javascript
// Fix B — Event-level gender inference
// For each (event_name, age_group) combination where 100% of games with known gender
// share one gender, apply that gender to null-gender games in the same group.
async function applyEventLevelGenderInference(db) {
  const eligibleGroups = await db.query(`
    SELECT event_name, age_group, MAX(gender) AS inferred_gender
    FROM games
    WHERE gender IS NOT NULL AND is_hidden = false
    GROUP BY event_name, age_group
    HAVING COUNT(DISTINCT gender) = 1
  `);

  let totalUpdated = 0;
  for (const row of eligibleGroups.rows) {
    const result = await db.query(`
      UPDATE games
      SET gender = $1
      WHERE event_name = $2
        AND age_group = $3
        AND gender IS NULL
        AND is_hidden = false
    `, [row.inferred_gender, row.event_name, row.age_group]);
    totalUpdated += result.rowCount;
  }

  console.log(`[Fix B] Event-level inference: ${totalUpdated} records updated`);
  return totalUpdated;
}
```

**Execution order:** Run **after** Fix A (sibling inference). Fix A resolves divisional nulls; Fix B resolves event-level nulls where sibling inference didn't apply.

**Where to call it:** In the main backfill execution sequence, after the existing sibling inference block completes.

**Risk assessment:** Only applies when 100% of an event+age_group combination shares a single gender — this is effectively zero-risk. An event called "2026 TX State Cup U14 Boys" where every non-null game is Boys is definitionally a Boys event.

**Pre-condition:** Run Query 1C from the Investigation Package first. If the top events are clearly gendered (e.g., state cups, ECNL RL, NPL), Fix B will have high impact. If nulls are scattered across hundreds of events with mixed gender, impact will be lower.

**Estimated impact:** 1,000–3,000 nulls resolved.

---

## Section 3 — Fix C: Expanded Team Name Patterns

**File:** `backfill-all-games.js` (or wherever `TEAM_NAME_GENDER_PATTERNS` is defined)
**Change type:** Array expansion — additive only, zero regression risk

### Current pattern array (approximate — verify line numbers in codebase)
```javascript
const TEAM_NAME_GENDER_PATTERNS = [
  { pattern: /\bboys?\b/i, gender: 'Boys' },
  { pattern: /\bgirls?\b/i, gender: 'Girls' },
  { pattern: /\bmale\b/i, gender: 'Boys' },
  { pattern: /\bfemale\b/i, gender: 'Girls' },
];
```

### Proposed additions
```javascript
const TEAM_NAME_GENDER_PATTERNS = [
  { pattern: /\bboys?\b/i, gender: 'Boys' },
  { pattern: /\bgirls?\b/i, gender: 'Girls' },
  { pattern: /\bmale\b/i, gender: 'Boys' },
  { pattern: /\bfemale\b/i, gender: 'Girls' },
  // New additions — common GotSport team name formats:
  { pattern: /\bB(\d{1,2})\b/, gender: 'Boys' },    // "B14 Division", "B9"
  { pattern: /\bG(\d{1,2})\b/, gender: 'Girls' },   // "G14 Division", "G9"
  { pattern: /\b(\d{1,2})B\b/, gender: 'Boys' },    // "14B Elite", "09B"
  { pattern: /\b(\d{1,2})G\b/, gender: 'Girls' },   // "14G Elite", "09G"
];
```

**Note on case sensitivity:** The `B(\d)` and `G(\d)` patterns do not use `/i` — this is intentional. Lowercase `b14` does not commonly indicate gender in GotSport team names; uppercase `B14` and `G14` are the standard encoded format. Using case-insensitive matching here would risk false positives on team names containing lowercase `b` or `g` followed by numbers in non-gender contexts.

**No pre-condition required.** This is purely additive — existing patterns are unchanged, new patterns only apply where current patterns fail to match.

**Estimated impact:** 500–1,500 nulls resolved.

---

## Section 4 — Fix D: Expanded Event Name Patterns

**File:** `backfill-all-games.js` (Step 4 event-name scanning block)
**Change type:** Array expansion — additive only

### Current event patterns (approximate — verify in codebase)
```javascript
// Existing patterns likely capture "boys" and "girls" as standalone words
```

### Proposed additions
```javascript
const EVENT_GENDER_PATTERNS = [
  // Existing (preserve these):
  /\bgirls?\b/i,
  /\bboys?\b/i,
  // New additions:
  /\bgirls?\s+(cup|league|classic|showcase|invitational|open|state|national|premier)/i,
  /\bboys?\s+(cup|league|classic|showcase|invitational|open|state|national|premier)/i,
  /(cup|league|classic|showcase|invitational|open|state|national|premier)\s+girls?\b/i,
  /(cup|league|classic|showcase|invitational|open|state|national|premier)\s+boys?\b/i,
  /\bwomen'?s?\b/i,
  /\becnl\s+girls?\b/i,
  /\becnl\s+boys?\b/i,
  /\bnpl\s+girls?\b/i,
  /\bnpl\s+boys?\b/i,
  /\bga\s+girls?\b/i,
  /\bga\s+boys?\b/i,
];
```

**Application logic:** When an event name matches one of these patterns, extract the gender signal and apply to all null-gender games in that event that also lack a team-name gender signal. Do not override existing non-null gender values.

**No pre-condition required.** Additive — cannot reduce gender coverage.

**Estimated impact:** 300–800 nulls resolved (events not caught by existing patterns).

---

## Section 5 — Deployment Order and Backfill Plan

### Recommended deployment sequence

| Order | Fix | Pre-condition | Reason |
|-------|-----|--------------|--------|
| **1** | Fix C — Team name patterns | None | Purely additive; zero risk; runs in seconds |
| **2** | Fix D — Event name patterns | None | Purely additive; zero risk; runs in seconds |
| **3** | Fix A — Sibling threshold | Query 1D result (confirm ≥500 candidates in 80–95% band) | Low risk; quantify before deploying |
| **4** | Fix B — Event-level inference | Query 1C result (confirm events are single-gender) | Zero risk but higher impact variability; run last to catch anything remaining |

**Rationale for C+D first:** They require no query results, cannot cause false positives, and can be deployed in the current session. Deploying C+D now means the backfill pass before Presten shares query results will already incorporate those fixes — reducing the null count before A+B run.

### Post-deploy backfill SQL

Run after all fixes are deployed to apply to existing records:

```sql
-- Measure before
SELECT
  COUNT(*) FILTER (WHERE gender IS NULL) AS null_before,
  COUNT(*) AS total,
  ROUND(COUNT(*) FILTER (WHERE gender IS NULL)::numeric / COUNT(*) * 100, 2) AS null_pct_before
FROM games WHERE is_hidden = false;

-- [Deploy code changes and run backfill-all-games.js]

-- Measure after
SELECT
  COUNT(*) FILTER (WHERE gender IS NULL) AS null_after,
  COUNT(*) AS total,
  ROUND(COUNT(*) FILTER (WHERE gender IS NULL)::numeric / COUNT(*) * 100, 2) AS null_pct_after
FROM games WHERE is_hidden = false;
```

Target: null_pct_after ≤ 1.5% (from current 3.3%).

---

## Section 6 — Deployment Decision Gate

| Fix | Pre-condition | FORGE can deploy? | Presten auth needed? |
|-----|--------------|-------------------|---------------------|
| Fix C — Team name patterns | None | **Yes — immediately** | No |
| Fix D — Event name patterns | None | **Yes — immediately** | No |
| Fix A — Sibling threshold 90% | Query 1D result | Yes, after count review (if ≥500 candidates) | No |
| Fix B — Event-level inference | Query 1C result | Yes, if ≥50% of nulls in eligible single-gender groups | No |

**If Presten does not share query results before May 1:** FORGE deploys C+D immediately and holds A+B. C+D alone are estimated to resolve 800–2,300 nulls (null rate from 3.3% → ~3.0%). Full deployment of all four fixes awaits query results.

---

## References

- `Infrastructure/Null Gender Audit — Investigation Package.md` — source queries and fix descriptions
- [[Parsing Rules]] — existing gender inference logic
- [[Data Quality]] — current null rate baseline (3.3% / ~12,000 nulls)
- [[Data Pipeline]] — Steps 2–5 implement parsing; Step 9 measures quality

*FORGE — 2026-04-29*
