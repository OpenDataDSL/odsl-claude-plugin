---
name: odsl-insights-extensions
description: Build OpenDataDSL insight reports and portal extensions, meaning interactive Mustache/HTML/JavaScript report pages, item, service and Excel insights, extension views, manifests (create-extension.odsl), feature and data policies, and publishing. Use when the user wants a new report page, dashboard-style insight, custom portal screen or extension on the OpenDataDSL platform, or wants to fix or restyle one.
---

# Building OpenDataDSL insights and extensions

An **insight** is an interactive report page: a Mustache script of HTML, CSS and JavaScript that the platform renders and shows in the portal, reading data through a JavaScript data API as the signed-in user. An **extension** packages one or more such pages as portal screens (views), together with its insights, policies and a manifest, so it can be installed in other environments or tenants.

Both use the same page style. Read `references/house-style.md` before writing any page, and start from `references/example-insight.mustache`, a complete working page.

## First, decide what is being built

| The user wants | Build | Registered with |
|---|---|---|
| A report in the portal's report list | A general insight | `InsightReport()` with a category of your choice |
| A report on a selected item (a curve, dataset, process, object ...) | An item insight | `InsightReport()` with the screen's category, e.g. `Curve Management Insights` (table in `references/insights.md`); the item arrives in `ReportProperties()` as `ID`, `DSID`, `ONDATE` ... |
| Data returned into Excel | An Excel insight | `InsightReport()` with category `Excel Insights` (see `references/insights.md`) |
| A new screen or tabs in the portal menu, possibly with its own insights | An extension | `FeaturePolicy()`, `InsightReport()`s, `Extension()` with views and resources (see `references/building-extensions.md`) |
| A report built on a schedule and stored | A Default or Fixed report plus a template | See `references/reports.md` |

Ask if it is unclear. Also ask, or find out, **which data the page reads**: the service, and the real field names. With the OpenDataDSL Fusion AI tools connected, look at real records (`find_types`, `find_data`, `find_curves`, `find_audit_records`, `find_automation_logs` and so on) before writing the page, so the filters and columns use field names that exist. Never guess field names silently; say which ones you assumed.

## Workflow

1. **Plan the page**: the filters in the header, the KPI cards, the tabs (Overview, the paged list, breakdowns, one tab specific to this data), and which requests feed each part (a paged `find` for the list, `ODSLaggregate` for KPIs and breakdowns).
2. **Write the page** from `references/example-insight.mustache`, following `references/house-style.md`. For an extension, also write its `.style` and `.common` partials and every view page.
3. **Write the registration**: the `InsightReport()` for each insight, and for an extension the full `create-extension.odsl` (feature policy, data policies if needed, insights, extension, views, resources). The manifest example in `references/building-extensions.md` is the template.
4. **Check** every page with the list at the end of `references/house-style.md`. If the `validate_script` tool is available, run it on every page and script.
5. **Save only when the user asks.** With the Fusion AI tools: `save_script` for pages (scriptType `mustache`), `create_report` for insight reports, and, where available, `create_policy` and `create_extension`. These change the user's tenant: confirm names, script paths and who can see it first. Otherwise hand over the files and the `create-extension.odsl` script for the user to run in VS Code with the OpenDataDSL extension.

Fusion AI in the portal also has a **Build an extension** playbook (`#extension-builder`) that plans, writes and saves a whole extension after an approval. Mention it when the user would rather do it in the portal.

## Rules that matter most

- **Two opening braces only in real Mustache tags.** Not in HTML comments, JavaScript or JavaScript strings. Document templates with `{{! ... }}`.
- **Server-side sort, paging and aggregation.** Use `find` with `sort` and a `Paging`; never sort one page in the browser. KPIs and breakdowns come from `ODSLaggregate` with plain stages.
- **KPI cards and the pager go at the top**, in a `.top-bar`, not below the table.
- **Escape every record value** inserted as HTML.
- **Show the failure reason** from a failed request in the page.
- **Include the shared style, never copy it.** `#insight-report-style` after the platform partials; your own overrides after it.
- **Extensions: list everything in resources.** Every page, partial and insight page in `e.resources.script`, every insight id in `e.resources.reportconfig`, every policy in `e.resources.policy`. Anything missing is not installed elsewhere.
- **Extension menu views receive the extension id in `properties.ID`**, not a selected item. Pass inputs to a view as other properties.
- **Views and resources need the extension saved again**; a changed page script takes effect as soon as it is saved.

## Naming conventions

| Thing | Convention | Example |
|---|---|---|
| Extension id | `publisher.code` | `acme.positions` |
| Feature policy | `extension.` + extension id | `extension.acme.positions` |
| Feature | extension id + view, or one for all views | `acme.positions.main` |
| Page scripts | a folder per extension, named by extension id and view | `acme\extensions\positions\acme.positions.limits` |
| Shared partials | extension id + `.style`, `.common` | `acme.positions.common` |
| Insight id and category | extension id + `.insight.` + name; category = extension id | `acme.positions.insight.history` |
| Standalone insight scripts | `insight-` + a descriptive name | `insight-curve-coverage` |

Script paths in ODSL strings use single backslashes. Icons are Bootstrap icon names, optionally with a colour class (`"clock-history text-primary"`).

## Reference map

| File | Use it for |
|---|---|
| `references/house-style.md` | Includes, page structure, CSS classes, palette, the data API, paging, data-access rules, Mustache rules, email templates, checks. Read first. |
| `references/example-insight.mustache` | A complete, tested insight page: filters, KPIs, pager, tabs, server-side sort and aggregation, CSV export |
| `references/insight-report-style.css` | The shared CSS, for exact class names (include the partial; do not copy it) |
| `references/building-extensions.md` | How extensions work, the manifest, views and menu groups, writing pages, insights and policies, publishing, the Fusion AI playbook, an HTML components example |
| `references/insights.md` | Insight types, the item insight categories and the properties each screen passes, Excel insights, the older table and chart templates |
| `references/reports.md` | Report configurations (Default, Fixed, Dynamic, Insight, Widget), schedules and the report services |

For ODSL itself (the `create-extension.odsl` script, report functions, process scripts), use the `odsl-scripting` skill.

Full documentation: https://doc.opendatadsl.com/docs/extensions/building/manifest
