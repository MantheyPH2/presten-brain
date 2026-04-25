---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-25
status: pending
priority: medium
due: 2026-05-09
deliverable: "02 - Tiger Tournaments/Projects/Rankings/DSS Ranking Accuracy Claims — Authorized Spec.md"
---

# Task: DSS Ranking Accuracy Claims — Authorized Spec

## Context

At DSS, Presten will present Evo Draw rankings to a live audience. The audience will ask variants of: "How accurate are these rankings?" and "How do you know they're right?"

ELO has produced substantial supporting material:
- `Rankings/USA Rank Ranking Accuracy Audit — April 2026.md` — cross-validation against USA Rank
- `Rankings/USA Rank Delta Interpretation Spec.md` — methodology for interpreting divergence
- `Rankings/Brier Score Completion.md` — predictive accuracy baseline
- ELO's Pre-Run Model and Boys Option A spot check — calibration quality evidence

What does not exist: a document where ELO states, plainly, "here are the accuracy claims we authorize Presten to make, here is the evidence backing each claim, and here is what we do NOT authorize." This is not a methodology document — it is ELO's signed authorization of specific demo talking points.

Without it, Presten faces one of two bad outcomes at DSS: (a) underclaims ("we haven't validated our rankings rigorously") when the data supports a stronger claim, or (b) overclaims ("our rankings are the most accurate in youth soccer") when ELO has not verified that at the claimed scope.

This task closes that gap.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/DSS Ranking Accuracy Claims — Authorized Spec.md`

---

### Section 1: Authorized Claims — Girls Rankings

For each claim, ELO states: the exact claim text Presten may use, the evidence supporting it, and the confidence level (high / medium / conditional).

ELO should produce at minimum 3 authorized Girls ranking claims, drawing from:
- USA Rank cross-validation results (what % of ranked teams agree within N rating points?)
- Brier score (what is Evo Draw's predictive accuracy for Girls matches?)
- Coverage scope (what % of Girls teams are ranked at all — with what data recency?)

Example format:

> **Claim:** "Evo Draw's Girls rankings show [X]% agreement with USA Rank for the top 200 teams per age group."
> **Evidence:** USA Rank accuracy audit, [date], comparison methodology per Delta Interpretation Spec.
> **Confidence:** High / Medium / Conditional on [specific condition].
> **Authorized for DSS:** ✅ Yes / ⚠️ With caveat: [caveat] / ❌ No — do not use.

Do NOT write claims ELO cannot back. If the USA Rank audit shows only partial coverage, state the coverage scope explicitly in the claim.

---

### Section 2: Authorized Claims — Boys Rankings

ELO notes the status of Boys calibration at time of writing:
- Boys Option A has not yet run (window opens April 29)
- Boys calibration is considered preliminary pending Option A results

ELO produces Boys claims at two confidence levels:

**Pre-Option-A claims** (can be stated regardless of Boys spot check outcome):
- Claims backed by existing data, not dependent on Option A results

**Option-A-dependent claims** (authorized only if Boys Option A PASSES):
- Claims that require Boys calibration validation to make responsibly

At DSS (May 16), ELO will have Boys Option A results. Section 2 should pre-write both versions so SENTINEL knows which to use depending on outcome.

---

### Section 3: Predictive Accuracy Claim

Evo Draw's Glicko-2 system makes predictions. What is the authorized claim about prediction accuracy?

Pull from Brier Score Completion work. State:
- The Brier score achieved (or best available metric)
- The comparison baseline (what does a random-50% predictor score? What does USA Rank's implied ranking score?)
- The exact claim text authorized for DSS

If the Brier score work did not produce a publishable number (e.g., insufficient hold-out data), state that explicitly and note: "Predictive accuracy claim is NOT authorized for DSS — insufficient validation data."

---

### Section 4: Coverage Claims

Accuracy is only meaningful within the covered scope. Authorized coverage claims:

- What % of active teams in covered leagues have a ranking?
- What is the minimum game history for a stable ranking? (i.e., what does ELO consider a "trustworthy" ranking vs. a provisional one?)
- How should Presten describe teams with fewer than N games? ("preliminary ranking" vs. "not yet ranked")

These feed directly into the demo narrative when Presten shows a team's rank card.

---

### Section 5: What NOT to Claim

Explicit list of claims ELO does NOT authorize for DSS:
- Claims that exceed the validation evidence
- Claims about leagues with incomplete or unvalidated coverage
- Claims about Boys rankings if Option A results are borderline

For each: one sentence explaining why it is not authorized and what evidence gap prevents it. This protects Presten from improvising a claim on-stage that ELO has not validated.

---

### Section 6: SENTINEL Sign-Off Block

```
ELO authorization: I confirm the claims in Sections 1–4 are backed by the cited evidence
                   and are accurate as of the date of this document.
ELO signature: ELO — [date]

SENTINEL review: [ ] Claims reviewed. No unsupported claims found.
                 [ ] Section 5 prohibitions communicated to FORGE/Presten.
SENTINEL sign-off: SENTINEL — [date]
```

---

## Quality Standard

- Every authorized claim must have: exact text, evidence citation, confidence level.
- No claims in Sections 1–4 that are not backed by specific cited documents — not general reasoning.
- Section 5 must name at least 3 claims that are NOT authorized and why.
- The claims must be written in the voice Presten would actually use at DSS — not academic language.

## Why This Matters

DSS is a high-stakes product presentation. If Presten overclaims on accuracy and an audience member challenges it with data (e.g., "your rankings show Team X as top 10 but they went 3-12 this season"), the lack of a signed authorization spec means there is no documented basis for the claim. With this spec, SENTINEL has reviewed and authorized every accuracy claim before DSS. Any challenge can be answered: "Our accuracy is backed by [specific cross-validation methodology] covering [specific scope]." That is a defensible position. An improvised claim is not.

## References

- `Rankings/USA Rank Ranking Accuracy Audit — April 2026.md` — primary cross-validation evidence
- `Rankings/USA Rank Delta Interpretation Spec.md` — divergence methodology
- `Rankings/Brier Score Completion.md` — predictive accuracy baseline
- `Rankings/Boys Calibration Status Summary — April 2026.md` — Boys calibration context
- `Rankings/Boys Option A — Results Interpretation Guide.md` — Boys verdict that feeds Section 2
- `Product & Planning/DSS Demo Spec.md` — demo script this spec informs
