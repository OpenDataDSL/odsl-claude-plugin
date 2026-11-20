---
name: odsl-scripting
description: Write, explain, fix and review OpenDataDSL (ODSL) scripts for the OpenDataDSL data platform, including functions, processes, transformers, workflows, automations and reading or saving timeseries, curves, events and master data. Use whenever the user mentions ODSL, OpenDataDSL, .odsl files, active variables like ${data:...}, or wants to automate something on their OpenDataDSL tenant.
---

# Writing ODSL scripts

OpenDataDSL (ODSL) is the scripting language of the OpenDataDSL platform, a data management platform for energy and commodity market data. It is a simple BASIC-like language with built-in types for timeseries, forward curves, calendars and events, and "active variables" (`${service:...}`) that read from and write to the user's platform tenant.

This skill helps you write correct ODSL the first time. ODSL looks familiar but differs from JavaScript and Python in ways that cause silent failures, so do not write it from general intuition. Check syntax in `references/` and always read `references/practical-notes.md` before handing a script over.

## Workflow

1. **Pin down what the script is for.** A one-off script, a function library others import, a scheduled process, a transformer for loading data, a workflow, or an automation all have different shapes. Ask if it is unclear.
2. **Use real ids.** Never invent data ids, curve ids, calendar names or dataset ids. If the OpenDataDSL Fusion AI tools are connected (tools such as `find_data`, `find_curves`, `find_timeseries`, `find_calendar`, `find_script`), use them to find the real ids. If they are not, ask the user for the ids, or write the script with clearly named placeholders and say so.
3. **Look up anything you are not sure of.** Grep the reference files (see the map below) for the command, function or type. Function names and argument order matter: for example `blackScholesCall(forward, strike, volatility, years)` takes volatility before time.
4. **Write the script** following the conventions below.
5. **Check it.** Go through `references/practical-notes.md` against the script. If the `validate_script` tool is available, run it and fix every error it reports, because the engine prints parse errors and still runs the part that parsed.
6. **Save only when asked.** With the Fusion AI tools, `save_script` stores a script and `create_process_for_script` schedules it. Both change the user's tenant, so confirm the name, folder and schedule with the user first.

## Language essentials

```js
/**
 * @category analytics
 * Short description of what this script does
 */

// Comments are // or /* */. Statements end at the line end, no semicolons.
name = "TTF"                       // text, numbers, true/false, Date("2026-10-01")
prices = [10, 20, 30]              // lists: prices.add(40), prices[0]
o = SimpleObject()                 // a free-form object
o.region = "NW Europe"

// Control flow: blocks close with end, for loops close with next
if price > 100
    label = "high"
elseif price > 50
    label = "medium"
else
    label = "low"
end

for i = 1 to 10 step 2
    print i
next

for p in prices
    log info "Price " + p
next

while x < 5
    x = x + 1
end

label = price > 100 ? "high" : "normal"   // only as a whole assignment
```

### Functions

Return a value with `return value`, or by assigning to the function's own name. Prefix your function names so they never clash with a built-in (a built-in with the same name silently wins).

```js
/**
 * Average of a timeseries over the last n values
 * @param ts The input timeseries
 * @param n Number of values
 */
function myLastAverage(ts, n)
    myLastAverage = mean(ts.last(n))
end
```

Parameters are copies unless declared `byref`. Use `byref` when the function must change the caller's object, and for reports, curves and timeseries whose details must be kept.

Put shared functions in their own script and `import ${script:"my-functions"}`. Import loads only the functions (no top-level statements run) and does not follow that script's own imports.

### Active variables: reading, finding, saving

```js
ts = ${data:"MY.OBJECT:PRICE"}                       // private item
pub = ${data:"#ECB_FX.EURZAR:SPOT"}                   // ids starting with # are public
recent = ${data:"#ECB_FX.EURZAR:SPOT"} for last(10)   // also: for yesterday, for from("2026-01-01"), for between("2026-01-01", yesterday)
curve = ${data:"MY.OBJECT:CURVE:2026-10-23"}          // a forward curve for one ondate
ondates = ${data:"MY.OBJECT:CURVE"}                   // the curve's ondates

items = find ${object} where dataset == "ARGUS_DEL" order by name asc
for it in items
    print it.name
next

save o log "Reason recorded in the audit log"
```

`find` supports `top n`, `unique field`, `where` conditions (`==`, `!=`, `<`, `>`, `in`, `nin`, `like`, `and`, `or`), `order by` and `project`. Iterate the result rather than relying on its `size()`.

### Errors and logging

```js
on error ignore        // failing statements set the variable `error` and the script goes on
on error exit          // the default: stop with the error
log info "message"     // also debug, warn, fatal
print "message"
fail "message"         // fail the run (processes and workflows)
```

`on error ignore` applies only in the function (or script) that runs it: a called function starts with `on error exit`, and its error is handled by the calling statement.

### Processes

A script run as a process gets its inputs in `input` and has no `#ONDATE`: use `${date:"today"}`. The value of the last statement is the result: `"failed"` marks the run failed, `fail "message"` fails it with a message, anything else completes it.

## Reference map

All reference files are generated from the public OpenDataDSL documentation. They are long, so grep them rather than reading them whole.

| File | Use it for |
|---|---|
| `references/practical-notes.md` | Behaviours that commonly break scripts. Read before every delivery. |
| `references/commands.md` | Syntax of every command: find, save, delete, for, if, function, import, transform, workflow, aggregate, set, send |
| `references/functions.md` | Every built-in function A to Z. Grep `### name(` |
| `references/types.md` | Properties and methods of each variable type (TimeSeries, Curve, Calendar, Date, Matrix ...) |
| `references/variables.md` | Guides and examples for each variable type, including smart, event and composed curves and timeseries |
| `references/services.md` | Every `${service:...}`: data, object, curve, calendar, script, process, report, automation, file readers (CSV, XLS, JSON, XML, FTP, S3, Azure Blob, mail) |
| `references/calendars.md` | Calendars, expiry calendars, period codes and timeseries ranges |
| `references/data-management.md` | Transformers, workflows and processes, types and data modelling, versioning, events, queues, monitoring |
| `references/automations.md` | Automations and every automation target, with ODSL and REST examples |
| `references/aggregation-stages.md` | Stages for `aggregate` and aggregation pipelines |
| `references/glossary.md` | Keyword cheat sheet |

Full documentation: https://doc.opendatadsl.com/docs/odsl

## Style for delivered scripts

- A `/** ... @category ... */` header saying what the script does.
- Short, descriptive variable names; one idea per line, since conditions and concatenations cannot run over two lines.
- `log info` lines at the key steps of anything that runs unattended, so the process log shows what happened.
- Tell the user what each placeholder id is and where to find the real one.
