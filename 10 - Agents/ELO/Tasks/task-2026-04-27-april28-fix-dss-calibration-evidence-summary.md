---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-27
status: completed
completed: 2026-04-27
priority: medium
due: 2026-05-02
deliverable: "02 - Tiger Tournaments/Projects/Rankings/April 28 Fix — DSS Calibration Evidence Summary.md"
---

# Task: April 28 Fix — DSS Calibration Evidence Summary

## Context

After April 29 gate results are filed, ELO will have a complete record of what the April 28 GA ASPIRE fix produced: pre-run predictions, gate results, and a synthesis brief. All of these are technical documents written for ELO and SENTINEL.

None of them are written for Presten to use in the DSS demo or in a conversation with a stakeholder. At DSS (May 16), if someone asks "have you verified your calibration changes worked?", Presten needs a plain-English answer with concrete evidence behind it — not a reference to technical gate documents.

This task: write the non-technical DSS-ready evidence summary that ELO can confidently stand behind and Presten can use in the demo or in pre-demo stakeholder conversations.

**Dependency:** This document is written after April 29 gate results are complete. File by May 2 (allowing time for review before May 9 DSS readiness assessment). If April 29 gates have not run by May 2, file a partial document with what is known and a note that it will be completed when April 29 data is available.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/April 28 Fix — DSS Calibration Evidence Summary.md`

---

### Section 1: What Was Fixed and Why

Two paragraphs, non-technical:

**Paragraph 1 — The problem:**
Describe the GA ASPIRE miscalibration in terms a parent or coach understands. Avoid "event_tier," "K-factor," "cal value." Use: "Girls teams playing in Georgia ASPIRE league events were being compared against a stronger benchmark than their actual competition level," or similar. State how long the issue existed, approximately how many teams were affected, and what the visible effect was (teams appeared ranked higher than their actual competitive tier).

**Paragraph 2 — The fix:**
Describe what was changed, in the same plain language. Avoid "UPDATE SET event_tier." Use: "We corrected the league classification for Georgia ASPIRE events so they are now benchmarked against leagues at the appropriate competitive tier." State when the fix was deployed (April 28).

---

### Section 2: Evidence the Fix Worked

Fill this section after April 29 gate results are available.

**Evidence Item 1: Rating shift direction and magnitude**
State, in one sentence, what ELO predicted would happen to Girls GA ASPIRE team ratings and what actually happened:

> "We predicted Girls GA ASPIRE teams would see their ratings adjust by [X–Y] points after reclassification. The actual adjustment was [Z] points — within our predicted range / slightly outside our predicted range (within acceptable bounds)."

**Evidence Item 2: No unintended effects on other teams**
State that Boys ratings did not change (expected) and that Girls ECNL/MLS NEXT ratings remained stable (expected):

> "Boys rankings were not affected, as expected — Boys teams do not play GA ASPIRE events. Girls ECNL and MLS NEXT team ratings shifted by less than [N] points, confirming the fix was scoped to GA ASPIRE events only."

**Evidence Item 3: The fix is permanent and forward-looking**
State that new games played in GA ASPIRE events will now be classified correctly from the point of ingestion — the fix is not just retroactive.

---

### Section 3: What This Means for Ranking Accuracy

One paragraph connecting the fix to the DSS ranking accuracy narrative.

The fix improved accuracy in a specific, measurable way. ELO states:
- Which age groups saw the most improvement (those with the highest GA ASPIRE game share)
- Whether the fix closes or reduces the gap in the USA Rank comparison (reference the USA Rank delta work if applicable)
- The calibration confidence ELO now has for Girls GA-heavy regions (Georgia, nearby states)

This section should allow Presten to say: "We found an issue, we fixed it, and we can show you the before/after effect on the rankings." That is the accuracy claim the DSS audience needs to hear.

---

### Section 4: Remaining Calibration Work (Honest Disclosure)

ELO states, briefly, what calibration work is still in progress. This section exists to prevent Presten from over-claiming at DSS.

Include:
- Boys calibration: spot check in progress (results May 5). Girls rankings are stronger confidence than Boys at this time. (If Boys Option A PASS is received before May 2, update this section.)
- U13/U14 calibration fix: scheduled post-DSS (May 17+). Minor accuracy improvement in younger age groups.
- State explicitly: "The GA ASPIRE fix is the most material accuracy improvement in this release cycle. All other calibration work is either already validated (Girls national leagues) or scheduled for post-DSS."

---

## Quality Standard

- Sections 1 and 3 must be written in Presten's voice — present tense, first person plural ("we corrected," "we predicted"). Not passive voice ("the fix was applied").
- Section 2 must include actual numbers from the April 29 gate results. Do not file this document before those results are available (or explicitly note the gap).
- Section 4 must be honest — do not omit known calibration work in progress. An honest disclosure in Section 4 is more persuasive at DSS than a claim of complete calibration.
- This document should stand alone. Presten should be able to hand it to a skeptical stakeholder and have it answer the question "how do you know your rankings are accurate?"

## Why This Matters

The technical gate documents answer "did the fix run?" and "are the numbers within expected range?" This document answers "what does this mean for the product?" Those are different questions, and the second one is the one that matters at DSS. ELO is the only agent that can write this document credibly — it owns the calibration system and its behavior. SENTINEL cannot synthesize the technical evidence into a non-technical claim with ELO's authority behind it. Writing this May 2 (before the May 9 readiness assessment) gives SENTINEL time to review it, ask ELO to sharpen anything unclear, and present it to Presten as pre-DSS reading.

## Definition of Done

- File at deliverable path by May 2
- Sections 1–4 present
- Section 2 filled with actual April 29 gate result numbers (or partial document noted)
- No jargon in Sections 1, 2, 3 that a parent or coach wouldn't understand
- Section 4 does not omit any known open calibration items
- ELO briefing note: "April 28 Fix DSS Evidence Summary filed. Core claim: [1 sentence summary of Section 3]. Open items disclosed: Boys spot check, U13/U14 post-DSS."

## References

- `Rankings/April 29 — Gate Results Synthesis Brief.md` — the technical source for Section 2 numbers
- `Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md` — the prediction ELO compares against in Section 2
- `Rankings/DSS Demo Talking Points — Rankings Features.md` — companion document for demo delivery (Section 5 of that doc handles the accuracy question)
- `Rankings/DSS Ranking Accuracy Claims Spec.md` — the accuracy claims framework this document contributes to
