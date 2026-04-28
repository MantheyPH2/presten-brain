---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-27
status: completed
completed: 2026-04-27
priority: medium
due: 2026-04-30
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/Source Submission MVP — Intake Form Technical Spec.md"
---

# Task: Source Submission MVP — Intake Form Technical Spec

## Context

The Source Submission MVP Session Plan (filed 2026-04-26) defines the work scope as a 3.5–4 hour session and describes the high-level goal: let FORGE submit and validate new sources without requiring a Presten coding session. The session plan exists. What does not exist: a precise technical specification of the intake form itself.

Before the MVP session can be executed, FORGE needs to define: what fields does a new source submission contain, what validation fires at intake, and how does a validated source entry flow into the pipeline config. Without this spec, the session plan becomes an improvisational coding session. With it, the session becomes a fill-in-the-spec build.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/Source Submission MVP — Intake Form Technical Spec.md`

---

### Section 1 — Source Entry Data Model

Define the exact fields a source submission must contain. For each field, specify:
- **Field name** (snake_case, as it will appear in config or DB)
- **Type** (string, integer, boolean, enum)
- **Required or optional**
- **Valid values / format** (e.g., `source_type: enum["gotsport_org", "sincsports", "affinitysoccer", "manual_csv"]`)
- **Purpose** (one line — what the pipeline uses this field for)

Minimum expected fields:
- `source_id` — unique identifier
- `source_type` — enum matching existing pipeline source classifier
- `org_id` or `source_url` — where to fetch data
- `league_hint` — optional, helps classify events if source doesn't self-identify
- `active` — boolean, whether pipeline should crawl this source
- `last_verified` — date FORGE last confirmed the source is live
- `notes` — free text for unusual config or access notes

FORGE may add fields. Justify each addition briefly.

---

### Section 2 — Validation Rules

For each required field, define the validation that fires when a new source entry is submitted:

| Field | Validation | Failure action |
|-------|-----------|----------------|
| `source_type` | Must match existing classifier enum | Reject with: "Unknown source_type. Valid values: [list]" |
| `org_id` | Must be numeric if source_type = gotsport_org | Reject with: "org_id must be numeric for GotSport sources" |
| (etc.) | | |

Two additional validations that must be present regardless of field values:
1. **Duplicate check:** reject if `source_id` or `org_id` already exists in config
2. **Reachability check:** optional but preferred — attempt a HEAD request to the source URL to confirm it is live before adding to config

---

### Section 3 — Intake → Pipeline Flow

Describe how a validated source entry moves from the intake form to active crawling:

1. FORGE submits source entry (how? — direct config file edit, API call, or SQL insert?)
2. Validation fires (inline or as a pre-commit check?)
3. Entry is written to (what file or table?) — name the exact config path from the org-ID expansion work
4. Pipeline picks up new entry on next run (no restart required? or does pipeline need a reload signal?)
5. FORGE confirms ingestion by running (which query?) after the next pipeline run

This section should be specific to the current pipeline architecture. Reference the Org-ID Config Edit Reference and the GotSport Org-ID Master Reference for field names and config file location.

---

### Section 4 — Session Execution Plan

Given this spec, write the step-by-step build plan for the MVP session. Each step: what to build, how long it takes (estimate), definition of done. Target total: ≤4 hours.

The session plan already filed (Apr 26) defines the phases. This section translates those phases into concrete build steps now that the data model and validation rules are specified.

---

### Section 5 — Testing Protocol

Before declaring the MVP complete:
- Add a test source entry (use a known-good GotSport org-ID already in production as the test subject)
- Confirm validation passes
- Confirm the entry appears in the pipeline config
- Confirm the next pipeline run ingests games from that source
- Remove the test entry

Define the exact SQL or file command for each step.

---

## Definition of Done

- All 5 sections present
- Data model has ≥7 fields, each fully defined
- Validation rules table covers all required fields + duplicate check
- Intake-to-pipeline flow specifies exact file paths and commands
- Session execution plan has step-level time estimates summing to ≤4 hours
- Testing protocol has copy-paste-ready commands for all 5 test steps

## Why This Matters

The source submission MVP is the bottleneck for expanding beyond the current org-ID set. Every new source — SincSports, Affinity, state league portals — currently requires a Presten coding session to add. The MVP removes that bottleneck. But building the MVP without a spec produces an ad-hoc tool that works once and breaks silently. This spec makes the MVP session a 4-hour build against a defined target.

## References

- `Infrastructure/Source Submission MVP — Session Plan.md` — high-level plan this spec operationalizes
- `Infrastructure/GotSport Org-ID Config Edit Reference.md` — config file structure
- `Infrastructure/GotSport Org-ID Master Reference.md` — current org-ID inventory
- `Infrastructure/Non-GotSport Top-Priority Source Ingestion Design.md` — sources the MVP must support
- `task-2026-04-26-non-gotsport-top-source-ingestion-design.md` — parent research task
