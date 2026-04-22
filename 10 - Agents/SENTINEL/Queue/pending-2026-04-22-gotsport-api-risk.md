---
type: agent-queue
agent: SENTINEL
category: alert
priority: high
date: 2026-04-22
status: pending
answer: ""
---

# Alert: GotSport API Dependency Risk — Escalation Based on New Evidence

## Summary

The GotSport API dependency risk (Strategic Insights, Risk 1: "Critical/Medium severity") has moved from theoretical to active. The rate limit change FORGE discovered is not just an operational inconvenience — it is evidence that GotSport is modifying their API behavior without notice. SENTINEL is escalating this from a roadmap consideration to a near-term decision item.

---

## What FORGE Found

GotSport reduced their public API rate limit from ~120 req/min to ~60 req/min sometime in the last two weeks. No developer changelog. No advance notice. The current scraper hit 429 errors on 34% of requests during peak windows, causing 847 teams to have stale data (19% of the active team roster). FORGE posted a strategy question (pending-2026-04-21-rate-limit) on how to respond to the rate limit change itself.

---

## Why This Is Bigger Than the Rate Limit

The rate limit change is a symptom. The underlying issue: Evo Draw's entire data supply chain depends on a single unauthenticated public API that:
- Has no formal agreement, no partnership, no notification channel
- Can be changed or removed at any time with no recourse
- Has already been changed once in the last two weeks
- Belongs to a company that operates its own competing rankings system (GotSport Rankings)

The Sinc Sports situation is the reference case: Cloudflare was added, scraper stopped working, 34,961 games became a permanent static dataset with no live updates. If GotSport follows this path, Evo Draw loses 1.49M games of active data pipeline — 90% of the platform — overnight.

The GotSport HTML scraper is a partial fallback (event-centric) but cannot replace the API's team discovery function (107,528 team IDs from the rankings endpoint). There is no other fallback. The webhook listed in the v2 spec was never implemented.

---

## Questions for Presten

### 1. Has there been any communication with GotSport?
Has Presten had any direct contact with GotSport's business or development team? Even informal contact? If not, this is the moment to make it. Positioning: Evo Draw drives user engagement with GotSport-hosted tournaments and provides tournament directors with a compelling reason to use GotSport for registration. This is a value-add framing, not a "we're scraping you" conversation.

### 2. Is a formal data partnership feasible?
A data partnership (even informal — a written understanding that GotSport will not block Evo Draw's scraper) would transform this from a critical risk to a managed one. Even a revenue-share arrangement (Evo Draw refers tournament teams to GotSport registration → GotSport provides data access) could work structurally.

### 3. DSS Registration Pace
Separate from the API issue: Dallas Spring Showcase is May 16-18 — 24 days away. Registration is at 308 of 390 teams (79% of target). Is there active outreach to fill the remaining 82 spots? Is 390 the break-even target, or is 308 already viable? SENTINEL does not have visibility into the business operations side of DSS.

### 4. GA Data Coverage
Strategic Insights flags GA as a significant knowledge gap. Questions:
- Do we have a confirmed count of GA games in the `youth_soccer` database? (We know GA data presumably flows through GotSport but this has never been explicitly audited.)
- Is the 140-point girls / 100-point boys GA calibration value empirically derived (like ECNL RL's 55.1% win rate) or was it manually set?
- Are we tracking GA ASPIRE tier games? Does ASPIRE need its own calibration value?

### 5. Unverified Team Merges Verification Campaign
The Dedup Strategy note confirms 9,136 unverified team merges in the database. Strategic Insights rates this as a significant moat-weakening risk. Should SENTINEL assign a verification campaign to FORGE? If so, should it be batch (automated confidence scoring) or manual (human review for high-volume teams)?

---

## Recommended Actions

**Immediate (this week):**
- FORGE is scoping the webhook fallback — let that proceed. It will clarify what a real secondary data path looks like and what implementation effort is required.
- Consider whether initiating contact with GotSport is appropriate now. The rate limit change gives a neutral conversational hook ("we noticed a rate limit change and wanted to understand your API policy direction").

**Short-term (before DSS):**
- Rate limit strategy decision from the pending-2026-04-21-rate-limit queue item is needed. FORGE is holding the backfill job pending this direction.
- Confirm DSS registration strategy for the 390-team target gap.

**Medium-term (May-August):**
- GotSport partnership conversation, regardless of outcome.
- Webhook fallback or alternative secondary data path scoping.

---

## FORGE Reference

The rate limit strategy question (Option A/B/C) is in FORGE's queue item `pending-2026-04-21-rate-limit`. That is a separate, more immediate decision. This queue item is about the broader strategic dependency, not the day-to-day rate limit handling.
