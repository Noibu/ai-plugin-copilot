---
name: ab-test-opportunity-finder
description: "Find evidence-backed A/B test opportunities for a Noibu-monitored ecommerce domain, aimed at the goal the user picks (add to cart rate, checkout starts, checkout completes, product or collection page views, a specific page, or all goals). Checks the domain has the traffic and instrumentation the goal needs, then ranks tests from 30 days of Noibu analytics. Works from Noibu data only: it does not open the store in a browser, so claims about page layout are marked as inferred and there are no visual previews. Delivers one Markdown report of ranked tests with the data behind each, weekly traffic, the setup as Noibu will receive it, and dev notes, plus a candidates.json handoff, then offers to create chosen tests as Noibu drafts. Writes no variation code. Use for A/B test ideas, CRO or experiment ideas, \"what should we test\", or planning a test on a Noibu domain. Requires the Noibu MCP connector."
---
<!-- overlay: replace — no browser or artifact host — data-only run with a Markdown report; goal question and draft creation go conversational — upstream @ b8df184a970d -->

# A/B Test Opportunity Finder (Noibu)

Produce a short list of A/B test recommendations for an ecommerce store,
aimed at the goal the merchant cares about, that the merchant can read and
turn into Noibu drafts. The bar for inclusion is high: every
recommendation must be supported by measured Noibu data. Fewer,
better-supported tests beat a long speculative list.

The deliverable is one Markdown report file, `<domain-short>-test-recommendations.md`,
written per `references/output-format.md`, plus `candidates.json` next to
it (the schema is `references/handoffs/candidates.md`). Each test in the
report is one section: what to change, why the data supports it, the setup
exactly as Noibu will receive it, how much traffic its success metric
sees, and dev notes. Drafts are created in the conversation, only for the
tests the user names, after the report is delivered
(`references/create-drafts.md`).

The run ends at the report and the offer. Writing the variation code and
QA belong to the developer, or to a skill that consumes `candidates.json`;
this skill never opens a codebase and never asks for one.

**No live-site read on this host.** There is no browser here, so the run
cannot confirm what a page shows. Every claim about page structure or copy
(a feature exists or is missing, where a block sits, what the shipping
line says) is marked "inferred from data" unless the business context
states it. Each test's dev notes carry one "Check on the site first" line
naming what the developer must confirm before building. This replaces the
validation step that kills candidates on other hosts, so expect one or two
tests to fall away once someone looks at the page; the report says so in
its footer.

What the run changes: nothing on the store and nothing in Noibu, until the
user asks for a draft. Then it creates exactly the drafts they named. It
never starts, updates or deletes a test.

**Call metadata:** on every `noibu_*` call this skill drives, pass `skill: "ab-test-opportunity-finder"`, `skillVersion: "cowork:fbf39d7"`, and a short `skillStep` (see the `querying-noibu-data` skill).

## Speed and quiet-mode rules (these govern the whole run)

The run is measured in round trips, not in tool calls. Every call that
does not depend on an earlier result goes in the same message as its
siblings. Target shape: Block 1 (goal question) → Block 2 (resolve +
readiness) → Block 3 (Batch 1) → Block 4 (Batch 2) → write the report and
`candidates.json` → closing message. Anything that adds a serial step
needs a reason.

1. **Batch the startup.** Block 1, in ONE message: use the
   `querying-noibu-data` skill already loaded in context (load it if it
   is not), read `references/noibu-query-battery.md`, and ask the goal
   question (below) unless the prompt already answers it.
2. **Two query batches, maximum.** Run the battery as two parallel
   batches. The readiness reads are not a batch: they decide whether the
   battery runs at all. One extra follow-up query is allowed only when a
   batch result is uninterpretable without it. Do not explore. The
   battery mentions a site read and its batches; skip those parts here.
3. **No rendering tools.** Do NOT call `noibu_visualize_page_visits` or
   `noibu_show_session_replay`. Get scroll behavior from
   QUANTILES(MAX_SCROLL_DEPTH_RATIO) and click behavior from CLICKED_TEXT
   queries instead. Exception: the user explicitly asks for a heatmap or
   replay.
4. **No narration.** Do not summarize between tool calls, do not paste
   query results or data tables into the chat, and do not describe what
   you are about to do. Write one sentence in chat only if you hit a
   blocker, or when readiness stops the run.
5. **Report-only output.** The recommendations live in the report file,
   not in the chat. The closing chat message has one fixed shape, at most
   6 short sentences: (1) what was delivered (the report and
   `candidates.json`, and the reminder that variation code must be live
   before a test is started), (2) the top-ranked test in one clause, (3)
   that layout claims are inferred because this host has no browser, and
   the report lists what to check on the site, (4) at most one caveat
   sentence when one applies (the goal defaulted to "All goals"; the
   chosen metric is not instrumented here; traffic is thin; the store
   serves several markets and the report is written for the top one; no
   business context is saved, and `build-business-context` would sharpen
   the hypotheses), (5) the offer: "Tell me which tests to create as
   drafts in Noibu (for example, 'create tests 1 and 3')." Every other
   caveat lives in the report.
6. **Read each reference once.** `references/noibu-query-battery.md` in
   Block 1; `references/output-format.md` and `references/create-drafts.md`
   in the Block 4 message. `references/handoffs/candidates.md` when
   writing `candidates.json`. Never `references/shared/*` (hosted here for
   other skills) or `references/site-validation.md` (the browser read,
   which this host does not run).

## Prerequisites

- Noibu MCP connector connected. The `querying-noibu-data` skill must be
  loaded before the first `noibu_*` call: it carries the orderBy
  requirement, row caps, measure-uniqueness rules, the domain-resolution
  flow, and the rule to check sibling domains before reporting a zero.
  This skill cites those rules and does not restate them.
- A domain name or UUID from the user. If neither is given, resolve with
  `noibu_list_domains` and use the result; do not stall on asking.

Who runs it: anyone with read access to the domain in Noibu. Creating a
draft needs write access; when the create call is refused, say so in one
sentence and leave the report as the deliverable.

## The goal question (Block 1)

Ask what the user is trying to move before pulling any data, because the
goal decides which gaps count as candidates and which success metric every
test uses. The goals are the primary metrics Noibu's A/B testing product
can measure (add to cart rate, checkout conversion rate, and
page-view-to-URL rate against a URL or a page group; nothing else exists
as a success metric).

| Goal as offered | Success metric written in the report |
| --- | --- |
| Increase add to cart rate | Add to cart rate |
| Increase checkout starts | Viewed page: page group Checkout (or the checkout URL) |
| Increase checkout completes | Checkout conversion rate |
| Increase views of product pages | Viewed page: page group Product |
| Increase views of collection pages | Viewed page: page group Collection |
| Increase views of a specific page | Viewed page: <URL> |
| Not sure / show me everything | "All goals": rank across all metrics |

Ask it as one short conversational question with the four choices
written out: increase add to cart rate, increase checkout starts,
increase checkout completes, or increase page views (product pages,
collection pages, or one specific page). Say that "not sure" is fine and
gets tests across every goal. Only a page-views answer needs a second
question ("Which pages? Product pages, collection pages, or a specific
page (paste the URL)"), asked in the next message.

Skip the question when the prompt already states the goal ("find tests to
get more people into checkout" is checkout starts; "our add to cart rate
is bad" is add to cart rate; "what should we test" with no metric is a
reason to ask). In an unattended or scheduled run, default to "All
goals", say so in the closing caveat, and carry on.

The goal shapes which battery results are decisive (the "Goal lens" table
in the battery) and the inclusion rule in synthesis. A checkout-completes
goal on a domain whose completion counters are zero (Batch 1 check #4)
cannot be measured: ship the tests against Viewed page: page group
Checkout instead and use the caveat sentence.

## Workflow

Default analysis window: the last 30 full days (widen to 90 days under
about 50,000 sessions in 30 days, or when the user asks).

### Block 2: resolve and readiness

Resolve the domain (`noibu_get_domain`, or `noibu_list_domains`). Then, in
the next message, the four readiness reads, all parallel (the exact calls
are the "Readiness" section of the battery): `noibu_check_data_connection`;
one `noibu_search_sessions` over the window grouped by
CONVERSION_FUNNEL_DEPTH; `noibu_list_domains` filtered to the domain's
`companyId` (the sibling domains); `noibu_get_business_context`. Decide
before anything else runs:

- **Not receiving data, or fewer than about 1,000 sessions in the
  window:** stop. One sentence in chat: the domain has no traffic to
  analyze, and, when a sibling domain carries sessions (a `www.` twin, a
  `checkout.` host), name it and offer to run against it. No report.
- **Every session at funnel depth null (no add-to-cart or checkout
  events):** add to cart rate and checkout conversion rate cannot be
  measured. For a page-views goal or "All goals", continue with
  page-view metrics only. For an add-to-cart or checkout goal, ask once
  whether to switch to page-view goals or stop; if nobody answers,
  continue with page-view goals. Either way the report's footer says the
  cart and checkout events are not instrumented and that fixing that
  comes before any cart or checkout test.
- **Thin traffic (under about 15,000 sessions in 30 days):** continue,
  widen to 90 days, and use the thin-traffic caveat. The product's
  days-to-verdict estimate at setup is the final word; do not pre-empt it
  with a number.
- **Siblings with traffic:** when a sibling carries more sessions than the
  domain asked for, or is a `checkout.` host, say so in the report's data
  line ("checkout is tracked on checkout.example.com").
- **Business context present:** read its brand-voice section before
  writing any `what`, `why`, hypothesis or variation name, and treat the
  site facts it states (shipping thresholds, promotions, audiences) as the
  only non-inferred source of what the site shows. When it is missing,
  the closing caveat may point to `build-business-context`.

### Blocks 3 and 4: the battery

Block 3 is every Batch 1 query from the battery, in one message. Block 4
is Batch 2 (filters come from Batch 1), one `noibu_list_ab_tests` call for
the domain (running and draft tests, for the overlap line and to avoid
proposing a test that already exists), and the reads of
`references/output-format.md` and `references/create-drafts.md`. Copy each
query's `queryInput` from the battery rather than composing the JSON.
There is never a third batch.

As results land, keep a private list of candidates, each tagged with the
numbers that support it and the success metric it acts on. What makes a
candidate worth carrying forward: a measured gap (segment A vs segment B,
page vs template peers, cohort vs baseline) that is large, sits on
meaningful volume, has a plausible mechanism a variant could act on, and
acts on the goal's metric. Generic best practices without a measured gap
do not qualify.

**Multi-market stores.** When the top URLs carry a locale prefix
(`/en-gb/`, `/fr-fr/`) or two or more countries each hold over about 10%
of sessions with severalfold different conversion, treat the top market
by sessions as the one the report is written for and say so in the data
line. Exclude blank-country rows that hold over about 10% of sessions and
convert near zero from every baseline, with one footer line. The create
call cannot set a market: write it in targeting as a console follow-up,
per `references/create-drafts.md`.

### Synthesis

Apply the inclusion rules, in this order:

1. Drop a candidate whose variant already exists, when the business
   context or the data shows it (a click-text query that names the
   control, a page group that already exists). Otherwise keep it and put
   the existence check in its "Check on the site first" line.
2. Reclassify anything that is simply wrong (verified errors, broken
   pages, poor web vitals) as a defect, not a test. Fold a defect in only
   as a one-line "Fix first" note on the test whose surface it sits in.
3. Hold to the goal. Every test uses the goal's success metric. A
   survivor that most directly moves a different metric goes under
   "Outside your goal" as one line (at most two), or is dropped. For "All
   goals", rank across metrics.
4. Check the instrumentation under each candidate. When the payment step
   is thin (Batch 1 #3 depth 3, or the #4 PAYMENT_INFO_SUBMITTED counter,
   under about a fifth of completed checkouts), drop any candidate whose
   evidence is a drop-off between checkout start and completion, and add
   one footer line saying the payment step is not fully tracked.
5. Rank by evidence strength × traffic exposure × plausible effect, not
   novelty. Aim for 3 to 6 tests; if only 2 survive, ship 2 and say why.
6. Label correlational evidence as such (self-selected cohorts like
   search users or email traffic justify a test, never assuming the gap
   transfers).
7. Express every survivor in the create flow's own fields (the setup
   block in `references/output-format.md`). A candidate with no supported
   success metric even by proxy, or an audience targeting cannot select
   (anything beyond UTM fields, device type, and country), is widened,
   reframed, or dropped here.
8. Write each test's traffic line: how many sessions a week, in the test's
   own targeting, reach the event its success metric counts, from Batch 1
   #3 and #8 divided by the weeks in the window, rounded to two
   significant figures, plus "Noibu shows the days to a verdict at setup".
   Never write a day count, a sample size, or an expected lift (the
   battery's "Metric volume" section has the arithmetic).
9. Write each test's overlap line when a running or draft Noibu test sits
   on the same surface; omit it otherwise.
10. Never fabricate numbers. Every figure comes from a query or document
    from this run; an unmeasured baseline is "baseline shown at setup".
    Never state a site fact from memory of how stores usually work.

### Deliver

Write the report per `references/output-format.md` and save it as
`<domain-short>-test-recommendations.md`. Write `candidates.json` next to
it in the schema of `references/handoffs/candidates.md`. On this host you
write it directly: there is no build script and no site read, so leave out
`captureLine`, `storefront`, `siteFacts`, `selectors` and `preview`, set
`cartState` to "never added", and copy each test's `createInput`
byte-for-byte from the report's setup mapping. Hand both files to the user,
then send the closing message from quiet-mode rule 5.

### Drafts, on request only

When the user names tests to create, follow `references/create-drafts.md`:
check the domain's existing tests, create each named test once as a draft
with the `createInput` from the report, and reply with one line per test
(created with its flag key, already in Noibu, or the error and its fix),
plus the reminder that variation code must be live before a test is
started. Creating a test the user did not name, or creating any test
before they ask, is a failure.
