---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-27
status: completed
completed: 2026-04-26
priority: high
due: 2026-05-12
deliverable: "02 - Tiger Tournaments/Projects/Rankings/DSS Competitive Accuracy Narrative.md"
---

# Task: DSS Competitive Accuracy Narrative

## Context

The DSS demo is May 16–18. Evo Draw's primary competitive differentiation from USARank is ranking accuracy — specifically, that Evo Draw's ELO-based system is more accurate than USARank's simpler approach because it accounts for opponent strength, game recency, and event tier weighting.

This claim is stated in SENTINEL's configuration as a strategic priority ("Compare against competitors for accuracy and completeness") and has been referenced in prior ELO briefings in the context of the USARank comparison execution packages. What does not exist is a single, clean, non-technical document that *makes the accuracy case* for a decision-maker audience (Presten, a DSS evaluator, or a soccer director who is considering Evo Draw).

This task is not a spec, not a technical comparison, and not an execution package. It is a **1–2 page narrative** that answers the question: *why should someone trust Evo Draw's rankings more than USARank's?* ELO writes it. SENTINEL reviews it. Presten uses it.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/DSS Competitive Accuracy Narrative.md`

Target length: 400–700 words. No tables, no SQL, no footnotes. This is a document a human reads in 3 minutes.

---

### Section 1: The Core Claim

One paragraph (3–5 sentences). State directly what Evo Draw's ranking system does differently from simple win-loss-based rankings. Do not hedge. Example structure:

> "Evo Draw ranks teams using an ELO system calibrated to [N] leagues and [N] seasons of youth soccer data. Unlike systems that weight all wins equally, Evo Draw adjusts each game's contribution based on [opponent rating / event tier / recency]. The result: a team that wins a U16 Girls MLS NEXT game gains more rating points than a team that wins a recreational-level tournament game — which is the right answer. A system that cannot make this distinction cannot produce accurate rankings."

The exact wording is ELO's to determine. The requirement: it must be readable by a non-technical soccer director.

---

### Section 2: Where It Shows Up

Two or three specific examples of where ELO's approach produces a meaningfully different (and more accurate) result than a simpler ranking system would. These should be concrete, not abstract:

- A class of scenario where event tier weighting changes the outcome significantly (e.g., a strong team that plays in a low-tier regional circuit vs. a weaker team that plays in ECNL — simpler systems often rank them comparably; ELO separates them)
- The GA ASPIRE recalibration as a real example: ELO identified that GA ASPIRE was miscalibrated, corrected it, and produced a measurable improvement in Girls rankings accuracy. A static, uncalibrated system would not have caught this.
- One specific age group or region where Evo Draw's rankings differ noticeably from USARank and the Evo Draw result is more defensible (draw from the USARank comparison execution data if available by due date; if not, use the narrative from the comparison spec)

---

### Section 3: What Makes This Hard

One short paragraph explaining why ranking accuracy in youth soccer is harder than it looks, and why most systems get it wrong. This builds the case for why ELO's approach matters:

- League fragmentation (hundreds of leagues, no national authority)
- Platform fragmentation (GotSport, SincSports, Affinity, manual data entry)
- Age group transitions (teams change age groups; ratings need continuity logic)
- Gender-specific calibration requirements (Boys and Girls events are not the same calibration problem)

Keep this brief — the goal is "this is why it's hard" not "here is how we solved it." The solution is the previous sections.

---

### Section 4: The Calibration Commitment

One paragraph on ELO's ongoing commitment to calibration accuracy:

- How often calibration values are reviewed (tie to SENTINEL's weekly review cadence)
- The GA ASPIRE fix as a concrete example of catching and correcting a calibration error
- The team merges audit as a concrete example of maintaining team identity accuracy

This is not a promise of perfection — it is a claim of a systematic, ongoing quality process that USARank and similar platforms do not have.

---

## Quality Standard

- No jargon that requires a data science background to understand. If ELO uses "K-factor," "Brier score," or "ELO rating," it must be briefly explained in plain language.
- No hedging. This is a competitive claims document — passive voice and qualifiers ("may be more accurate," "tends to produce better results") undermine it.
- The GA ASPIRE fix should appear as a real example, not a hypothetical. It is the strongest available proof of Evo Draw's calibration discipline.
- The USARank comparison must be substantive. Either cite actual delta data (if available from the comparison execution packages), or cite the methodology difference directly. Do not just say "more accurate than USARank" without explaining how or by how much.

## Why This Matters

At DSS, Presten will be asked some version of "why are your rankings better?" SENTINEL does not currently have a clean answer to give. The ranking accuracy spec, the Brier score completion, the calibration monitoring — these are internal QA documents, not demo-ready answers. This narrative converts the internal quality work into a client-facing argument. ELO is the right author because ELO understands the mechanism. SENTINEL reviews for clarity and competitive accuracy. Presten delivers at DSS. If this document doesn't exist, the answer to "why better?" is either improvised or absent — both are worse than having it.

## References

- `Rankings/Calibration Values — League Hierarchy Reconciliation.md` — calibration accuracy foundation (complete this task first or in parallel)
- `Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md` — GA ASPIRE mechanism description
- `Rankings/Boys Calibration Status Summary — April 2026.md` — Boys calibration context
- Prior USARank comparison execution packages (any filed results by May 12)
- [[Competitors]] — competitive positioning context
