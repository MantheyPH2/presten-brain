# Command Center Tauri App — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a Tauri desktop app ("Command Center") that visualizes 5 AI agents working in an animated office, with a delivery board for communication and inline response capability.

**Architecture:** Tauri (Rust backend for filesystem access + file watching) with React + Vite + TypeScript frontend. SVG agent avatars with CSS/Framer Motion animations. Real-time vault file watching triggers UI updates. No server or database �� the Obsidian vault IS the data layer.

**Tech Stack:** Tauri v2, Rust, React 18, Vite, TypeScript, Framer Motion, Tailwind CSS, gray-matter (frontmatter parsing), chokidar (via Tauri fs-watch)

---

### Task 1: Install Prerequisites

- [ ] **Step 1: Install Rust and Cargo**

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
source "$HOME/.cargo/env"
rustc --version
cargo --version
```

Expected: Rust 1.7x+ and Cargo installed.

- [ ] **Step 2: Install Tauri CLI**

```bash
cargo install tauri-cli
cargo tauri --version
```

Expected: `tauri-cli 2.x.x`

- [ ] **Step 3: Verify Node.js and npm**

```bash
node -v  # should be 18+
npm -v   # should be 9+
```

- [ ] **Step 4: Commit — no files, just verify tools**

---

### Task 2: Scaffold Tauri + React Project

**Files:**
- Create: `~/Desktop/Projects/command-center/` (full project scaffold)

- [ ] **Step 1: Create the project**

```bash
cd ~/Desktop/Projects
npm create tauri-app@latest command-center -- --template react-ts
cd command-center
npm install
```

- [ ] **Step 2: Install additional dependencies**

```bash
npm install framer-motion tailwindcss @tailwindcss/vite gray-matter
npm install -D @types/node
```

- [ ] **Step 3: Configure Tailwind**

Update `vite.config.ts`:

```typescript
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import tailwindcss from "@tailwindcss/vite";

export default defineConfig({
  plugins: [react(), tailwindcss()],
  clearScreen: false,
  server: {
    port: 1420,
    strictPort: true,
  },
});
```

Replace contents of `src/index.css`:

```css
@import "tailwindcss";
```

- [ ] **Step 4: Configure Tauri for filesystem access**

Update `src-tauri/capabilities/default.json` to include filesystem and watcher permissions:

```json
{
  "$schema": "../gen/schemas/desktop-schema.json",
  "identifier": "default",
  "description": "Capability for the main window",
  "windows": ["main"],
  "permissions": [
    "core:default",
    "core:window:default",
    "core:window:allow-set-title",
    "fs:default",
    "fs:allow-read-text-file",
    "fs:allow-write-text-file",
    "fs:allow-exists",
    "fs:allow-read-dir",
    "fs:allow-create",
    "fs:allow-mkdir",
    {
      "identifier": "fs:scope",
      "allow": [
        { "path": "$HOME/Documents/Obsidian Vault/**" }
      ]
    }
  ]
}
```

- [ ] **Step 5: Configure Tauri window**

Update `src-tauri/tauri.conf.json` — set app name, window title, default size:

```json
{
  "productName": "Command Center",
  "version": "0.1.0",
  "identifier": "com.prestenbrain.commandcenter",
  "build": {
    "frontendDist": "../dist"
  },
  "app": {
    "windows": [
      {
        "title": "COMMAND CENTER",
        "width": 1200,
        "height": 800,
        "resizable": true,
        "minWidth": 900,
        "minHeight": 600
      }
    ]
  }
}
```

- [ ] **Step 6: Verify scaffold runs**

```bash
cd ~/Desktop/Projects/command-center
npm run tauri dev
```

Expected: Tauri window opens with default React template.

- [ ] **Step 7: Commit**

```bash
cd ~/Desktop/Projects/command-center
git init && git add -A && git commit -m "feat: scaffold Tauri + React + Tailwind project"
```

---

### Task 3: Vault Data Layer

**Files:**
- Create: `src/lib/vault.ts` — reads/writes vault files via Tauri fs API
- Create: `src/lib/types.ts` — TypeScript types for agents, briefings, queue items
- Create: `src/lib/constants.ts` — vault paths, agent definitions, colors

- [ ] **Step 1: Create types**

Write `src/lib/types.ts`:

```typescript
export type AgentName = "FORGE" | "ELO" | "SENTINEL" | "ORACLE" | "FLUX";

export type AgentState =
  | "working"
  | "delivering"
  | "reviewing"
  | "scanning"
  | "idle"
  | "waiting";

export type QueueCategory =
  | "question"
  | "update"
  | "alert"
  | "idea"
  | "recommendation";

export type QueuePriority = "low" | "medium" | "high" | "urgent";

export type QueueStatus = "pending" | "answered" | "resolved";

export interface AgentConfig {
  name: AgentName;
  role: string;
  world: string;
  reportsTo: string;
  manages?: AgentName[];
  color: string;
}

export interface Briefing {
  agent: AgentName;
  date: string;
  status: string;
  tasksCompleted: number;
  tasksInProgress: number;
  questionsPosted: number;
  flags: string[];
  content: string;
  filePath: string;
}

export interface QueueItem {
  agent: AgentName;
  category: QueueCategory;
  priority: QueuePriority;
  date: string;
  status: QueueStatus;
  answer: string;
  content: string;
  title: string;
  filePath: string;
}

export interface AgentData {
  config: AgentConfig;
  state: AgentState;
  latestBriefing: Briefing | null;
  pendingQueue: QueueItem[];
}
```

- [ ] **Step 2: Create constants**

Write `src/lib/constants.ts`:

```typescript
import { AgentConfig } from "./types";

export const VAULT_PATH = "Documents/Obsidian Vault";
export const AGENTS_PATH = `${VAULT_PATH}/10 - Agents`;

export const AGENTS: AgentConfig[] = [
  {
    name: "FORGE",
    role: "Pipeline & Data",
    world: "Evo Draw",
    reportsTo: "SENTINEL",
    color: "#4a9eff",
  },
  {
    name: "ELO",
    role: "Ranking & Algorithm",
    world: "Evo Draw",
    reportsTo: "SENTINEL",
    color: "#4a9eff",
  },
  {
    name: "SENTINEL",
    role: "Quality & Completeness",
    world: "Evo Draw",
    reportsTo: "Presten",
    manages: ["FORGE", "ELO"],
    color: "#4a9eff",
  },
  {
    name: "ORACLE",
    role: "Personal Intelligence",
    world: "Personal",
    reportsTo: "Presten",
    color: "#4a9eff",
  },
  {
    name: "FLUX",
    role: "Brand Builder",
    world: "Exel Labs",
    reportsTo: "Presten",
    color: "#4a9eff",
  },
];

export const WING_LAYOUT = {
  evoDraw: ["FORGE", "ELO", "SENTINEL"] as const,
  other: ["ORACLE", "FLUX"] as const,
};
```

- [ ] **Step 3: Create vault reader**

Write `src/lib/vault.ts`:

```typescript
import { readTextFile, writeTextFile, readDir, exists } from "@tauri-apps/plugin-fs";
import { homeDir } from "@tauri-apps/api/path";
import { AGENTS_PATH } from "./constants";
import type {
  AgentName,
  Briefing,
  QueueItem,
  QueueStatus,
} from "./types";

function parseFrontmatter(content: string): { data: Record<string, any>; body: string } {
  const match = content.match(/^---\n([\s\S]*?)\n---\n?([\s\S]*)$/);
  if (!match) return { data: {}, body: content };

  const data: Record<string, any> = {};
  for (const line of match[1].split("\n")) {
    const idx = line.indexOf(":");
    if (idx === -1) continue;
    const key = line.slice(0, idx).trim();
    let val: any = line.slice(idx + 1).trim();
    if (val === "true") val = true;
    else if (val === "false") val = false;
    else if (/^\d+$/.test(val)) val = parseInt(val, 10);
    else if (val.startsWith('"') && val.endsWith('"')) val = val.slice(1, -1);
    else if (val.startsWith("[") && val.endsWith("]")) {
      val = val.slice(1, -1).split(",").map((s: string) => s.trim());
    }
    data[key] = val;
  }
  return { data, body: match[2] };
}

async function getBasePath(): Promise<string> {
  const home = await homeDir();
  return home;
}

export async function readAgentBriefings(agent: AgentName): Promise<Briefing[]> {
  const base = await getBasePath();
  const dir = `${base}${AGENTS_PATH}/${agent}/Briefings`;

  const dirExists = await exists(dir);
  if (!dirExists) return [];

  const entries = await readDir(dir);
  const briefings: Briefing[] = [];

  for (const entry of entries) {
    if (!entry.name?.endsWith(".md")) continue;
    const filePath = `${dir}/${entry.name}`;
    const content = await readTextFile(filePath);
    const { data, body } = parseFrontmatter(content);

    briefings.push({
      agent: data.agent || agent,
      date: data.date || entry.name.replace(".md", ""),
      status: data.status || "unknown",
      tasksCompleted: data.tasks_completed || 0,
      tasksInProgress: data.tasks_in_progress || 0,
      questionsPosted: data.questions_posted || 0,
      flags: data.flags || [],
      content: body,
      filePath,
    });
  }

  return briefings.sort((a, b) => b.date.localeCompare(a.date));
}

export async function readAgentQueue(agent: AgentName): Promise<QueueItem[]> {
  const base = await getBasePath();
  const dir = `${base}${AGENTS_PATH}/${agent}/Queue`;

  const dirExists = await exists(dir);
  if (!dirExists) return [];

  const entries = await readDir(dir);
  const items: QueueItem[] = [];

  for (const entry of entries) {
    if (!entry.name?.endsWith(".md")) continue;
    const filePath = `${dir}/${entry.name}`;
    const content = await readTextFile(filePath);
    const { data, body } = parseFrontmatter(content);

    items.push({
      agent: data.agent || agent,
      category: data.category || "update",
      priority: data.priority || "medium",
      date: data.date || "",
      status: data.status || "pending",
      answer: data.answer || "",
      content: body,
      title: body.split("\n").find((l: string) => l.startsWith("# "))?.slice(2) || entry.name.replace(".md", ""),
      filePath,
    });
  }

  return items.sort((a, b) => {
    const priorityOrder = { urgent: 0, high: 1, medium: 2, low: 3 };
    return priorityOrder[a.priority] - priorityOrder[b.priority];
  });
}

export async function answerQueueItem(
  filePath: string,
  answer: string
): Promise<void> {
  const content = await readTextFile(filePath);
  const updated = content
    .replace(/status: pending/, "status: answered")
    .replace(/answer: ""/, `answer: "${answer.replace(/"/g, '\\"')}"`);
  await writeTextFile(filePath, updated);
}

export async function getAllPendingQueue(): Promise<QueueItem[]> {
  const agents: AgentName[] = ["FORGE", "ELO", "SENTINEL", "ORACLE", "FLUX"];
  const all: QueueItem[] = [];
  for (const agent of agents) {
    const items = await readAgentQueue(agent);
    all.push(...items.filter((i) => i.status === "pending"));
  }
  return all.sort((a, b) => {
    const priorityOrder = { urgent: 0, high: 1, medium: 2, low: 3 };
    return priorityOrder[a.priority] - priorityOrder[b.priority];
  });
}
```

- [ ] **Step 4: Commit**

```bash
git add src/lib/ && git commit -m "feat: add vault data layer �� types, constants, file reader"
```

---

### Task 4: Agent Workstation Component

**Files:**
- Create: `src/components/AgentWorkstation.tsx` — single agent workstation with avatar and state
- Create: `src/components/AgentAvatar.tsx` — SVG avatar with animation states

- [ ] **Step 1: Create AgentAvatar component**

Write `src/components/AgentAvatar.tsx` — an SVG character in light blue uniform with animation states. The avatar should be a simple stylized figure at a desk:

- Seated at desk: typing animation (hands moving)
- Delivering: standing, walking motion
- Idle: leaned back, subtle breathing
- Scanning: radar pulse on monitor
- Reviewing: turned toward adjacent station

Use CSS keyframe animations for each state. The avatar wears a light blue (#4a9eff) uniform. Simple, clean geometric style.

- [ ] **Step 2: Create AgentWorkstation component**

Write `src/components/AgentWorkstation.tsx`:

```typescript
import { motion } from "framer-motion";
import { AgentAvatar } from "./AgentAvatar";
import type { AgentData } from "../lib/types";

interface Props {
  agent: AgentData;
  onClick: () => void;
}

export function AgentWorkstation({ agent, onClick }: Props) {
  const isActive = agent.state !== "idle";

  return (
    <motion.div
      className="relative flex flex-col items-center cursor-pointer rounded-xl border p-4 transition-all"
      style={{
        borderColor: isActive ? "#4a9eff33" : "#e2e5ea",
        backgroundColor: isActive ? "#f0f7ff" : "#f2f4f7",
        boxShadow: isActive ? "0 0 20px rgba(74, 158, 255, 0.1)" : "none",
      }}
      whileHover={{ scale: 1.02, boxShadow: "0 4px 12px rgba(0,0,0,0.08)" }}
      onClick={onClick}
    >
      {/* Status dot */}
      <div
        className="absolute top-3 right-3 h-2.5 w-2.5 rounded-full"
        style={{
          backgroundColor: isActive ? "#22c55e" : "#9ca3af",
        }}
      />

      {/* Agent avatar at desk */}
      <div className="h-28 w-36">
        <AgentAvatar state={agent.state} />
      </div>

      {/* Agent info */}
      <div className="mt-2 text-center">
        <div className="text-sm font-semibold text-gray-900">
          {agent.config.name}
        </div>
        <div className="text-xs text-gray-500">{agent.config.role}</div>
        <div className="mt-1 text-xs text-gray-400 capitalize">
          {agent.state}
        </div>
      </div>

      {/* Pending count badge */}
      {agent.pendingQueue.length > 0 && (
        <div className="absolute -top-1 -right-1 flex h-5 w-5 items-center justify-center rounded-full bg-blue-500 text-[10px] font-bold text-white">
          {agent.pendingQueue.length}
        </div>
      )}
    </motion.div>
  );
}
```

- [ ] **Step 3: Commit**

```bash
git add src/components/ && git commit -m "feat: add agent workstation and avatar components"
```

---

### Task 5: Delivery Board Component

**Files:**
- Create: `src/components/DeliveryBoard.tsx` — the bottom panel showing queue cards
- Create: `src/components/DeliveryCard.tsx` — individual delivery card
- Create: `src/components/ResponseInput.tsx` — text input for answering questions

- [ ] **Step 1: Create DeliveryCard component**

Write `src/components/DeliveryCard.tsx` — a card showing a queue item with:
- Light blue left border stripe
- Type badge (QUESTION/UPDATE/ALERT/IDEA/RECOMMENDATION) with appropriate colors
- Priority indicator
- Agent name
- Preview text (first 100 chars of content)
- Timestamp
- Click to expand full content
- Framer Motion animate-in (slide from right)

- [ ] **Step 2: Create ResponseInput component**

Write `src/components/ResponseInput.tsx` — a text input that appears when a card is expanded:
- Clean input field with Send button
- On submit: calls `answerQueueItem()` from vault.ts
- Shows "Sent" confirmation briefly
- Card updates to "answered" state

- [ ] **Step 3: Create DeliveryBoard component**

Write `src/components/DeliveryBoard.tsx`:

```typescript
import { motion, AnimatePresence } from "framer-motion";
import { DeliveryCard } from "./DeliveryCard";
import type { QueueItem } from "../lib/types";

interface Props {
  items: QueueItem[];
  onAnswer: (filePath: string, answer: string) => void;
}

export function DeliveryBoard({ items, onAnswer }: Props) {
  const newCount = items.filter((i) => i.status === "pending").length;
  const urgentCount = items.filter((i) => i.priority === "urgent").length;

  return (
    <div className="border-t border-gray-200 bg-white p-4">
      {/* Header */}
      <div className="mb-3 flex items-center gap-4">
        <h2 className="text-sm font-semibold text-gray-900">
          DELIVERY BOARD
        </h2>
        <span className="rounded-full bg-blue-50 px-2 py-0.5 text-xs text-blue-600">
          {newCount} new
        </span>
        {urgentCount > 0 && (
          <span className="rounded-full bg-red-50 px-2 py-0.5 text-xs text-red-600">
            {urgentCount} urgent
          </span>
        )}
      </div>

      {/* Cards */}
      <div className="flex gap-3 overflow-x-auto pb-2">
        <AnimatePresence>
          {items.map((item) => (
            <DeliveryCard
              key={item.filePath}
              item={item}
              onAnswer={onAnswer}
            />
          ))}
        </AnimatePresence>
        {items.length === 0 && (
          <div className="py-8 text-center text-sm text-gray-400 w-full">
            All clear — no pending items.
          </div>
        )}
      </div>
    </div>
  );
}
```

- [ ] **Step 4: Commit**

```bash
git add src/components/ && git commit -m "feat: add delivery board, cards, and response input"
```

---

### Task 6: Agent Detail Panel

**Files:**
- Create: `src/components/AgentDetail.tsx` — expanded view when clicking an agent

- [ ] **Step 1: Create AgentDetail component**

Write `src/components/AgentDetail.tsx` — a slide-out panel or modal showing:
- Agent name, role, world
- Current state with animation
- Today's briefing (rendered markdown)
- Recent briefing history (last 7 days)
- Pending queue items for this agent
- Task list
- Link to open Config.md in Obsidian (via `open` URL scheme: `obsidian://open?vault=Obsidian%20Vault&file=...`)
- Close button

Use Framer Motion for slide-in animation from the right.

- [ ] **Step 2: Commit**

```bash
git add src/components/AgentDetail.tsx && git commit -m "feat: add agent detail panel"
```

---

### Task 7: Main App Layout

**Files:**
- Modify: `src/App.tsx` — main layout assembling all components
- Create: `src/hooks/useVaultData.ts` — hook that loads and refreshes vault data

- [ ] **Step 1: Create useVaultData hook**

Write `src/hooks/useVaultData.ts` — a React hook that:
- Loads all agent briefings and queue items on mount
- Polls every 30 seconds for changes (fallback for file watcher)
- Returns `{ agents: AgentData[], pendingQueue: QueueItem[], loading: boolean }`
- Determines agent state based on latest briefing timestamp and queue activity

- [ ] **Step 2: Write main App layout**

Write `src/App.tsx`:

```typescript
import { useState } from "react";
import { useVaultData } from "./hooks/useVaultData";
import { AgentWorkstation } from "./components/AgentWorkstation";
import { DeliveryBoard } from "./components/DeliveryBoard";
import { AgentDetail } from "./components/AgentDetail";
import { answerQueueItem } from "./lib/vault";
import { WING_LAYOUT } from "./lib/constants";
import type { AgentName } from "./lib/types";

export default function App() {
  const { agents, pendingQueue, loading } = useVaultData();
  const [selectedAgent, setSelectedAgent] = useState<AgentName | null>(null);

  const getAgent = (name: string) =>
    agents.find((a) => a.config.name === name);

  const handleAnswer = async (filePath: string, answer: string) => {
    await answerQueueItem(filePath, answer);
  };

  if (loading) {
    return (
      <div className="flex h-screen items-center justify-center bg-white">
        <div className="text-gray-400">Loading Command Center...</div>
      </div>
    );
  }

  return (
    <div className="flex h-screen flex-col bg-white">
      {/* Header */}
      <header className="flex items-center justify-between border-b border-gray-200 px-6 py-3">
        <h1 className="text-sm font-bold tracking-widest text-gray-900">
          COMMAND CENTER
        </h1>
        <span className="text-xs text-gray-400">Presten Manthey</span>
      </header>

      {/* Office */}
      <main className="flex-1 overflow-auto p-6">
        {/* Evo Draw Wing */}
        <div className="mb-2">
          <span className="text-[10px] font-medium uppercase tracking-widest text-gray-400">
            Evo Draw Wing
          </span>
        </div>
        <div className="mb-6 grid grid-cols-3 gap-4">
          {WING_LAYOUT.evoDraw.map((name) => {
            const agent = getAgent(name);
            return agent ? (
              <AgentWorkstation
                key={name}
                agent={agent}
                onClick={() => setSelectedAgent(name)}
              />
            ) : null;
          })}
        </div>

        {/* Personal + Exel Labs Wing */}
        <div className="mb-2">
          <span className="text-[10px] font-medium uppercase tracking-widest text-gray-400">
            Personal & Exel Labs Wing
          </span>
        </div>
        <div className="grid grid-cols-2 gap-4 max-w-[66%]">
          {WING_LAYOUT.other.map((name) => {
            const agent = getAgent(name);
            return agent ? (
              <AgentWorkstation
                key={name}
                agent={agent}
                onClick={() => setSelectedAgent(name)}
              />
            ) : null;
          })}
        </div>
      </main>

      {/* Delivery Board */}
      <DeliveryBoard items={pendingQueue} onAnswer={handleAnswer} />

      {/* Agent Detail Panel */}
      {selectedAgent && (
        <AgentDetail
          agent={getAgent(selectedAgent)!}
          onClose={() => setSelectedAgent(null)}
        />
      )}
    </div>
  );
}
```

- [ ] **Step 3: Commit**

```bash
git add src/ && git commit -m "feat: assemble main app layout with office wings and delivery board"
```

---

### Task 8: Build, Test, and Polish

- [ ] **Step 1: Run the app in dev mode**

```bash
cd ~/Desktop/Projects/command-center
npm run tauri dev
```

Verify: Window opens, agents display in office layout, delivery board shows at bottom.

- [ ] **Step 2: Test with sample data**

Create sample briefing and queue files in the vault to test rendering:

```bash
V="/Users/p/Documents/Obsidian Vault"
# Sample FORGE briefing
cat > "$V/10 - Agents/FORGE/Briefings/2026-04-21.md" << 'EOF'
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

# FORGE Daily Briefing — 2026-04-21

## What I did today
- Reviewed GotSport API scraper performance
- Identified 847 teams with stale data (>30 days since last game)
- Improved dedup matching threshold from 0.85 to 0.82

## What I found
- TGS scraper returning null names for 12 ECNL RL teams
- GotSport API rate limit appears to have changed (seeing 429s at 800ms)

## In progress
- Investigating null-name TGS issue

## Tomorrow's plan
- Fix TGS null-name issue
- Test new dedup threshold on full dataset
EOF

# Sample FORGE queue item
cat > "$V/10 - Agents/FORGE/Queue/pending-2026-04-21-rate-limit.md" << 'EOF'
---
type: agent-queue
agent: FORGE
category: question
priority: high
date: 2026-04-21
status: pending
answer: ""
---

# GotSport API Rate Limit Change

I'm seeing HTTP 429 responses at 800ms delay. The previous 1200ms delay worked fine. Should I:

A) Increase delay to 1500ms (safer but slower crawl)
B) Add exponential backoff (smarter but more complex)
C) Leave at 1200ms and retry on 429 (current behavior may be temporary)

What's your preference?
EOF
```

Verify in the app: FORGE shows as active, briefing renders, queue card appears on delivery board.

- [ ] **Step 3: Test response flow**

Click the queue card, type a response, hit Send. Verify:
- Queue file updates with the answer
- Status changes from pending to answered
- Card updates in the UI

- [ ] **Step 4: Build production binary**

```bash
cd ~/Desktop/Projects/command-center
npm run tauri build
```

Expected: `.dmg` or `.app` file in `src-tauri/target/release/bundle/`

- [ ] **Step 5: Test production build**

Open the built `.app` from the bundle directory. Verify it works identically to dev mode.

- [ ] **Step 6: Commit final state**

```bash
git add -A && git commit -m "feat: Command Center v0.1 — working desktop app with agent visualization"
```
