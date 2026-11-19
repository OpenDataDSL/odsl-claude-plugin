# House style and data access for insight and extension pages

The OpenDataDSL portal's own insights and extensions follow this style. Pages that follow it look like part of the portal and stay fast on large data.

## Includes

| Page type | Includes, in this order |
|---|---|
| Insight outside an extension | `{{> #odsl-portal }}`, `{{> #insight-lib }}`, `{{> #insight-report-style }}` |
| Extension view or extension insight | `{{> #odsl-portal}}`, `{{> #odsl-service}}`, `{{> #insight-report-style}}`, the extension's `.style` partial, the extension's `.common` partial |

- `#odsl-portal`: icons and the portal theme.
- `#insight-lib`: the data API plus `ReportProperties()` (the report's properties, such as `ID`, `DSID` and `ONDATE` for item insights) and `changeRangeHelper(range, includeTime, callback)` for preset date ranges.
- `#odsl-service`: the data API for extension pages. Add `#odsl-signalr` for live updates.
- `#insight-report-style`: the shared CSS (a reference copy is in `insight-report-style.md`). Never paste or redefine it; put small overrides in a `<style>` block after the include, with a one-line comment saying why.

Partials whose names start with `#` are the platform's own. Your partials are included by their full script name.

## Page structure

1. `.controls-container` with an `h2` title (`{{name}}` gives the report's name) and the filters in `.controls` / `.controls-left`, each in a `.filter-picker` with a `label` and a `select` or `input` (`date`, `datetime-local`, `text`).
2. `.top-bar` at the TOP of the page holding `.kpi-cards` (`.kpi-card` with `.kpi-label` and `.kpi-value`; modifiers `primary`, `positive`, `negative`, `watch`) and the `.pager-row` (`.pager-summary` and First / Prev / Next / Last buttons).
3. `.tab-bar` with `.tab-btn` buttons (`.active` on the current one), switched by a `showTab(name)` function. Usual tabs: **Overview** (KPIs and a couple of top-5 tables), the **list** (the paged table), one or more **breakdowns** by the data's natural dimensions, and one tab specific to what is most useful about this data.
4. Content in `.section` or `.card-block` panels with an `h3`.

Other classes:

| Class | Use |
|---|---|
| plain `table` | Styled automatically |
| `num` | Numeric cells (right aligned) |
| `id` | Technical identifiers (monospace) |
| `message-cell` | Long free text that should wrap |
| `scroll-table-wrap` | A long table with a sticky header |
| `matrix-wrap` | A pivot-style grid (`td.positive`, `td.negative`, `td.neutral`) |
| `row-selectable`, `row-selected` | A master row that drives a detail panel |
| `badge` with `pos`, `neg`, `watch` | Statuses |
| `pos`, `neg`, `zero` | Coloured numbers |
| `loading-note`, `as-of`, `sub-title` | Notes and captions |
| `load-toggle` (`disabled`) | A link-style button, such as Export (CSV) |
| `accordion-item`, `accordion-header`, `accordion-body` (`open`) | Expandable detail |
| `debug-details`, `debug-status-box` | An optional collapsed debug log |

Palette (CSS variables): `--primary #1B4F8A`, `--secondary #2E75B6`, `--bg #f7f9fb`, `--card-bg #ffffff`, `--card-border #D6E4F7`, `--muted-bg #f4f6f8`, `--muted-border #e0e3e7`, `--text #1A1A1A`, `--text-muted #666666`, `--positive #1a7f37`, `--negative #c0392b`, `--watch #b8860b`. Use the variables, not fixed colours. Prefix your own classes with a short code for the extension.

## The data API

Every request runs as the signed-in user, so the platform's policies apply. All functions return promises.

| Function | Use |
|---|---|
| `find({service, source, filter, projection, sort, search, paging, queryParams})` | Lists. Prefer this: it sends `sort` to the server. |
| `ODSLlist({service, source, filter, projection, paging, queryParams})` | Older list call. It does NOT send `sort`; use `queryParams: {_sort: JSON.stringify({field: -1})}` if you use it. |
| `ODSLGet({service, source, id, projection, range, params})` | One record |
| `ODSLaggregate({service, source, pipeline})` | Server-side aggregation (no row limit) |
| `ODSLUpdate({service, source, body, params})` | Save a record (POST of the whole object) |
| `ODSLDelete({service, source, id})` | Delete a record |
| `distinct(service, source, filter, field)` | Distinct values of a field, for filter drop-downs |
| `new Paging()`, `UnlimitedPaging()` | Paging (see below) |

**`source`**: services that hold private and public data take `source: 'private'`, `'public'` or `'common'`. Single-source services take no `source`, for example `automation`, `automationlog`, `metric`, `policy`, `group`, `user`, `aiusage`.

**Paging**: `const paging = new Paging(); paging.limit = 50;` then give `paging` to `find`. After the call, `paging.items` is the total count (`-2` when the server did not count it), `paging.summary` is the "Results: 1-50 (120)" text, and `firstPage()`, `prevPage()`, `nextPage()`, `lastPage()` (each returns true when it moved), `hasPreviousPage()` and `hasNextPage()` drive the pager. Reset to the first page when a filter changes (`paging.page = 1; paging.skip = 0;`).

**Rules that keep pages correct and fast:**

- Page long lists; never pull everything into the browser to show one page.
- Sort on the server. Sorting the current page in the browser gives the wrong order across pages.
- Build KPIs and breakdowns with `ODSLaggregate` and plain stages (`$match`, `$group`, `$sum`, `$avg`, `$min`, `$max`, `$addToSet`, `$sort`, `$limit`, `$project`, `$count`). To count distinct values, group twice. Avoid `$cond`, `$ifNull`, `$objectToArray` and `$unwind` unless you have tested them on the service: run two simple pipelines instead (for example "all" and "failed" counts).
- Date filters: the platform turns date strings in filters into dates, so write `{timestamp: {$gte: '2026-10-01T00:00:00'}}`.
- Text that looks like a number or a date in an aggregation `$match` (for example `"2026M09"` or `"85"`) can be converted before matching and then match nothing. Prefix it with `string@` (`"string@2026M09"`) to keep it as text.
- Project only the fields you show.
- Delivery data for today has usually not loaded yet; end default date ranges at yesterday for delivery reports.
- CSV export of a paged list: fetch again with `UnlimitedPaging()` and build the CSV from the full result, not from the rendered page.
- Show the reason when a request fails: the error thrown by the data API includes the platform's reason. Show it in the page, not just the console.
- Escape every value from a record before inserting it as HTML (`textContent`, or an `escapeHtml` helper).
- Searching data ids: the data service text search does not find curves. Filter on `_type` instead: curves are `VarCurveDates`, `VarSmartCurve`, `VarEventCurve`, `VarComposedCurve`; timeseries are `VarTimeSeries`, `VarEventTimeSeries`, `VarSmartTimeSeries`. A `VarCurve` is one stored curve for one ondate, not a curve to offer in a picker.

## Mustache rules

The platform renders every page with Mustache before the browser sees it.

1. **Never write two opening braces in a row anywhere except a real Mustache tag**: not in an HTML comment, not in JavaScript, not in a JavaScript string. Mustache processes them wherever they are. Build such text from single characters (`'{' + '{'`).
2. **Document a template with `{{! ... }}` comments**, never HTML comments containing tags (an include tag inside an HTML comment still includes the partial, and a partial that mentions its own include tag includes itself until it fails).
3. **Report properties in JavaScript**: use `ReportProperties()` (from `#insight-lib`), or in an extension page with `function viewProps() { return {{#json}}{{properties}}{{/json}}; }` written in the page itself.
4. **Never nest a section inside itself** (`{{#items}}...{{#items}}`): inside the section the context falls back to the outer list and repeats it. Use one `{{#items}}...{{/items}}` and a sibling `{{^items}}...{{/items}}` for the empty case.
5. **Keys that are data cannot be looped.** Mustache only reads fixed field names; reshape the data into a list of `{name, value}` objects first.
6. **Server-rendered templates show integers as `8.0`.** In report and automation output templates, put counts and display values into the data as text, and branch on lists or booleans, not on a number being zero.
7. **Dot paths work without opening a section** (`{{created.$date}}`); inside `{{#created}}...{{/created}}` refer to the fields directly (`{{$date}}`).
8. In a CSV or JSON output template, put a doc comment at the start of the first content line: a comment on its own line leaves a blank first line.

## Email templates

Emails (automation output, scheduled report digests) are a different medium:

- Self-contained: no partials and no JavaScript.
- `<table>` layout with an inline `style="..."` on every element. A `<style>` block in `<head>` is only a bonus, since some email clients strip it. No flexbox, grid or CSS variables.
- Use the same palette by hand so emails look like the portal.
- No conditional colouring by numeric thresholds: ask for a precomputed field (for example `severity: "negative"`) in the data instead.
- Doc comments with `{{! ... }}` only.

## Checks before delivering a page

- Keep the file ASCII (no curly quotes, dashes or other non-ASCII characters), as the portal's own pages are.
- The `<script>` block has no syntax errors (`node --check` on the extracted script).
- Every `getElementById("x")` has a matching `id="x"`.
- `<style>` and `<script>` tags balance.
- No two opening braces outside real Mustache tags.
- Every record value shown is escaped.
- If possible, run the script with stubbed `find` / `ODSLaggregate` / `Paging` and a sample record (for example in Node with jsdom) to check the filters it sends and the values it shows.
