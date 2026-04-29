---
type: agent-task
assigned_by: SENTINEL
assigned_to: FORGE
date: 2026-04-29
time: "11:15"
priority: medium
due: "May 9 (before SnapSoccer build window)"
status: completed
completed: 2026-04-29
topic: snapsoccer-sincsports-pre-check-protocol
---

# Task: SnapSoccer — SincSports Accessibility Pre-Check Protocol

## Objective

Create a 5-item browser check for `soccer.sincsports.com/schedule.aspx` that Presten can execute in under 5 minutes during any upcoming browser session. This is the sole pre-condition FORGE identified before beginning the May 10–17 SnapSoccer build.

## Background

The SnapSoccer Authorization Response Package identifies one pre-condition: confirmation that `soccer.sincsports.com/schedule.aspx` is accessible before May 11. The SnapSoccer build cannot start blind on day 1 if the source is gated, requires authentication, or has changed structure since FORGE's research. A 5-minute check eliminates that risk entirely.

The check can be folded into the TYSA/EDP/USL Academy browser session if Presten opens one before May 10. If no session opens, the check must happen no later than May 9 (the day before the build window opens).

## Deliverable

**File:** `Infrastructure/SnapSoccer — SincSports Accessibility Pre-Check.md`

## Required Content

### Header

State: "This check takes 5 minutes. Run it during any browser session before May 10. Report results to FORGE in next session message."

### The 5 Checks

Present as a numbered checklist Presten can fill in real time:

1. **URL accessible:** Navigate to `soccer.sincsports.com/schedule.aspx`. Does it load? (Yes / No / Redirected to: _____)

2. **Authentication required:** Is there a login wall before viewing schedule data? (No login required / Login required / Other: _____)

3. **Schedule data visible:** Can you see team names, game dates, and scores without logging in? (Yes / Partial — specify: _____ / No)

4. **URL parameter present:** Look at the URL when viewing a specific tournament/event schedule. Is there a `tid=` or similar parameter visible? (Yes — paste example URL: _____ / No / Unclear)

5. **Mobile layout / anti-scrape signals:** Any CAPTCHA, Cloudflare, or unusual redirect on load? (None / Yes — describe: _____)

### Section 2 — Result Routing

| Result | FORGE Action |
|--------|-------------|
| All 5 PASS (accessible, no login, data visible, tid= present, no anti-scrape) | SnapSoccer build proceeds May 10 as planned |
| Check 2 FAIL (login required) | FORGE pauses build; files revised risk assessment within 24 hrs; notifies SENTINEL |
| Check 3 FAIL (no data visible) | FORGE pauses build; assesses alternate endpoint; notifies SENTINEL |
| Check 5 FAIL (anti-scrape signals) | FORGE proceeds cautiously; adds rate-limit mitigation to Day 1 build scope |

### Section 3 — Timing Guidance

"Ideal: fold into TYSA/EDP/USL Academy browser session (any time before May 10). Latest acceptable: May 9 standalone check. If no browser session opens by May 9, FORGE will request SENTINEL escalate to Presten."

## Notes

- FORGE should not wait until May 9 to raise this if no session has opened by May 5
- If a session opens between now and May 10, FORGE flags this check in its session prep message so Presten knows to include it
- The check is non-blocking for May 1 pipeline work — this is a May 10 pre-condition only
