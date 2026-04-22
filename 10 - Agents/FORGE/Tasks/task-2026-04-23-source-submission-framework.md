---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-23
status: pending
priority: high
---

# Task: Design Source Submission Framework

## Context

USA Rank allows users to submit missing result links and claims 400+ source websites. We have 7 sources and no mechanism for external parties to contribute. This is one of the largest competitive gaps identified in our intel brief.

The insight is that we don't need to build 400 scrapers — we need tournament directors and users to bring events to us. This is a community-sourced data moat.

Read before starting:
- `02 - Tiger Tournaments/Projects/Knowledge/USA Rank Competitive Intel.md` — "Where They Get Games" section and Phase 1 action items
- `02 - Tiger Tournaments/Projects/Data Sources/Data Sources Overview.md` — current source architecture and priority system
- `02 - Tiger Tournaments/Projects/Pipeline/Data Pipeline.md` — how Step 0 feeds the pipeline
- `02 - Tiger Tournaments/Projects/Infrastructure/Server and Frontend.md` — current server (Node.js, port 3000, 5 endpoints)
- `02 - Tiger Tournaments/Projects/Infrastructure/Database Schema.md` — games table structure

## What You Need to Design

### 1. Submission Interface (What Users See)
- What does a tournament director submit? (event URL, source type, age groups, date range?)
- What does a parent/coach submit when they notice a missing result?
- Should submission be a web form, email, or both?
- What confirmation do submitters receive?

### 2. Submission Queue (Backend)
- Design the database table for storing submissions (columns: id, submitted_by, url, source_type, event_name, status, notes, created_at, processed_at)
- What statuses does a submission move through? (pending → reviewed → scraping → imported → rejected)
- Who reviews submissions? (Presten manually, or automated triage first?)

### 3. Source Classification Logic
- When a URL is submitted, how do we classify the source? (GotSport event? SnapSoccer? Unknown?)
- Can we auto-detect the web vendor from the URL pattern?
- Known patterns: gotsport.com, snapsoccer.com, sincsports.com, affinitysoccer.com, gotsoccer.com, tourneymachine.com
- Design a source-detection function that maps URL → source family

### 4. Scraper Routing
- If submission is a GotSport event URL → route to existing GotSport HTML Scraper
- If submission is an unknown source → flag for manual scraper build
- Design a routing table: source_family → scraper_script → priority

### 5. Notification and Feedback Loop
- How does submitter know their event was added?
- Should we track which submitter "owns" submitted events? (Community credit system?)
- What email/notification mechanism fits the current stack?

## What Done Looks Like

Produce a design document at:
`02 - Tiger Tournaments/Projects/Product & Planning/Source Submission Framework Design.md`

The document must include:
1. User flow diagram (text-based is fine: Submitter → Form → Queue → Triage → Scraper → DB)
2. Database schema for the submissions table (SQL CREATE TABLE statement)
3. Source detection logic (pseudocode or JS function skeleton)
4. Scraper routing table (source family → scraper)
5. API endpoint design: `POST /api/submit-source` — request body, response, validation rules
6. Priority and effort estimates for each component (Low/Med/High effort)
7. MVP scope: what is the minimum version that lets Presten start accepting submissions before DSS?

## Success Criteria

- Design document written with all 7 sections complete
- SQL schema is production-ready (Presten can run it against `youth_soccer` DB without modification)
- MVP scope is clear — Presten knows exactly what to build first
- The system is compatible with the existing Node.js server and PostgreSQL setup
