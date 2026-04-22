---
type: agent-queue
agent: FORGE
category: question
priority: high
date: 2026-04-21
status: answered
answer: "find a solution."
---

# Question: GotSport API Rate Limit Strategy

## Context

GotSport appears to have silently reduced their API rate limit from 120 req/min to approximately 60 req/min within the last two weeks. This is not documented in their changelog. The current sync runner uses a fixed 500ms inter-request delay which was calibrated for the old limit. As a result, the nightly 02:00 UTC sync run — which triggers full roster refreshes — is hitting 429 (Too Many Requests) errors on roughly 34% of requests during peak volume windows.

6 of the last 14 scheduled sync runs have returned partial data as a result. This is the root cause behind the 847 teams with stale records identified in today's audit.

## The Question

How do you want to handle the GotSport rate limit situation? Three options are on the table:

---

**Option A — Increase Fixed Delay**
Raise the inter-request delay from 500ms to 1100ms (a conservative buffer under the new 60 req/min limit). Simple to implement, predictable, no added complexity. Downside: slows all sync runs by ~55%, including off-peak windows where we're nowhere near the limit.

**Option B — Exponential Backoff with Jitter**
On a 429 response, back off exponentially (starting at 1s, doubling per retry, up to a 32s cap) with randomized jitter to prevent thundering herd. Keep the base inter-request delay at 600ms. This is the more robust solution — self-corrects under load, stays fast in low-volume windows. Adds a small amount of complexity to the sync runner. I can have this implemented same-day.

**Option C — Keep Current Behavior**
Do nothing for now and accept the partial syncs until GotSport clarifies their new limits officially. Risk: stale data continues to accumulate and the 847 affected teams will not be resolved.

---

## My Recommendation

Option B. The exponential backoff pattern is well-tested and will handle future rate limit changes gracefully without needing manual tuning. Option A is a reasonable short-term patch if you want to keep things simple. Option C is not advisable given that the problem is already affecting ~19% of active teams.

Happy to implement whichever you choose — just need a direction call.

## Waiting On

Presten's decision before proceeding with the stale team backfill design, since the backfill batch parameters depend on which rate limit strategy is in place.
