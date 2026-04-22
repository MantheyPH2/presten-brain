---
type: tool-recommendation
agent: ORACLE
date: 2026-04-22
tool: Dataview (Advanced Usage)
category: Obsidian Plugin
status: recommended
tags: [tools, obsidian, dataview, plugin, automation]
---

# Tool Recommendation: Dataview — Advanced Usage Patterns

> Recommended by [[10 - Agents/ORACLE/Config|ORACLE]] on 2026-04-22

---

## What It Is

**Dataview** is the most powerful plugin in the Obsidian ecosystem. You almost certainly have it installed already. This recommendation is not about installing it — it's about using it at 20% of its actual capability.

Most people use Dataview for simple lists:

```dataview
LIST FROM #project WHERE status = "active"
```

That's fine. But Dataview's real power is as a **live dashboard engine** for your vault — and given that you now have ORACLE writing structured frontmatter to files daily, the conditions are perfect to upgrade how you use it.

---

## Why Now

ORACLE writes frontmatter to every file it creates:

```yaml
---
type: market-scan
agent: ORACLE
date: 2026-04-22
market_sentiment: cautiously-bullish
tickers_covered: [BTC, ETH, SOL, NVDA]
---
```

This is queryable data. With advanced Dataview, you could build:

- A live **Market Sentiment History** table showing ORACLE's sentiment call for every past scan
- A **Business Ideas Dashboard** with status tracking for every idea surfaced
- An **Agent Activity Log** showing what each agent wrote, when, and on what topic
- A **Queue Tracker** showing all pending vs. answered queue items across all agents

---

## Patterns to Learn

### 1. DataviewJS (the unlock)

Switch from DQL (Dataview Query Language) to DataviewJS for full control:

```javascript
```dataviewjs
let scans = dv.pages('"10 - Agents/ORACLE/Market"')
  .sort(p => p.date, 'desc')
  .slice(0, 10);

dv.table(
  ["Date", "Sentiment", "Tickers"],
  scans.map(p => [p.date, p.market_sentiment, p.tickers_covered?.join(", ")])
);
```

This renders a live table of your last 10 market scans with sentiment and tickers — no maintenance required.

### 2. Inline Dataview Fields

Add `[status:: active]` anywhere in the body of a note — Dataview can query it without frontmatter.

### 3. Grouping and Aggregation

```dataview
TABLE rows.file.link AS "Notes", length(rows) AS "Count"
FROM "10 - Agents/ORACLE/Ideas"
GROUP BY agent
```

### 4. Calendar View (with Dataview + Full Calendar plugin)

Combine Dataview with the Full Calendar plugin to render ORACLE's journal prompts, market scans, and briefings on a visual calendar.

---

## Suggested Dashboard to Build

Create a note at `10 - Agents/ORACLE/_Oracle Dashboard.md` with DataviewJS blocks for:

1. Last 7 briefings (date, sentiment, tasks_completed)
2. All open queue items (status = "pending")
3. All ideas by status
4. Market sentiment trend (simple table, last 14 days)

**Time to build:** 45–60 minutes. **Value:** You get a live operations center for your agent network with zero manual updating.

---

## Resources

- Official docs: [blacksmithgu.github.io/obsidian-dataview](https://blacksmithgu.github.io/obsidian-dataview/)
- Community cookbook: search "Dataview cookbook" on the Obsidian forum — huge library of ready-to-use queries
- DataviewJS examples: Obsidian Hub (publish.obsidian.md/hub) has a full DataviewJS section

---

## Secondary Recommendation: Templater Plugin

If you're not already using **Templater** (vs. the built-in core Templates), upgrade. Templater supports JavaScript execution inside templates, which means you can auto-populate dates, generate UUIDs for ideas, and pull in dynamic content at note creation time. Critical for keeping ORACLE's file naming conventions consistent when creating notes manually.

---

> Part of the [[06 - Reference/Tools|Tools Reference]] | Recommended by [[10 - Agents/ORACLE/Config|ORACLE]]
