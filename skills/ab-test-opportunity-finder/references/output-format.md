<!-- overlay: replace — no artifact or page builder on this host — the deliverable is a Markdown report file; field rules unchanged — upstream @ c8d5f1e75f0e -->

# Output format: the "Recommended tests" report

The deliverable is ONE Markdown file, `<domain-short>-test-recommendations.md`,
saved and handed to the user, with `candidates.json` next to it. Do NOT
paste the recommendations into the conversation. The chat gets only the
closing message in the fixed shape SKILL.md defines (quiet-mode rule 5).

The report's job: let a busy merchant read what each test is, see why the
data supports it, see what to confirm on the site, and pick the tests to
create in Noibu. Every test is one section in that order: what to change,
why, the setup as Noibu will receive it, traffic, dev notes.

## Layout

```markdown
# Recommended tests: <domain>

<goal line: "Goal: Increase add to cart rate (success metric: Add to cart rate)", or "Goal: All goals">
<data line: "Noibu, <window> (<N> days, about <sessions> sessions, <mobile share> on mobile)", plus one clause when it applies: the market the report is written for, or "checkout is tracked on <sibling domain>">
Based on Noibu data only. Page layout and copy were not checked on the live site.

## 1. <title>

**Where:** <surface path(s)>
**What to change:** <one or two plain sentences: the change a shopper would notice>
**Why:** <one or two sentences: the measured problem, at most 3 numbers; correlational evidence labeled in one clause>
**Traffic:** <the metric-volume line>
**Already running:** <the overlap line, only when it applies>
**Fix first:** <only when a verified defect sits on this surface>

**Setup in Noibu**
- Title: <title>
- Hypothesis: <hypothesis>
- Success metric: <label> (<baseline>)
- Secondary metrics: <up to 3, or None>
- Targeting: <conditions, or Everyone (skip this section)>
- Variations: A: Original (control) · B: <name>, split equally

**Dev notes:** <1 to 3 sentences per lettered variation, then the "Check on the site first" line>

## 2. ...

## Outside your goal
- <at most 2 one-line candidates on another metric; omit for "All goals">

## Notes
- <footer lines, each only when it applies: the multi-market note, "mobile layout was not checked" when a test rests on mobile evidence, the baseline note, the thin-traffic note, the instrumentation note, the blank-country exclusion; and always: "Expect one or two tests to fall away once someone checks the live page.">
```

Each test also has a `createInput`: the exact `noibu_create_ab_test`
arguments for it, minus `domainId` and `rationale`, per the mapping in
`references/create-drafts.md`. It is not printed in the report; it goes
into `candidates.json`, and a draft is created from it. The report's setup
block and `createInput` must say the same thing.

## Voice

Write for a busy merchant scanning on a phone. Short sentences. Plain
words. One idea per sentence. Say "shoppers" not "sessions" where the
meaning survives. Never use em dashes: use colons, commas, periods, or
parentheses. No hype adjectives, no pitch framing. At most 3 numbers per
test in `why`; pick the ones that carry the argument. Every number must
come from this run's queries, pages, or documents. A baseline you measured
is "about X%" (the battery derives it from session funnel depth and the
product's own definition may differ, so the setup screen's figure wins);
one you did not measure is "baseline shown at setup".

## Field-by-field rules (the `setup` block and `createInput`)

### Title

Name the change, not the metric. About 30 to 45 characters so it scans in
the tests list, and it must still make sense months later. The backend
derives the test's flag key from it, and a title already used on the
domain is rejected, so never reuse one.

### Hypothesis

Match the shape of the field's own example ("Moving the add to cart button
higher on the PDP will increase add to cart rate."). One sentence:
mechanism → expected direction → the success metric.

### Success metric

Exactly one, and it must be the metric the variant acts on most directly,
not the most impressive one. When the user chose a goal, every test uses
that goal's metric: the goal was the brief, and a test whose most direct
metric is something else does not answer it (it can be one line in
`outsideGoal` instead).

The product supports exactly three success metrics (the set the Noibu
create-test API accepts: ADD_TO_CART_RATE, CHECKOUT_CONVERSION_RATE,
PAGE_VIEW_TO_URL_RATE with either a page URL or a page group). Write the
label the way the setup screen shows it:

| Write this label | Use when the variant directly drives... |
| --- | --- |
| Add to cart rate | adding a product to the cart |
| Checkout conversion rate | completing a purchase |
| Viewed page: \<URL\> | reaching one specific page (a landing page, the checkout URL, a sale page) |
| Viewed page: page group \<name\> | reaching any page in a Noibu page group (product pages, collection pages, checkout, cart) |

So "more checkout starts" is `Viewed page: page group Checkout` (or the
checkout URL when the domain has no Checkout group), "more product page
views" is `Viewed page: page group Product`, and "more collection views" is
`Viewed page: page group Collection`. Use the page group names exactly as
the domain's data reports them (Batch 1 query #8 lists them); if the
domain has no page groups, fall back to the URL form with the path from
Batch 1 #5. Average order value exists as a secondary metric
only. Do not use metrics outside this set (no bounce rate, revenue per
session, exit rate, scroll depth, email capture, or per-step completion
rates). If a test's natural KPI is unsupported, choose the nearest
supported metric and, at most, note in one clause what the test
consequently does not measure.

If checkout completion is not instrumented on the domain (Batch 1 check
#4), do not use Checkout conversion rate; use `Viewed page: page group
Checkout` (or the checkout URL) and say in one clause that the test
measures reaching checkout, not buying.

### Secondary metrics

Up to 3. They give directional context and never decide the winner, so pick
the ones that would catch a side effect of this specific variant (a cart
change that might depress order size: Average order value; a routing change
that might just shuffle traffic: Viewed page on the destination). "None" is
a fine value; do not pad.

### Targeting

Only write conditions the product can express: UTM campaign, UTM medium,
UTM source, UTM term, UTM content, device type (Mobile, which includes
tablets, or Desktop), and location (country, set in the console).
Conditions combine with "and". Example: "UTM medium is email and device
type is Mobile". If the evidence is not segment-specific, write "Everyone
(skip this section)": targeting narrows traffic, and less traffic means a
longer wait for a verdict, so recommend a condition only when the measured
gap lives in that segment. If a candidate's natural audience cannot be
expressed with these conditions (returning customers, cart value bands,
logged-in state), either widen it to everyone and note the dilution in one
clause, or drop the candidate; do not invent targeting the setup screen
does not offer. On a multi-market store, write the market as a location
condition to be added in the console after creation ("Location is United
Kingdom, set in the console"), and keep `createInput` free of it. (Custom attributes registered through the SDK can extend
targeting, but that is dev work, not a setup-screen option; if a test
genuinely needs one, say so in dev notes, not in Targeting.)

### Variations

Variation A is always "Original" and is the control; never rename it. Name
B (and C at most: more variants split traffic thinner and stretch the
verdict) with a short noun phrase describing the experience, because that
name is what the results page, the verdict banner, and the SDK's flag value
will carry ("Button above info box", not "Variant B"). Default to "split
equally"; recommend an uneven split only with a stated reason.

### Dev notes

Noibu owns the split, targeting, and metrics; the store's own code decides
what each variation renders, by reading the assignment from Noibu's SDK
(the pattern is documented in `references/shared/noibu-feature-flag-sdk.md`
for the skill that writes the code), so a developer is in the loop on every
test and the variation code must be live before the test starts. These
notes are the one to three sentences per lettered variation that tell that
developer what to build: the page, the element or block, and what changes.
Concrete enough to start from ("PDP: move the whole add-ons block after the
trust badges block in the product info column"), never actual code, and
never instructions for A (the control renders the current experience by
definition). Name the block to change, not the inner element. End each
test's notes with one "Check on the site first" line: what the developer
must confirm on the live page before building (the control looks as the
report assumes, the variant does not already exist), because this host
does not read the site.

### Fix first

Omit when there is nothing verified to fix. When a verified defect (error,
broken element, poor vitals, from this run's queries) sits inside a
test's surface, one line: what to fix and why it would otherwise pollute
the test's result.

## Guardrails and results framing

Guardrails are fixed and automatic in the product: error rate, P95 LCP,
sample ratio mismatch. They warn; they never block. Do not present ordinary
metrics as "guardrails" and do not invent per-test guardrails. Results in
the product are Bayesian ("chance to win", verdict banners); do not explain
this in the report and do not fabricate expected lifts, run durations, or
power calculations. No confidence ratings, no sizing, no duration
estimates: the product computes "About N days to reach a verdict" from the
real traffic at setup. The one traffic figure a card carries is
`metricVolume`: the measured weekly count of the success metric's event,
which is a fact from this run's queries, not a projection.

## Ranking

Order by evidence strength × traffic exposure × plausible effect, not
novelty. Aim for 3 to 6 tests.
