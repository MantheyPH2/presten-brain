---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-27
status: completed
priority: high
due: 2026-05-07
deliverable: "02 - Tiger Tournaments/Projects/Rankings/DSS Demo Talking Points — Rankings Features.md"
---

# Task: DSS Demo Talking Points — Rankings Features

## Context

Multiple documents cover the technical side of the DSS demo: the DSS Demo Spec, DSS Feature Readiness Tracker, and Ranking Accuracy Claims Spec. None of them contain what Presten actually says during the demo.

The DSS audience is decision-makers, not engineers. They will ask questions like "Why should I trust these rankings over USA Rank?" and "What does rank band 2 mean for a recruiting coach?" Presten cannot answer those questions by reading from a spec. ELO — the system that computes the rankings — is the right author for these talking points, because ELO understands the "why" behind calibration, rank bands, and event strength.

This task: write the rehearsal-ready talking points Presten uses at DSS for ELO-owned features.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/DSS Demo Talking Points — Rankings Features.md`

---

### Section 1: The Core Claim (30-Second Version)

One paragraph Presten can say to open the rankings section of the demo. Should convey:
- What Evo Draw rankings measure (performance relative to competition level, not just wins)
- Why calibration matters (a win against a weak team is worth less than a win against a top team)
- The key differentiator from USA Rank and GotSoccer (ELO states this based on the USA Rank comparison work)

Write it in plain English, for a non-technical audience. No jargon. Avoid "Elo rating system" unless explained in one clause.

---

### Section 2: Rank Bands — What to Show and What to Say

The demo will show rank bands (per `Rankings/DSS Ranking Accuracy Claims Spec.md` and the Rank Bands Implementation Plan).

For each of the following talking points, write 2–3 sentences Presten says during the demo:

**TP-1: What rank bands are**
Explain what a rank band is (the range a team's true strength falls in, accounting for uncertainty). Why a band is more honest than a single number for teams with few games.

**TP-2: How bands narrow over time**
Show (or describe) how a team's band narrows as they play more games. Frame this as a feature, not a limitation.

**TP-3: The recruiting use case**
A recruiting coach looking at a U16G team with a tight, high band vs. a team with a wide, uncertain band. What should the coach conclude? Give Presten the language to explain this in 2–3 sentences without prompting.

---

### Section 3: Team Comparison — Demo Script

The demo will compare two specific teams (ELO selects the demo examples based on rank bands and event strength data). Write the script for this comparison:

**Comparison pair:** ELO selects two Girls teams with contrasting rank bands in the same age group. Name the teams (or use placeholder names if real team names are unavailable at writing time).

**Script structure:**
1. "Let's look at [Team A] and [Team B] — both U[N]G teams in [region]."
2. "[Team A]'s rank: [N]. Band: [N–N]. Here's what that means..."
3. "[Team B]'s rank: [N]. Band: [N–N]. Here's what's different..."
4. "If you're a coach evaluating these two teams, here's what Evo Draw tells you that USA Rank can't..."

Fill in what ELO knows. If specific team data isn't available at writing time, write the script with `[Team A placeholder]` and note what data to fill in during April 29 or May 9 sessions.

---

### Section 4: Event Strength — One-Liner

Event strength is in the demo (Phase 1, per authorization criteria). Write one sentence Presten says to explain why event strength matters, followed by one example. Audience is a parent or coach, not a statistician.

Example structure: "Not all wins are equal. A team that beats [top club] at [ECNL event] is playing harder competition than a team with the same record at a local league — and our system knows the difference."

ELO writes the actual version using real data from the League Hierarchy.

---

### Section 5: Handling the "Accuracy" Question

The most likely skeptical question at DSS: "How do you know these rankings are accurate?"

Write a 4–5 sentence answer Presten can give that:
1. Doesn't over-claim ("We're always right")
2. References the USA Rank comparison (N teams checked, % agreement)
3. Explains the calibration approach at a one-sentence level
4. Closes with why the calibration is improving (April 28 GA ASPIRE fix, boys calibration in progress)

This is the most important talking point in the document. Write it as if Presten is answering a skeptic, not reciting a fact sheet.

---

### Section 6: What NOT to Say

Three things Presten should avoid saying at DSS, with brief reasons:

1. Anything that implies rankings are final/authoritative — ELO flags why this undermines trust rather than building it
2. Specific numbers that may change between now and May 16 (ELO lists which numbers are stable vs. volatile)
3. Claims about Boys calibration until Option A is confirmed — note what to say instead if Boys rankings come up

---

## Quality Standard

- Every talking point must be written in Presten's voice — present tense, direct, conversational. Not "The system calculates..." but "We weight each win based on..."
- Section 5 (accuracy question) must be a complete, rehearsable answer — not bullet points.
- Demo examples in Section 3 must use real (or realistic placeholder) data, not abstract descriptions.
- The document should be usable as a rehearsal script: Presten reads it, practices it, and walks into the demo room with it memorized.

## Definition of Done

- File written at the deliverable path
- Sections 1–6 all present
- Section 3 has a named comparison pair (or named placeholders with data-fill instructions)
- Section 5 is a full prose answer, not bullets
- Section 6 has exactly 3 items
- FORGE briefing summary: "DSS Demo Talking Points written. Core claim drafted. Accuracy answer ready. Comparison pair: [names]. Flags for Presten: [list from Section 6]."

## References

- `Rankings/DSS Ranking Accuracy Claims Spec.md` — accuracy claims that inform Section 5
- `Rankings/Rank Bands — Implementation Plan.md` — rank bands feature that informs Section 2
- `Rankings/USARank Delta Interpretation Spec.md` — USA Rank comparison data for Section 5
- `Rankings/DSS Competitive Accuracy Narrative.md` — filed April 26, context for competitive comparison
- `Rankings/Event Strength — Phase 1 Implementation Plan.md` — event strength feature for Section 4
- `Product & Planning/DSS Feature Readiness Tracker — May 2026.md` — which features are confirmed in demo
