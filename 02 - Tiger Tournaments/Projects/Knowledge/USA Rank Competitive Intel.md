---
title: USA Rank / USA Sport Statistics — Competitive Intelligence
tags: [competitor, intel, usa-rank, evo-draw]
created: 2026-04-22
updated: 2026-04-22
source: Public website research
---

# USA Rank / USA Sport Statistics — Competitive Intel

> Part of [[Competitors]] | [[Evo Draw Home]]

**Goal: Match everything they do, then beat them.**

---

## Where They Get Games

- Publicly claim **400+ different source websites**
- Add events/results daily
- Users can submit missing result links
- Official posted results only (not parent-entered scores)
- "It does not matter who the web vendor is" — source-agnostic ingestion

### Confirmed Source Families

| Source Family | Confidence | Evidence |
|--------------|-----------|----------|
| **SincSports / SoccerInCollege** | High | FAQ says users log in with SincSports. Login page uses SoccerInCollege credentials. |
| **SnapSoccer** | High | Repeated event references: "SnapSoccer Fall Playdates", "Spring Playdates" |
| **USYS / State Associations** | High | State cups, Presidents Cup, state leagues, NPL in event names |
| **MLS NEXT / Academy** | High | "National Academy Championships Wk 1 MLS Next" in events |
| **USA Rank Internal** | High | usarank.com/usarankevents — their normalized internal layer |
| **GotSport** | Unconfirmed | Not directly proven on their public pages yet |
| **TGS / AthleteOne** | Unconfirmed | Not directly proven yet |
| **Standalone tournaments** | Certain | Long tail of 400+ sites, user-submitted |

### What This Means for Evo Draw
We have 7 sources. They have 400+. The gap is massive. Priority actions:
- [ ] Build a source-submission system (let tournament directors add their events)
- [ ] Add SnapSoccer as a data source
- [ ] Add SincSports/SoccerInCollege if accessible
- [ ] Add USYS State Cup results
- [ ] Build a long-tail tournament ingestion framework

---

## How They Rank Teams

### Core Methodology (Publicly Stated)
- Rankings update **daily**
- Use the **last 20 games in the last year**
- **Goal difference** based rating
- **Recency weighting** — recent games matter more
- **Opponent strength weighting** — games against similar strength count more
- Teams rise if they **do better than predicted**
- Rating is "measured in goals"
- **League quality is NOT hardcoded** — emerges from inter-league results
- Schedule rating = average opponent rating over last year
- Teams need **6-10 recent results** against ranked teams to appear
- **7 months inactivity** = drop from rankings
- Penalty shootouts = ties at full time

### Ranking Bands
| Band | Rank Range |
|------|-----------|
| Gold | 1-100 |
| Silver | 101-250 |
| Bronze | 251-400 |
| Red | 401-700 |
| Blue | 701-1050 |
| Green | 1051+ |

### Club Rankings
- Average of top team ratings
- Ages U11 through U17
- Club needs teams in at least 5 age groups

### How This Compares to Evo Draw
| Feature | USA Rank | Evo Draw |
|---------|----------|----------|
| Algorithm | Goal-difference based, last 20 games | Elo/Glicko v7, 9+1 phases |
| Update frequency | Daily | Batch (overnight pipeline) |
| Sources | 400+ | 7 |
| League weighting | Emergent from results | Calibration values (hardcoded) |
| Activity window | Last year, 20 games | Full history |
| Inactivity drop | 7 months | Not documented |
| Rank bands | Gold/Silver/Bronze/Red/Blue/Green | None currently |

---

## Match Predictions

- **No public standalone prediction tool** found
- They use an internal **expectation model** (implied by "teams rise/fall vs predicted")
- Event pages show: opponent ranks, before/after movement, upset rates, strength indicators
- Best inference: they convert rating gaps into expected goal margins, compare actual vs expected

### What Evo Draw Already Has That They Don't
- Public H2H violations endpoint
- Explicit match prediction capability (if built into UI)
- Cross-league win rate analysis (575K games)

---

## Public Ranked Team Data

### Entry Points
- `usarank.com/rank/view` — ranking finder
- `usarank.com/teamlist/{id}/{year}/{agegroup}` — team pages
- `usarank.com/usaschedules/{id}/{year}/{agegroup}` — schedule pages

### Sample Ranked Teams Captured

**U13 Girls:**
- The Football Academy NJ 2012 Girls Black, #4
- Wall Soccer Club Wall Elite Porto, #12
- FC Stars G2012 ECNL White, #121
- NE Surf G2012 State Navy, #198

**U14 Girls:**
- Maryland Rush Azzurri 2011, #144
- NJ Premier FC G2011, #810

### Data Extraction Opportunity
Ranked teams are exposed page-by-page. A structured scraper of `usarank.com/teamlist/` pages could extract:
- Team names and ranks by age group
- Before/after rank movements
- Opponent ranks and scores
- Event strength metrics

---

## Action Items — Match Then Beat

### Phase 1: Match (What they have that we don't)
- [ ] Daily ranking updates (we do batch overnight — match their cadence)
- [ ] 400+ source ingestion (we have 7 — build source submission framework)
- [ ] Rank bands (Gold/Silver/Bronze/etc — implement in UI)
- [ ] Public ranked team pages by age group (we have the data, need the UI)
- [ ] Club rankings (average top team ratings, U11-U17)
- [ ] Event strength ratings (High/Avg/Low/Spread/Upsets)
- [ ] User-submitted missing result links

### Phase 2: Beat (Our advantages to weaponize)
- [ ] Cross-league analysis (575K games — they don't have this publicly)
- [ ] Explicit match predictions with win probabilities
- [ ] Methodology transparency page (they're opaque)
- [ ] Division placement as a service (DSS model)
- [ ] College recruiting integration
- [ ] Real-time tournament bracket generation
- [ ] Open team merge verification (our 27,820 merges is a moat)

### Phase 3: Dominate
- [ ] Build a public API
- [ ] Mobile app
- [ ] Coach/parent notification system
- [ ] Automated event detection and ingestion
- [ ] AI-powered scouting reports

---

*Intel gathered April 22, 2026 from public sources only.*
