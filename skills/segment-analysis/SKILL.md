---
name: segment-analysis
description: "Analyze segment and traffic performance using Noibu data. Use when you want to know which channels, devices, or countries convert best, how mobile compares to desktop, where your traffic is coming from, which segments are underperforming, or where to find your best and worst converting customer segments."
---
<!-- overlay: replace — no dashboard-rendering tool on this host — triage board becomes a markdown report; offers become conversational — upstream @ df2f03fcd2c7 -->

# Noibu Segment Conversion Analysis

Surfaces which segments are winning or losing, and why — as a markdown report
built from Noibu session data.

**Call metadata:** on every `noibu_*` call this skill drives, pass `skill: "segment-analysis"`, `skillVersion: "cowork:fbf39d7"`, and a short `skillStep` (see the `querying-noibu-data` skill).

## How it works

- **Quick answer** (focused question) → run 1–2 queries, answer directly, offer to go deeper.
- **Full analysis** (broad request, bare invocation, or "yes" to the offer) → the workflow below.

## Setup — before any query

**Work quietly.** The first thing the user sees is the report — no "let me…" commentary before it.

- **Resolve the domain first.** Use what the user gave (name or UUID). If nothing, ask conversationally which one, listing their domains — don't ask about anything else. If exactly one domain, skip the question.
- **Confirm every field name by role** before using it. Reference files name fields by role, never hard-coded column names.
- **Default window:** last 30 days unless the user specifies otherwise.
- **If the dataset is near-zero** (total sessions 0 or a handful), say plainly that the domain has no traffic in this window and offer to widen the range.

---

## Quick answer

For focused questions ("which channel converts best?", "how does mobile compare to desktop?", "which countries are underperforming?"):

1. Run only the 1–2 queries needed and answer directly.
2. Ask conversationally whether they'd like the full analysis: "Want me to run a full segment analysis — survey device, country, and channel, then dig into anomalies? Reply yes or no."

Don't run the full workflow or render the report for a quick answer.

---

## Full analysis

**Loading reference files:** Use the `Read` tool. All files live in a `references/` subdirectory next to this SKILL.md — derive the base path from wherever this file was loaded from.

1. Read `references/queries.md` now, before running any queries.
2. Run the workflow from queries.md.
3. Render the report as markdown — see "Rendering the report" below.

---

## Follow-up questions after a report

A user question naming a specific segment signal from a report already in this conversation ("why is Canada converting so low?", "dig into the campaign drag") — handle as a focused follow-up, not a re-run of the full analysis.

- Reuse evidence already gathered; only re-query if genuinely new depth is needed.
- Diagnose root cause first, then route the action:
  - UX / campaign / localization / tracking gap → one concrete owner-routed action + the check that confirms resolution.
  - Priority error or CWV → invoke the `tech-diagnosis` skill inline if it's available; otherwise handle the cause inline — don't mention or recommend installing another skill.
- **Respond in plain chat text.** Format: root cause in 1–2 sentences · evidence table (≤8 rows) · recommended action.

---

## Rendering the report

**Guiding principle: insights first, data second.** The report must be scannable in 30 seconds. Every table that follows is supporting evidence, not the headline.

**Structure:**

### Section 1 — Key findings & recommended actions

Write this section after both phases are complete — it should synthesize everything, including deeper analysis discoveries. Lead with up to 3 numbered finding + action pairs, ordered by impact. Every finding must name a specific segment and a concrete number. Every action must be specific enough to hand off. No tables in this section.

### Section 2 — Segment overview

One table per dimension, each with a specific insight callout:

- **Devices** — table: Device | Sessions | Conversion rate | Revenue/session.
- **Countries** (top segments) — table: Country | Sessions | Conversion rate | Revenue/session.
- **Channels** (UTM source/medium) — table: Channel | Sessions | Conversion rate | Revenue/session.

### Section 3 — Evidence from deeper analysis

This section exists to show why the actions in Section 1 are warranted — not to restate them. Each entry is the evidence trail for a finding already summarized above. Do not repeat the finding or the action here. Cap tables at 8 rows.

### No signals this run

If nothing stands out above the site-wide benchmark and the bot-filter gate (see `references/queries.md`, Step 1.5), say so plainly instead of padding with weak signals.

---

## Exporting the report

Triggered by the user asking to export, download, or save the report as a document.

Export-only — do not call any Noibu tools and do not re-query. Reuse the data already in this conversation. Read `references/export-pdf.md` and follow the instructions there.

## Recurring reports

If the user asks to put this on a schedule: this skill doesn't create or manage recurring runs itself. Point them at Cowork's own automation instead — e.g. "You can set that up yourself in Cowork's Customize → Automated tasks, pointing it at this same prompt." Nothing here creates, schedules, or persists a recurring run on their behalf.

---

## Data quality notes

- **Currency anomalies**: Very high revenue per session for a single country
  usually means local currency, not USD. Flag it rather than presenting it
  at face value.
- **Untagged traffic**: A large share of sessions with no channel attribution
  is a tracking gap worth noting explicitly.
- **Awareness campaigns at zero conversion**: Expected — but still worth
  calling out so the user can confirm the campaign objective is non-conversion.
- **Express checkout on mobile**: More completed purchases than payment page
  views is normal — Apple Pay and Shop Pay bypass the payment step. Explain
  it rather than treating it as a data error.

The report as rendered in the conversation is the deliverable — conclude here.
