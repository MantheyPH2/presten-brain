---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-25
status: pending
priority: medium
due: 2026-05-10
deliverable: "02 - Tiger Tournaments/Projects/Product & Planning/DSS Demo — Feature Sequence Plan.md"
---

# Task: DSS Demo — Feature Sequence Plan

## Context

The DSS demo is May 16. ELO has built an extensive pre-demo infrastructure:
- `DSS Feature Readiness Tracker` — which features are authorized and when
- `DSS Ranking Accuracy Claims Spec` — what accuracy claims can be made
- `May 9 DSS Readiness Final Assessment Spec` — final readiness gate
- `DSS Demo Spec` — overall demo context and objectives
- `DSS Data Readiness Validation Plan` — data quality verification

What does not exist: a document that answers "what do we show, in what order, for how long, and what do we say at each step?" The readiness infrastructure confirms features are ready. The sequence plan defines the demo itself.

Without a sequence plan, the May 16 demo session starts with Presten opening psql (or the Evo Draw interface) and improvising the order. That is fine if the demo has one feature. With 6–8 potentially authorized features and a live audience, improvised sequencing produces a flat, meandering presentation. A structured 15-minute sequence with defined transitions and a narrative arc produces a demo that lands.

ELO is the right agent to write this because ELO understands which features are most compelling, which comparisons tell the best story, and how the calibration narrative should build across age groups.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Product & Planning/DSS Demo — Feature Sequence Plan.md`

This is a living document. ELO writes the first draft based on the current Feature Readiness Tracker. After May 9 go/no-go, ELO updates it to reflect the final authorized feature set. Presten uses the final version on May 16.

---

### Section 1: Demo Parameters

```
Target audience: DSS stakeholders (Presten's context — ELO fills based on available info)
Duration: 15 minutes (default) — confirm with Presten if different
Format: Live database + [UI / psql / both] — ELO notes which is assumed
Goal: [one sentence — what should the audience believe after this demo?]
```

ELO states the goal in terms of what the audience should believe, not what features will be shown. Example: "After the demo, the audience should believe Evo Draw's rankings are the most accurate youth soccer rankings in [region/country]." The goal shapes which features lead and which support.

---

### Section 2: Feature Inventory (Current Authorized Features)

Based on the current Feature Readiness Tracker, list all features that are currently authorized or conditionally authorized:

| Feature | Authorization Status | ELO Confidence | Demo Value (1–5) | Dependency |
|---------|---------------------|----------------|-----------------|-----------|
| Girls calibration (GA ASPIRE fix) | Pending April 29 | | | April 28 execution |
| Boys calibration (Option A) | Pending May 5 | | | Option A spot check |
| Event Strength Phase 1 | Pending April 29 G2 + stability | | | Girls cal auth |
| Rank Bands | Pending April 29 G3 | | | April 29 |
| Club Rankings (Girls) | Pending May 9 | | | Calibration stability |
| Club Rankings (Boys) | Conditional: pending May 5 | | | Option A result |
| ECNL Dedup (ecnl_verified) | Pending May 1 health | | | May 1 pipeline |
| Brier Score / Predictive accuracy | Conditionally authorized | | | Post-April-29 update |
| Game coverage narrative | Authorization TBD | | | May 1 pipeline |

ELO fills the "Demo Value" column now, before May 9, so the post-May-9 update is a fast filter (remove features that didn't clear, re-order the rest) rather than a full re-assessment.

Demo Value guide: 5 = best standalone story for a non-technical audience. 1 = interesting only to a technical audience. 3 = strong with context.

---

### Section 3: Proposed Sequence (Draft)

ELO proposes the sequence based on current feature inventory. The sequence should:
- Open with the feature that establishes Evo Draw's credibility fastest (highest Demo Value, lowest technical explanation required)
- Build to more technical features after credibility is established
- End with a "where this is going" forward-looking item (event strength, club rankings, or coverage narrative)

**Format for each sequence item:**

```
[N]. [Feature Name] — [2–3 min]
What to show: [specific query or UI view — be concrete]
What to say: [2–3 sentence script or key talking points]
Compelling comparison: [USA Rank comparison, year-over-year, or specific team example if applicable]
Transition to next item: [one sentence bridge]
```

ELO writes the full sequence for all currently conditionally-authorized features. Mark items that may be removed post-May-9 with `[CONDITIONAL — remove if not authorized]`.

---

### Section 4: Contingency Sequences

Two fallback sequences:

**Fallback A — Boys Option A fails (Girls-only demo):**
Reorder the Section 3 sequence removing all Boys features. Which feature moves up? What changes in the opening narrative?

**Fallback B — Club Rankings not ready (no May 9 auth):**
Remove Club Rankings from the sequence. Does anything fill the gap, or does the demo shorten to 12 minutes? What replaces the "where this is going" closing item?

Each fallback is 2–3 sentences describing the reordering decision, not a full rewrite. ELO updates this section after May 9 if either contingency fires.

---

### Section 5: Specific Examples and Reference Teams/Clubs

The 3–5 specific teams or clubs ELO recommends as demo subjects. These should be:
- Teams that a DSS audience would recognize or find impressive
- Teams with clear before/after stories (e.g., a GA ASPIRE team that moved significantly post-fix)
- Reference clubs from `Rankings/Club Rankings — Reference Clubs.md` if Club Rankings is authorized

| Entity | Why This One | Feature It Demonstrates | Age Group |
|--------|-------------|------------------------|-----------|
| | | | |

ELO recommends from its knowledge of the rankings. If Boys Option A is uncertain, recommend at least 3 Girls examples and note Boys examples as conditional.

---

### Section 6: May 9 Update Protocol

After the May 9 readiness assessment:
- ELO updates Section 2 Feature Inventory with final authorization statuses
- ELO updates Section 3 sequence: remove any CONDITIONAL items that did not clear
- ELO updates Section 5 with final reference teams (if Boys fails, remove Boys examples)
- ELO files the updated version as `DSS Demo — Feature Sequence Plan — Final.md` in the same folder
- ELO notes in the next briefing: "Final demo sequence plan filed. Features in sequence: [N]. Duration estimate: [N] minutes."

---

## Definition of Done

**Draft (due May 10):**
- All 6 sections present
- Feature Inventory (Section 2) filled with all current features and Demo Value ratings
- Full proposed sequence (Section 3) written for all conditionally-authorized features
- Both contingency sequences (Section 4) written
- At least 3 specific example teams/clubs in Section 5
- ELO summary in briefing: "Draft demo sequence plan filed. [N] features in sequence. Top Demo Value features: [list top 3]. May 9 update pending."

**Final (after May 9 readiness assessment):**
- CONDITIONAL items resolved
- Sequence reflects final authorized feature set
- Specific examples confirmed against post-April-28 rankings
- Filed as "Final" version before May 12

## Why This Matters

The DSS demo is the reason this entire preparation cycle exists. SENTINEL has spent the past 3 weeks reviewing readiness specs, monitoring specs, execution packages, and authorization gates — all in service of a 15-minute demonstration on May 16. Without a sequence plan, that 15 minutes risks being technically accurate but narratively weak. The sequence plan is ELO's contribution to making the demo land: the right features in the right order, with the right examples and the right talking points. It is the last document in the preparation chain and the one Presten most directly uses on the day.

## References

- `Product & Planning/DSS Feature Readiness Tracker — May 2026.md` — feature authorization status
- `Product & Planning/DSS Demo Spec.md` — overall demo context and objectives
- `Rankings/Club Rankings — Reference Clubs.md` — reference clubs for Section 5
- `Rankings/DSS Ranking Accuracy Claims Spec.md` — authorized accuracy claims for talking points
- `Rankings/Boys Option A Verdict Filing Document.md` — determines Boys features in Section 4 Fallback A
- `Rankings/May 9 DSS Readiness Final Assessment Spec.md` — triggers the May 9 update to this document
