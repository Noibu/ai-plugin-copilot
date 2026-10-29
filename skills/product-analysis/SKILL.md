---
name: product-analysis
description: "Analyze product and collection performance using Noibu data. Use when you want to know which products are underperforming, what your best-selling product type is, why a product isn't converting, how your collections are performing, which products get views but no sales, or where shoppers drop off in the product funnel."
---
<!-- overlay: replace — no dashboard-rendering tool on this host — triage board becomes a markdown report; the Investigate button becomes a conversational follow-up — upstream @ 6540109fe1f2 -->

# Noibu Product & Collection Performance Analysis

Surfaces which products and collections are winning or losing, and why — as a
markdown report built from Noibu session and page data.

**Call metadata:** on every `noibu_*` call this skill drives, pass `skill: "product-analysis"`, `skillVersion: "cowork:fbf39d7"`, and a short `skillStep` (see the `querying-noibu-data` skill).

## How it works

- **Quick answer** (one focused question) → run 1–2 queries, answer directly, offer to go deeper.
- **Full analysis** (broad request, bare invocation, or "yes" to the offer) → the workflow below.

## Setup — before any query

**Work quietly.** Resolving the domain, loading reference files, reading field names, and running queries all happen silently — no "let me…" commentary. The report is the first substantive output.

- **Resolve the domain first.** Use what the user gave (name or UUID). If nothing, ask conversationally which one, listing their domains — don't ask about anything else. If exactly one domain, skip the question.
- **Use the `querying-noibu-data` reference already loaded in context** — do not read it again. It maps role-based names to real tools/columns and documents query constraints.
- **Confirm every field name by role** before using it. Steps name fields by role, never hard-coded column names.
- **Default window:** from the context reference; if none, last 30 days.
- **If the entire dataset is near-zero** (total sessions 0 or a handful), say plainly the domain has no traffic in this window and offer to widen the range or pick another domain.

---

## Quick answer

For focused questions ("which products get the most views?", "how is the Sale collection performing?"):

1. Run only the 1–2 queries needed.
2. Answer directly — a few sentences and a small table if useful.
3. Ask conversationally whether they'd like the full analysis; if yes, proceed below.

Don't run the full workflow or render the report for a quick answer.

---

## Full analysis

A broad request, a bare invocation, or "yes" to the quick-answer offer.

**Loading reference files:** Use the `Read` tool. All files live in a `references/` subdirectory next to this SKILL.md — derive the base path from wherever this file was loaded from.

1. Read `references/queries.md` now, before running any queries.
2. Run the four-step workflow from queries.md.
3. Render the report as markdown — see "Rendering the report" below.

---

## Follow-up questions after a report

A user question naming a specific product or collection signal from a report already in this conversation ("why isn't the Devon Knit Polo converting?", "dig into the Sale collection drop") — handle as a focused follow-up, not a re-run of the full analysis.

- **Don't re-run the breakdown that already surfaced the signal** — reuse the evidence already gathered; only query again if genuinely new depth is needed.
- **Go one level deeper** for root cause. A single follow-up may surface more than one cause.
- **Route the what-next by cause type — decide this *after* diagnosing:**
  - **UX / merchandising / funnel / ops / localization** → write the action inline: one concrete, owner-routed step + the check that confirms resolution.
  - **Priority error or performance/CWV** → invoke the `tech-diagnosis` skill directly (read it via the `Read` tool using its path from the available skills list) and continue inline. If `tech-diagnosis` isn't available, handle the cause inline instead — don't mention or recommend installing another skill.
- **Answer in plain chat text** — root cause in 1–2 sentences, then a small markdown table of supporting evidence (≤8 rows), then the recommended action or tech-diagnosis output. Keep it tight; don't re-render the whole report.

---

## Rendering the report

**Guiding principle: insights first, data second.** The report must be scannable in 30 seconds. Every table that follows is supporting evidence, not the headline.

### Section 1 — Key findings & recommended actions *(always first)*

Write this section **after the overview and all deeper follow-ups are complete** — it should synthesize everything, including deeper follow-up discoveries. Number each pair so Section 3 can reference them without repeating them.

Lead with up to 3 finding + action pairs. This is the most important part of the report.

Format each as:

> **1. Finding:** [Product/collection name] + [one concrete number] + [why it matters in one clause].
> **Suggested Action:** [One specific, testable thing to do about it.]

Rules:
- Order by impact, not by how obvious the finding is.
- Every finding must name a specific product, collection, or type — no generic observations.
- Every action must be concrete enough to hand off.
- Cap at 3. If more signals exist, add one short line: "N more signals in the data below."

### Section 2 — Overview

- **Context line** — `[total sessions] · [site CVR%] conversion rate · Median order $[X]`.
- **Product funnel** — a markdown table, product-scoped (sessions that viewed any product), not site-wide sessions:

  | Stage | Sessions | % of Viewers | Drop to Next Step |
  |---|---|---|---|
  | Viewed Product | … | 100% | … |
  | Added to Cart | … | …% | …% |
  | Completed Order | … | …% | …% |

  Callout the largest drop.
- **Top products** (top 5 by sessions) — table: Product | Sessions | Add-to-cart rate | Conversion rate. Bold any product referenced in Section 1.
- **Top collections** (top 5 by sessions) — table: Collection | Sessions | Conversion rate | Revenue/session. Bold any collection referenced in Section 1.
- **Top product types** (top 5 by sessions) — table: Type | Sessions | Conversion rate | Revenue/session. Omit this table entirely if all types are within 20% of each other — no signal worth showing.

### Section 3 — Evidence from deeper analysis *(one entry per deeper follow-up)*

This section exists to show *why* the actions in Section 1 are warranted — not to restate them. Each entry is the evidence trail for a finding already summarized above. Do not repeat the finding or the action here.

For each deeper follow-up query, write a tight two-line entry:

- **What the data showed:** specific numbers from the query — one or two sentences max.
- **Supports:** reference the numbered action from Section 1 this evidence backs up (e.g., "→ supports Action 2").

If a deeper follow-up query revealed something not yet captured in Section 1, surface it there first (add or update a finding + action pair), then reference it here.

No raw data dumps. If a table helps, cap it at 8 rows.

### No signals this run

If no products, collections, or types stand out above the traffic thresholds for this store's size, say so plainly instead of padding with weak signals: "CVR and ATC rates are consistent across top products and collections, with no outliers above the traffic thresholds for this store size. The Overview above shows the current state."

---

## Exporting the report

Triggered by the user asking to export, download, or save the report as a document.

Export-only — do not call any Noibu tools and do not re-query. Reuse the data already in this conversation. Read `references/export-pdf.md` and follow the instructions there.

## Recurring reports

If the user asks to put this on a schedule: this skill doesn't create or manage recurring runs itself. Point them at Cowork's own automation instead — e.g. "You can set that up yourself in Cowork's Customize → Automated tasks, pointing it at this same prompt." Nothing here creates, schedules, or persists a recurring run on their behalf.

---

## Data quality notes

- **CVR is only comparable within the same price tier.** High-ticket products and collections will always convert at lower rates than low-ticket items — this is expected, not a signal. Never flag low CVR as underperformance without a same-tier benchmark. Use product and collection names to infer price tier; when no same-tier benchmark exists, rely on `ATC_LIFT_OPPORTUNITY` and funnel depth instead. This limitation applies at every level — product, collection, and product type. Price data is not available in Noibu session data.
- **Revenue per product unavailable.** Order value = total cart for the session — all products in that session get credited the full value. Never label it "product revenue." Use purchase session count as the primary volume metric; median cart value as secondary.
- **Product title variants.** Same base product may appear as multiple rows (e.g. "WOMENS CLASSIC TEE - WT0200005" vs "WOMEN'S CLASSIC TEE - WT0200005"). Flag near-duplicates; don't silently merge.
- **URL fragmentation.** Always use URL CONTAINS with a SKU code or slug fragment in page visit queries.
- **Null funnel depth = viewed only.** Always label as "Viewed only", not blank or missing.
