---
name: tech-diagnosis
description: "Diagnose technical issues and Core Web Vital performance problems using Noibu data, and recommend a fix. Use when you want to know what's broken across the store, why an error is happening, what's causing errors, the root cause of poor page performance, how to fix a technical issue, why an error rate spiked, or where errors are concentrated."
---
<!-- overlay: replace — no dashboard-rendering or picker tool on this host — findings and closing offer are plain conversational text; no code-repo or ecommerce-platform connector is reachable on Microsoft Cowork (an MCP server arrives only inside a deployed plugin package, and no such plugin exists in its catalog), so the connector-setup flow is dropped; repo/platform enrichment and the PR-draft offer are kept but gated on runtime tool presence and never solicited or named when absent — upstream @ 2782c9f1a0d4 -->

# Tech Diagnosis

- **Proactive** — no specific issue. Scans for highest-value errors and CWV problems, surfaces a prioritized list.
- **Reactive** — a specific signal is named. Produces a self-contained finding (Summary → Cause → Fix).

---

**Call metadata:** on every `noibu_*` call this skill drives, pass `skill: "tech-diagnosis"`, `skillVersion: "cowork:fbf39d7"`, and a short `skillStep` (see the `querying-noibu-data` skill).

## Setup (always runs first)

1. **Noibu MCP** — confirm connected. If not, stop.
2. **Domain** — UUID → use it. Name only → resolve. Nothing → list and ask. Capture **platform** (Shopify, Magento, etc.).
3. **Mode + window** — detect mode (see below). Default window: 30 days.
4. **Task list** — note the mode-appropriate tasks below and track progress against them internally as you go.

No connector setup on this host (see Connectors below). `$HOME/.tech-diagnosis-config.json` is optional and holds only `platform_overrides` (see Configuration). Share preferences are asked per session.

## Mode detection

- **Proactive** — no specific issue given. Domain name alone is proactive. Examples: "run tech diagnosis", "what's broken", "scan for issues".
- **Share with vendor** — the operator asks to share a specific issue with its vendor. Skip setup and diagnosis → go to "Share with vendor".
- **Ignore issue** — the operator asks to ignore or close a specific issue by id. Skip setup and diagnosis → go to "Ignore issue".
- **Reactive** — specific issue named alongside domain: error name, metric + page, Noibu issue ID, page URL, or structured handoff from another skill.
- **Ambiguous** — ask: "Are you looking for a scan of what's broken across the store, or do you have a specific issue in mind?"

---

## Rules

- **No client-side data joins.** Each Noibu source on its own terms.
- **Don't overstate impact.** "Present in X sessions", not "blocking X users". No revenue projections or revLost figures.
- **Plain language.** Translate Noibu labels; keep identifiers in code style.
- **Suppress citation sections.**
- **First visible output is the result, never narration.** No preambles, filler, or "let me pull X".
- **Never narrate internals.** No tool names, query parameters, API state values, filter logic, or fallback decisions. Retries happen silently.
- **Pause only at designated points, as plain conversational text — never a modal tool.** No exceptions — connector setup and share-destination prompts are conversational text too. Reactive: once after the closing offer. Proactive: after the findings list.
- **After presenting the findings or closing offer, add exactly one short line** telling the operator how to act (e.g. "Ask me to look into any of these, or say 'ignore #[id]' to dismiss one."). No summary, no recap, nothing else.
- **Page criticality is internal only.** Checkout > Cart > PDP > Collection > Home > Other. Never a labeled field.
- **Name third-party services specifically.** Stack trace says Klaviyo, GTM, Shop Pay → use that name.
- **No console links or replay URLs in output** (chat or share). Operators may not have console access. Inline the data instead. On explicit request only, provide links.

---

# Proactive mode

**First action — before anything else:** send "Scanning [domain] for the top technical errors and performance issues — this may take a moment."

## Tasks
1. Scan errors
2. Scan performance
3. Rank findings and present them
4. Deep-dive *(added when the operator picks a finding)*

## Step P1: Scan for errors

**Before fetching any errors:** read `references/platforms/[platform].md` (fall back to `generic.md`). Extract the merchant code paths, platform paths, and telemetry paths. These are the patterns used to classify frames — do not skip this step.

Two passes in parallel.

**Pass 1 — Fixable errors:**
Use an error search query — **not the pre-ranked priority list** — so you control the candidate pool.

Search the last 7 days of errors, sorted by occurrence count highest first, 30 per page, active states only (new, open, in-progress). **Exclude at query time:** HTTP errors (4XX/5XX), network/connection errors (status 0, ERR_NETWORK), and "Script error" type. Metadata only — no stack traces in this call.

**Always fetch up to 3 pages.** Process each page before fetching the next — use the pagination cursor from the response to request subsequent pages. Stop early only if you already have 2 or more fixable candidates. After 3 pages or when no more pages are available, proceed with whatever you found.

For each candidate on each page, apply filters:
1. **Recency** — at least one occurrence in the last 48 hours. If not → skip.
2. **Volume** — more than 10 occurrences in the 7-day window. If not → skip.

For each survivor, fetch the full error detail individually to get the JavaScript stack trace frames. Classify using the paths extracted from the platform file:
- **No frames** → skip.
- **All frames match platform or telemetry paths** → skip.
- **At least one frame matches a merchant path** → keep as fixable candidate.

Before extracting top 2: **deduplicate** — if two candidates share the same primary file and error message/type, keep only the higher-scoring one and note the other ID as a related issue. Second slot goes to the next distinct error.

**Pass 2 — Vendor errors:**
Use the same search — last 7 days, sorted by occurrence count, 30 per page. Apply 48h recency and >10 volume floor. For each survivor, fetch the full error detail individually and classify frames against the platform file. Extract top 2 vendor candidates by score.

**Classification** — use the platform file already loaded above.
- **Fixable** — at least one frame in merchant-owned code paths.
- **Platform** — frames only in core platform infrastructure (Shopify CDN scripts, monorail, checkout, shopifysvc.com); or any HTTP error (4XX/5XX) with no JS stack trace; or any network/connection failure (status 0, ERR_NETWORK, request aborted) with no JS stack trace. Discarded regardless of endpoint. No exceptions.
- **Telemetry** — frames in known analytics/pixel/beacon scripts (see platform file). Discard silently.
- **Vendor** — third-party frames that aren't telemetry, including `*.shopifyapps.com` and Shopify product endpoints with their own support channel (Shop app, Shop Pay).

Discard platform and telemetry. **Classification is mechanical — no judgment calls.**

**Score:** `occurrences (7-day total) × criticality_weight`
Weights: Checkout=5, Cart=4, PDP=3, Collection=2, Home=1, Other=0.5

## Step P2: Scan for performance

Fetch CWV (LCP, INP, CLS) across all page groups. Filter to pages with at least one metric outside the Good band (see `references/performance.md`).

**Score:** `traffic_weight × severity_score` — traffic_weight normalized (highest=1.0); severity: Poor=3pts, Needs Improvement=1pt per metric.

Take top 2. Capture: page/group, worst metric + p75 value, band, traffic volume.

## Step P3: Rank findings and present them

Rank all findings by score × criticality weight (ties by page criticality).

Present as a plain markdown report, in this order:

**Issues you can fix** — up to 2 fixable errors, each as:
- **[Error type]** — [Active | Re-emerging | Improving]
- [Plain-English title]
- [1–2 sentences: what's wrong, where, who's affected]
- [number] occurrences · last seen [date] · [pages] · [browser/OS]

If none: "No active errors found that you can fix directly."

**Performance** — up to 2 findings, each as:
- **[LCP | INP | CLS]** — [Active | Re-emerging | Improving]
- [Plain-English title]
- [1–2 sentences: what's slow, where, how bad]
- p75 [value] — [Good | Needs improvement | Poor] (thresholds in `references/performance.md`)
- [traffic volume · page group · device]

If none: "No significant performance issues found."

**Vendor issues** (only if any exist) — up to 2, each as:
- **[Vendor name] — [Error type]**
- [Plain-English title]
- [1–2 sentences: what's happening, which vendor, impact]
- [occurrence count · vendor name · pages affected]

If none, omit this section entirely (don't say "no vendor issues" — that's a coverage statement, not a finding).

**Closing line** (exactly one, per the Rules above): "Ask me to fix or share any of these — for example 'fix #[issue-id]', 'share #[issue-id] with the vendor', or 'ignore #[issue-id]'."

If the operator asks to put this scan on a recurring cadence: this skill doesn't create or manage recurring runs itself. Point them at Cowork's own automation instead — e.g. "You can set that up yourself in Cowork's Customize → Automated tasks, pointing it at this same prompt." Nothing here creates, schedules, or persists a recurring run on their behalf.

---

## Fix (triggered)

Triggered by the operator asking to fix or dig into a specific issue (e.g. "fix #445", "get the fix for the LCP issue on checkout").

Add a "Deep-dive" task. Enter Reactive mode from Step 2 (domain, platform, window already resolved).

---

## Share with vendor (triggered)

Triggered by the operator asking to share a specific issue with its vendor (e.g. "share #445 with the vendor").

Fetch error detail. Generate a copyable vendor-facing summary (what, which pages, occurrence count, browser/OS breakdown).

Present the summary as copyable text, then ask in one line: "Want me to send this via email or Slack?" — but only if those connectors are already active in the session. If neither is connected, output the summary only. No preferences are saved.

---

## Ignore issue (triggered)

Triggered by the operator asking to ignore or close a specific issue by id (e.g. "ignore issue #445").

Extract issue ID. Call `noibu_update_issue` to close/ignore. Confirm: "Issue #[id] marked as ignored." No further output.

---

# Reactive mode

## Connectors (never solicited)

Microsoft Cowork admits an MCP server only inside a deployed plugin package, so the operator cannot add one themselves. **Never ask them to connect anything, and never name a connector that isn't there** — no "this would be more accurate if", no explaining why something is missing. Go straight from the signal to the data.

Tools may still be present because an admin deployed a plugin carrying them. Check the session tool list silently and use whatever is there; when a tool is absent, proceed without it and say nothing.

## Tasks
- Understanding the issue
- Measure the impact
- Trace the cause
- Recommend a fix
- Optional handoff *(conditional)*

## Step 1: Identify the signal

Translate the prompt to symptom, scope, and context. If clear, fetch immediately — don't narrate it back. If ambiguous, ask one clarifying question.

| Prompt | Signal |
|---|---|
| "Why is LCP poor on checkout?" | Performance — LCP — checkout |
| "What's causing errors on Safari mobile?" | Errors — Safari mobile |

## Step 2: Fetch focused data

Targeted queries only — no broad surveys. Run in parallel where possible.
- **Errors** — priority issues matching scope; full detail for top 1–2. If this issue had related issues noted during proactive deduplication, fetch those too. Surface in the fix: "This fix should also resolve issue #[id] — same root cause, different trigger path."
- **Performance** — CWV at p75 for affected URLs/page group, by device and browser.
- **Combined** — both, scoped to the relevant page/segment.

## Step 3: Render Summary

For each finding:
1. Finding title as a `##` heading
2. 2–3 sentence narrative: impact only — who's affected, how many, what they experience, trend. No error codes or endpoint names.
3. Impacted-sessions figures: session count + % of total traffic, plain-English week-over-week change (calibrated to volume — plain English for small numbers, percentages for high volume, never "WoW" acronym).
4. Top pages affected: 3–5 row table (URL + sessions)

Post-processing: see `references/errors.md` (errors) and `references/performance.md` (performance). Rank by severity → page criticality → sessions. Cap at 3–5 findings.

## Step 4: Trace the cause

- **Noibu AI diagnosis** — use Why/Impact as the basis for the cause narrative.
- **Code-repo enrichment** — only if repo tools (e.g. GitHub) are in the session: search for the affected element, fetch ~20 lines of context. Absent → skip silently, do not mention it.
- **Platform enrichment** — only if store tools (e.g. Shopify) are in the session: query store data to confirm or refute the hypothesis. Absent → skip silently, do not mention it.
- **Live page inspection** — this host has no browser-automation connector, so always fall back to `web_fetch` `https://{domain}{path}` (construct from resolved domain + Noibu paths, no need to ask) → pattern-based only, if `web_fetch` is unavailable or a fetch call fails. After fetching, extract only `<script>` `src` and `crossorigin` — discard full HTML immediately. For INP without page inspection, fetch clicked selectors from Noibu.

If any enrichment source errors, skip it immediately and continue — treat as unavailable, no retries.

See `references/errors.md` and `references/performance.md` for guidance.

**Render cause:** open with "From here on, I'm reasoning over the data rather than reporting it…". Commit to the primary cause; note alternatives in one sentence.

## Step 5: Closing offer

Determine the fix internally but **do not render it yet**. Commit to the most likely fix; note alternatives in one sentence. Present the options as plain conversational text:

> What would you like to do next?
> - **[Automatically draft a pull request (PR)](low|medium|high risk · Accept PR to apply fix)** — offer this **only** when repo tools are in the session **and** a specific file and line was identified. If either is untrue, omit the line entirely: no disabled variant, no explanation of why it isn't there.
> - **Show me the fix**
> - **Open a ticket** — offer this only when a ticket tool (Jira, Linear, GitHub Issues, Notion, …) is in the session. None present → omit the line, say nothing about it.
> - **Share findings**

Nothing renders after this prompt except the operator's reply.

**When "Show me the fix" is chosen** — render fix only (no repeating Summary or Cause), then show the closing offer again.

### Output structure

```
## [Finding title]

### Summary
[2–3 sentences — impact only. No error codes or endpoint names.]
[Impacted-sessions figures: session count + % of total + trend]
**Top pages affected:** 3–5 row table

### What's likely causing it
*From here on, I'm reasoning over the data rather than reporting it...*
[1–2 sentences — hypothesis only. Start from why.]

[Closing offer]
"Ask me to [draft a PR with the fix, ]show the fix, open a ticket, or share findings — or choose another issue to dig into." (include the bracketed clause only if the PR option was offered)

--- after "Show me the fix" ---

### How to fix it
[Specific change only. Code or config, testing note, alternative pointer.]

### Technical details  [on request or via the share flow]
```

## Step 6: Handoff *(conditional)*

See `references/handoff.md` for the full ticket and report flow.

---

## Configuration

`$HOME/.tech-diagnosis-config.json` is optional and holds one key:

```json
{
  "platform_overrides": {
    "merchant_paths": ["..."],
    "vendor_paths": ["..."]
  }
}
```

Read during stack-frame classification (`references/errors.md`); absent means fall back to the built-in platform reference file. No connector state is stored or remembered, because there is nothing to connect. Ticket and share preferences are asked per session.
