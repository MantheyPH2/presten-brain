---
title: DSS Demo Talking Points — Rankings Features
type: demo-script
author: ELO
date: 2026-04-26
status: ready — confirm Boys section after Option A results
due: 2026-05-07
related:
  - "[[DSS Ranking Accuracy Claims — Authorized Spec]]"
  - "[[Rank Bands — Implementation Plan]]"
  - "[[USA Rank Delta Interpretation Spec — April 2026]]"
  - "[[DSS Competitive Accuracy Narrative]]"
  - "[[Event Strength Phase 1 — Rankings Impact Preview]]"
tags: [dss, demo, talking-points, rankings, rank-bands, event-strength, evo-draw]
---

# DSS Demo Talking Points — Rankings Features

> **Purpose:** Rehearsal-ready talking points for Presten to use at DSS on May 16. Read this, practice it, walk in with it memorized. Written in Presten's voice — present tense, direct, conversational.
>
> **Before the demo:** Confirm Boys section applies based on Option A verdict by May 9. Confirm team examples still in correct bands after April 29 recompute (see Section 3 update note).

---

## Section 1: The Core Claim (30-Second Version)

"We rank youth soccer teams based on who they beat, not just that they beat someone. Every win in our system is weighted by two things: how strong the opponent is, and how competitive the event they played in was. A win over an ECNL program at an ECNL event is worth more than a win over the same team at a local tournament — because the competition context is different. That's the key difference from USA Rank and GotSoccer. They track results. We measure *what those results mean* relative to the competitive environment they happened in. The result is a ranking that separates a team that went 15-3 against ECNL competition from a team that went 15-3 against state-level competition — even before they ever play each other."

---

## Section 2: Rank Bands — What to Show and What to Say

### TP-1: What Rank Bands Are

"A raw ranking number tells you where a team sits in a list, but it doesn't tell you what that means. Is ranked 47th elite? Solid regional program? Depends entirely on the age group and how many teams are in the pool. Rank Bands solve that. Instead of 'you're 47th,' you can say 'you're in the Silver band' — which means top 2 to 8 percent of every ranked program in your age group. The band means the same thing whether there are 800 teams or 2,000 teams in the pool. We use six bands: Gold, Silver, Bronze, Red, Blue, and Green. Gold is genuinely national-caliber — the top 1 percent. Green is the starting point for teams just entering the ranking system. Every band is a real measurement, not a participation label."

### TP-2: How Bands Narrow Over Time

"When a team first shows up in our system, their band is wide — that's honest. We don't have enough data to be certain. As they play more games, the band tightens. A team with 8 games might span two bands worth of uncertainty. The same team after 25 games has a tight, confident band. That's a feature, not a limitation — it tells you how much to trust the number. A tight Gold band means we've seen enough games to be sure. A wide Silver band means they're probably Silver, but could be Gold or Bronze. No other ranking system shows you that uncertainty. Ours does."

### TP-3: The Recruiting Use Case

"Here's how a recruiting coach uses this. You're looking at two U16 Girls programs — both Silver band. Team A has a tight band, 30 games in our system. Team B has a wide band, 10 games. Team A's Silver is confirmed — they've played enough games against enough competition that the rating has settled. Team B is probably Silver, but there's meaningful uncertainty. A coach relying on rank number alone can't see that difference. The band does. It tells you which evaluation is worth more of your time."

---

## Section 3: Team Comparison — Demo Script

> **Data note:** Examples are from the Rank Bands Demo Examples Document (filed 2026-04-25). Confirm these teams are still in their expected bands after April 29 recompute before running this script at DSS. Run the post-April-28 confirmation query from `Rankings/Rank Bands — Demo Examples Document.md` Section 5 on or after April 29.

**Comparison pair:** Football Academy NJ (U13G, Gold) and PA Classics (U15G, Bronze).

---

1. "Let me show you what this looks like on real teams. I'll pull up Football Academy NJ — they're U13 Girls. USA Rank has them at number 4 nationally. We agree they're elite."

   *[Run individual team lookup query for Football Academy NJ, U13G]*

2. "[Rating value]. Gold band. Within Gold, they're ranked [within_band_rank] in their age group. That's the math behind the badge. This is a team with [game_count] games in our system — every one of those games went into computing that number."

3. "Now let me show you something more useful for the people in this room. PA Classics — U15 Girls. Bronze band."

   *[Run individual team lookup query for PA Classics, U15G]*

4. "[Rating value]. They're not USA Rank top 10. They're not in a full ECNL program. But they have [game_count] games in our system, and they're in the top 15 to 20 percent of all U15 Girls programs we rank. A director at PA Classics can look at this and say: 'We're Bronze. Our path to Silver means closing a [Silver floor - PA Classics rating] point gap. Here's what kind of competition we need to play to close it.' USA Rank doesn't give you that. We do."

5. "If you're a coach evaluating these two teams, here's what Evo Draw tells you that USA Rank can't: Football Academy NJ is Gold-confirmed with [game_count] games. PA Classics is Bronze-confirmed with [game_count] games. Both numbers are trustworthy. You know exactly where to spend your recruiting time."

> **Boys note:** If Boys rankings are authorized (Option A PASS by May 9), you can run the same demo for Boys — substitute a Gold-band MLS NEXT Boys team and a Bronze-band Boys program. Do NOT run Boys comparisons if Option A result is pending or FAIL.

---

## Section 4: Event Strength — One-Liner

"Not all wins are equal. A team that beats a top club at an ECNL event is playing harder competition than a team with the same record at a regional league — and our system knows the difference. We assign an event strength rating to every event in our database based on the average quality of teams that competed in it. When you look at a team's history, you can see not just that they went 8-2 at an event — but that the event itself was high-strength, because it pulled in ECNL-caliber competition from three states. That's context no win-loss record can give you on its own."

---

## Section 5: Handling the "Accuracy" Question

This is the most likely skeptical question at DSS. Rehearse this answer specifically.

"That's the right question, and I'll give you a straight answer. We don't claim to be perfect — no ranking system is, and anyone who tells you otherwise is selling you something. What we can tell you is this: we checked our top-ranked Girls teams against USA Rank, which has been running for years, and for the elite tier — the top programs in U13 and U14 Girls — we broadly agree. The teams USA Rank says are the best are the teams we say are the best. That's a meaningful validation check, not just us agreeing with ourselves.

The difference from USA Rank is that our calibration is explicit and auditable. We know exactly how much each win is worth based on which league it happened in. We derived those weights from 575,000 cross-league games — head-to-head data between leagues that let us calculate how much stronger an ECNL win is than an NPL win. USA Rank's weighting methodology is a black box. Ours isn't.

We also catch our own errors. In April 2026, we identified that Girls teams playing in GA ASPIRE events were being over-credited by 40 rating points per game due to a tier misclassification. We found it, quantified it, and corrected it before this demo. A system without explicit calibration weights has no mechanism to find or fix that kind of error. We do. We're not the most accurate ranking system in the world — we're a ranking system that knows when it's wrong and corrects it. For youth soccer, that's the best you can honestly offer."

---

## Section 6: What NOT to Say

1. **Do not imply rankings are final or authoritative.** Saying "this is their real ranking" or "our system is definitive" erodes trust rather than building it. Rankings update every pipeline run. Say "this is our current assessment" or "based on the games we have." If a parent disagrees about a specific team's placement, the correct response is: "we'd want to look at what games are in our system for that team — coverage gaps are real and we track them."

2. **Do not quote specific numbers that may shift before May 16.** Volatile numbers: any specific team's current rating (changes with each pipeline run), any aggregate count like "we rank X teams" (fill this with actual count from the data readiness validation on May 14-15, not from memory). Stable numbers: the 575,000-game cross-league dataset, the number of event tiers (10), the six band names (Gold through Green), the 6-game minimum for ranking eligibility, the 15+ game threshold for a stable band.

3. **Do not make claims about Boys calibration until Option A is confirmed.** If Boys rankings come up before Option A results are in: "Boys rankings use the same engine as Girls — same algorithm, same calibration structure. MLS NEXT Boys values are our most validated cohort, empirically confirmed from thousands of cross-league games. We're running a spot check on Boys rankings right now to confirm the U13 and U14 Boys ratings meet our internal quality threshold before we feature them prominently. We'll have that result before May 9." Do not say "Boys rankings are fully calibrated" — they haven't been gender-separately Brier-validated.

---

*ELO — 2026-04-26*
