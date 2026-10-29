<!-- overlay: replace — no bundled pdf skill — hand off the HTML file itself, not a rendered PDF — upstream @ d8aa7dae8113 -->

# Export the report

Build a clean document from the data already in context. The full analysis results are visible earlier in this conversation — use those numbers directly. Do not call any Noibu tools. Do not re-query.

Build the HTML as a clean document. No card chrome, no CSS variables, no dashboard layout. Use plain HTML elements with minimal inline styles.

**Structure** (in order):
1. `<h1>` — domain and date range (e.g. "Segment Analysis — atari.com · May 17–Jun 16 2026")
2. **Overview section** — `<h2>Overview</h2>`, the key stats as a sentence, then the three dimension tables as plain `<table>` elements (Devices, Countries, Channels)
3. For each priority finding — `<h2>[Finding type] — [Segment name]</h2>` (e.g. "Campaign Drag — Facebook CPC"), then a short paragraph with the key number and why it matters

**CSS — minimal, no variables:**
```html
<style>
  body { font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif; font-size: 14px; color: #111; padding: 32px; max-width: 800px; margin: 0 auto; }
  h1 { font-size: 20px; font-weight: 600; margin: 0 0 4px; }
  h2 { font-size: 15px; font-weight: 600; margin: 32px 0 8px; border-top: 1px solid #e0e0e0; padding-top: 20px; }
  p { line-height: 1.6; margin: 0 0 12px; color: #444; }
  table { width: 100%; border-collapse: collapse; font-size: 13px; margin-bottom: 16px; }
  th { text-align: left; font-size: 11px; text-transform: uppercase; letter-spacing: 0.05em; color: #666; border-bottom: 1px solid #e0e0e0; padding: 0 12px 6px 0; }
  td { padding: 7px 12px 7px 0; border-bottom: 0.5px solid #f0f0f0; }
  h2 + p { margin-top: 0; }
</style>
```

`page-break-inside: avoid` on each `<h2>` section.

**Handing off the finished document.** Write the finished HTML to the workspace as `segment-analysis-[domain]-[date].html`, then state the **filename** and tell them it appears in this session's file list and syncs to OneDrive under `Documents/Cowork`. Don't invent a filesystem path — this host doesn't give you one. This host has no bundled `pdf`-rendering skill, so hand off the HTML file itself rather than a rendered PDF — tell the user plainly that it's HTML, not a PDF, and that they can open it in a browser and print to PDF (Ctrl/Cmd+P → Save as PDF) if they need that exact format. Don't claim a PDF was produced.
