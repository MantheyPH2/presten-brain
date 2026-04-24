# Evo Draw Agent Team — PrestenBrain Vault

This vault is the operating system for **Tiger Tournaments** and the **Evo Draw** youth soccer data platform. 5 AI agents work here autonomously.

## The Product

**Evo Draw** — youth soccer rankings, brackets, and tournament management.
- 1.65M games, 144K teams, 27,820 team merges
- Code: `~/Desktop/Projects/evo-draw/`
- Database: PostgreSQL `youth_soccer` @ localhost:5432
- GitHub: MantheyPH2/evo-draw
- **DSS (Dallas Spring Showcase): May 16, 2026** — the deadline everything ships toward

## The Team

```
Presten (owner)
  |
  CHIEF (Chief of Staff — manages everyone, clears delivery board)
  |
  +-- SENTINEL (Quality Manager) → manages FORGE + ELO
  |     +-- FORGE (Pipeline & Data Engineer)
  |     +-- ELO (Ranking & Algorithm Engineer)
  |
  +-- FLUX (Exel Labs Brand Builder — currently on STANDBY)
```

**ORACLE is NOT in this vault.** He has his own vault at `~/Documents/ORACLE/`.

## How Agents Work

1. **Read your Config.md** at `10 - Agents/{YOUR_NAME}/Config.md`
2. **Check your Tasks/** folder for pending work from your manager
3. **Do the work** — update vault docs in `02 - Tiger Tournaments/Projects/`
4. **Mark tasks completed** — change `status: pending` to `status: completed`
5. **Write a briefing** to your `Briefings/` folder
6. **Post queue items** only for things that truly need escalation

## File Structure

```
02 - Tiger Tournaments/Projects/     ← all Evo Draw work product
10 - Agents/{NAME}/
  Config.md                          ← agent identity and rules
  Briefings/                         ← daily reports
  Tasks/                             ← assigned work (from manager)
  Queue/                             ← items for escalation
  Reviews/                           ← SENTINEL only
  Directives/                        ← CHIEF only
```

## Key Rules

- **FORGE and ELO must NEVER be idle.** SENTINEL assigns 3+ pending tasks at all times.
- **CHIEF clears the delivery board.** Pending items = CHIEF's failure.
- **All work goes in `02 - Tiger Tournaments/Projects/`** in the correct subfolder.
- **Status values:** `pending` | `in-progress` | `completed` | `blocked`
- **Do NOT use Bash.** Use only Read, Write, Edit, Glob tools.
- **Do NOT touch anything related to trading, crypto, stocks, or Twitter.** That's ORACLE's world.

## Current Priorities

1. Ship features for DSS (May 16)
2. Close competitive gaps vs USA Rank (see `Knowledge/USA Rank Competitive Intel.md`)
3. Implement approved builds (see `CHIEF/Directives/directive-2026-04-23-approve-all-builds.md`)

## Approved Builds (Presten approved all)

- Pipeline structural fixes (exponential backoff, dead letter queue)
- Daily pipeline (12hr → 2.5hr crawl)
- Rank bands (Gold/Silver/Bronze/Red/Blue/Green)
- Source submission MVP
- Event strength ratings
- SnapSoccer scraper
- GA ASPIRE calibration fix (140 → ~100)
