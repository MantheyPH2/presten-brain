---
title: Evo Draw — DSS Coverage Narrative
tags: [infrastructure, dss, coverage, reporting, evo-draw, forge]
created: 2026-04-25
updated: 2026-04-25
author: FORGE
status: filed — update if browser session or Stage 1 completes before May 9
task: task-2026-04-25-dss-coverage-narrative
source: "Infrastructure/Pre-DSS Source Coverage Report — May 2026.md"
---

# Evo Draw — DSS Coverage Narrative

> **Audience:** DSS decision-makers. Non-technical. Soccer-knowledgeable.
> **Purpose:** Presten's talking track for coverage questions during the May 16 DSS presentation.
> **Claim label:** Option A (confirmed as of 2026-04-25 — browser session not yet completed, Stage 1 crawl not yet run). If both complete before May 9, update to Option B per the note in Section 1.

---

## Section 1: What Evo Draw Covers

Evo Draw currently includes rankings for competitive travel soccer teams across 7 confirmed-active national and regional leagues, covering both Boys and Girls age groups from U10 through U19 in the United States. Data is updated on each pipeline run — beginning May 1, 2026, this becomes a daily automated process. The platform tracks teams from the elite national level down through regional competitive play.

**Option B note:** If the browser lookup session and Stage 1 TX crawl both complete before May 9, this section may be updated to: "Evo Draw includes rankings across [expanded count] leagues, including USYS state association play in [confirmed states]." Do not make this claim until Stage 1 results and browser session confirm the expanded scope.

---

## Section 2: Key Leagues Covered

- **MLS NEXT** — elite national Boys competitive development league; 27,600+ games ingested
- **ECNL Girls** — premier national club league, Girls; primary source TGS AthleteOne through June 2026
- **ECNL Boys** — premier national club league, Boys; same source and timeline as Girls
- **Girls Academy (GA)** — national Girls competitive league; GA ASPIRE tier recalibration executing April 28, fully corrected post-session
- **GA Boys** — national Boys development pathway under the Girls Academy structure
- **ECNL RL Girls** — ECNL Regional League, Girls tier (~300 clubs)
- **ECNL RL Boys** — ECNL Regional League, Boys tier (~400 clubs)

Additional active leagues (ingestion recency verification in progress ahead of May 9):
- **NPL** (National Premier League) — Boys and Girls
- **DPL** (Development Player League) — Girls, GA college-placement pathway
- **Pre-ECNL** — Boys and Girls, U10–U12 age groups

---

## Section 3: Coverage Gaps (Honest)

Evo Draw is focused on competitive travel soccer. It does not include:

- **USYS state association unranked divisions** — state cup and state league play below the ranked tier. This is the largest volume gap. The expansion path is defined and in progress; live data is not available for DSS on May 16.
- **Illinois and Midwest Affinity states** — Illinois Youth Soccer (IYSA) and neighboring Affinity Soccer platform states use a different registration system. Integration requires new engineering; no path before June 2026.
- **High school soccer** — outside scope. Evo Draw covers competitive club travel programs, not interscholastic competition.
- **Recreational leagues** — outside scope. The platform is designed for competitive travel rosters and does not include house or recreational registration data.

---

## Section 4: Coverage Growth — What's Coming

Following the May 1 pipeline deployment, Evo Draw will begin expanding USYS state association coverage via a staged org-ID crawl expansion. Priority 1 states — California, Florida, New York, New Jersey, Washington, Pennsylvania, and Ohio — are targeted first, representing the majority of U.S. competitive youth soccer registration outside Texas. Combined with Priority 2 states (CO, VA, MD, NC, GA, AZ, MA, TN, SC), the expansion adds an estimated 60,000–120,000 games across approximately 12 state associations at spring peak season volume.

Timeline is contingent on two actions completing before May 9: the GotSport browser lookup session (org-ID confirmation for USYS states) and the TX Stage 1 test crawl (validation gate for full expansion). If both complete on schedule, full USYS state expansion can be authorized before May 16.

---

## Section 5: Data Freshness

GotSport-sourced games — covering MLS NEXT, GA, ECNL RL, NPL, DPL, and Pre-ECNL — update on each pipeline run. Beginning May 1, this is a daily automated schedule; prior to May 1, ingestion runs on-demand. ECNL games (Girls and Boys) are sourced from TGS AthleteOne on TGS's reporting schedule through June 2026, when ECNL migrates to GotSport ingestion.

Rankings recompute on every ingestion run — there is no separate weekly batch. A game ingested Monday appears in rankings on Monday's run. The lag between a game being played and appearing in Evo Draw rankings reflects the time between the result being reported to the source platform and the next pipeline execution: GotSport-sourced results typically appear within 24–48 hours of organizer reporting. TGS AthleteOne ECNL results follow TGS's own reporting cadence, which varies by event.

---

## References

- [[Pre-DSS Source Coverage Report — May 2026]] — source data for all claims above
- `Rankings/DSS Ranking Accuracy Claims — Authorized Spec.md` — ELO accuracy claims (must remain consistent with this narrative)
- `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` — org-ID confirmation status
- `Infrastructure/GotSport Org-ID Expansion Stage 2 — Pre-Authorization Criteria.md` — Stage 2 expansion scope

*FORGE — 2026-04-25*
