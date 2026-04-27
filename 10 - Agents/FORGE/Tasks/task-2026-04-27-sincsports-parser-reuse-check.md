---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-27
status: open
priority: medium
due: 2026-05-03
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/SnapSoccer — SincSports Parser Reuse Assessment.md"
---

# Task: SnapSoccer — SincSports Parser Reuse Assessment

## Context

FORGE's April 26 15:46 briefing (Non-GotSport Top Sources — Ingestion Design) identified SnapSoccer as the highest-priority non-GotSport source to activate post-DSS. The ingestion design confirmed:

- SnapSoccer runs on the SincSports platform — the same platform as 34,961 existing games in the Evo Draw DB
- The existing SincSports bulk import used the same HTML table structure
- Build estimate: **5–9 hours (test run) / 8–12 hours (production)** — but drops to **4–6 hours** if the existing SincSports parser is reusable
- FORGE's own briefing: "a 30-minute code check would confirm" whether reuse is viable

The 4–6 hour estimate vs. 8–12 hour estimate is not a rounding difference — it's the difference between a single tight session and a multi-session effort. This assessment should happen now, before the post-DSS sprint is scoped, so Presten has accurate numbers when prioritizing.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/SnapSoccer — SincSports Parser Reuse Assessment.md`

---

### Section 1: Existing Parser Inventory

List the existing SincSports-related code files and what each does:

| File | Function | Last Modified | Notes |
|------|----------|--------------|-------|
| | | | |

If FORGE cannot locate the existing parser (e.g., it was a one-off script not preserved in the codebase), state this explicitly — it changes the reuse assessment to "start from scratch."

---

### Section 2: SnapSoccer Data Structure Comparison

Compare the existing SincSports parser's expected input against SnapSoccer's public game listing pages:

| Field | Existing SincSports Format | SnapSoccer (snapsoccer.org) Format | Compatible? |
|-------|--------------------------|-------------------------------------|------------|
| Home team name | | | |
| Away team name | | | |
| Score | | | |
| Date | | | |
| Event/tournament name | | | |
| Age group | | | |
| Gender | | | |
| Game ID or URL | | | |

"Compatible" means the field is present in both, in the same or easily-mapped format. "Incompatible" means the field is absent or requires non-trivial transformation.

---

### Section 3: Reuse Verdict

State one of three verdicts:

**REUSE** — Existing parser handles SnapSoccer with ≤ 2 hours of adaptation work. Revised estimate: 4–6 hours test run / 6–8 hours production.

**ADAPT** — Existing parser handles 50–75% of SnapSoccer's data model but requires substantial modification. Revised estimate: 6–9 hours test run / 9–12 hours production.

**REBUILD** — SnapSoccer's structure diverges significantly from existing SincSports parser, or parser is not locatable. Original estimate stands: 5–9 hours test run / 8–12 hours production.

State the verdict, the primary reason, and the specific incompatibility or compatibility that drove it.

---

### Section 4: Adaptation Scope (if REUSE or ADAPT)

If verdict is REUSE or ADAPT, describe what changes are needed:

- Which functions/modules need modification
- What new logic is required (e.g., pagination handling, URL construction, field remapping)
- Estimated hours for adaptation only (not full build)

If verdict is REBUILD, describe the one or two structural differences that make reuse impractical.

---

### Section 5: Recommended Next Step

One sentence: the concrete action Presten should take next on SnapSoccer ingestion, based on this assessment. Should reference the post-DSS sprint timeline (target activation: May 17+).

---

## Quality Standard

- Section 2 must be filled in. If SnapSoccer's public pages are not accessible for comparison (auth-gated), state this and note it as a risk to the ingestion design.
- The reuse verdict must be one of the three options above — do not use hedged language ("it depends," "possibly"). Pick the closest verdict and state the rationale.
- If the existing parser is not locatable, the task is still complete: state the finding clearly and update the build estimate to the "start from scratch" range.

## Why This Matters

The SnapSoccer ingestion design is approved in principle. The only open question before scheduling a build session is whether the existing SincSports parser cuts the effort in half. If it does, the post-DSS sprint plan looks different — SnapSoccer goes first, in a single afternoon session. If it doesn't, the ordering may shift. SENTINEL cannot advise Presten on sprint sequencing without this number. The assessment itself is 30 minutes of code reading; the deliverable file structures the finding for the record.

## References

- `Infrastructure/Non-GotSport Top Sources — Ingestion Design.md` — the ingestion design this assessment updates
- SnapSoccer public game listings: `snapsoccer.org` (public, no auth required per ingestion design)
- Existing SincSports bulk import code in the Evo Draw codebase
