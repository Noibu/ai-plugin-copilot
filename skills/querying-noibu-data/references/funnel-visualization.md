<!-- overlay: replace — no chart-rendering tool on this host — funnel renders as a markdown table — upstream @ 671622c86f93 -->

# Funnel visualization

Read this reference when the user asks to chart, visualize, or "show" an ecommerce conversion funnel (e.g. "show me the funnel", "checkout funnel chart", "purchase journey chart", "render the funnel for last 7 days vs previous"). Fetch the per-step session counts from `noibu_search_sessions` (or `noibu_get_page_visits`) first, then present them as a markdown table.

Analytical funnel prompts like "where do users drop off" or "abandonment by step" terminate at `noibu_get_page_visits` (see `references/page-visits.md`) — do not detour through this reference.

## How to present

1. Fetch the per-step session counts (`noibu_search_sessions` or `noibu_get_page_visits`).
2. For each step, compute:
   - **Conversion %** — this step's sessions ÷ step 1's sessions.
   - **Drop-off %** — 1 − (this step's sessions ÷ previous step's sessions). Step 1 has no drop-off.
3. Render a markdown table:

   | Step | Sessions | Conversion % | Drop-off % |
   |---|---|---|---|
   | Add to cart | 6,574 | 100% | — |
   | Checkout started | 2,420 | 36.8% | 63.2% |
   | Payment submitted | 1,391 | 21.2% | 42.5% |
   | Checkout completed | 1,283 | 19.5% | 7.8% |

If the user asked for a period-over-period comparison (e.g. "last 7 days vs previous"), add a **Δ** column: the percentage-point change in that step's Conversion % vs. the prior period.

Format large numbers with thousands separators; round percentages to one decimal place.
