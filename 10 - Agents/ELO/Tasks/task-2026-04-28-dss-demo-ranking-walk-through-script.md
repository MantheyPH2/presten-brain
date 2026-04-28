---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-28
status: completed
priority: medium
due: 2026-05-07
completed: 2026-04-28
deliverable: "02 - Tiger Tournaments/Projects/Product & Planning/DSS Demo — Ranking Walk-Through Script.md"
---

# Task: DSS Demo — Ranking Walk-Through Script

## Context

The DSS demo is May 16. ELO owns the ranking accuracy narrative — the claim that Evo Draw's rankings are better than competitors. The existing DSS demo spec covers the feature sequence. The DSS ranking accuracy claims spec covers what numbers to cite. What does not exist: an actual walk-through script that tells Presten, in demo-ready language, how to present the rankings to a non-technical audience.

The ranking system is the core competitive advantage. How it gets explained in 90 seconds determines whether the audience understands it or leaves confused. ELO is best positioned to write this script because ELO understands what the numbers mean, which examples are cleanest, and which claims are defensible.

This is not a sales document. It is a rehearsal guide — Presten walks through it before May 16 and knows exactly what to show, what to say, and what questions to expect.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Product & Planning/DSS Demo — Ranking Walk-Through Script.md`

---

### Section 1: Ranking Narrative (60 seconds)

The opening explanation of how Evo Draw ranks teams. Write this as spoken text — not bullets, but sentences Presten can say out loud.

**Purpose:** Answer "how does Evo Draw know which team is better?" in terms any soccer parent understands.

**Must cover:**
- Why game results beat standings (head-to-head across leagues)
- Why opponent quality matters (a win against MLS NEXT means more than a win against a local rec team)
- Why recency is factored (this season's form, not three years of results)

**Must avoid:**
- "ELO" (technical term; confuses non-technical audience)
- "K-factor" or "multiplier"
- Statistical jargon

**Length:** 60–90 seconds when read aloud. ELO times this.

---

### Section 2: The Demo Team (Pick One)

ELO selects a specific team to use as the anchor example in the demo. This team must:
- Be a Girls team (Girls calibration is confirmed; Boys is contingent on April 29)
- Be in a league the DSS audience will recognize (ECNL, MLS NEXT, or GA)
- Have a ranking that is notably different from their win-loss record (demonstrates the system's intelligence)
- Be unlikely to have a controversial ranking (no teams in a known dedup dispute)

**ELO states:** Team name, age group, current ranking (approximate), win-loss record, and why the contrast tells a clear story.

---

### Section 3: The 3-Step Show

Three things Presten does in the UI to demonstrate the ranking system. Each step = one action + one sentence.

**Step 1 — Show the team's ranking:**
What to click/search, what to show on screen, what to say.

**Step 2 — Show rank band:**
How rank bands are displayed for this team, what the band means in plain language.

**Step 3 — Show top teams in their age group:**
How to navigate to the top-10 or top-20 list, one observation Presten makes ("notice that the top-ranked team in Girls U16 has played 4 MLS NEXT opponents this season — the system rewards that").

---

### Section 4: The USARank Comparison Line

One sentence Presten says about how Evo Draw compares to USARank. This must be:
- Factual (based on the USARank delta analysis ELO completed)
- Specific (cite a percentage or a concrete discrepancy, not "we are more accurate")
- Non-disparaging (do not name USARank negatively — "Evo Draw accounts for opponent quality; rankings that use only win-loss records cannot")

ELO writes this line and cites the backing analysis document.

---

### Section 5: Expected Questions and Prepared Answers

Five questions the audience is likely to ask, with the answer Presten should give.

| Question | Answer |
|----------|--------|
| "Why is my daughter's team ranked lower than I expected?" | |
| "How often do rankings update?" | |
| "Does this include club rankings?" | |
| "What about boys teams?" | |
| "Is this better than MaxPreps / USARank?" | |

ELO writes the answers. Answers must be ≤ 3 sentences. No hedging — specific and confident.

---

### Section 6: What NOT to Demo

Things ELO flags as demo risks — features or examples Presten should avoid in the ranking section:
- Teams in known dedup dispute (list by age group if applicable)
- Age groups where Boys calibration is uncertain (Boys U-groups if Option A not confirmed)
- Any ranking that looks anomalous and could undermine credibility
- The team_merges number (27,820) — too easy to misinterpret as errors

---

## Definition of Done

- Script filed at deliverable path by May 7
- Section 1 narrative is 60–90 seconds when read aloud (ELO confirms this by timing)
- Section 2 demo team is a real, named team with actual ranking and win-loss data
- Section 5 answers are ≤ 3 sentences each, confident, no hedging
- Section 6 contains at least 3 specific avoid items
- SENTINEL review: SENTINEL reads this before May 9 readiness check and flags any claims that are not supported by confirmed data

## Why This Matters

The DSS Feature Readiness Tracker and the Demo Spec define what features are demo-ready. They do not tell Presten how to present the rankings in a way that lands. A technically correct ranking system presented confusingly will not move the DSS audience. This script ensures the strongest feature gets its strongest possible showing.

## References

- `task-2026-04-25-dss-demo-spec.md` — overall demo sequence
- `task-2026-04-25-dss-ranking-accuracy-claims-spec.md` — accuracy claims ELO can make
- `task-2026-04-25-dss-demo-feature-sequence-plan.md` — feature order in demo
- `task-2026-04-23-rank-bands-design.md` — rank bands system ELO presents
- `task-2026-04-23-usarank-comparison.md` — comparison basis for Section 4
- `task-2026-04-26-may9-readiness-final-assessment-spec.md` — readiness gate this feeds
