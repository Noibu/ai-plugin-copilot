---
name: build-business-context
description: "Builds a rich business-context Markdown file so Noibu skills stop hallucinating and reason with real context about the business. Use this skill whenever someone wants to create, set up, or improve their business context file, onboard a new store to Noibu, 'build my context file', 'make a context file', capture company background for the assistant, or asks why answers keep getting their business wrong. Also trigger on looser phrasings like 'help the assistant understand my business', 'document my store for Noibu', 'what should I tell Noibu about my company', or when a user is starting with Noibu and has no context file yet. Even if the user just says 'build the context file' or 'set up context', use this skill. It does deep web research on the business, interviews the user about goals, money leaks, deployment process, and team setup, writes a clean Markdown file, then persists the context to the Noibu backend for the domain."
---
<!-- overlay: replace — no interactive picker tool — interview goes conversational; drops Claude-specific branding (CLAUDE.md → business-context file) — upstream @ 7d279ce1dd2a -->

# Build Business Context (the MD file)

Noibu's analytics skills are dramatically more useful when they have real context
about the business. Without it, they fall back on generic e-commerce assumptions and
produce confidently wrong conclusions — flagging a PDP as "broken" when it has a
deliberate minimum-order quantity, or treating B2B abandonment like B2C abandonment.
This skill exists to capture that context once, in a single Markdown file the user can
keep and ship to their whole team, so nobody has to spend an hour pushing back to get
to the truth.

**Your job:** produce one well-structured business-context Markdown file by combining
(1) deep web research you run yourself and (2) a focused interview with the user about
things only they know. The output is a file — not a chat answer.

The section-by-section skeleton — and the model for tone, depth, and structure — lives in
`references/md-template.md`. Read it early; it is both the template to fill in and the
guide to the format the finished file should follow. The full bank of interview questions
lives in `references/interview-questions.md`.

---

**Call metadata:** on every `noibu_*` call this skill drives, pass `skill: "build-business-context"`, `skillVersion: "cowork:fbf39d7"`, and a short `skillStep` (see the `querying-noibu-data` skill).

## How this works, end to end

1. **Context existence check** - check if this domain already has context in Noibu. Resolve the domain and check for an existing context up front, so you don't do the work twice.
2. **Setup** — a few quick questions so you know what this file needs to cover, including whether they have recorded calls (Gong / Fellow) you can mine.
3. **Research** — you do deep web research on the business yourself, optionally pull and mine their recorded call transcripts, and report back what you found.
4. **Interview** — you ask the user, in small conversational batches, the things research and transcripts can't tell you.
5. **Assemble** — you write the Markdown file, clearly marking anything still unconfirmed.
6. **Deliver** — you save the file and hand it over, with notes on what to verify.
7. **Persist** — you push the finished context into the Noibu backend so it's stored against the domain.

Do not skip straight to writing the file. The whole value here is that the file is
*grounded* — every claim either comes from research you can point to or from the user
directly. Guessing defeats the purpose.

---

## Test mode (fetch + persist only)

A persist-only mode exists to exercise the two Noibu integration points (the Step 1
fetch and the Step 7 persist) without the generative middle. It triggers **only** on an
explicit "test mode" request accompanied by an uploaded Markdown file — never infer it.
Read `references/test-mode.md` and follow it exactly when that happens.

---

## Step 1 — Resolve the domain and check for existing context (do this first)

Before anything else — before the other setup questions, before research — pin down the
domain and find out whether it already has context:

1. **Ask which website / store this is for** and get the actual domain (e.g.
   `creativebag.com`). Confirm you've got the right site ("Just to confirm — this is
   [domain-name], the [one-line description]?").
2. **Resolve it** with `noibu_get_domain` (fall back to `noibu_list_domains`). Keep both the
   domain **UUID** (`id`) and its integer **`numId`** — the read uses the UUID, the final
   persist uses the `numId`, so holding both now means no re-resolving later. Then call
   `noibu_get_business_context` with the UUID (`domainId`).
3. **Gate on the result.** A context already exists when the response has a `businessContext`
   with a non-empty `fullContent`. If it does, tell the user and ask whether to regenerate —
   only continue if they say yes (you'll overwrite it at the end). If `businessContext` is
   null, continue.

Only once you're cleared to proceed, ask the rest of the setup questions below.

## Step 2 — Setup questions

Ask these together in one short batch — they're quick and they shape everything else:

1. **Do you already have any context files, brand guidelines, personas, or a strategy
   doc I should fold in?** If yes, read them before researching so you don't
   re-derive what they already wrote.
2. **Do you record customer or sales calls in Gong or Fellow?** If yes, ask whether you
   should pull the transcripts and mine them for context. Call transcripts are one of
   the richest grounding sources you can get — customers describe their pains, goals,
   buying process, and gotchas in their own words, which is far more reliable than
   anything inferred from a website. If they say yes, note which tool(s) and roughly
   which calls matter (a specific account, a date range, or "all of them"), so you can
   scope the pull in Step 3.

If they don't know the domain or the file is for a business that isn't online yet,
adapt — but you almost always need the domain to do useful research.

---

## Step 3 — Deep web research

Now go learn everything publicly knowable about this business, using whatever web
search / fetch tools are available in this session. This is read-heavy work — spend
real effort on it, since it's what keeps the eventual recommendations from being
hallucinated. Work through the questions below one at a time rather than skimming;
depth here is the whole point of the step.

Give yourself: the confirmed company name and domain, and the specific questions
below to answer.

Aim to answer as many of these as you can from public sources:

- **What they sell** — core product categories, flagship products, price points.
- **Who they sell to** — B2B, B2C, or both; consumer vs. trade; geography.
- **How long they've been around** — founding year, company age, any history/heritage.
- **Ownership structure** — family-owned, private, PE-backed, public, franchise, etc.
- **Size signals** — rough employee count, number of locations, revenue band if public.
- **Ecommerce platform** — Shopify, BigCommerce, Magento/Adobe Commerce, WooCommerce,
  Salesforce Commerce, custom, etc. (Check page source, `myshopify` redirects,
  `cdn.shopify.com`, BigCommerce headers, builtwith-style signals, careers pages
  mentioning the stack.)
- **Apps / tech in the stack** — reviews (Yotpo, Okendo), search (Searchanise,
  Algolia), loyalty, subscriptions, payment/checkout providers, analytics, CDPs.
- **Positioning & differentiators** — how they describe themselves, what they claim
  sets them apart, price posture (premium vs. value).
- **Competitors** — who they name or who obviously competes with them.
- **Brand voice** — read their homepage, About page, and a few product pages. Note
  spelling conventions (e.g. Canadian/UK spelling), tone, and personality.
- **Recent news** — funding, acquisitions, expansions, leadership changes, awards.

Then **report back what you found** in a short, skimmable summary, and explicitly
flag what you *couldn't* confirm. Tell the user something like: "Here's what I dug up.
I couldn't confirm your platform or your payment processor from the outside — those are
on my question list. Anything below that's wrong, correct me now." Letting them correct
research before the interview is much cheaper than fixing the final file.

Treat every research finding as provisional until the user confirms it. Mark
unconfirmed items as `[CONFIRM]` in the eventual file rather than stating them as fact.

---

## Step 3b — Mine recorded calls (Gong / Fellow), if the user has them

Only if the user said in setup that they record calls and want them pulled — skip
entirely otherwise. When they do, it is often the highest-value input to the whole
file: read `references/mine-recorded-calls.md` for how to scope, read, and attribute
transcripts, and fold the findings into the same report-back as the web research.

---

## Step 4 — The interview (conversational, in small batches)

Research can tell you what the business *is*. It cannot tell you what the business
*wants*, where it's bleeding money, or how it ships code. That's what the interview is
for — and it's the single most important input for keeping future recommendations
grounded.

**Ask in small, themed batches — not one giant questionnaire.** Three or four related
questions at a time, then react to the answers and ask smart follow-ups before moving
to the next theme. This feels like a conversation, surfaces better answers, and lets
you adapt (e.g., if they say "we have no engineers," your deployment follow-ups change
entirely). The full question bank with follow-ups is in
`references/interview-questions.md` — read it and pull from it; don't ask every
question mechanically.

**Ask as plain conversational text, not a picker widget** — this host has no
interactive-choice tool, so batch 3-4 related questions together as prose. Where your
research strongly suggests a likely answer, say so and ask for a quick confirm (e.g.
"I'm guessing you're on Shopify, based on your checkout URLs — is that right, or
something else?") rather than listing it as one of several unlabeled options; that's
faster to answer than an open question and doesn't need a widget to do it. For
questions where several answers can be true at once (e.g. which money leaks concern
them, which KPIs they watch, which channels they use), just ask them to list all that
apply in their reply.

**Keep the genuinely open questions as prose**, asked as a short follow-up after the
rest of a batch. Two kinds resist becoming a quick-confirm: exact numbers
(target/current ROAS, conversion rate, AOV) and the "what about your store makes
standard metrics misleading?" narrative — that one's whole value is the unprompted
answer, so don't try to shortcut it with suggested options.

Cover these themes, roughly in this order:

1. **Goals for the year.** What does success look like this year? Revenue target,
   growth target, a specific initiative? What's the single most important outcome?
2. **Where they think they're losing money.** Where on the site does it feel like money
   leaks out? Checkout? Mobile? A specific page or step? Paid traffic that doesn't
   convert? This is gold — it tells future skills where to look first.
3. **Performance targets and current numbers.** Target ROAS vs. current ROAS. Target
   conversion rate vs. current. AOV. Any KPI they actively watch. Numbers they give
   you are far more trustworthy than anything inferred.
4. **The customer, in their words.** Who's the most valuable customer? Any B2B/B2C
   split? Buyer personas? Anything about their store that makes standard e-commerce
   metrics misleading (minimum order values, minimum quantities, multi-session B2B
   buying, etc.)? Capture these "gotchas" carefully — they are exactly what prevents
   wrong analysis.
5. **Tech stack & site quirks.** Confirm the platform you researched. Custom theme or
   stock? Custom checkout? Payment processor and vaulting. Key apps. Anything fragile
   or non-standard that makes code changes risky.
6. **Deployment & dev team.** What's the typical deployment process? Do they have
   internal engineers, an agency, a freelancer, or no one? How do changes get reviewed
   and shipped? Is there a staging environment? How risk-averse is the team about
   merging changes? This determines what kinds of recommendations are even realistic.
7. **Support / CS workflow.** Team size, ticketing system (or lack of one), how issues
   get triaged and handed off.
8. **Marketing & channels.** Which channels they use, which are working, which are
   underused, biggest near-term opportunity. Seasonal patterns.
9. **How they want the assistant to work with them.** Tone, format, plain language vs.
   technical, plan-before-acting, how opinionated to be.

Keep it human. If an answer opens a door ("paid is killing us"), walk through it before
moving on. You don't need an answer to every question — capture what they have, mark
the rest, and move on. A shorter honest file beats a padded one full of guesses.

---

## Step 5 — Assemble the Markdown file

Build the file from `references/md-template.md`, populated with research + interview
answers, matching the depth and voice its section prompts describe.

Principles that make these files actually work:

- **Most important guidance first.** Lead with whatever would most change how the
  assistant reasons about this store — usually the customer framing and the "how to
  read our analytics" gotchas. A reader skimming the top should immediately get the
  things that prevent wrong conclusions.
- **Explain the *why*, not just the rule.** "Accordion clicks are research behaviour,
  not abandonment — B2B buyers check case quantities before a later logged-in reorder"
  beats a bare "don't flag accordion clicks." Future analysis reasons better with the
  reason.
- **Adapt sections to the business.** The template is a starting point, not a form.
  Drop sections that don't apply, add ones that do. A single-founder DTC store and a
  40-year family B2B supplier need different files.
- **Mark every unconfirmed thing `[CONFIRM]`.** Never print a guessed payment
  processor or platform as fact. It's better to flag a gap than to seed a future wrong
  analysis. Collect these into a short "Items to confirm" list at the very end so the
  user can quickly finalize.
- **End with a standing-reminders checklist.** A short numbered list restating the most
  important rules (the customer lens, the money-leak hotspots, the high-risk areas).
  This counters context dilution in long sessions — it keeps the critical rules in view
  even deep into a chat.
- **No em-dashes in the file** unless the user's own brand voice uses them. Keep prose
  clean and plain.

**Filename — always `<domain>-context.md`.** Use the domain name verbatim, lowercased, with
the suffix appended (render `www.my-store.com` as `www.my-store.com-context.md`). Don't vary
the suffix — this fixed `*-context.md` convention is what lets consuming skills find the
file by glob.

**Sentinel header — required.** Make the very first line of the file a sentinel comment
so a reader can identify the document and match it to a domain without parsing the prose:

```
<!-- noibu-business-context: uuid=<domain-uuid> domain=<domain-name> generated=<YYYY-MM-DD> -->
```

Use the domain **UUID** resolved in Step 1 and the bare store domain name. Skills that consume
this file match on this `uuid` (not the filename), so it must be present and correct — a file
whose sentinel `uuid` doesn't match the current run is ignored. Put the sentinel above the
`# [Company] — Business Context` title.

Save the file to the workspace, then state the **filename** and tell them it appears in this session's file list and syncs to OneDrive under `Documents/Cowork`. Don't invent a filesystem path — this host doesn't give you one.

---

## Step 6 — Deliver

Present the file and give the user three things, briefly:

1. **What's in it** — one or two sentences, not a recap of every section.
2. **What to confirm** — the `[CONFIRM]` items, so they can finalize quickly.
3. **How it's used going forward** — once persisted (Step 7), Noibu skills pick it up
   automatically for this domain, for the whole team, without anyone re-uploading it
   per session. If the user also wants a copy for their own records, tell them the file
   is theirs to keep — they can save it wherever they keep other store documentation.

If there are any `[CONFIRM]` items in the file, ask them to go through them and you'll update
the business context.

When finalized, tell the operator the file will now be saved to Noibu's system so other team
members get it automatically.

---

## Step 7 — Persist to the Noibu backend

The saved Markdown file is the local artifact; the backend is where the context lives
so Noibu can use it. The domain was already resolved and the overwrite already cleared
in Step 1, so this is a single write: call `noibu_insert_business_context` with the
domain's integer **`numId`** as `domainNumId` (NOT the UUID) and the entire Markdown file
as **`fullContent`** — exactly as saved, nothing dropped, no splitting or summarizing. The
whole document goes into the one `fullContent` field. Then tell the user it's saved to Noibu for `<domain>`.

---

## Hard rules

- Never print researched facts (platform, processor, ownership, etc.) as confirmed —
  mark unconfirmed items `[CONFIRM]`.
- Always do the web research before the interview, and report findings before asking
  questions, so the user corrects bad research cheaply. Spend real effort on the
  research — it's what keeps recommendations grounded instead of hallucinated.
- Ask whether the user records calls in Gong or Fellow, and only pull transcripts if
  they say yes and ask you to. Scope large pulls with the user first; never silently
  read hundreds of calls. Attribute transcript-derived facts to the call they came from.
- Ask interview questions in small conversational batches (3-4 per batch), never as one
  wall of questions, and never via a picker widget — this host doesn't have one. Lead
  with a research-informed guess and ask for a quick confirm where you have one;
  reserve open prose for genuinely open questions (exact numbers, the "what makes our
  metrics misleading" narrative).
- The output is always a saved Markdown file, never just a chat response.
- Lead the file with the highest-leverage context, and always include a
  standing-reminders checklist at the end.
- Adapt the structure to the business; the template is a guide, not a fill-in form.
- Resolve the domain and check for an existing context in Step 1, before any research.
  If one exists, get the user's go-ahead to regenerate before doing the work.
- Always persist the finished context to the Noibu backend: call
  `noibu_insert_business_context` with the domain's `numId` (`domainNumId`) and the entire
  Markdown file as `fullContent`.
