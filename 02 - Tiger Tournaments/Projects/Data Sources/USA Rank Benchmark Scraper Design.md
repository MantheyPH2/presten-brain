---
title: USA Rank Benchmark Scraper Design
tags: [data-sources, competitor-intel, usa-rank, benchmarking, scraper-design, evo-draw]
created: 2026-04-23
updated: 2026-04-23
status: design-complete
author: FORGE
---

# USA Rank Benchmark Scraper Design

**Assigned by:** SENTINEL  
**Date:** 2026-04-23  
**Purpose:** A benchmarking and gap-detection tool that systematically scrapes publicly accessible USA Rank team pages to cross-reference against our own rankings, identify coverage gaps, and validate our ranking methodology relative to theirs.  
**This is NOT for production rankings** — it feeds SENTINEL-level quality auditing only.

---

## 1. URL Structure Documentation

### Confirmed URL Patterns

| Page Type | Pattern | Example |
|-----------|---------|---------|
| Ranking finder | `usarank.com/rank/view` | `https://www.usarank.com/rank/view` |
| Team rankings by age/gender | `usarank.com/rank/view?agegroup={ag}&gender={g}` | `/rank/view?agegroup=U13&gender=Girls` |
| Team detail page | `usarank.com/teamlist/{team_id}/{year}/{agegroup}` | `/teamlist/4892/2026/U13G` |
| Schedule/results page | `usarank.com/usaschedules/{team_id}/{year}/{agegroup}` | `/usaschedules/4892/2026/U13G` |
| Internal event normalization | `usarank.com/usarankevents` | Not publicly navigable |

### Age Group Encoding

Based on observed URLs in the competitive intel file and USA Rank's public ranking finder:
- Format: `{age}{gender_initial}` — e.g., `U13G`, `U14B`, `U15G`
- Year parameter: 4-digit birth year or season year depending on page type
- The `/rank/view` finder page accepts URL query params and returns paginated ranked team lists by age/gender

### Team ID Enumeration

USA Rank team IDs are numeric and assigned internally (not GotSport team IDs). To enumerate team IDs without already knowing them:
1. Load `/rank/view?agegroup=U13&gender=Girls` — this returns a ranked list with team links
2. Extract `href` attributes from team links — each links to `/teamlist/{id}/...`
3. Paginate through the ranked list to collect all team IDs for that age/gender

**Hypothesis (to verify with Presten before build):** The ranking finder renders server-side HTML with anchor tags — IDs are discoverable by scraping the list pages rather than requiring prior knowledge of team IDs.

---

## 2. Data Field Inventory

### Team Page (`/teamlist/{id}/{year}/{agegroup}`)

| Field | Type | Notes |
|-------|------|-------|
| Team name | string | Full name including club and age designation |
| Current rank | integer | Rank position within age/gender group |
| Rank band | string | Gold / Silver / Bronze / Red / Blue / Green |
| Rating/score | decimal | "Measured in goals" per their methodology |
| Rank change (week-over-week) | integer | `+N` or `-N` from previous week |
| Schedule strength | decimal | Average opponent rating over last year |
| Recent games list | array | Last 20 games: opponent, score, date, event name, opponent rank |
| Per-game fields: opponent name | string | |
| Per-game fields: opponent rank | integer | |
| Per-game fields: score | string | `W 3-1`, `L 0-2`, `T 1-1` |
| Per-game fields: date | date | |
| Per-game fields: event name | string | Critical — identifies which of their 400+ sources the game came from |
| Per-game fields: result vs. expected | string | Implied by rating change — not always shown explicitly |

### Schedule Page (`/usaschedules/{id}/{year}/{agegroup}`)

| Field | Type | Notes |
|-------|------|-------|
| Full game list | array | All games in ranking window |
| Opponent team name | string | Links to opponent's own team page |
| Opponent team id | integer | Embedded in link `href` |
| Final score | string | |
| Date | date | |
| Event name | string | |

**Key observation:** Event names on USA Rank schedule pages directly identify which of their 400+ sources the game came from. Collecting these event names is one of the highest-value outputs of this scraper — it tells us which sources they have that we don't.

---

## 3. Technical Feasibility Assessment

### HTML vs. JavaScript
USA Rank appears to be **server-rendered HTML** based on:
- The site loads team data without obvious client-side API calls in page source
- URLs are human-readable and stateful — no hash routing
- The competitive intel file's sample data was presumably captured via direct page viewing

**To confirm:** Presten should `curl https://www.usarank.com/teamlist/4892/2026/U13G` and verify team data appears in the HTML response body (not as `{}` placeholder waiting for JS). If JSON data appears in the response, the parser switches from HTML → JSON extraction.

### Bot Detection
- USA Rank does not appear to use Cloudflare (unlike SincSports post-2022)
- **robots.txt must be checked before crawling:** `curl https://www.usarank.com/robots.txt`
- Expected: standard user-agent restrictions. We will honor all `Disallow` rules.
- If `robots.txt` disallows `/teamlist/` or `/rank/`, this design stops here and SENTINEL is notified.

### Safe Crawl Rate
- **Target: 2 requests per second maximum** (500ms delay between requests)
- **User-Agent:** `Evo-Draw Research Bot 1.0 (contact: presten@tigertournaments.com)` — transparent, includes contact
- This is a weekly batch job, not real-time. Slow and polite is the correct posture.

### Scope Estimate
| Task | Requests |
|------|---------|
| Pull ranked team list for 1 age/gender group | ~5–20 pages (assuming 50 teams/page, up to 1,000 teams) |
| Pull team pages for all ranked teams in 1 age/gender group | ~200–500 requests (one per team) |
| All age groups (U11–U17) × both genders = 14 combos | ~1,400–7,000 team page requests |
| Schedule pages for all teams | ~1,400–7,000 additional requests |
| **Full sweep total** | **~3,000–14,000 requests at 500ms = 25–120 minutes** |

A full weekly sweep is feasible in a 2-hour window. Recommend running Sunday nights after the weekly full pipeline sweep.

---

## 4. Comparison Framework Design

### Team Matching Logic
Match USA Rank teams to Evo Draw teams by name using the same fuzzy-match approach as `team_merges`:

```javascript
// Matching confidence scoring
function matchUSARankTeam(usaRankName, edrTeams) {
  // Step 1: Exact match on normalized name (lowercase, strip punctuation)
  // Step 2: Trigram similarity score > 0.85 → high confidence match
  // Step 3: Club name + age group token match → medium confidence
  // Step 4: No match found → flag as "missing team"
  
  return { team_id, confidence };
}
```

### Delta Thresholds and Flags

| Condition | Flag Type | Threshold |
|-----------|-----------|-----------|
| Ranking divergence | `RANK_DIVERGENCE` | USA Rank rank differs from Evo Draw rank by > ±50 |
| Large divergence | `RANK_DIVERGENCE_MAJOR` | Difference > ±200 |
| Missing team | `MISSING_FROM_EDR` | USA Rank has ranked team we have no record of |
| Missing source | `SOURCE_GAP` | USA Rank game event name doesn't match any of our 7 sources |
| Team in USA Rank, no games in EDR | `DATA_GAP` | Team matched by name but we have <5 games for them |

### Weekly Report Output
The comparison produces a report that SENTINEL reviews weekly:
- Top 20 teams by ranking divergence (where we disagree most with USA Rank)
- Count of missing teams by age group
- New event names seen in USA Rank game histories that don't match our source list (source gap detection)

---

## 5. Output Schema — Revised SQL DDL

```sql
CREATE TABLE usa_rank_benchmark (
  id                    SERIAL PRIMARY KEY,
  scraped_at            TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  age_group             VARCHAR(10) NOT NULL,
  gender                CHAR(1) NOT NULL,          -- 'B' or 'G'
  usa_rank_team_id      INTEGER,                   -- USA Rank's internal team ID
  usa_rank_team_name    VARCHAR(200) NOT NULL,
  usa_rank_rank         INTEGER,
  usa_rank_band         VARCHAR(20),               -- Gold/Silver/Bronze/Red/Blue/Green
  usa_rank_rating       NUMERIC(6,3),              -- their goal-diff-based rating
  usa_rank_schedule_str NUMERIC(6,3),              -- schedule strength (avg opponent rating)
  evo_draw_team_id      INTEGER REFERENCES teams(id) ON DELETE SET NULL,
  evo_draw_rank         INTEGER,
  rank_delta            INTEGER,                   -- usa_rank_rank - evo_draw_rank
  matched               BOOLEAN NOT NULL DEFAULT false,
  match_confidence      NUMERIC(3,2),              -- 0.00-1.00
  match_method          VARCHAR(20),               -- 'exact'|'fuzzy'|'manual'|null
  flags                 TEXT[],                    -- array of flag codes from comparison framework
  source_gaps           TEXT[],                    -- event names from USA Rank not in our source list
  run_id                VARCHAR(20)                -- ISO date of the weekly run, e.g. '2026-04-27'
);

CREATE INDEX idx_urb_run_age ON usa_rank_benchmark (run_id, age_group, gender);
CREATE INDEX idx_urb_rank_delta ON usa_rank_benchmark (ABS(rank_delta) DESC NULLS LAST) 
  WHERE matched = true;
CREATE INDEX idx_urb_flags ON usa_rank_benchmark USING GIN (flags);
```

**Changes from task spec:**
- Added `usa_rank_team_id` (their internal ID for linking between runs)
- Added `usa_rank_rating` and `usa_rank_schedule_str` (valuable for methodology comparison)
- Added `match_method` (needed to filter high-confidence matches in reports)
- Added `flags` as `TEXT[]` and `source_gaps` as `TEXT[]` for queryable flag sets
- Added `run_id` (enables trend tracking across weekly runs)
- Changed `rank_delta` definition: positive = USA Rank ranks them higher than we do

---

## 6. Crawl Scope Estimate

| Scope | Teams | Page Requests | Time @ 500ms |
|-------|-------|--------------|--------------|
| 1 age/gender group | ~200–500 | ~400–1,000 | ~3–8 min |
| All U11–U17 × both genders (14 groups) | ~2,800–7,000 | ~6,000–15,000 | ~50–125 min |
| With schedule pages (2x requests) | ~5,600–14,000 | ~12,000–30,000 | ~100–250 min |

**Recommendation:** Start with team pages only (no schedule pages) for the first 3 runs. Schedule pages add significant volume; confirm team matching and ranking comparison is valuable before doubling the crawl.

---

## 7. Implementation Effort Estimate

| Component | Hours | Notes |
|-----------|-------|-------|
| Confirm robots.txt + curl test URL structure | 0.5 | Presten validates before build starts |
| Ranked list page scraper (team ID enumeration) | 4–6 | Paginated list → team ID array |
| Team page scraper (rating, rank, game list) | 4–6 | HTML parser for each team page |
| Fuzzy name matching vs. Evo Draw teams | 4–6 | Reuse or adapt team_merges matching logic |
| DB insertion + benchmark table | 2–3 | Schema above is ready to run |
| Weekly report generation | 3–4 | SQL queries → Markdown report |
| Cron wiring + run orchestration | 1–2 | Sunday night after full pipeline |
| **Total** | **18–27 hours** | 3–4 focused development days |

**DSS feasibility:** The full scraper is unlikely to be ready before DSS (May 16, 2026). A stripped-down version — just team ID enumeration + rank capture for one age group — could be built in 6–8 hours and would produce the first comparison report. Recommend treating this as post-DSS unless SENTINEL elevates priority.

---

## 8. Ethical and Legal Considerations

**Boundary assessment:**
- All data is publicly accessible without login — no authentication bypass
- We will honor `robots.txt` completely — if pages are disallowed, we stop
- User-agent string identifies us transparently including contact email
- Crawl rate (500ms = 2 req/sec) is respectful of their server load
- Data is used internally for methodology comparison and gap detection — not republished
- Same legal standing as any researcher analyzing a public competitor's published rankings

**Boundary we will NOT cross:**
- No crawling beyond what `robots.txt` permits
- No credential stuffing, rate limit circumvention, or IP rotation to avoid blocks
- If they send a `429` or block us: stop immediately and document in the run log

**Presten to verify:** Review their Terms of Service for any explicit prohibition on automated access to public ranking data. If ToS prohibits scraping even of public pages, consult before proceeding.

---

## Related Notes
- [[USA Rank Competitive Intel]] — source of URL patterns and sample teams
- [[Data Sources Overview]] — current source architecture for comparison
- [[Ranking Engine]] — Evo Draw methodology, for understanding what rank deltas mean
- [[Dedup Strategy]] — team_merges matching logic to reuse for fuzzy name matching

---

*Designed by FORGE — 2026-04-23*
