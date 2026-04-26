---
title: April 29 Pipeline Validation Spec
tags: [infrastructure, pipeline, validation, ga-aspire, ecnl, deployment, evo-draw, forge]
created: 2026-04-25
updated: 2026-04-25
author: FORGE
status: filed — pipeline code inspection blocked; see Section 3 and blocker note
task: task-2026-04-25-april29-pipeline-validation-spec
---

# April 29 Pipeline Validation Spec

> **Purpose:** Confirm whether the pipeline *code* correctly handles the two schema changes applied on April 28 — GA ASPIRE event tier fix and the `ecnl_verified` column. Catches code-level regressions before the May 1 deployment. The April 28 session fixes the DB; this spec confirms the pipeline won't silently undo those fixes on first run.

> [!warning] Pipeline Code Access Blocker
> FORGE operates from the Obsidian vault context and cannot access the pipeline codebase (`crawl-gotsport-api-parallel.js`, `tgs_scraper`, etc.) to perform the file:line inspections required by Sections 1 and 2. All answers requiring code inspection are marked **BLOCKER — PIPELINE CODE NOT ACCESSIBLE FROM VAULT**. Presten must perform or authorize these code inspections before April 28, or FORGE must be granted access to the pipeline repo.
>
> **Resolution:** Presten reviews the pipeline code against the specific questions below and fills in the answer column. Alternatively, share the relevant pipeline files with FORGE in a session that includes codebase access.

---

## Section 1: GA ASPIRE Event Tier — Pipeline Write Behavior

After the April 28 UPDATE, existing games have correct `event_tier` values. The forward-looking question: when the pipeline ingests a new game from a GA ASPIRE tournament, what will it write?

| Question | Answer | Source (file:line) |
|----------|--------|-------------------|
| How does the pipeline determine `event_tier` for a new game? | BLOCKER — pipeline code not accessible from vault | `crawl-gotsport-api-parallel.js` or event classifier module — inspect at pipeline root |
| Where is the `ga_aspire` string defined (or absent) in the classifier? | BLOCKER — inspect event_tier classification logic | Likely in an event classifier or constants file — search for `ga_aspire` and `event_tier` |
| Will new GA ASPIRE games ingested after April 28 get `event_tier = 'ga_aspire'` or `'ga'`? | BLOCKER — depends on classifier answer above | Same source |
| If the pipeline overwrites `event_tier` on re-ingestion, does the April 28 UPDATE survive a re-scrape of existing GA ASPIRE games? | BLOCKER — inspect whether INSERT logic uses ON CONFLICT DO UPDATE for event_tier | Likely in the game insert/upsert function |

**Pass condition:** Pipeline correctly writes `ga_aspire` for new games AND does not overwrite corrected existing game rows on re-scrape.

**Fail condition:** Pipeline re-classifies GA ASPIRE games as `ga` on re-scrape — the April 28 UPDATE reverts the moment the pipeline runs. If FAIL, a code change is required before May 1; FORGE will identify and document the specific change upon codebase access.

**Critical risk if not resolved:** On May 1 first pipeline run, GA ASPIRE event_tier rows revert to `ga`. ELO recalibration for U13/U14 Girls reverts to pre-fix state. This will not be visible until May 2 monitoring queries show GA ASPIRE ratings shifted back. Fixing it then requires a second April 28-equivalent session.

---

## Section 2: `ecnl_verified` — Pipeline Write Behavior

After the April 28 `ALTER TABLE` + backfill, historical ECNL games are `ecnl_verified = TRUE`. For new games ingested after April 28:

| Question | Answer | Source (file:line) |
|----------|--------|-------------------|
| Does the pipeline (`tgs_scraper`) currently write `ecnl_verified` for new game inserts? | BLOCKER — inspect tgs_scraper INSERT payload | `tgs_scraper` root or game insert function — search for `ecnl_verified` |
| If not: does it default to FALSE (correct) or NULL (problem)? | BLOCKER — depends on column DEFAULT and INSERT behavior | If INSERT omits `ecnl_verified`, column DEFAULT FALSE applies — safe, but TGS ECNL games accumulate as FALSE indefinitely |
| Is there a code change required to set `ecnl_verified = TRUE` on ingestion for ECNL-source games? | YES — per `Infrastructure/ECNL ecnl_verified Column — Population Plan.md` Section 3. The change is identified; whether it has been made is BLOCKER. | `tgs_scraper` column mapping or INSERT construction |
| Has the required tgs_scraper code change (set `ecnl_verified: true` when `event_tier === 'ecnl'`) been made? | BLOCKER — inspect tgs_scraper | The Population Plan (filed 2026-04-25) specified this as a TODO for the May 1 pipeline session |

**The required code change (from Population Plan Section 3):**

In the TGS scraper's column mapping or INSERT construction, add:

```javascript
// When source = 'tgs' or 'tgs_athleteone' AND event_tier resolves to 'ecnl':
ecnl_verified: true
```

**Pass condition:** `tgs_scraper` writes `ecnl_verified = TRUE` for games where `source IN ('tgs', 'tgs_athleteone') AND event_tier = 'ecnl'`.

**Fail condition:** `tgs_scraper` leaves `ecnl_verified` as NULL or FALSE for new ECNL games. This means the backfill (Step 2b, April 28) correctly marks historical games TRUE, but every new game ingested after April 28 returns to FALSE. The `ecnl_verified` flag accumulates an ever-growing population of FALSE new games, making it progressively less reliable as the June 2 migration verification baseline.

**If FAIL:** This is a May 1 blocker. The code change must be applied before May 1 pipeline launch. FORGE will provide the exact file path and change once pipeline codebase access is available.

---

## Section 3: May 1 Deployment Go / No-Go Recommendation

| Check | Status | Impact if Not Fixed Before May 1 |
|-------|--------|----------------------------------|
| GA ASPIRE forward-write correct | BLOCKER — code not inspected | April 28 DB fix reverts on first pipeline run; U13/U14 Girls ELO recalibration undone |
| GA ASPIRE re-scrape safe (ON CONFLICT behavior) | BLOCKER — code not inspected | Re-ingestion of existing GA ASPIRE games overwrites corrected event_tier rows |
| `ecnl_verified` write on new TGS ECNL games | BLOCKER — code not inspected | New ECNL games accumulate as `ecnl_verified = FALSE`; June 2 migration baseline degraded |
| tgs_scraper code change deployed | BLOCKER — code not inspected | ECNL dedup feature (`ecnl-transition-dedup.js`) non-functional post-May-1; verified-flag approach fails |

**Current status: CANNOT DETERMINE PASS/FAIL.** All four checks require pipeline codebase access.

**Recommendation:** Before April 28, Presten should perform a 15-minute code review answering the 8 questions in Sections 1 and 2 above, then fill in this table. If any check FAILS, FORGE needs to know before April 28 so the code change can be scoped and scheduled alongside the May 1 deployment session.

If code inspection cannot be completed before April 28: run the April 28 DB session as planned, but flag May 1 deployment as CONDITIONAL. After April 28, run a test pipeline ingestion on a known GA ASPIRE game and a known ECNL game, verify the written field values in the DB, and confirm PASS/FAIL empirically before the full May 1 launch.

---

## Appendix: Pipeline Code Review Checklist for Presten

To complete this spec, Presten reviews the following in the pipeline repo:

**GA ASPIRE (Section 1):**
- [ ] Search for `ga_aspire` in the codebase. If found: note file:line. If not found: this is a FAIL — the classifier does not know about `ga_aspire`.
- [ ] Find the `event_tier` classification logic. Confirm it handles GA ASPIRE events distinctly from standard GA.
- [ ] Find the game INSERT/upsert function. Confirm whether `event_tier` is included in an `ON CONFLICT DO UPDATE` clause. If yes: April 28 fix will revert on re-scrape of existing games.

**ecnl_verified (Section 2):**
- [ ] Search for `ecnl_verified` in `tgs_scraper`. If found: note current value assignment. If not found: column is not yet written by the scraper.
- [ ] Confirm whether the tgs_scraper code change from the Population Plan has been applied (search for `ecnl_verified: true` in the scraper).
- [ ] If not applied: confirm this is on the May 1 session plan before the pipeline launches.

---

## References

- `Infrastructure/ECNL ecnl_verified Column — Population Plan.md` — tgs_scraper change requirement (Section 3)
- `Infrastructure/April 28 Execution Log.md` — DB changes this spec validates against
- `Infrastructure/May 1 Deployment — Day-Of Runbook.md` — pipeline launch this spec prepares
- `Rankings/Post-Fix Calibration Stability Monitoring Spec.md` — monitoring that catches forward regressions if this spec is skipped

*FORGE — 2026-04-25*
