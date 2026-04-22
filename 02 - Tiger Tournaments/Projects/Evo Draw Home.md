---
title: Evo Draw Home
aliases: [Evo Draw, Dashboard, MOC]
tags: [project, moc, evo-draw, youth-soccer]
created: 2026-04-21
updated: 2026-04-21
status: active
parent: "[[Tiger Tournaments]]"
---

# Evo Draw — Map of Content

> Part of [[Tiger Tournaments]] | Back to [[000 - Presten Manthey]]

**Evo Draw** is a youth soccer data platform that aggregates game results from multiple sources, deduplicates and normalizes them, and computes national rankings across leagues. Built and operated by **Presten Manthey** under **[[Tiger Tournaments]]**.

---

## Platform Stats (as of 2026-04-21)

| Metric            | Value       |
| ----------------- | ----------- |
| Total Games       | ~1,650,000  |
| Visible Games     | ~1,640,000  |
| Teams             | 144,676     |
| Team Merges       | 27,820      |
| Events            | ~10,500     |
| Data Sources      | 7           |

---

## Major Sections

### Data Ingestion
- [[Data Sources Overview]] — all 7 sources with volume breakdowns
- [[GotSport]] — primary source (API + HTML scrapers)
- [[TGS AthleteOne]] — ECNL/club data via JSON API
- [[Sinc Sports]] — bulk-imported, Cloudflare-blocked
- [[MLS NEXT Source]] — tagged MLS NEXT subset

### Scrapers
- [[GotSport API Scraper]] — parallel crawler, primary ingestion
- [[GotSport HTML Scraper]] — server-rendered HTML fallback
- [[TGS Scraper]] — AthleteOne JSON crawler

### Pipeline
- [[Data Pipeline]] — full 9-step processing chain
- [[Dedup Strategy]] — cross-source and within-source deduplication
- [[Parsing Rules]] — age, gender, round, season parsing

### Leagues & Hierarchy
- [[League Hierarchy]] — full tier system (Elite → High Comp → Competitive)
- [[MLS NEXT]] — boys #1 league
- [[ECNL]] — #2 boys / #1 girls (tied)
- [[ECNL RL]] — regional feeder league
- [[Pre-ECNL]] — U10-U12 developmental
- [[GA]] — Girls Academy, Tier 1 girls league

### Rankings
- [[Ranking Engine]] — Elo/Glicko v7 with 9 phases
- [[Division Calibration]] — DSS division placement system

### Strategy
- [[Strategic Insights]] — cross-domain analysis from the knowledge graph (12 insights, action items)
- [[Action Items Tracker.base|Action Items Tracker]] — .base database view of strategic action items

### Dashboards & Views
- [[Evo Draw Dashboard.base|Evo Draw Dashboard]] — .base database view of all project notes (freshness, connections, hub scores)
- [[Platform Architecture.canvas|Platform Architecture]] — .canvas visual flowchart of the full data pipeline
- [[Vault Dashboard.base|Vault Dashboard]] — .base vault-wide view (in vault root)

### Events
- [[Dallas Spring Showcase]] — May 16-18, 2026 at MoneyGram Park

### Infrastructure
- [[Database Schema]] — PostgreSQL tables and key columns
- [[Server and Frontend]] — Node.js + React/Vite stack

### Knowledge Base
- [[Age Groups and Birth Years]] — mapping table + format rules
- [[Tournament Landscape]] — major national events calendar
- [[College Recruiting]] — NCAA pathway and showcase strategy
- [[Governing Bodies]] — US Club Soccer vs USYS
- [[Competitors]] — ranking system landscape and our 7 gaps
- [[Cross-League Analysis]] — 575K game win-rate study
- [[Data Quality]] — completeness metrics and known gaps
- [[Scope Rules]] — what we include/exclude
- [[Recent Changes 2024-2026]] — industry shifts
- [[Team Name Matching]] — GotSport name mismatch resolution

### Templates
- [[Daily Note - Evo Draw]] — daily operations template

---

## Technical Stack

| Component  | Detail                                                  |
| ---------- | ------------------------------------------------------- |
| Code       | `~/Desktop/Projects/evo-draw/`                          |
| GitHub     | https://github.com/MantheyPH2/evo-draw                 |
| Database   | PostgreSQL `youth_soccer` @ localhost:5432, user: `p`   |
| Frontend   | React + Vite at `frontend-v2/`, branded emerald/cyan    |
| Server     | Node.js on port 3000 (`server.js`)                      |
| Build      | `cd frontend-v2 && npm run build && cd .. && rm -rf dist && cp -r frontend-v2/dist .` |

---

## Key Dates

| Date           | Event                                          |
| -------------- | ---------------------------------------------- |
| May 16-18 2026 | [[Dallas Spring Showcase]]                     |
| June 2026      | [[ECNL]] migrates AthleteOne to [[GotSport]]  |
| 2026-27 season | Age group shift: birth year to school year     |

> [!tip] Quick Start
> To run the full pipeline: `bash run-pipeline.sh` from the project root. For overnight batch: `bash run-overnight.sh`. See [[Data Pipeline]] for step details.
