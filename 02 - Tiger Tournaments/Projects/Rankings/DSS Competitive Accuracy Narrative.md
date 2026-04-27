---
title: DSS Competitive Accuracy Narrative
type: deliverable
author: ELO
date: 2026-04-26
status: filed
tags: [rankings, accuracy, competitive, dss, narrative, evo-draw]
---

# Why Evo Draw's Rankings Are More Accurate

## The Core Claim

Evo Draw ranks teams using an Elo/Glicko v7 algorithm trained on 570,000+ youth soccer games, calibrated across 10 distinct league tiers. Unlike systems that weight all wins equally, Evo Draw adjusts every game's contribution to a team's rating based on three factors simultaneously: the opponent's current rating, the event tier (MLS NEXT carries more weight than a regional league), and recency. The result: a team that beats a U16 Girls ECNL opponent gains meaningfully more rating points than a team that beats a club in a lower-tier regional circuit — which is the correct answer. A ranking system that cannot make this distinction will always produce distorted rankings, no matter how many games it processes.

## Where It Shows Up

**Event tier weighting produces separations that goal-difference systems miss.** Consider two Girls U15 teams with identical win-loss records: one plays its full season in GA ASPIRE events, the other in ECNL. USARank's methodology derives league quality emergently from inter-league results — which means it requires direct cross-league matchups to calibrate. Evo Draw does not wait for those matchups. Our calibration values are derived from 575,000 cross-league games and hardcoded per tier: ECNL Girls at 130, GA ASPIRE at 100. The ECNL team's wins are weighted 30% higher per game before any head-to-head data is needed. That pre-calibration advantage means Evo Draw separates these teams accurately from day one of the season, not after enough cross-league games accumulate.

**The GA ASPIRE recalibration is a concrete example of the discipline this requires.** Evo Draw identified in April 2026 that GA ASPIRE events were being classified under the `ga` event tier (calibration value: 140) instead of the correct `ga_aspire` tier (calibration value: 100). The result was that Girls teams playing in GA ASPIRE events were receiving 40 points of inflated calibration per game — invisible to anyone who didn't compare ECNL vs GA ASPIRE win rates directly. Evo Draw caught it, quantified it, and corrected it in a controlled fix on April 28. A system without explicit calibration values has no mechanism to catch this kind of error: if calibration is entirely emergent, there is no "correct value" to check against.

**Evo Draw's algorithm is not just a rating — it enforces head-to-head truth.** Phase 10 of the ranking engine is an iterative head-to-head enforcement pass: if team A has beaten team B directly but ranks below them, the system swaps them. This is not possible in goal-difference systems, which can only move teams up or down based on recent results — they cannot guarantee that a team that beat another team outranks that team. Evo Draw guarantees it (within a 250-point gap threshold to prevent single upsets from cascading into rank inversions across hundreds of teams).

## What Makes This Hard

Ranking youth soccer teams nationally is structurally harder than ranking professional leagues. There is no governing body that requires teams to play cross-league opponents — competitive exposure is patchy and uneven. Game data lives across seven or more distinct platforms (GotSport, TGS AthleteOne, Sinc Sports, Affinity, and more), each with different export formats and team naming conventions. The same team can appear under four different names across platforms. Age group transitions mean that a team's historical rating from last season must carry forward correctly when U14s become U15s. Boys and Girls do not share calibration parameters — the strongest Boys league (MLS NEXT, 160) and the strongest Girls leagues (GA, 140; ECNL, 130) sit at different absolute levels, because the competitive structure of Boys and Girls youth soccer is not symmetrical. Each of these is a place where a simpler system cuts corners. Evo Draw addresses each one explicitly.

## The Calibration Commitment

Evo Draw's calibration values are reviewed on a recurring basis, not set-and-forgotten. The GA ASPIRE correction in April 2026 was identified through systematic cross-league win-rate comparison — the same methodology used to validate all 10 tier calibration values. The team merges audit (27,820 merge records linking the same team across platforms) is another expression of this commitment: a team's rating is only as accurate as the game history it's computed from, and that history is only accurate if duplicate records are collapsed correctly. These are not one-time implementation decisions. They are ongoing quality processes. USARank and comparable systems do not publish their calibration methodology, do not document tier-specific weighting, and have no public mechanism for identifying or correcting miscalibrations. Evo Draw does.
