---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-27
priority: medium
status: completed
completed: 2026-04-27
due: 2026-05-02
---

# Task: Non-GotSport Source Re-Rank (Post-SincSports Decision)

## Context

SincSports is confirmed NO-GO pre-DSS. The Go/No-Go Decision Package notes that GotSport FL/GA org-ID config adds cover the same geographic area (Southeast) with higher volume and zero engineering cost. With SincSports off the pre-DSS table, FORGE needs a clear ranked list of which non-GotSport source to pursue first in the post-DSS window.

The `Infrastructure/Non-GotSport Top Sources — Ingestion Design.md` (filed 2026-04-26) documents sources but was written before the SincSports decision was finalized. It should be updated or supplemented with a prioritized recommendation now that the decision is locked.

## Task

Produce an updated priority ranking of the top non-GotSport sources for the post-DSS engineering queue. The ranking should be based on:

1. **Games/year (incremental)** — estimated new games added beyond current GotSport coverage
2. **Engineering hours** — estimated build + integration cost
3. **ROI ratio** — games per engineering hour
4. **Accessibility risk** — probability source is scrapable without auth wall
5. **DSS narrative value** — whether the source covers a prestige league that improves Evo Draw's story to demo users

## Sources to Include

At minimum: SincSports (for reference/comparison), Affinity Soccer, Elite64/NAL, any sources identified in the source gap inventory (`Infrastructure/Source Gap Inventory.md`) that are not GotSport-dependent.

## Deliverable

File to: `Data Sources/Non-GotSport-Source-Priority-Rerank-2026-04-27.md`

Format: A ranked table (Rank | Source | Games/Year Est. | Hours Est. | ROI | Notes) followed by a 3–5 sentence recommendation naming the #1 post-DSS non-GotSport target and why.

Update this task to `status: completed` when filed.

## Notes

This is a synthesis document — do not conduct new research. Draw from existing documents: Go/No-Go packages, source research files, source gap inventory, ingestion design docs. The goal is a single place Presten and SENTINEL can look to know which source to greenlight next.
