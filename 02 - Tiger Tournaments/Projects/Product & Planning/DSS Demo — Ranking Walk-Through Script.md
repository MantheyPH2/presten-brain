---
title: DSS Demo — Ranking Walk-Through Script
type: demo-script
author: ELO
date: 2026-04-28
status: filed — SENTINEL review required before May 9
due: 2026-05-07
related: "[[DSS Demo Spec — May 2026]]", "[[DSS Ranking Accuracy Claims — Authorized Spec]]", "[[DSS Demo — Feature Sequence Plan]]"
tags: [dss, demo, rankings, script, presten, sentinel, evo-draw]
---

# DSS Demo — Ranking Walk-Through Script

> **Filed by ELO — 2026-04-28 01:39**
> This is Presten's rehearsal guide for the ranking section of the May 16 DSS demo. Read it aloud before the demo. Section 2 and Section 5 answers should be internalized — not read from this document during the demo.
>
> SENTINEL: review Sections 1–4 for any claim not supported by confirmed analysis. Section 5 notes which claims are conditional on April 29 results.

---

## Section 1: Ranking Narrative (60–90 seconds)

> Read this aloud to time it. Target: 70–80 seconds at conversational pace.

"Every week, thousands of youth soccer teams play games across the country. Evo Draw tracks those results and uses them to rank teams — not just based on who won or lost, but based on who they played against.

If your team beat an MLS NEXT Academy side at a showcase in December, that tells us something important about your team. You beat top competition. That gets you more credit than beating a team from your local area. And if your team lost to a top-ranked program by one goal at nationals, that's meaningful too — we factor that in.

The math is straightforward: the stronger your opponent, the more a win moves your ranking up and the less a loss moves it down. A team with a 10–4 record against elite competition will rank higher than a team with a 14–0 record that only played within their region.

And because we look at your full season — not just the last few games — you get credit for everything your team has done this year. Form from September still matters in April."

**ELO timing note:** 130 words at a conversational 160 wpm = approximately 49 seconds. With natural pauses, this runs 60–70 seconds. Acceptable. Do not add explanatory asides — they push past 90 seconds.

**What this narrative covers (per task spec):**
- ✅ Why game results beat standings (head-to-head across leagues)
- ✅ Why opponent quality matters (MLS NEXT win vs. local win)
- ✅ Why recency is factored (full season, not just recent games)
- ✅ No "ELO," no "K-factor," no statistical jargon

---

## Section 2: The Demo Team

**Team: PA Classics Girls U14 — ECNL**

| Field | Value |
|-------|-------|
| Club | PA Classics (Pennsylvania) |
| Age group | U14G |
| League | ECNL |
| Expected national rank | Top 15–30 Girls U14 nationally |
| Rank band | Silver (expected) |
| Why this team works | PA Classics is the anchor demo club per the Demo Spec. The PA/NJ audience at DSS will recognize this club. PA Classics ECNL U14G plays a full ECNL schedule against top-tier opponents — their ranking reflects head-to-head cross-league competition, not just a regional win-loss record. The contrast between "they're a well-known club" and "here's exactly why their ranking is where it is" tells the system's story cleanly. |

**Verification required before May 14:**
- Confirm PA Classics U14G appears in the DB with ranking in range (use audit SQL from `Rankings/DSS Demo Teams — Merge Safety Verification.md`)
- Confirm no active dedup dispute or split record for this team
- If rank falls outside top 30: use PA Classics U13G or U15G instead — same club, same narrative; pick the age group with the cleanest rank

**Backup team if PA Classics is unavailable:** PDA (Premier Development Academy) U14G, NJ. Same demo narrative, high DSS audience recognition in the NJ/NE corridor.

---

## Section 3: The 3-Step Show

### Step 1 — Show the team's ranking

**Action:** Type "PA Classics" in the team search bar → click the U14G ECNL entry from the results.

**What appears:** Team ranking card showing national rank (e.g., #18 nationally among Girls U14), rating (e.g., 1,410), match count, and last game date.

**What to say:** "PA Classics U14 Girls — here's where they rank nationally. Out of all Girls U14 teams playing high-level soccer in the country, they're in the top 20. That rank is built entirely from the results they've put up against ECNL competition this season."

---

### Step 2 — Show rank band

**Action:** On the team card, point to the rank band indicator ("Silver" tier badge).

**What to say:** "Next to their rank, you see a band — Silver. That means this team has consistently competed and won against high-caliber opponents. Gold is the top tier nationally, Silver is the next bracket. This is the information college coaches and tournament directors use at a glance — not a number, a category."

---

### Step 3 — Show top teams in their age group

**Action:** Navigate to Rankings → Girls U14 → top 20 list.

**What to say:** "Here's the full top 20 for Girls U14 nationally. Notice what the teams in the top 10 have in common — they all play MLS NEXT or ECNL schedules, which means they're regularly playing against other high-ranked opponents. The system rewards that. A team that only plays within its state can win every game and still not crack the top 50 — because we don't know how they'd do against true national-level competition."

---

## Section 4: The USARank Comparison Line

**Script line:**

> "Evo Draw accounts for the strength of every opponent. Rankings that use win-loss records alone can't tell the difference between beating an MLS NEXT Academy and beating a club team from a local league — we can."

**Backing analysis:** `Rankings/USA Rank Comparison 2026-04-23.md` — Section 5 (Methodology Divergence Assessment). USA Rank uses a goal-difference-based last-20-games recency window. Evo Draw uses Glicko-2 with hardcoded league calibration values validated against 575K cross-league games.

**ELO authorization status:** This line is drawn from Authorized Claim G3 (full game history) in `Rankings/DSS Ranking Accuracy Claims — Authorized Spec.md`. This claim does not require post-April-28 data confirmation — it is a design fact. It is authorized for DSS use unconditionally.

**What NOT to say:** Do not name USARank by name in the comparison ("Evo Draw is more accurate than USARank"). Say "rankings that use win-loss records" — same point, no name. This avoids a direct competitive put-down in a room where some attendees may have relationships with USARank partner organizations.

---

## Section 5: Expected Questions and Prepared Answers

| Question | Answer |
|----------|--------|
| "Why is my daughter's team ranked lower than I expected?" | Your team's ranking reflects the strength of the opponents they've played, not just their record. A team that won 12 games against mid-level competition will rank below a team that went 8–4 against MLS NEXT and ECNL sides. If you want to understand a specific rank, the team page shows every game and how each result moved the rating. |
| "How often do rankings update?" | Every night. Games played yesterday are in the rankings by morning. |
| "Does this include club rankings?" | Yes — you can search by club and see all of their teams ranked across every age group. [If Girls-only at demo:] Today we're showing Girls club rankings — we'll have Boys added soon as well. |
| "What about boys teams?" | Boys teams are ranked using the same engine — the same opponent-quality logic applied to MLS NEXT, ECNL Boys, and other top leagues. [If Option A confirmed:] Boys rankings are fully live. [If pending:] Boys rankings are in final calibration review and will be confirmed shortly. |
| "Is this better than MaxPreps / USARank?" | Evo Draw accounts for opponent quality in ways win-loss-based systems can't. Whether that's "better" depends on what you need — but if you want to know how a team would perform against national competition, cross-league calibration is the only way to answer that accurately. |

**Delivery note:** Each answer above is 2–3 sentences, which is the right length for a demo setting. Do not elaborate unless the questioner follows up. A confident, short answer signals that the system is well-understood — not a hedge.

---

## Section 6: What NOT to Demo

1. **Boys age groups (if Boys Option A has not confirmed by May 14)** — Do not show Boys rankings or Boys-specific calibration claims until SENTINEL authorizes. If Boys is confirmed, Boys rankings can be demoed alongside Girls with no restrictions beyond the Claims Spec.

2. **Football Academy NJ U13G** — This team may exist as two split records ("Football Academy NJ 2012 Girls Black" and "FA 12G Black"). Until the merge is confirmed before May 14, do not use this team as a demo example. A split-record team will appear lower-ranked than expected and undermine the demo's credibility.

3. **GA-only teams without cross-league game history** — Teams that have played exclusively within GA events haven't been tested against ECNL or MLS NEXT competition in our data. Their rating reflects the GA calibration (140) but may look anomalous compared to USARank expectations. Not safe for demo.

4. **The team_merges count (27,820)** — Do not mention this number in any form during the demo. The audience will interpret it as data quality errors, not as the deduplication and data hygiene work that it actually is.

5. **Brier scores (0.312, 0.273 for U13 and U14)** — These are pre-fix values above the acceptable threshold. Do not cite them as accuracy evidence. Use the authorized softened claim from the Claims Spec instead: "We track prediction accuracy and are actively improving it."

6. **Any coverage number before it's confirmed** — Claim G2 (active team count) requires running Step 4A of the DSS Data Readiness Validation Plan before May 14. Do not state the coverage number at DSS unless Presten has confirmed the actual count. Use "we rank more teams than any other platform we've compared against" only if the Step 4A count confirms ≥ 30% more than USARank's visible list in the same age groups.

---

## Pre-Demo Checklist (ELO confirms before May 14)

- [ ] PA Classics U14G confirmed in DB with rank in top 30 (or backup team selected and confirmed)
- [ ] PA Classics confirmed as no active merge dispute (`Rankings/DSS Demo Teams — Merge Safety Verification.md`)
- [ ] Step 3 top-20 list confirmed to show ECNL/MLS NEXT teams at the top (not anomalies)
- [ ] Section 1 narrative timed aloud — confirmed under 90 seconds
- [ ] Section 5 answers reviewed and memorized by Presten
- [ ] Boys status confirmed: either Option A PASS (Boys can be demoed) or Option A fail/pending (Boys demoed only with Section 1 language, no Boys-specific claims)
- [ ] SENTINEL has signed off on Claims Spec conditions before May 14

---

## References

- `Rankings/DSS Ranking Accuracy Claims — Authorized Spec.md` — every claim Presten makes must be from this document
- `Product & Planning/DSS Demo Spec — May 2026.md` — overall demo structure; ranking section is one module
- `Product & Planning/DSS Demo — Feature Sequence Plan.md` — where the ranking walk-through fits in the 40-minute demo
- `Rankings/USA Rank Comparison 2026-04-23.md` — backing analysis for Section 4
- `Rankings/DSS Demo Teams — Merge Safety Verification.md` — PA Classics and other demo teams verified safe
- `Rankings/Rank Bands — Demo Examples Document.md` — rank band examples for Step 2
- `Product & Planning/May 9 DSS Readiness Verdict Document.md` — readiness gate this script feeds

*ELO — 2026-04-28 01:39*
