---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-26
status: completed
completed: 2026-04-26
priority: medium
due: 2026-04-29
deliverable: "02 - Tiger Tournaments/Projects/Product & Planning/Source Submission MVP — Session Plan and Schedule.md"
---

# Task: Source Submission MVP — Session Plan and Schedule

## Context

The Source Submission MVP has a complete implementation plan (`Product & Planning/Source Submission MVP — Implementation Plan.md`): table schema (`source_submissions`), POST endpoint (`/api/source-submissions`), and submission form with 4–6 hour build estimate. It was targeted to ship before DSS (May 16–18). DSS is now 20 days away.

No confirmation exists that the MVP has shipped. No session has been scheduled where FORGE or Presten actually builds it. The implementation plan specifies what to build; what does not exist is when to build it and what the session looks like.

This task produces a concrete session plan: a scheduled implementation session, how Presten runs it, what the milestone markers are, and when FORGE has a running MVP to test against.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Product & Planning/Source Submission MVP — Session Plan and Schedule.md`

---

### Section 1: Scheduled Session Window

State the proposed session window for building the MVP. Given current constraints:
- April 28: DB execution session (no capacity for feature work)
- April 29: ELO analysis session (no capacity for feature work)
- April 30 – May 3: First available window

Proposed window: **[FORGE proposes date]** — estimated 4–6 hours.

If FORGE cannot determine the session date (depends on Presten's calendar), file this section with `TBD — Presten to confirm` and include a latest acceptable date: **May 5** (leaves 11 days before DSS for testing and QA).

---

### Section 2: Pre-Session Checklist

What must be confirmed before the implementation session begins:

- [ ] Presten has reviewed the implementation plan (`Source Submission MVP — Implementation Plan.md`) — estimated read time: 10 minutes
- [ ] Database has write access in the session environment
- [ ] The `source_submissions` table does not already exist (or schema is confirmed if it does)
- [ ] FORGE's API route structure has been reviewed (confirm existing routes follow the same pattern as the new endpoint)
- [ ] SnapSoccer source research is NOT blocking this MVP (confirm: these are independent work items)

---

### Section 3: Session Milestones

| Milestone | Description | Estimated Time | Success Signal |
|-----------|-------------|----------------|----------------|
| M1 | `source_submissions` table created, migration run | 30 min | `\d source_submissions` shows schema |
| M2 | POST `/api/source-submissions` endpoint live | 90 min | `curl` POST returns 201 with submission_id |
| M3 | Submission form renders in browser | 60 min | Form visible at [route], all 4 fields present |
| M4 | End-to-end test: form → endpoint → DB | 30 min | Row appears in `source_submissions` after form submit |
| M5 | Admin review view (read-only list of submissions) | 60 min | Presten can view submitted sources at admin route |

Total: ~4.5 hours to M4. M5 is optional for initial MVP.

If any milestone takes 2× its estimate, FORGE flags to SENTINEL and Presten decides whether to continue or defer M5.

---

### Section 4: Post-Session Acceptance Criteria

MVP is considered shipped when:

1. `source_submissions` table exists in production database
2. POST endpoint accepts a valid submission and returns 201
3. Form is accessible at a defined URL
4. At least one test submission exists in the database from a Presten form submission
5. FORGE files a session completion note in the next briefing with a link to the admin review view

---

### Section 5: DSS Relevance

One paragraph. What does having this MVP before DSS enable?

- Community members at DSS who want to submit missing game sources can do so in real time
- SENTINEL can present "we have a source submission pipeline" as evidence of community integration (one of the 7 competitive gaps vs. USA Rank)
- Post-DSS, submitted sources feed directly into FORGE's source gap inventory

What this MVP does NOT need to do before DSS:
- Auto-process submissions (manual review is acceptable)
- Handle file uploads (text fields only)
- Integrate with the crawl pipeline automatically

---

## Quality Standard

- The session plan must have a specific proposed date (or "TBD — Presten to confirm" with a hard latest-acceptable date of May 5).
- The milestone table must have explicit success signals — not "seems to work."
- If FORGE identifies that the implementation plan requires revision before the session (e.g., a dependency it missed), flag it in this document and include what revision is needed and how long it adds.

## Why This Matters

The implementation plan has existed since April 23. It has not moved to execution because no one has scheduled the session. Every day without a scheduled session is a day that compresses the pre-DSS timeline. This task converts the implementation plan from a document into a time-boxed session with named milestones. Once the session plan exists, SENTINEL can confirm it with Presten and hold FORGE accountable to the milestone dates.

## References

- `Product & Planning/Source Submission MVP — Implementation Plan.md` — what to build
- `Data Sources/SnapSoccer Source Research.md` — separate work item, not a dependency
- `Product & Planning/DSS Feature Readiness Tracker.md` — Source Submission is a feature SENTINEL tracks
