---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-25
status: completed
priority: medium
due: 2026-05-07
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/Evo Draw — DSS Coverage Narrative.md"
---

# Task: Evo Draw — DSS Coverage Narrative

## Context

The Pre-DSS Source Coverage Report (`Infrastructure/Pre-DSS Source Coverage Report — May 2026.md`) is a technical inventory: 17 leagues/sources, Active/Pending/Research/Not Covered tiers, Option A and Option B claims. It is an excellent internal reference for SENTINEL. It is not a document Presten can speak from during a DSS presentation.

What is missing: a one-page narrative translation of the coverage report for a non-technical audience. DSS is a decision-maker audience. They will ask: "How many teams are in here? What leagues? How current is it?" Presten needs crisp, defensible answers — not 17 rows of a coverage matrix.

This document converts FORGE's technical coverage assessment into Presten's talking track.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/Evo Draw — DSS Coverage Narrative.md`

Single-page document. Five sections, each 1–3 sentences or a short list.

---

### Section 1: What Evo Draw Covers (Option B Claim)

Pull from the Source Coverage Report Option B definition. Write 2–3 sentences Presten can say out loud:

> "Evo Draw currently includes rankings for [N] leagues across [N] age groups, covering [state/national scope]. Data is updated [daily / weekly / on-pipeline-run]. The platform tracks teams from the national level down through state association play."

Fill in the bracketed values from the coverage report. Use Option B if the browser session and Stage 1 crawl have completed; use Option A otherwise. Mark clearly which claim this narrative uses.

**Claim label:** `Option B (pending browser session + Stage 1)` OR `Option A (confirmed as of April 25)` — FORGE marks which is current when writing.

---

### Section 2: Key Leagues Covered

A bulleted list of the top-tier leagues that would resonate with a DSS soccer-knowledgeable audience. Not all 17. The 6–8 that matter:

- e.g., ECNL Girls / ECNL Boys (if applicable)
- MLS NEXT
- GA (Girls Academy)
- [State-level: USL Academy, EDP, USYS affiliates — include if confirmed by browser session]

One line per league: name + one-phrase description (e.g., "ECNL — premier national club league, Girls and Boys").

Do not include leagues that are Pending or Research Required unless they are confirmed by the time this document is written. Mark any conditional entries with `[pending browser session confirmation]`.

---

### Section 3: Coverage Gaps (Honest)

What Evo Draw does NOT include, stated simply and without apology. 2–4 bullets.

Example framing: "Evo Draw is focused on competitive travel soccer. It does not include rec leagues, high school, or [specific gaps from the report]."

SENTINEL expects honesty here — overstating coverage at DSS will create credibility problems later. Name the real gaps.

---

### Section 4: Coverage Growth (What's Coming)

Pull from the Stage 2 expansion plan and the Source Coverage Report Pending tier. 2–3 sentences on the roadmap:

> "Following the May 1 pipeline deployment, Evo Draw will expand to include [Stage 2 leagues/regions]. By [date], coverage will grow to [N] state associations and [additional leagues]."

Use specific numbers and dates where available. Do not promise timelines FORGE is not confident in.

---

### Section 5: Data Freshness

One paragraph: how often does Evo Draw's data update? What is the lag between a game being played and it appearing in rankings?

Pull from the daily pipeline design documents. Be specific: "games ingested [daily / within N hours of reporting], rankings recomputed [on each ingestion run / weekly / etc.]."

If data freshness varies by source, say so: "GotSport-sourced games update [cadence]; other sources [cadence]."

---

## Quality Standard

- Every factual claim must trace to a filed FORGE document (source coverage report, pipeline design, etc.). No invented numbers.
- The Option A vs. Option B distinction must be explicit — FORGE must not present Option B as confirmed if the browser session has not run.
- Tone is confident, not hedged. "Evo Draw covers X" not "Evo Draw attempts to cover X." Where there are limits, name them cleanly rather than softening every sentence.
- Length: fits on one printed page. If it runs longer, cut Section 3 or 4, not Section 1 or 2.

## Why This Matters

SENTINEL currently has no document that answers the question "What do I say about Evo Draw's coverage in the DSS presentation?" The Coverage Report answers "What does the data say?" This document answers "What does Presten say?" Without it, Presten improvises coverage claims on May 16, which risks both over-promising and under-communicating what makes Evo Draw credible. Writing it now — three weeks before DSS — gives SENTINEL and Presten time to refine the language and ensure ELO's accuracy claims and FORGE's coverage claims are consistent with each other.

## References

- `Infrastructure/Pre-DSS Source Coverage Report — May 2026.md` — source data for all claims
- `Rankings/DSS Ranking Accuracy Claims — Authorized Spec.md` — ELO's authorized accuracy claims (must be consistent with this narrative)
- `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` — confirmed org-ID status for league claims
- `Infrastructure/GotSport Org-ID Expansion Stage 2 — Pre-Authorization Criteria.md` — Stage 2 scope for Section 4
