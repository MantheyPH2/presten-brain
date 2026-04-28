---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-27
status: completed
completed: 2026-04-27
priority: medium
due: 2026-05-03
deliverable: "02 - Tiger Tournaments/Projects/Data Sources/Non-GotSport Source Priority Matrix — April 2026.md"
---

# Task: Non-GotSport Source Priority Matrix

## Context

FORGE has produced individual research documents for three non-GotSport sources:

- **Affinity Soccer** — `Data Sources/Affinity-Soccer-Access-Path-Research-2026-04-27.md`
- **NAL** — `Data Sources/NAL-Platform-Identification-Research-2026-04-27.md`
- **SnapSoccer** — `Data Sources/SnapSoccer-TID-Coverage-Estimate-2026-04-27.md` and `Data Sources/SnapSoccer-Minimum-Viable-Parser-Spec-2026-04-27.md`

These exist as separate research documents. What does not exist: a single, consolidated prioritization that lets SENTINEL and Presten answer the question "which non-GotSport source do we build next, and why?"

The May 17 post-DSS sprint includes SnapSoccer as a planned build. But there may be sources FORGE has identified (or should check) that have higher game volume, easier access, or more strategic coverage value than SnapSoccer. Without a consolidated matrix, SENTINEL cannot confirm that SnapSoccer is the right first non-GotSport build — it can only confirm that SnapSoccer is a reasonable one.

FORGE may also have encountered additional sources in its source coverage research (Source Gap Inventory, competitive coverage gap analysis) that haven't been written up as individual research documents but are worth including in the matrix.

---

## What to Produce

File: `02 - Tiger Tournaments/Projects/Data Sources/Non-GotSport Source Priority Matrix — April 2026.md`

---

### Section 1: Source Inventory

List every non-GotSport source FORGE has identified. At minimum: Affinity Soccer, NAL, SnapSoccer. Include any others from the Source Gap Inventory or competitive coverage analysis.

For each, one-line summary: platform type, what leagues/games it covers, current research status (research complete, browser check pending, no research yet).

---

### Section 2: Priority Matrix

| Source | Est. Game Volume | Build Effort (hrs) | Access Confidence | Strategic Value | FORGE Priority Score |
|--------|-----------------|-------------------|-------------------|-----------------|----------------------|
| SnapSoccer | ~5,000–7,000 (net new) | ~4–6 hrs | Medium (browser check pending) | Regional gap-fill (Gulf Coast) | [score] |
| Affinity Soccer | [estimate] | [estimate] | [Low/Med/High] | [value statement] | [score] |
| NAL | [estimate] | [estimate] | [Low/Med/High] | [value statement] | [score] |
| [other sources] | | | | | |

**Scoring methodology:** FORGE defines the scoring formula. Suggested inputs: game volume (higher = better), build effort (lower = better), access confidence (higher = better), and strategic coverage value (SENTINEL will weight this qualitatively). State the formula explicitly.

**Access confidence definitions:**
- High: Public access confirmed; parser architecture is known
- Medium: GotSport or similar platform suspected; browser check required to confirm
- Low: Platform unknown or gated behind registration/payment

---

### Section 3: Sequencing Recommendation

Based on the matrix, FORGE recommends a build sequence for non-GotSport sources post-DSS sprint:

1. **First build (May 17):** [Source] — because [reason]
2. **Second build (June–July sprint):** [Source] — because [reason]
3. **Conditional build:** [Source] — if [condition] is confirmed via browser check

FORGE must explicitly state whether its sequencing recommendation differs from the current plan (SnapSoccer first on May 17). If SnapSoccer is still the right first build, say so directly with the comparison rationale.

---

### Section 4: Outstanding Access Unknowns

For any source where access is unconfirmed, list the specific browser check FORGE recommends Presten run and the expected time investment.

| Source | Browser Check Required | Est. Time | What Confirms ACCESSIBLE |
|--------|----------------------|-----------|-------------------------|
| SnapSoccer | soccer.sincsports.com schedule page | ~20 min | schedule.aspx loads with game rows visible |
| Affinity Soccer | [specific URL pattern to check] | ~[time] | [what ACCESSIBLE looks like] |
| NAL | [GotSport browser check, NAL org-ID lookup] | ~10 min | Org-ID found, schedule visible |

---

### Section 5: Sources Explicitly Deprioritized

List any sources FORGE investigated and is recommending against building in the May–August window, with the reason.

Example: "Platform X — gated behind paid API; estimated $X/month; not worth pursuing until post-DSS traffic data justifies cost."

This section prevents SENTINEL or Presten from re-raising deprioritized sources without context.

---

## Quality Standard

- Priority scores must be numeric and formula-driven, not subjective.
- Build effort estimates must be in hours, not "easy/medium/hard."
- Game volume estimates should match the numbers in the individual research documents. If they conflict, explain why.
- Section 3 must name a first, second, and conditional build. "TBD" is not acceptable.
- Do not include sources for which FORGE has zero information — note them as "identified, no research conducted."

## Why This Matters

SENTINEL's post-DSS coverage gap work requires knowing which non-GotSport sources to pursue, in what order, with what effort. The individual research documents are inputs to a decision; this matrix is the decision. Without it, SENTINEL and Presten read three separate research docs and synthesize priorities informally — which produces inconsistent decisions and missed comparisons. This matrix makes the priority explicit and auditable.

The May 17 SnapSoccer build is the working plan. This matrix confirms it or corrects it before May 17 arrives.

## References

- `Data Sources/Affinity-Soccer-Access-Path-Research-2026-04-27.md`
- `Data Sources/NAL-Platform-Identification-Research-2026-04-27.md`
- `Data Sources/SnapSoccer-TID-Coverage-Estimate-2026-04-27.md`
- `Data Sources/SnapSoccer-Minimum-Viable-Parser-Spec-2026-04-27.md`
- `Infrastructure/Source Gap Inventory — April 2026.md` — additional sources from coverage analysis
- `Infrastructure/Competitive Coverage Gap Inventory.md` — strategic coverage context
