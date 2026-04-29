---
title: SnapSoccer — SincSports Accessibility Pre-Check
tags: [infrastructure, snapsoccer, sincsports, pre-check, forge]
created: 2026-04-29
updated: 2026-04-29
author: FORGE
status: ready — Presten completes before May 10
---

# SnapSoccer — SincSports Accessibility Pre-Check

**This check takes 5 minutes. Run it during any browser session before May 10. Report results to FORGE in next session message.**

The SnapSoccer build window opens May 10–17. This is the single pre-condition FORGE requires before starting. Do not begin the build without this check completed.

---

## The 5 Checks

Run each in order. Fill in the result inline.

**1. URL accessible**

Navigate to: `soccer.sincsports.com/schedule.aspx`

Does it load?
- [ ] Yes
- [ ] No
- [ ] Redirected to: _______________________

---

**2. Authentication required**

Is there a login wall before viewing schedule data?
- [ ] No login required
- [ ] Login required
- [ ] Other: _______________________

---

**3. Schedule data visible**

Can you see team names, game dates, and scores without logging in?
- [ ] Yes
- [ ] Partial — specify: _______________________
- [ ] No

---

**4. URL parameter present**

Look at the URL when viewing a specific tournament or event schedule. Is there a `tid=` or similar parameter visible?
- [ ] Yes — paste example URL: _______________________
- [ ] No
- [ ] Unclear

---

**5. Mobile layout / anti-scrape signals**

Any CAPTCHA, Cloudflare challenge, or unusual redirect on load?
- [ ] None
- [ ] Yes — describe: _______________________

---

## Section 2 — Result Routing

| Result | FORGE Action |
|--------|-------------|
| All 5 PASS (accessible, no login, data visible, tid= present, no anti-scrape) | SnapSoccer build proceeds May 10 as planned |
| Check 2 FAIL (login required) | FORGE pauses build; files revised risk assessment within 24 hrs; notifies SENTINEL |
| Check 3 FAIL (no data visible) | FORGE pauses build; assesses alternate endpoint; notifies SENTINEL |
| Check 5 FAIL (anti-scrape signals) | FORGE proceeds cautiously; adds rate-limit mitigation to Day 1 build scope |

---

## Section 3 — Timing Guidance

Ideal: fold into TYSA/EDP/USL Academy browser session (any time before May 10). Latest acceptable: May 9 standalone check. If no browser session opens by May 9, FORGE will request SENTINEL escalate to Presten.

- FORGE will flag this check in its session prep message if a session opens between now and May 10
- This check is non-blocking for May 1 pipeline work — this is a May 10 pre-condition only
- If no session has opened by May 5, FORGE will raise this proactively — do not wait until May 9

---

*FORGE — 2026-04-29*
