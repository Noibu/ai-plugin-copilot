<!-- overlay: replace — no artifact runtime for an on-page button on this host — drafts are created in the conversation, only for the tests the user names — upstream @ f1ef72abf674 -->

# Creating tests as Noibu drafts

Purpose: turn the recommendations the user picks into real DRAFT tests in
their Noibu account. A draft serves no traffic and stays fully editable;
starting it is a separate action in the console, after the variation code
is deployed.

## When to create

Only after the report is delivered, and only the tests the user names
("create tests 1 and 3", "create all of them"). Never create a test the
user did not name, and never create one because they seemed likely to want
it. When the request is ambiguous ("set them up"), ask once which tests.

## How to create

1. Call `noibu_list_ab_tests` for the domain once. A named test whose title
   is already on the domain is reported as "already in Noibu" and not
   created again (the backend rejects duplicate titles anyway).
2. For each remaining named test, call `noibu_create_ab_test` once, with
   the domain's `domainId`, a one-line `rationale` (the test's `why`), and
   every field of that test's `createInput` unchanged.
3. Reply with one line per test: created (with the flag key and id from the
   result), already in Noibu, or the error and its fix in one clause. When
   a call returns no answer, say the draft may exist and to check the
   console before retrying; do not retry on your own.
4. End with the one reminder that matters: deploy the variation code before
   starting a test.

Never start, update, or delete a test: editing, starting, and deleting
stay in the console.

## Mapping a setup block to `createInput`

The setup block is written in the create screen's vocabulary, so each
field maps directly. `domainId` and `rationale` are added at create time;
everything else goes in `createInput` exactly as the tool takes it.

| Setup block field | `createInput` |
| --- | --- |
| Title | `title` (verbatim; the backend derives the flag key from it) |
| Hypothesis | `hypothesis` (verbatim) |
| Success metric: Add to cart rate | `successMetric: {metric: "ADD_TO_CART_RATE"}` |
| Success metric: Checkout conversion rate | `successMetric: {metric: "CHECKOUT_CONVERSION_RATE"}` |
| Success metric: Viewed page: \<URL\> | `successMetric: {metric: "PAGE_VIEW_TO_URL_RATE", pageUrl: "<URL>"}` (exact page URL) |
| Success metric: Viewed page: page group \<name\> | `successMetric: {metric: "PAGE_VIEW_TO_URL_RATE", pageGroup: "<name>"}` (name exactly as Batch 1 query #8 listed it) |
| Secondary metrics (up to 3) | `secondaryMetrics`: same mapping, plus Average order value → `{metric: "AVERAGE_ORDER_VALUE"}`; "None" → omit |
| Targeting: device type is Mobile / Desktop | `deviceTypes: ["mobile"]` / `["other"]` (no "desktop" value exists; "other" is what the console shows as Desktop) |
| Targeting: UTM \<parameter\> is \<value\> | `utmConditions: [{parameter: "utm_<parameter>", values: ["<value>"]}]`, one entry per condition (ANDed, matching the block's "and") |
| Targeting: location (country) | not supported by the tool: leave it out of `createInput` and say in `setup.targeting` that the country condition is added in the console after creation |
| Targeting: Everyone (skip this section) | omit `deviceTypes` and `utmConditions` |
| Variations: A: Original (control) · B: \<name\>, split equally | `variations: [{name: "Original", isControl: true, trafficSplit: 50}, {name: "<B name>", isControl: false, trafficSplit: 50}]` (three-way: 34/33/33); explicit percentages carry over as written |

The control is always the current experience: `isControl: true` goes on
"Original" and nowhere else. Splits must sum to 100. Variation names
become the flag values the SDK serves, verbatim, so keep the report's
names and the `createInput` names identical.

## What to say

The closing message of the run offers draft creation in one sentence and
carries the reminder that variation code must be live before a test is
started (shoppers bucketed into a variation whose code is not live see the
control while being counted against it, and that data cannot be repaired).
