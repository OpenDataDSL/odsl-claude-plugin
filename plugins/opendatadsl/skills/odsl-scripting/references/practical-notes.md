# Practical notes: ODSL behaviours that break scripts

Check every script against this list before handing it over. Each item was confirmed against the ODSL engine.

## Functions

- **Return by assigning to the function name.** In a function that calls another of your functions, do not use `return`: it can end the calling function too, silently. Write `myTotal = result` instead of `return result`. A plain `return` in a function that calls no other user function is fine.
- **A function with the same name as a built-in is never called.** The built-in wins with no error. Prefix your own function names (`myClean`, `rptBuild`).
- **Parameters are copies unless declared `byref`.** Changing an object or list parameter changes a copy. Declare `byref` to change the caller's value, and for reports (a report read with `${report:id}` loses its `data` when passed as a copy) and timeseries (a copy loses its `timezone`).
- **Functions have their own scope.** A variable set in a called function does not change the caller's variable.
- **`import` loads functions only.** It runs none of the imported script's top-level statements and does not follow that script's own imports: if A imports B and B imports C, A must import C itself.

## Syntax

- **No line continuation.** A condition or concatenation cannot continue on the next line (a trailing `+`, `and` or `or` is a syntax error). Build long text in steps: `msg = "a"` then `msg = msg + "b"`.
- **No method call on a call result.** `f(x).value` and `BusinessCalendar().getDates(...)` do not parse. Assign first: `r = f(x)` then `r.value`.
- **Variables first set inside an `if` block are local to it.** Set the variable before the block.
- **Reserved words cannot be names.** `default`, `use`, `call` and other keywords break the parse as variable, parameter or field names. Use `o.set("default", x)` for such a field. Identifiers cannot start with `_`.
- **Negation.** `!` only applies to a variable (`!found`). There is no `not (...)`; write the opposite comparison or `x == false`.
- **No `%`.** Use the `mod` function.
- **Strings have no escapes**, and a string cannot end with a backslash (it swallows the rest of the line). For a backslash in a regular expression use `"[\\]"`. Text with a line break can be a literal string spanning lines.
- **Mixed arithmetic: use brackets.** Write `(a / b) * c`, not `a / b * c`, so the result does not depend on the engine version.
- **Conditions are not general expressions.** `x = a == b` works, but `x = a and b` does not, and `cond ? a : b` only works as a whole assignment.

## Values and comparisons

- **`1 == 1.0` is false.** A number read back as `85.0` never equals `85`. Compare with a tolerance or keep exact values as text.
- **Numbers display with 6 decimals.** `"" + 32.15` is `32.150000`. Format before building messages. Avoid `set precision` in shared scripts: it changes the setting for everything running in the same engine.
- **Text does not order.** `"2026-10-01" < "2026-09-30"` fails. Compare `Date` values (`Date("2026-10-01")`), which support `<`, `<=`, `+` and `-` days.
- **`x.id` reads the object's own id, not a field called `id`.** For a parameter or loop variable that is its variable name. Read a field called `id` with `x.get("id")`.
- **Numeric-looking text turns into a number** with `obj.set(name, "007")` and with secret values. Assign with `obj.x = "007"` or keep such values in a list.
- **Concatenating null gives the text `{null}`.** Check for null first.
- **Reading a property of null does not raise an error.** Check the value you got.
- **Lists** have no `contains`, `indexOf` or `sort`. Use a list filter (`items[price > 80]`, one condition per filter, filter again for more) or loop. `keys(obj)` returns the keys sorted.
- **Text functions:** `left(s, n)` errors when the text is shorter than `n`; `substring(s, from, to)` needs all three arguments. `length`, `lower`, `upper`, `trim`, `split`, `contains`, `replaceAll` (Java regex) and `indexOf` on text all exist. There is no JSON parse function.

## Error handling

- **`on error ignore`** sets the variable `error` to the message and carries on. The failing assignment does not happen, so reset the target variable first.
- **It leaks.** After a function runs `on error ignore`, errors in functions it calls later can also be ignored, so a failed step can look successful. Put `on error ignore` and `on error exit` in a small helper that does one risky thing and calls no other function of yours after it, and have it return both the value and the error.
- **Parse errors do not stop the script.** The engine prints `line n:m ...` and runs what it parsed. Validate the script (the `validate_script` tool, or the VS Code extension) and fix every reported error.

## Finding and reading data

- **Iterate `find` results** instead of testing `size()`, which can be 0 or -2 for results that do have rows.
- **`order by` is two words**: `order by eventtime asc, eventid asc`.
- **`find unique` can repeat values** on some platform versions. Add `"_count=true"` to the service options, e.g. `${data:"private","_raw=true","_count=true"}`.
- **Curve ids:** `OBJ:CURVE` returns the list of ondates the curve has (or can be built for); `OBJ:CURVE:2026-10-23` is the curve for that ondate. Use the ondates list to find the latest ondate rather than searching for stored curve ids, because smart and event curves are built on demand.
- **Curve functions that change their input:** `normaliseCurve`, `extendCurve` and `forwardFillCurve` change the curve they are given, even a clone. Read the curve again if you need the original.
- **Gaps:** `fill(ts, "linear")` works; `"previous"` is not supported.
- **Classify data** with `typeOf`: `contains(typeOf(v), "TimeSeries")`, `contains(typeOf(v), "Curve")`.

## Saving

- **Objects merge by default.** `save o` merges into the stored object; add `replace` to replace it.
- **Event ids:** `Event(date)` makes its id from the start time only, so several events at the same time overwrite each other. Set the id yourself: `e.id = "ID_" + format(d, "yyyyMMdd") + "_" + ref`.
- **Saving events:** build them in a list on an object (`o = Object()`, `o.id = "MY.OBJECT"`, `o.TRADES = []`, `o.TRADES.add(e)`, `save o`).

## Secrets

- **Never put a key, token or password in a script.** Store it as a platform secret and read it in the script through the `secret` active variable service, written like any other service read: the service name `secret`, a colon and the secret's name in quotes. Use the secret's `.value` where the credential is needed, for example in a URL parameter or an `AccountKey` option.
- **Printing a secret shows only stars**, so a script can log its steps safely.
- **Ask the user for the secret's name, never its value.** They create secrets themselves in the portal or with the `Secret` variable type. Do not ask for, repeat or store a credential in the conversation.
- Secret names may use `_` and `-`; in a company's own Azure Key Vault a `_` in the name is read as `-`.

## Processes and automations

- **Process scripts have no `#ONDATE`.** Use `${date:"today"}`. Inputs arrive in `input`. The last value is the result: `"failed"` fails the run, `fail "message"` fails it with a message.
- **Email automations:** set `attachment = false` for a plain email. When it is missing, the data is sent as an attachment.
- **Automation templates must match an offer exactly.** The portal lists an automation only when its template text is exactly one the platform offers for that item. Copy it from `find_automation_target` or the portal rather than writing it by hand.
- **Build automations** for smart and event curves and timeseries are created by the platform when you save those items. Do not create them yourself.

## Mustache templates (automation transformers, report templates)

- **Numbers render as `8.0`.** Put counts and display values into the data as text, and branch on lists or booleans, not on numbers being zero.
- **Comments:** use `{{! ... }}`, never a `{{...}}` tag inside an HTML comment, which still gets substituted.
- **Do not nest a section inside itself** (`{{#items}}...{{#items}}`). Use `{{#items}}...{{/items}}` with a sibling `{{^items}}...{{/items}}` for the empty case.
