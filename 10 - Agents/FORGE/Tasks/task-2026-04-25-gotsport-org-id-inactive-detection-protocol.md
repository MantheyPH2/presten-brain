---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-25
status: completed
completed: 2026-04-25
priority: medium
due: 2026-04-30
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/GotSport Org-ID Inactive Detection Protocol.md"
---

# Task: GotSport Org-ID Inactive Detection Protocol

## Context

Stage 2 expansion will add up to 19 new org-IDs to the GotSport crawl config (7 P1 state assocs, 9 P2 state assocs, 3 league entities). After these are added and Stage 2 runs, some will return 0 games. SENTINEL and FORGE currently have no documented protocol for diagnosing why a crawl returns 0 games for a given org-ID.

Three distinct causes produce the same symptom (0 new games returned):

1. **Genuinely inactive**: The org-ID is in an offseason period or has no scheduled games in the current window. Expected. No action needed.
2. **Auth failure**: The GotSport browser session has expired or the org-ID requires different authentication. The crawl connects but returns 0 results due to permission. Should trigger re-authentication, not removal from config.
3. **Config error**: The org-ID is wrong, typo'd, or requires different parameters (query format, date range). The crawl contacts the wrong endpoint. Requires config correction.

Without this protocol, every 0-game return requires FORGE to investigate from scratch. With it, a 3-query diagnostic flow identifies the root cause in under 5 minutes.

This protocol becomes critical immediately after Stage 2 launches, when multiple new org-IDs run for the first time and some will inevitably return 0 results.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/GotSport Org-ID Inactive Detection Protocol.md`

---

### Section 1: Baseline Expectation Setting

Before running Stage 2, FORGE documents the expected active season for each state association org-ID based on known soccer calendars.

| Org-ID Category | Active Season Window | Expected Offseason |
|-----------------|---------------------|-------------------|
| State cup assocs (TX, CA, FL, etc.) | [FORGE fills from known state cup schedules] | [FORGE fills] |
| ECNL / ECNL RL league entities | Year-round (TGS-scraped; GotSport supplemental) | [note if any] |
| MLS NEXT state | [season window] | [offseason months] |

If Stage 2 runs during an offseason month for a given org-ID, 0 games is expected. This section is the reference for that expectation.

---

### Section 2: Diagnostic Flow for 0-Game Returns

When a Stage 2 org-ID returns 0 new games in a crawl run, FORGE runs this 3-step diagnostic:

**Step 1: Check historical game count**
```sql
-- Has this org_id ever returned games?
SELECT org_id, COUNT(*) as total_games, MAX(game_date) as most_recent_game
FROM games
WHERE org_id = '[target_org_id]'
GROUP BY org_id;
```
- If `total_games = 0` AND org is new: proceed to Step 2 (may be first crawl, auth issue, or offseason).
- If `total_games > 0` AND `most_recent_game > 60 days ago`: suspect auth failure or offseason. Proceed to Step 2.
- If `total_games > 0` AND `most_recent_game < 30 days ago`: recent games exist, this run may be a timing issue. Wait for next run before diagnosing.

**Step 2: Check cross-org baseline (auth health check)**
```sql
-- Are other org-IDs returning games in the same crawl run?
-- (Compare game ingestion timestamps from the most recent crawl run)
SELECT org_id, COUNT(*) as games_ingested, MAX(ingested_at) as last_ingested
FROM games
WHERE ingested_at >= NOW() - INTERVAL '24 hours'
GROUP BY org_id
ORDER BY last_ingested DESC
LIMIT 20;
```
- If other org-IDs ARE returning games in the same run window: the crawl is running; this specific org-ID has a problem (not global auth). Proceed to Step 3.
- If NO org-IDs are returning games: global crawl failure or global auth expiry. Do not diagnose individual org-IDs — fix the global issue first.

**Step 3: Classify the cause**

| Observation | Cause | Action |
|-------------|-------|--------|
| org-ID is new + Step 2 shows other orgs returning games + season is active | Config error (wrong org-ID or params) | Review config entry against browser lookup data. Check org-ID format. |
| org-ID is new + Step 2 shows other orgs returning games + season is offseason | Expected inactive | Log as "offseason — expected." Check again in [active season start]. |
| org-ID previously returned games + Step 2 shows other orgs returning games + season is active | Auth failure for this specific org-ID | Trigger targeted re-authentication for this org. Check if org requires different session or permission level. |
| No orgs returning games | Global crawl or auth failure | Run global auth refresh. Check crawl process health. |

---

### Section 3: Removal Threshold

An org-ID should be removed from the crawl config only when:

1. The org-ID is confirmed inactive (returned 0 games for **3 consecutive months** during active season), AND
2. The org-ID has been confirmed correct (not a config error), AND
3. SENTINEL has been informed and approved removal.

Do not remove an org-ID based on a single 0-game run. Do not remove during offseason.

When FORGE recommends removal: file a Queue item in `10 - Agents/FORGE/Queue/` with:
- Org-ID
- Historical game count (total, last 90 days)
- Season status at time of diagnosis
- Diagnostic steps completed
- Recommendation: remove / hold / investigate

---

### Section 4: Auth Expiry Detection (Global)

The GotSport browser session may expire without warning. Signs of global auth expiry:

- Multiple org-IDs simultaneously return 0 games in a single run (when prior runs were healthy)
- Step 2 diagnostic shows 0 or near-0 game ingestion across all org-IDs
- Any crawl log error messages referencing authentication, 401, or session token

If global auth expiry is suspected:
1. Presten re-authenticates at `system.gotsport.com` (same session as browser lookup)
2. FORGE restarts the next crawl run
3. FORGE files alert in briefing: "GotSport auth expiry detected and resolved. [Date]. All org-IDs returning to normal ingestion by [expected date]."

---

## Quality Standard

- All SQL queries must reference actual table/column names from the database schema. If FORGE does not have confirmed column names, write `[confirm: column_name]` placeholders.
- Section 3 removal threshold must be specific (3 months, not "an extended period").
- The diagnostic flow must terminate in a classification — no path should end in "investigate further."

## Why This Matters

Stage 2 adds 19 org-IDs simultaneously. On the first crawl run, SENTINEL will ask: "Which of the 19 are working?" Without this protocol, that question requires case-by-case investigation. With it, FORGE runs the 3-step diagnostic for each 0-game org-ID and reports: "3 expected offseason, 2 confirmed working, 1 potential config error under investigation." That is the information quality SENTINEL needs to authorize Stage 2 as healthy or flag it for remediation.

## References

- `Infrastructure/GotSport Org-ID Master Reference.md` — the complete org-ID list this protocol monitors
- `Infrastructure/Stage 2 Expansion — SENTINEL Authorization Gate.md` — Stage 2 authorization this protocol informs
- `Infrastructure/Daily Pipeline Morning-After Validation Runbook.md` — the broader health check this protocol feeds into
- `Infrastructure/GotSport Browser Lookup Session Prep Package.md` — the source of org-ID confirmation data
