---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-27
status: pending
priority: medium
due: 2026-04-30
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/Stage 1 TX Crawl — Expected Output Spec.md"
---

# Task: Stage 1 TX Crawl — Expected Output Spec

## Context

The Stage 1 backfill is the TX test cohort: a targeted crawl of Texas GotSport organizations to validate pipeline performance before Stage 2. The Stage 1 Backfill Results Report (`task-2026-04-25-stage1-backfill-results-report.md`) is blocked on Presten executing the crawl. That remains blocked. But there is useful pre-crawl work FORGE can do now.

The May 1 Day-Of Runbook contains a placeholder: "Expected game count: pending Stage 1 results." The Morning-After Runbook references "GREEN criteria" for game counts. Neither document has numbers, because Stage 1 hasn't run. FORGE has all the information needed to produce a reasonable pre-crawl estimate: TX org-IDs in the config, per-league game volume rates from the GotSport org-ID research, and the pipeline's per-org behavior from prior backfill work.

Writing the expected output spec now converts two unknowns (May 1 expected game count, Stage 1 pass/fail criteria) into estimated ranges — before Stage 1 runs. When Stage 1 results come in, Presten and SENTINEL immediately know if they're in range or out of range, rather than assessing with no baseline.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/Stage 1 TX Crawl — Expected Output Spec.md`

---

### Section 1: TX Org-ID Scope

List every Texas organization currently in the Stage 1 crawl config (from `Infrastructure/full-gotsport-org-id-list.md` and/or the pre-staged additions doc). For each:

| Org-ID | Entity Name | Type | Game Volume Estimate (per season) | Source of Estimate |
|--------|-------------|------|----------------------------------|-------------------|

"Game Volume Estimate" = best estimate of annual games that org_id hosts on GotSport. Pull from the org-ID expansion research tasks or the source gap inventory. If no estimate exists, note "unknown — assess post-crawl."

---

### Section 2: Aggregate Expected Range

Based on Section 1:

- **Low estimate:** Sum of low-end volume estimates across all TX org-IDs, factoring in season timing (April = spring season active, fall season games not yet played)
- **High estimate:** Sum of high-end estimates
- **Expected:** A single "most likely" number FORGE believes is the best point estimate

State the formula used:
`Expected TX game count = Σ(per-org estimated games) × [season fraction for April]`

If season fraction is unknown, state the assumption (e.g., "assuming 40% of annual games played by end of April").

---

### Section 3: Pass/Fail Criteria for Stage 1

Define the criteria Presten and SENTINEL use to assess the Stage 1 crawl result:

| Result Category | Criterion | Interpretation |
|----------------|-----------|---------------|
| GREEN — healthy | Game count ≥ [low estimate] | Stage 1 succeeded; Stage 2 authorized (pending other gates) |
| YELLOW — investigate | Game count between [50% of low] and [low estimate] | Partial success — identify which org-IDs returned low game counts |
| RED — failure | Game count < [50% of low estimate], or 0 games from any active org-ID | Stage 1 failed; do not proceed to Stage 2 |

Add a secondary criterion: **ecnl_verified population rate**. After the crawl, run S4-4 (from the Section 4 Query Package). If population rate is 0% for ECNL games, flag to SENTINEL.

---

### Section 4: Post-Crawl Verification Queries

Three SQL queries Presten runs immediately after the Stage 1 crawl completes:

**Q1 — TX Game Count By Org-ID**
```sql
-- ELO/FORGE writes the query
-- Returns: org_id, game_count ordered by game_count DESC
-- Expected: all active TX org-IDs show > 0 games
```

**Q2 — Crawl Recency Check**
```sql
-- Returns: most recent game_date ingested per org_id
-- Expected: all org_ids show games within the current season window
```

**Q3 — Source Error Check**
```sql
-- Returns: any source-level errors or null org_id entries from this crawl run
-- Expected: 0 errors
```

---

### Section 5: May 1 Expected Count Inheritance

State explicitly how the Stage 1 results feed the May 1 Day-Of Runbook:

- When Stage 1 runs and produces a GREEN result: FORGE updates the May 1 Day-Of Runbook placeholder ("Expected game count: pending Stage 1 results") with the Stage 1 actual count ± 20% tolerance.
- This is the "first run expected game count" that SENTINEL uses on May 2 to assess GREEN/YELLOW/RED.
- If Stage 1 does not run before May 1: state the fallback approach (use the low estimate from Section 2 as the GREEN minimum, flag in the May 1 briefing that the estimate is pre-crawl).

---

## Quality Standard

- Volume estimates must reference their source (org-ID research doc, known league game rates, or explicit "no data — placeholder only"). Do not invent numbers without stating the basis.
- Pass/fail criteria must be numbers, not qualitative descriptions.
- If FORGE genuinely has no data to estimate a volume (e.g., an org-ID researched but with no game count found), mark that org-ID as "volume unknown" and state the impact on overall estimate uncertainty.
- Section 5 is the actionable output: after Stage 1 runs, FORGE uses this spec to update the May 1 Runbook in under 5 minutes.

## Why This Matters

Every monitoring document downstream of Stage 1 (May 1 Runbook, Morning-After Runbook, Stage 2 Authorization Gate) has a dependency on "expected game count" that is currently unfilled. FORGE can estimate that number now. If the estimate is off by 50%, it still gives SENTINEL a baseline to assess against — which is infinitely more useful than no baseline. Writing this spec before Stage 1 runs is cheap; reconstructing it after a failed Stage 1 run under time pressure is not.

## References

- `Infrastructure/full-gotsport-org-id-list.md` — TX org-IDs
- `task-2026-04-24-full-gotsport-org-id-list.md` — org-ID volume research source
- `Infrastructure/May 1 Deployment — Day-Of Runbook.md` — inherits expected count from this spec
- `Infrastructure/Daily Pipeline Morning-After Validation Runbook.md` — GREEN criteria that depend on expected count
- `Infrastructure/April 29 — Section 4 Source Baseline Query Package.md` — Q4 (ecnl_verified) runs after Stage 1
