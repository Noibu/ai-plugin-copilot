---
name: checkout-analysis
description: "Analyze checkout performance and health using Noibu data. Use when you want to know where shoppers drop off in the checkout funnel, what payment or delivery methods customers are using, why completion rates are low, what priority errors are hurting checkout, or how cart and order values benchmark."
---
<!-- overlay: replace — no dashboard-rendering tool on this host — triage board becomes a markdown report; the Investigate button becomes a conversational follow-up — upstream @ 6165ba92392f -->

# Noibu Checkout Performance & Health Analysis

Surfaces where shoppers drop off in checkout and what to do about it, as a markdown
report built from Noibu session, value, and error data.

**Call metadata:** on every `noibu_*` call this skill drives, pass `skill: "checkout-analysis"`, `skillVersion: "cowork:fbf39d7"`, and a short `skillStep` (see the `querying-noibu-data` skill).

## How it works

Run Setup, then pick one of two behaviors from the user's prompt:

- **Quick answer** (one focused question) → run 1–2 queries, answer directly, offer to go deeper.
- **Full analysis** (broad request, bare invocation, or "yes" to the offer) → the four-step workflow below.

Full analysis flow: **(1)** broad overview (Q1–Q7, plus Q1b when available, current + prior window) → **(2)** cross-reference + period-over-period → **(3)** pick the top 3 regressions → **(4)** run targeted follow-ups → render the report. Signals are flagged on **change vs the prior window**, not absolute level — see `references/queries.md` "Signal model".

## Setup — before any query

**Work quietly.** Don't narrate plumbing — resolving the domain, loading reference files, reading the report format, and deciding which tools, queries, or fields to use all happen silently, with no "let me…" commentary or explanation of *how* the analysis will be assembled. The first thing the user sees is the Step 1 overview line ("Starting with a broad look…"); after that, narrate only real analytical progress (what the data shows), never file reads, tool setup, field-name lookups, or other implementation/processing detail about how the answer was produced.

- **Resolve the domain first; keep the company id it returns.** If the user gave a domain (name or UUID), use it. If not, ask conversationally which one, listing the account's domains — don't interrogate for anything else; take the default window and proceed. If the account has exactly one domain, skip the question and use it. Domain resolution returns a company id alongside the domain UUID; some tools (priority errors, data-connection checks) need that company id too — carry both.
- **Load the Noibu context reference** (the `querying-noibu-data` skill/reference). It maps the role-based names used here ("session query tool", "funnel depth field", etc.) to the real Noibu tools/columns and documents query constraints. If it isn't available, discover tools and field names from the live Noibu API / tool schema instead — don't stop or guess.
- **Confirm every field name by role before using it** (from the context reference, else the live schema). Steps below name fields by role, never by hard-coded column, so when the API changes only the lookup moves.
- **Default analysis window:** the one in the context reference; if none, last 30 days.

### Session tool — what it can group and filter by

- **Group by:** device, country, browser, OS, UTM source/medium, landing URL, bounced, discount-applied, and **funnel depth** (how far the session got).
- **Filter on:** device, country, bounced, checkout-completed, landing URL, UTM source.
- **Funnel depth — confirm the exact field name; the prefix matters.** It *is* groupable (working name ~`CONVERSION_FUNNEL_DEPTH`), but near-variants are rejected. If a group-by is rejected, look up the exact name and retry; only fall back to per-stage count measures if it truly isn't available.
- **Payment and delivery method are NOT groupable.** Attempt those slices only if the reference/schema exposes such a field; if not, skip them (see Principle 1).
- **Checkout sub-step submission counts (contact / address / shipping / payment) and cart value are per-session fields, not group-by dimensions.** They feed Q1b's abandonment-reason classification (row-level, one row per session), not the grouped session queries above. These are standard fields on every account — run Q1b unconditionally; only skip its output (whole table, or individual buckets) if the results actually come back empty (Principle 1 applied to the data, not to whether the fields are named in a reference doc).

## Two principles that govern everything

**Principle 1 — analyze what's present, never flag what's missing.** Which fields are populated varies by platform, integration, and account. An empty/null result is normal — that signal just isn't available for this domain, not broken.

- Silently skip any empty/null slice and build from the data you *do* have. Omit its table entirely — no zero-rows, no "no data" placeholder.
- Never create a finding, action, or caveat about missing data, tracking, capture, or coverage. That's out of scope.
- There's always a useful story in the present data (funnel progression, device/country splits, conversion, errors, populated value/method fields). Lead with it confidently, as if it's the full picture.

**Principle 2 — trust cart → completion when intermediate events are uninstrumented.** If checkout-started and/or payment-submitted are near-zero while completions are healthy, those events aren't instrumented here. Don't treat the apparent "drops" as real — use cart → completion as the headline conversion metric and lean on segment splits (device, country, discount) for the drop story.

**Exception to Principle 1 — the whole domain is empty.** If the *entire* dataset is near-zero (total sessions 0 or a handful), there's no story to tell. Say plainly the domain has essentially no traffic in this window and offer to widen the range or pick another domain. This is a "no data in the window" statement (fine to surface), not a tracking-coverage complaint (still out of scope).

---

## Quick answer

A specific, focused question ("where do people drop off?", "what payment methods are used?", "errors on my checkout pages?"):

1. Load `references/queries.md` and run only the 1–2 query specs needed (e.g. funnel → Q1, why people abandon → Q1b, payment methods → Q3, errors → Q5).
2. Answer in a few sentences + a small table if useful.
3. Ask conversationally whether they'd like the full analysis; if yes, run Full analysis.

Don't run the full workflow or render the report for a quick answer.

## Follow-up questions after a report

A user question naming a specific signal from a report already in this conversation ("why is mobile checkout completion down?", "dig into the payment method drop") — handle as a focused follow-up, not a re-run of the full analysis. The follow-up specs are in `references/queries.md` (Step 4).

- **Don't re-run the breakdown that already surfaced the signal.** The Step 4 follow-up for this signal already ran to rank and headline it — reuse that evidence; only query again if the needed data wasn't pulled.
- **Go one level deeper** for root cause: e.g. error detail on the affected page, or live page inspection — genuinely new work beyond the breakdown. A single follow-up may surface more than one cause, and of mixed type (e.g. a UX/form issue *plus* two priority errors, or a slow page).
- **Route the what-next by cause type — decide this *after* diagnosing:**
  - **UX / funnel / market / discount / ops causes** → write the what-next inline: one concrete, owner-routed step + the check that confirms resolution.
  - **Priority error or performance/CWV causes** → if **tech-diagnosis** is available, hand each one off to it (it owns root-cause-to-fix, platform guidance, and the ticket/share flow). Suggest one handoff per issue (carrying its own `humanId` or page), so multiple causes become multiple options the user can pick from:
    - Fixable error → suggest asking tech-diagnosis to fix #[humanId] on [domain]
    - Vendor/third-party error → suggest asking tech-diagnosis to share #[humanId] with the vendor on [domain]
    - Performance → suggest asking tech-diagnosis to look at [LCP|INP|CLS] on [page-url] for [domain]
    - **If tech-diagnosis is NOT available, don't suggest it.** Handle the cause inline: name the issue (`humanId` / page), give a one-line likely root cause and a concrete next step. Don't mention or recommend installing another skill.
- **Offer to run the next step yourself when it's Noibu-answerable — this applies across every cause type above, not just performance ones.** Before handing a what-next off entirely to a human team, check whether it's something the Noibu tools can actually answer right now: page-visit Core Web Vitals (LCP/INP/CLS) for the affected URL, session replay or session detail for the sessions in this bucket, click data for the page, or a deeper breakdown of this same segment. When it is, end the answer with a one-line offer naming the specific action (e.g. "Want me to pull LCP/INP for the checkout page on mobile?" or "Want me to pull a few session replays from this bucket?") instead of only describing it as a task for someone else's team to go check manually. Don't offer this for what-nexts that are genuinely external (e.g. "have the payment processor check gateway logs," "ask the theme team to review the code") — only for what the Noibu toolset itself can pull. This offer is separate from (and can sit alongside) a tech-diagnosis suggestion — it's for the cases where the answer can be fetched directly.
- **Answer in chat with:** the *why* (the breakdown already gathered, ≤8 rows, plus any deeper root-cause finding), per cause either the inline what-next or the tech-diagnosis suggestion, and — where it applies — the one-line offer to pull further Noibu data directly. Keep it tight; don't re-render the whole report.

## Full analysis

A broad request, a bare invocation, or "yes" to the quick-answer offer. Run the whole workflow, then render the report as markdown:

1. **Broad overview** — fire Q1–Q7, plus Q1b when its fields are available, for the current window plus the prior-window counterparts for the change-detection metrics (`queries.md`, Step 1).
2. **Cross-reference + period-over-period** — compute current rates and their delta vs the prior window (Step 2).
3. **Pick the top 3 regressions** to dig into, using the routing table (Step 3). The only non-regression finding is a qualified active error (Q5).
4. **Follow-up queries** — run only the follow-ups those signals call for (Step 4).
5. **Render the report** — see "Rendering the report" below.

## Rendering the report

**Guiding principle: insights first, data second.** The report must be scannable in 30 seconds. Every table is supporting evidence for a finding already named above.

### Section 1 — Key findings & recommended actions *(always first)*

Write after both phases are complete. Number each pair, ranked by sessions affected (the incremental sessions lost to each regression — see the sessions-affected formulas in `references/queries.md`, Step 4).

> **1. Finding:** [specific step, device, market, or page] + [the move vs the prior window, or the qualified error] + [why it matters].
> **Suggested Action:** [One specific, testable thing to do.]

Rules:
- Every change-based finding must be a regression vs the prior window — never the absolute level or a gap vs another segment or the site average.
- **Markets are relative.** A chronically-low or 0% market that didn't change is not a finding.
- **No discount-value finding** unless the discount rate or discounted conversion regressed.
- The only non-regression finding is a qualified active error (Q5).
- If nothing regressed and no error qualifies, say so plainly instead of padding with weak signals — see "No regressions this run" below.
- Cap at 3. If more signals exist: "N more signals in the data below."

### Section 2 — Overview

- **Context line** — `[total sessions] · [ATC%] added to cart`.
- **Checkout funnel** — a markdown table:

  | Stage | Sessions | % of Cart | Drop to Next Step | Δ vs prior |
  |---|---|---|---|---|
  | Added to cart | … | 100% | … | — |
  | Started checkout | … | …% | …% | …pp |
  | Payment submitted | … | …% | …% | …pp |
  | Completed order | … | …% | …% | …pp |

  Callout the single largest *reliable* drop and its rate. Express checkout caveat if depth-4 > depth-3. If intermediate steps are uninstrumented (Principle 2), show only the instrumented steps rather than a misleading drop.
- **Funnel health** — Checkout completion X% (Δ N pp vs prior) · Mobile Y% / Desktop Z% conversion · Checkout errors N.
- **Order profile** — Median order $X · Discounted N% · Items/order N. Principle 1: drop any unpopulated metric, and drop the whole line if none are populated.
- **Payment methods** (completed orders only) — table: Payment Method | Orders | Median Order Value. Survivorship caveat: abandoned sessions' payment method is not captured.
- **Delivery methods** (completed orders only) — table: Delivery Method | Orders | Median Order Value | Median Shipping Cost. Exclude rows where median order value = $0 (in-store pickup / B2B / retail channels). Flag non-English method names with very high order values as likely local-currency anomalies.

### Section 3 — Checkout page errors

List each qualified error from Q5: humanId, title, error type, sessions affected. If none: "No active priority errors detected on checkout pages."

### Section 4 — Evidence from deeper analysis *(one entry per Step 4 follow-up)*

For each Step 4 query, a tight two-line entry:
- **What the data showed:** specific numbers, one or two sentences max.
- **Supports:** reference the numbered finding from Section 1 (e.g., "→ supports Finding 2").

No raw data dumps. Cap tables at 8 rows.

### No regressions this run

When no regression clears the threshold and no error qualifies: "No checkout regressions vs the prior [window] and no active fixable errors — checkout is holding steady. The Overview above shows the current state." If there's no prior window to compare (sparse history), say that instead: "No prior [window] to compare against, so no change-based signals this run — the Overview shows the current state."

## Exporting the report

Triggered by the user asking to export, download, or save the report as a document.

Export-only — **do not re-run the analysis or any queries.** Reuse the results already in this conversation. Read `references/export-pdf.md` and follow the instructions there.

## Recurring reports

If the user asks to put this on a schedule: this skill doesn't create or manage recurring runs itself. Point them at Cowork's own automation instead — e.g. "You can set that up yourself in Cowork's Customize → Automated tasks, pointing it at this same prompt." Nothing here creates, schedules, or persists a recurring run on their behalf.

## Ignoring an issue

Triggered by the user asking to ignore or close a specific issue by its humanId (e.g. "ignore issue #445").

Call `noibu_update_issue` to set that issue's status to Ignored/Closed (confirm the exact status value from the tool schema), then confirm in one line (e.g. "Ignored #445 — it won't surface on future runs"). If `noibu_update_issue` isn't available, say so briefly rather than failing.

## Data quality notes

- **Payment and delivery data reflects completed orders only.** Payment method
  and delivery method fields are only populated on sessions where checkout was
  completed. Never imply a payment method was rejected because of low volume —
  it may simply not have been selected by completers.
- **Express checkout inflates depth-4 vs depth-3.** Apple Pay, Shop Pay, and
  similar flows bypass the payment information page. Depth-4 exceeding depth-3
  is normal — always explain this rather than treating it as a data error.
- **Discount survivorship bias.** Only sessions that reach checkout can have a
  discount applied. The discount rate from Q2 is among checkout-entering sessions
  only.
- **Use the discount value recorded at checkout completion, not at checkout start.**
  Many shoppers apply discount codes after initiating checkout. The checkout-start
  discount fields significantly undercount discount usage — always use the
  completion-time discount value field.
