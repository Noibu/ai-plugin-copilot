<!-- overlay: replace — no dashboard-rendering tool on this host — response is a markdown report, not an HTML widget — upstream @ dec82a877028 -->

# Inline snapshot (response shape)

This is the exact shape of Store Pulse's response — rendered every run, stateless,
nothing persists. Run the `core-kpis` and `purchase-funnel` queries yourself (plus any
optional blocks picked), following `references/blocks/<id>.md` for each — then render
directly as markdown in the chat response. There is no artifact, no widget, and no
re-fetching later; this is one-shot, computed-then-printed output.

## Response shape

1. **Editorial summary first** — 2–3 sentences of plain prose, no bullets, no headers.
   Lead with the most notable thing (a drop, a surprise, a trend). Mix wins and
   concerns. Use specific numbers, not exhaustive detail.
2. **Then the markdown report** — the `# Store Pulse — [domain] ([period label])`
   header, Core KPIs table, Purchase funnel table, and any optional block sections —
   see SKILL.md's "Report" step for the exact table shapes.
3. **The recurring-cadence line** — see SKILL.md's "Suggest — don't set up — a
   recurring check".
4. **Then the investigate-thread offer** — see "Where to next" below.

These run in order — don't offer the investigate-thread question before the summary
and report exist; the options in step 4 are derived from what the summary names.

## Where to next

After the report and the recurring-cadence line, offer a plain-text follow-up — no
special picker, just a short conversational list:

> "Want to dig into any of these? [1], [2], or [3] — or we can leave it here."

- 3 options drawn from the summary you just wrote — each targeting a specific signal
  (drop, surprise, trend, outlier), same order as mentioned in the summary. Frame as
  something to investigate, e.g. "Why did conversion fall 8pp?"
- If fewer than 3 distinct threads surfaced, fill in with defaults: "Where's the
  biggest revenue opportunity?", "What changed since yesterday?"

Route the reply:
- **A named thread** → treat as a fresh question. Don't answer from the snapshot alone
  — let normal skill routing pick the right Noibu skill. Frame in the user's terms, not
  Store Pulse's block names.
- **Declines / "not now"** → acknowledge briefly, end.
