---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-24
status: completed
completed: 2026-04-24
priority: medium
due: 2026-04-28
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/Source Gap Inventory — April 2026.md"
---

# Task: Source Gap Inventory — All Leagues vs Active Sources

## Context

SENTINEL's core mandate is product completeness. Per the Config: "For every league in [[League Hierarchy]], verify that game coverage is comprehensive."

The current vault documents individual sources (GotSport, SnapSoccer, direct scraper, Tournament Director exports) but does not have a single inventory showing, for each league in the League Hierarchy, whether a data source exists and is actively ingesting. SENTINEL cannot assess coverage gaps without this.

FORGE owns the data pipeline and knows the ingestion sources. This task: write the inventory.

## What to Write

Write: `02 - Tiger Tournaments/Projects/Infrastructure/Source Gap Inventory — April 2026.md`

---

### Section 1: Inventory Table

For every league listed in `02 - Tiger Tournaments/Projects/Leagues/League Hierarchy.md`, write one row:

| League | Gender | Source Type | Source ID / Config Key | Last Known Ingestion | Status |
|--------|--------|-------------|----------------------|---------------------|--------|
| ECNL | Girls | GotSport org_id | [org_id] | [date or "unknown"] | Active / Stale / No source |
| MLS NEXT | Boys | GotSport org_id | [org_id] | [date or "unknown"] | Active / Stale / No source |
| GA | Both | GotSport org_id | [org_id] | [date or "unknown"] | Active / Stale / No source |
| ... | | | | | |

**Status definitions:**
- **Active** — source is in the crawl config, ingested within last 60 days
- **Stale** — source is in the crawl config but no games ingested in 60+ days (may need GotSport org-ID refresh or new events)
- **No source** — league has no corresponding GotSport org_id or scraper configured

Pull source information from:
- The GotSport org_id list (from `Infrastructure/gotsport-org-id-list.md` or equivalent)
- The crawl configuration files (document the config file path)
- FORGE's knowledge of what's been ingested based on briefing history

If you cannot confirm "Last Known Ingestion" from vault records alone: write "unknown — query needed" and provide the SQL:
```sql
SELECT e.source, MAX(g.created_at) AS last_game_ingested
FROM games g
JOIN events e ON e.id = g.event_id
WHERE e.source = '[source_identifier]'
GROUP BY e.source;
```

---

### Section 2: Coverage Gap Summary

After the table, write a ranked list of gaps:

**Priority 1 (No source — league produces games that should be in Evo Draw):**
List any league from the Hierarchy that has no configured source. These are the most severe gaps — games from these leagues are completely absent from rankings.

**Priority 2 (Stale source — source exists but may be out of date):**
List sources where the last ingestion date is either unknown or > 60 days old. These leagues may have incomplete recent game records.

**Priority 3 (Source exists but known limitations):**
E.g., SnapSoccer is NO-GO (documented in FORGE's 11:16 briefing). Affinity Soccer (IL + Midwest) is deprioritized. List these with their deprioritization rationale.

---

### Section 3: Next Actions

For each Priority 1 gap: write a one-line action. Format:
- `[League] — Research GotSport org_id or alternative source. Assign to FORGE if a GotSport org_id is findable; escalate to SENTINEL if source type is unknown.`

For each Priority 2 gap: write a one-line action.
- `[League] — Verify if org_id is still valid. Check if new GotSport events exist under this org_id since last ingestion.`

SENTINEL will use this list to assign follow-up research tasks.

---

## Definition of Done

- Inventory table covers every league in League Hierarchy (Girls and Boys circuits listed separately where calibration differs)
- Every league has a Status: Active, Stale, or No source — not "unknown" for Status itself
- Last Known Ingestion may be "unknown — query needed" but Source ID and Status must be filled in
- Priority 1 and Priority 2 gaps are identified and listed
- Summary in briefing: "Source gap inventory complete. Priority 1 gaps (no source): [N]. Priority 2 gaps (stale): [N]. Active sources confirmed: [N]."

## Why This Matters

SENTINEL cannot answer "is game coverage comprehensive?" without knowing what's actually being ingested. The GotSport org-ID expansion (Presten's task), SnapSoccer NO-GO decision, and Affinity Soccer deferral all imply known gaps — but no one has written them in one place. This inventory is the foundation for all future source research tasks SENTINEL assigns to FORGE.

## References

- `02 - Tiger Tournaments/Projects/Leagues/League Hierarchy.md` — master league list
- `02 - Tiger Tournaments/Projects/Infrastructure/gotsport-org-id-*.md` — GotSport source configs (use whatever files exist)
- `10 - Agents/FORGE/Briefings/2026-04-24-11:16.md` — SnapSoccer NO-GO documented
- SENTINEL Config — "Find game sources: actively search for new data sources for games across all leagues"
