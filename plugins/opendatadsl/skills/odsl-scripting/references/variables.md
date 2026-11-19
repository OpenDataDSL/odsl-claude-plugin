# ODSL variable types: guides and examples

Narrative guide to each variable type, with examples.

Generated from the OpenDataDSL documentation (https://doc.opendatadsl.com/docs).


---

<!-- source: product/developer/odsl/Variables/AI.md -->

# AI Agent, AI Assistant and AI Tool

These variables hold the Fusion AI configuration that you can create and save from ODSL: agents, custom assistants and custom tools. They are typed objects, so you set their properties like any other object. See [Fusion AI](https://doc.opendatadsl.com/docs/extensions/fusion-ai/overview) for what each one does.

## Construction

|**Function**|**Description**|
|-|-|
|AIAgent()|Creates a new AI Agent|
|AIAssistant()|Creates a new AI Assistant|
|AITool()|Creates a new AI Tool|

## Properties

|**Variable**|**Property**|**Description**|**Type**|
|-|-|-|-|
|AIAgent|name|The name of the agent|String|
|AIAgent|icon|The icon shown for the agent|String|
|AIAgent|cron|An optional schedule to run the agent on|String|
|AIAssistant|name|The name of the assistant|String|
|AIAssistant|description|What the assistant is for, used to choose it|String|
|AIAssistant|icon|The icon shown for the assistant|String|
|AIAssistant|systemPrompt|The instructions given to Fusion when the assistant is used|String|
|AIAssistant|onlyUseCustomTools|True to let the assistant use only your custom tools, defaults to false|Boolean|
|AITool|name|The function name of the tool|String|
|AITool|description|What the tool does, used by Fusion to decide when to call it|String|
|AITool|icon|The icon shown for the tool|String|
|AITool|script|The private script, with category aitool, that implements the tool|String|

An agent also needs a `messages` list: the questions it asks, each an object with a `content` property.

## Example

```js
agent = AIAgent()
agent.id = "morning-gas-check"
agent.name = "Morning gas check"
agent.cron = "30 7 * * 1-5 * Europe/London"
q = Object()
q.content = "Did the ICE Endex TTF settlement prices arrive yesterday?"
agent.messages = [q]
save ${aiagent:agent}
```

## Further Reading

* [AI Agent service](https://doc.opendatadsl.com/docs/api/rest/service/aiagent), [AI Assistant service](https://doc.opendatadsl.com/docs/api/rest/service/aiassistant) and [AI Tool service](https://doc.opendatadsl.com/docs/api/rest/service/aitool) in the REST API


---

<!-- source: product/developer/odsl/Variables/Action.md -->

Action
======

An action variable is the definition of an action that can be used in a workflow

## Construction
An action is a structure that contains some configuration information and some code to perform a task within a workflow and takes the following form:

```js
actionname = action in "category"
    // Description
    (in name as type (description)? (optional)?)*
    (out name as type (description)?)*
    exit ("transition")*

    (statement)*
    return "transition"
end
``` 

An example of a simple greeting action:
```js
hello_message = action in "general"
    // Greets the passed-in user name
    in user as Scalar
    out message as Scalar
    exit "ok"

    output.message = "Hello " + input.user
    return "ok"
end
```

## Properties
An action has the following properties:

|**Name**|**Description**|**Type**|
|-|-|-|
|category|A category is used to group similar actions together|Scalar(String)|
|description|A description of what this action does|Scalar(String)|
|exits|The exit transitions from this action|List(String)|
|id|The ID of this action|Scalar(String)|
|inputs|The input variable args|List(Arg)|
|outputs|The output variable args|List(Arg)|
|script|The actual code used to create this action|Scalar(String)|
|version|The version of this action|Object|
|filename|The filename of this script|String|
|outputFormat|An optional output format, e.g. application/xml, text/html, text/csv|String|
|scriptType|The type of this script, e.g. one of [odsl, mustache, py, css, html, js, mjs, cjs]|String|

## Methods
An action has the following methods:

|**Name**|**Description**|**Return Type**|
|-|-|-|
|run(in, out)|Runs this action and returns the name of the exit transition|Scalar(String)|
|loadFrom(file)|Load the script from a local file|Void|

### Example Usage
You can run an action in ODSL code by creating an input object and an output object.
The input object should contain the **in** arguments required by the action and the output object should be empty.

When the action runs, it populates the output object with any output arguments defined.

```js
oi = Object()
oi.user = "World"
oo = Object()
hello_message.run(oi, oo)
print oo
```

This outputs:
```json
{
  "_id": "oo",
  "_type": "#Object",
  "_links": {},
  "hello_message": {
    "_id": "output",
    "message": "Hello World"
  }
}
```

## Further Reading
* [Action service](https://doc.opendatadsl.com/docs/odsl/service/action) for saving and using actions in the server.


---

<!-- source: product/developer/odsl/Variables/Alert.md -->

# Alert

An **alert** is a problem with your data, such as a late dataset delivery or a failed quality check. In ODSL an alert is an object read from the [alert service](https://doc.opendatadsl.com/docs/odsl/service/alert): change its fields and save it to change the alert. To raise an alert, save a `SimpleObject` with at least a `title`.

## Properties

|**Name**|**Description**|**Type**|
|-|-|-|
|_id|The alert id, such as ALR-2026-000401 (ALC- common, ALP- public)|String|
|title|Short description|String|
|message|What is wrong|String|
|description|More detail|String|
|category|A grouping, such as dataset or curve|String|
|issue|The type, such as delivery, completeness, quality, build or export|String|
|severity|critical, high, medium or low|String|
|status|open or closed|String|
|agent|Who it is with: client (your team), opendatadsl or a data vendor's name|String|
|subject|What it is about: service, id and ondate|Object|
|key|The id of the problem, used to count repeats|String|
|occurrences|How many times it was raised while open|Int|
|resolution|What was wrong and what was done|String|
|comment|Set to add a comment when saving|String|
|tags|Your own labels|List|
|timeline|Everything that happened to the alert|List|
|_version|The version read, checked when saving|Int|

See the [Alerts reference](https://doc.opendatadsl.com/docs/extensions/alerts/reference) for every field.

## Example

```js
// Raise an alert about a stale broker feed
n = SimpleObject()
n.title = "NGE.BROKER.NBP prices look stale since Monday"
n.severity = "high"
n.issue = "quality"
n.subject = SimpleObject()
n.subject.service = "dataset"
n.subject.id = "NGE.BROKER.NBP"
n.key = "northgate/stale/NGE.BROKER.NBP"
save ${alert:n}

// Close an alert
a = ${alert:"ALR-2026-000402"}
a.status = "closed"
a.resolution = "Loader fixed; file reloaded at 10:15"
save ${alert:a}
```


---

<!-- source: product/developer/odsl/Variables/Arg.md -->

Arg
===================

An arg variable is a configuration of an input or output argument of a workflow or action

## Construction
An arg is only constructed as part of an action, gateway or workflow.
The syntax for the arg is as follows:

```js
name as type (description)? (optional)?
```

* name is the variable id that you want to know this argument by
* type is the expected type of the variable
* description is the user description of the argument
* optional is a boolean that indicates this is not a mandatory argument (only relevant for input arguments)

## Properties
An arg has the following properties:

|**Name**|**Description**|**Type**|
|-|-|-|
|id|The id of the argument|Scalar(String)|
|argType|The expected variable type of the argument|Scalar(String)|
|description|The user description of the argument|Scalar(String)|
|optional|A boolean indicating if this is optional or mandatory|Scalar(Boolean)|

## Further Reading
* Workflow
* Action


---

<!-- source: product/developer/odsl/Variables/Automation.md -->

# Automation

An automation runs an action (an automation target) when something happens to an item, for example emailing a curve when it is updated. You create automations in ODSL with an **AutomationBuilder**: give it the item, choose one of the automation templates the item offers (the [listAutomations](https://doc.opendatadsl.com/docs/api/rest/special_functions) special function lists them), set the template's properties and build it. See [Automations](https://doc.opendatadsl.com/docs/topics/automations/basics).

## Construction

|**Function**|**Description**|
|-|-|
|AutomationBuilder(service, source)|Creates a new OpenDataDSL automation using a builder|
|AutomationBuilder(service, source, key)|Creates a new OpenDataDSL automation using a builder|

## Builder methods

|**Name**|**Description**|**Return Type**|
|-|-|-|
|withAutomationTemplate(template)|Select the template to use for this automation|AutomationBuilder|
|set(name, value)|Set a property on the automation|AutomationBuilder|
|build()|Build the automation you have configured with this builder|Automation|

## Properties

|**Name**|**Description**|**Type**|
|-|-|-|
|conditions|The conditions for when this automation will fire|List|
|enabled|Set to false to disable this automation|Boolean|
|icon|The icon (bootstrap icons) to be shown next the automation in the GUI - usually the same as the target icon|String|
|name|The READONLY name of this automation, this is generated using the template and properties|String|
|owner|The READONLY owner of this automation, used for setting security policies on the automation|String|
|properties|The properties to send to the target|Object|
|target|The automation target id|String|
|template|The text template used to create this automation|String|

## Example

```js
builder = AutomationBuilder("object", "private", "NGE.GAS.TTF")
// template is the text of one of the automations this item offers,
// as listed by the listAutomations special function
builder.withAutomationTemplate(template)
builder.set("to", "market-data@northgate-energy.com")
automation = builder.build()
save ${automation:automation}
```

## Automation targets

An automation target is the action an automation runs. You can create your own with **AutomationTarget()**.

## Properties

|**Name**|**Description**|**Type**|
|-|-|-|
|actions|The actions this target can work with, leave empty for all|List|
|allowPropertyChange|True if we allow the user to specify a check for a property changing to either a specific value or anything|Boolean|
|allowTransformation|True if we allow the user to specify a transformer to transform the data to be submitted|Boolean|
|code|The code id of the automation target|String|
|description|The long description of the automation target|String|
|icon|The code for the icon (bootstrap icons) to be shown next the automation target name in the GUI|String|
|inputs|The input configuration information|Object|
|name|The name of the automation target|String|
|properties|The properties array to send to the target|Object|
|publisher|The publisher of the automation target|String|
|script|The script name that performs the action|String|
|services|The services this target can work with, leave empty for all|List|
|tags|A list of tags used to categorise this automation target|List|
|template|The text used to collect information from the user and the data to collect|String|

## Further Reading

* [Automation service](https://doc.opendatadsl.com/docs/api/rest/service/automation) and [Automation Target service](https://doc.opendatadsl.com/docs/api/rest/service/automationtarget) in the REST API


---

<!-- source: product/developer/odsl/Variables/Batch.md -->

Batch
=====

A batch variable is a custom list variable that contains variables and supplemental information needed to update into the object repository

## Construction
A batch can be constructed in the following ways:

```js
// RECOMMENDED: Creating a batch from the PROCESS variable
batch=PROCESS.createBatch()

// Directly creating a batch
batch = Batch()
```

Creating a batch using the createBatch of a process sets the following properties on the batch:
* service as the service property on the process
* name as the name property on the process
* correlationId as the unique executionId of the process

## Properties
A batch has the following properties:

|**Name**|**Description**|**Type**|
|-|-|-|
|id|The id of the batch|Scalar(String)|
|loader|The name of the loader|Scalar(String)|
|name|The name of the batch or process that runs this batch|Scalar(String)|
|service|The service name of the batch or process that runs this batch|Scalar(String)|
|size|The number of objects in the batch|Scalar(Int)|
|description|The description of this list|String|
|fast|Indicates that this batch is loading events using the fast method, no versioning or corrections|Boolean|
|history|Indicates that this batch is loading history|Boolean|

## Methods
A batch variable has the following methods:

|**Name**|**Description**|**Return Type**|
|-|-|-|
|add(Object)|Adds an object to the batch|this|
|addAll(List(Object))|Adds all the objects from the supplied list to this batch|this|
|subList(int,int)|Gets a list with the elements in the supplied range|List|
|remove(index)|Removes an element from the list at the specified position (0 being first)|Void|
|remove(item)|Removes an element from the list|Void|
|set(index, element)|Replaces the element at the specified position in this list with the specified element|Void|


---

<!-- source: product/developer/odsl/Variables/Calendar.md -->

Calendar
========

A calendar variable type represents an [ODSL Calendar](https://doc.opendatadsl.com/docs/odsl/calendar/calendars). It is used with timeseries, curves and other time-bound resources. See [here](https://doc.opendatadsl.com/docs/odsl/calendar/calendars) for more detailed information on ODSL Calendars

## Introduction

The calendar variable wraps a standard [ODSL calendar](https://doc.opendatadsl.com/docs/odsl/calendar/Calendars) and allows you to use the calendar in your scripts.

## Construction

Calendars are usually read from the [Calendar Service](https://doc.opendatadsl.com/docs/odsl/service/calendar) and referenced using their code, e.g. “business”, but some basic calendars can be constructed as follows:
```js
// Create a daily calendar
daily = DailyCalendar()

// Create a business (Monday to Friday) calendar
business = BusinessCalendar()

// Create an empty holiday calendar
holiday = HolidayCalendar()

// Create a monthly calendar
monthly = MonthlyCalendar()

// Create an hourly calendar
hourly = IntradayCalendar(1h)

/** 
 Create an hourly calendar using a holiday calendar and 
 exclude hours 0-6 and 20-23
**/
peak = IntradayCalendar(1h, holiday, [0,1,2,3,4,5,6,20,21,22,23])
```

### Constructor functions

|**Function**|**Description**|
|-|-|
|AnnualCalendar()|Creates a new Annual Calendar|
|BlockCalendar(holiday, observed, start, end)|Creates a new Block Calendar|
|BlockCalendar(holiday, observed, start, end, timezone)|Creates a new Block Calendar|
|BusinessCalendar()|Creates a new Business (Monday to Friday) Calendar|
|Calendar(code)|Creates a new Calendar|
|CombinedHolidayCalendar()|Creates a new Combined Holiday Calendar|
|CombinedHolidayCalendar(cal1, cal2)|Creates a new Combined Holiday Calendar using the 2 passed in Holiday Calendars|
|DailyCalendar()|Creates a new Daily Calendar|
|HolidayCalendar()|Creates a new Holiday Calendar|
|HourlyBlockCalendar(holiday, start, end)|Creates a new Hourly Block Calendar|
|HourlyBlockCalendar(holiday, start, end, timezone)|Creates a new Hourly Block Calendar|
|IntradayCalendar(period)|Creates a new Intraday Calendar|
|IntradayCalendar(period, holiday)|Creates a new Intraday Calendar|
|IntradayCalendar(period, holiday, withoutHours)|Creates a new Intraday Calendar|
|IntradayCalendar(period, holiday, withoutHours, timezone)|Creates a new Intraday Calendar|
|IntradayCalendar(period, holiday, withoutHours, timezone, useHolidays)|Creates a new Intraday Calendar|
|MonthlyCalendar()|Creates a new Monthly Calendar|
|MonthlyCalendar(code, name, period)|Creates a new Monthly Calendar with a period specifier|
|MonthlyCalendar(period, dom, holcal, rollbackwards)|Creates a new Monthly Calendar|
|OffsetDailyCalendar(timezone, dayOffset, hour)|Creates a new Offset Daily Calendar|
|QuarterlyCalendar()|Creates a new Quarterly Calendar|
|WeeklyCalendar(dow)|Creates a new Weekly Calendar|
|WeeklyCalendar(dow, holcal, rollbackwards)|Creates a new Weekly Calendar|
|WeeklyCalendar(dow, holcal, rollbackwards, periods, referenceDate)|Creates a new Weekly Calendar|

## Properties

A calendar has the following properties:

|**Name**|**Description**|**Type**|
|-|-|-|
|code|The code of the underlying ODSL Calendar|String|
|name|The name of the underlying ODSL Calendar|String|
|timezone|Get the timezone of the Calendar - only supported on Intraday calendars|String|
|rules|The holiday rules for this calendar (Only applicable for holiday calendars)|List(String)|

## Methods

A calendar has the following methods:

|**Name**|**Description**|**Return Type**|
|-|-|-|
|getDates(start, end)|Gets a list of dates from this calendar for the supplied date range|List(Date)|
|getDatesFrom(start, number)|Gets number + 1 list of dates from the supplied start date|List(Date)|
|addDays(Date, int)|Adds the supplied number of days to the supplied date and returns the new date|Date|
|includes(Date)|Tests to see if the supplied date aligns with this calendar|Scalar(Boolean)|
|next(Date)|Gets the next date after the supplied date|Date|
|previous(Date)|Gets the previous date before the supplied date|Date|
|addRule(String)|Adds a holiday rule (Only applicable for holiday calendars)|void|
|getNextWorkingDay(Date)|Get the next working day after the supplied start date (Only applicable for holiday calendars)|Date|
|getNextNonWorkingDay(Date)|Get the next non-working day after the supplied start date (Only applicable for holiday calendars)|Date|
|getLastWorkingDay(Date)|Get the last working day of the week that includes the supplied start date (Only applicable for holiday calendars)|Date|
|setUseHolidays(Boolean)|Tells this intraday calendar to use holidays as full days, i.e. for off-peak calendars|Void|
|getHolidaysForYear(int)|Gets the list of holidays for the specified year (Only applicable for holiday calendars)|List(name, date)|

### Some example usage

```js
// Create a daily calendar
daily = DailyCalendar()

first = Date("2020-10-01")
last = Date("2020-10-31")

print daily.next(first)

// Create an empty holiday calendar
holiday = HolidayCalendar()

// Add some rules
holiday.addRule("Every Saturday")
holiday.addRule("Every Sunday")
holiday.addRule("First Tuesday in October")

// Get the list of dates for October 2020
dates = holiday.getDates(first, last)

print dates
```


---

<!-- source: product/developer/odsl/Variables/Cell.md -->

Cell
====

A cell represents a single location-aware cell within a spreadsheet


---

<!-- source: product/developer/odsl/Variables/ComposedCurve.md -->

# ComposedCurve and ComposedTimeSeries

A **ComposedCurve** or **ComposedTimeSeries** is a curve or timeseries that takes its values from one element (column) of a [Composer](https://doc.opendatadsl.com/docs/extensions/composer/overview) composition. Add one to an object to make a composition column available as ordinary data.

## Construction

|**Function**|**Description**|
|-|-|
|ComposedCurve(composition, element)|Creates a new Composed Curve using a composition and element|
|ComposedTimeSeries(composition, element)|Creates a new Composed TimeSeries using a timeseries composition and element|

## Properties

|**Name**|**Description**|**Type**|
|-|-|-|
|composition|The composition that contains this curve|String|
|description|A description of the curve|String|
|element|The element in the composition|String|
|identity|The configured identity for this curve|Object|
|name|The name of the curve|String|

## Example

```js
obj = ${object:"NGE.SPREADS"}
obj.CSS_DE = ComposedCurve("NGE_CLEAN_SPARK_SPREAD", "spread")
save obj
```

To create the composition itself, use **Composition()**, which creates a [report configuration](https://doc.opendatadsl.com/docs/odsl/variable/report) of type Composition.


---

<!-- source: product/developer/odsl/Variables/Contract.md -->

Contract
========

A contract is a single future period in time that represents a tradable future contract or a forecasted value on a curve

## Introduction

A contract, when used with commodity forward curves, is a structure that defines a delivery period for that contract and and associated information such as price and last trading day.

## Construction

To create a new contract, you need to use the contract construction function as shown below:
```js
contract = Contract(CurveDate, PeriodCode, value)
```
The 3 parameters are as follows:

*   CurveDate - this is the CurveDate for this contract, it usually is the same as the curve that this contract is attached to, but can be different if you want to calculate expiry dates differently 
*   PeriodCode - this is a string representation of the delivery [period code](https://doc.opendatadsl.com/docs/odsl/calendar/period-code), in either absolute or relative format.
*   Value - This is the price/value for the contract
    

Example creation of a contract:
```js
// Create a curve
eombus = ExpiryCalendar(BusinessCalendar())
eombus.addRule("go to the end of the previous month")
ondate = CurveDate(Date("2020-10-13"), eombus)
curve = Curve(ondate)

// Create a contract
contract = Contract(ondate, "2020M11", 25.67)

// Add it to the curve
curve.add(contract)
```
This is what that contract looks like in JSON:
```json
{
    "tenor": "M01",
    "value": 25.67,
    "ondate": {
        "curveDate": "2020-10-13",
        "expiryCalendar": "eombus"
    },
    "properties": {
        "absolute": "2020M11",
        "end": "2020-11-30T00:00:00",
        "expiry": "2020-10-31T00:00:00",
        "relative": "M01",
        "start": "2020-11-01T00:00:00"
    }
}
```

You can see in the above example that there are a few properties automatically created and are described in the next section

## Properties

A contract has the following properties:

|**Name**|**Description**|**Type**|
|-|-|-|
|ondate|The CurveDate for this contract|CurveDate|
|tenor|The relative tenor of this contract|Scalar(String)|
|value|The value of this contract|Scalar|
|absolute|The absolute tenor name|Scalar(String)|
|relative|The relative tenor name|Scalar(String)|
|periodCodeType|The period code type, e.g. Month|Scalar(String)|
|expiry|The expiry or last trading date|Date|
|start|The start date of delivery of this contract|Date|
|end|The end date of delivery of this contract|Date|
|*|Any other dynamic property you want to add|Any|
|scalarValue|Get the value of this contract including the configured precision|String|
|status|The status information for this contract|Status|

## Methods

|**Name**|**Description**|**Return Type**|
|-|-|-|
|generateNext()|Generate a contract with the same type that is after this one|Contract|
|immediatelyAfter()|Get the contract with the same type that is immediately after this one or null if there isn't one|Contract|
|immediatelyBefore()|Get the contract with the same type that is immediately before this one or null if there isn't one|Contract|
|isImmediatelyAfter(other)|A test to see if the supplied contract is immediately after this one|Boolean|
|isImmediatelyBefore(other)|A test to see if the supplied contract is immediately before this one|Boolean|
|isOverlapping(other)|A test to see if the this contract is overlapping the supplied contract|Boolean|
|isWithin(other)|A test to see if the this contract is entirely within the supplied contract|Boolean|
|next()|Get the contract with the same type that is after this one or null if there isn't one|Contract|
|overlaps()|Gets a list of contracts that are overlapping with this contract|Contracts|
|previous()|Get the contract with the same type that is before this one or null if there isn't one|Contract|
|within()|Gets a list of contracts that are within this contract|Contracts|


---

<!-- source: product/developer/odsl/Variables/Contracts.md -->

Contracts
=========

Contracts are used on a curve as a set of contract objects that represent the future periods and values

## Introduction

The contracts on a curve are simple a map indexed by the relative tenor of each individual contract.

## Methods

Contracts has the following methods:

|**Name**|**Description**|**Return Type**|
|-|-|-|
|filter(PeriodCodeType)|Creates a new set of contracts that only has the contracts matching the supplied period code type|Contracts|

## Using Contracts
An example of filtering a list of contracts to return only months

```js
eombus = ExpiryCalendar(BusinessCalendar())
eombus.addRule("go to the end of the previous month")
ondate = CurveDate(Date("2020-10-13"), eombus)
curve = Curve(ondate)

curve.add(Contract(ondate, "2020M11", 25.67))
curve.add(Contract(ondate, "2020M12", 25.75))
curve.add(Contract(ondate, "2021Q01", 25.8))
curve.add(Contract(ondate, "2021Q02", 25.83))
curve.add(Contract(ondate, "2021Q03", 25.85))
curve.add(Contract(ondate, "2021Q04", 25.89))
curve.add(Contract(ondate, "2022Y", 25.99))

// Filter out just the months
months = curve.contracts.filter("Month")
print json(months)
```


---

<!-- source: product/developer/odsl/Variables/Curve.md -->

Curve
=====

A curve is a structure that represents a set of values with a time index sometime in the future. It can be used for commodity and financial forward curves, yield curves and forecasts (weather, prices etc.)

## Introduction

A curve is a very flexible and dynamic way of expressing point-in-time based future values, both real and forecasted. A simple curve comprises:

*   CurveDate - This is the date for when this curve is valid and all the data contained in the curve is relative to this date. The CurveDate also contains an expiry calendar which is used to calculate the relative future date/time of the curve contracts    
*   Contracts - This is a set of Contract objects that contain a [period code](https://doc.opendatadsl.com/docs/odsl/calendar/period-code) which determines the relative future period that this contract covers, a value and any other properties you want to add
*   Currency - An optional currency code
*   Units - An optional units code
*   User properties - Any other properties you want to add onto the curve
    
## Construction

To construct a new curve, you use the curve function which takes a CurveDate as a parameter as shown in the syntax below:
```js
curve = Curve(curveDate)
```
An example of constructing a new curve:
```js
eombus = ExpiryCalendar(BusinessCalendar())
eombus.addRule("go to the end of the previous month")
ondate = CurveDate(Date("2020-10-13"), eombus)
curve = Curve(ondate)
```
In the above example:

1.  We create a new ExpiryCalendar using a business calendar    
2.  Add an expiry rule to go to the end of the previous month    
3.  Create a CurveDate for a specific date and using the expiry calendar we used above    
4.  Create a new curve with the CurveDate created above
    

## Properties

A curve has the following properties:

|**Name**|**Description**|**Type**|
|-|-|-|
|ondate|The date for which the curve corresponds to|CurveDate|
|currency|The currency of this curve|Scalar(String)|
|units|The units of measure for this curve|Scalar(String)|
|contracts|The set of contracts on this curve|Contracts|
|size|The number of contracts on this curve|Scalar(int)|
|precision|The data value [precision](https://doc.opendatadsl.com/docs/kb/precision#data-precision-settings) configuration|String|
|*|Any other dynamic properties added|Any|
|description|A description of the curve|String|
|identity|The configured identity for this curve|Object|
|name|The name of the curve|String|
|timezone|The timezone for this curve|String|

Examples of curve properties:
```js
eombus = ExpiryCalendar(BusinessCalendar())
eombus.addRule("go to the end of the previous month")
ondate = CurveDate(Date("2020-10-13"), eombus)
curve = Curve(ondate)

// Set the currency and units
curve.currency = "EUR"
curve.units = "MWH"

// Add a dynamic property 'category'
curve.category = "EUROPEAN POWER"
```

## Methods

A curve has the following methods:

|**Name**|**Description**|**Return Type**|
|-|-|-|
|add(Contract)|Adds a contract to a curve|This curve|
|addNew(contract)|Adds the supplied contract to this curve only if the tenor doesn't already exist on this curve|Curve|
|addNew(contracts)|Adds the supplied contracts to this curve only if the tenor doesn't already exist on this curve|Curve|
|contains(tenor)|Checks to see if this curve contains the supplied relative tenor|Boolean|
|convert(currency, units)|Convert this curve using currency and units|Curve|

Full example using properties and methods:

```js
eombus = ExpiryCalendar(BusinessCalendar())
eombus.addRule("go to the end of the previous month")
ondate = CurveDate(Date("2020-10-13"), eombus)
curve = Curve(ondate)

curve.currency = "EUR"
curve.units = "MWH"

curve.category = "EUROPEAN POWER"

curve.add(Contract(ondate, "2020M11", 25.67))
curve.add(Contract(ondate, "2020M12", 25.75))
curve.add(Contract(ondate, "2021Q01", 25.8))
curve.add(Contract(ondate, "2021Q02", 25.83))
curve.add(Contract(ondate, "2021Q03", 25.85))
curve.add(Contract(ondate, "2021Q04", 25.89))
curve.add(Contract(ondate, "2022Y", 25.99))

print json(curve)
```


---

<!-- source: product/developer/odsl/Variables/CurveDate.md -->

CurveDate
=========================

A CurveDate is used as the valuation date of a curve. It is a date with an expiry calendar

## Construction

A CurveDate is constructed using a date and an expiry calendar as follows:
```js
ondate = CurveDate(Date, ExpiryCalendar)
```
Here is an example of constructing a curve date:
```js
eombus = ExpiryCalendar(BusinessCalendar())
eombus.addRule("go to the end of the previous month")
ondate = CurveDate(Date("2020-10-13"), eombus)
```
In the above example:

1.  We create a new ExpiryCalendar using a business calendar    
2.  Add an expiry rule to go to the end of the previous month    
3.  Create a CurveDate for a specific date and using the expiry calendar we used above
    
## Properties

A CurveDate has the following properties:

|**Name**|**Description**|**Type**|
|-|-|-|
|dow|The name of the day of the week, e.g. Monday|Scalar(String)|
|day|The day of the month|Scalar(Integer)|
|month|The month of the year (1-12)|Scalar(Integer)|
|monthName|The name of the month, e.g. January|Scalar(String)|
|year|The year|Scalar(Integer)|
|hour|The hour of the day (0-23)|Scalar(Integer)|
|minute|The minute of the hour (0-59)|Scalar(Integer)|
|second|The second of the minute (0-59)|Scalar(Integer)|
|calendar|The expiry calendar associated with this curve date (readonly)|ExpiryCalendar|
|ISOYear|Get the ISO week based year|String|
|halfHourlyTenor|Get the absolute half hourly tenor from this date|String|
|hourlyTenor|Get the absolute hourly tenor from this date|String|
|millis|in millis|String|
|quarter|Get the quarter|String|
|quarterHourlyTenor|Get the absolute quarter hourly tenor from this date|String|
|timezone|The timezone of this date|String|
|week|Get the week of the year|String|
|weekOfMonth|Get the week of the year|String|

Example usage of properties:
```js
// Creates a new curve date variable
eombus = ExpiryCalendar(BusinessCalendar())
eombus.addRule("go to the end of the previous month")
ondate = CurveDate(Date("2020-10-13"), eombus)

print datetime.hour
print datetime.minute
print datetime.second
```

## Methods

The date variable has the following methods:

|**Name**|**Description**|**Returns**|
|-|-|-|
|next()|Returns the next calendar day after this one|Date|
|next(Calendar)|Returns the next day after this one using the supplied calendar|Date|
|previous()|Returns the previous calendar day before this one|Date|
|previous(Calendar)|Returns the previous day before this one using the calendar|Date|
|isHoliday(Calendar)|Returns true is this date is a holiday in the supplied calendar|Scalar(Boolean)|
|getExpiry(Period)|Gets the expiry date for the supplied period code|Date|
|dayOfMonth(day)|Returns a new Date as the specific day of the month|Date|
|dayOfMonth(day, Calendar)|Returns a new Date as the specific day of the month using the supplied calendar|Date|
|endOfDay()|Returns a new Date as the end of the day|Date|
|endOfMonth()|Returns a new Date as the end of the month|Date|
|endOfMonth(Calendar)|Returns a new Date as the end of the month using the supplied calendar|Date|
|getEnd(period)|Returns the end date for the supplied period code|Date|
|getStart(period)|Returns the start date for the supplied period code|Date|
|inTimezone(timezone)|Returns this date/time in the supplied timezone|Date|
|isAfter(other)|Returns true if this date is after the supplied date|String|
|isBefore(other)|Returns true if this date is before the supplied date|String|
|startOfDay()|Returns a new Date as the start of the day|Date|
|startOfMonth()|Returns a new Date as the start of the month|Date|
|startOfMonth(Calendar)|Returns a new Date as the start of the month using the supplied calendar|Date|

Example using methods:
```js
// Create a curve date for the 13th October 2020
eombus = ExpiryCalendar(BusinessCalendar())
eombus.addRule("go to the end of the previous month")
ondate = CurveDate(Date("2020-10-13"), eombus)

// Get the expiry date for the December 2020
print ondate.getExpiry("2020M12")
```


---

<!-- source: product/developer/odsl/Variables/CurveSeries.md -->

# CurveSeries

A **CurveSeries** is a curve held as a timeseries of delivery periods: one value for each period in the calendar, for example each hour of a power curve, with the curve date it belongs to. It has all the properties and methods of a [TimeSeries](https://doc.opendatadsl.com/docs/odsl/variable/timeseries).

## Construction

|**Function**|**Description**|
|-|-|
|CurveSeries(calendar, ondate)|Creates a new Curve Series|
|CurveSeries(calendar, ondate, data)|Creates a new Curve Series|
|CurveSeries(calendar, ondate, start, data)|Creates a new Curve Series|


---

<!-- source: product/developer/odsl/Variables/Date.md -->

Date
====================

A date variable is a representation of a Date or a Date and Time in the UTC timezone

## Construction

You can construct a date using a constructor function
```js
// Creates a new date variable with the current date with no time
today = Date()

// Creates a new date variable for the 8th October 2020
date = Date("2020-10-08")

// Creates a new date variable for the 8th October 2020 at 12:46
datetime = Date("2020-10-08T12:46:00")

/**
 Creates a new date variable using the start of the delivery period
 for October 2020 (2020M10) i.e. 1st October 2020
 **/
datepc = Date("2020M10")

// Creates a new date variable for the 8th October 2020 using a different format
df = Date("06/10/2020", "dd/MM/yyyy")

You can also use the [date active variable](https://opendatadsl.atlassian.net/wiki/spaces/DOCUMENTAT/pages/32971/Date+Service) to construct a date, e.g.

// Creates a new date variable for the next Monday (or today if today is Monday)
dav = ${date:"monday"}
```

### Constructor functions

|**Function**|**Description**|
|-|-|
|Date()|Creates a new Date variable as the current date|
|Date(date)|Creates a new Date variable|
|Date(date, format)|Creates a new Date variable|
|ISODate(date)|Creates a new Date variable from the supplied ISO Date formatted string|
|TimeStamp()|Creates a new Date variable as the current date and time|

## Properties

A date has the following properties:

|**Name**|**Description**|**Type**|
|-|-|-|
|dow|The name of the day of the week, e.g. Monday|Scalar(String)|
|day|The day of the month|Scalar(Integer)|
|month|The month of the year (1-12)|Scalar(Integer)|
|monthName|The name of the month, e.g. January|Scalar(String)|
|year|The year|Scalar(Integer)|
|hour|The hour of the day (0-23)|Scalar(Integer)|
|minute|The minute of the hour (0-59)|Scalar(Integer)|
|second|The second of the minute (0-59)|Scalar(Integer)|
|ISOYear|Get the ISO week based year|String|
|halfHourlyTenor|Get the absolute half hourly tenor from this date|String|
|hourlyTenor|Get the absolute hourly tenor from this date|String|
|millis|in millis|String|
|quarter|Get the quarter|String|
|quarterHourlyTenor|Get the absolute quarter hourly tenor from this date|String|
|timezone|The timezone of this date|String|
|week|Get the week of the year|String|
|weekOfMonth|Get the week of the year|String|

Example usage of properties:
```js
// Creates a new date variable for the 8th October 2020 at 12:46
datetime = Date("2020-10-08T12:46:00")
print datetime.hour
print datetime.minute
print datetime.second
```

## Methods

The date variable has the following methods:

|**Name**|**Description**|**Returns**|
|-|-|-|
|next()|Returns the next calendar day after this one|Date|
|next(Calendar)|Returns the next day after this one using the supplied calendar|Date|
|previous()|Returns the previous calendar day before this one|Date|
|previous(Calendar)|Returns the previous day before this one using the calendar|Date|
|startOfMonth()|Returns a new Date as the start of the month|Date|
|startOfMonth(Calendar)|Returns a new Date as the start of the month using the supplied calendar|Date|
|endOfMonth()|Returns a new Date as the end of the month|Date|
|endOfMonth(Calendar)|Returns a new Date as the end of the month using the supplied calendar|Date|
|dayOfMonth(day)|Returns a new Date as the specific day of the month|Date|
|dayOfMonth(day, Calendar)|Returns a new Date as the specific day of the month using the supplied calendar|Date|
|isHoliday(Calendar)|Returns true is this date is a holiday in the supplied calendar|Scalar(Boolean)|
|endOfDay()|Returns a new Date as the end of the day|Date|
|inTimezone(timezone)|Returns this date/time in the supplied timezone|Date|
|isAfter(other)|Returns true if this date is after the supplied date|String|
|isBefore(other)|Returns true if this date is before the supplied date|String|
|startOfDay()|Returns a new Date as the start of the day|Date|

Example using methods:
```js
// Create a date for the 5th October 2020
date5oct = Date("2020-10-05")

// Get the previous calendar day
date4oct = date5oct.previous()

/**
 Checks if the 4th October 2020 is a holiday according to the 
 business (Monday to Friday) calendar
**/
print date4oct.isHoliday("business")

// Prints out the previous date according to the business calendar
print date5oct.previous("business")
```

## Operations

Dates can be used in mathematical operations, for example subtracting one date from another produces a Duration variable. Here are a list of valid date operations:

### Subtraction

Subtracting one date from another creates a duration, e.g.
```js
d1 = Date("2020-06-08")
d2 = ${date:"2020-06-09"}

// Test for date subtraction
dur1D = d2 - d1
assertEquals("1D", dur1D)
```

Subtracting a number from a date produces a new date with that number of days subtracted, e.g.

```js
// Test for date and numeric subtraction
datesub1 = Date("2020-10-08") - 1
assertEquals("2020-10-07T00:00:00", datesub1)
```

Subtracting a duration from a date produces a new date with the duration subtracted from it, e.g.

```js
// Test for date and duration subtraction
datesub2 = Date("2020-10-08") - 1M1D
assertEquals("2020-09-07T00:00:00", datesub2)
```

### Addition

Adding a duration to a date creates a new date, e.g.
```js
// Test for date and duration addition
datedur = Date("2020-10-08") + 1D
assertEquals("2020-10-09T00:00:00", datedur)
```

Adding a number to a date creates a new date with that number of days added, e.g.
```js
// Test for date and numeric addition
datenum = Date("2020-10-08") + 5
assertEquals("2020-10-13T00:00:00", datenum)
```

## More Examples
### Start of the month

```js
//#region Set start of month
d1 = Date("2023-10-15")
d2 = d1.startOfMonth()
print d2

d1 = Date("2023-10-15")
d2 = d1.startOfMonth("BUSINESS")
print d2
//#endregion
```

### End of the month

```js
//#region Set end of month
d1 = Date("2024-03-15")
d2 = d1.endOfMonth()
print d2

d1 = Date("2024-03-15")
d2 = d1.endOfMonth("BUSINESS")
print d2
//#endregion
```

### Day of the month

```js
//#region Day of the month
d1 = Date("2024-05-15")
d2 = d1.dayOfMonth(11)
print d2

d1 = Date("2024-05-15")
d2 = d1.dayOfMonth(11, "BUSINESS")
print d2
//#endregion
```


---

<!-- source: product/developer/odsl/Variables/DeclaredType.md -->

DeclaredType
============================

A DeclaredType variable is a wrapper for a type and is used by the type service and creating objects.

## Construction
You declare a type using a structure containing the type definition and takes the following form:

```js
typename = (versioned)? type (extends typename)?
    (comment)?
    
    // Property
    name as type(qualifier?) (not null)? (default expression)? (bucket bucketDefinition)?
    
    // Smart Data
    name as smart (timeseries|curve) (with)? expression
    
    // Constraint
    constraint name (check|unique) (mark)? (condition (, condition)* | profile for checkname(param (,param)*))
    
end
```

### Adding property definitions
A property definition declares that this type has a specifically named property of a certain type.
The type can be one of the following:

|Type|Description|
|-|-|
|name|A special type declaring that this field is the default property to be used as the *name* of the object|
|description|A special type declaring that this field is the default property to be used as the *description* of the object|
|dimension|A special type declaring that this field is to be used as a filter property when navigating objects of this type|
|string|A scalar string property|
|number|A scalar numeric property|
|date|A scalar date or timestamp property|
|boolean|A scalar boolean (true|false) property|
|list|A list of variables|
|object|A dynamic object|
|duration|A duration variable|
|geometry|A geometry variable|
|timeseries|A timeseries|
|curve|A curve|

#### Not null
If you add the 'not null' modifier on the property definition, OpenDataDSL will not allow null values for this property.

#### Default
A way of counteracting the 'not null' modifier is to use a 'default' modifier that sets a default value for a property if it doesn't have a value.
The default value can be a scalar value or can be an expression that looks up a value in other objects.

##### Examples
```js
address = type
    name as String() not null
    type as Dimension()
    typeName as String() default ${object:"abc_metadata"}[type].name
end
```

#### Bucket
You can define a bucketing scheme for a property using the 'bucket' modifier, this takes the form:

```js
bucket (boundaries array|buckets integer (granularity (definition)?)
```

e.g.

```js
...
year as Dimension(number) not null bucket boundaries [1980, 1990, 2000, 2010, 2020, 2030]
...
```

### Adding smart data properties
Smart data properties allow you to create smart timeseries or curves which will be dynamically added to all objects that directly inherit this type.

For example, if you have an ASK and BID timeseries on the objects of a certain type, you can add a MID smart timeseries that dynamically calculates the mid point between the ASK and BID timeseries.

```js
...
MID = smart timeseries with (ASK+BID)/2
...
```

### Adding constraints
You can add constraints on the objects on a type that will help to improve the quality of your data.

```constraint name (check|unique) (mark)? (condition (, condition)* | for checkname(param (,param)*))```

#### name
The name of the constraint

#### check or unique
Check defines that this is a quality check type constraint and unique adds a unique index such that no 2 documents can be added with the same unique combination of fields

#### mark
Normally if a constraint catches a quality failure, the update will be rejected. If you add the 'mark' modifier, the object is marked with a status and message that the constraint check failed.

#### timeseries checks
You can perform tests for the following quality issues:

* Missing data using ```for missing()```

#### Examples
```js
constraint type_valid check (type in ${object:"abc_metadata"}.types)
constraint zip_valid check mark (zip matches "^\d{5}(-\d{4})?$")
constraint hl_valid check mark (hl in ["high","low"])
constraint zip_index unique (zip,hl)
```

## Properties

|**Name**|**Description**|**Type**|
|-|-|-|
|allowLinks|If true, we are allowed to add links to the objects of this type|Boolean|
|event|True if this type is used to create events|Boolean|
|mergeEvent|True if you want to merge events rather than replace them|Boolean|
|showId|Adds the id to the stored entity, defaults to true|Boolean|
|showStatus|Adds status to the stored entity, defaults to true|Boolean|
|showType|Adds the type to the stored entity, defaults to true|Boolean|
|system|True if this is a system managed type|Boolean|


---

<!-- source: product/developer/odsl/Variables/Documentation.md -->

# Documentation

A **Documentation** variable holds documentation for an item or a service: markdown content, JSON with a template to render it, or a link to a web page. It is saved with the [documentation service](https://doc.opendatadsl.com/docs/api/rest/service/documentation).

## Construction

|**Function**|**Description**|
|-|-|
|Documentation()|Creates a new default documentation configuration|
|DocumentationLink(url)|Creates a new documentation URL Link configuration|
|DocumentationLink(url, selector)|Creates a new documentation URL Link configuration|

## Properties

|**Name**|**Description**|**Type**|
|-|-|-|
|content|The markdown content of this documentation|String|
|documentationType|The type of this documentation, one of Default, JSON or Link|String|
|entity|The entity or id this documentation is for|String|
|json|JSON content - only use if this is a JSON type|Object|
|language|The language of this documentation (optional - default to English en)|String|
|name|The name of this documentation|String|
|scope|The scope or service this documentation is for|String|
|selector|The optional selector used with the Web URL to select part of a web page - only if this is a link type|String|
|tags|Tags for this documentation|List|
|template|Template to convert JSON to HTML - only use if this is a JSON type|String|
|url|The Web URL of this documentation - only if this is a link type|String|

## Example

```js
doc = Documentation()
doc.entity = "NGE.GAS.TTF"
doc.scope = "object"
doc.name = "TTF gas prices"
doc.content = "## TTF\nSettlement prices for the Dutch TTF gas hub."
save ${documentation:doc}
```


---

<!-- source: product/developer/odsl/Variables/Duration.md -->

Duration
========================

A duration variable represents a period of time such as 3 days or 4 and a half minutes in [ISO 8601 duration format](https://doc.opendatadsl.com/docs/kb/duration)

## Construction

A duration can be constructed using a code or by subtracting 2 dates
```js
// Construct a duration of 1 day
oneday = Duration("1D")

// Subtract 2 dates to create a duration
d1 = Date("2020-06-08")
d2 = Date("2020-06-09")
dur1D = d2 - d1
```

### Duration code

A duration code is comprised of a list of periods of time defined in the table below:

|**Period**|**Description**|
|-|-|
|s|Seconds|
|m|Minutes|
|h|Hours|
|D|Days|
|W|Weeks|
|M|Months|
|Y|Years|

Each period has a number and a period code, e.g. 1D is 1 day

Here are some examples:

|**Code**|**Amount of time**|
|-|-|
|1D|1 day|
|4h30m|4 and 1/2 hours|
|60h|60 hours|
|2M|2 months|
|1Y2M12D6h30m|1 year, 2 months, 12 days, 6 hours and 30 minutes|
|12W|12 weeks|

## Properties

The following is a list of properties on a duration variable:

|**Name**|**Description**|**Type**|
|-|-|-|
|years|The number of defined years in this duration|Scalar(Integer)|
|months|The number of defined months in this duration|Scalar(Integer)|
|days|The number of defined days in this duration|Scalar(Integer)|
|hours|The number of defined hours in this duration|Scalar(Integer)|
|minutes|The number of defined minutes in this duration|Scalar(Integer)|
|seconds|The number of defined seconds in this duration|Scalar(Integer)|
|durationCalendar|Get a calendar representing this duration|Calendar|


---

<!-- source: product/developer/odsl/Variables/Event.md -->

Event
======

An event variable is an object that has happened or will happen at a point in time or over a period of time.

## Construction
An event can be constructed in one of 2 ways

### Standard construction

This creates an event with an eventtime specified as the constructor argument.

```js
evtemp = Event(${date:"now"})
``` 

### Curve contract construction

This creates an event with all the properties of a curve contract (ondate, absolute, relative, expiry, start and end of delivery)

```js
cd = CurveDate("2024-02-02", "#REOMHENG")
e = Event(cd, "2024M06")
```

## Properties
An event has the following properties:

|**Name**|**Description**|**Type**|
|-|-|-|
|eventtime|The time of the event message|String|
|eventstart|The point in time when the event started/starts - defaults to the time of the event message|String|
|eventend|The point in time when the event ended/ends - default to the eventstart|String|
|timezone|The timezone of the event message|String|
|eventtype|The type of event - defaults to #Event|String|
|event|The identifier for this group of events|String|
|eventid|The unique identifier for this specific event|String|
|*|Any other properties|Any|
|eventlist|The non-object specific name of this group of events|String|
|source|The source of this event, usually null|String|
|status|The status of this event|Status|

## Methods
An event has the following methods:

|**Name**|**Description**|**Return Type**|
|-|-|-|
|createId()|Creates a unique id for this event based on the event start and end time|Void|

## Example Usage
An event or list of events are always associated with a Master Data Record (Object)

### Saving events

```js
evtemp = Event(${date:"now"})
evtemp.eventtype = "#Metric"
evtemp.type = "temperature"
evtemp.value = 14
evtemp.id = ObjectId()
evhum = Event(${date:"now"})
evhum.eventtype = "#Metric"
evhum.type = "humidity"
evhum.value = 25
evhum.id = ObjectId()
s9754 = ${object:"s9754"}
s9754.MESSAGE = [evtemp, evhum]
save ${object:s9754}
```

## Further Reading
* [Event service](https://doc.opendatadsl.com/docs/odsl/service/event) for saving and using events in the server.


---

<!-- source: product/developer/odsl/Variables/EventCurve.md -->

EventCurve
=====

An EventCurve is a curve that is dynamically built from events.

## Introduction

An EventCurve is a very flexible and dynamic way of expressing point-in-time based future values, both real and forecasted from events that are stored in your private database, the common database and the public database.

## Construction

To construct a new EventCurve, you use the EventCurve function as shown in the syntax below:
```js
curve = EventCurve(event, calendar, property, tenor)
```
An example of constructing a new EventCurve:
```js
curve = EventCurve("AAA:EVENTS", "#REOMHENG", "price", "absolute")
```

## Properties

An EventCurve has the following properties:

|**Name**|**Description**|**Type**|
|-|-|-|
|event|The name of the eventlist that this Curve uses (in the form objectId:eventListId)|String|
|property|The name of the property in the event that this Curve uses as the value|String|
|tenor|The name of the property in the event that this Curve uses as the tenor/maturity code|String|
|calendar|The expiry calendar for this Curve|String|
|holidayCalendar|The holiday calendar for this Curve - this is used to determine what dates this curve should be published on|String|
|filter|Optional filter used to filter the events used to build this Curve|String|
|source|Optional source of the events used to build this Curve. Can be private, common or public, defaults to private|String|
|valueTransformation|Optional value transformation expression to change all values in the curve, e.g. 'value * 1.1'|String|
|currency|An optional currency code|String|
|units|An optional units code|String|
|properties|Any other properties you want to add onto the curve|Object|
|precision|The [precision](https://doc.opendatadsl.com/docs/kb/precision#data-precision-settings) configuration for this curve|Precision|
|include|Additional properties from an event to add as columns in the curve contracts|String or Array| 
|contracts|Get the forward contracts on this curve|Contracts|
|description|A description of the curve|String|
|identity|The configured identity for this curve|Object|
|name|The name of the curve|String|
|ondate|The date for which this curve corresponds to|CurveDate|
|timezone|The timezone for this curve|String|

## Methods

|**Name**|**Description**|**Return Type**|
|-|-|-|
|addNew(contract)|Adds the supplied contract to this curve only if the tenor doesn't already exist on this curve|Curve|
|addNew(contracts)|Adds the supplied contracts to this curve only if the tenor doesn't already exist on this curve|Curve|
|build()|Builds this Event Curve|Curve|
|contains(tenor)|Checks to see if this curve contains the supplied relative tenor|Boolean|
|convert(currency, units)|Convert this curve using currency and units|Curve|
|size()|The number of contracts on this curve|String|

## Examples

#### Creating and saving an event curve
```js
EC = EventCurve("AAA:EVENTS", "#REOMHENG", "price", "relative")
EC.precision.scale = 8
AAA = Object()
AAA.EC = EC
save AAA
```

#### Using value transformation
```js
EC = EventCurve("AAA:EVENTS", "#REOMHENG", "price", "relative")
EC.valueTransformation = "value * 1.1"
```

#### Additional properties
You can include additional columns in the curve from event properties, using the include property, e.g.

```js
EC = EventCurve("AAA:EVENTS", "#REOMHENG", "price", "relative")
EC.include = "vendorId"
```

To include multiple additional columns use an array, e.g.

```js
EC = EventCurve("AAA:EVENTS", "#REOMHENG", "price", "relative")
EC.include = ["vendorId", "internalId"]
```

## Calendars
When dynamically retrieving the historic timeseries for absolute or relative tenors, the calendar for that timeseries is determined using the following logic:

* If a property exists with the name of the tenor, e.g. *2025M01* and the value of that property is a calendar code, that will be used.
* If a holidayCalendar is set on the event curve, that will be used.
* The underlying trading calendar on the expiry calendar will be used.

:::info
**Functionality change since January 2025 (Jan25 release)**

Previously the holidayCalendar property was used if present, otherwise the SPARSE calendar would be used
:::

## See Also
* Curve


---

<!-- source: product/developer/odsl/Variables/EventOptions.md -->

EventOptions
=====

An EventOptions is options data (an option chain for each maturity) that is dynamically built from events.

## Introduction

An EventOptions says which events hold the options data and how to read them: one event for each ondate, tenor, call or put, and strike. It is saved as a data item of an object, and the options are built from the events each time you read them. With `positions` set it becomes a positions view, which returns only the options you hold. See the [Options topic](https://doc.opendatadsl.com/docs/topics/options/intro).

## Construction

To construct a new EventOptions, use the EventOptions function:
```js
options = EventOptions(eventlist)
```
An example of constructing EventOptions on the ICE Endex EUA futures options:
```js
options = EventOptions("ICE.NDEX.EFO:OPTIONS")
options.source = "common"
```

## Properties

An EventOptions has the following properties:

|**Name**|**Description**|**Type**|
|-|-|-|
|event|The name of the eventlist that these options use (in the form objectId:eventListId)|String|
|source|Optional source of the events: private, common or public (default private, public for # event lists)|String|
|filter|Optional filter (JSON) used to filter the events|String|
|tenor|The name of the property in the event with the tenor/maturity code of the option (default absolute)|String|
|strike|The name of the property in the event with the strike price (default STRIKE_PRICE)|String|
|callPut|The name of the property in the event that says if the option is a call or a put (default CONTRACT_TYPE)|String|
|callValue|The value of the callPut property for a call option (default C)|String|
|putValue|The value of the callPut property for a put option (default P)|String|
|value|The main value column, used when no column is given (default SETTLEMENT_PRICE)|String|
|columns|The event properties shown in the strike table (default SETTLEMENT_PRICE, OPTION_VOLATILITY, DELTA_FACTOR, OPEN_INTEREST, TOTAL_VOLUME)|List or String|
|timezone|The timezone of the ondates (default UTC)|String|
|calendar|The optional expiry calendar of the options|String|
|holidayCalendar|The optional holiday calendar, the calendar of the single option timeseries (default daily)|String|
|currency|An optional currency code|String|
|units|An optional units code|String|
|name|The name of the options|String|
|description|A description of the options|String|
|positions|Optional eventlist with the positions held; when set only the held options are returned|String|
|positionSource|The source of the position events: private (default), common or public|String|
|positionFilter|Optional filter (JSON) used to filter the position events, e.g. on a book or product|String|
|positionTenor|The property in the position events with the tenor, absolute or relative (default tenor)|String|
|positionStrike|The property in the position events with the strike price (default strike)|String|
|positionCallPut|The property in the position events that says if it is a call or a put (default callput)|String|
|positionVolume|The property in the position events with the volume held, negative for a short position (default volume)|String|
|positionPrice|The property in the position events with the traded price (default price)|String|
|identity|The configured identity for these options|Object|
|properties|Any other properties you want to add onto the options|Object|

## Methods

|**Name**|**Description**|**Return Type**|
|-|-|-|
|build(ondate)|Builds these options for an ondate on the server; in your own scripts read `${data:"ID:YYYY-MM-DD"}` instead|Options|
|ondates()|Gets the ondates these options can be built for (on the server)|OptionsDates|

## Reading the options

|**Id**|**What you get**|
|-|-|
|`OBJ:ID`|The ondates (OptionsDates)|
|`OBJ:ID:YYYY-MM-DD`, `OBJ:ID:L`|The [Options](https://doc.opendatadsl.com/docs/odsl/variable/options) for an ondate, or the latest|
|`OBJ:ID:TENOR:STRIKE:C`|A [TimeSeries](https://doc.opendatadsl.com/docs/odsl/variable/timeseries) of the call's main value, one value per ondate (P for the put)|
|`OBJ:ID:TENOR:STRIKE:C:COLUMN`|A TimeSeries of one column of the call|

## Examples

#### Creating and saving event options
```js
o = EventOptions("ICE.NDEX.EFO:OPTIONS")
o.name = "EUA futures options"
o.source = "common"
o.columns = ["SETTLEMENT_PRICE", "OPTION_VOLATILITY", "DELTA_FACTOR"]
NGE = Object()
NGE.id = "NGE.EUA"
NGE.OPTIONS = o
save NGE
```

#### Reading the options for an ondate
```js
eua = ${data:"NGE.EUA:OPTIONS:2026-09-16"}
print eua["2026M09"][85].call.SETTLEMENT_PRICE
```

#### A positions view
```js
p = EventOptions("ICE.NDEX.EFO:OPTIONS")
p.source = "common"
p.positions = "NGE.EUA:POSITIONS"
NGE = Object()
NGE.id = "NGE.EUA"
NGE.POSITIONS_VIEW = p
save NGE
```
See [Traded options](https://doc.opendatadsl.com/docs/topics/options/traded-options) for uploading positions.


---

<!-- source: product/developer/odsl/Variables/EventTimeSeries.md -->

# EventTimeSeries

An **EventTimeSeries** is a timeseries built from the events in an event list, using one property of the events as the value. It is built when it is read, so it always reflects the latest events. See also [EventCurve](https://doc.opendatadsl.com/docs/odsl/variable/eventcurve).

## Construction

|**Function**|**Description**|
|-|-|
|EventTimeSeries(eventlist, property)|Creates a new Event TimeSeries using an EventList and the name of the property to use for the TimeSeries values|

## Properties

|**Name**|**Description**|**Type**|
|-|-|-|
|calendar|The calendar that this timeseries uses|Calendar|
|currency|The ISO currency of this timeseries|String|
|description|The description of this timeseries|String|
|event|The name of the eventlist that this TimeSeries uses (in the form objectId:eventListId)|String|
|filter|Optional filter used to filter the events used to build this TimeSeries|String|
|identity|The configured identities|Object|
|index|Optional date field to use for the index used to build this TimeSeries, defaults to start|String|
|name|The name of the timeseries|String|
|observed|The observed setting of the timeseries used for aggregating|TimeSeriesObserved|
|precision|The precision configuration for this timeseries|Precision|
|property|The name of the property in the event that this TimeSeries uses as the value|String|
|source|The source of this timeseries|String|
|start|The starting date/time of the timeseries|Date|
|tenor|The tenor of this timeseries|String|
|timezone|The timezone of this timeseries, defaults to UTC|String|
|units|The ISO units of this timeseries|String|
|valueType|The value type of the timeseries (TRACKED or BASIC)|TimeValueType|

## Methods

|**Name**|**Description**|**Return Type**|
|-|-|-|
|build()|Builds this Event TimeSeries|TimeSeries|

It also has all the methods of a [TimeSeries](https://doc.opendatadsl.com/docs/odsl/variable/timeseries).

## Example

```js
obj = ${object:"NGE.TRADES"}
obj.FILL_PRICE = EventTimeSeries("NGE.TRADES:FILLS", "price")
save obj
```


---

<!-- source: product/developer/odsl/Variables/Executable.md -->

Executable
==========================

An executable wrapper variable holds either an expression of a method


---

<!-- source: product/developer/odsl/Variables/Expiry-Calendar.md -->

Expiry Calendar
========================================

An expiry calendar variable type represents an Expiry Calendar which is a special type of calendar that uses rules to calculate when trading stops for a future price for a particular commodity

## Introduction

The expiry calendar variable wraps an [expiry calendar](https://doc.opendatadsl.com/docs/odsl/calendar/expiry) and allows you to use them in your scripts.

## Construction

Expiry calendars are usually read in from the [expiry calendar service](https://doc.opendatadsl.com/docs/odsl/service/expiry) using their code, but can also be constructed using the ExpiryCalendar function. Below are a few examples of constructing an expiry calendar:

```js
// Creating an expiry calendar using a standard business calendar
eombus = ExpiryCalendar(BusinessCalendar())

// Add an expiry rule to go to the end of the previous month as the expiry date
eombus.addRule("go to the end of the previous month")

// Read an expiry calendar from the expiry calendar service
eomeng = ${expiry:"#EOMENG"}
```

### Constructor functions

|**Function**|**Description**|
|-|-|
|ExpiryCalendar(code)|Reads an expiry Calendar|
|ExpiryCalendar(holiday)|Creates a new expiry Calendar|
|ExpiryCalendar(holiday, delivery)|Creates a new expiry Calendar|
|FinancialExpiryCalendar(code, holiday, cross, base, offset)|Creates a financial expiry Calendar|

## Methods

An expiry calendar variable has the following methods:

|**Name**|**Description**|**Return Type**|
|-|-|-|
|addRule(String)|Adds the supplied expiry rule string to this expiry calendar|Void|

See [Expiry Calendar](https://doc.opendatadsl.com/docs/odsl/calendar/expiry) for full details about the rules that can be added to an expiry calendar

## Properties

|**Name**|**Description**|**Type**|
|-|-|-|
|name|Get the name of the underlying ODSL Calendar|String|


---

<!-- source: product/developer/odsl/Variables/Extension.md -->

# Extension

An extension variable is the definition of an ODSL platform extension.

## Construction
An extension is created using the **Extension()** function, e.g.

```js
e = Extension()
```

## Properties
An extension has the following properties:

|**Name**|**Description**|**Type**|
|-|-|-|
|_id|Unique id for the extension (must be \{publisher\}.\{code\}|String|
|publisher|The publisher of the extension|String|
|code|The code id of the extension|String|
|name|The name of the extension|String|
|description|The long description of the extension to be shown in the extension catalog|String|
|documentation|A URL to some external documentation|String|
|icon|The icon (bootstrap icons) to be shown next the extension name in the catalog|String|
|views|An array of views to be shown in the portal or excel add-in|ExtensionView[]|
|resources|A set of resources used by this extension that will be published/installed|Object (See Below)|
|currency|The currency for the price of this extension|String|
|price|The price of this extension (0 for free)|String|
|tags|A list of tags that are used to categorise this extension|List|

### Extension View

The Extension View entity is used to configure the views in the portal and excel add-in and contains the following:

|**Name**|**Description**|**Type**|
|-|-|-|
|id|The unique identifier for this view in the extension|String|
|section|The section in the menu to display this view, defaults to Extension|String|
|name|The name of the view as shown in the menu|String|
|description|A description of the view|String|
|tabname|The name of the tab, defaults to 'Main'|String|
|appname|The name of the view in the Apps catalogue, only used if the tabname is 'Apps'|String|
|excel|True if this is an Excel Add-in view, defaults to false|Boolean|
|icon|The icon shown next to the name in the GUI|String|
|feature|The feature policy used to determine if a user can see this view|String|
|script|The mustache script containing the code for this view|String|
|insights|Set this to true if this app is a new menu item and you want to show an Insights tab|Boolean|

### Resources

Extension resources are a list of entities which need to be installed with the extension.
Resources are a SimpleObject, with each key in the document being a service name.
The value of the service can either be key=values or an array, e.g.

```js
e.resources.script.uninstall = "example-odsl\examples\Extension\odsl.example.uninstall"
e.resources.policy = ["extension.odsl.example"]
```

## Methods
An extension has the following methods:

|**Name**|**Description**|**Return Type**|
|-|-|-|
|addView(name)|Adds a view to an extension and returns the new view|ExtensionView|

## Examples

### Creating a simple extension

```js
e = Extension()
e.id = "odsl.example"
e.publisher = "odsl"
e.code = "example"
e.name = "Example Extension"

e.resources.script.install = "example-odsl\examples\Extension\odsl.example.install"
e.resources.script.upgrade = "example-odsl\examples\Extension\odsl.example.upgrade"
e.resources.script.uninstall = "example-odsl\examples\Extension\odsl.example.uninstall"
e.resources.policy = ["extension.odsl.example"]

v = e.addView("test")
v.excel = false
v.feature = "odsl.example.test"
v.icon = "graph-up-arrow"
v.script = "example-odsl\examples\Extension\odsl.example.test"
v.section = "Examples"

save e
```

### Publishing an extension

```js
e = ${extension:"odsl.example"}
publish e to "public"
```

### Installing an extension

```js
e = ${extension:public/"odsl.example"}
publish e to "private"
```

### Uninstalling an extension

```js
delete ${extension:"odsl.example"}
```

:::warning
If you run the command to uninstall an extension that has been created by you, it will be completely removed and would need to be re-created if needed
:::

## Further Reading
* [Extension service](https://doc.opendatadsl.com/docs/odsl/service/extension) for saving and using extensions.


---

<!-- source: product/developer/odsl/Variables/Extractor.md -->

Extractor
=========================

A wrapper for an extractor, used by the extractor service

## Properties

|**Name**|**Description**|**Type**|
|-|-|-|
|category|The category of this script, e.g. ETL|String|
|description|A description of what this script does|String|
|filename|The filename of this script|String|
|outputFormat|An optional output format, e.g. application/xml, text/html, text/csv|String|
|script|The script text|String|
|scriptType|The type of this script, e.g. one of [odsl, mustache, py, css, html, js, mjs, cjs]|String|

## Methods

|**Name**|**Description**|**Return Type**|
|-|-|-|
|loadFrom(file)|Load the script from a local file|Void|
|run()|Runs this script|Void|
|run(globalContext)|Runs this script|Void|


---

<!-- source: product/developer/odsl/Variables/FeaturePolicy.md -->

A feature policy variable contains information about which features in the Web Portal that users can see

## Introduction

Feature Policy variables are used to restrict access to certain features within the platform, thereby focussing users on the tasks they need to perform.

:::caution
Feature policies only change what the Web Portal shows, they are not a security control. To restrict access to data, use [security policies](https://doc.opendatadsl.com/docs/odsl/variable/policy).
:::

For how feature policies are combined and more examples, see [Feature Policies](https://doc.opendatadsl.com/docs/topics/policies/feature).

## Defining a Feature Policy

Below is a table with all the properties of a feature policy:

|**Name**|**Type**|**Example**|**Description**|
|-|-|-|-|
|_id|String|RunAllReports|The unique id for this policy|
|category|String|TeamUsers|An optional category used for filtering policies|
|description|String|My test policy|A descriptive name for the policy|
|enabled|Boolean|true|True if it is enabled, false to disable it, by default the policy is enabled|
|deny|Boolean|true|By default, all features are available to everyone, so you need to set this to true to deny access to features|
|members|List|user@company.com|The user emails or [policy user groups](https://doc.opendatadsl.com/docs/topics/policies#policy-user-groups) this policy applies to, or * for everyone (Azure Active Directory group id's are not used by feature policies)|
|features|List|\["scripts"\]|The list of features that this policy applies to|

## Methods

A policy has the following methods:

|**Name**|**Description**|**Return Type**|
|-|-|-|
|addMember(name)|Adds a member to this policy|void|
|removeMember(name)|Removes a member from this policy|void|

Example policy definition:
```json
{
  "_id": "HideScripts",
  "category": "TradingTeam",
  "description": "Hide Scripts menu item from users",
  "deny": true,
  "members": [
    "user@company.com"
  ],
  "features": [
    "scripts"
  ],
  "enabled": true
}
```

## Portal Feature Names
The following table shows the names of the features for each portal item:

|**Name**|**Portal Item**|
|-|-|
|masterData|Master Data|
|data|Data|
|events|Events|
|smartData|Smart Data|
|geoData|GEO Data|
|reports|Reports|
|curves|Curves|
|datasets|Datasets|
|processes|processes|
|executions|Executions|
|processTimeline|Process Timeline|
|dataPackages|Data Packages|
|queues|Queues|
|types|Types|
|calendars|Calendars|
|scripts|Scripts|
|dashboards|Dashboards|
|alerts|Alerts|
|groups|Groups|

Extensions can also add tabs and views with their own feature names, e.g. `odsl.aggregator.report` for the Aggregator tab on the Reports screen. Use these names in a feature policy in the same way.

## How Feature Policies are Combined

1. Start from the features switched on for your company.
2. Every enabled allow feature policy the user is a member of switches its features on.
3. Every enabled deny feature policy the user is a member of switches its features off.

Deny always wins. When your company is onboarded, a feature policy called `EnergyUserFeatures` gives everyone the standard menu items.

## Properties

|**Name**|**Description**|**Type**|
|-|-|-|
|tags|Tags to help filter and find policies|List|

## Updating, Finding and Deleting Feature Policies

### Saving a feature policy

To save a feature policy in OpenDataDSL code, use the save command as follows:
```js
HideScripts = FeaturePolicy()
HideScripts.description = "Hide Scripts menu item from users"
HideScripts.features = ["scripts"]
HideScripts.addMember("user:security:policy:BusinessUsers")
HideScripts.deny = true
HideScripts.enabled = true
save HideScripts
```

### Listing feature policies

To find feature policies, you use the ODSL find command, e.g.
```js
policies = find ${policy} where _type="FeaturePolicy"
```
You can use the unique keyword to just get all the ids of your policies, e.g.
```js
find unique _id from ${policy} where _type="FeaturePolicy"
```

### Retrieving a specific feature policy

To get a specific named feature policy, you use the policy active variable. 
You can then examine the information on the policy, e.g.
```js
p = ${policy:"HideScripts"}
print p.description
```

### Disabling a feature policy
You can disable a feature policy by setting the enabled flag to false, e.g.
```js
p = ${policy:"HideScripts"}
p.enabled = false
save p
```

### Deleting a feature policy

To delete a feature policy, you issue the delete command, e.g.
```js
delete ${policy:"HideScripts"}
```


---

<!-- source: product/developer/odsl/Variables/GeoSpatial.md -->

# GeoSpatial

GeoSpatial variables hold GeoJSON geometry: points, lines and polygons, used as properties of master data and in geospatial queries. Coordinates are given as [longitude, latitude]. See [Geospatial data and queries](https://doc.opendatadsl.com/docs/odsl/dm/geospatial).

## Construction

|**Function**|**Description**|
|-|-|
|GeometryCollection()|Creates a GeometryCollection geometry object|
|LineString()|Creates a LineString geometry object with the supplied list of coordinates|
|MultiLineString()|Creates an empty MultiLineString geometry object|
|MultiPoint()|Creates a MultiPoint geometry object with the supplied list of coordinates|
|MultiPolygon()|Creates a MultiPolygon geometry object with the supplied polygons|
|Point(longitude, latitude)|Creates a single coordinate point|
|Polygon()|Creates a Polygon geometry object with a multiple rings containing the supplied geometries|
|Sphere()|Creates a centerSphere geometry object used in geospatial queries|

**Sphere** creates the $centerSphere (centre and radius in radians) and $nearSphere (centre with minimum and maximum distance in metres) shapes used in geospatial queries.

## Example

```js
site = Object()
site.id = "NGE.SITE.EEMSHAVEN"
site.location = Point([6.83, 53.44])
save site
```


---

<!-- source: product/developer/odsl/Variables/Group.md -->

A group is a list of things such as favourites or export items.

## Introduction
A basic group contains a list of either static or dynamic things which can be used throughout the platform.

## Construction

### Constructor functions

|**Function**|**Description**|
|-|-|
|ApprovalGroup()|Creates a new curve approval group|
|BuildGroup()|Creates a new curve build group|
|ExportGroup()|Creates a new curve export group|
|Group()|Creates a new group|
|QualityGroup()|Creates a new curve quality group|
|TargetGroup()|Creates a new subscription target group|
|UserGroup()|Creates a new user group|

## Defining a Group
Below is a table with all the properties of a basic group:

|**Name**|**Type**|**Example**|**Description**|
|-|-|-|-|
|_id|ObjectId|67d3ee92196dab047409db1a|The unique auto-generated id for this group|
|service|String|data|The name of the service where the items for this group exist|
|type|String|favourite|The type of this group - can be anything, but there are some [special group types](#special-group-types)|
|category|String|Power|A category is used to categorise groups according to similar types|
|name|String|Netherlands Spot|The name of the group|
|description|String|Spot Data for the Netherlands|A description of the group|
|shared|Boolean|true|True if the group is accessible to everyone, false if only you can see it|
|items|Array|...|The list of items in this group (if it is static)|
|dynamic|Boolean|false|True if this is a dynamic group and the items are read using the dynamicOptions|
|dynamicOptions.service|String|object|The service used to get the list of id's, defaults to the group service|
|dynamicOptions.source|String|public|The source of the items, defaults to private|
|dynamicOptions.filter|String|_type='#ForeignExchange'|The filter used to dynamically find the items for this group|
|dynamicOptions.field|String|_id|The field for the id of the item, defaults to _id|
|dynamicOptions.objectfilter|String|_type='#ForeignExchange'|Use this to filter data based on master data properties|

## Managing groups in ODSL code

### Creating groups

#### Creating a static group

```js
bg = Group()
bg.name="My Items"
bg.category = "Data Lists"
bg.service = "object"
bg.type = "favourite"
bg.items = ["ICE.IFEU.B","ICE.NDEX.NLB"]
save bg
```

#### Creating a dynamic group

```js
bg = Group()
bg.name = "FX Data"
bg.category = "Data Lists"
bg.service = "object"
bg.type = "favourite"
bg.dynamicOptions.source = "public"
bg.dynamicOptions.filter = "_type='#ForeignExchange'"
save bg
```

#### Creating a dynamic group using an object filter

```js
bg = Group()
bg.name = "FX Data"
bg.category = "Data Lists"
bg.service = "data"
bg.type = "favourite"
bg.dynamicOptions.objectfilter = "source='ICE' and exchange='NDEX'"
bg.dynamicOptions.filter = "_type != 'VarCurve'"
save bg
```

### Listing groups

```js
find ${group} project type, category, name
```

### Getting a group

You can refer to a group either with its ```_id``` or using the format ```service:type:category:name```

#### Getting a group using the _id

```js
g = ${group:"67d3ee92196dab047409db1a"}
print g
```

#### Getting a group using type, category and name

```js
g = ${group:"data:favourite:Data Lists:FX Data"}
print g
```

### Adding items to a static group

Getting the group and adding the item to the items array

```js
g = ${group:"object:favourite:Data Lists:My Items"}
g.items.add("ICE.NDEX.NLP")
save g
```

### Removing an item from a static group

Getting the group, removing the item and saving the group with the replace option

```js
g = ${group:"object:favourite:Data Lists:My Items"}
g.items.remove("ICE.NDEX.NLP")
save g replace
```

Directly removing an item

```js
delete ${group:"object:favourite:Data Lists:My Items/items/ICE.NDEX.NLP"}
```

### Rolling back changes to a group

All changes to groups are tracked and versioned, you can rollback to a previous version of a group using the delete command

```js
delete ${group:"object:favourite:Data Lists:My Items"}
```

### Listing versions of a group

```js
v = ${group:"object:favourite:Data Lists:My Items":*}
print v
```

### Getting a specific version of a group

```js
v = ${group:"object:favourite:Data Lists:My Items":1}
print v
```

### Tagging a version of a group

You can tag a version of a group and directly refer to that version using the tag

```js
tag ${group:"object:favourite:Data Lists:My Items":1} as "TEST"
```

### Getting a version using the tag

```js
v = ${group:"object:favourite:Data Lists:My Items":TEST}
print v
```

### Deleting a group

To fully delete a group, use the version *

```js
delete ${group:"object:favourite:Data Lists:My Items":*}
```

## Methods

|**Name**|**Description**|**Return Type**|
|-|-|-|
|containsKey(key)|True if this object contains the provided key|Boolean|
|isEmpty()|A check to see if there are any properties on this object|Boolean|
|isPlain()|A check to see if this is a plain SimpleObject, i.e. standard JSON|Boolean|
|keys()|Get a list of keys for this object|List|
|keys(prefix)|Get a list of keys for this object|List|
|newLevel(name)|Adds a new SimpleObject level to this object|SimpleObject|
|remove(key)|Remove a property from this object|Object|
|set(key, value)|Adds a property to this object|Object|
|setPlain(plain)|Sets/unsets this object as plain, i.e. standard JSON|Object|

## Using groups in ODSL code

You can use groups when searching for data in other services.

### Example using the object service

Let's assume we have a group defined with 2 items in the object service, e.g.

```js
bg = Group()
bg.name="My Items"
bg.category = "Data Lists"
bg.service = "object"
bg.type = "favourite"
bg.items = ["ICE.IFEU.B","ICE.NDEX.NLB"]
save bg
```

We can retrieve data from the object service using the group in an ```in``` clause as follows:

```js
find ${object:"all"} where _id in group("object:favourite:Data Lists:My Items") project name
```

### Example using curves in the data service

Let's assume we have a group defined with 2 curves in the data service, e.g.

```js
bg = Group()
bg.name="Settlements"
bg.category = "Data Lists"
bg.service = "data"
bg.type = "favourite"
bg.items = ["ICE.IFEU.B:SETTLE","ICE.NDEX.NLO:SETTLE"]
save bg
```

We can retrieve data from the object service using the group in an ```in``` clause and a specific ondate as follows:

```js
find for ondate("2025-03-13") ${data} where _id in group("data:favourite:Data Lists:Settlements")
```

We can also use a relative ondate as follows:

```js
find for ondate("L-1") ${data} where _id in group("data:favourite:Data Lists:Settlements")
```

## Special Group Types

There are some pre-defined special groups which control specific functionality within the platform.

### Curve Build Group

A curve build group has specific configuration information regarding the cut-off time for when curves need to be ready for and the rules to run if they are not ready by that time.

The type for the group is set to ```build```
The category for the group is set to ```curve```
The service for the group is set to ```curve```

A curve build group adds these additional fields to a group

|**Name**|**Type**|**Example**|**Description**|
|-|-|-|-|
|timezone|String|Europe/Berlin|Sets the timezone for the cut-off time, defaults to UTC|
|calendar|String|#HENG|Sets the calendar used to determine the days that the curve is expected to build|
|cron|String||Sets the cut-off cron expression after which a substitute curve is created if it has not been built yet|
|offset|integer|0|Sets the number of days back to calculate the ondate from the cut off time, defaults to 0|
|useLatest|Boolean|true|A built in rule that uses the latest successfully build curve as a substitute if todays curve is missing|
|script|String||Sets the script that contains the function used to build a substitute curve if the curve isn't available by the cut-off time|
|expression|String||Sets the expression/function contained in the script used to build a substitute curve if the curve isn't available by the cut-off time|

#### An example build group

```js
g = BuildGroup()
g.name = "default2"
g.description = "Default Build Group"
g.calendar = "BUSINESS"
g.cron = "45 23 ? * MON-FRI * EU1"
g.offset = 0
g.timezone = "Europe/London"
g.useLatest = true
save g
```

### Quality Group

A quality group is a list of checks to perform on curves or datasets.

A quality group contains ```checks``` rather than items in a service.

The type for the group is set to ```quality```
The category and service for the group is set to ```curve``` or ```dataset```

A quality group adds these additional fields to a group

|**Name**|**Type**|**Example**|**Description**|
|-|-|-|-|
|script|String||The script that contains the functions used for these quality checks|
|checks|List||A list of checks to perform|

A check is defined as

|**Name**|**Type**|**Example**|**Description**|
|-|-|-|-|
|name|String|Test for zeros|The human readable name of the check|
|expression|String||The expression/function used to run this check|

#### An example quality group

```js
g = QualityGroup()
g.name = "default"
g.description = "Default Quality Group"
g.category = "curve"
g.script = "#QualityCheckCurves"
g.addCheck("Check for tenor gaps", "gapCheck()")
g.addCheck("Check for missing values", "missingCheck()")
g.addCheck("Check for zero values", "zeroCheck()")
save g
```

### Export Group

An export group is a list of items that a function is run at a set time, usually to export the data to a downstream system.

The type for the group is set to ```export```
The category and service for the group is set to ```curve``` or ```dataset```

An export group adds these additional fields to a group

|**Name**|**Type**|**Example**|**Description**|
|-|-|-|-|
|enabled|boolean|true|A flag to enable or disable this export group|
|cron|String||Set a cron expression to run the export|
|offset|Integer|0|Sets the number of days back to calculate the ondate from the cut off time, defaults to 0|
|calendar|String|#HENG|Sets the calendar used to determine the days that the curve is expected to be exported|
|version|String|APPROVED|Set the name of the curve version to export, leave null to use the latest version|
|script|String||Sets the script that contains the functions used for these exports|
|expression|String||Sets the expression/function contained in the script used to export the data|

#### An example export group

```js
g = ExportGroup()
g.name = "DAILY"
g.description = "Daily Curve Export Group"
g.enabled = true
g.cron = "00 23 ? * MON-FRI *"
g.calendar = "BUSINESS"
g.offset = 0
g.script = 'odsl-demo\Exports\daily_export_report'
g.expression = "export()"
save g
```

### Approval Group

An approval group is a list of email addresses of users that can approve the list of curves.

The type for the group is set to ```approval```
The category and service for the group is set to ```curve```

An approval group adds these additional fields to a group

|**Name**|**Type**|**Example**|**Description**|
|-|-|-|-|
|approvers|List|...|The list of email addresses of users that can approve the curves|
|version|String|APPROVED|Set the name of the curve version upon approval, defaults to APPROVED|

#### An example approval group

```js
g = ApprovalGroup()
g.name = "default"
g.description = "Default Approval Group"
g.approvers.add("demo@opendatadsl.com")
g.version = "APPROVED"
save g
```


---

<!-- source: product/developer/odsl/Variables/Index_.md -->

# Index

An **Index** defines a database index on a service, which makes queries on the indexed fields faster. Indexes are saved with the [index service](https://doc.opendatadsl.com/docs/api/rest/service/index).

## Construction

|**Function**|**Description**|
|-|-|
|Index()|Creates a new Database Index|

## Properties

|**Name**|**Description**|**Type**|
|-|-|-|
|hidden|Hidden indexes are not visible to the query planner and cannot be used to support a query.  By hiding an index from the planner, users can evaluate the potential impact of dropping an index without actually dropping the index. If the impact is negative, the user can unhide the index instead of having to recreate a dropped index.|Boolean|
|keys|The list of keys and index type to use in the index|Object|
|partialCondition|Partial indexes only index the documents in a collection that meet a specified filter expression. By indexing a subset of the documents in a collection, partial indexes have lower storage requirements and reduced performance costs for index creation and maintenance.|String|
|sparse|If set to true, the index only references objects with the indexed keys|Boolean|
|ttl|TTL indexes are special single-field indexes that MongoDB can use to automatically remove documents from a collection after a certain amount of time or at a specific clock time. Data expiration is useful for certain types of information like machine generated event data, logs, and session information that only need to persist in a database for a finite amount of time.|Integer|
|unique|If set to true, it constrains all values in the keys of the index to be unique|Boolean|

## Example

```js
idx = Index()
idx.id = "region_1"
idx.keys = Object()
idx.keys.region = 1
save ${index:idx}
```


---

<!-- source: product/developer/odsl/Variables/Link.md -->

Link
====================

A wrapper variable for creating links or relationships between objects

## Construction

### Constructor functions

|**Function**|**Description**|
|-|-|
|Link()|Creates a link that can be added to an item|

## Methods

|**Name**|**Description**|**Return Type**|
|-|-|-|
|set(key, value)|Set a property, e.g. name on this reference|Void|


---

<!-- source: product/developer/odsl/Variables/List.md -->

List
====================

A list represents a list of other variables

## Constructor

To create a list, you can use a constructor or use square brackets to enclose a list of variables.
```js
// Create an empty list
l1 = List()

// Also construct an empty list
l1 = []

// Create a list of variables
l2 = ["Hello", 1, l1]
```

:::info
Notice how you can also add a list as an element of a list, in fact you can add any other variable as an element of a list.
:::

## Properties

A list has the following properties:

|**Name**|**Description**|**Type**|
|-|-|-|
|size|Returns the number of elements in the list|Scalar(Integer)|
|description|The description of this list|String|
|name|The name of this list|String|

## Methods

A list has the following methods:

|**Name**|**Description**|**Return Type**|
|-|-|-|
|add(element)|Add a variable to the list of elements|The current list|
|add(position, element)|Adds a variable to the list in the supplied position (0 being first)|The current list|
|addAll(list)|Adds all the elements from the supplied list into this list|The current list|
|subList(start, end)|Gets a sublist from the elements in the list from the start to the end|New List|
|remove(index)|Removes an element from the list at the specified position (0 being first)|Void|
|remove(item)|Removes an element from the list|Void|
|set(index, element)|Replaces the element at the specified position in this list with the specified element|Void|

## Using Lists

You can directly access an element in a list using the square brackets and an index starting at 0, e.g. the following code would print out World
```js
// Create a list of variables
l3 = ["Hello", "World"]
print l3[1]
```
You can also replace elements in the list using the same technique, e.g.
```js
// Replace World with ODSL
l3[1] = "ODSL"
print l3[1]
```
You can also iterate through the elements of the list using a for loop, e.g.
```js
// Iterate through the list
for element in l3
    print element
next
```


---

<!-- source: product/developer/odsl/Variables/Matrix.md -->

Matrix
====================

A matrix represents a 2-dimensional array of numeric values - each row and column is labelled by default as R1, C1 etc. but these can be whatever you like.

## Constructor

To create a matrix, you can use the following constructor.
```js
// Create a matrix with 4 rows and 5 columns
m1 = Matrix(4, 5)

// Create a matrix with 3 rows and 3 columns with the labels A, B and C
m2 = Matrix(["A","B","C"])
```

### Constructor functions

|**Function**|**Description**|
|-|-|
|IdentityMatrix(size)|Creates a new identity matrix of the given size|
|Matrix(labels)|Creates a new matrix from a list of labels|
|Matrix(labels, v)|Creates a new matrix from a list of labels|
|Matrix(x, y)|Creates a new matrix|
|Matrix(y, x, v)|Creates a new matrix, initialising all values to the provided value|

## Properties

A matrix has the following properties:

|**Name**|**Description**|**Type**|
|-|-|-|
|name|The name of the matrix|String|
|description|A description of the matrix|String|
|ondate|The date this matrix represents|Date|
|xLabels|The list of x or column labels|String Array|
|yLabels|The list of y or row labels|String Array|
|xSize|The number of columns|Integer|
|ySize|The number of rows|Integer|
|currency|The currency of the values in this matrix|String|
|units|The units of the values in this matrix|String|
|timezone|The timezone for the values in this matrix|String|
|precision|The data value [precision](https://doc.opendatadsl.com/docs/kb/precision#data-precision-settings) configuration|String|
|identity|The configured identity for this curve|Object|

## Methods

A matrix has the following methods:

|**Name**|**Description**|**Return Type**|
|-|-|-|
|setLabels(String Array)|Sets the labels for the rows and columns of a symmetrical matrix|The current matrix|
|setXLabels(String Array)|Sets the labels for the columns of this matrix|The current matrix|
|setYLabels(String Array)|Sets the labels for the rows of this matrix|The current matrix|
|setColumnData(column, list)|Sets the values for the specified column name or index|The current matrix|
|setRowData(row, list)|Sets the values for the specified row name or index|The current matrix|
|setValue(column, row, value)|Sets the value for the specified column and row index|The current matrix|
|setValueAt(column, row, value)|Sets the value for the specified column and row label|The current matrix|
|getValue(column, row)|Gets the value for the specified column and row index|Double value|
|getColumnForLabel(column)|Gets the column index for a specified column label|Integer|
|getRowForLabel(row)|Gets the row index for a specified row label|Integer|
|isSameDimensions(matrix)|Tests to see if the passed in matrix is the same dimensions as this matrix|Boolean|
|transpose()|Returns a 90 degree transposed version of this matrix|Matrix|
|forwardFill()|Fills null values with the previous value, operates row by row|Void|
|getColumn(col)|Gets the data for a column|List|
|getGroupForColumn(col)|Gets the group name for the column number of this matrix (0-based)|String|
|getGroupForRow(row)|Gets the group name for the row number of this matrix (0-based)|String|
|getLabelForColumn(col)|Gets the column label for the column number of this matrix (0-based)|String|
|getLabelForRow(row)|Gets the row label for the row number of this matrix (0-based)|String|
|getRow(col)|Gets the data for a row|List|
|getValueAt(y, x)|Gets the value at the specified column and row|Number|
|subMatrix(rc1, rc2)|Creates a new matrix as a subset of this matrix|Matrix|
|subMatrix(y1, x1, y2, x2)|Creates a new matrix as a subset of this matrix|Matrix|

## Using Matrices

### Create a matrix and set some values
```js
m2 = Matrix(["A","B","C","D","E"])
m2.setValueAt("A","A",12.2)
m2.setValue(0,1,12.5)
print m2
```

### An example of some matrix operations
```js
set precision 1

m1 = Matrix(["A","B","C"])
m1.setRowData("A", [1,2,3])
m1.setRowData("B", [4,5,6])
m1.setRowData("C", [7,8,9])
print m1

m2 = Matrix(["A","B","C"])
m2.setColumnData("A", [1,2,3])
m2.setColumnData("B", [4,5,6])
m2.setColumnData("C", [7,8,9])
print m2

print m1 + m2
print m1 * 2
print 1/m1
print m1 ^ 2

m3 = Matrix(2, 3)
m3.setColumnData("C1", [1,2,3])
m3.setColumnData("C2", [4,5,6])

print m3
print m3.transpose()
```

## Further Reading
* [Matrix Functions](https://doc.opendatadsl.com/docs/odsl/function/matrix)


---

<!-- source: product/developer/odsl/Variables/Message.md -->

# Message

A **Message** is a string with a numeric code, used to return a coded result from a script or function. It has all the properties and methods of a [Scalar](https://doc.opendatadsl.com/docs/odsl/variable/scalar).

## Construction

|**Function**|**Description**|
|-|-|
|Message()|Creates a new coded message|

## Properties

|**Name**|**Description**|**Type**|
|-|-|-|
|code|Integer code for this coded message|Integer|


---

<!-- source: product/developer/odsl/Variables/MetricTimeSeries.md -->

## Introduction
Metric Timeseries represent a timeseries of metrics from various metric providers.

A metric timeseries consists of the following:

### Provider
The name of the metric provider

### Metric
The name of the metric to read from the provider

:::info
See [Metrics](https://doc.opendatadsl.com/docs/it/metrics) for full details on providers and metric names
:::

### Filter
An optional filter to filter the metrics in the value calculation

### Calendar
This is the interval frequencies for value aggregation, usually one of:

* #PT1M - 1 minutely
* #PT5M - 5 minutely
* #PT15M - 15 minutely
* #PT30M - 30 minutely
* #PT1H - hourly
* #PT6H - 6 hourly
* #PT12H - 12 hourly
* #P1D - daily

### Observed
The aggregation method, usually one of:

* summed - Sum up all the values for the interval
* averaged - Average the values for the interval
* high - The maximum value during the interval
* low - The minimum value during the interval

## Creating a MetricTimeSeries

### Azure Service Bus Metrics
An example of creating a metric timeseries for the average number of active messages on a queue named etrm aggregated 5 minutely
```js
ActiveMessages_1h = MetricTimeSeries("AzureServiceBus", "ActiveMessages")
ActiveMessages_1h.calendar = ${calendar:"#PT5M"}
ActiveMessages_1h.queue = "etrm"
ActiveMessages_1h.observed="averaged"
```

### ODSL User Request Metrics
An example of creating metric timeseries  for the total user read requests for the service 'user' aggregated hourly 
```js
ReadRequests_1h = MetricTimeSeries("ODSLRequest", "ReadRequests")
ReadRequests_1h.calendar = ${calendar:"#PT1H"}
ReadRequests_1h.observed = "summed"
ReadRequests_1h.filter = ?service='user'
```

## Properties

|**Name**|**Description**|**Type**|
|-|-|-|
|calendar|The calendar that this timeseries uses|Calendar|
|currency|The ISO currency of this timeseries|String|
|description|The description of this timeseries|String|
|filter|Filter the metrics|String|
|identity|The configured identities|Object|
|localValues|The list of time observations for this timeseries, returned as a list of TimeValues in the local timezone of this TimeSeries|List|
|metric|The name of the metric|String|
|name|The name of the timeseries|String|
|observations|The list of values for this timeseries, returned as a list of Vars|List|
|observed|The observed setting of the timeseries used for aggregating|TimeSeriesObserved|
|precision|The precision configuration for this timeseries|Precision|
|provider|The name of the metric provider|String|
|source|The source of this timeseries|String|
|start|The starting date/time of the timeseries|Date|
|tenor|The tenor of this timeseries|String|
|timezone|The timezone of this timeseries, defaults to UTC|String|
|units|The ISO units of this timeseries|String|
|valueType|The value type of the timeseries (TRACKED or BASIC)|TimeValueType|
|values|The list of time observations for this timeseries, returned as a list of TimeValues|List|

## Methods

|**Name**|**Description**|**Return Type**|
|-|-|-|
|addCheck(check)|Add a quality check directly to this Timeseries|TimeSeries|
|addValue(value)|Adds a new value to the end of the timeseries using the calendar|TimeSeries|
|applyPrecision()|Apply the precision defined to all the observations of this timeseries|Void|
|convert(currency, units)|Convert this timeseries using currency and units|TimeSeries|
|from(start)|Creates a new TimeSeries from a specified start date|TimeSeries|
|getLast(date)|Get the value at the supplied date or the latest value before that date|Var|
|getLastNValues(n)|Returns a list of the last N values from this TimeSeries|List|
|last(n)|Creates a new TimeSeries using the last n values from this TimeSeries|TimeSeries|
|range(start, end)|Creates a new TimeSeries from a range of values from this TimeSeries|TimeSeries|
|withRange(start, end)|Changes this timeseries to match the provided date range|Void|


---

<!-- source: product/developer/odsl/Variables/MissingValue.md -->

MissingValue
============================

A MissingValue is a real variable but it doesn’t have a value yet, it is usually used in TimeSeries to represent a TimeValue in a calendar that has no value

## Methods

|**Name**|**Description**|**Return Type**|
|-|-|-|
|isMissing()|Checks to see if the value is null or missing|Boolean|
|isZero()|Checks to see if the value is zero|Boolean|
|setPrecisionRoundingMode(mode)|Set the rounding mode for numbers|Scalar|
|setPrecisionScale(scale)|Set the number of decimal places to round to|Scalar|


---

<!-- source: product/developer/odsl/Variables/Null.md -->

Null
====================

Null represents an empty or no value variable

## Construction

### Constructor functions

|**Function**|**Description**|
|-|-|
|Null()|Creates a Null variable|


---

<!-- source: product/developer/odsl/Variables/Object.md -->

Object
======================

An object is a very versatile variable type that allows you to add any other variables to it including Object variable types. This creates a dynamic JSON type structure

## Construction

You can construct an object using a constructor function.
```js
// Create a new empty object
obj = Object()
```

### Constructor functions

|**Function**|**Description**|
|-|-|
|Object()|Creates a new Object of type #Object|
|Object(type)|Creates a new Object of the specified type|
|SimpleObject()|Creates a new SimpleObject or JSON Document|

## Properties

All properties on an object are dynamic, which means you can add any properties you like, e.g.

```js
// Create a new empty object
obj = Object()
obj.name = "ODSL"
obj.weight = 264.5
obj.start = Date("2020-10-08")
obj.children = List()
obj.children.addAll(["O","D","S","L"])
obj.address = Object()
obj.address.city = "London"
print json(obj)
```

This produces the following JSON output:
```json
{
  "_id": "obj",
  "address": {
    "_id": "address",
    "city": "London"
  },
  "children": [
    "O",
    "D",
    "S",
    "L"
  ],
  "name": "ODSL",
  "start": {
    "_id": "start",
    "value": "2020-10-08T00:00:00",
    "_type": "VarDate"
  },
  "weight": "264.5"
}
```

|**Name**|**Description**|**Type**|
|-|-|-|
|access|Get the name of the access data role for this object|String|


---

<!-- source: product/developer/odsl/Variables/Options.md -->

Options
=====

An Options variable holds the options for one ondate: a list of tenors (maturities), each with an option chain of strikes with the call and put values side by side.

## Introduction

You get an Options variable by reading an [EventOptions](https://doc.opendatadsl.com/docs/odsl/variable/eventoptions) for an ondate:

```js
eua = ${data:"NGE.EUA:OPTIONS:2026-09-16"}
```

An Options variable is made of three levels:

|**Level**|**Type**|**What it is**|**Index**|
|-|-|-|-|
|Options|Options|The options for an ondate|by tenor: `eua["2026M09"]` or `eua["M00"]`|
|Tenor|OptionTenor|One maturity and its option chain|by strike: `t[85]`|
|Strike|OptionStrike|One strike, with its call and put values|`s.call`, `s.put`|

## Options properties

|**Name**|**Description**|**Type**|
|-|-|-|
|id|The id of the options, ending with the ondate|String|
|ondate|The ondate of these options (YYYY-MM-DD)|String|
|name|The name of the options|String|
|description|A description of the options|String|
|currency|The currency code|String|
|units|The units code|String|
|timezone|The timezone of the ondate|String|
|value|The main value column|String|
|columns|The columns in the strike tables|List|
|tenors|The tenors (maturities) in delivery order|List|
|identity|The configured identity|Object|
|properties|The properties: `event`, `modified`, and for a positions view `positions`, `positionDate` and `unpriced`|Object|

## Options methods

|**Name**|**Description**|**Return Type**|
|-|-|-|
|tenor(code)|Gets a tenor (maturity) by its tenor, absolute or relative code, null when it is not there|OptionTenor|
|value(tenor, strike, callPut, column)|Gets a value of one option|Scalar|
|count()|The number of options (call and put prices at a strike count separately)|Number|
|size()|The number of tenors|Number|

## OptionTenor properties

|**Name**|**Description**|**Type**|
|-|-|-|
|tenor|The tenor of this maturity (the value of the tenor property, e.g. 2026M09)|String|
|absolute|The absolute tenor code, e.g. 2026M09|String|
|relative|The relative tenor code on the ondate, e.g. M01|String|
|start|The start of the delivery period (date)|String|
|end|The end of the delivery period (date, inclusive)|String|
|expiry|The expiry date of the options|String|
|strikes|The strikes (option chain rows) in ascending order|List|

## OptionTenor methods

|**Name**|**Description**|**Return Type**|
|-|-|-|
|strike(price)|Gets the option chain row for a strike, null when the strike is not there|OptionStrike|
|nearest(price)|Gets the option chain row with the strike nearest to a price (e.g. the at-the-money strike for the underlying price)|OptionStrike|
|values(callPut, column)|Gets one column of the calls (C) or puts (P) as a list in strike order (missing values are null)|List|
|strikePrices()|Gets the strike prices as a list in ascending order|List|
|size()|The number of strikes|Number|

## OptionStrike properties and methods

|**Name**|**Description**|**Type**|
|-|-|-|
|strike|The strike price, as exact text so that `strike == 85` works|Scalar|
|call|The call option values by column (empty when there is no call at this strike)|Object|
|put|The put option values by column (empty when there is no put at this strike)|Object|
|get(callPut, column)|Gets a value of the call (C) or put (P) at this strike|Scalar|

In a positions view the held side also has `POSITION_VOLUME` and `TRADED_PRICE`.

## OptionsDates

Reading an EventOptions without an ondate gives an OptionsDates: the ondates the options can be built for, each referencing `OBJ:ID:YYYY-MM-DD`. It works like the [CurveDates](https://doc.opendatadsl.com/docs/odsl/variable/curve) of a curve, including ranges and the `L`, `L-1` and `-YYYY-MM-DD` ids.

## Examples

#### Going through the option chains
```js
eua = ${data:"NGE.EUA:OPTIONS:2026-09-16"}
for t in eua
    print t.tenor + " expires " + t.expiry
    for s in t
        print s.strike
        print s.call
        print s.put
    next
next
```

#### A volatility smile
```js
sep = eua["2026M09"]
strikes = sep.strikePrices()
vols = sep.values("C", "OPTION_VOLATILITY")
```

#### The at-the-money strike
```js
atm = eua["2026M09"].nearest(86.4)
print atm.strike
print atm.call.DELTA_FACTOR
```

#### Strikes matching a condition
```js
above = eua["2026M09"].strikes[strike > 82]
```


---

<!-- source: product/developer/odsl/Variables/Policy.md -->

Policy
=======================

A policy variable contains information about security access to data

## Introduction

Policy variables are used to restrict access to certain data within the platform.

There are three kinds of policy, all saved in the policy service and told apart by their `_type`:

|**Variable**|**_type**|**Purpose**|
|-|-|-|
|Policy|Policy|Security policies: which records users can create, read, update, delete or run (this page)|
|[FeaturePolicy](https://doc.opendatadsl.com/docs/odsl/variable/featurepolicy)|FeaturePolicy|Which Web Portal menu items and extension views users see|
|[RedactionPolicy](https://doc.opendatadsl.com/docs/odsl/variable/redactionpolicy)|RedactionPolicy|Which master data fields are removed from the records users read|

For an overview and worked examples, see [Policies](https://doc.opendatadsl.com/docs/topics/policies) and [Security Policies](https://doc.opendatadsl.com/docs/odsl/topics/policies).

## Defining a Policy

Below is a table with all the properties of a policy:

|**Name**|**Type**|**Example**|**Description**|
|-|-|-|-|
|_id|String|RunAllReports|The unique id for this policy|
|category|String|TeamUsers|An optional category used for filtering policies|
|description|String|My test policy|A descriptive name for the policy|
|enabled|Boolean|true|True if it is enabled, false to disable it, by default the policy is enabled|
|source|String|private|The data source, defaults to private if omitted|
|service|String|data|The service that this policy relates to or * for all services|
|condition|String|source = 'ICE'|The condition used to filter the data that this policy restricts|
|deny|Boolean|false|True if this is a deny policy, false if this is an allow policy|
|actions|List|read|The [actions](#actions) that this policy applies to|
|members|List|user@company.com|The user emails, policy user groups or Azure Active Directory group id's this policy applies to|

## Actions

|**Action**|**Description**|
|-|-|
|create|Create new records|
|read|Read records|
|update|Update existing records|
|delete|Delete records|
|run|Run the record, only for services that can be run such as reports and processes|
|editPublicTypeProperties|Change master data properties declared on a public type (see [Type property permissions](#type-property-permissions))|
|editPrivateTypeProperties|Change master data properties declared on a private type (see [Type property permissions](#type-property-permissions))|

`setFullAccess()` adds create, read, update, delete and run. The two type property permissions are never included and must be added explicitly.

## Members

|**Member**|**Example**|**Description**|
|-|-|-|
|*|*|Everyone in your company|
|Email|user@company.com|A single user|
|Policy user group|user:security:policy:DataStewards|Everyone in a [policy user group](https://doc.opendatadsl.com/docs/topics/policies#policy-user-groups) you manage in the platform|
|Azure Active Directory group|39c6cccd-d6ea-4ac1-b564-8e4d1e28d75d|Everyone in the AAD security group|

## How policies are combined

* Allow policies are combined with **or**, deny policies with **and not**. If a user has no policy for an action, the action is not allowed.
* A deny policy is ignored when the user also has an allow policy with exactly the same condition text, which lets you deny something to everyone except a few people.
* For create the new record must match the policy conditions; for update and delete the stored record must match; for read the conditions filter the results.

See [How access is decided](https://doc.opendatadsl.com/docs/odsl/topics/policies#how-access-is-decided) for the full rules, defaults and built-in policies.

## Type property permissions

Properties declared on a master data record's type, or on any type it extends, are protected by two permissions on the `object` service:

|**Permission**|**Covers properties declared on**|
|-|-|
|editPublicTypeProperties|A public type, with an id starting with `#`, e.g. `#Commodity`|
|editPrivateTypeProperties|One of your own private types|

* Both are **denied by default**: nobody has them, not even users with full access, until a policy grants them.
* Whether a property is public or private depends on the type that declares it, not on the record's own type. A private type `GasHub` that extends `#Commodity` has private properties (declared on `GasHub`) and public properties (declared on `#Commodity`).
* They only apply when an existing record is changed. Adding, changing or removing a declared property needs the permission; creating a new record does not.
* Properties that are not declared on the type, the standard `#Object` properties such as `name` and `description`, smart data, and the curves, timeseries and events held on a record are not covered and only need `update`.
* The user also needs `update` access to the record.
* A change without the permission is rejected with a `403 Forbidden` error that names the properties and the permission needed.

Example: data stewards can change type properties on all master data:

```js
StewardsEditTypeProperties = Policy()
StewardsEditTypeProperties.description = "Data stewards maintain type properties"
StewardsEditTypeProperties.service = "object"
StewardsEditTypeProperties.addActions(["update", "editPublicTypeProperties", "editPrivateTypeProperties"])
StewardsEditTypeProperties.addMember("user:security:policy:DataStewards")
save StewardsEditTypeProperties
```

The `rights` function of the policy service returns `editPublicTypeProperties` and `editPrivateTypeProperties` for a record when the user has them:

```
GET https://api.opendatadsl.com/api/policy/v1?_function=rights&service=object&key=ICE.IFEU.B
```

## Methods

A policy has the following methods:

|**Name**|**Description**|**Return Type**|
|-|-|-|
|addMember(name)|Adds a member to this policy|void|
|removeMember(name)|Removes a member from this policy|void|
|addAction(action)|Adds an action to this policy|void|
|addActions(action[])|Adds multiple actions to this policy|void|
|setFullAccess()|Sets this policy to cover create, read, update, delete and run|void|

Example policy definition:
```json
{
  "_id": "DenyAccessToBWSSBData",
  "category": "TradingTeam",
  "description": "Deny all access to BWSSB data",
  "source": "private",
  "service": "object",
  "condition": "source = 'BWSSB'",
  "deny": true,
  "actions": [
    "create",
    "read",
    "update",
    "delete",
    "run"
  ],
  "members": [
    "user@company.com"
  ],
  "enabled": true
}
```

## Properties

|**Name**|**Description**|**Type**|
|-|-|-|
|tags|Tags to help filter and find policies|List|

## Updating, Finding and Deleting Policies

### Saving a policy

To save a policy in OpenDataDSL code, use the save command as follows:
```js
RunAllReports = Policy()
RunAllReports.description = "Run all reports"
RunAllReports.service = "report"
RunAllReports.addMember("user@company.com")
RunAllReports.addAction("run")

save RunAllReports
```

### Listing policies

To find policies, you use the ODSL find command, e.g.
```js
policies = find ${policy} where _type="Policy"
```
You can use the unique keyword to just get all the ids of your policies, e.g.
```js
policynames = find unique _id from ${policy} where _type="Policy"
```

### Retrieving a specific policy

To get a specific named policy, you use the policy active variable. 
You can then examine the information on the policy, e.g.
```js
p = ${policy:"ReadExampleReports"}
print p.description
```

### Disabling a policy
You can disable a policy by setting the enabled flag to false, e.g.
```js
p = ${policy:"ReadExampleReports"}
p.enabled = false
save p
```

### Deleting a policy

To delete a policy, you issue the delete command, e.g.
```js
delete ${policy:"ReadExampleReports"}
```

## Special Condition Variables
You can use the following variables in conditions:

* ```{me}``` - this will be replaced by the current users email, e.g. ```"owner = '{me}'"```

## Feature Policies

A feature policy controls which menu items and extension views a user sees in the Web Portal. It has `features` (a list of feature names) instead of service, source, condition and actions.

* Allow feature policies switch features on, deny feature policies switch them off, and deny always wins.
* They only change what the portal shows: they do not stop access to data. Use a security policy for that.

```js
HideScripts = FeaturePolicy()
HideScripts.description = "Hide Scripts menu item from users"
HideScripts.features = ["scripts"]
HideScripts.deny = true
HideScripts.addMember("user:security:policy:BusinessUsers")
save HideScripts
```

See [FeaturePolicy](https://doc.opendatadsl.com/docs/odsl/variable/featurepolicy) for the properties and feature names, and [Feature Policies](https://doc.opendatadsl.com/docs/topics/policies/feature) for examples.

## Redaction Policies

A redaction policy removes fields from the master data records its members read. It has `redactions` (a list of field names, with dot notation for nested fields) instead of service, source, condition and actions.

* It applies to the `object` service when records are read, listed or searched.
* Members cannot use redacted fields in filters, searches, sorts or aggregations.
* There is no allow or deny: the fields are always removed for the policy's members.

```js
HideContractPricing = RedactionPolicy()
HideContractPricing.description = "Analysts do not see contract pricing"
HideContractPricing.addFieldRedaction("contract.price")
HideContractPricing.addMember("user:security:policy:Analysts")
save HideContractPricing
```

See [RedactionPolicy](https://doc.opendatadsl.com/docs/odsl/variable/redactionpolicy) for the properties, and [Redaction Policies](https://doc.opendatadsl.com/docs/topics/policies/redaction) for examples.


---

<!-- source: product/developer/odsl/Variables/Process.md -->

Process
=======================

A process variable contains all the information to configure and run a process

## Introduction

Process variables are used to configure a process that is run in the ODSL cloud. The process is run in 1 of 3 ways:

*   Time triggered using a cron expression    
*   Event triggered from an automation    
*   Manually triggered using the run command in ODSL or sending a REST API request
    

## Defining a Process

Below is a table with all the properties of a process:

|**Name**|**Type**|**Example**|**Description**|
|-|-|-|-|
|service|String|ETL|The service name of this process, this is used to categorise processes|
|name|String|TEST|The name of the process|
|description|String|My test process|A descriptive name for the process|
|enabled|Boolean|true|True if it is enabled, false to disable it from running|
|processType|String|Ristretto|'Ristretto - short running, low resources' 'Lungo - longer running, more resources'|
|workflow/script|String|test_wf|The name of the workflow or script|
|settings|Object|factor=12|A list of variables that are set before the process is run|
|input|Object|date=”2021-05-17”|An object used as input to the process which can be overridden if manually running the process. In the case of a workflow, this can be used in the actions|
|cron|String|25 12 ? * MON-FRI *|[CRON Expression](https://doc.opendatadsl.com/docs/kb/cron)
|properties|Object|process.properties.vendor = "European Central Bank"|User definable meta-data about the process|

Example process definition:
```js
print "Create a process"
process = Process()
process.service = "ETL"
process.name = "ECB\_FX\_TEST"
process.description = "TEST ETL Process for ECB_TEST"
process.processType = "Ristretto"
process.enabled = true
process.input.url="https://www.ecb.europa.eu/stats/eurofxref/eurofxref-daily.xml"
process.input.tx = "#ECB_FX"
process.workflow="#wf\_xml\_data_loader"
process.cron = "38 07 ? * * *"
process.properties.vendor = "European Central Bank"
process.settings.factor = 1.2
save ${process:process}
```

## Properties

|**Name**|**Description**|**Type**|
|-|-|-|
|dataPackage|Sets this to be a deployable data packages|Boolean|

## Methods

|**Name**|**Description**|**Return Type**|
|-|-|-|
|run()|Run this process|Any|

## Updating, Finding and Deleting Processes

### Saving a process

To save a process in OpenDataDSL code, use the save command as follows:
```js
process = Process()
process.service = "ETL"
process.name = "ECB\_FX\_TEST"
save ${process:process}
```

### Listing processes

To find processes, you use the ODSL find command, e.g.
```js
processes = find ${process}
```
You can use the unique keyword to just get all the names of your processes, e.g.
```js
processnames = find unique name from ${process}
```

### Retrieving a specific process

To get a specific named process, you use the process active variable. You can then examine the information on the process, e.g.
```js
proc = ${process:"ECB_FX"}
print proc.cron
```

### Rolling back changes to a process

Whenever you change a process and save it, it saves as a new version of the process and archives the old version. If you need to roll back to the previous version, you issue a delete command with no specific version information. This removes the current version and un-archives the previous version to make it the new current version, e.g.
```js
delete ${process:"ECB_FX"}
```

### Completely removing a process

To completely remove a process, you issue the delete command with * as the version number, e.g.
```js
delete ${process:"ECB_FX":*}
```

## Running a Process

You can manually run any process and optionally pass input data to the process.

### Running a process in OpenDataDSL

To run a process in ODSL code, use the run command, e.g.
```js
run ECB_FX
```

To pass input data to a process, e.g. for running a curve for a specific date - use run with, e.g.
```js
ondate = Object()
ondate.date = "2021-05-17"
run WHEAT_CURVE with ondate
```
### Running a process using the API

To run a process using the REST API, you call the **process-exec** resource as follows:

```JSON
POST https://api.opendatadsl.com/api/process-exec/v1?_reason=test
Authorization: Bearer {{token}}

{
  "name": "ECB_FX"
}
```

To pass input data, add an input object into the body, e.g.

```json
POST https://api.opendatadsl.com/api/process-exec/v1?_reason=test
Authorization: Bearer {{token}}

{
  "name": "WHEAT_CURVE",
  "input": {
    "date": "2021-05-17"
  }
}
```

## Further Reading
* [Process service](https://doc.opendatadsl.com/docs/odsl/service/process)


---

<!-- source: product/developer/odsl/Variables/RedactionPolicy.md -->

A redaction policy variable contains a list of master data fields that are hidden from some users

## Introduction

Redaction Policy variables are used to remove sensitive fields, such as prices or internal notes, from master data records before they are returned to the policy's members. The rest of the record is still returned.

Redaction applies to master data (the `object` service) when records are read, listed or searched. Members cannot filter, search, sort, group or aggregate on redacted fields. For how redaction works and more examples, see [Redaction Policies](https://doc.opendatadsl.com/docs/topics/policies/redaction).

## Defining a Redaction Policy

Below is a table with all the properties of a redaction policy:

|**Name**|**Type**|**Example**|**Description**|
|-|-|-|-|
|_id|String|HideContractPricing|The unique id for this policy|
|category|String|TeamUsers|An optional category used for filtering policies|
|description|String|Hide contract pricing|A descriptive name for the policy|
|enabled|Boolean|true|True if it is enabled, false to disable it, by default the policy is enabled|
|members|List|user@company.com|The user emails or policy user groups this policy applies to, or * for everyone|
|redactions|List|\["contract.price"\]|The fields to remove, using dot notation for a field inside a property|

A redaction policy has no allow or deny: its fields are always removed for its members.

## Methods

A redaction policy has the following methods:

|**Name**|**Description**|**Return Type**|
|-|-|-|
|addMember(name)|Adds a member to this policy|void|
|removeMember(name)|Removes a member from this policy|void|
|addFieldRedaction(name)|Adds a field to redact|void|

Example redaction policy definition:
```json
{
  "_id": "HideContractPricing",
  "_type": "RedactionPolicy",
  "category": "TradingTeam",
  "description": "Analysts do not see contract pricing",
  "redactions": [
    "contract.price",
    "contract.currency"
  ],
  "members": [
    "user:security:policy:Analysts"
  ],
  "enabled": true
}
```

## Properties

|**Name**|**Description**|**Type**|
|-|-|-|
|deny|Determines if this policy is an explicit allow or deny policy|Boolean|
|tags|Tags to help filter and find policies|List|

## Updating, Finding and Deleting Redaction Policies

### Saving a redaction policy

To save a redaction policy in OpenDataDSL code, use the save command as follows:
```js
HideContractPricing = RedactionPolicy()
HideContractPricing.description = "Analysts do not see contract pricing"
HideContractPricing.addFieldRedaction("contract.price")
HideContractPricing.addFieldRedaction("contract.currency")
HideContractPricing.addMember("user:security:policy:Analysts")
save HideContractPricing
```

### Listing redaction policies

To find redaction policies, you use the ODSL find command, e.g.
```js
policies = find ${policy} where _type="RedactionPolicy"
```

### Retrieving a specific redaction policy

To get a specific named redaction policy, you use the policy active variable, e.g.
```js
p = ${policy:"HideContractPricing"}
print p.redactions
```

### Disabling a redaction policy
You can disable a redaction policy by setting the enabled flag to false, e.g.
```js
p = ${policy:"HideContractPricing"}
p.enabled = false
save p
```

### Deleting a redaction policy

To delete a redaction policy, you issue the delete command, e.g.
```js
delete ${policy:"HideContractPricing"}
```


---

<!-- source: product/developer/odsl/Variables/Report.md -->

Report
=====

A report is a configuration containing an expression that can be run to produce a user definable report.

## Introduction
A report is a flexible way of running some code to produce an output that can be shown in the Web Portal and the Excel Add-in or sent to a downstream system.

An example of a report is an aggregation of some data grouped by a specific field.

## Construction

To construct a new report configuration, you use the report function as shown in the syntax below:
```js
report = Report()
```

### Constructor functions

|**Function**|**Description**|
|-|-|
|Composition()|Creates a new composition configuration|
|DynamicReport()|Creates a new dynamic report configuration|
|FixedReport()|Creates a new Fixed report configuration|
|InsightReport()|Creates a new insight report configuration|
|Report()|Creates a new report configuration|
|WidgetReport()|Creates a new widget report configuration|

## Properties

A report has the following properties:

|**Name**|**Description**|**Type**|
|-|-|-|
|category|The category of this report, used for filtering|String|
|name|The name of the report|String|
|description|A description of the report|
|script|The name of the script used for this report|String|
|expression|The expression used to run this report|String|
|tags|A list of tags used to filter for this report|List of Strings|
|cacheOptions|Set the SMART caching options for this report|SmartReportCachingOptions|
|cron|The cron expression to trigger this report, you must also set the enabled flag to true|String|
|currency|The optional currency of this report, can be used in the function using #REPORT.currency|String|
|defaultRange|The optional default range for this report|String|
|enabled|The enabled flag is used in conjunction with the cron expression to turn on/off the schedule triggered running of this report|Boolean|
|end|The end date used to generate this report|Date|
|hideExcel|Hides this report from the Excel Add-in|Boolean|
|hideList|Hides this report from the Reports list|Boolean|
|mergeKeys|Fixed reports only: the field names used to match array elements when merging updates, the first one present in an element is used|List|
|mergeUpdates|Fixed reports only: when true, updating the report for an ondate merges the supplied data into the existing report instead of creating a new version|Boolean|
|ondate|The report publish date|Date|
|report|The id of the report|String|
|reportType|The type of this report, one of Default, Fixed, Dynamic, Insight or Widget|String|
|start|The start date used to generate this report|Date|
|subCategory|The user defined sub category for this report|String|
|template|The mustache template used when outputting this report|String|
|timezone|The optional timezone of this report, can be used in the function using #REPORT.timezone|String|
|units|The optional units of this report, can be used in the function using #REPORT.units|String|

Examples of report properties:
```js
rep = Report()
rep.name = "SKU Report"
rep.script = "testReportScript"
rep.expression = "SKUReport()"
rep.id = "REPORT.SKU"

save rep
```

## Methods

A report has the following methods:

|**Name**|**Description**|**Return Type**|
|-|-|-|
|build()|Run the expression to build the report|[report](#filled-out-report)|
|build(date)|Run the expression to build the report specifying a date to start from|[report](#filled-out-report)|
|build(date, date)|Run the expression to build the report specifying a date range|[report](#filled-out-report)|
|withAggregationOnDateGenerator(service, source, aggregation)|Use an aggregation pipeline to generate the dates for this report|ODGAggregation|
|withCalendarOnDateGenerator(calendar, start)|Use a calendar to generate the dates for this report|ODGCalendar|
|withCurveOnDateGenerator(id)|Use the dates of a curve to generate the dates for this report|ODGCurve|
|withDistinctOnDateGenerator(service, source, field)|Use a distinct find query to generate the dates for this report|ODGDistinct|
|withDistinctOnDateGenerator(service, source, field, filter)|Use a distinct find query to generate the dates for this report|ODGDistinct|
|withFilterOnDateGenerator(service, source, filter, projection)|Use a filter to generate the dates for this report|ODGFilter|
|withMatrixOnDateGenerator(id)|Use the dates of a matrix to generate the dates for this report|ODGMatrix|

The result of running any of the build commands is a *filled out* report.

## Filled out report

### Properties

A *filled out* report has the following properties additional to the report configuration properties:

|**Name**|**Description**|**Type**|
|-|-|-|
|data|The variable created from running the report|Any|
|errorMessage|If the report fails to execute, this will be filled in with the error message|String|
|scriptVersion|The version of the script used to run this expression|Number|

## Examples

### Create a report

```js
rep = Report()
rep.name = "SKU Report"
rep.script = "testReportScript"
rep.expression = "SKUReport()"
rep.id = "REPORT.SKU"

save rep
```

### Get a report configuration and run it

```js
rep = ${reportconfig:"REPORT.SKU"}
test = rep.build("2023-02-01", "2023-02-28")
print test.data
```

### Run the report

```js
rep = ${report:"REPORT.SKU"}
print rep.data
```

### Run the report with a range

```js
rep = ${report:"REPORT.SKU", "_range=from(2023-02-01)"}
print rep.data
```

### Run and save a report

```js
save ${report:"REPORT.SKU"}
```

### Get a specific saved report

```js
rep = ${report:"REPORT.SKU:2026-09-30"}
print rep.data
```

### Get the latest saved report

```js
rep = ${report:"REPORT.SKU:L"}
print rep.data
```

### Run and save a report with a range

```js
save ${report:"TEST", "_range=from(2023-02-01)"}
```

### Save a var as a report

```js
TEST = ["Hello","World"]
save ${report:TEST}
print ${report:"TEST:L"}
```


---

<!-- source: product/developer/odsl/Variables/ReportCreator.md -->

# ReportCreator

A **ReportCreator** is a [report configuration](https://doc.opendatadsl.com/docs/odsl/variable/report) with parameters: people fill in the parameters to create their own reports from it, and an example report is built from each parameter's example value. It is saved with the [report creator service](https://doc.opendatadsl.com/docs/api/rest/service/reportcreator).

## Construction

|**Function**|**Description**|
|-|-|
|ReportCreator()|Creates a new report creator|

It has all the properties and methods of a [report configuration](https://doc.opendatadsl.com/docs/odsl/variable/report), and a `parameters` list. Each parameter has a `name`, `type`, `description` and `example`.


---

<!-- source: product/developer/odsl/Variables/Row.md -->

Row
===================

A row represents a single row of data usually associated with a row in a relational database table. A row is similar to an object in that it has dynamic properties except that you can only add Scalars or Dates as elements to a row

## Construction

You use a constructor to create an empty row:
```js
// Create an empty row
row = Row()
```

## Properties

As with an object, you can dynamically add properties, but only of type Scalar or Date, e.g.
```js
// Create an empty row
row = Row()
row.key = "1234-name"
row.name = "Name"
```
If you need to add a property to a row which is a reserved name like id, you can use the addColumn method e.g.
```js
// Create an empty row
row = Row()
row.addColumn("id", 1234)
```

## Methods

The following methods are available for a row:

|**Name**|**Description**|**Return Type**|
|-|-|-|
|addColumn(name, value)|Adds a column to a row with the supplied name and value|void|


---

<!-- source: product/developer/odsl/Variables/Scalar.md -->

Scalar
======================

A scalar is a single value of either string, boolean (true/false) or numeric

## Construction

You construct a scalar variable by assigning a value to it, e.g.
```js
// Constructing a string scalar
a = "String"

// Constructing a boolean scalar
b = true

// Constructing a numeric (integer) scalar
c = 10

// Constructing a numeric (decimal) scalar
d = 3.14
```
:::note
Numeric values always obey the currently configured precision settings which default to 6 decimal places and half_up rounding
:::

### Constructor functions

|**Function**|**Description**|
|-|-|
|ObjectId()|Creates a unique object id|
|String(text)|Creates a new string scalar using the supplied text|

## Methods

|**Name**|**Description**|**Return Type**|
|-|-|-|
|isMissing()|Checks to see if the value is null or missing|Boolean|
|isZero()|Checks to see if the value is zero|Boolean|
|setPrecisionRoundingMode(mode)|Set the rounding mode for numbers|Scalar|
|setPrecisionScale(scale)|Set the number of decimal places to round to|Scalar|


---

<!-- source: product/developer/odsl/Variables/Script.md -->

Script
======================

A script variable is a wrapper for an ODSL script. It is used by the script service and can also run the script

## Construction

### Constructor functions

|**Function**|**Description**|
|-|-|
|Script()|Creates a new Script|

## Properties

|**Name**|**Description**|**Type**|
|-|-|-|
|category|The category of this script, e.g. ETL|String|
|description|A description of what this script does|String|
|filename|The filename of this script|String|
|outputFormat|An optional output format, e.g. application/xml, text/html, text/csv|String|
|script|The script text|String|
|scriptType|The type of this script, e.g. one of [odsl, mustache, py, css, html, js, mjs, cjs]|String|

## Methods

|**Name**|**Description**|**Return Type**|
|-|-|-|
|loadFrom(file)|Load the script from a local file|Void|
|run()|Runs this script|Void|
|run(globalContext)|Runs this script|Void|


---

<!-- source: product/developer/odsl/Variables/Secret.md -->

Secret
======================

A secret is a scalar whose value cannot be printed or logged, it only really is useful when used with the Secret Service where secrets can be stored and then retrieved by scripts or workflows. Usually secrets are used to store passwords and database connection information etc.

## Construction

You can construct a secret using one of the constructor functions:
```js
// Create a new secret
sec = Secret("password!")
```
If you try to print a secret, you just see the following:
```js
`********`
```

## Properties

|**Name**|**Description**|**Type**|
|-|-|-|
|id|The name of the secret|Scalar(String)|
|value|String|The actual secret|
|enabled|Boolean|A boolean to enable or disable this secret, defaults to true|
|notBefore|String|An optional date string indicating when this secret is valid from|
|expires|String|An optional date string indicating when this secret is valid to|

## Example Usage

### Update a secret
```js
//#region Add a secret
test = Secret("this_is_a_test")
test.notBefore = "2024-06-01"
test.expires = "2024-12-31"
save test
//#endregion
```

## More information
* [Secret Service](https://doc.opendatadsl.com/docs/odsl/service/secret)


---

<!-- source: product/developer/odsl/Variables/SmartCurve.md -->

A Smart Curve is an expression that is used to build curves on demand

## Introduction
Smart Curves are an easy but powerful way of defining curve derivations.

A Smart Curve consists of the following:

### Base Curve
The base curve is mandatory and can be either a real curve or another Smart Curve.
The base curve is used to determine the curve dates that this curve has available and can also be used in the expression using the variable name ```BASE```.

### Expression
The expression is also mandatory and is a string formula used to build the curves for this Smart Curve.

* It can be a simple operation like ```BASE*1.2```
* It can use built-in functions like ```bootstrapCurve(BASE)```
* It can use functions in the #CurveScript (if you have not specified a custom script) like ```flatCurve(12, 1)```
* It can use your own functions in a specified custom script like ```myfunction(BASE)```

### Script
The script is an optional saved custom script with user defined functions. 
These functions can then be used in the expression.
The function script can also `import` the default #CurveScript, so your functions can use those too.

### References to other data
You can optionally add references to other curves, Smart Curves or TimeSeries that can also be used in your expression

## Construction

You construct a Smart Curve using one of the following constructors:

```js
// Default constructor
sc = SmartCurve()

// Specify base curve id and expression
sc = SmartCurve("CORN:CURVE", "interpolate(BASE,'BACKWARD')")

// Specify base curve and expression
sc = SmartCurve(${data:"CORN:CURVE"}, "interpolate(BASE,'BACKWARD')")

// Specify a base curve reference and expression
sc = SmartCurve(ref(${data:"CORN:CURVE"}), "interpolate(BASE,'BACKWARD')")
```

## Properties

A SmartCurve has the following properties:

|**Name**|**Description**|**Type**|
|-|-|-|
|baseCurve|The curve that this Smart Curve uses for CurveDates|ID String, Curve or Reference|
|expression|The expression used to generate the curves|String|
|script|Optional custom script name used with the expression|String|
|currency|Optional currency of the curves, defaults to base curve currency|String|
|units|Optional units of the curves, defaults to base curve units|String|
|cacheOptions|The options used to cache this smart curve|SmartCurveCacheOptions|
|category|The category to place this curve in|String|
|commodity|The commodity for this curve|String|
|description|A description of the curve|String|
|identity|The configured identity for this curve|Object|
|market|The market to place this curve in|String|
|name|The name of the curve|String|
|precision|The precision configuration for this curve|Precision|
|priority|The build priority, 1=high, 2=medium, 3=low|Integer|
|product|The product category for this curve|String|
|timezone|The timezone for this curve|String|

:::info
If a currency and/or units are set, the base curve and any other curves added to this Smart Curve will be converted to the specified currency and units
:::

A SmartCurve also supports adding dynamic properties, e.g.

```js
sc = SmartCurve("SMART_TEST:CURVE", "BASE*1.02")
sc.product = "CORN"
```

Any dynamic properties added to a SmartCurve can be used in the expression including references to other curves and TimeSeries.
Here are some examples of using properties within the Smart Curve expression:

```js
// Example using another curve
sc1 = SmartCurve()
sc1.baseCurve="#DCE.AG.CN.A.NO1_SOYBEAN.FUT:HIGH"
sc1.LOW=ref(${data:"#DCE.AG.CN.A.NO1_SOYBEAN.FUT:LOW"})
sc1.expression="(BASE+LOW)/2"

// Example using a numeric factor
sc2 = SmartCurve()
sc2.baseCurve="#DCE.AG.CN.A.NO1_SOYBEAN.FUT:HIGH"
sc2.factor=1.2
sc2.expression="BASE*factor"
```

## Methods

A SmartCurve has the following methods:

|**Name**|**Description**|**Return Type**|
|-|-|-|
|build(Date)|Builds a curve for the specified date|Curve|

Below is an example of creating a Smart Curve and building a curve for a specific date using it:

```js
sc1 = SmartCurve()
sc1.baseCurve="#DCE.AG.CN.A.NO1_SOYBEAN.FUT:HIGH"
sc1.factor=1.2
sc1.expression="BASE*factor"

c1 = sc1.build("2021-10-27")
print c1.contracts
```

## Adding to an object and saving
A SmartCurve needs to be added to an object in order to save it to the database, e.g.

```js
CORN_TEST = Object()
CORN_TEST.category = "TEST"

cf = SmartCurve("#MATBAROFEX.ROS.CORN.FUT:CLOSE", "interpolate(bootstrapCurve(BASE),'BACKWARD')")
CORN_TEST.CURVE = cf

save ${object: CORN_TEST}
```

## Using custom scripts
As stated in the introduction, you can use your own custom scripts containing functions that can be used in the Smart Curve expression.

Below is an example of a custom script named ```SmartCurveCustomScript```

```js
import ${script:"#CurveScript"}

function agricultural(input)
    // Agricultural curve bootstrapped and backward filled
    boots = bootstrapCurve(input)
    agricultural = interpolate(boots, "BACKWARD")
end
```

:::note
This script needs to be uploaded into the DataBase (In VSCode - right-click in the editor and select 'Upload this script')
:::

Define our curve using this custom script:

```js
SOYBEAN = Object()
SOYBEAN.CURVE = SmartCurve("#DCE.AG.CN.A.NO1_SOYBEAN.FUT:CLOSE", "agricultural(BASE)")
SOYBEAN.CURVE.script = "SmartCurveCustomScript"
```

And we can now run a test build:

```js
c1 = SOYBEAN.CURVE.build("2021-10-27")
print c1.contracts
```


---

<!-- source: product/developer/odsl/Variables/SmartTimeSeries.md -->

A Smart TimeSeries is an expression that is used to build TimeSeries on demand

## Introduction
Smart TimeSeries are an easy but powerful way of defining TimeSeries derivations.

A Smart TimeSeries consists of the following:

### Base TimeSeries
The base TimeSeries is mandatory and can be either a real TimeSeries or another Smart TimeSeries.
The base TimeSeries is usually used in the expression using the variable name ```BASE```.

### Expression
The expression is also mandatory and is a string formula used to build the TimeSeries.

* It can be a simple operation like ```BASE*1.2```
* It can use built-in functions like ```scale(BASE, 'MONTHLY', 'summed')```
* It can use functions in the #TimeSeriesScript (if you have not specified a custom script) like ```fill(BASE, 'LINEAR')```
* It can use your own functions in a specified custom script like ```myfunction(BASE)```

### Script
The script is an optional saved custom script with user defined functions.
These functions can then be used in the expression.
The function script can also `import` the default #TimeSeriesScript, so your functions can use those too.

### References to other TimeSeries
You can optionally add references to other TimeSeries or Smart TimeSeries that can also be used in your expression

## Construction

You construct a Smart TimeSeries using one of the following constructors:

```js
// Specify base TimeSeries id and expression
st = SmartTimeSeries("CORN:CLOSE", "fill(BASE,'LINEAR')")

// Specify a start date, calendar and expression
// This can be used to create a TimeSeries that has the same scalar value from the start date till now
// Or you can use an expression to build a TimeSeries
st = SmartTimeSeries("2022-01-01", "MONTHLY", "12.5")
```

## Properties

A SmartTimeSeries has the following properties:

|**Name**|**Description**|**Type**|
|-|-|-|
|baseTimeSeries|The TimeSeries that this Smart TimeSeries uses for a base|ID String, TimeSeries or Reference|
|expression|The expression used to generate the TimeSeries|String|
|script|Optional custom script name used with the expression|String|
|currency|Optional currency of the TimeSeries, defaults to base TimeSeries currency|String|
|units|Optional units of the TimeSeries, defaults to base TimeSeries units|String|
|calendar|The calendar that this timeseries uses|Calendar|
|description|The description of this timeseries|String|
|identity|The configured identities|Object|
|localValues|The list of time observations for this timeseries, returned as a list of TimeValues in the local timezone of this TimeSeries|List|
|name|The name of the timeseries|String|
|observations|The list of values for this timeseries, returned as a list of Vars|List|
|observed|The observed setting of the timeseries used for aggregating|TimeSeriesObserved|
|precision|The precision configuration for this timeseries|Precision|
|source|The source of this timeseries|String|
|start|The starting date/time of the timeseries|Date|
|tenor|The tenor of this timeseries|String|
|timezone|The timezone of this timeseries, defaults to UTC|String|
|valueType|The value type of the timeseries (TRACKED or BASIC)|TimeValueType|
|values|The list of time observations for this timeseries, returned as a list of TimeValues|List|

:::info
If a currency and/or units are set, the base TimeSeries and any other TimeSeries added to this Smart TimeSeries will be converted to the specified currency and units
:::

A SmartTimeSeries also supports adding dynamic properties, e.g.

```js
sc = SmartTimeSeries("SMART_TEST:CLOSE", "BASE*1.02")
sc.product = "CORN"
```

Any dynamic properties added to a SmartTimeSeries can be used in the expression including references to other TimeSeries.
Here are some examples of using properties within the Smart TimeSeries expression:

```js
// Example using another TimeSeries
st1 = SmartTimeSeries("TEST_SMART_TS:ASK", "(BASE+BID)/2")
st1.BID = ref("data","TEST_SMART_TS:BID")

// Example using a numeric factor
st2 = SmartTimeSeries("TEST_SMART_TS:CLOSE", "BASE*factor")
st2.factor=1.2
```

## Methods

A SmartTimeSeries has the following methods:

|**Name**|**Description**|**Return Type**|
|-|-|-|
|build()|Builds a TimeSeries|TimeSeries|
|addCheck(check)|Add a quality check directly to this Timeseries|TimeSeries|
|addValue(value)|Adds a new value to the end of the timeseries using the calendar|TimeSeries|
|applyPrecision()|Apply the precision defined to all the observations of this timeseries|Void|
|convert(currency, units)|Convert this timeseries using currency and units|TimeSeries|
|from(start)|Creates a new TimeSeries from a specified start date|TimeSeries|
|getLast(date)|Get the value at the supplied date or the latest value before that date|Var|
|getLastNValues(n)|Returns a list of the last N values from this TimeSeries|List|
|last(n)|Creates a new TimeSeries using the last n values from this TimeSeries|TimeSeries|
|range(start, end)|Creates a new TimeSeries from a range of values from this TimeSeries|TimeSeries|
|withRange(start, end)|Changes this timeseries to match the provided date range|Void|

Below is an example of creating a Smart TimeSeries and building a TimeSeries using it:

```js
st1 = SmartTimeSeries("TEST_SMART_TS:CLOSE", "BASE*factor)
st1.factor=1.2

t1 = st1.build()
print t1.values
```

## Adding to an object and saving
A SmartTimeSeries needs to be added to an object in order to save it to the database, e.g.

```js
CORN_TEST = Object()
CORN_TEST.category = "TEST"

cf = SmartTimeSeries("TEST_SMART_TS:CLOSE", "interpolate(BASE,'LINEAR')")
CORN_TEST.FILLED = cf

save ${object: CORN_TEST}
```


---

<!-- source: product/developer/odsl/Variables/Spreadsheet.md -->

Spreadsheet
===========================

A spreadsheet represents a set of cells within a row and column bound matrix


---

<!-- source: product/developer/odsl/Variables/Table.md -->

Table
=====================

A table is a set of rows within a column oriented matrix

## Construction

### Constructor functions

|**Function**|**Description**|
|-|-|
|Table()|Creates a new Table|


---

<!-- source: product/developer/odsl/Variables/Task.md -->

A task is a physical task assigned to a user from either a user or the system

## Construction
You create a task using the Task() constructor, e.g.

```js
mytask = Task()
```

## Properties

A task has the following properties:

|**Name**|**Description**|**Type**|
|-|-|-|
|category|The category for this task|String|
|description|A description of this task|String|
|user|The user assigned to this task|String|
|created|The creation timestamp (READ ONLY)|Date|
|assignedBy|The user or system that assigned this task to the user|String|
|complete|The completion state of this task|Boolean|
|completed|The timestamp the task was completed|Date|
|notify|A flag to indicate that the user should be notified by email - defaults to true|Boolean|
|access|Get the name of the access data role for this object|String|

## Methods

A task has the following methods:

|**Name**|**Description**|**Return Type**|
|-|-|-|
|finish()|Marks the task as complete and saves it to the database|Void|


---

<!-- source: product/developer/odsl/Variables/TimeOfDay.md -->

# TimeOfDay

A **TimeOfDay** is a time of the day, optionally in a timezone, for example the time a dataset is expected each day.

## Construction

|**Function**|**Description**|
|-|-|
|TimeOfDay(time)|Creates a new TimeOfDay variable using the supplied String in the format [+]hh:mm [tz]|

## Properties

|**Name**|**Description**|**Type**|
|-|-|-|
|minutes|The number of minutes from midnight|Integer|

## Example

```js
cutoff = TimeOfDay("18:00 Europe/London")
```


---

<!-- source: product/developer/odsl/Variables/TimeSeries.md -->

TimeSeries
==========

A timeseries is a list of values which are indexed by dates. A timeseries could be the price of a particular stock recorded at a certain time of each day (say the time that the stock exchange closes)

## Introduction

A TimeSeries represents a list of observations or metrics where each observed value is indexed by a point in time. Every TimeSeries has a calendar which helps align the observations and make working with TimeSeries much easier.

## Construction

You construct a timeseries using one of the following constructors:

```js
// Creates a TimeSeries aligned to the business calendar
ts1 = TimeSeries("business")

/**
 Creates a TimeSeries aligned to the business calendar 
 with the first value of 12.5 on the 1st October 2020
**/
ts2 = TimeSeries("2020-10-01", "business", 12.5)

/**
 Creates a TimeSeries aligned to the business calendar 
 with a list of values starting on the 1st October 2020
**/
ts3 = TimeSeries("2020-10-01", "business", [12.5,12.6,12.7,12.8,12.9])

/**
 Creates a TimeSeries aligned to the business calendar 
 with the first value of Hello on the 1st October 2020.
 The fourth parameter is the data type
**/
ts4 = TimeSeries("2020-10-01", "business", "Hello", "any")

// Creates a TimeSeries with multiple observations at various time points
ts = TimeSeries(["2020-11-21T08:40:12","2020-09-11T19:27:22"], "SPARSE", [12.5,15.5])
```

## Properties

A timeseries has the following properties:

|**Name**|**Description**|**Type**|
|-|-|-|
|name|The name of the timeseries|String|
|description|A description of this timeseries|String|
|source|The source of this timeseries|String|
|currency|The ISO currency|String|
|units|The ISO units|String|
|tenor|The tenor of this timeseries|String|
|timezone|The timezone, defaults to UTC|String|
|start|The first index of this timeseries|Date|
|calendar|The calendar that this timeseries uses|Calendar|
|observations|The list of values without the indexes|List|
|values|The list of dates and values|List(TimeValue)|
|localValues|The list of dates and values with the dates in the timezone of the timeseries|List(TimeValue)|
|valueType|The value type either TRACKED or BASIC|String|
|observed|The observed setting of the timeseries used for aggregating, can be beginning, end, summed, averaged, high or low|String|
|precision|The data value [precision](https://doc.opendatadsl.com/docs/kb/precision#data-precision-settings) configuration|String|
|identity|The configured identities|Object|

A timeseries also supports adding dynamic properties, e.g.

```js
/**
 Creates a TimeSeries aligned to the business calendar 
 with the first value of 12.5 on the 1st October 2020
**/
ts2 = TimeSeries("2020-10-01", "business", 12.5)

// Add a product property
ts2.product = "Crude Oil"
```

## Methods

A timeseries has the following methods:

|**Name**|**Description**|**Return Type**|
|-|-|-|
|add(Date, value)|Adds a new value to the timeseries at the specified index|This timeseries|
|add(Date, value, List)|Adds a new value to the timeseries at the specified index with a list of status values|This timeseries|
|add(timeseries)|Adds the values of the supplied timeseries to this timeseries|This timeseries|
|add(Date, List)|Adds a list of values to the timeseries starting at the specified index|This timeseries|
|addValue(value)|Adds a new value to the end of this timeseries|This timeseries|
|range(start,end)|Returns a subset of this timeseries for the specified date range|timeseries|
|last(n)|Returns a subset of this timeseries for the last n observations|timeseries|
|from(start)|Returns a subset of this timeseries from the specified start date|timeseries|
|getLastNValues(n)|Returns the last n values of this timeseries|List(TimeValue)|
|addCheck(check)|Adds a quality check to this timeseries|timeseries|
|applyPrecision()|Apply the precision defined to all the observations of this timeseries|Void|
|convert(currency, units)|Convert this timeseries using currency and units|TimeSeries|
|getLast(date)|Get the value at the supplied date or the latest value before that date|Var|
|withRange(start, end)|Changes this timeseries to match the provided date range|Void|

## Accessing Observations

You can treat a timeseries like a special list in order to access the observations using the date as an index, e.g.

```js
/**
 Creates a TimeSeries aligned to the business calendar 
 with a list of values starting on the 1st October 2020
**/
ts3 = TimeSeries("2020-10-01", "business", [12.5,12.6,12.7,12.8,12.9])

// Print out the value for the 2nd October 2020
print ts3["2020-10-02"]
```

If the requested index doesn’t exist, you will get a Missing Value or NaN, e.g.

```js
/**
 Creates a TimeSeries aligned to the business calendar 
 with a list of values starting on the 1st October 2020
**/
ts3 = TimeSeries("2020-10-01", "business", [12.5,12.6,12.7,12.8,12.9])

// The 3rd October 2020 is a Saturday, so there is no value 
print ts3["2020-10-03"]
```

You can also use numeric indexes to retrieve values from a timeseries, e.g.

```js
/**
 Creates a TimeSeries aligned to the business calendar 
 with a list of values starting on the 1st October 2020
**/
ts3 = TimeSeries("2020-10-01", "business", [12.5,12.6,12.7,12.8,12.9])

// Print out the value for the first index
print ts3[0].value
```


---

<!-- source: product/developer/odsl/Variables/TimeValue.md -->

TimeValue
=========================

A TimeValue represents a single observed value in a timeseries

## Properties

A TimeValue has the following properties:

|**Name**|**Description**|**Type**|
|-|-|-|
|time|The time index|DateTime|
|value|The value of this TimeValue|Scalar|
|status|The [status](https://doc.opendatadsl.com/docs/kb/status) of this value|Status|


---

<!-- source: product/developer/odsl/Variables/Transformer.md -->

Transformer
===========================

A wrapper for a data transformer, used with the transformer service

## Properties

|**Name**|**Description**|**Type**|
|-|-|-|
|category|The category of this script, e.g. ETL|String|
|description|A description of what this script does|String|
|filename|The filename of this script|String|
|outputFormat|An optional output format, e.g. application/xml, text/html, text/csv|String|
|script|The script text|String|
|scriptType|The type of this script, e.g. one of [odsl, mustache, py, css, html, js, mjs, cjs]|String|

## Methods

|**Name**|**Description**|**Return Type**|
|-|-|-|
|loadFrom(file)|Load the script from a local file|Void|
|run()|Runs this script|Void|
|run(globalContext)|Runs this script|Void|


---

<!-- source: product/developer/odsl/Variables/Transition.md -->

Transition
==========================

A transition variable is the definition of a transition that is used in a workflow

## Construction
A transition is only constructed as part of a workflow.
The syntax for the transition is as follows:

```js
"name" -> activity_name
```

* name is the name of the transition as defined by the activity you are connecting from
* activity_name is the name of the activity you want to route to

## Properties
A transition has the following properties:

|**Name**|**Description**|**Type**|
|-|-|-|
|id|The name of the transition|Scalar(String)|
|transition|The activity name to transition to|Scalar(String)|

## Further Reading
* Workflow
* Action


---

<!-- source: product/developer/odsl/Variables/Variables.md -->

## Introduction

Variables are used within the language to represent some data, this could simply be a number or it could be a more complex object such as a TimeSeries.

Variables in OpenDataDSL are typeless, meaning you don't need to initialise a variable of a certain type although it is implied by the value you give it or the way you construct it, e.g.

```js
// This creates a String or rather a Scalar variable of type String 
myvar = "Hello World"

// This creates a Date variable (initialised as now) using a constructor
mydate = Date()
```

## Naming Conventions
In OpenDataDSL, you can have both public and private data and objects, so it is important to understand how you reference these in your scripts and objects.

If an id is prefixed with a '#' character denotes that an object is public - which is available to everyone. If not, then the object is private - which is visible only to users within your own tenancy.

Below is an example of how this works with types and objects
```js
// private type
pritype = ${type:"Person"}
// public type
pubtype = ${type:"#Location"}
// private object
johndoe = ${object:"JohnDoe"}
// public object
gb = ${object:"#LOC_GBR"}
```

## Built-in Variable Types

### [Scalar](https://doc.opendatadsl.com/docs/odsl/variable/Scalar)
A scalar is a single value of either string, boolean (true/false) or numeric

### Date
A date variable is a representation of a Date or a Date and Time in the UTC timezone

### Duration
A duration variable represents a period of time such as 3 days or 4 and a half minutes in [ISO 8601 duration format](https://doc.opendatadsl.com/docs/kb/duration)

### Object
An object is a very versatile variable type that allows you to add any other variables to it including Object variable types. This creates a dynamic JSON type structure

### Secret
A secret is a scalar whose value cannot be printed or logged, it only really is useful when used with the Secret Service where secrets can be stored and then retrieved by scripts or workflows. Usually secrets are used to store passwords and database connection information etc.

### Calendar
A calendar variable type represents an [ODSL Calendar](https://doc.opendatadsl.com/docs/odsl/calendar/calendars). It is used with timeseries, curves and other time-bound resources

### Expiry Calendar Variable
An expiry calendar variable type represents an Expiry Calendar which is a special type of calendar that uses rules to calculate when trading stops for a future price for a particular commodity

### Transformer
A wrapper for a data transformer, used with the transformer service
    
### Extractor
A wrapper for an extractor, used by the extractor service
    
### Link
A wrapper variable for creating links or relationships between objects
    
### DeclaredType
A DeclaredType variable is a wrapper for a type and is used by the type service and creating objects
    
### Executable
An executable wrapper variable holds either an expression of a method
    
### Void
This is an empty variable used for methods that don’t return a specific variable
    
### Null
Null represents an empty or no value variable
    
### MissingValue
A MissingValue is a real variable but it doesn’t have a value yet, it is usually used in TimeSeries to represent a TimeValue in a calendar that has no value
    

### List
---------------------------------------
A list represents a list of other variables

VirtualList
A VirtualList is a list that is paged to improve performance and is used with the find command when listing resources from services
    

### TimeSeries
---------------------------------------
A timeseries is a list of values which are indexed by dates. A timeseries could be the price of a particular stock recorded at a certain time of each day (say the time that the stock exchange closes)

### Smart TimeSeries
A Smart TimeSeries is an expression that is used to build TimeSeries on demand

### TimeValue
A TimeValue represents a single observed value in a timeseries
    

### Curve
---------------------------------------
A curve is a structure that represents a set of values with a time index sometime in the future. It can be used for commodity and financial forward curves, yield curves and forecasts (weather, prices etc.)

### EventCurve
A curve that is dynamically built from events

### EventOptions
Options data (an option chain for each maturity) that is dynamically built from events

### Options
The options for one ondate: the tenors, each with an option chain of strikes with the call and put values

### SmartCurve
A Smart Curve is an expression that is used to build curves on demand

### CurveDate
A CurveDate is used as the valuation date of a curve. It is a date with an expiry calendar
    
### Contracts
Contracts are used on a curve as a set of contract objects that represent the future periods and values
    
### Contract
A contract is a single future period in time that represents a tradable future contract or a forecasted value on a curve
    

### Spreadsheet
---------------------------------------

A spreadsheet represents a set of cells within a row and column bound matrix

### Cell
A cell represents a single location-aware cell within a spreadsheet   

### Table
A table is a set of rows within a column oriented matrix

### Row
A row represents a single row of data usually associated with a row in a relational database table. A row is similar to an object in that it has dynamic properties except that you can only add Scalars or Dates as elements to a row

### Matrix
A matrix represents a 2-dimensional array of numeric values

### Workflow
---------------------------------

A workflow represents a set of actions and transitions that form an executable flow diagram

### Action
An action variable is the definition of an action that can be used in a workflow
    
### Arg
An arg variable is a configuration of an input or output argument of a workflow action
    
### Transition
A transition variable is the definition of a transition that is used in a workflow
    
### Process
---------------------------------------

A process variable contains all the information to configure and run a process

### Batch
A batch variable is a custom list variable that contains variables and supplemental information needed to update into the object repository
    
### Script
A script variable is a wrapper for an ODSL script. It is used by the script service and can also run the script
    
### Task
A task is a physical task assigned to a user from either a user or the system

### Report
A report is a configuration containing an expression that can be run to produce a user definable report

### Policy
---------------------------------------

A policy variable allows you to restrict access to data within any service

### TimeOfDay
A time of the day, optionally in a timezone

### Message
A string with a numeric code

### EventTimeSeries
A timeseries built from the events in an event list

### CurveSeries
A curve held as a timeseries of its delivery periods

### ComposedCurve and ComposedTimeSeries
A curve or timeseries that takes its values from one element of a composition

### ReportCreator
A report configuration with parameters that people fill in to create their own reports

### Documentation
Documentation for an item or a service

### Automation
An action that runs when something happens to an item, created with an automation builder

### Alert
An alert about your data: read, raise, comment on and close it with the alert service

### Index
A database index on a service

### GeoSpatial
Points, lines and polygons for geospatial data and queries

### AI Agent, AI Assistant and AI Tool
Fusion AI agents, custom assistants and custom tools

## Common properties and methods

Every variable has these, so they are not repeated on each variable's page:

|**Name**|**Description**|**Type**|
|-|-|-|
|links|The items linked to this variable|List|
|add(link)|Adds a linked item, see Link|Void|

Variables that are versioned when saved, such as objects, scripts, calendars and curves, also have a `version` property with the version information.


---

<!-- source: product/developer/odsl/Variables/VirtualList.md -->

VirtualList
===========================

A VirtualList is a list that is paged to improve performance and is used with the find command when listing resources from services

## Properties

|**Name**|**Description**|**Type**|
|-|-|-|
|description|The description of this list|String|
|name|The name of this list|String|
|size|The size of this list|Any|

## Methods

|**Name**|**Description**|**Return Type**|
|-|-|-|
|addAll(elements)|Adds a list to this list returning a new combined list|List|
|remove(index)|Removes an element from the list at the specified position (0 being first)|Void|
|remove(item)|Removes an element from the list|Void|
|set(index, element)|Replaces the element at the specified position in this list with the specified element|Void|
|subList(start, end)|Gets a list from a sublist of this list|List|


---

<!-- source: product/developer/odsl/Variables/Void.md -->

Void
====================

This is an empty variable used for methods that don’t return a specific variable


---

<!-- source: product/developer/odsl/Variables/Workflow.md -->

Workflow
========================

A workflow represents a set of actions and transitions that form an executable flow diagram

## Construction
A workflow variable is a structure that encapsulates the actions and routing needed to run a workflow.

### Workflow structure syntax
The basic structure of a workflow is shown below:

```js
workflowname = workflow in "category"
    // Description
    (in name as type (description)? (optional)?)*
    (out name as type (description)?)*
    exit ("transition")*

    START
    (PHASE|EVENT)*
end    
```

* workflowname is the variable name that will be the id of the workflow
* category is the group to categorise the workflow with similar workflows
* description is a user description to understand what the workflow does
* the **in** syntax represent the input arguments for this workflow
* the **out** syntax represents the output data for this workflow
* the **exit** syntax represents the list of transition names from this workflow
* START is the Start event syntax
* PHASE is the Phase syntax
* EVENT is the Event syntax

### Start event syntax
The start event represents the initial point of the workflow which then routes using a transition to the first activity to run.

The **START** event is defined as:

```js
WF_START
    TRANSITION
end
```

* TRANSITION is the Transition syntax which routes to the first activity to run.

### Phase syntax
A phase represents a section of the workflow that is independently tracked and contains all the workflow activities.

The **PHASE** structure is defined as:
 
```js
phase "name" (retries int)? (delay int timeunit)? (then reschedule in timeunit)? (external)?
    (statement|WORKFLOW_BODY)*
end
```

* retries is the number of attempts to retry this phase if it fails, default is 0
* delay is the amount of time to delay before retrying, default is 30 seconds
* reschedule will happen if the retries have been exhausted and will re-run the entire workflow
* external indicates that this phase runs external to the workflow and will be reported independently
* statement is any OpenDataDSL statement
* WORKFLOW_BODY is the Workflow body syntax

### Event syntax
Events are generally used to mark the termination of a workflow to indicate success or failure.

The **EVENT** structure is defined as:

```js
WF_EVENT name as "success|failed"
    (assign|return|TRANSITION)*
end
```

* name is the activity name which can be routed to using a transition
* assign is an assignment statement, e.g. output.message = "ok"
* return is a return statement, e.g. return "ok"
* TRANSITION is the Transition syntax indicating where to route to next

### Workflow body syntax
This is the main list of activities performed by the workflow.

The **WORKFLOW_BODY** syntax is defined as:

```js
(EVENT|ACTION|GATEWAY|WORKFLOW)
```

* EVENT is an Event syntax
* ACTION is an Action syntax
* GATEWAY is a Gateway syntax
* WORKFLOW is a Workflow syntax

### Transition syntax
A transition represents a connector between activities which defines the routing within the workflow.

The **TRANSITION** syntax is defined as:

```js
"name" -> activity_name
```

* name is the name of the transition as defined by the activity you are connecting from
* activity_name is the name of the activity you want to route to

### Action syntax
An action is a configuration of a workflow action within the workflow.
 
The **ACTION** syntax is defined as:

```js
WF_ACTION activity_name input
    (assign|TRANSITION)*
end
```

* activity_name is the unique name (within the workflow) that you give to this activity, this will be used to route the flow to
* input is the name of the variable to assign input data to, this will be used in the configuration of the action
* assign is an assignment statement, usually used for assigning input data to the input arguments on the action
* TRANSITION is the Transition syntax used to map the exit transitions of the action to activities in this workflow

### Gateway syntax
A gateway is a special type of Action used as a decision node within a workflow.

The **GATEWAY** syntax is defined as:

```js
WF_GATEWAY activity_name input
    (assign|TRANSITION)*
end
```

* activity_name is the unique name (within the workflow) that you give to this activity, this will be used to route the flow to
* input is the name of the variable to assign input data to, this will be used in the configuration of the gateway
* assign is an assignment statement, usually used for assigning input data to the input arguments on the gateway
* TRANSITION is the Transition syntax used to map the exit transitions of the gateway to activities in this workflow

### Workflow syntax
The workflow syntax within a workflow is used to configure another workflow as a sub-workflow.

The **WORKFLOW** syntax is defined as:

```js
WF_WORKFLOW activity_name input
    (assign|TRANSITION)*
end
```

* activity_name is the unique name (within the workflow) that you give to this activity, this will be used to route the flow to
* input is the name of the variable to assign input data to, this will be used in the configuration of the sub-workflow
* assign is an assignment statement, usually used for assigning input data to the input arguments on the sub-workflow
* TRANSITION is the Transition syntax used to map the exit transitions of the sub-workflow to activities in this workflow

## Properties
A workflow variable has the following properties:

|**Name**|**Description**|**Type**|
|-|-|-|
|category|A category is used to group similar workflows together|Scalar(String)|
|description|A description of what this workflow does|Scalar(String)|
|exits|The exit transitions from this workflow|List(String)|
|id|The ID of this action|Scalar(String)|
|inputs|The input variable args|List(Arg)|
|outputs|The output variable args|List(Arg)|
|script|The actual code used to create this workflow|Scalar(String)|
|version|The version of this workflow|Object|
|filename|The filename of this script|String|
|outputFormat|An optional output format, e.g. application/xml, text/html, text/csv|String|
|scriptType|The type of this script, e.g. one of [odsl, mustache, py, css, html, js, mjs, cjs]|String|

## Methods
A workflow variable has the following methods:

|**Name**|**Description**|**Return Type**|
|-|-|-|
|run(in, out)|Runs this workflow and returns the name of the exit transition|Scalar(String)|
|loadFrom(file)|Load the script from a local file|Void|

### Example Usage
You can run a workflow in ODSL code by creating an input object and an output object.
The input object should contain the **in** arguments required by the workflow and the output object should be empty.

When the workflow runs, it populates the output object with any output arguments defined.

```js
oi = Object()
oi.user = "World"
oo = Object()
workflow.run(oi, oo)
print oo
```

Example output:
```json
{
  "_id": "oo",
  "_type": "#Object",
  "_links": {},
  "workflow": {
    "_id": "output",
    "act_print": {
      "_id": "output",
      "message": "Hello World"
    }
  }
}
```

## Further Reading
* [Workflow service](https://doc.opendatadsl.com/docs/odsl/service/workflow) for saving and using workflows in the server.
