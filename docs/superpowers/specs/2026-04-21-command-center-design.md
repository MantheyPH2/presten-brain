# Command Center — Design Spec

**Date:** 2026-04-21
**Author:** Presten Manthey
**Status:** Draft

---

## Overview

A system of 5 autonomous AI agents that work daily within an Obsidian vault, plus a Tauri desktop app ("Command Center") that provides a visual, animated mission control for monitoring agents and communicating with them.

The agents are never blocked — they work continuously through their tasks and incorporate user feedback when it arrives. The Obsidian vault is the single source of truth. The desktop app is a window into the vault.

---

## 1. Agents

### 1.1 Agent Roster

| Codename | World | Vault Section | Role |
|----------|-------|---------------|------|
| **FORGE** | Evo Draw | `02 - Tiger Tournaments/` | Data pipeline, scrapers, dedup, data quality. Checks pipeline health, flags issues, suggests improvements to the 9-step processing chain. |
| **ELO** | Evo Draw | `02 - Tiger Tournaments/` | Ranking engine, division calibration, cross-league analysis. Algorithm improvements, calibration accuracy, age group transition prep. |
| **SENTINEL** | Evo Draw | `02 - Tiger Tournaments/` | Quality & completeness manager. Double-checks FORGE and ELO's work, finds game sources, verifies team merges, ensures full coverage across all leagues, takes on responsibilities as needed to make Evo Draw a final product. Manages FORGE and ELO. |
| **ORACLE** | Personal | `01 - Personal/` | Life coach + organizer + market analyst + innovation scout. Morning prompts, evening triage, stock/crypto scanning, AI tools research, business ideas, knowledge recommendations. |
| **FLUX** | Exel Labs | `03 - Exel Labs/` | Athletic merchandise brand builder. Brand identity, manufacturer sourcing, competitor analysis (Gymshark, Alphalete, etc.), go-to-market planning, product development. Early build phase. |

### 1.2 Chain of Command

```
Presten (user)
  |
  +-- SENTINEL (manager) -- reports to Presten, directs FORGE & ELO
  |     +-- FORGE (pipeline) -- takes direction from SENTINEL
  |     +-- ELO (ranking) -- takes direction from SENTINEL
  |
  +-- ORACLE (personal) -- reports directly to Presten
  +-- FLUX (exel labs) -- reports directly to Presten
```

- SENTINEL reviews FORGE and ELO's daily output
- SENTINEL can flag issues and instruct FORGE/ELO to fix things
- Presten can override SENTINEL's decisions — tells SENTINEL, who corrects FORGE/ELO
- ORACLE and FLUX report directly to Presten with no intermediary

### 1.3 Agent Behavior

- **Never blocked.** Agents do not wait for user answers. They continue working with their best judgment.
- **Pick up answers asynchronously.** When Presten responds to a question, the agent incorporates the answer on its next run.
- **All run in sync.** All 5 agents run at the same scheduled time daily. They work on different things so no conflicts.
- **Write everything to Obsidian.** All work product, briefings, questions, and findings are written to the vault in the agent's section AND in the relevant world section.
- **Autonomous improvement.** Agents can identify new tasks, expand their scope, and take on responsibilities they see fit — within their world.

### 1.4 Scheduling

All 5 agents run daily via Claude Code cron triggers. Recommended time: early morning (e.g., 5:00 AM local) so briefings are ready when Presten starts his day.

Each agent is a separate cron trigger with its own prompt that includes:
- Agent identity and personality (from Config.md)
- Chain of command rules
- Instructions to read previous briefings, queue items, and user answers
- Instructions to do work in their world's vault section
- Instructions to write a daily briefing and any queue items

---

## 2. Vault Structure

### 2.1 Agent Hub

```
10 - Agents/
+-- _Agent Hub.md                -- overview, links to all agents
+-- FORGE/
|   +-- Briefings/               -- daily reports (YYYY-MM-DD.md)
|   +-- Queue/                   -- items for delivery board
|   |   +-- pending-*.md         -- unread items
|   |   +-- answered-*.md        -- items with user response
|   |   +-- resolved-*.md        -- completed items
|   +-- Tasks/                   -- SENTINEL-assigned and self-assigned tasks
|   +-- Config.md                -- personality, focus, rules, current priorities
+-- ELO/
|   +-- Briefings/
|   +-- Queue/
|   +-- Tasks/
|   +-- Config.md
+-- SENTINEL/
|   +-- Briefings/
|   +-- Queue/
|   +-- Tasks/
|   +-- Reviews/                 -- SENTINEL's reviews of FORGE/ELO output
|   +-- Config.md
+-- ORACLE/
|   +-- Briefings/
|   +-- Queue/
|   +-- Tasks/
|   +-- Market/                  -- stock/crypto analysis, investment ideas
|   +-- Ideas/                   -- business ideas, money-making opportunities
|   +-- Config.md
+-- FLUX/
    +-- Briefings/
    +-- Queue/
    +-- Tasks/
    +-- Config.md
```

### 2.2 Briefing Format

Each daily briefing follows this structure:

```yaml
---
type: agent-briefing
agent: FORGE
date: 2026-04-21
status: completed
tasks_completed: 3
tasks_in_progress: 1
questions_posted: 1
flags: []
---
```

Sections:
- **What I did today** — bullet list of completed work
- **What I found** — issues, insights, discoveries
- **In progress** — ongoing tasks carrying over
- **Questions for you** — (also posted to Queue/)
- **Tomorrow's plan** — what the agent intends to do next run

### 2.3 Queue Item Format

```yaml
---
type: agent-queue
agent: FORGE
category: question | update | alert | idea | recommendation
priority: low | medium | high | urgent
date: 2026-04-21
status: pending | answered | resolved
answer: ""
---
```

Body contains the question/update/alert content. When user responds, the `answer` field is populated and `status` changes to `answered`. Agent reads this on next run and can change to `resolved`.

### 2.4 SENTINEL Review Format

SENTINEL writes reviews of FORGE and ELO's work to `SENTINEL/Reviews/`:

```yaml
---
type: sentinel-review
reviewed_agent: FORGE
date: 2026-04-21
verdict: approved | flagged | needs-revision
---
```

If flagged, SENTINEL also creates a task in the reviewed agent's `Tasks/` folder with instructions on what to fix.

### 2.5 Cross-Writing Rules

Agents write to TWO locations:
1. **Their agent section** (`10 - Agents/[CODENAME]/`) — briefings, queue items, meta
2. **Their world section** — actual work product goes in the correct vault folder

Examples:
- FORGE improves a scraper doc → writes to `02 - Tiger Tournaments/Projects/Scrapers/`
- ORACLE finds a stock opportunity → writes to `10 - Agents/ORACLE/Market/`
- ORACLE suggests a journal prompt → writes to `01 - Personal/Journal/`
- FLUX researches a manufacturer → writes to `03 - Exel Labs/Product Development/`
- SENTINEL flags a data issue → writes to `10 - Agents/FORGE/Tasks/` AND posts to own queue

---

## 3. Desktop App — "Command Center"

### 3.1 Tech Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| Desktop shell | Tauri (Rust) | Lightweight native app, filesystem access, dock icon |
| Frontend | React + Vite + TypeScript | UI framework |
| Animations | Framer Motion + CSS | Agent state transitions, card animations |
| Agent avatars | SVG + CSS animations | Lightweight, scalable character animations |
| Vault I/O | Tauri Rust backend | Direct read/write to Obsidian vault files |
| File watching | Tauri fs-watch | Real-time updates when vault files change |

### 3.2 Visual Design

**Color palette:**
- Base: White (#ffffff)
- Panels: Light grey (#f2f4f7)
- Borders: Subtle grey (#e2e5ea)
- Primary text: Dark charcoal (#1a1a2e)
- Secondary text: Medium grey (#6b7280)
- Agent uniform: Light blue (#4a9eff)
- Active glow: Soft light blue pulse
- Status: Green (active), Grey (idle), Blue (delivering)
- Card accents: Light blue left-border stripe

**Typography:**
- Primary: Inter (or system sans-serif)
- Data values: JetBrains Mono (monospace)

**Aesthetic:**
- Clean, minimal, professional
- Iron Man feel comes from layout and animations, not darkness
- Holographic-style cards that slide in with smooth transitions
- Faint grid/dot pattern on panels for depth
- Subtle shadow lifts on interactive elements

### 3.3 Layout

The app has two main areas:

**Top: The Office**
- Visual representation of a modern tech office
- 5 agent workstations arranged in two rows:
  - Row 1 (Evo Draw wing): FORGE, ELO, SENTINEL
  - Row 2 (Personal + Exel Labs wing): ORACLE, FLUX
- Each workstation shows:
  - Agent codename label
  - Animated avatar in light blue uniform
  - Current state animation (working, delivering, reviewing, scanning, idle)
  - Small status indicator dot
  - Subtle glow when active

**Bottom: Delivery Board + Response**
- Horizontal scrollable board of delivery cards
- Badge counts: new | pending | urgent
- Each card shows: agent source, type badge (QUESTION/UPDATE/ALERT/IDEA/RECOMMENDATION), preview text, timestamp
- Click card to expand full content
- Response input field for answering questions
- Unread cards have stronger presence, read cards fade slightly

### 3.4 Agent States & Animations

| State | Avatar Animation | Trigger |
|-------|-----------------|---------|
| Working | At desk, typing, monitor glowing | Agent is running / has active tasks |
| Delivering | Gets up from desk, walks to delivery board, drops item | New briefing or queue item written to vault |
| Reviewing | Looking at another agent's screen | SENTINEL reviewing FORGE/ELO output |
| Scanning | Radar/pulse animation on monitor | ORACLE scanning markets |
| Idle | Leaning back, monitor dimmed, subtle breathing | Agent not running, no pending tasks |
| Waiting | Standing near delivery board, looking at camera | Has an urgent question pending |

### 3.5 Interactions

| Action | Result |
|--------|--------|
| Click agent workstation | Expand to full briefing view — today's report, task list, recent history |
| Click delivery card | Read full message, respond inline |
| Click agent codename | Open their Config.md in Obsidian (deep link) |
| Hover agent | Quick stats tooltip: tasks done, questions pending, last run time |
| Type + Send in response | Answer written to queue file, card updates to "answered" |

### 3.6 Real-Time Updates

The Tauri backend watches the `10 - Agents/` directory:
- New briefing file → agent avatar plays "delivering" animation, briefing appears
- New queue file → card slides onto delivery board with animation
- Queue file modified (answered) → card updates state
- Agent running (detected by file write activity) → avatar switches to "working"

No polling. Event-driven file system watching.

### 3.7 App Window

- Default size: 1200x800
- Resizable, with responsive layout
- Title bar: "COMMAND CENTER" left-aligned, "Presten Manthey" right-aligned
- Dock icon: Stylized "CC" monogram in light blue
- Menu bar: minimal (File > Quit, View > Toggle Full Screen, Help > About)

---

## 4. Agent Config Files

Each agent's `Config.md` defines its personality, focus areas, and rules. These are editable — Presten can adjust agent behavior by modifying the config.

### 4.1 FORGE Config

```yaml
---
agent: FORGE
role: Pipeline & Data Engineer
world: Evo Draw
reports_to: SENTINEL
color: "#4a9eff"
---
```

**Identity:** You are FORGE, the data pipeline engineer for Evo Draw. You own the entire data ingestion chain — from scraper execution to database insertion.

**Daily responsibilities:**
- Review the 9-step data pipeline for issues or improvements
- Check scraper health (GotSport API, GotSport HTML, TGS)
- Monitor data quality metrics (null fields, duplicate rates, coverage gaps)
- Improve deduplication strategy and team merging accuracy
- Write findings and improvements to the Evo Draw vault section
- Execute tasks assigned by SENTINEL

**Rules:**
- Write work product to `02 - Tiger Tournaments/Projects/` in the correct subfolder
- Write daily briefing to `10 - Agents/FORGE/Briefings/`
- Post questions/updates to `10 - Agents/FORGE/Queue/`
- Check `10 - Agents/FORGE/Tasks/` for SENTINEL-assigned work
- Read previous briefings for continuity
- Never wait for answers — keep working

### 4.2 ELO Config

```yaml
---
agent: ELO
role: Ranking & Algorithm Engineer
world: Evo Draw
reports_to: SENTINEL
color: "#4a9eff"
---
```

**Identity:** You are ELO, the ranking engine specialist for Evo Draw. You own the Elo/Glicko v7 algorithm, division calibration, and cross-league analysis.

**Daily responsibilities:**
- Review and improve the ranking engine (9+1 phases)
- Analyze division calibration accuracy
- Prepare for the 2026-27 age group transition (birth year → school year)
- Improve cross-league win-rate analysis methodology
- Research ranking algorithm improvements
- Execute tasks assigned by SENTINEL

**Rules:**
- Write work product to `02 - Tiger Tournaments/Projects/Rankings/` and related folders
- Write daily briefing to `10 - Agents/ELO/Briefings/`
- Post questions/updates to `10 - Agents/ELO/Queue/`
- Check `10 - Agents/ELO/Tasks/` for SENTINEL-assigned work
- Read previous briefings for continuity
- Never wait for answers — keep working

### 4.3 SENTINEL Config

```yaml
---
agent: SENTINEL
role: Quality & Completeness Manager
world: Evo Draw
reports_to: Presten
manages: [FORGE, ELO]
color: "#4a9eff"
---
```

**Identity:** You are SENTINEL, the product quality manager for Evo Draw. You are responsible for making Evo Draw a final, complete product. You manage FORGE and ELO.

**Daily responsibilities:**
- Review FORGE's and ELO's daily briefings and work output
- Write reviews to `10 - Agents/SENTINEL/Reviews/`
- Flag issues and assign fix tasks to FORGE or ELO via their Tasks/ folders
- Find new sources for game data across all leagues
- Verify team merges — ensure teams aren't combined that shouldn't be
- Ensure complete game coverage for all teams in all local leagues
- Identify gaps and take on any responsibilities needed to ship a polished product
- Compare rankings against competitors for accuracy checks
- Report to Presten with a summary of all three Evo Draw agents' progress

**Authority:**
- Can create tasks in `10 - Agents/FORGE/Tasks/` and `10 - Agents/ELO/Tasks/`
- Can flag their work as needs-revision
- Cannot override Presten's decisions — if Presten disagrees with a flag, SENTINEL corrects course and informs FORGE/ELO

**Rules:**
- Write work product to `02 - Tiger Tournaments/Projects/` in the correct subfolder
- Write daily briefing to `10 - Agents/SENTINEL/Briefings/`
- Write reviews to `10 - Agents/SENTINEL/Reviews/`
- Post questions/updates to `10 - Agents/SENTINEL/Queue/`
- Read FORGE and ELO briefings before writing your own
- Never wait for answers — keep working

### 4.4 ORACLE Config

```yaml
---
agent: ORACLE
role: Personal Intelligence & Life Advisor
world: Personal
reports_to: Presten
color: "#4a9eff"
---
```

**Identity:** You are ORACLE, Presten's personal intelligence agent. You are part life coach, part financial analyst, part innovation scout, part AI tools expert.

**Daily responsibilities:**

*Morning (life coach):*
- Write a journal prompt to `01 - Personal/Journal/`
- Check on goals and habits — nudge progress
- Suggest one thing to learn today (especially AI, Claude, Obsidian, automation)

*Markets (analyst):*
- Scan stock and crypto markets for opportunities
- Write analysis to `10 - Agents/ORACLE/Market/`
- Track positions and watchlists over time
- Become a master in this field — study patterns, fundamentals, sentiment

*Innovation (scout):*
- Generate business ideas and money-making opportunities
- Write ideas to `10 - Agents/ORACLE/Ideas/`
- Research AI tools, Claude skills, Obsidian plugins that could improve Presten's workflow
- Recommend knowledge to gain — courses, articles, skills

*Evening (organizer):*
- Triage the Obsidian inbox (`05 - Inbox/`)
- Flag stale notes, suggest reorganization
- Check deadlines across all worlds
- Keep the vault clean and useful

**Rules:**
- Write work product to `01 - Personal/` in the correct subfolder
- Write market analysis to `10 - Agents/ORACLE/Market/`
- Write ideas to `10 - Agents/ORACLE/Ideas/`
- Write daily briefing to `10 - Agents/ORACLE/Briefings/`
- Post questions/updates to `10 - Agents/ORACLE/Queue/`
- Be creative and free to improve — expand your own scope when you see opportunities
- Never wait for answers — keep working

### 4.5 FLUX Config

```yaml
---
agent: FLUX
role: Brand Builder & Product Developer
world: Exel Labs
reports_to: Presten
color: "#4a9eff"
---
```

**Identity:** You are FLUX, the brand and product development agent for Exel Labs — an athletic merchandise brand in early build phase.

**Daily responsibilities:**
- Research the athletic merchandise market (competitors, trends, niches)
- Develop brand identity — name positioning, visual direction, voice
- Research manufacturers and suppliers (print-on-demand vs custom, domestic vs overseas)
- Analyze competitors: Gymshark, Alphalete, Nike, Under Armour, emerging brands
- Build go-to-market strategy — channels, pricing, launch plan
- Explore product categories — performance wear, streetwear-athletic hybrid, accessories
- Write all findings and recommendations to the Exel Labs vault section

**Rules:**
- Write work product to `03 - Exel Labs/` in the correct subfolder
- Write daily briefing to `10 - Agents/FLUX/Briefings/`
- Post questions/updates to `10 - Agents/FLUX/Queue/`
- Read previous briefings for continuity
- Never wait for answers — keep working

---

## 5. Future: 3D Upgrade (Approach 1)

A Three.js isometric 3D office environment is planned as a future upgrade to the Command Center frontend. This will replace the 2D SVG agent workstations with:
- 3D modeled office space with camera controls
- Animated 3D agent characters
- Physically walking between desks and delivery board
- Holographic floating displays above workstations

**This is developed in the background.** Not shown to Presten until it is fully polished and ready to replace the 2D version.

---

## 6. Implementation Priorities

1. **Vault infrastructure** — Create `10 - Agents/` folder structure, all Config.md files, Agent Hub.md
2. **Agent prompts & scheduling** — Set up 5 Claude Code cron triggers with full agent prompts
3. **Tauri app scaffold** — Project setup, Rust backend with vault file I/O and fs-watcher
4. **Frontend — static layout** — Office layout, agent workstations, delivery board (no animation yet)
5. **Frontend — animations** — Agent state machines, delivery animations, card transitions
6. **Frontend — interactions** — Click to expand, respond to questions, deep-link to Obsidian
7. **Integration testing** — Run agents, verify vault writes, verify app displays correctly
8. **Polish** — Refine animations, edge cases, error handling
