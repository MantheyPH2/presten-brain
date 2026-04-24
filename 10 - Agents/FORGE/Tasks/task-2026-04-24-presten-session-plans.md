---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-24
status: pending
priority: high
due: 2026-04-25
---

# Task: Create Presten Execution Session Plans — 1h / 3h / 6h Blocks

## Context

Every unblocked critical-path item is waiting on Presten execution time. SENTINEL has flagged five execution items across multiple briefings. The problem is not that Presten doesn't know what to do — it's that the items are scattered across task files, queue items, and reports. The board shows:

- ECNL dedup P0 fix: 15 minutes (one-line change, spec filed)
- GA ASPIRE tier fix: 45–60 minutes (SQL written and staged)
- USA Rank SQL audit: 30–45 minutes (16 queries pre-written)
- Backoff code change: ~60 minutes (spec in `Reports/backoff-implementation-2026-04-24.md`)
- TX org-ID browser lookup + Stage 1 backfill pilot: ~45–90 minutes (procedures documented)
- Event Strength implementation: ~7 hours (full session)

Presten does not know at the start of a session how much time is available. If session plans are pre-built, Presten opens one document and starts executing — no context-switching to figure out what's highest priority.

## What to Build

Write: `02 - Tiger Tournaments/Projects/Product & Planning/Presten Session Plans — April 2026.md`

### Header: Priority Decision Guide

At the top of the document, before any session blocks:

```
If you have 15 minutes: Apply the ECNL dedup P0 fix (one-line change).
If you have 1 hour: Run the 1-Hour Block below.
If you have 3 hours: Run the 3-Hour Block below.
If you have a full day: 3-Hour Block in the morning + Event Strength session in the afternoon.
If you are unsure what to do first: Start at Step 1 of the 1-Hour Block.
```

---

### Session Block 1: 1-Hour Block — "Quick Wins"

**Total time: ~60 minutes. Unblocks: June 1 risk, GA accuracy, calibration approval.**

**Step 1: Apply ECNL dedup P0 fix (~15 minutes)**
- File to modify: `[path to ecnl-transition-dedup.js]` — the exact path is in `Reports/ecnl-dedup-verified-fix-2026-04-24.md`
- Action: apply the one-line diff documented in that report
- Confirm it worked: run `grep -n "verified" [path to file]` and confirm the fix is present
- Time box: 15 minutes maximum. This is a one-line change.

**Step 2: Execute GA ASPIRE tier fix (~45 minutes)**
- Source: `Reports/ga-aspire-tier-fix-2026-04-[date].md` — all SQL is pre-written
- Action: open a psql session against `youth_soccer` at localhost:5432 (user `p`) and execute the queries in order
- Expected output: UPDATE statement affecting 350–1,050 rows
- Confirm it worked: run the verification query from the report; confirm affected teams moved from `ga_aspire` incorrect tier to correct tier
- Time box: 45 minutes including verification.

**Exit check:** After completing both steps, confirm in the terminal that no errors occurred. Note: the GA ASPIRE fix triggers a ranking recompute — if the pipeline does not run automatically, schedule or trigger it manually.

---

### Session Block 2: 3-Hour Block — "Critical Path Unblock"

**Total time: ~3 hours. Includes everything in the 1-Hour Block, plus backoff deployment and Stage 1 backfill.**

Complete the 1-Hour Block first, then:

**Step 3: Apply exponential backoff fix to the sync runner (~60 minutes)**
- Source: `Reports/backoff-implementation-2026-04-24.md` — the exact code diff is in this document, copy-paste ready
- File to modify: `crawl-gotsport-api-parallel.js` (or the equivalent sync runner documented in the report)
- Action: implement `fetchWithBackoff()` per the spec. Base delay 600ms, 1s on first 429, doubling to 32s cap, ±20% jitter, 5 max retries, log every backoff event.
- Test: run the sync runner against a single team ID after implementation. Confirm no 429 errors, confirm the new log format appears.
- Time box: 60 minutes. If you hit an unexpected issue, stop and note the blocker — do not spend more than 60 minutes.

**Step 4: TX org-ID browser lookup + Stage 1 backfill pilot (~45 minutes)**
- Source: `Data Sources/GotSport USYS Org-ID Master List.md` — lookup procedure for TX org ID is documented here
- Action: in a browser, navigate to the GotSport TX state association page and retrieve the org ID. Update the JSON config block in the document with the confirmed TX org ID.
- Then: execute Stage 1 of the stale team backfill pilot per `Infrastructure/Stale Team Backfill Runbook — April 2026.md` — 10 teams, 15-minute run. Go/no-go criteria are in the runbook. If Stage 1 passes, proceed to Stage 2 the same day (50 teams).
- IMPORTANT: Do NOT run Stage 3 of the backfill on the same day as the first daily pipeline run. The sequencing constraint is documented in the runbook.
- Time box: 45 minutes total for lookup + Stage 1.

---

### Session Block 3: 6-Hour Block — "DSS Feature Work"

**Total time: 6–8 hours. Includes the 3-Hour Block plus a full Event Strength implementation session.**

Complete the 3-Hour Block first. Then:

**Step 5: Event Strength 7-hour implementation session**
- Source: the Event Strength implementation plan (check `Product & Planning/` for the current file)
- This is its own full session. Do not attempt to compress it. It takes 7 hours.
- Pre-flight: before starting, run the Event Strength post-compute validation spec (`Rankings/Event Strength Post-Compute Validation — 2026-04-24.md`) to understand what correct output looks like before you implement.
- If you are starting this after completing the 3-Hour Block in the same day, take a break before beginning.

---

### Bonus: 30-Minute Quick Win (Standalone)

If you have 30 minutes and want to close a blocked item:

**USA Rank SQL audit**
- Source: `Rankings/USA Rank Ranking Accuracy Audit — April 2026.md` — 16 queries pre-written
- Action: psql session, run the 16 lookups, fill in the Evo Draw columns in `Rankings/USA Rank Comparison 2026-04-23.md`
- This unblocks calibration approval. It is purely data entry — the SQL is already written.

---

## What to Document in Your Briefing

When you file this deliverable, include:
- Vault path to the Session Plans document
- Confirm that all source document paths referenced in the session plans are verified (file exists at the path you cite)
- Note any source documents you could not locate so SENTINEL can fix the reference

## Definition of Done

- All four session blocks are written with actionable steps (not "do X" — must include file paths, expected outputs, time boxes)
- Source document paths are specific vault paths, not vague descriptions
- The priority decision guide at the top is present and can be read in under 30 seconds
- Every step in the 1-Hour Block is executable without opening any other document
- Summary in briefing: session plans filed, all source paths verified

## Why This Matters

SENTINEL has now flagged five Presten execution items across eight consecutive briefings. The bottleneck is not knowledge — it's friction. One document, one decision guide, no context-switching. That is the standard for an execution-ready product.

## References

- `Reports/ecnl-dedup-verified-fix-2026-04-24.md` — ECNL P0 fix diff
- `Reports/ga-aspire-tier-fix-*` — GA ASPIRE SQL
- `Reports/backoff-implementation-2026-04-24.md` — backoff code spec
- `Data Sources/GotSport USYS Org-ID Master List.md` — TX org ID lookup
- `Infrastructure/Stale Team Backfill Runbook — April 2026.md` — backfill stages
- `Rankings/USA Rank Ranking Accuracy Audit — April 2026.md` — 16 audit queries
