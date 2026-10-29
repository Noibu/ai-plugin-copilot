---
name: accessibility-audit
description: "Audit a website for WCAG 2.1 AA accessibility violations using automated scanning and Noibu traffic data, and optionally draft the fixes. Use when you want to know what's inaccessible across the store, check WCAG or ADA compliance, find contrast or heading errors, audit for screen reader support, generate an accessibility report, or fix accessibility violations with pull requests or tickets."
---
<!-- overlay: replace — no widget, scheduling or connector-setup surface on this host — the fix-automation offer becomes plain conversational text gated on runtime tool presence, the recurring-schedule flow is dropped, and the report is handed off as an HTML file because there is no bundled pdf skill — upstream @ 022e5f5dcd19 -->

# Accessibility Audit

**Requires a linked computer running the desktop companion app** — see **Setup** below before anything else.

Someone asks for an audit right now; it produces an HTML report, then offers to open PRs or tickets for the top fixable findings if the tools for that are in the session. There is no recurring-schedule flow on this host.

---

**Call metadata:** on every `noibu_*` call this skill drives, pass `skill: "accessibility-audit"`, `skillVersion: "cowork:fbf39d7"`, and a short `skillStep` (see the `querying-noibu-data` skill).

## Setup

The scan needs `mcp__remote-devices__Claude_Browser__*`, which exists only in a session linked to a computer running the desktop companion app. If those tools are genuinely absent (not just offline): tell the operator plainly this needs a linked computer with the desktop app open, and stop. Never offer a reduced fallback (e.g. manual web-fetch review) — it's a materially weaker audit than this skill promises.

**Nothing else is required to run a plain audit** — it never touches a repo or ticket platform on its own.

1. **Scanning browser** — the built-in browser (`mcp__remote-devices__Claude_Browser__*`). Once confirmed present (see above), it still needs the linked desktop app to be open and online, not just installed, at the moment it's used.
2. **Domain** — UUID → use it. Name only → `noibu_get_domain`. Neither → `noibu_list_domains`; use it if there's exactly one, else ask. Never block on this.

Repo and ticket tools are only needed if the operator opts into automated PRs/tickets, and only if such tools are already in the session — see Step 4. They are never solicited.

---

## Rules

- **Never expose the scanning tool by name.** No "axe-core" in customer-facing output.
- **Every WCAG criterion links to its W3C Understanding page**: `https://www.w3.org/WAI/WCAG21/Understanding/<slug>.html`, confirmed via axe-core's `helpUrl`.
- **"Pageviews," never "visits."**
- **"GitHub Issue," never bare "Issue."**
- **"Automated tickets"** is the umbrella term for any ticket/issue automation produces. Never "Link" or split by platform in a table.
- **"Severity"** = axe-core's `impact` (critical/serious/moderate/minor). **"Priority"** = the traffic-weighted ranking score (`references/wcag-violations.md`) — determines row order, never shown as a column.
- **Automation naming**: config key `auto_pr`, card label "Auto-draft PR," button copy "Automatically draft a pull request (PR)." Never "Auto-fix."
- **Never merge anything.** A human does all merges, issue-closing, and status changes.
- **Link out, don't embed derived status** — never reproduce a GitHub Issue/ticket's live status in the report.
- **Pool before scoring or reporting anything.** Same selector + criterion = one finding (`references/wcag-violations.md` → Pooling).
- **A "link" means an actual `<a href>`, never styled text.** External URLs: `<a href="...">text</a>`. Same-document (e.g. Key Findings row → detail section): `<a name="finding-N"/>` + `<a href="#finding-N">`.
- **A subagent (the Agent/Task tool) may run Step 1's scan only** — read-only and self-contained, so delegating it to keep this session's context light is fine. It must return the raw pooled findings (data only); nothing else may be delegated. **Steps 2–4 always run directly in this session** — the report build, the chat reply, and every `SendUserFile` call. A subagent's own tool calls (including `SendUserFile`) don't reliably surface to the operator, and its prose summary drifts from Step 3's required chat structure.

---

# On-demand flow

## Step 1 — Resolve and scan

1. Confirm the built-in browser is available, then resolve the domain.
2. Pull the top 10 pages by 30-day pageviews.
3. For each page: navigate to the plain URL (no query parameter), inject axe-core, call `axe.run()`. See `references/wcag-violations.md` → **Scan mechanics**. No screenshots — selector + HTML snippet is enough to locate a finding.
4. Pool every violation (see **Pooling**), score with the Priority formula, sort descending.

This step alone may run in a subagent (see **Rules**) if that helps keep the session's context light — it must hand back the pooled, ranked findings as data, and nothing more. Steps 2 onward always run directly in this session.

## Step 2 — Generate the report

Build the report from the pooled, ranked findings as a self-contained HTML document — there is no bundled pdf skill on this host. Plain HTML elements, minimal inline CSS, no card chrome or dashboard layout, same as the other skills' exports.

- **File name**: `[Domain] Accessibility Audit [MM] [DD] [YYYY]` (e.g. `Carsncards.com Accessibility Audit 09 09 2026`).
- Title, subtitle (domain, standard, page count), audit date, one-line spacer, divider rule.
- Disclaimer (verbatim): *"This skill generates automated WCAG 2.1 AA accessibility reports. Automated tools detect 30%–90% of issues on the pages that were scanned. Results are informational only, not legal advice or a compliance certification. Manual testing by a qualified professional is required for complete coverage."*
- **Executive summary** — intro paragraph, stats paragraph, then severity metric cards. No specific finding named here.
  - **Intro** (2–3 sentences): pages scanned and why, total pageviews covered, AA-is-the-legal-standard line, and (if automation already ran this cycle) how many findings were auto-filed as tickets.
  - **Stats paragraph**: raw instance count + pooled/unique fix count, with the pooling relationship spelled out. E.g. *"We found 271 violation instances across the pages. Many repeat the same issue on multiple pages — once pooled, that's 27 unique fixes."* No severity breakdown or rule-type count here.
  - **Metric cards** — one row, four cards (Critical/Serious/Moderate/Minor) as a single four-column `<table>`, each cell bordered and shaded with a large bold number over a small label. Colors: critical `#A32D2D`/`#FCEBEB`, serious `#854F0B`/`#FAEEDA`, moderate `#8A6D0B`/`#FEF6DA`, minor neutral gray. Zero counts still get a card. Equal-width cells, full content width, matching padding and border.
- **Pages scanned** — table: page (hyperlinked to its live URL), 30-day pageviews, violation instances.
- **Key findings** — table: # / Finding (links to detail section) / WCAG (linked) / Severity / Automated tickets (only if at least one exists at generation time).
- One detail section per finding: severity badge inline and left-aligned at the start of the "Pages affected / Pageviews affected" line (e.g. **Severity:** SERIOUS | Pages affected: 10 | Pageviews affected (30-day, est.): 95,120); WCAG Violation line (criterion + link); plain-English description; CSS selector and HTML snippet from `nodes` (monospace block); Suggested fix (+ ticket link if one exists); Scanned pages affected (comma-separated, each hyperlinked) as the final line.
  - **Every element in a finding's detail section is its own block with real spacing, never packed tight against its neighbor** — the severity/pages-affected header line, the WCAG line, the description, the Selector label and block, the Example HTML label and block, the Suggested fix line, the Automated ticket link line (when present), and the Scanned pages affected line. Give each a non-zero `margin-bottom` sized to its font; nothing should render with less than ~6px of breathing room from what's above or below it.
  - The document is left-aligned by default; the only centered elements are the metric cards' number and label.

No appendix, no per-page violation matrix.

**Before delivering the report**, verify a link resolves in each of: a Pages Scanned row, a Key Findings row, a WCAG criterion, a finding's Scanned pages affected line. Also visually check **every** finding's full detail section — not a sample — for overlap between any two adjacent elements (metric cards included). Fix any defect before Step 3 — don't ship and caveat.

## Step 3 — Reply and deliver the report

One turn, exactly these two pieces in order, nothing else. The file card always renders last regardless of call order, so call it last — and once it's called, the turn is over: no closing sentence, no recap, no "let me know if..." after it.

1. **Chat reply**, three short paragraphs:
   - **Scope** — pages scanned and why, total pageviews, AA-standard line. Same content as the report's intro.
   - **Stats** — raw instance count + pooled/unique fix count with the pooling relationship, then severity as a bullet list (always all four, even at zero):
     > We found 271 violation instances across the pages. Many repeat the same issue on multiple pages — once pooled, that's 27 unique fixes:
     > - 24 serious
     > - 2 moderate
     > - 1 minor
     > - 0 critical
   - **Top violation** — name the single highest-priority finding, e.g. *"The highest-priority issue is **[finding name]**, affecting an estimated [pageviews] pageviews across [N] pages"* — close with *"...the rest of the findings and suggested fixes are in the report below."* This is the only finding named in chat; the report's own intro/stats stay aggregate-only.
2. **Deliver the report file.** No text of any kind after this call.

Step 4's fix-automation offer, if it applies at all, comes after as its own turn — don't fold it into this one.

## Step 4 — Offer fix automation (only when the tools are already there)

**Never ask the operator to connect anything, and never name a tool that isn't in the session.** A plugin's tools arrive already deployed or not at all — the operator cannot add one — so a request to connect is a dead end. Check the session tool list silently.

**No repo or ticket tools present → skip this step entirely.** No offer, no explanation of what could have been offered, no mention of what's missing. Step 3 was the end of the run.

**Repo or ticket tools present** → after the report is delivered, as its own turn, offer in plain chat only what those tools can actually do — drafting PRs, opening tickets, or both. One short question; ignoring it is the decline.

If the operator accepts, apply the fix-selection rule against the report just generated:

> **Fix-selection rule**: for the top 10 fixable pooled findings (by Priority), draft a PR (`references/platforms/<platform>.md`) or open a ticket, whichever was chosen. Not traceable to a file → ticket only, labelled `needs-investigation`, doesn't consume a slot. Never touch checkout code or merchant-managed settings. Never merge anything.

1. Resolve what the chosen path needs, asked once and only once: for PRs, which repo holds the theme (`owner/name`); for tickets, the team or project. Nothing is persisted — this session only.
2. Run the fix-selection rule. Every ticket and PR description uses the same fields as the report's finding detail section (Step 2): severity/pages affected/pageviews affected header line, WCAG line (linked), plain-English description, selector + HTML snippet, suggested fix (or the `needs-investigation` note), and a **Scanned pages affected** line listing every page the pooled finding appears on — each one an actual hyperlink to its live URL, never bare page names like "Homepage."

   Reply with one line per finding that got something created — the finding name, then an arrow, then the ticket/PR reference **hyperlinked to that ticket or PR's real URL** (the platform tool's create call returns it; if a platform ever doesn't return a URL, use the platform's standard item-URL pattern rather than posting a bare ID like `NOI-12454` or `#172` with no link). Don't reuse the wording of any example above verbatim — it's illustrating the shape, not text to copy. More than 10 fixable findings → append *"N more findings weren't addressed this run — they'll be picked up on the next scan."* The report isn't regenerated.

There is no recurring-schedule flow on this host — don't offer one, and don't explain its absence.
