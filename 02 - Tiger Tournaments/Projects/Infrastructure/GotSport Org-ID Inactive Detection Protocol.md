---
title: GotSport Org-ID Inactive Detection Protocol
tags: [infrastructure, gotsport, org-ids, pipeline, stage2, monitoring, evo-draw, forge]
created: 2026-04-25
updated: 2026-04-25
author: FORGE
status: active — use immediately after Stage 2 launches
task: task-2026-04-25-gotsport-org-id-inactive-detection-protocol
---

# GotSport Org-ID Inactive Detection Protocol

> **Purpose:** When a Stage 2 org-ID returns 0 new games in a crawl run, this 3-step diagnostic identifies whether the cause is genuine inactivity, auth failure, or config error — in under 5 minutes. Replaces case-by-case investigation with a repeatable decision flow.

> **When to use:** After any crawl run where a known org-ID shows 0 new games ingested. Mandatory for all new Stage 2 org-IDs on their first 3 runs.

---

## Section 1: Baseline Season Expectations

Before Stage 2 runs, FORGE documents expected active season windows per org-ID category. A 0-game return during an expected offseason is not a problem.

| Org-ID Category | Active Season Window | Expected Offseason | Notes |
|-----------------|---------------------|-------------------|-------|
| USYS State Cup Associations (TX, CA, FL, NY, NJ, PA, WA) | September–June (state cup events typically Nov–May) | July–August | State league games may run year-round; cup games are seasonal |
| ECNL / ECNL RL league entities | September–June | Late June–August | TGS-scraped primarily; GotSport supplemental |
| MLS NEXT state affiliates | August–June | Late June–July | Spring final events extend to late June |
| USL Academy | August–May | June–July | Tournament/showcase schedule may vary |
| EDP Soccer (Northeast regional) | September–June | July–August | Spring season extends to late June |
| Elite 64 | Varies — primarily fall (Sept–Nov) showcase events | December–August | Not a year-round league; primarily event-based |

> [!note] These windows are approximate based on known USYS and national league calendars. Update this table after Stage 2 first runs — actual org-level game dates will refine these estimates.

---

## Section 2: Diagnostic Flow for 0-Game Returns

When a Stage 2 org-ID returns 0 new games in a crawl run, run the following 3 steps in order.

### Step 1: Check Historical Game Count

Has this org-ID ever produced games in the database?

```sql
-- Has this org_id ever returned games?
SELECT source_org_id,
       COUNT(*) AS total_games,
       MAX(game_date) AS most_recent_game,
       MIN(game_date) AS earliest_game
FROM games
WHERE source_org_id = '[target_source_org_id]'
GROUP BY source_org_id;
```

**Interpret the result:**

| Result | Interpretation | Next step |
|--------|---------------|-----------|
| `total_games = 0` AND org-ID is new (< 3 runs) | No history yet. May be first crawl, offseason, or auth/config issue. | Proceed to Step 2. |
| `total_games > 0` AND `most_recent_game < 30 days ago` | Recent games exist — this run may be a timing or scheduling gap. | Wait for next scheduled run. If 0 games again → proceed to Step 2. |
| `total_games > 0` AND `most_recent_game` is 30–90 days ago | Possibly offseason. Cross-reference with Section 1 season window. | If in-season: proceed to Step 2. If offseason: log as expected and monitor. |
| `total_games > 0` AND `most_recent_game > 90 days ago` during active season | Likely auth failure or config change. | Proceed to Step 2 immediately. |

---

### Step 2: Check Cross-Org Baseline (Auth Health Check)

Are OTHER org-IDs returning games in the same crawl run? If yes, the crawl itself is working and the problem is specific to this org-ID.

```sql
-- Are other org-IDs returning games in the same crawl window?
SELECT source_org_id,
       source_name,
       COUNT(*) AS games_ingested,
       MAX(created_at) AS last_ingested
FROM games
WHERE created_at >= NOW() - INTERVAL '24 hours'
GROUP BY source_org_id, source_name
ORDER BY last_ingested DESC
LIMIT 20;
```

**Interpret the result:**

| Result | Interpretation | Next step |
|--------|---------------|-----------|
| Other org-IDs have recent `created_at` timestamps | Crawl is globally healthy. Problem is specific to this org-ID. | Proceed to Step 3. |
| No org-IDs have recent `created_at` timestamps (< 5 games total in last 24h) | Global crawl failure or global auth expiry. Do not diagnose individual org-IDs until global issue is resolved. | See Section 4 (Auth Expiry Detection). |
| Only new Stage 2 org-IDs are returning 0; all legacy org-IDs are returning games | Config error specific to new entries. | Proceed to Step 3 for each new org-ID showing 0. |

---

### Step 3: Classify the Cause

Combine Step 1 result, Step 2 result, and the Section 1 season window to classify:

| Step 1 Result | Step 2 Result | Season Window | Classification | Action |
|---------------|---------------|---------------|---------------|--------|
| New org-ID, 0 history | Other orgs returning games | Active season | **Config error** — wrong org-ID or incorrect query parameters | Review config entry against browser lookup data. Check `source_org_id` format and value. Reference `Infrastructure/GotSport Org-ID Master Reference — April 2026.md`. |
| New org-ID, 0 history | Other orgs returning games | Offseason | **Expected inactive** | Log as "offseason — expected." Set a calendar reminder to re-check at season start. |
| Previously active org-ID | Other orgs returning games | Active season | **Auth failure** for this specific org-ID | Trigger targeted re-authentication at `system.gotsport.com`. Check if org requires a different session, permission level, or org-ID format. |
| New or existing org-ID | No orgs returning games | Any | **Global crawl/auth failure** | Do not classify individual org-IDs. See Section 4. |
| Previously active org-ID | Other orgs returning games | Offseason | **Expected seasonal gap** | Log as expected. No action. Monitor at season start. |

**Record the classification in the FORGE briefing:** "Org-ID `[id]` — 0 games — classified as `[config error / expected inactive / auth failure / global failure]` — action taken: `[action]`."

---

## Section 3: Removal Threshold

**Do not remove an org-ID based on a single 0-game run.** The removal threshold requires all three conditions to be met:

1. The org-ID has returned 0 games for **3 consecutive months during its active season** (as defined in Section 1).
2. The org-ID value has been confirmed correct (not a config error) via browser lookup.
3. SENTINEL has reviewed and approved the removal.

**When FORGE recommends removal**, file a Queue item in `10 - Agents/FORGE/Queue/` with:

```
Org-ID: [value]
Entity name: [name]
Historical total game count: [total]
Games last 90 days: [count]
Season status at time of recommendation: [active / offseason]
Consecutive 0-game runs during active season: [count]
Org-ID confirmed correct: [YES / NO — reference]
Diagnostic steps completed: [Step 1 result / Step 2 result / Step 3 classification]
Recommendation: REMOVE / HOLD / INVESTIGATE
```

**Do not remove during offseason.** An org-ID returning 0 games in July may have 500 games in October.

---

## Section 4: Auth Expiry Detection (Global)

The GotSport browser session may expire silently. Global auth expiry is distinct from individual org-ID inactivity.

**Signs of global auth expiry:**
- Multiple org-IDs simultaneously return 0 games in a single crawl run when prior runs were healthy
- Step 2 query shows 0 or near-0 total games across all org-IDs for the past 24 hours
- Crawl logs contain HTTP 401 errors, session token errors, or authentication failure messages

**Response protocol if global auth expiry is suspected:**

1. Presten re-authenticates at `system.gotsport.com` (same session used for browser lookup). Reference: `Infrastructure/GotSport Browser Lookup Session Prep Package.md`.
2. FORGE restarts the crawl in the next scheduled run window (do not force an immediate re-run against all org-IDs — this can spike rate limits).
3. FORGE files alert in briefing:

```
GotSport auth expiry detected: [date]
Resolution: Presten re-authenticated at [date/time]
Recovery: All org-IDs expected to return to normal ingestion by [expected date based on next scheduled crawl]
Org-IDs confirmed recovered: [list after next run]
```

**Do not diagnose individual org-IDs until global auth is restored.** A global auth failure will cause every org-ID to fail — diagnosing individual org-IDs during a global outage produces misleading classification results.

---

## Section 5: Quick Diagnostic Reference Card

```
0 GAMES FOR ORG-ID → START HERE

Step 1: Run historical count query
  └─ total_games = 0 AND new org-ID → Step 2
  └─ most_recent_game < 30d ago → wait one run, then Step 2 if still 0
  └─ most_recent_game 30-90d ago AND offseason → log as expected, monitor
  └─ most_recent_game > 90d ago AND active season → Step 2 (urgent)

Step 2: Run cross-org baseline query
  └─ Other orgs returning games → Step 3 (org-specific problem)
  └─ No orgs returning games → Section 4 (global failure)

Step 3: Classify
  └─ New org + active season + other orgs healthy → CONFIG ERROR → fix config
  └─ New org + offseason + other orgs healthy → EXPECTED → log and monitor
  └─ Previously active + active season + other orgs healthy → AUTH FAILURE → re-auth this org
  └─ Any org + no orgs returning → GLOBAL FAILURE → Section 4
```

---

## References

- `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` — the complete org-ID list this protocol monitors; Section 1 tracks active status
- `Infrastructure/Stage 2 Expansion — SENTINEL Authorization Gate.md` — Stage 2 authorization this protocol informs
- `Infrastructure/Daily Pipeline Morning-After Validation Runbook.md` — per-run health check this protocol feeds into
- `Infrastructure/GotSport Browser Lookup Session Prep Package.md` — source of org-ID confirmation data; used to verify config correctness in Step 3

*FORGE — 2026-04-25*
