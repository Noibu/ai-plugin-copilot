---
name: store-pulse
description: An instant, at-a-glance read on your store's health — run on demand. Use to check on how the overall store is doing.
---
<!-- overlay: replace — no saved-dashboard or scheduling surface on this host — stateless per-run markdown snapshot only — upstream @ eb6518bc30fd -->

# Store Pulse

A stateless health check. Every run resolves the domain, pulls fresh Core KPIs + Purchase Funnel data (plus any optional blocks requested), and prints a markdown report directly in the chat response. Nothing persists between runs — no saved state, no config file. Re-run any time for a fresh read.

**Call metadata:** on every `noibu_*` call this skill drives, pass `skill: "store-pulse"`, `skillVersion: "cowork:fbf39d7"`, and a short `skillStep` (see the `querying-noibu-data` skill).

## Flow

### 1. Resolve the domain

- Always call `noibu_list_domains` first (unless the user already gave you a UUID — use it directly).
- **User named a domain** → case-insensitive match (accept `www.` variants). If matched, use that UUID + name. If not, tell them the named domain isn't in their Noibu account, list what is, and stop. Never silently substitute a different domain.
- **User didn't name a domain** → one result, use it. Multiple results → ask in plain text, e.g. "Which store should I check — [name 1], [name 2], or [name 3]?" and wait for their reply. Don't guess.

### 2. Pick the window

Default to the **last 24h vs. the prior 24h**. If the user asks for a different window ("this week", "last 30 days"), use that instead — same query shape, different `dateTimeRange`. State the window used in the report header.

### 3. Choose blocks

`core-kpis` and `purchase-funnel` are always included. Ask once, in plain conversational text (no special widget):

> "I'll pull Core KPIs and the Purchase Funnel by default. Want me to also include Top products, Channel performance, or Paid ad performance?"

Read `references/blocks/<id>.md` for each block picked (default and optional) before querying — those files define the exact metrics, MCP query shapes, methodology tooltips, empty-state handling, and polarity. Don't rewrite or second-guess them; reuse the query shapes and table columns verbatim.

### 4. Fetch the data

Run the MCP queries from each included block's reference file for both the current and prior windows. Compute deltas vs. the prior period of equal length.

### 5. Report

Print a clean markdown report directly in the chat response — headers, tables, bullet points, nothing else:

```markdown
# Store Pulse — [domain] ([period label])

[1-2 sentence editorial summary — lead with the most notable thing, mix wins and concerns, use specific numbers]

## Core KPIs

| Metric | Value | vs prior |
|---|---|---|
| Sessions | ... | ... |
| Engagement | ... | ... |
| Conversion | ... | ... |
| AOV | ... | ... |
| RPS | ... | ... |

## Purchase funnel

| Step | % of sessions | Sessions | Δ pp |
|---|---|---|---|
| Enter store | 100% | ... | ... |
| View product | ... | ... | ... |
| Add to cart | ... | ... | ... |
| Start checkout | ... | ... | ... |
| Checkout complete | ... | ... | ... |

[+ one section per optional block picked, using that block's table shape from its reference file — Top products, Channel performance, Paid ad performance]
```

Use each block's own empty-state and polarity rules (e.g., "No traffic in this window") rather than blank cells or guessed numbers.

See `references/inline-snapshot.md` for the full response-shape contract — the exact order of the editorial summary, this report, the recurring-cadence line, and the investigate-thread follow-up below.

### 6. Suggest — don't set up — a recurring check

One line, near the end, after the report:

> "Want this on a recurring cadence? You can set that up yourself in Cowork's Customize → Automated tasks, with a prompt like: 'Run my store health check for [domain] and summarize any notable changes.'"

This is something the user sets up themselves through Cowork's own automated-tasks feature — nothing here creates, schedules, or persists a recurring run on their behalf.

### 7. Offer a next thread to pull on

After the recurring-cadence line, offer a short plain-text follow-up drawn from the summary — see `references/inline-snapshot.md`'s "Where to next" for the exact options and routing.

## File map

```
store-pulse/
├── SKILL.md
├── references/blocks/{core-kpis,purchase-funnel,top-products,channel-performance,paid-performance}.md
└── references/inline-snapshot.md
```

Read `references/blocks/<id>.md` for block specs (metrics, MCP query shapes, methodology tooltips, empty states, polarity). Read `references/inline-snapshot.md` for the full response-shape contract.
