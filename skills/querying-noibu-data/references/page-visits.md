<!-- overlay: replace — no scrollmap-rendering tool on this host — clickmap keeps its console link, scrollmap falls back to aggregate stats — upstream @ aa887f827ac8 -->

# Page visits analytics

Read this reference when the user asks page-level questions: per-page traffic, time on page, web vitals (LCP/CLS/INP/FCP/TTFB/FID), landing/exit pages, visual errors, scroll depth, click/scroll behaviour, or cohort behaviour for a specific URL.

## noibu_get_page_visits

Page-level analytics. One row per page visit. `orderBy` is REQUIRED (inside `queryInput`). Use for:

- Per-page traffic: URL visit counts, per-page engagement, bounce by URL
- User interaction — **per-page only**: `CLICKED_SELECTOR_COUNT` (total clicks), `CLICKED_SELECTORS` (CSS selectors), `CLICKED_TEXT` (text users click — best per-page CTA / rage-click / friction signal). Site-wide aggregation across sessions → `noibu_search_sessions`'s `CLICKED_TEXT`, not this tool.
- Page timing: `PAGE_VISIT_DURATION`, `PAGE_VISIT_START_TIME`, `PAGE_VISIT_END_TIME`
- Performance / web vitals — `LCP`, `CLS`, `INP`, `FCP`, `TTFB`, `FID`. **Use
  `QUANTILE_75` first** (the canonical Core Web Vitals statistic — Google's
  CrUX defines the "good / needs-improvement / poor" thresholds at p75, and
  the Noibu Console ranks pages by p75). Only use `MEDIAN` if the user
  explicitly asks for the median, or `QUANTILES` for the full distribution.
- Visual UX defects: `VISUAL_ERROR_COUNT`, `VISUAL_ERROR_SNIPPETS` collection
- Engagement depth: `MAX_SCROLL_DEPTH`, `MAX_PAGE_HEIGHT`, `VIEWPORT_HEIGHT`,
  `VIEWPORT_WIDTH`
- Navigation role: `IS_LANDING_PAGE`, `IS_EXIT_PAGE` (booleans per page visit)
- Page context: `PAGE_TITLE`, `LANGUAGE`, `REFERRING_URL`, `PREV_URL`, `PREV_PAGE_GROUPS` collection
  (`REFERRING_URL` keeps its query string — it is the only field that retains URL parameters;
  see "Recovering URL parameters" in SKILL.md)
- Page-level conversion: `CHECKOUT_COMPLETED` per-page, which specific pages are visited
  by converting sessions
- Cohort analysis (every page-visit row carries denormalized session context — no JOIN
  needed): `SESSION_BOUNCED`, `SESSION_CONVERSION_FUNNEL_DEPTH`, `SESSION_UTM_*`,
  `SESSION_ORDER_ID`, `SESSION_CUSTOMER_ID`, `SESSION_WS_PATH`
- Segmentation by: URL, `PAGE_TITLE`, `LANGUAGE`, browser, OS, device type, country,
  region, checkout status, `IS_LANDING_PAGE`, `IS_EXIT_PAGE`, `PREV_URL`,
  `SESSION_BOUNCED`, `SESSION_CONVERSION_FUNNEL_DEPTH`, `SESSION_UTM_*`

## Web vitals: aggregate at p75, not the mean

When the user asks "how is my LCP" / "what's my INP" / "which pages are slow", lead with `QUANTILE_75` of the relevant field. The 75th percentile is the canonical Core Web Vitals statistic (Google CrUX defines the good/poor thresholds at p75, and the Noibu Console ranks pages by p75). Do NOT use `AVG` — outliers distort it. Only use `MEDIAN` if the user explicitly asks for the median, or `QUANTILES` when they want the full distribution.

Web vitals and `VISUAL_ERROR_COUNT` live here, not in error tools. "Slow pages", "broken feeling", "UX quality" questions route to this tool — do NOT reach for error tools.

## Clickmaps: callable — scrollmaps: fall back to aggregate stats

When the user wants to **see** click behaviour ("show me the clickmap"), `noibu_visualize_page_visits` (`visualization.clickMap`) is the right call — it returns ranked per-selector click, conversion, and revenue numbers, plus a `consoleUrl`. Render the numbers as a markdown table and state the `consoleUrl` plainly; note it opens the general heat-maps section rather than the exact url/device/date-range queried, and don't claim a heatmap image is visible — you can't confirm this host rendered one.

Scrollmaps are **not** callable this way — `noibu_visualize_page_visits` has no reliable scrollmap equivalent here. Fall back to `noibu_get_page_visits` aggregate stats and present them as a markdown table:

- Click behaviour (if not using the clickmap call above): `CLICKED_SELECTOR_COUNT` (total clicks), `CLICKED_TEXT` (top clicked text/elements) — segment by URL, ranked
- Scroll behaviour: `MAX_SCROLL_DEPTH` (e.g. average or `QUANTILE_75` scroll depth by URL)
