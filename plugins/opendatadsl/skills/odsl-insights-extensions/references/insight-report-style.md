# The shared house style CSS

A reference copy of the platform's `#insight-report-style` partial, for exact class names. Do not paste it into pages: include the partial and add only small overrides after it.

```css
/* The shared house style, included in pages with the #insight-report-style partial.
   Reference copy: do not paste it into pages. Include the partial and add only small overrides after it. */

/* =========================================================================
   OpenDataDSL house style for interactive Mustache/HTML business reports.
   Shared partial - include it (standard partial-include syntax, this
   file's own name) near the top of a report, alongside the odsl-portal
   and insight-lib includes, rather than pasting this block into every
   report file. Originally copied from
   odsl-report-style/assets/base.css and then inlined per-report; this is
   now the single source of truth for it, extracted so every report shares
   one copy and picks up fixes automatically.

   Don't redefine the palette or component patterns here - if a report
   needs something different for one of these classes, override it in
   that report's own  block (which comes after this include, so
   the normal CSS cascade lets a later same-selector rule win). Add new
   report-specific rules to that report's own  block, not here;
   only promote a rule to this shared file when it's genuinely reusable
   across reports.
   ========================================================================= */

:root {
  --primary: #1B4F8A;
  --secondary: #2E75B6;
  --bg: #f7f9fb;
  --card-bg: #ffffff;
  --card-border: #D6E4F7;      /* also doubles as the alt-row / accordion-header tint */
  --muted-bg: #f4f6f8;
  --muted-border: #e0e3e7;
  --text: #1A1A1A;
  --text-muted: #666666;
  --positive: #1a7f37;
  --negative: #c0392b;
  --watch: #b8860b;             /* third severity tier, e.g. "needs attention but not critical" */
}

* { box-sizing: border-box; }

body {
  font-family: 'Segoe UI', 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
  font-size: 13px;
  background: var(--bg);
  color: var(--text);
  margin: 20px;
}

h2, h3 { color: var(--primary); margin: 0; }

/* ---- Page header banner: dark-blue bar with white text. One per report. ---- */
.controls-container {
  background: var(--card-bg);
  border: 1px solid var(--card-border);
  border-radius: 8px;
  padding: 14px 18px;
  margin-bottom: 24px;
}
.controls-container h2 {
  font-size: 16px;
  background: var(--primary);
  color: #ffffff;
  margin: -14px -18px 12px -18px;
  padding: 12px 18px;
  border-radius: 8px 8px 0 0;
}
.controls {
  display: flex; justify-content: space-between; align-items: center;
  flex-wrap: wrap; gap: 12px;
}
.controls-left { display: flex; align-items: center; gap: 16px; flex-wrap: wrap; }
.controls-left label { font-size: 12px; color: var(--text-muted); margin-right: 6px; }
.controls-left select {
  font-size: 12px; padding: 4px 8px;
  border: 1px solid var(--muted-border); border-radius: 4px; background: var(--card-bg);
}

/* ---- Tab bar - underlines the active tab in the same primary blue used
   for section headings, rather than introducing a new accent color. ---- */
.tab-bar {
  display: flex;
  gap: 4px;
  border-bottom: 2px solid var(--card-border);
  margin-bottom: 20px;
  flex-wrap: wrap;
}
.tab-btn {
  font-family: inherit;
  font-size: 13px;
  font-weight: 600;
  color: var(--text-muted);
  background: none;
  border: none;
  border-bottom: 3px solid transparent;
  padding: 10px 16px;
  cursor: pointer;
  margin-bottom: -2px;
}
.tab-btn:hover { color: var(--primary); }
.tab-btn.active { color: var(--primary); border-bottom-color: var(--primary); }

/* ---- Ordinary section card headings: colored text with a strong bottom
   rule, not a filled banner. ---- */
.section, .card-block {
  background: var(--card-bg);
  border: 1px solid var(--card-border);
  border-radius: 8px;
  padding: 14px 16px;
  margin-bottom: 20px;
}
.section h3, .card-block h3 {
  font-size: 14px; font-weight: 600;
  border-bottom: 2px solid var(--primary);
  padding-bottom: 8px; margin-bottom: 12px;
}

#loadingIndicator { font-size: 12px; color: var(--text-muted); padding: 10px 0; }
.as-of { font-size: 12px; color: var(--text-muted); margin-top: 4px; font-weight: bold; }

.filter-picker { display: flex; align-items: center; gap: 8px; margin-top: 14px; flex-wrap: wrap; }
.filter-picker label {
  font-size: 11px; color: var(--text-muted); text-transform: uppercase;
  letter-spacing: 0.03em; font-weight: 600;
}
.filter-picker select, .filter-picker input[type="date"], .filter-picker input[type="datetime-local"], .filter-picker input[type="text"] {
  padding: 6px 10px; border-radius: 6px; border: 1px solid var(--muted-border);
  font-size: 13px; font-family: inherit; background: var(--card-bg); color: var(--text);
}
.filter-picker select { cursor: pointer; }
.filter-picker .field-spacer { width: 16px; }

.loading-note {
  color: var(--watch); font-style: italic;
  background: #FFF3CD; border: 1px solid #F0DFA0; border-radius: 8px;
  padding: 12px 16px; margin: 0;
}

/* ---- KPI cards, and the pager row that sits alongside them at the top
   of a paged tab (see .pager-row below). ---- */
.top-bar { display: flex; justify-content: space-between; align-items: center; gap: 16px; margin-bottom: 20px; flex-wrap: wrap; }
.kpi-cards { display: flex; gap: 16px; margin: 20px 0 30px 0; flex-wrap: wrap; }
.kpi-card {
  flex: 1 1 0; padding: 16px 20px; border-radius: 8px;
  background: var(--muted-bg); border: 1px solid var(--muted-border);
}
.kpi-card.primary { background: #eaf3ff; border-color: #b9d9ff; }
.kpi-label { font-size: 11px; color: var(--text-muted); text-transform: uppercase; letter-spacing: 0.03em; white-space: nowrap; min-width: 120px; }
.kpi-value { font-size: 1.8rem; font-weight: bold; margin-top: 4px; color: var(--text); white-space: nowrap; }
.kpi-card.positive .kpi-value { color: var(--positive); }
.kpi-card.negative .kpi-value { color: var(--negative); }
.kpi-card.watch .kpi-value { color: var(--watch); }

.accordion-item {
  background: var(--card-bg); border: 1px solid var(--card-border);
  border-radius: 8px; margin-bottom: 12px; overflow: hidden;
}
.accordion-header {
  display: flex; justify-content: space-between; align-items: center;
  padding: 12px 16px; cursor: pointer; background: var(--card-border);
}
.accordion-header:hover { background: #c3d9f0; }
.accordion-header .title-wrap { display: flex; align-items: center; }
.accordion-header .title { font-size: 14px; font-weight: bold; color: var(--primary); }

.accordion-toggle-icon {
  display: inline-block; width: 0; height: 0; margin-right: 10px;
  border-top: 5px solid transparent; border-bottom: 5px solid transparent;
  border-left: 6px solid var(--text-muted);
  transition: transform .15s ease; flex-shrink: 0;
}
.accordion-toggle-icon.open { transform: rotate(90deg); }

.accordion-body { display: none; padding: 16px; border-top: 1px solid var(--card-border); }
.accordion-body.open { display: block; }

/* ---- Status badges. Always paired with a title="" tooltip. ---- */
.badge {
  font-size: 12px; font-weight: bold; padding: 3px 10px; border-radius: 12px;
  background: var(--card-bg); border: 1px solid var(--muted-border); color: var(--text-muted);
}
.badge.pos { color: var(--positive); border-color: var(--positive); }
.badge.neg { color: var(--negative); border-color: var(--negative); }
.badge.watch { color: var(--watch); border-color: var(--watch); }
.badge-group { display: flex; gap: 8px; align-items: center; }

/* ---- Tables ---- */
table { width: 100%; border-collapse: collapse; font-size: 12px; }
th, td { border: 1px solid var(--card-border); padding: 5px 6px; text-align: center; }
thead th { background: var(--primary); color: #fff; font-weight: normal; }
tbody tr:nth-child(even) td { background: #f4f8fd; }

.sub-title {
  font-size: 11px; text-transform: uppercase; letter-spacing: .03em;
  color: var(--text-muted); margin-bottom: 6px;
}

.pos { color: var(--positive); }
.neg { color: var(--negative); }
.zero { color: var(--text-muted); }

.load-toggle {
  font-size: 12px; color: var(--secondary); cursor: pointer;
  border: 1px solid var(--secondary); border-radius: 4px;
  padding: 4px 10px; display: inline-block; background: var(--card-bg);
  user-select: none;
}
.load-toggle:hover { background: #eaf3ff; }
.load-toggle-row { display: flex; gap: 8px; flex-wrap: wrap; }
.load-toggle.disabled { opacity: 0.4; cursor: default; pointer-events: none; }

/* ---- Server-side pager, driven by the #odsl-service Paging class -
   typically placed inside .top-bar alongside a .kpi-cards block. ---- */
.pager-row { display: flex; align-items: center; gap: 8px; flex-wrap: wrap; }
.pager-summary { font-size: 12px; color: var(--text-muted); margin: 0 4px; }

.matrix-wrap { overflow-x: auto; }
.matrix-wrap thead th {
  padding: 6px 4px; font-size: 11px;
  white-space: nowrap; overflow: hidden; text-overflow: ellipsis;
}
.matrix-wrap table { table-layout: fixed; }
.matrix-wrap th:first-child, .matrix-wrap td:first-child {
  width: 140px; text-align: left; padding-left: 8px; background: var(--muted-bg); color: var(--text);
}
.matrix-wrap td.positive { color: var(--positive); font-weight: bold; }
.matrix-wrap td.negative { color: var(--negative); font-weight: bold; }
.matrix-wrap td.neutral { color: var(--text-muted); }

/* ---- Scrollable table wrapper with a sticky header - used by most
   reports for their main data table(s) instead of loading everything
   unbounded onto the page. ---- */
.scroll-table-wrap {
  max-height: 420px;
  overflow-y: auto;
  overflow-x: auto;
  border: 1px solid var(--card-border);
  border-radius: 8px;
}
.scroll-table-wrap table { border: none; }
.scroll-table-wrap thead th { position: sticky; top: 0; z-index: 1; }

/* Technical identifiers (ids, keys, paths) - monospace so they stay easy
   to scan. */
.id { font-family: "Courier New", monospace; font-weight: 600; color: var(--primary); }

/* Free-text values (descriptions, messages, change values, names) can
   run long - left-align and let them wrap rather than forcing every row
   to the width of the longest value. */
.message-cell { text-align: left; white-space: normal; word-break: break-word; }

/* Numeric columns - right align rather than centering, so a column of
   numbers stays easy to visually compare. */
.num { text-align: right; }

/* Clickable/selectable table rows, e.g. a master list driving a detail
   panel below it. */
.row-selectable { cursor: pointer; }
.row-selectable:hover td { background: #eaf3ff; }
.row-selected td { background: var(--card-border) !important; font-weight: 600; }

.debug-details { margin-top: 20px; }
.debug-details summary { cursor: pointer; color: var(--text-muted); font-size: 13px; }
.debug-status-box {
  background: #EEF4FA; border: 1px solid #C7DAEC; border-radius: 8px;
  padding: 10px 16px; margin-top: 10px; font-size: 12px;
  font-family: "Courier New", monospace;
}
.debug-status-box div { padding: 2px 0; }
.debug-raw { display: flex; gap: 16px; flex-wrap: wrap; margin-top: 12px; }
.debug-raw > div { flex: 1; min-width: 260px; }
.debug-raw pre {
  background: #FAFBFD; border: 1px solid var(--card-border); border-radius: 8px;
  padding: 10px; font-size: 11px; max-height: 260px; overflow: auto;
  white-space: pre-wrap; word-break: break-word;
}

@media (max-width: 1000px) {
  .spread-top, .flex-row { flex-direction: column; }
}
```
