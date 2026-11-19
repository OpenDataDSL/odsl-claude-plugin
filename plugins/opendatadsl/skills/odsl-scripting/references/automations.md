# Automations and automation targets

What automations are, how to create them in ODSL, and every automation target.

Generated from the OpenDataDSL documentation (https://doc.opendatadsl.com/docs).


---

<!-- source: topics/automation/automate_basics.md -->

Automations in OpenDataDSL are **event-driven data flows**: when something happens in the platform, such as data being updated, a process failing or a dataset becoming late, an automation runs an action automatically.

## What are automations?

An automation watches one or more items and listens for specific actions on them. When a matching action happens, the automation sends the item's data to an **automation target**, which does the work: send an email, publish to a queue, run a process, build a curve and so on.

This makes automations the main way to build reactive data pipelines without polling or extra scheduling.

You can create and manage automations in three ways:

- In the portal, from the **Automations** panel of an item, or with the [Automations extension](https://doc.opendatadsl.com/docs/extensions/automations/overview)
- In ODSL, using the [Automation](https://doc.opendatadsl.com/docs/odsl/variable/automation) type and the `automation` service
- With the [Automation REST API](https://doc.opendatadsl.com/docs/api/rest/service/automation)

## Triggering events

Every automation is triggered by an **action** on an item in a **service**. The table below lists the services you will use most and the actions each one raises.

| Service | Actions |
|---|---|
| alert | raised, closed, reopened, agent, severity, comment |
| curve | create, update, delete, build, success, warning, failed |
| data | create, update, delete, validated, invalid |
| dataset | update, late, complete, missing, correction |
| extension | create, update, delete, late, complete, missing, install, upgrade, uninstall |
| group | create, update, delete, itemadded, itemremoved, itemupdated |
| process | create, update, delete, rollback, promote, tag, untag, rename, run, success, warning, failed |
| queue | create, update, delete, message |
| report | create, update, delete, run, success, warning, failed |
| script | create, update, delete, rollback, promote, tag, untag, rename, run, success, warning, failed |
| scriptlog | create |
| validation | create, update, delete, success, warning, failed |

Other services, for example `object`, `calendar`, `type`, `action`, `workflow`, `extractor`, `transformer`, `reportconfig`, `dashboard`, `documentation` and `policy`, raise `create`, `update` and `delete`.

A few points to know:

- **Timeseries, curves and event lists are all in the `data` service.** An event list is a data item with the id `OBJECT:EVENTLIST`, so automations on events use the `data` service.
- `validated` and `invalid` are only raised for curves and timeseries, by the [Validation](https://doc.opendatadsl.com/docs/extensions/validation/overview) extension.
- Dataset actions are raised by dataset monitoring with source `private`, using the dataset id (`provider.feed.product`) as the item id.
- A `create` condition always uses the id `*`, because the item does not exist yet.

## Anatomy of an automation

An automation is stored as a JSON document in the `automation` service. Here is an example that emails the operations team when a dataset is late:

```json
{
  "target": "odsl.email",
  "template": "When 'ICE.NDEX.NLB' is late, send an email to [to] with subject [subject]",
  "icon": "envelope-at text-outlook",
  "enabled": true,
  "conditions": [
    {
      "service": "dataset",
      "action": "late",
      "id": "ICE.NDEX.NLB",
      "source": "private"
    }
  ],
  "properties": {
    "to": "ops-team@mycompany.com",
    "subject": "ICE Endex NLB settlement prices are late",
    "attachment": false
  }
}
```

### Fields

#### `target`
The id of the [automation target](https://doc.opendatadsl.com/docs/topics/automations/targets) that receives the data and acts on it, for example `odsl.email`, `odsl.queue` or `odsl.process`. Target ids are `publisher.code`; all the targets provided by OpenDataDSL start with `odsl.`.

#### `template`
The sentence that describes the automation. It is required. The platform creates the automation's `name` from it by replacing each `[property]` with that property's value, so the example above is named:

*When 'ICE.NDEX.NLB' is late, send an email to 'ops-team@mycompany.com' with subject 'ICE Endex NLB settlement prices are late'*

The portal's Automations panel offers a fixed set of templates for each item, made from the action and the target's own template (see [Templates](#templates) below). Use exactly one of those texts, otherwise the automation still runs but is not shown in the item's Automations panel.

#### `icon`
The [Bootstrap Icons](https://icons.getbootstrap.com/) name shown next to the automation in the portal, usually the target's icon. You can add a colour class, for example `envelope-at text-outlook`.

#### `enabled`
Set to `false` to pause the automation without deleting it. The default is `true`.

#### `conditions`
A list of trigger conditions. The automation runs when **any** condition matches. Each condition has:

| Field | Required | Description |
|---|---|---|
| `service` | Yes | The service to watch, for example `dataset` |
| `action` | Yes | The action that triggers the automation, for example `late` |
| `id` | Yes | The id of the item to watch, or `*` for any item |
| `source` | Yes | The source of the item: `private`, `public` or `common` |
| `did` | No | A dependent id. When set, the target is sent the dependent item instead of the item that triggered it. The platform uses this to build an event curve when its event list is updated |

#### `properties`
The values the target needs, named after the inputs in the target's template, for example `to` and `subject` for an email. Some targets also need fixed properties: an email needs `attachment` set to `false` (or `true` to attach the data). Each target's page lists what it needs.

When the automation runs, the platform also adds these properties for the target:

| Property | Description |
|---|---|
| `id` | The id of the item that triggered the automation |
| `ondate` | The curve date of a curve, or the delivery date of a dataset |
| `range` | The range of observations of a timeseries update |

Properties starting with `@` change when and how the automation runs, see [Advanced features](https://doc.opendatadsl.com/docs/topics/automations/creating).

#### `name`, `owner`
These are set by the platform when the automation is saved. The **owner** is whoever saved the automation last: the automation runs with the owner's security policies, so the owner needs access to the item and to whatever the target does.

## Templates

The templates offered for an item are made like this:

| Kind | Template |
|---|---|
| Standard | `When '<id>' <action text>, <target template>` |
| With a transformer | `When '<id>' <action text>, transform the data using [@transformer], then <target template>` |
| Property change (update) | `When property [@propertyName] on '<id>' is set to [@propertyValue], <target template>` |
| Property set (create, delete) | `When 'anything' is created with property [@propertyName] set to [@propertyValue], <target template>` |
| Alerts | `When an alert <action text> for '<id>', <target template>` |

`<id>` is the item id, or `anything` for `*`. The action text is:

| Action | Text | Action | Text |
|---|---|---|---|
| create | is created | complete | is complete |
| update | is updated | correction | has a correction |
| delete | is deleted | success | is successful |
| rename | is renamed | warning | has a warning |
| rollback | is rolled back | failed | has failed |
| promote | is promoted | validated | passes validation (or its failure is overridden) |
| tag | is version tagged | invalid | fails validation |
| untag | is version un-tagged | itemadded | has an item added |
| late | is late | itemremoved | has an item removed |
| missing | is missing | itemupdated | has an item updated |

For alerts: raised (is raised), closed (is closed), reopened (is reopened), agent (is handed to a different agent), severity (changes severity) and comment (has a comment added, edited or deleted).

The transformer form is only offered for targets that allow transformation, and the property change form only for targets that allow property checks. The [targets page](https://doc.opendatadsl.com/docs/topics/automations/targets) lists both for every target.

:::tip Get the exact text from the platform
The `listAutomations` [special function](https://doc.opendatadsl.com/docs/api/rest/special_functions) returns the offers for an item, with the exact template, target, icon, conditions and default properties:

```
GET https://api.opendatadsl.com/api/dataset/v1/private/ICE.NDEX.NLB?_function=listAutomations
```
:::

## Automations created for you

Some automations are created and kept up to date by the platform or by extensions, and you do not normally create them yourself:

- **Build automations:** saving a Smart Curve, Event Curve, Smart TimeSeries or Event TimeSeries creates the automation that rebuilds it when its inputs change, and deleting the item removes it
- **Validation checks:** the Validation extension keeps an `odsl.validation` automation for each validation configuration
- **Compositions:** Composer keeps an `odsl.composition` automation for each composition

## Next steps

- [Creating Automations](https://doc.opendatadsl.com/docs/topics/automations/creating): create, change, test and delete automations in ODSL and the REST API
- [Automation Targets](https://doc.opendatadsl.com/docs/topics/automations/targets): the available targets and the properties each one needs
- [Automations extension](https://doc.opendatadsl.com/docs/extensions/automations/overview): manage and monitor automations in the portal


---

<!-- source: topics/automation/creating-automations.md -->

You can create automations in the portal, in ODSL scripts, or with the REST API. This guide shows common scenarios with worked examples in ODSL and the REST API. To create automations in the portal, see the [Automations extension](https://doc.opendatadsl.com/docs/extensions/automations/creating).

## Before you start

You need to know:

- The **service**, **source** and **id** of the item to watch, and the **action** that should trigger the automation: see the [trigger reference](https://doc.opendatadsl.com/docs/topics/automations/basics#triggering-events)
- The **target** to send the data to, and the properties it needs: see [Automation Targets](https://doc.opendatadsl.com/docs/topics/automations/targets)
- The **template** text for the automation: see [Templates](https://doc.opendatadsl.com/docs/topics/automations/basics#templates)

The automation runs with the security policies of the user who saved it, so that user needs access to the item and to whatever the target does.

## Two ways to create an automation in ODSL

**1. Set every field yourself.** Build an empty automation, then set its target, template, conditions and properties. This always works and is the way used in the rest of this guide.

```js
//#region Set every field
a = AutomationBuilder("dataset", "private", "ICE.NDEX.NLB").build()

c = SimpleObject()
c.service = "dataset"
c.source = "private"
c.id = "ICE.NDEX.NLB"
c.action = "late"
conds = []
conds.add(c)

a.conditions = conds
a.target = "odsl.email"
a.template = "When 'ICE.NDEX.NLB' is late, send an email to [to] with subject [subject]"
a.icon = "envelope-at text-outlook"
a.properties.to = "ops-team@mycompany.com"
a.properties.subject = "ICE Endex NLB settlement prices are late"
a.properties.attachment = false
a.enabled = true
save ${automation:a}
//#endregion
```

**2. Start from one of the item's offers.** `withAutomationTemplate` looks up the automations the item offers (the same list the portal shows) and copies the target, icon, conditions and default properties of the offer whose template matches the text exactly. You then only set the properties.

```js
//#region Start from an offer
ab = AutomationBuilder("dataset", "private", "ICE.NDEX.NLB")
ab.withAutomationTemplate("When 'ICE.NDEX.NLB' is late, send an email to [to] with subject [subject]")
ab.set("to", "ops-team@mycompany.com")
ab.set("subject", "ICE Endex NLB settlement prices are late")
a = ab.build()
save ${automation:a}
//#endregion
```

:::warning
If the text does not match an offer exactly, `withAutomationTemplate` does nothing and the save fails because the automation has no target. Use the `listAutomations` [special function](https://doc.opendatadsl.com/docs/api/rest/special_functions) to see the exact texts.
:::

## Creating a basic automation

This automation emails the operations team when a dataset is late. The `attachment` property must be `false` for a plain email; without it the data is sent as an attachment.

**ODSL:**

```js
//#region Email when a dataset is late
a = AutomationBuilder("dataset", "private", "ICE.NDEX.NLB").build()
c = SimpleObject()
c.service = "dataset"
c.source = "private"
c.id = "ICE.NDEX.NLB"
c.action = "late"
conds = []
conds.add(c)
a.conditions = conds
a.target = "odsl.email"
a.template = "When 'ICE.NDEX.NLB' is late, send an email to [to] with subject [subject]"
a.icon = "envelope-at text-outlook"
a.properties.to = "ops-team@mycompany.com"
a.properties.subject = "ICE Endex NLB settlement prices are late"
a.properties.attachment = false
save ${automation:a}
//#endregion
```

**REST API:**

```json
POST https://api.opendatadsl.com/api/automation/v1

{
  "target": "odsl.email",
  "template": "When 'ICE.NDEX.NLB' is late, send an email to [to] with subject [subject]",
  "icon": "envelope-at text-outlook",
  "enabled": true,
  "conditions": [
    {
      "service": "dataset",
      "action": "late",
      "id": "ICE.NDEX.NLB",
      "source": "private"
    }
  ],
  "properties": {
    "to": "ops-team@mycompany.com",
    "subject": "ICE Endex NLB settlement prices are late",
    "attachment": false
  }
}
```

If a request fails, the reason is in the `x-odsl-error` response header.

## Multiple conditions on one automation

An automation runs when **any** of its conditions matches, so one automation can watch several items or actions. This one emails the curves team when a curve build succeeds or fails.

**ODSL:**

```js
//#region Email on curve build success or failure
a = AutomationBuilder("curve", "private", "MY.GAS.HUB:FORWARD_CURVE").build()
conds = []
actions = ["success", "failed"]
for act in actions
    c = SimpleObject()
    c.service = "curve"
    c.source = "private"
    c.id = "MY.GAS.HUB:FORWARD_CURVE"
    c.action = act
    conds.add(c)
next
a.conditions = conds
a.target = "odsl.email"
a.template = "When 'MY.GAS.HUB:FORWARD_CURVE' is successful, send an email to [to] with subject [subject]"
a.icon = "envelope-at text-outlook"
a.properties.to = "curves-team@mycompany.com"
a.properties.subject = "Forward curve build finished"
a.properties.attachment = false
save ${automation:a}
//#endregion
```

**REST API:**

```json
POST https://api.opendatadsl.com/api/automation/v1

{
  "target": "odsl.email",
  "template": "When 'MY.GAS.HUB:FORWARD_CURVE' is successful, send an email to [to] with subject [subject]",
  "icon": "envelope-at text-outlook",
  "enabled": true,
  "conditions": [
    { "service": "curve", "action": "success", "id": "MY.GAS.HUB:FORWARD_CURVE", "source": "private" },
    { "service": "curve", "action": "failed", "id": "MY.GAS.HUB:FORWARD_CURVE", "source": "private" }
  ],
  "properties": {
    "to": "curves-team@mycompany.com",
    "subject": "Forward curve build finished",
    "attachment": false
  }
}
```

:::note
The template can only describe one action, so the item's Automations panel shows this automation as the "is successful" offer. If you want each case to show separately in the portal, create one automation per action.
:::

## Triggering a process from an automation

Run a process when a dataset's delivery is complete. The process receives the dataset delivery as its input data.

**ODSL:**

```js
//#region Run a process when a dataset is complete
a = AutomationBuilder("dataset", "private", "ICE.NDEX.NLB").build()
c = SimpleObject()
c.service = "dataset"
c.source = "private"
c.id = "ICE.NDEX.NLB"
c.action = "complete"
conds = []
conds.add(c)
a.conditions = conds
a.target = "odsl.process"
a.template = "When 'ICE.NDEX.NLB' is complete, run [process]"
a.icon = "play-fill text-success"
a.properties.process = "MY_ENRICHMENT_PROCESS"
save ${automation:a}
//#endregion
```

**REST API:**

```json
POST https://api.opendatadsl.com/api/automation/v1

{
  "target": "odsl.process",
  "template": "When 'ICE.NDEX.NLB' is complete, run [process]",
  "icon": "play-fill text-success",
  "enabled": true,
  "conditions": [
    { "service": "dataset", "action": "complete", "id": "ICE.NDEX.NLB", "source": "private" }
  ],
  "properties": {
    "process": "MY_ENRICHMENT_PROCESS"
  }
}
```

## Sending data to a queue

Send a curve to a downstream system's queue each time it is updated. Curves and timeseries are in the `data` service, with ids of the form `OBJECT:DATA`.

**ODSL:**

```js
//#region Send a curve to a queue on update
a = AutomationBuilder("data", "private", "MY.GAS.HUB:FORWARD_CURVE").build()
c = SimpleObject()
c.service = "data"
c.source = "private"
c.id = "MY.GAS.HUB:FORWARD_CURVE"
c.action = "update"
conds = []
conds.add(c)
a.conditions = conds
a.target = "odsl.queue"
a.template = "When 'MY.GAS.HUB:FORWARD_CURVE' is updated, send a message to [queue]"
a.icon = "send text-red"
a.properties.queue = "etrm-curves"
a.properties.subject = ""
save ${automation:a}
//#endregion
```

**REST API:**

```json
POST https://api.opendatadsl.com/api/automation/v1

{
  "target": "odsl.queue",
  "template": "When 'MY.GAS.HUB:FORWARD_CURVE' is updated, send a message to [queue]",
  "icon": "send text-red",
  "enabled": true,
  "conditions": [
    { "service": "data", "action": "update", "id": "MY.GAS.HUB:FORWARD_CURVE", "source": "private" }
  ],
  "properties": {
    "queue": "etrm-curves",
    "subject": ""
  }
}
```

## Automations on events

An event list is a data item with the id `OBJECT:EVENTLIST`, so an automation on events watches the `data` service. Updating events in the list raises an `update` on the event list.

```js
//#region Email the trades when the trade events are updated
a = AutomationBuilder("data", "private", "MY.PORTFOLIO:TRADES").build()
c = SimpleObject()
c.service = "data"
c.source = "private"
c.id = "MY.PORTFOLIO:TRADES"
c.action = "update"
conds = []
conds.add(c)
a.conditions = conds
a.target = "odsl.email_attachment"
a.template = "When 'MY.PORTFOLIO:TRADES' is updated, send an email to [to] with subject [subject], add the data as an attachment named [attachmentName]"
a.icon = "envelope-paper text-outlook"
a.properties.to = "risk-team@mycompany.com"
a.properties.subject = "New trades"
a.properties.attachmentName = "trades.json"
a.properties.attachment = true
save ${automation:a}
//#endregion
```

### Dependent ids

A condition can have a dependent id, `did`. When it is set, the target is sent the dependent item rather than the item that triggered the automation. The platform uses this for the build automations of event curves and event timeseries: the condition watches the event list, and `did` is the curve to build.

```json
{
  "target": "odsl.ecurve",
  "template": "When 'MY.GAS.HUB:FORWARD_EVENTS' is updated, build Event Curve [curve]",
  "conditions": [
    {
      "service": "data",
      "action": "update",
      "id": "MY.GAS.HUB:FORWARD_EVENTS",
      "did": "MY.GAS.HUB:FORWARD_CURVE",
      "source": "private"
    }
  ],
  "properties": {
    "curve": "MY.GAS.HUB:FORWARD_CURVE"
  }
}
```

You don't need to create build automations yourself: the platform creates them when you save an event curve, smart curve, event timeseries or smart timeseries, and removes them when you delete it.

## Finding automations

```js
//#region Find automations
// All automations that send email
emails = find ${automation} where target = "odsl.email"
for a in emails
    print a.name
next
//#endregion
```

The list has one entry per condition, so an automation with two conditions is listed twice. Reading one automation by its id returns the whole automation:

```js
a = ${automation:"6718d2c4e5f1a20b3c4d5e6f"}
```

In the REST API, list with `GET https://api.opendatadsl.com/api/automation/v1` (with `_filter` as usual) and read one with `GET https://api.opendatadsl.com/api/automation/v1/{id}`.

## Changing or disabling an automation

Read the automation, change it and save it again. Set `enabled` to `false` to pause it without deleting it.

**ODSL:**

```js
//#region Disable an automation
a = ${automation:"6718d2c4e5f1a20b3c4d5e6f"}
a.enabled = false
save ${automation:a}
//#endregion
```

**REST API:**

Read the automation, change `enabled`, and post the whole document back with its `_id`:

```json
GET https://api.opendatadsl.com/api/automation/v1/6718d2c4e5f1a20b3c4d5e6f

POST https://api.opendatadsl.com/api/automation/v1

{
  "_id": "6718d2c4e5f1a20b3c4d5e6f",
  "target": "odsl.email",
  "template": "When 'ICE.NDEX.NLB' is late, send an email to [to] with subject [subject]",
  "icon": "envelope-at text-outlook",
  "enabled": false,
  "conditions": [
    { "service": "dataset", "action": "late", "id": "ICE.NDEX.NLB", "source": "private" }
  ],
  "properties": {
    "to": "ops-team@mycompany.com",
    "subject": "ICE Endex NLB settlement prices are late",
    "attachment": false
  }
}
```

:::note
Saving an automation makes you its owner, so it then runs with your security policies.
:::

## Testing an automation

A test run runs the automation once, now, for the item in its first condition as it is at the moment, whether the automation is enabled or not. It is logged with the reason `Test Run`.

```
GET https://api.opendatadsl.com/api/automation/v1/6718d2c4e5f1a20b3c4d5e6f?_run=true
```

Add `&_ondate=2026-10-28` to run it for a curve date or delivery date. An automation whose condition id is `*` cannot be test run.

Every run, test or real, is recorded in the `automationlog` service with its status and message:

```js
logs = find ${automationlog} where status = "failed"
```

## Deleting an automation

**ODSL:**

```js
//#region Delete an automation
delete ${automation:"6718d2c4e5f1a20b3c4d5e6f"}
//#endregion
```

**REST API:**

```
DELETE https://api.opendatadsl.com/api/automation/v1/6718d2c4e5f1a20b3c4d5e6f
```

:::tip Finding your automation id
The platform gives each automation an id when it is first saved. Find it with `find ${automation}`, in the item's Automations panel, or in the Automations extension.
:::

---

## Advanced features

Properties that start with `@` change when an automation runs and what data the target receives.

### Transforming data with a mustache template

Targets that allow transformation (email, queue and storage targets) accept an `@transformer` property: the name of a mustache script, or inline mustache text. The data is transformed with it before it is sent to the target. This example sends a curve as a CSV attachment using a mustache script `mycompany\templates\curve_csv` that you have saved in the script service.

**ODSL:**

```js
//#region Email a curve as CSV
a = AutomationBuilder("data", "private", "MY.GAS.HUB:FORWARD_CURVE").build()
c = SimpleObject()
c.service = "data"
c.source = "private"
c.id = "MY.GAS.HUB:FORWARD_CURVE"
c.action = "update"
conds = []
conds.add(c)
a.conditions = conds
a.target = "odsl.email_attachment"
a.template = "When 'MY.GAS.HUB:FORWARD_CURVE' is updated, transform the data using [@transformer], then send an email to [to] with subject [subject], add the data as an attachment named [attachmentName]"
a.icon = "envelope-paper text-outlook"
a.properties.to = "analysts@mycompany.com"
a.properties.subject = "Forward curve"
a.properties.attachmentName = "forward_curve.csv"
a.properties.attachment = true
a.properties.add("@transformer", "mycompany\templates\curve_csv")
save ${automation:a}
//#endregion
```

**REST API:**

```json
POST https://api.opendatadsl.com/api/automation/v1

{
  "target": "odsl.email_attachment",
  "template": "When 'MY.GAS.HUB:FORWARD_CURVE' is updated, transform the data using [@transformer], then send an email to [to] with subject [subject], add the data as an attachment named [attachmentName]",
  "icon": "envelope-paper text-outlook",
  "enabled": true,
  "conditions": [
    { "service": "data", "action": "update", "id": "MY.GAS.HUB:FORWARD_CURVE", "source": "private" }
  ],
  "properties": {
    "to": "analysts@mycompany.com",
    "subject": "Forward curve",
    "attachmentName": "forward_curve.csv",
    "attachment": true,
    "@transformer": "mycompany\\templates\\curve_csv"
  }
}
```

### Running only when a property changes

For `update`, set both `@propertyName` and `@propertyValue` to run the automation only when that property of the item changes to that value. Use `*` as the value to run on any change of the property. This example emails the data team when a master data record's `status` changes to `retired`.

**ODSL:**

```js
//#region Email when status changes to retired
a = AutomationBuilder("object", "private", "MY.GAS.HUB").build()
c = SimpleObject()
c.service = "object"
c.source = "private"
c.id = "MY.GAS.HUB"
c.action = "update"
conds = []
conds.add(c)
a.conditions = conds
a.target = "odsl.email"
a.template = "When property [@propertyName] on 'MY.GAS.HUB' is set to [@propertyValue], send an email to [to] with subject [subject]"
a.icon = "envelope-at text-outlook"
a.properties.to = "data-team@mycompany.com"
a.properties.subject = "MY.GAS.HUB has been retired"
a.properties.attachment = false
a.properties.add("@propertyName", "status")
a.properties.add("@propertyValue", "retired")
save ${automation:a}
//#endregion
```

**REST API:**

```json
POST https://api.opendatadsl.com/api/automation/v1

{
  "target": "odsl.email",
  "template": "When property [@propertyName] on 'MY.GAS.HUB' is set to [@propertyValue], send an email to [to] with subject [subject]",
  "icon": "envelope-at text-outlook",
  "enabled": true,
  "conditions": [
    { "service": "object", "action": "update", "id": "MY.GAS.HUB", "source": "private" }
  ],
  "properties": {
    "to": "data-team@mycompany.com",
    "subject": "MY.GAS.HUB has been retired",
    "attachment": false,
    "@propertyName": "status",
    "@propertyValue": "retired"
  }
}
```

### Running only for items with a property value

For `create` and `delete`, `@propertyName` and `@propertyValue` run the automation only when the created or deleted item has that property value. Combined with the id `*`, this lets you react to a kind of item. This example emails the data team whenever a master data record of type `#Commodity` is created.

**ODSL:**

```js
//#region Email when a commodity is created
a = AutomationBuilder("object", "private").build()
c = SimpleObject()
c.service = "object"
c.source = "private"
c.id = "*"
c.action = "create"
conds = []
conds.add(c)
a.conditions = conds
a.target = "odsl.email"
a.template = "When 'anything' is created with property [@propertyName] set to [@propertyValue], send an email to [to] with subject [subject]"
a.icon = "envelope-at text-outlook"
a.properties.to = "data-team@mycompany.com"
a.properties.subject = "New commodity created"
a.properties.attachment = false
a.properties.add("@propertyName", "_type")
a.properties.add("@propertyValue", "#Commodity")
save ${automation:a}
//#endregion
```

**REST API:**

```json
POST https://api.opendatadsl.com/api/automation/v1

{
  "target": "odsl.email",
  "template": "When 'anything' is created with property [@propertyName] set to [@propertyValue], send an email to [to] with subject [subject]",
  "icon": "envelope-at text-outlook",
  "enabled": true,
  "conditions": [
    { "service": "object", "action": "create", "id": "*", "source": "private" }
  ],
  "properties": {
    "to": "data-team@mycompany.com",
    "subject": "New commodity created",
    "attachment": false,
    "@propertyName": "_type",
    "@propertyValue": "#Commodity"
  }
}
```

### Summary of `@` properties

| Property | Applies to | Description |
|---|---|---|
| `@transformer` | Targets that allow transformation | The name of a mustache script, or inline mustache text, used to transform the data before the target receives it |
| `@propertyName` and `@propertyValue` | `update` | Run only when the named property changes to the value, or changes at all with `*` |
| `@propertyName` and `@propertyValue` | `create`, `delete` | Run only when the created or deleted item's property equals the value |

Both `@propertyName` and `@propertyValue` must be set for either to have an effect.

---

## Next steps

- [Automation Targets](https://doc.opendatadsl.com/docs/topics/automations/targets): all the targets and the properties each one needs
- [Automations extension](https://doc.opendatadsl.com/docs/extensions/automations/monitoring): monitor automation runs and failures in the portal
- [Automation Basics](https://doc.opendatadsl.com/docs/topics/automations/basics): the trigger reference and how templates are made


---

<!-- source: topics/automation/targets/automate_blob.md -->

The Azure storage targets write the triggering data as a file to your own Azure storage whenever an automation fires. There are two built-in storage targets: one for Azure Blob Storage and one for Azure Data Lake Storage (ADLS).

| Id | Name | Template | Icon |
|---|---|---|---|
| `odsl.blob` | Save to an Azure Blob | `send to [path] in an Azure Blob Container [container] at URL [storage]` | `database text-success` |
| `odsl.adls` | Save to an Azure Data Lake | `send to [path] in an Azure Datalake Storage Container [container] at URL [storage]` | `database-fill text-info` |

Both targets work with any service and any action, and both support the `@transformer` property and property-change filtering.

## Inputs

| Input | Description |
|---|---|
| `storage` | The URL of your Azure storage |
| `container` | The name of the container to write to |
| `path` | The folder path within the container |
| `name` | The file name. It can include dates, for example `${date:yyyyMMdd}` |

:::warning The file name is required
`name` does not appear in the template, but the target needs it to write the file, so always set it.
:::

You can also set `contentType`, which defaults to `application/json`.

## Default properties

| Target | Property | Value |
|---|---|---|
| `odsl.blob` | `adls` | `false` |
| `odsl.adls` | `adls` | `true` |

Include the `adls` property in every storage automation you create.

## How it works

- The file is written with the triggering data, after any `@transformer` has been applied
- Dates in `name` are filled in from the ondate of the triggering data, or today when there is no ondate
- The file is written using the OpenDataDSL application's credentials in **your** Azure (Entra) tenant, so your storage account must grant that application access to the container

---

## odsl.blob: Save to an Azure Blob

### Example: save a dataset when it is complete

**ODSL:**

```js
a = AutomationBuilder("dataset", "private", "MY.PROVIDER.FEED").build()

c = SimpleObject()
c.source = "private"
c.service = "dataset"
c.id = "MY.PROVIDER.FEED"
c.action = "complete"
conds = []
conds.add(c)
a.conditions = conds

a.target = "odsl.blob"
a.template = "When 'MY.PROVIDER.FEED' is complete, send to [path] in an Azure Blob Container [container] at URL [storage]"
a.icon = "database text-success"
a.properties.storage = "https://mycompanystorage.blob.core.windows.net"
a.properties.container = "market-data"
a.properties.path = "feeds/MY.PROVIDER.FEED"
a.properties.name = "MY.PROVIDER.FEED_${date:yyyyMMdd}.json"
a.properties.adls = false
a.enabled = true
save ${automation:a}
```

**REST API:**

```json
POST https://api.opendatadsl.com/api/automation/v1

{
  "target": "odsl.blob",
  "template": "When 'MY.PROVIDER.FEED' is complete, send to [path] in an Azure Blob Container [container] at URL [storage]",
  "icon": "database text-success",
  "enabled": true,
  "conditions": [
    {
      "source": "private",
      "service": "dataset",
      "id": "MY.PROVIDER.FEED",
      "action": "complete"
    }
  ],
  "properties": {
    "storage": "https://mycompanystorage.blob.core.windows.net",
    "container": "market-data",
    "path": "feeds/MY.PROVIDER.FEED",
    "name": "MY.PROVIDER.FEED_${date:yyyyMMdd}.json",
    "adls": false
  }
}
```

---

## odsl.adls: Save to an Azure Data Lake

Use this target for an Azure Data Lake Storage Gen2 container, for example one used by Azure Synapse or Databricks. It takes the same inputs as `odsl.blob`, with `adls` set to `true`.

### Example: save a curve as CSV when it is updated

This example also uses `@transformer` to turn the curve into CSV before it is written. `curve-to-csv` is the name of a mustache script in your tenant.

**ODSL:**

```js
ab = AutomationBuilder("data", "private", "MY_OBJECT:CURVE")
ab.set("@transformer", "curve-to-csv")
a = ab.build()

c = SimpleObject()
c.source = "private"
c.service = "data"
c.id = "MY_OBJECT:CURVE"
c.action = "update"
conds = []
conds.add(c)
a.conditions = conds

a.target = "odsl.adls"
a.template = "When 'MY_OBJECT:CURVE' is updated, transform the data using [@transformer], then send to [path] in an Azure Datalake Storage Container [container] at URL [storage]"
a.icon = "database-fill text-info"
a.properties.storage = "https://mycompanylake.dfs.core.windows.net"
a.properties.container = "curves"
a.properties.path = "forward/MY_OBJECT"
a.properties.name = "CURVE_${date:yyyyMMdd}.csv"
a.properties.contentType = "text/csv"
a.properties.adls = true
a.enabled = true
save ${automation:a}
```

**REST API:**

```json
POST https://api.opendatadsl.com/api/automation/v1

{
  "target": "odsl.adls",
  "template": "When 'MY_OBJECT:CURVE' is updated, transform the data using [@transformer], then send to [path] in an Azure Datalake Storage Container [container] at URL [storage]",
  "icon": "database-fill text-info",
  "enabled": true,
  "conditions": [
    {
      "source": "private",
      "service": "data",
      "id": "MY_OBJECT:CURVE",
      "action": "update"
    }
  ],
  "properties": {
    "storage": "https://mycompanylake.dfs.core.windows.net",
    "container": "curves",
    "path": "forward/MY_OBJECT",
    "name": "CURVE_${date:yyyyMMdd}.csv",
    "contentType": "text/csv",
    "adls": true,
    "@transformer": "curve-to-csv"
  }
}
```

:::tip
If many automations write to the same storage account and container, create a [custom target](https://doc.opendatadsl.com/docs/topics/automations/targets/custom) with `storage` and `container` fixed, so users only give the path and file name.
:::

---

## Choosing between the two targets

| | `odsl.blob` | `odsl.adls` |
|---|---|---|
| **Storage type** | Azure Blob Storage | Azure Data Lake Storage Gen2 |
| **adls property** | `false` | `true` |
| **Icon** | `database text-success` | `database-fill text-info` |

---

## Related pages

- [Automation Basics](https://doc.opendatadsl.com/docs/topics/automations/basics): how automations work, conditions and templates
- [Creating Automations](https://doc.opendatadsl.com/docs/topics/automations/creating): step-by-step guide to setting up automations
- [Custom Targets](https://doc.opendatadsl.com/docs/topics/automations/targets/custom): fix the storage account and container for your team
- [Email Target](https://doc.opendatadsl.com/docs/topics/automations/targets/email): send data by email instead of writing to storage
- [Automation Targets](https://doc.opendatadsl.com/docs/topics/automations/targets): overview of all available targets


---

<!-- source: topics/automation/targets/automate_curve.md -->

The curve build targets build a Smart or Event Curve when the data it depends on is updated.

| Id | Name | Template | Services | Actions |
|---|---|---|---|---|
| `odsl.curve` | Build a Smart Curve | `build Smart Curve [curve]` | data | update |
| `odsl.ecurve` | Build an Event Curve | `build Event Curve [curve]` | event, data | update |

Neither target supports `@transformer` or property-change filtering. The `event` service is mapped to `data`, so conditions on event lists use the `data` service. The icons are `graph-up text-success` for Smart Curves and `graph-up-arrow text-primary` for Event Curves.

## Inputs

| Input | Description |
|---|---|
| `curve` | The id of the Curve to build |

## Created for you by the platform

You don't normally create these automations yourself. When you save a [Smart Curve](https://doc.opendatadsl.com/docs/odsl/variable/smartcurve) or an [Event Curve](https://doc.opendatadsl.com/docs/odsl/variable/eventcurve), the platform creates its build automations, and when you delete it the platform removes them.

You can list them with ODSL:

```js
find ${automation} where target = "odsl.curve"
find ${automation} where target = "odsl.ecurve"
```

## What an Event Curve build automation looks like

Event lists are data items, so an Event Curve build automation has a condition on the **event list** with the `data` service and the `update` action. The condition also has `did` set to the id of the Event Curve, so the platform knows which item the automation belongs to. For an Event Curve `MY_OBJECT:CURVE` built from the event list `MY_OBJECT:TRADES`, the automation looks like this:

```json
{
  "target": "odsl.ecurve",
  "template": "When 'MY_OBJECT:TRADES' is updated, build Event Curve [curve]",
  "icon": "graph-up-arrow text-primary",
  "enabled": true,
  "conditions": [
    {
      "source": "private",
      "service": "data",
      "id": "MY_OBJECT:TRADES",
      "action": "update",
      "did": "MY_OBJECT:CURVE"
    }
  ],
  "properties": {
    "curve": "MY_OBJECT:CURVE"
  }
}
```

## Reacting to a build

A built curve is saved like any other data, so you can create your own automations on it with the `data` service and the `update` action. Use the id of the built Curve, not the id of its inputs. For example, to email the team each time the Event Curve is rebuilt:

**ODSL:**

```js
a = AutomationBuilder("data", "private", "MY_OBJECT:CURVE").build()

c = SimpleObject()
c.source = "private"
c.service = "data"
c.id = "MY_OBJECT:CURVE"
c.action = "update"
conds = []
conds.add(c)
a.conditions = conds

a.target = "odsl.email"
a.template = "When 'MY_OBJECT:CURVE' is updated, send an email to [to] with subject [subject]"
a.icon = "envelope-at text-outlook"
a.properties.to = "trading@mycompany.com"
a.properties.subject = "MY_OBJECT:CURVE has been rebuilt"
a.properties.attachment = false
a.enabled = true
save ${automation:a}
```

**REST API:**

```json
POST https://api.opendatadsl.com/api/automation/v1

{
  "target": "odsl.email",
  "template": "When 'MY_OBJECT:CURVE' is updated, send an email to [to] with subject [subject]",
  "icon": "envelope-at text-outlook",
  "enabled": true,
  "conditions": [
    {
      "source": "private",
      "service": "data",
      "id": "MY_OBJECT:CURVE",
      "action": "update"
    }
  ],
  "properties": {
    "to": "trading@mycompany.com",
    "subject": "MY_OBJECT:CURVE has been rebuilt",
    "attachment": false
  }
}
```

---

## Related pages

- [Automation Basics](https://doc.opendatadsl.com/docs/topics/automations/basics): how automations work, conditions and templates
- [Creating Automations](https://doc.opendatadsl.com/docs/topics/automations/creating): step-by-step guide to setting up automations
- [TimeSeries Build Targets](https://doc.opendatadsl.com/docs/topics/automations/targets/timeseries): the equivalent targets for Smart and Event TimeSeries
- [Email Target](https://doc.opendatadsl.com/docs/topics/automations/targets/email): send a notification when a build completes
- [Export Group Target](https://doc.opendatadsl.com/docs/topics/automations/targets/curve-export): export data when it changes
- [Automation Targets](https://doc.opendatadsl.com/docs/topics/automations/targets): overview of all available targets


---

<!-- source: topics/automation/targets/automate_curve_export.md -->

The export group target runs an export group whenever an automation fires, so that the data in the group is exported as soon as it changes.

| Id | Name | Template | Icon |
|---|---|---|---|
| `odsl.export_group` | Export a group of data | `run export task for [group]` | `folder-symlink text-stream` |

The target works with any service and any action. It supports property-change filtering but not `@transformer`.

## Inputs

| Input | Description |
|---|---|
| `group` | The name of the export group to run |

## How it works

The export group named in `group` is run for the ondate of the triggering data, when there is one. See [Curve Exports](https://doc.opendatadsl.com/docs/topics/curves/export) for how export groups are set up.

---

## Example: run an export group when a dataset is complete

**ODSL:**

```js
a = AutomationBuilder("dataset", "private", "MY.PROVIDER.FEED").build()

c = SimpleObject()
c.source = "private"
c.service = "dataset"
c.id = "MY.PROVIDER.FEED"
c.action = "complete"
conds = []
conds.add(c)
a.conditions = conds

a.target = "odsl.export_group"
a.template = "When 'MY.PROVIDER.FEED' is complete, run export task for [group]"
a.icon = "folder-symlink text-stream"
a.properties.group = "DAILY_SETTLEMENT_EXPORT"
a.enabled = true
save ${automation:a}
```

**REST API:**

```json
POST https://api.opendatadsl.com/api/automation/v1

{
  "target": "odsl.export_group",
  "template": "When 'MY.PROVIDER.FEED' is complete, run export task for [group]",
  "icon": "folder-symlink text-stream",
  "enabled": true,
  "conditions": [
    {
      "source": "private",
      "service": "dataset",
      "id": "MY.PROVIDER.FEED",
      "action": "complete"
    }
  ],
  "properties": {
    "group": "DAILY_SETTLEMENT_EXPORT"
  }
}
```

## Example: run an export group when a curve is updated

**ODSL:**

```js
a = AutomationBuilder("data", "private", "MY_OBJECT:CURVE").build()

c = SimpleObject()
c.source = "private"
c.service = "data"
c.id = "MY_OBJECT:CURVE"
c.action = "update"
conds = []
conds.add(c)
a.conditions = conds

a.target = "odsl.export_group"
a.template = "When 'MY_OBJECT:CURVE' is updated, run export task for [group]"
a.icon = "folder-symlink text-stream"
a.properties.group = "DAILY_SETTLEMENT_EXPORT"
a.enabled = true
save ${automation:a}
```

**REST API:**

```json
POST https://api.opendatadsl.com/api/automation/v1

{
  "target": "odsl.export_group",
  "template": "When 'MY_OBJECT:CURVE' is updated, run export task for [group]",
  "icon": "folder-symlink text-stream",
  "enabled": true,
  "conditions": [
    {
      "source": "private",
      "service": "data",
      "id": "MY_OBJECT:CURVE",
      "action": "update"
    }
  ],
  "properties": {
    "group": "DAILY_SETTLEMENT_EXPORT"
  }
}
```

:::warning Several triggers, several exports
If you create an export group automation on several items that are updated at the same time, the whole group is exported once for each of them. Where possible, trigger the export from a single upstream event, such as the dataset that drives all the data in the group.
:::

---

## Related pages

- [Curve Build Targets](https://doc.opendatadsl.com/docs/topics/automations/targets/curve): how Smart and Event Curves are built
- [TimeSeries Build Targets](https://doc.opendatadsl.com/docs/topics/automations/targets/timeseries): how Smart and Event TimeSeries are built
- [Azure Storage Targets](https://doc.opendatadsl.com/docs/topics/automations/targets/blob): another way to deliver data to storage
- [Automation Basics](https://doc.opendatadsl.com/docs/topics/automations/basics): how automations work, conditions and templates
- [Creating Automations](https://doc.opendatadsl.com/docs/topics/automations/creating): step-by-step guide to setting up automations
- [Automation Targets](https://doc.opendatadsl.com/docs/topics/automations/targets): overview of all available targets


---

<!-- source: topics/automation/targets/automate_dataset.md -->

The dataset check targets run the checks defined on a dataset whenever it is updated. They keep data quality checks running as part of an event-driven pipeline.

| Id | Name | Template | Icon |
|---|---|---|---|
| `odsl.qualitycheck` | Run dataset quality checks | `run quality checks on [dsid]` | `ui-checks text-red` |
| `odsl.criticalcheck` | Run dataset critical checks | `run critical checks on [dsid]` | `ui-checks text-red` |

Both targets only work with the `dataset` service and the `update` action. Neither supports `@transformer` or property-change filtering.

## Inputs

| Input | Description |
|---|---|
| `dsid` | The id of the dataset whose checks should run |

## How it works

The target runs the dataset's quality or critical checks for the ondate of the update. The checks themselves are defined on the dataset, not on the automation; the automation only decides when they run.

Dataset automations use the `dataset` service, the `private` source, and the dataset id (`provider.feed.product`) as the condition id.

---

## Example: run quality checks when a dataset is updated

**ODSL:**

```js
a = AutomationBuilder("dataset", "private", "MY.PROVIDER.FEED").build()

c = SimpleObject()
c.source = "private"
c.service = "dataset"
c.id = "MY.PROVIDER.FEED"
c.action = "update"
conds = []
conds.add(c)
a.conditions = conds

a.target = "odsl.qualitycheck"
a.template = "When 'MY.PROVIDER.FEED' is updated, run quality checks on [dsid]"
a.icon = "ui-checks text-red"
a.properties.dsid = "MY.PROVIDER.FEED"
a.enabled = true
save ${automation:a}
```

**REST API:**

```json
POST https://api.opendatadsl.com/api/automation/v1

{
  "target": "odsl.qualitycheck",
  "template": "When 'MY.PROVIDER.FEED' is updated, run quality checks on [dsid]",
  "icon": "ui-checks text-red",
  "enabled": true,
  "conditions": [
    {
      "source": "private",
      "service": "dataset",
      "id": "MY.PROVIDER.FEED",
      "action": "update"
    }
  ],
  "properties": {
    "dsid": "MY.PROVIDER.FEED"
  }
}
```

## Example: run critical checks when a dataset is updated

**ODSL:**

```js
a = AutomationBuilder("dataset", "private", "MY.PROVIDER.FEED").build()

c = SimpleObject()
c.source = "private"
c.service = "dataset"
c.id = "MY.PROVIDER.FEED"
c.action = "update"
conds = []
conds.add(c)
a.conditions = conds

a.target = "odsl.criticalcheck"
a.template = "When 'MY.PROVIDER.FEED' is updated, run critical checks on [dsid]"
a.icon = "ui-checks text-red"
a.properties.dsid = "MY.PROVIDER.FEED"
a.enabled = true
save ${automation:a}
```

**REST API:**

```json
POST https://api.opendatadsl.com/api/automation/v1

{
  "target": "odsl.criticalcheck",
  "template": "When 'MY.PROVIDER.FEED' is updated, run critical checks on [dsid]",
  "icon": "ui-checks text-red",
  "enabled": true,
  "conditions": [
    {
      "source": "private",
      "service": "dataset",
      "id": "MY.PROVIDER.FEED",
      "action": "update"
    }
  ],
  "properties": {
    "dsid": "MY.PROVIDER.FEED"
  }
}
```

---

## Related pages

- [Automation Basics](https://doc.opendatadsl.com/docs/topics/automations/basics): how automations work, conditions and templates
- [Creating Automations](https://doc.opendatadsl.com/docs/topics/automations/creating): step-by-step guide to setting up automations
- [Email Target](https://doc.opendatadsl.com/docs/topics/automations/targets/email): send a notification when a dataset is late, missing or complete
- [Automation Targets](https://doc.opendatadsl.com/docs/topics/automations/targets): overview of all available targets


---

<!-- source: topics/automation/targets/automate_email.md -->

The email targets send an email whenever an automation fires. There are two built-in email targets: one sends the data in the body of the email, the other sends it as an attachment.

| Id | Name | Template | Icon |
|---|---|---|---|
| `odsl.email` | Send an email | `send an email to [to] with subject [subject]` | `envelope-at text-outlook` |
| `odsl.email_attachment` | Send an email with an attachment | `send an email to [to] with subject [subject], add the data as an attachment named [attachmentName]` | `envelope-paper text-outlook` |

Both targets work with any service and any action, and both support the `@transformer` property and property-change filtering.

## How it works

- With `attachment` set to `false`, the body of the email is the triggering data, after any `@transformer` has been applied
- With `attachment` set to `true`, the body is "Your message is attached" and the data is attached as a file named `attachmentName`
- If `attachment` is missing it **defaults to true**, so a plain email automation must include `attachment: false`
- `subject` and `attachmentName` can contain dates, which are filled in from the ondate of the triggering data when there is one, for example `${date:yyyy-MM-dd}`

:::warning Recipients must be in your domain
Recipients must be in your tenant's own email domain, for example `@mycompany.com`, unless your tenant has the `UNRESTRICTED_EMAILS` setting set to true. You can send to several recipients by separating the addresses with a comma, semicolon or space.
:::

---

## odsl.email: Send an email

### Inputs

| Input | Description |
|---|---|
| `to` | The email address(es) of the recipients |
| `subject` | The subject of the email, can include dates |

### Default properties

| Property | Value |
|---|---|
| `attachment` | `false` |

### Example: email when a dataset is late

**ODSL:**

```js
a = AutomationBuilder("dataset", "private", "ICE.NDEX.NLB").build()

c = SimpleObject()
c.source = "private"
c.service = "dataset"
c.id = "ICE.NDEX.NLB"
c.action = "late"
conds = []
conds.add(c)
a.conditions = conds

a.target = "odsl.email"
a.template = "When 'ICE.NDEX.NLB' is late, send an email to [to] with subject [subject]"
a.icon = "envelope-at text-outlook"
a.properties.to = "ops@mycompany.com;data-team@mycompany.com"
a.properties.subject = "ICE.NDEX.NLB is late for ${date:yyyy-MM-dd}"
a.properties.attachment = false
a.enabled = true
save ${automation:a}
```

**REST API:**

```json
POST https://api.opendatadsl.com/api/automation/v1

{
  "target": "odsl.email",
  "template": "When 'ICE.NDEX.NLB' is late, send an email to [to] with subject [subject]",
  "icon": "envelope-at text-outlook",
  "enabled": true,
  "conditions": [
    {
      "source": "private",
      "service": "dataset",
      "id": "ICE.NDEX.NLB",
      "action": "late"
    }
  ],
  "properties": {
    "to": "ops@mycompany.com;data-team@mycompany.com",
    "subject": "ICE.NDEX.NLB is late for ${date:yyyy-MM-dd}",
    "attachment": false
  }
}
```

### Using the builder shortcut

When the template is exactly one of the offers the platform makes for the item, `withAutomationTemplate` copies the target, icon, conditions and default properties (including `attachment: false`) for you:

```js
ab = AutomationBuilder("dataset", "private", "ICE.NDEX.NLB")
ab.withAutomationTemplate("When 'ICE.NDEX.NLB' is late, send an email to [to] with subject [subject]")
ab.set("to", "ops@mycompany.com")
ab.set("subject", "ICE.NDEX.NLB is late")
a = ab.build()
save ${automation:a}
```

:::warning
The shortcut only works when the text matches an offer exactly. If it does not match, no error is raised and the automation is left empty. Use the explicit pattern above when in doubt.
:::

---

## odsl.email_attachment: Send an email with an attachment

### Inputs

| Input | Description |
|---|---|
| `to` | The email address(es) of the recipients |
| `subject` | The subject of the email, can include dates |
| `attachmentName` | The file name of the attachment. Defaults to the id of the triggering item, can include dates |

### Default properties

| Property | Value |
|---|---|
| `attachment` | `true` |

### Example: send a dataset as an attachment when it is complete

**ODSL:**

```js
a = AutomationBuilder("dataset", "private", "MY.PROVIDER.FEED").build()

c = SimpleObject()
c.source = "private"
c.service = "dataset"
c.id = "MY.PROVIDER.FEED"
c.action = "complete"
conds = []
conds.add(c)
a.conditions = conds

a.target = "odsl.email_attachment"
a.template = "When 'MY.PROVIDER.FEED' is complete, send an email to [to] with subject [subject], add the data as an attachment named [attachmentName]"
a.icon = "envelope-paper text-outlook"
a.properties.to = "analyst@mycompany.com"
a.properties.subject = "MY.PROVIDER.FEED for ${date:yyyy-MM-dd}"
a.properties.attachmentName = "MY.PROVIDER.FEED_${date:yyyyMMdd}.json"
a.properties.attachment = true
a.enabled = true
save ${automation:a}
```

**REST API:**

```json
POST https://api.opendatadsl.com/api/automation/v1

{
  "target": "odsl.email_attachment",
  "template": "When 'MY.PROVIDER.FEED' is complete, send an email to [to] with subject [subject], add the data as an attachment named [attachmentName]",
  "icon": "envelope-paper text-outlook",
  "enabled": true,
  "conditions": [
    {
      "source": "private",
      "service": "dataset",
      "id": "MY.PROVIDER.FEED",
      "action": "complete"
    }
  ],
  "properties": {
    "to": "analyst@mycompany.com",
    "subject": "MY.PROVIDER.FEED for ${date:yyyy-MM-dd}",
    "attachmentName": "MY.PROVIDER.FEED_${date:yyyyMMdd}.json",
    "attachment": true
  }
}
```

---

## Transforming the data before sending

Set `@transformer` to the name of a mustache script, or to inline mustache text, to reshape the data before it is put in the email body or attachment, for example to produce CSV. The template then includes the transform step:

**ODSL:**

```js
ab = AutomationBuilder("dataset", "private", "MY.PROVIDER.FEED")
ab.set("@transformer", "dataset-to-csv")
a = ab.build()

c = SimpleObject()
c.source = "private"
c.service = "dataset"
c.id = "MY.PROVIDER.FEED"
c.action = "complete"
conds = []
conds.add(c)
a.conditions = conds

a.target = "odsl.email_attachment"
a.template = "When 'MY.PROVIDER.FEED' is complete, transform the data using [@transformer], then send an email to [to] with subject [subject], add the data as an attachment named [attachmentName]"
a.icon = "envelope-paper text-outlook"
a.properties.to = "analyst@mycompany.com"
a.properties.subject = "MY.PROVIDER.FEED for ${date:yyyy-MM-dd}"
a.properties.attachmentName = "MY.PROVIDER.FEED_${date:yyyyMMdd}.csv"
a.properties.attachment = true
a.enabled = true
save ${automation:a}
```

**REST API:**

```json
POST https://api.opendatadsl.com/api/automation/v1

{
  "target": "odsl.email_attachment",
  "template": "When 'MY.PROVIDER.FEED' is complete, transform the data using [@transformer], then send an email to [to] with subject [subject], add the data as an attachment named [attachmentName]",
  "icon": "envelope-paper text-outlook",
  "enabled": true,
  "conditions": [
    {
      "source": "private",
      "service": "dataset",
      "id": "MY.PROVIDER.FEED",
      "action": "complete"
    }
  ],
  "properties": {
    "to": "analyst@mycompany.com",
    "subject": "MY.PROVIDER.FEED for ${date:yyyy-MM-dd}",
    "attachmentName": "MY.PROVIDER.FEED_${date:yyyyMMdd}.csv",
    "attachment": true,
    "@transformer": "dataset-to-csv"
  }
}
```

In this example `dataset-to-csv` is the name of a mustache script in your tenant. In ODSL, `@` properties are set with the builder's `set` method before calling `build()`.

---

## Choosing between the two targets

| | `odsl.email` | `odsl.email_attachment` |
|---|---|---|
| **Use when** | You want the data in the email body | You want to deliver the data as a file |
| **attachment property** | `false` | `true` |
| **Attachment name** | Not used | `attachmentName`, defaults to the item id |
| **Icon** | `envelope-at text-outlook` | `envelope-paper text-outlook` |

---

## Related pages

- [Automation Basics](https://doc.opendatadsl.com/docs/topics/automations/basics): how automations work, conditions and templates
- [Creating Automations](https://doc.opendatadsl.com/docs/topics/automations/creating): step-by-step guide to setting up automations
- [Automation Targets](https://doc.opendatadsl.com/docs/topics/automations/targets): overview of all available targets


---

<!-- source: topics/automation/targets/automate_jira.md -->

The JIRA target posts to a JIRA incoming webhook whenever an automation fires, so that a JIRA automation rule can create a task. Use it to raise issues automatically, for example when a dataset is late.

| Id | Name | Template | Icon |
|---|---|---|---|
| `odsl.jira` | Create a JIRA task | `create a JIRA task at [url] using [template]` | `clipboard-plus text-primary` |

The target works with any service and any action. It supports property-change filtering but not `@transformer`.

## Inputs

| Input | Description |
|---|---|
| `url` | The JIRA incoming webhook URL |
| `template` | A mustache template for the task, can be left empty |

:::note Message formatting
In the current automation handlers, the triggering data is posted to `url` as JSON and the `template` input is **not applied**. The receiving JIRA webhook gets the raw JSON data, not a formatted message. This target also does not support `@transformer`. Set `template` because the target asks for it, but do not rely on it to format the task.
:::

:::tip
Because the data arrives as JSON, configure your JIRA automation rule to read the fields it needs from the incoming JSON.
:::

---

## Example: raise a JIRA task when a dataset is missing

**ODSL:**

```js
a = AutomationBuilder("dataset", "private", "ICE.NDEX.NLB").build()

c = SimpleObject()
c.source = "private"
c.service = "dataset"
c.id = "ICE.NDEX.NLB"
c.action = "missing"
conds = []
conds.add(c)
a.conditions = conds

a.target = "odsl.jira"
a.template = "When 'ICE.NDEX.NLB' is missing, create a JIRA task at [url] using [template]"
a.icon = "clipboard-plus text-primary"
a.properties.url = "https://automation.atlassian.com/pro/hooks/your-webhook-id"
a.properties.template = ""
a.enabled = true
save ${automation:a}
```

**REST API:**

```json
POST https://api.opendatadsl.com/api/automation/v1

{
  "target": "odsl.jira",
  "template": "When 'ICE.NDEX.NLB' is missing, create a JIRA task at [url] using [template]",
  "icon": "clipboard-plus text-primary",
  "enabled": true,
  "conditions": [
    {
      "source": "private",
      "service": "dataset",
      "id": "ICE.NDEX.NLB",
      "action": "missing"
    }
  ],
  "properties": {
    "url": "https://automation.atlassian.com/pro/hooks/your-webhook-id",
    "template": ""
  }
}
```

---

## Related pages

- [Automation Basics](https://doc.opendatadsl.com/docs/topics/automations/basics): how automations work, conditions and templates
- [Creating Automations](https://doc.opendatadsl.com/docs/topics/automations/creating): step-by-step guide to setting up automations
- [Webhook Target](https://doc.opendatadsl.com/docs/topics/automations/targets/webhook): post to any other webhook
- [Custom Targets](https://doc.opendatadsl.com/docs/topics/automations/targets/custom): fix the JIRA webhook URL for your team
- [Automation Targets](https://doc.opendatadsl.com/docs/topics/automations/targets): overview of all available targets


---

<!-- source: topics/automation/targets/automate_process.md -->

The process target starts a process whenever an automation fires. Use it to run a loader, calculation or other workflow as soon as the data it depends on arrives, without polling or waiting for a schedule.

| Id | Name | Template | Icon |
|---|---|---|---|
| `odsl.process` | Run a process | `run [process]` | `play-fill text-success` |

The target works with any service and any action. It supports property-change filtering but not `@transformer`.

## Inputs

| Input | Description |
|---|---|
| `process` | The name of the process to run |

## How it works

- The process named in `process` is started, and runs as the process's own run-as user
- The triggering data and the automation properties are passed to the process
- The process's own input values are merged into the triggering data passed to the process
- The automation properties include `id` and, when there is one, `ondate` of the triggering item

---

## Example: run a process when a dataset is complete

**ODSL:**

```js
a = AutomationBuilder("dataset", "private", "MY.PROVIDER.FEED").build()

c = SimpleObject()
c.source = "private"
c.service = "dataset"
c.id = "MY.PROVIDER.FEED"
c.action = "complete"
conds = []
conds.add(c)
a.conditions = conds

a.target = "odsl.process"
a.template = "When 'MY.PROVIDER.FEED' is complete, run [process]"
a.icon = "play-fill text-success"
a.properties.process = "MY_ENRICHMENT_PROCESS"
a.enabled = true
save ${automation:a}
```

**REST API:**

```json
POST https://api.opendatadsl.com/api/automation/v1

{
  "target": "odsl.process",
  "template": "When 'MY.PROVIDER.FEED' is complete, run [process]",
  "icon": "play-fill text-success",
  "enabled": true,
  "conditions": [
    {
      "source": "private",
      "service": "dataset",
      "id": "MY.PROVIDER.FEED",
      "action": "complete"
    }
  ],
  "properties": {
    "process": "MY_ENRICHMENT_PROCESS"
  }
}
```

## Example: run a process when a curve is updated

**ODSL:**

```js
a = AutomationBuilder("data", "private", "MY_OBJECT:CURVE").build()

c = SimpleObject()
c.source = "private"
c.service = "data"
c.id = "MY_OBJECT:CURVE"
c.action = "update"
conds = []
conds.add(c)
a.conditions = conds

a.target = "odsl.process"
a.template = "When 'MY_OBJECT:CURVE' is updated, run [process]"
a.icon = "play-fill text-success"
a.properties.process = "MY_RISK_CALCULATION"
a.enabled = true
save ${automation:a}
```

**REST API:**

```json
POST https://api.opendatadsl.com/api/automation/v1

{
  "target": "odsl.process",
  "template": "When 'MY_OBJECT:CURVE' is updated, run [process]",
  "icon": "play-fill text-success",
  "enabled": true,
  "conditions": [
    {
      "source": "private",
      "service": "data",
      "id": "MY_OBJECT:CURVE",
      "action": "update"
    }
  ],
  "properties": {
    "process": "MY_RISK_CALCULATION"
  }
}
```

:::tip Process or script?
Use the process target when you want the run tracked as a process execution with its own run-as user and inputs. For a lightweight response that only needs a single script, use the [Script Target](https://doc.opendatadsl.com/docs/topics/automations/targets/script).
:::

---

## Related pages

- [Automation Basics](https://doc.opendatadsl.com/docs/topics/automations/basics): how automations work, conditions and templates
- [Creating Automations](https://doc.opendatadsl.com/docs/topics/automations/creating): step-by-step guide to setting up automations
- [Script Target](https://doc.opendatadsl.com/docs/topics/automations/targets/script): run a single ODSL script instead of a process
- [Custom Targets](https://doc.opendatadsl.com/docs/topics/automations/targets/custom): fix the process name in a target for your team
- [Automation Targets](https://doc.opendatadsl.com/docs/topics/automations/targets): overview of all available targets


---

<!-- source: topics/automation/targets/automate_queue.md -->

The queue targets send the triggering data as a message to one of your tenant's queues whenever an automation fires. This decouples the platform from the systems that consume the data: they read messages from the queue in their own time.

| Id | Name | Template | Icon |
|---|---|---|---|
| `odsl.queue` | Send a message to a queue | `send a message to [queue]` | `send text-red` |
| `odsl.queue_subject` | Send a message to a queue with a subject | `send a message to [queue] with [subject]` | `send text-red` |

Both targets work with any service and any action, and both support the `@transformer` property and property-change filtering.

## How it works

- The data, after any `@transformer` has been applied, is sent as a JSON message to your tenant queue named in `queue`
- The message subject is `subject`, which can include dates filled in from the ondate of the triggering data, for example `${date:yyyy-MM-dd}`
- `odsl.queue` sends the message with an empty subject

See [Queues](https://doc.opendatadsl.com/docs/odsl/dm/queues) for how to work with queues.

---

## odsl.queue: Send a message to a queue

### Inputs

| Input | Description |
|---|---|
| `queue` | The name of the queue to send the message to |

### Default properties

| Property | Value |
|---|---|
| `subject` | `""` (empty) |

### Example: send a dataset to a queue when it is complete

**ODSL:**

```js
a = AutomationBuilder("dataset", "private", "MY.PROVIDER.FEED").build()

c = SimpleObject()
c.source = "private"
c.service = "dataset"
c.id = "MY.PROVIDER.FEED"
c.action = "complete"
conds = []
conds.add(c)
a.conditions = conds

a.target = "odsl.queue"
a.template = "When 'MY.PROVIDER.FEED' is complete, send a message to [queue]"
a.icon = "send text-red"
a.properties.queue = "market-data-out"
a.properties.subject = ""
a.enabled = true
save ${automation:a}
```

**REST API:**

```json
POST https://api.opendatadsl.com/api/automation/v1

{
  "target": "odsl.queue",
  "template": "When 'MY.PROVIDER.FEED' is complete, send a message to [queue]",
  "icon": "send text-red",
  "enabled": true,
  "conditions": [
    {
      "source": "private",
      "service": "dataset",
      "id": "MY.PROVIDER.FEED",
      "action": "complete"
    }
  ],
  "properties": {
    "queue": "market-data-out",
    "subject": ""
  }
}
```

---

## odsl.queue_subject: Send a message to a queue with a subject

Use this target when the systems reading the queue use the message subject to route or filter messages, for example when several kinds of data go to one shared queue.

### Inputs

| Input | Description |
|---|---|
| `queue` | The name of the queue to send the message to |
| `subject` | The message subject, can include dates |

### Example: send an updated curve to a queue with a subject

**ODSL:**

```js
a = AutomationBuilder("data", "private", "MY_OBJECT:CURVE").build()

c = SimpleObject()
c.source = "private"
c.service = "data"
c.id = "MY_OBJECT:CURVE"
c.action = "update"
conds = []
conds.add(c)
a.conditions = conds

a.target = "odsl.queue_subject"
a.template = "When 'MY_OBJECT:CURVE' is updated, send a message to [queue] with [subject]"
a.icon = "send text-red"
a.properties.queue = "market-data-out"
a.properties.subject = "CURVE_${date:yyyyMMdd}"
a.enabled = true
save ${automation:a}
```

**REST API:**

```json
POST https://api.opendatadsl.com/api/automation/v1

{
  "target": "odsl.queue_subject",
  "template": "When 'MY_OBJECT:CURVE' is updated, send a message to [queue] with [subject]",
  "icon": "send text-red",
  "enabled": true,
  "conditions": [
    {
      "source": "private",
      "service": "data",
      "id": "MY_OBJECT:CURVE",
      "action": "update"
    }
  ],
  "properties": {
    "queue": "market-data-out",
    "subject": "CURVE_${date:yyyyMMdd}"
  }
}
```

---

## Transforming the message

Set `@transformer` to the name of a mustache script, or to inline mustache text, to reshape the data into the structure the consuming system expects. The template then includes the transform step, for example:

`When 'MY.PROVIDER.FEED' is complete, transform the data using [@transformer], then send a message to [queue]`

**ODSL:**

```js
ab = AutomationBuilder("dataset", "private", "MY.PROVIDER.FEED")
ab.set("@transformer", "dataset-to-message")
a = ab.build()

c = SimpleObject()
c.source = "private"
c.service = "dataset"
c.id = "MY.PROVIDER.FEED"
c.action = "complete"
conds = []
conds.add(c)
a.conditions = conds

a.target = "odsl.queue"
a.template = "When 'MY.PROVIDER.FEED' is complete, transform the data using [@transformer], then send a message to [queue]"
a.icon = "send text-red"
a.properties.queue = "market-data-out"
a.properties.subject = ""
a.enabled = true
save ${automation:a}
```

**REST API:**

```json
POST https://api.opendatadsl.com/api/automation/v1

{
  "target": "odsl.queue",
  "template": "When 'MY.PROVIDER.FEED' is complete, transform the data using [@transformer], then send a message to [queue]",
  "icon": "send text-red",
  "enabled": true,
  "conditions": [
    {
      "source": "private",
      "service": "dataset",
      "id": "MY.PROVIDER.FEED",
      "action": "complete"
    }
  ],
  "properties": {
    "queue": "market-data-out",
    "subject": "",
    "@transformer": "dataset-to-message"
  }
}
```

:::tip Several consumers
If several systems need the same data, create one automation per queue. Each automation fires independently from the same condition.
:::

---

## Choosing between the two targets

| | `odsl.queue` | `odsl.queue_subject` |
|---|---|---|
| **Subject** | Empty | Set by you, can include dates |
| **Best for** | Queues that carry one kind of message | Shared queues where consumers filter by subject |
| **Transformer support** | Yes | Yes |

---

## Related pages

- [Automation Basics](https://doc.opendatadsl.com/docs/topics/automations/basics): how automations work, conditions and templates
- [Creating Automations](https://doc.opendatadsl.com/docs/topics/automations/creating): step-by-step guide to setting up automations
- [Custom Targets](https://doc.opendatadsl.com/docs/topics/automations/targets/custom): fix the queue name in a target for your team
- [Automation Targets](https://doc.opendatadsl.com/docs/topics/automations/targets): overview of all available targets


---

<!-- source: topics/automation/targets/automate_report.md -->

The report targets run a report whenever an automation fires. There are two built-in report targets: one runs the report, the other runs it for a range you give.

| Id | Name | Template | Icon |
|---|---|---|---|
| `odsl.report` | Run a report | `run [report]` | `file-text text-onenote` |
| `odsl.report_range` | Run a report with a range | `run [report] for [range]` | `file-text text-onenote` |

Both targets work with any service and any action. They support property-change filtering but not `@transformer`: the output is controlled by the report itself.

## How it works

- The report named in `report` is run
- When the triggering data has an ondate, for example a dataset ondate or a curve date, the report is run for that date
- With `odsl.report_range`, the `range` property is also passed to the report

---

## odsl.report: Run a report

### Inputs

| Input | Description |
|---|---|
| `report` | The name of the report to run |

### Example: run a report when a dataset is complete

**ODSL:**

```js
a = AutomationBuilder("dataset", "private", "MY.PROVIDER.FEED").build()

c = SimpleObject()
c.source = "private"
c.service = "dataset"
c.id = "MY.PROVIDER.FEED"
c.action = "complete"
conds = []
conds.add(c)
a.conditions = conds

a.target = "odsl.report"
a.template = "When 'MY.PROVIDER.FEED' is complete, run [report]"
a.icon = "file-text text-onenote"
a.properties.report = "MY_DAILY_SUMMARY_REPORT"
a.enabled = true
save ${automation:a}
```

**REST API:**

```json
POST https://api.opendatadsl.com/api/automation/v1

{
  "target": "odsl.report",
  "template": "When 'MY.PROVIDER.FEED' is complete, run [report]",
  "icon": "file-text text-onenote",
  "enabled": true,
  "conditions": [
    {
      "source": "private",
      "service": "dataset",
      "id": "MY.PROVIDER.FEED",
      "action": "complete"
    }
  ],
  "properties": {
    "report": "MY_DAILY_SUMMARY_REPORT"
  }
}
```

---

## odsl.report_range: Run a report with a range

### Inputs

| Input | Description |
|---|---|
| `report` | The name of the report to run |
| `range` | The range to run the report for |

### Example: run a report for a range when a curve is updated

**ODSL:**

```js
a = AutomationBuilder("data", "private", "MY_OBJECT:CURVE").build()

c = SimpleObject()
c.source = "private"
c.service = "data"
c.id = "MY_OBJECT:CURVE"
c.action = "update"
conds = []
conds.add(c)
a.conditions = conds

a.target = "odsl.report_range"
a.template = "When 'MY_OBJECT:CURVE' is updated, run [report] for [range]"
a.icon = "file-text text-onenote"
a.properties.report = "MY_CURVE_REPORT"
a.properties.range = "between(T-1M,T)"
a.enabled = true
save ${automation:a}
```

**REST API:**

```json
POST https://api.opendatadsl.com/api/automation/v1

{
  "target": "odsl.report_range",
  "template": "When 'MY_OBJECT:CURVE' is updated, run [report] for [range]",
  "icon": "file-text text-onenote",
  "enabled": true,
  "conditions": [
    {
      "source": "private",
      "service": "data",
      "id": "MY_OBJECT:CURVE",
      "action": "update"
    }
  ],
  "properties": {
    "report": "MY_CURVE_REPORT",
    "range": "between(T-1M,T)"
  }
}
```

---

## Choosing between the two targets

| | `odsl.report` | `odsl.report_range` |
|---|---|---|
| **Range** | Not set | Set per automation |
| **Best for** | Reports run for the date of the triggering data | Reports that need a specific window |
| **Transformer support** | No | No |

---

## Related pages

- [Automation Basics](https://doc.opendatadsl.com/docs/topics/automations/basics): how automations work, conditions and templates
- [Creating Automations](https://doc.opendatadsl.com/docs/topics/automations/creating): step-by-step guide to setting up automations
- [Export Group Target](https://doc.opendatadsl.com/docs/topics/automations/targets/curve-export): run an export group when data changes
- [Automation Targets](https://doc.opendatadsl.com/docs/topics/automations/targets): overview of all available targets


---

<!-- source: topics/automation/targets/automate_script.md -->

The script target runs an ODSL script whenever an automation fires. It is the most flexible built-in target: any logic you can write in ODSL can run in response to a platform event.

| Id | Name | Template | Icon |
|---|---|---|---|
| `odsl.script` | Run a script | `run [script]` | `file-play text-office` |

The target works with any service and any action. It supports property-change filtering but not `@transformer`: shape the data inside the script instead.

## Inputs

| Input | Description |
|---|---|
| `script` | The name of the ODSL script to run |

## What the script receives

| Variable | Description |
|---|---|
| `#DATA` | The data that triggered the automation |
| `#PROPERTIES` | The automation properties, including `id`, `ondate` and `range`, which are added automatically when they apply |

For example, a script can read the id and date of the triggering item:

```js
print "Automation fired for " + #PROPERTIES.id
print "Ondate: " + #PROPERTIES.ondate
d = #DATA
```

---

## Example: run a script when a dataset is complete

**ODSL:**

```js
a = AutomationBuilder("dataset", "private", "MY.PROVIDER.FEED").build()

c = SimpleObject()
c.source = "private"
c.service = "dataset"
c.id = "MY.PROVIDER.FEED"
c.action = "complete"
conds = []
conds.add(c)
a.conditions = conds

a.target = "odsl.script"
a.template = "When 'MY.PROVIDER.FEED' is complete, run [script]"
a.icon = "file-play text-office"
a.properties.script = "post-process-feed"
a.enabled = true
save ${automation:a}
```

**REST API:**

```json
POST https://api.opendatadsl.com/api/automation/v1

{
  "target": "odsl.script",
  "template": "When 'MY.PROVIDER.FEED' is complete, run [script]",
  "icon": "file-play text-office",
  "enabled": true,
  "conditions": [
    {
      "source": "private",
      "service": "dataset",
      "id": "MY.PROVIDER.FEED",
      "action": "complete"
    }
  ],
  "properties": {
    "script": "post-process-feed"
  }
}
```

## Example: run a script when a property changes

This automation only fires when the `status` property of the curve is changed to `approved`. It uses property-change filtering, so both `@propertyName` and `@propertyValue` are set and the template uses the property-change form.

**ODSL:**

```js
ab = AutomationBuilder("data", "private", "MY_OBJECT:CURVE")
ab.set("@propertyName", "status")
ab.set("@propertyValue", "approved")
a = ab.build()

c = SimpleObject()
c.source = "private"
c.service = "data"
c.id = "MY_OBJECT:CURVE"
c.action = "update"
conds = []
conds.add(c)
a.conditions = conds

a.target = "odsl.script"
a.template = "When property [@propertyName] on 'MY_OBJECT:CURVE' is set to [@propertyValue], run [script]"
a.icon = "file-play text-office"
a.properties.script = "publish-approved-curve"
a.enabled = true
save ${automation:a}
```

**REST API:**

```json
POST https://api.opendatadsl.com/api/automation/v1

{
  "target": "odsl.script",
  "template": "When property [@propertyName] on 'MY_OBJECT:CURVE' is set to [@propertyValue], run [script]",
  "icon": "file-play text-office",
  "enabled": true,
  "conditions": [
    {
      "source": "private",
      "service": "data",
      "id": "MY_OBJECT:CURVE",
      "action": "update"
    }
  ],
  "properties": {
    "script": "publish-approved-curve",
    "@propertyName": "status",
    "@propertyValue": "approved"
  }
}
```

:::tip Script target or custom target?
The script target lets the person creating the automation choose any script. If you want to fix the script and ask only for specific inputs, create a [Custom Target](https://doc.opendatadsl.com/docs/topics/automations/targets/custom) backed by your script instead. For multi-step work that should also be tracked as a process execution, use the [Process Target](https://doc.opendatadsl.com/docs/topics/automations/targets/process).
:::

---

## Related pages

- [Automation Basics](https://doc.opendatadsl.com/docs/topics/automations/basics): how automations work, conditions and templates
- [Creating Automations](https://doc.opendatadsl.com/docs/topics/automations/creating): step-by-step guide to setting up automations
- [Process Target](https://doc.opendatadsl.com/docs/topics/automations/targets/process): run a process instead of a single script
- [Custom Targets](https://doc.opendatadsl.com/docs/topics/automations/targets/custom): create a script-backed target with its own inputs
- [Automation Targets](https://doc.opendatadsl.com/docs/topics/automations/targets): overview of all available targets


---

<!-- source: topics/automation/targets/automate_teams.md -->

The Teams target posts to a Microsoft Teams channel webhook whenever an automation fires.

| Id | Name | Template | Icon |
|---|---|---|---|
| `odsl.teams` | Send a Teams message | `send a Teams message to [url] using [template]` | `microsoft-teams text-teams` |

The target works with any service and any action. It supports property-change filtering but not `@transformer`.

## Inputs

| Input | Description |
|---|---|
| `url` | The webhook URL of the Teams channel |
| `template` | A mustache template for the message |

:::note Message formatting
In the current automation handlers, the triggering data is posted to `url` as JSON and the `template` input is **not applied**. The receiving channel gets the raw JSON data, not a formatted message. This target also does not support `@transformer`. Set `template` because the target asks for it, but do not rely on it to format the message.
:::

---

## Example: post to Teams when a dataset is late

**ODSL:**

```js
a = AutomationBuilder("dataset", "private", "ICE.NDEX.NLB").build()

c = SimpleObject()
c.source = "private"
c.service = "dataset"
c.id = "ICE.NDEX.NLB"
c.action = "late"
conds = []
conds.add(c)
a.conditions = conds

a.target = "odsl.teams"
a.template = "When 'ICE.NDEX.NLB' is late, send a Teams message to [url] using [template]"
a.icon = "microsoft-teams text-teams"
a.properties.url = "https://mycompany.webhook.office.com/webhookb2/your-channel-webhook"
a.properties.template = "dataset-late-message"
a.enabled = true
save ${automation:a}
```

**REST API:**

```json
POST https://api.opendatadsl.com/api/automation/v1

{
  "target": "odsl.teams",
  "template": "When 'ICE.NDEX.NLB' is late, send a Teams message to [url] using [template]",
  "icon": "microsoft-teams text-teams",
  "enabled": true,
  "conditions": [
    {
      "source": "private",
      "service": "dataset",
      "id": "ICE.NDEX.NLB",
      "action": "late"
    }
  ],
  "properties": {
    "url": "https://mycompany.webhook.office.com/webhookb2/your-channel-webhook",
    "template": "dataset-late-message"
  }
}
```

---

## Fixing the channel URL in a custom target

For a shared channel, you can create a custom target that wraps the built-in Teams handler (`@TeamsChannelTarget`) with the `url` fixed in its properties. Users then pick the target without ever handling the webhook URL. See [Custom Targets](https://doc.opendatadsl.com/docs/topics/automations/targets/custom) for how to do this.

---

## Related pages

- [Automation Basics](https://doc.opendatadsl.com/docs/topics/automations/basics): how automations work, conditions and templates
- [Creating Automations](https://doc.opendatadsl.com/docs/topics/automations/creating): step-by-step guide to setting up automations
- [Webhook Target](https://doc.opendatadsl.com/docs/topics/automations/targets/webhook): post to any other webhook
- [Custom Targets](https://doc.opendatadsl.com/docs/topics/automations/targets/custom): fix the channel URL for your team
- [Automation Targets](https://doc.opendatadsl.com/docs/topics/automations/targets): overview of all available targets


---

<!-- source: topics/automation/targets/automate_timeseries.md -->

The timeseries build targets build a Smart or Event TimeSeries when the data it depends on is updated.

| Id | Name | Template | Services | Actions |
|---|---|---|---|---|
| `odsl.timeseries` | Build a Smart TimeSeries | `build Smart Timeseries [timeseries]` | data | update |
| `odsl.etimeseries` | Build an Event TimeSeries | `build Event Timeseries [timeseries]` | data | update |

Neither target supports `@transformer` or property-change filtering. The `odsl.etimeseries` target is used by the platform for Event TimeSeries builds.

## Inputs

| Input | Description |
|---|---|
| `timeseries` | The id of the TimeSeries to build |

## Created for you by the platform

You don't normally create these automations yourself. When you save a [Smart TimeSeries](https://doc.opendatadsl.com/docs/odsl/variable/smarttimeseries) or an [Event TimeSeries](https://doc.opendatadsl.com/docs/odsl/variable/eventtimeseries), the platform creates its build automations, and when you delete it the platform removes them.

You can list them with ODSL:

```js
find ${automation} where target = "odsl.timeseries"
find ${automation} where target = "odsl.etimeseries"
```

## What an Event TimeSeries build automation looks like

Event lists are data items, so an Event TimeSeries build automation has a condition on the **event list** with the `data` service and the `update` action. The condition also has `did` set to the id of the Event TimeSeries, so the platform knows which item the automation belongs to. For an Event TimeSeries `MY_OBJECT:EVENT_SERIES` built from the event list `MY_OBJECT:TRADES`, the automation looks like this:

```json
{
  "target": "odsl.etimeseries",
  "template": "When 'MY_OBJECT:TRADES' is updated, build Event Timeseries [timeseries]",
  "enabled": true,
  "conditions": [
    {
      "source": "private",
      "service": "data",
      "id": "MY_OBJECT:TRADES",
      "action": "update",
      "did": "MY_OBJECT:EVENT_SERIES"
    }
  ],
  "properties": {
    "timeseries": "MY_OBJECT:EVENT_SERIES"
  }
}
```

## Reacting to a build

A built timeseries is saved like any other data, so you can create your own automations on it with the `data` service and the `update` action. Use the id of the built TimeSeries, not the id of its inputs. For example, to email the team each time the Event TimeSeries is rebuilt:

**ODSL:**

```js
a = AutomationBuilder("data", "private", "MY_OBJECT:EVENT_SERIES").build()

c = SimpleObject()
c.source = "private"
c.service = "data"
c.id = "MY_OBJECT:EVENT_SERIES"
c.action = "update"
conds = []
conds.add(c)
a.conditions = conds

a.target = "odsl.email"
a.template = "When 'MY_OBJECT:EVENT_SERIES' is updated, send an email to [to] with subject [subject]"
a.icon = "envelope-at text-outlook"
a.properties.to = "trading@mycompany.com"
a.properties.subject = "MY_OBJECT:EVENT_SERIES has been rebuilt"
a.properties.attachment = false
a.enabled = true
save ${automation:a}
```

**REST API:**

```json
POST https://api.opendatadsl.com/api/automation/v1

{
  "target": "odsl.email",
  "template": "When 'MY_OBJECT:EVENT_SERIES' is updated, send an email to [to] with subject [subject]",
  "icon": "envelope-at text-outlook",
  "enabled": true,
  "conditions": [
    {
      "source": "private",
      "service": "data",
      "id": "MY_OBJECT:EVENT_SERIES",
      "action": "update"
    }
  ],
  "properties": {
    "to": "trading@mycompany.com",
    "subject": "MY_OBJECT:EVENT_SERIES has been rebuilt",
    "attachment": false
  }
}
```

---

## Related pages

- [Automation Basics](https://doc.opendatadsl.com/docs/topics/automations/basics): how automations work, conditions and templates
- [Creating Automations](https://doc.opendatadsl.com/docs/topics/automations/creating): step-by-step guide to setting up automations
- [Curve Build Targets](https://doc.opendatadsl.com/docs/topics/automations/targets/curve): the equivalent targets for Smart and Event Curves
- [Email Target](https://doc.opendatadsl.com/docs/topics/automations/targets/email): send a notification when a build completes
- [Export Group Target](https://doc.opendatadsl.com/docs/topics/automations/targets/curve-export): export data when it changes
- [Automation Targets](https://doc.opendatadsl.com/docs/topics/automations/targets): overview of all available targets


---

<!-- source: topics/automation/targets/automate_webhook.md -->

The webhook target posts to any HTTP webhook endpoint whenever an automation fires. Any external system that accepts incoming webhooks can be connected to OpenDataDSL automations this way.

| Id | Name | Template | Icon |
|---|---|---|---|
| `odsl.webhook` | Send a webhook message | `send a message to a webhook at [url] using [template]` | `wifi text-orange` |

The target works with any service and any action. It supports property-change filtering but not `@transformer`.

## Inputs

| Input | Description |
|---|---|
| `url` | The webhook URL to post to |
| `template` | A mustache template for the message |

:::note Message formatting
In the current automation handlers, the triggering data is posted to `url` as JSON and the `template` input is **not applied**. The receiving system gets the raw JSON data, not a formatted message. This target also does not support `@transformer`. Set `template` because the target asks for it, but do not rely on it to format the payload.
:::

---

## Example: notify an external system when a dataset is complete

**ODSL:**

```js
a = AutomationBuilder("dataset", "private", "MY.PROVIDER.FEED").build()

c = SimpleObject()
c.source = "private"
c.service = "dataset"
c.id = "MY.PROVIDER.FEED"
c.action = "complete"
conds = []
conds.add(c)
a.conditions = conds

a.target = "odsl.webhook"
a.template = "When 'MY.PROVIDER.FEED' is complete, send a message to a webhook at [url] using [template]"
a.icon = "wifi text-orange"
a.properties.url = "https://integration.mycompany.com/hooks/odsl"
a.properties.template = "dataset-complete-message"
a.enabled = true
save ${automation:a}
```

**REST API:**

```json
POST https://api.opendatadsl.com/api/automation/v1

{
  "target": "odsl.webhook",
  "template": "When 'MY.PROVIDER.FEED' is complete, send a message to a webhook at [url] using [template]",
  "icon": "wifi text-orange",
  "enabled": true,
  "conditions": [
    {
      "source": "private",
      "service": "dataset",
      "id": "MY.PROVIDER.FEED",
      "action": "complete"
    }
  ],
  "properties": {
    "url": "https://integration.mycompany.com/hooks/odsl",
    "template": "dataset-complete-message"
  }
}
```

---

## Choosing between webhook and other targets

| Target | Best for |
|---|---|
| `odsl.webhook` | Any external system with an incoming webhook |
| `odsl.teams` | Microsoft Teams channels |
| `odsl.jira` | Creating JIRA tasks |
| `odsl.email` | Email notifications |
| `odsl.queue` | Your tenant's message queues |

---

## Related pages

- [Automation Basics](https://doc.opendatadsl.com/docs/topics/automations/basics): how automations work, conditions and templates
- [Creating Automations](https://doc.opendatadsl.com/docs/topics/automations/creating): step-by-step guide to setting up automations
- [Teams Target](https://doc.opendatadsl.com/docs/topics/automations/targets/teams): post to a Microsoft Teams channel
- [Queue Target](https://doc.opendatadsl.com/docs/topics/automations/targets/queue): send data to a queue for downstream systems
- [Custom Targets](https://doc.opendatadsl.com/docs/topics/automations/targets/custom): fix the webhook URL for your team
- [Automation Targets](https://doc.opendatadsl.com/docs/topics/automations/targets): overview of all available targets


---

<!-- source: topics/automation/targets/custom.md -->

Custom targets are automation targets you create for your own tenant. There are two kinds:

- **Pre-configured built-in targets**: wrap a built-in target and fix some of its properties, so users only fill in the rest
- **Script-backed targets**: run your own ODSL script when the automation fires

They are useful for:

- Shared infrastructure such as a Teams channel or webhook, where the URL is fixed
- Azure storage where the account and container are the same across your organisation
- Logic that no built-in target covers, such as sending data to another system's API

---

## Creating a target

Create a target with `AutomationTarget()`, set its properties and save it:

| Property | Description |
|---|---|
| `publisher` | Your organisation or team identifier |
| `code` | The code of the target. The target id is `publisher.code` |
| `name` | The name shown in the portal |
| `description` | A short description of what the target does |
| `icon` | The icon, a Bootstrap icon name optionally with a colour class, for example `database text-success` |
| `script` | A built-in handler such as `@BlobTarget`, or the id of your own ODSL script |
| `template` | The target template, for example `send to [path] in the market data container`. Each `[input]` is filled in by the user |
| `inputs` | Descriptions of the inputs, set with `at.inputs.path.description = "..."` |
| `properties` | Fixed property values passed to the target on every run |
| `services` | The services the target can be used with, `["*"]` for all |
| `actions` | The actions the target can be used with, `["*"]` for all, or for example `["update"]` |
| `tags` | Tags used to group the target in the portal |
| `allowTransformation` | Whether automations using this target can use `@transformer` |
| `allowPropertyChange` | Whether automations using this target can use property-change filtering |

`save at` saves the target as a private target in your tenant. Its id is `publisher.code`, and that is what automations use as their `target`.

:::note Replacing a public target
A private target with the same id as a public target replaces the public one for your tenant. Only OpenDataDSL can publish public targets.
:::

---

## Pre-configured built-in targets

Set `script` to the built-in handler and fix the values you want in `at.properties`. When an automation uses the target, the fixed values are passed to the handler along with the values the user supplies.

### Example: a fixed Azure Blob container

This target fixes the storage URL and container for the built-in blob handler, so users only give the path and file name.

```js
at = AutomationTarget()
at.publisher = "mycompany"
at.code = "blob-market-data"
at.name = "Save to the market data container"
at.description = "Write data to the shared market-data Azure Blob container"
at.icon = "database text-success"
at.script = "@BlobTarget"
at.template = "save to [path] as [name] in the market data container"
at.inputs.path.description = "The folder path in the container"
at.inputs.name.description = "The file name, can include dates"
at.properties.storage = "https://mycompanystorage.blob.core.windows.net"
at.properties.container = "market-data"
at.properties.adls = false
at.allowTransformation = true
at.allowPropertyChange = true
at.services = ["*"]
at.actions = ["*"]
at.tags = ["Azure", "Storage"]
save at
```

An automation using this target only needs `path` and `name`:

**ODSL:**

```js
a = AutomationBuilder("dataset", "private", "MY.PROVIDER.FEED").build()

c = SimpleObject()
c.source = "private"
c.service = "dataset"
c.id = "MY.PROVIDER.FEED"
c.action = "complete"
conds = []
conds.add(c)
a.conditions = conds

a.target = "mycompany.blob-market-data"
a.template = "When 'MY.PROVIDER.FEED' is complete, save to [path] as [name] in the market data container"
a.icon = "database text-success"
a.properties.path = "feeds/MY.PROVIDER.FEED"
a.properties.name = "MY.PROVIDER.FEED_${date:yyyyMMdd}.json"
a.enabled = true
save ${automation:a}
```

**REST API:**

```json
POST https://api.opendatadsl.com/api/automation/v1

{
  "target": "mycompany.blob-market-data",
  "template": "When 'MY.PROVIDER.FEED' is complete, save to [path] as [name] in the market data container",
  "icon": "database text-success",
  "enabled": true,
  "conditions": [
    {
      "source": "private",
      "service": "dataset",
      "id": "MY.PROVIDER.FEED",
      "action": "complete"
    }
  ],
  "properties": {
    "path": "feeds/MY.PROVIDER.FEED",
    "name": "MY.PROVIDER.FEED_${date:yyyyMMdd}.json"
  }
}
```

### Example: a fixed Teams channel

The same pattern works for the built-in Teams handler, `@TeamsChannelTarget`, with the channel webhook URL fixed so users never handle it:

```js
at = AutomationTarget()
at.publisher = "mycompany"
at.code = "teams-ops"
at.name = "Send a message to the Operations Teams channel"
at.description = "Post to the Operations channel in Microsoft Teams"
at.icon = "microsoft-teams text-teams"
at.script = "@TeamsChannelTarget"
at.template = "send a message to the Operations Teams channel"
at.properties.url = "https://mycompany.webhook.office.com/webhookb2/your-channel-webhook"
at.allowTransformation = false
at.allowPropertyChange = true
at.services = ["*"]
at.actions = ["*"]
at.tags = ["Teams"]
save at
```

As with the built-in Teams target, the data is posted as JSON. See the note on the [Teams Target](https://doc.opendatadsl.com/docs/topics/automations/targets/teams) page.

---

## Script-backed targets

Set `script` to the id of one of your ODSL scripts. When the automation fires, the script runs with these variables:

| Variable | Description |
|---|---|
| `#PROPERTIES` | The automation properties: the user's inputs, the target's fixed properties, and `id`, `ondate` and `range` when they apply |
| `#OUTPUT` | The triggering data, after any `@transformer` has been applied |
| `#LOG` | The automation log entry for this run. Set its status and message, or let the result of the script decide the status |

A simple script could look like this:

```js
print "Automation fired for " + #PROPERTIES.id
data = #OUTPUT
// ... send data to another system, update an object, and so on
#LOG.message = "Processed " + #PROPERTIES.id
```

### Example: the Molecule ETRM extension

The Molecule integration extension uses a script-backed target to send market data to Molecule ETRM whenever data is updated:

```js
at = AutomationTarget()
at.publisher = "odsl"
at.code = "molecule-md"
at.name = "Send market data to Molecule ETRM"
at.description = "Sends market data to Molecule ETRM"
at.icon = "cloud-upload"
at.script = "odsl-extensions\extensions\integration\molecule\molecule-automation-md"
at.template = "send to Molecule ETRM Market Data API"
at.allowTransformation = false
at.allowPropertyChange = false
at.services = ["data"]
at.actions = ["update"]
at.tags = ["ETRM"]
save at
```

The target has no inputs, so an automation using it needs no properties:

**ODSL:**

```js
a = AutomationBuilder("data", "private", "MY_OBJECT:CURVE").build()

c = SimpleObject()
c.source = "private"
c.service = "data"
c.id = "MY_OBJECT:CURVE"
c.action = "update"
conds = []
conds.add(c)
a.conditions = conds

a.target = "odsl.molecule-md"
a.template = "When 'MY_OBJECT:CURVE' is updated, send to Molecule ETRM Market Data API"
a.icon = "cloud-upload"
a.enabled = true
save ${automation:a}
```

**REST API:**

```json
POST https://api.opendatadsl.com/api/automation/v1

{
  "target": "odsl.molecule-md",
  "template": "When 'MY_OBJECT:CURVE' is updated, send to Molecule ETRM Market Data API",
  "icon": "cloud-upload",
  "enabled": true,
  "conditions": [
    {
      "source": "private",
      "service": "data",
      "id": "MY_OBJECT:CURVE",
      "action": "update"
    }
  ],
  "properties": {}
}
```

:::tip
If you find yourself setting the same `storage` and `container` on every blob automation, create a custom target with those values fixed. It saves repetition and makes shared infrastructure harder to misconfigure.
:::

---

## Related pages

- [Automation Basics](https://doc.opendatadsl.com/docs/topics/automations/basics): how automations work, conditions and templates
- [Creating Automations](https://doc.opendatadsl.com/docs/topics/automations/creating): step-by-step guide to setting up automations
- [Azure Storage Targets](https://doc.opendatadsl.com/docs/topics/automations/targets/blob): the built-in blob and ADLS targets
- [Email Target](https://doc.opendatadsl.com/docs/topics/automations/targets/email): the built-in email targets
- [Automation Targets](https://doc.opendatadsl.com/docs/topics/automations/targets): overview of all available targets


---

<!-- source: topics/automation/targets/targets.md -->

An automation target defines **what happens** when an automation fires. Every automation has exactly one target, which receives the triggering data and acts on it: sending an email, running a process, writing to storage, building a curve, and so on.

Every target has an id made of a publisher and a code, `publisher.code`. All built-in targets are published by OpenDataDSL, so their ids start with `odsl.`, for example `odsl.email` or `odsl.blob`. You use this full id in the `target` field of an automation.

Targets come in two kinds:

- **Built-in targets**: provided by OpenDataDSL and available to all tenants
- **Custom targets**: created by you, either wrapping a built-in target with fixed properties, or backed by your own ODSL script. See [Custom Targets](https://doc.opendatadsl.com/docs/topics/automations/targets/custom)

---

## Built-in targets

The Transform and Property change columns show whether the target supports the `@transformer` property and property-change filtering (see [Special @ properties](#special--properties) below).

### Communication

| Id | Name | Transform | Property change | Page |
|---|---|---|---|---|
| `odsl.email` | Send an email | Yes | Yes | [Email Target](https://doc.opendatadsl.com/docs/topics/automations/targets/email) |
| `odsl.email_attachment` | Send an email with an attachment | Yes | Yes | [Email Target](https://doc.opendatadsl.com/docs/topics/automations/targets/email) |
| `odsl.teams` | Send a Teams message | No | Yes | [Teams Target](https://doc.opendatadsl.com/docs/topics/automations/targets/teams) |
| `odsl.webhook` | Send a webhook message | No | Yes | [Webhook Target](https://doc.opendatadsl.com/docs/topics/automations/targets/webhook) |
| `odsl.jira` | Create a JIRA task | No | Yes | [JIRA Target](https://doc.opendatadsl.com/docs/topics/automations/targets/jira) |

### Storage

| Id | Name | Transform | Property change | Page |
|---|---|---|---|---|
| `odsl.blob` | Save to an Azure Blob | Yes | Yes | [Azure Storage Targets](https://doc.opendatadsl.com/docs/topics/automations/targets/blob) |
| `odsl.adls` | Save to an Azure Data Lake | Yes | Yes | [Azure Storage Targets](https://doc.opendatadsl.com/docs/topics/automations/targets/blob) |

### Queuing

| Id | Name | Transform | Property change | Page |
|---|---|---|---|---|
| `odsl.queue` | Send a message to a queue | Yes | Yes | [Queue Target](https://doc.opendatadsl.com/docs/topics/automations/targets/queue) |
| `odsl.queue_subject` | Send a message to a queue with a subject | Yes | Yes | [Queue Target](https://doc.opendatadsl.com/docs/topics/automations/targets/queue) |

### Compute

| Id | Name | Transform | Property change | Page |
|---|---|---|---|---|
| `odsl.process` | Run a process | No | Yes | [Process Target](https://doc.opendatadsl.com/docs/topics/automations/targets/process) |
| `odsl.script` | Run a script | No | Yes | [Script Target](https://doc.opendatadsl.com/docs/topics/automations/targets/script) |
| `odsl.report` | Run a report | No | Yes | [Report Target](https://doc.opendatadsl.com/docs/topics/automations/targets/report) |
| `odsl.report_range` | Run a report with a range | No | Yes | [Report Target](https://doc.opendatadsl.com/docs/topics/automations/targets/report) |
| `odsl.export_group` | Export a group of data | No | Yes | [Export Group Target](https://doc.opendatadsl.com/docs/topics/automations/targets/curve-export) |

### Curve and TimeSeries

These build automations are created and removed automatically by the platform when you save or delete a Smart or Event Curve or TimeSeries.

| Id | Name | Transform | Property change | Page |
|---|---|---|---|---|
| `odsl.curve` | Build a Smart Curve | No | No | [Curve Build Targets](https://doc.opendatadsl.com/docs/topics/automations/targets/curve) |
| `odsl.ecurve` | Build an Event Curve | No | No | [Curve Build Targets](https://doc.opendatadsl.com/docs/topics/automations/targets/curve) |
| `odsl.timeseries` | Build a Smart TimeSeries | No | No | [TimeSeries Build Targets](https://doc.opendatadsl.com/docs/topics/automations/targets/timeseries) |
| `odsl.etimeseries` | Build an Event TimeSeries | No | No | [TimeSeries Build Targets](https://doc.opendatadsl.com/docs/topics/automations/targets/timeseries) |

### Dataset

| Id | Name | Transform | Property change | Page |
|---|---|---|---|---|
| `odsl.qualitycheck` | Run dataset quality checks | No | No | [Dataset Check Targets](https://doc.opendatadsl.com/docs/topics/automations/targets/dataset-checks) |
| `odsl.criticalcheck` | Run dataset critical checks | No | No | [Dataset Check Targets](https://doc.opendatadsl.com/docs/topics/automations/targets/dataset-checks) |

### Extensions

These targets belong to extensions. The extension creates and maintains the automations that use them, so you don't normally create them yourself.

| Id | Name | Maintained by |
|---|---|---|
| `odsl.validation` | Run validation checks | The [Validation](https://doc.opendatadsl.com/docs/extensions/validation/overview) extension |
| `odsl.composition` | Build composition | [Composer](https://doc.opendatadsl.com/docs/extensions/composer/overview) |
| `odsl.data_reconciliation` | Run a data reconciliation task | The [Data Reconciliation](https://doc.opendatadsl.com/docs/extensions/data-reconciliation/overview) extension |

---

## What an automation looks like

An automation is made up of:

| Field | Description |
|---|---|
| `target` | The target id, for example `odsl.email` |
| `template` | The sentence describing the automation, made from the condition prefix and the target's template |
| `icon` | The icon shown in the portal |
| `enabled` | Whether the automation is active, defaults to `true` |
| `conditions` | A list of conditions, each with `source`, `service`, `id` and `action` |
| `properties` | The values for the target's inputs, plus any default properties the target needs |

The `name` of the automation is generated from the template by replacing each `[input]` with its value, and the `owner` is set to whoever saves it. The automation runs with the owner's policies.

### Templates

Each target has a template such as `send an email to [to] with subject [subject]`. The automation template is that text with a prefix describing the condition, for example:

`When 'ICE.NDEX.NLB' is late, send an email to [to] with subject [subject]`

The portal only lists an automation against an item when its template is exactly one of the offers the platform makes for that item, so always build the template from the rules on [Automation Basics](https://doc.opendatadsl.com/docs/topics/automations/basics).

### Creating an automation

The explicit pattern below always works. Set the target, template, icon, conditions and properties yourself, including any default properties the target needs (here `attachment` set to `false`).

**ODSL:**

```js
a = AutomationBuilder("dataset", "private", "ICE.NDEX.NLB").build()

c = SimpleObject()
c.source = "private"
c.service = "dataset"
c.id = "ICE.NDEX.NLB"
c.action = "late"
conds = []
conds.add(c)
a.conditions = conds

a.target = "odsl.email"
a.template = "When 'ICE.NDEX.NLB' is late, send an email to [to] with subject [subject]"
a.icon = "envelope-at text-outlook"
a.properties.to = "ops@mycompany.com"
a.properties.subject = "ICE.NDEX.NLB is late"
a.properties.attachment = false
a.enabled = true
save ${automation:a}
```

**REST API:**

```json
POST https://api.opendatadsl.com/api/automation/v1

{
  "target": "odsl.email",
  "template": "When 'ICE.NDEX.NLB' is late, send an email to [to] with subject [subject]",
  "icon": "envelope-at text-outlook",
  "enabled": true,
  "conditions": [
    {
      "source": "private",
      "service": "dataset",
      "id": "ICE.NDEX.NLB",
      "action": "late"
    }
  ],
  "properties": {
    "to": "ops@mycompany.com",
    "subject": "ICE.NDEX.NLB is late",
    "attachment": false
  }
}
```

If the request is rejected, the reason is in the `x-odsl-error` response header.

### The builder shortcut

If you know the exact text of one of the item's offers, `withAutomationTemplate` copies the target, icon, conditions and the target's default properties from that offer. You then only `set` the inputs and `build` the automation.

```js
ab = AutomationBuilder("dataset", "private", "ICE.NDEX.NLB")
ab.withAutomationTemplate("When 'ICE.NDEX.NLB' is late, send an email to [to] with subject [subject]")
ab.set("to", "ops@mycompany.com")
ab.set("subject", "ICE.NDEX.NLB is late")
a = ab.build()
save ${automation:a}
```

:::warning Exact match only
`withAutomationTemplate` only works when the text matches an offer exactly, character for character. If nothing matches, no error is raised but the automation is left empty, so it has no target or conditions. Use the explicit pattern when in doubt.
:::

The only builder methods are `withAutomationTemplate`, `set` and `build`. Everything else (target, template, icon, conditions, properties, enabled) is set on the automation returned by `build()`.

### Finding, testing and deleting automations

```js
// Find all your email automations
find ${automation} where target = "odsl.email"

// Get and delete an automation by its id
a = ${automation:"6720f1c2a5b3c4d5e6f70812"}
delete ${automation:"6720f1c2a5b3c4d5e6f70812"}
```

Automation ids are assigned by the platform. To test an automation once, call `GET https://api.opendatadsl.com/api/automation/v1/<id>?_run=true`, optionally adding `&_ondate=YYYY-MM-DD`. The run is logged in the automation log with the reason `Test Run`. Automations on `*` (any item) cannot be test run.

---

## Properties added automatically

As well as the properties you set, the target receives:

| Property | Description |
|---|---|
| `id` | The id of the item that triggered the automation |
| `ondate` | The curve date or dataset ondate, when there is one |
| `range` | The range, for timeseries |

---

## Special @ properties

Some targets accept special `@` properties that control when the automation fires and what data the target receives.

| Property | Description |
|---|---|
| `@transformer` | The name of a mustache script, or inline mustache text, applied to the data before the target receives it. Only for targets that support transformation |
| `@propertyName` | The name of the property on the triggering item to filter on |
| `@propertyValue` | The value the property must have. Use `*` to match any change of the property |

`@propertyName` and `@propertyValue` must **both** be set for the filter to apply:

- On an `update` condition (targets with property-change support), the automation only fires when that property changed to the given value, or on any change of it when the value is `*`
- On a `create` or `delete` condition (any target), the automation only fires when the record's property equals the given value

These properties change the template. For example:

| Use | Template |
|---|---|
| Transformer | `When 'ICE.NDEX.NLB' is late, transform the data using [@transformer], then send an email to [to] with subject [subject]` |
| Property change | `When property [@propertyName] on 'MY_OBJECT:CURVE' is set to [@propertyValue], send an email to [to] with subject [subject]` |
| Property set on create | `When 'anything' is created with property [@propertyName] set to [@propertyValue], send an email to [to] with subject [subject]` |

See [Automation Basics](https://doc.opendatadsl.com/docs/topics/automations/basics) for the full template rules.

---

## Custom targets

You can create your own targets to pre-configure built-in targets for your team, or to back a target with your own ODSL script. Common patterns include:

- Fixing a Teams or webhook URL so users never handle it directly
- Fixing a storage account and container so users only need to give a path and file name
- Backing a target with an ODSL script for logic that no built-in target covers

See [Custom Targets](https://doc.opendatadsl.com/docs/topics/automations/targets/custom) for the full guide.

---

## Related pages

- [Automation Basics](https://doc.opendatadsl.com/docs/topics/automations/basics): how automations work, conditions and templates
- [Creating Automations](https://doc.opendatadsl.com/docs/topics/automations/creating): step-by-step guide to setting up automations
- [Custom Targets](https://doc.opendatadsl.com/docs/topics/automations/targets/custom): create pre-configured or script-backed targets
