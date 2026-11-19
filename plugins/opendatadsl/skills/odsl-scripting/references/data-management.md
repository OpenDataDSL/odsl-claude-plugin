# Data management with ODSL

Transformers, workflows and processes, data modelling with types, versioning, identities, events, queues, monitoring.

Generated from the OpenDataDSL documentation (https://doc.opendatadsl.com/docs).


---

<!-- source: topics/data-management/Currency-Providers.md -->

# Currency Providers

Whenever OpenDataDSL converts a curve or timeseries to another currency, it needs a set of exchange rates to do it. The set of rates it uses is the **currency provider**: an FX dataset such as `ECB_FX` (European Central Bank reference rates) or `BOE_FX` (Bank of England daily spot rates).

By default every conversion uses `ECB_FX`. This page explains how to choose a different provider for your whole organisation, for a single Smart Curve or Smart TimeSeries, for a Composer worksheet or column, or for a single request.

---

## When you need a different provider

The ECB reference rates are a good default, but they are not always the right rates for the job:

- Your finance team reports in GBP and expects Bank of England rates, so every converted price should agree with their numbers.
- Your trades settle against your bank's daily rates, for example `ABN_FX`, and your positions should be valued with the same rates.
- You hold your own rates, such as month-end rates agreed with your auditors, and want specific curves converted with those.
- One curve needs a different source from the rest, for example an Indian product converted with Reserve Bank of India rates (`RBI_FX`).

The public FX datasets you can choose from are listed on the [Foreign Exchange Rates](https://doc.opendatadsl.com/docs/data/fx) page.

---

## How the provider is chosen

When a value is converted, the provider is taken from the first of these that is set:

| Order | Where it is set | Applies to | Example |
|-|-|-|-|
| 1 | The target currency itself, as `CURRENCY@DATASET` | That one conversion | `GBP@BOE_FX` |
| 2 | The `CCY_DATASET` property of a Smart Curve or Smart TimeSeries, or the **FX provider** in Composer | Everything that Smart Curve, Smart TimeSeries or Composer column converts | `BOE_FX` |
| 3 | The `CCY_DATASET` property of your tenant | Every conversion in your organisation | `BOE_FX` |
| 4 | The platform default | Everything else | `ECB_FX` |

So you can set your organisation's default once on the tenant, and still override it for a curve or a single request.

:::note
Only currency conversion uses a provider. Unit and timezone conversions are not affected by any of these settings.
:::

---

## Setting your organisation's default

Add a `CCY_DATASET` property to your tenant record with the id of the FX dataset to use. For example, to use Bank of England rates for every conversion:

| Property | Example | Notes |
|-|-|-|
| `CCY_DATASET` | `BOE_FX` | The FX dataset used for every conversion that doesn't choose its own |
| `CCY_DATASET_SOURCE` | `public` | Optional. `public` (the default) for a platform dataset, `private` for one you load yourself |

You update the tenant record with the [Tenant Service](https://doc.opendatadsl.com/docs/api/rest/service/tenant):

```js
POST https://api.opendatadsl.com/api/tenant/v1

{
  "properties": {
    "name": "Acme Energy Trading Ltd",
    "address1": "1 Canary Wharf",
    "city": "London",
    "postcode": "E14 5AB",
    "CCY_DATASET": "BOE_FX"
  }
}
```

:::caution
The tenant service replaces the whole `properties` object; it does not merge it. Read your tenant first with `GET https://api.opendatadsl.com/api/tenant/v1` and send every existing property back along with the new one. Any property you leave out, such as your address or Fusion AI budget settings, is removed.
:::

The new default applies to conversions started after the change, although it can take a few minutes to reach every server. A process or build that is already running keeps the provider it started with.

---

## Choosing a provider for a Smart Curve or Smart TimeSeries

A Smart Curve or Smart TimeSeries with a `currency` converts its inputs to that currency when it is built. To use a different provider for those conversions, add a `CCY_DATASET` property to it:

```js
sc = SmartCurve("#ICE.NBP:FUT", "BASE")
sc.currency = "EUR"
sc.CCY_DATASET = "BOE_FX"
```

The same works on a Smart TimeSeries:

```js
sts = SmartTimeSeries("#ICE.NBP:SPOT", "BASE")
sts.currency = "EUR"
sts.CCY_DATASET = "BOE_FX"
```

If the provider is a private dataset, also set `CCY_DATASET_SOURCE` to `private`:

```js
sc.CCY_DATASET = "ACME_FX"
sc.CCY_DATASET_SOURCE = "private"
```

A Smart Curve or Smart TimeSeries without these properties uses your tenant's default, or `ECB_FX` if your tenant doesn't set one.

---

## Choosing a provider in Composer

In a Composer worksheet, the **FX provider** setting under **Conversion defaults** sets the provider for every column on the worksheet, and a column's own **FX provider** overrides it for that column. Leave it empty to use your organisation's default. See [Composer workspace](https://doc.opendatadsl.com/docs/extensions/composer/workspace) for the conversion settings.

:::note
The Composer **FX provider** setting picks a public FX dataset. To convert with your own private rates in Composer, make the private dataset your tenant's default, as described in [Using your own rates](#using-your-own-rates).
:::

---

## Choosing a provider for a single conversion

Anywhere you give a target currency, you can add the provider after an `@`. This always wins over the Smart Curve, Composer and tenant settings.

The full format is `CURRENCY:PROFILE@DATASET`, where the profile and the dataset are both optional:

| Value | Meaning |
|-|-|
| `GBP` | Convert to GBP with the default provider and the `SPOT` profile |
| `GBP@BOE_FX` | Convert to GBP with Bank of England rates |
| `GBP:SPOT@BOE_FX` | The same, with the profile given explicitly |

:::caution
Put the profile before the `@`. `GBP@BOE_FX:SPOT` is read as a dataset called `BOE_FX:SPOT`, which doesn't exist.
:::

In the REST API, use it in the `currency` query parameter of the [Data Service](https://doc.opendatadsl.com/docs/api/rest/service/data):

```js
GET https://api.opendatadsl.com/api/data/v1/public/%23ICE.TTF:SPOT?currency=GBP@BOE_FX
```

In ODSL, use it in the `convert` method of a curve or timeseries:

```js
ttf = ${data:"#ICE.TTF:SPOT"}
ttf_gbp = ttf.convert("GBP@BOE_FX", "MWh")
```

The second argument of `convert` is the target units; giving the series' own units converts the currency only.

---

## Using your own rates

You can load your own rates as a private FX dataset and use it as a provider. The platform finds rates in the currency service, so your data needs to look like the public FX datasets:

| Requirement | Details |
|-|-|
| Type | Each currency pair is an object of type `#ForeignExchange` |
| `dataset` | The dataset id, for example `ACME_FX`. This is the value you use as the provider |
| `base` | The base currency of the pair, for example `EUR` |
| `currency` | The quote currency of the pair, for example `USD` |
| Rates | A timeseries on the pair (for example `ACME_FX.EURUSD:SPOT`) holding the amount of `currency` per 1 `base` |

Then point the conversions at it with `CCY_DATASET_SOURCE` set to `private`, for example on your tenant:

| Property | Value |
|-|-|
| `CCY_DATASET` | `ACME_FX` |
| `CCY_DATASET_SOURCE` | `private` |

:::note
The tenant's `CCY_DATASET_SOURCE` is only used when its own `CCY_DATASET` is the provider. If a Smart Curve or a request picks a public dataset such as `BOE_FX`, those rates are still read from the public data, so a private tenant default doesn't break other conversions.
:::

---

## How rates are found

For a conversion from one currency to another, the provider is searched in this order:

| Step | Looks for | Example converting USD to GBP with `ECB_FX` |
|-|-|-|
| 1 | The direct pair | A pair with base `USD` and currency `GBP` |
| 2 | The inverse pair, using 1 / rate | A pair with base `GBP` and currency `USD` |
| 3 | A cross rate through the dataset's base currency | `EUR` to `GBP` divided by `EUR` to `USD` |

A provider therefore doesn't need every pair: a dataset quoted against one base currency, like `ECB_FX` against EUR or `BOE_FX` against GBP, can convert between any two of its currencies.

Each value is converted with the rate for its own date: a timeseries value uses the rate on that day, and a curve uses the rate on the curve date.

| What happens | Why |
|-|-|
| The converted value is `NaN` | The provider has the pair but no rate for that date, for example a bank holiday |
| The conversion fails with "Currency for conversion between ..." | The provider has no way to convert between the two currencies |

:::tip
If converted values come out as `NaN` on some days, check the provider's calendar. Central bank datasets only publish on their own business days, so a different provider may cover the dates you need.
:::

---

## Summary

| To | Set |
|-|-|
| Change the provider for your whole organisation | `CCY_DATASET` on the tenant |
| Use your own rates for your whole organisation | `CCY_DATASET` and `CCY_DATASET_SOURCE` = `private` on the tenant |
| Change the provider for one Smart Curve or Smart TimeSeries | `CCY_DATASET` (and `CCY_DATASET_SOURCE` for private rates) on it |
| Change the provider in Composer | **FX provider** on the worksheet or column |
| Change the provider for one request or calculation | `CURRENCY@DATASET` as the target currency |


---

<!-- source: topics/data-management/Data Transformation.md -->

Data Transformation
===================================

Transformers are a simple definition that allows you to map data from a single [object](https://doc.opendatadsl.com/docs/odsl/variable/object) to a [list](https://doc.opendatadsl.com/docs/odsl/variable/list) of [objects](https://doc.opendatadsl.com/docs/odsl/variable/object) or [rows](https://doc.opendatadsl.com/docs/odsl/variable/row)

## Introduction

Usually, the most complex part of an ETL process is the T - the transformation. This involves mapping the source data into the destination data structure. The source data could be any structured or unstructured format and therefore any software used to automate that transformation needs to be extremely flexible.

In the previous section, we introduced data modelling which, in most cases, will be the destination data structure for the transformation.

This guide takes you through the process of understanding and using ODSL [transformers](https://doc.opendatadsl.com/docs/odsl/variable/transformer) as a way of mapping extracted data into your [declared types](https://doc.opendatadsl.com/docs/odsl/variable/DeclaredType).

## Syntax

A transformer is defined using the following syntax:
```
name = transform input into (type|object|rows) as varname
  // Transformer Description
  (transformOptions)?
  (assign|method|if)*
end
```

### Transformer Definition

The first line in the creation of a transformer contains the input, output and flow information.

#### Input definition

The input to a transformer must be either an [object](https://doc.opendatadsl.com/docs/odsl/variable/Object) or a [list](https://doc.opendatadsl.com/docs/odsl/variable/List) and is usually the resultant variable of using an extract service.

#### Output definition

The output can be one of the following:

*   A named [declared type](https://doc.opendatadsl.com/docs/odsl/variable/DeclaredType) - the output will be a list of objects of the declared type
    
*   The word object - in which case it will transform the input into a list of generic [objects](https://doc.opendatadsl.com/docs/odsl/variable/Object)
    
*   The word rows - in which case it will transform the input into a list of generic [rows](https://doc.opendatadsl.com/docs/odsl/variable/Row)
    

#### Flow definition

The declaration of the ‘as varname' will be the variable name used for each element of the input as it traverses the input structure.

### Transformer Options

Transformer Options can be any of:

create with varname (, varname)*
unique property = value
on error ignore
ignore condition (, condition)*

#### Create with option

This is used to define the name of the input structure element that triggers the creation of a new output entity. If no create with option is defined, it uses the unique option to determine element creation.

#### Unique option

This is used to define a unique key which helps the transformer determine which output element to add data to. When the value of the evaluated key changes during the transformer flow, a new element is created.

#### On error option

If the ‘on error ignore’ option is set, any errors during the transformer body stage are logged and ignored.

#### Ignore option

The ignore option allows you to create a condition or set of conditions, which if any are true, will ignore and discard the current output element

### Transformer Body

The body of a transformer can be an assignment, a method or an if/end structure containing other assign, method or if structures.

#### Body assignment

This represents a single direct mapping of an input structure element to an output data property.

#### Body method

This represents a method call, e.g. if you have created a custom method on your declared type or a standard or user defined function

#### Body if structure

The is used to check a condition and follow different routes within the tranformer.

## Examples

### Simple static example

This simple example shows all of the features of a transformer and can be easily edited and tested using the [VSCode extension](https://doc.opendatadsl.com/docs/user/vscode).

Here is the preparation, we create the input data (this example just uses a static object with some data), our output data type and some metadata which we can use to decorate our output data:

```js
// Example input data structure
json = `{
    timestamp: "12Oct2020",
    data: \[
        {
            identity: "A",
            value: 1.2
        },
        {
            identity: "B",
            value: 1.5
        },
        {
            identity: "C",
            value: 999
        }
    \]
}`

input = ${json:json}

// Create our output data type
example = type
    name as String()
    description as String()
    value as Scalar()
end

// Create some metadata
metadata = Object()
metadata.A = "A description"
metadata.B = "B description"
```
Our transformer is then defined and run as follows:
```js
// Create the transformer
tx = transform input into example as x
    create with data
    unique name = x.identity
    on error ignore
    ignore x.identity == "C"
    name = x.identity
    description = metadata.get(x.identity)
    value = x.value
end

// Run the transformer
result = tx.run(input)
```

This will create a list with 2 objects of type ‘example’

If we wanted to convert the input into a list of rows that we can load into a relational database, we can use a transformer like this:

```js
rowtx = transform input into rows as x
    create with data
    unique name = x.identity
    on error ignore
    ignore x.identity == "C"
    name = x.identity
    description = metadata.get(x.identity)
    value = x.value
end
```

You can see, in this case, it is exactly the same apart from we specify we are transforming the input into ‘rows’

This creates the following output:


---

<!-- source: topics/data-management/Data-Auditing.md -->

Data Auditing
=============

This guide takes you through the extensive auditing within OpenDataDSL and how you can utilise it to monitor what has changed in the system and track changes over time

## Introduction
Audit records are created for every action performed by users on the data stored in OpenDataDSL.

## Information Recorded
### Audit Record
An audit record contains the following information:

|**Name**|**Description**|**Type**|
|-|-|-|
|_id|Unique identifier|ObjectID|
|action|The action performed|String|
|service|The name of the service|String|
|source|The source of the data|String|
|user|The user id who performed the action|String|
|invocationId|The request id for the action|String|
|description|A description or reason for the change|String|
|key|The ID of the entity affected by the action|String|
|version|The version number of the affected entity|Number|
|ref|A link to the entity affected by the action|Reference|
|status|The HTTP status of the action|Number|
|timestamp|The timestamp of when this action occurred|Datetime|
|changes|The actual changes that were made|Changes Object|

### Changes Record
The changes information in the audit record contains the following information:

|**Name**|**Description**|**Type**|
|-|-|-|
|kind|The type of change|String - N=New, E=Edited, D=Deleted|
|lhs|The old value|Any|
|rhs|The new value|Any|
|path|The path to the changed value in the entity|String array|

### Example Audit Record

The following example audit record is the recorded information after the **name** property in an object called **AUDIT01** 
is changed from *'Audit Test'* to *'Audit Testing'*  

```json
{
  "_id": "650abb58baa8e12c386fccb2",
  "action": "update",
  "changes": [
    {
      "kind": "E",
      "lhs": "Audit Test",
      "path": [
        "name"
      ],
      "rhs": "Audit Testing"
    }
  ],
  "invocationId": "aeca52ba-3c7b-47e8-94b3-813cdec26dd1",
  "key": "AUDIT01",
  "ref": {
    "_type": "VarReference",
    "_service": "object",
    "_oid": "AUDIT01"
  },
  "service": "object",
  "source": "private",
  "status": 200,
  "timestamp": "2023-09-20T09:28:56.559",
  "user": "user@example.com",
  "version": 0
}
```


---

<!-- source: topics/data-management/Data-Events.md -->

This guide explains how to load data to your own data models using events and then define curves using that data.

## Introduction

Events are a thing that has happened at a point in time, similar to an observation in a TimeSeries only with a lot more information.

Example events are:

* An order placed with a broker, exchange etc.
* A trade made with a broker, exchange etc.
* A planned or unplanned outage (REMIT Urgent Market Message)

## An Event

This section describes the features of an event. 

### Event Time
An event has an event time - this can be used as the time that event was captured or, in the case of futures, the trade/curve/ondate for the valuation.

### Start and End Time
An event has a start and end time, this is used to specify the date/time range that the event covers.

### Event List Id
This is the id that relates all similar events together

### Event Type
Much like objects (master data), events have types which define the basic structure and properties of the event.

## Creating an Event Type
Creating an event type is the same as creating an object type, you then simply need to add the boolean property **event** as true.

### Example Event Type

**OpenDataDSL:**

```js
TestEventType = type
	PRICE as number()
end
TestEventType.event = true
save TestEventType
```

**REST API:**

```js
POST https://api.opendatadsl.com/api/type
Authorization: Bearer {{token}}

{
  "_id": "TestEventType",
  "_type": "Type",
  "_name": "Test Event Type",
  "_event": true,
  "PRICE": {
    "name": "PRICE",
    "type": "number",
    "_ptype": "Property"
  }
}
```

## Creating Events
Events need to be added to master data in the same way as timeseries, curves etc.

### Use of a DSID
If you intend to use the dataset monitoring tools, you can add a _dsid (Dataset ID) property to your events.
A _dsid comprises of 3 sections separated by dots - ```PROVIDER.FEED.PRODUCT```

For example: ODSL.TRADER1.NBP

All events for the same combination of provider, feed and product should use the same _dsid.

:::info
It is recommended that you use your own company name/code for the provider so as not to be confused with data providers 
:::

### Create events for use with curves

**OpenDataDSL:**

You can create events using curve date and tenor which will populate the eventtime, eventstart, eventend, absolute, relative and expiry fields automatically.

```js
expcal = ${expiry:"#REOD"}
ondate = CurveDate("2024-05-30", expcal)

e1 = Event(ondate, "M01")
e1.id = "ID_2024-05-30_M01"
e1._dsid = "ODSL.TRADER1.NBP"
e1.eventtype = "TestEventType"
e1.PRICE = 35.025
e2 = Event(ondate, "M02")
e2.id = "ID_2024-05-30_M02"
e2._dsid = "ODSL.TRADER1.NBP"
e2.eventtype = "TestEventType"
e2.PRICE = 35.125
e3 = Event(ondate, "M03")
e3.id = "ID_2024-05-30_M03"
e3._dsid = "ODSL.TRADER1.NBP"
e3.eventtype = "TestEventType"
e3.PRICE = 35.155

EXAMPLE = Object()
EXAMPLE.EVENTS = [e1, e2, e3]

save EXAMPLE
```

**REST API:**

```js
POST https://api.opendatadsl.com/api/object
Authorization: Bearer {{token}}

{
  "_id":"EXAMPLE",
  "_type":"#Object",
  "EVENTS":[
    {
      "_type": "VarEvent",
      "_id": "ID_2024-05-30_M01",
      "_dsid": "ODSL.TRADER1.NBP",
      "eventtype": "TestEventType",
      "eventtime": "2024-05-30",
      "eventstart": "2024-06-01",
      "eventend": "2024-06-30",
      "absolute": "2024M06",
      "relative": "M01",
      "PRICE": 35.025
    },
    {
      "_type": "VarEvent",
      "_id": "ID_2024-05-30_M02",
      "_dsid": "ODSL.TRADER1.NBP",
      "eventtype": "TestEventType",
      "eventtime": "2024-05-30",
      "eventstart": "2024-07-01",
      "eventend": "2024-07-31",
      "absolute": "2024M07",
      "relative": "M02",
      "PRICE": 35.125
    },
    {
      "_type": "VarEvent",
      "_id": "ID_2024-05-30_M03",
      "_dsid": "ODSL.TRADER1.NBP",
      "eventtype": "TestEventType",
      "eventtime": "2024-05-30",
      "eventstart": "2024-08-01",
      "eventend": "2024-08-31",
      "absolute": "2024M08",
      "relative": "M03",
      "PRICE": 35.155
    }
  ]
}
```

## Creating Event Curves
An event curve dynamically creates curves from an event list.
In order to use events for curves, they must have at a minimum:
* The **eventtime** must represent the curve date
* It must have a property named **relative** containing a relative [period code](https://doc.opendatadsl.com/docs/kb/pc)
* It must have a property named **absolute** containing an absolute [period code](https://doc.opendatadsl.com/docs/kb/pc)
* It can optionally have a property called **expiry** containing the last trade date of the tenor, if this exists, the contract will use the eventstart and eventend properties as the start and end of delivery  

**OpenDataDSL:**

The EventCurve is constructed using the following properties:
* Event list id
* Expiry Calendar
* Property on the event containing the values
* Property on the event containing the absolute or relative tenor

```js
EXAMPLE = Object()
EXAMPLE.PRICE = EventCurve("EXAMPLE:EVENTS", "#REOD", "PRICE", "absolute")
save EXAMPLE
```

**REST API:**

```js
POST https://api.opendatadsl.com/api/type
Authorization: Bearer {{token}}

{
  "_id": "EXAMPLE",
  "_type": "#Object",
  "PRICE": {
    "_id": "PRICE",
    "_type": "VarEventCurve",
    "event": "EXAMPLE:EVENTS",
    "property": "PRICE",
    "tenor": "absolute",
    "calendar": "#REOD"
  }
}
```


---

<!-- source: topics/data-management/Data-Identities.md -->

# Data Identities

## The Problem

When you push data from OpenDataDSL to a downstream system via a queue, webhook, or automation, the item's OpenDataDSL ID is unlikely to match the ID that the target system uses. For example:

- OpenDataDSL ID: `ICE.NBP_M.NBP:SETTLE`
- SAP commodity ID: `GAS-UK-NBP-01`
- Internal ETRM ID: `C2`

Without identities, downstream applications must maintain their own mapping tables to translate between OpenDataDSL IDs and their own references — adding complexity and a second source of truth that can drift out of sync.

## The Solution

The `identity` property on any data item holds a simple key-value map of system names to IDs. When the data is delivered to a downstream system, the identity map travels with it, so the receiving application can look up its own identifier directly from the payload.

```
data item
└── identity
    ├── sap        → "GAS-UK-NBP-01"
    ├── molecule   → "C2"
    └── bloomberg  → "NBPGBP Comdty"
```

The identity property is fully dynamic — you define the system names yourself, and each data item can have as many or as few identities as needed.

---

## Supported Types

Identities can be added to the following data item types:

| Type | Description |
|------|-------------|
| `TimeSeries` | Regular time series |
| `SmartTimeSeries` | Derived / calculated time series |
| `Curve` | Forward curve |
| `SmartCurve` | Derived / calculated forward curve |
| `CurveSeries` | Series of curves |
| `EventTimeSeries` | Time series of events |
| `EventCurve` | Curve of events |

---

## Adding Identities

### Smart Curve

**ODSL:**

```js
sc = SmartCurve("AAA_IDENTITY:CURVE", "BASE*1.1")
sc.identity.molecule = "C2"
sc.identity.sap = "S2"

AAA_IDENTITY = Object()
AAA_IDENTITY.SCURVE = sc
save AAA_IDENTITY
```

**REST:**

```json
POST https://api.opendatadsl.com/api/object/v1
Authorization: Bearer {{token}}

{
  "_id": "AAA_IDENTITY",
  "SCURVE": {
    "_type": "VarSmartCurve",
    "expression": "BASE*1.1",
    "input": "AAA_IDENTITY:CURVE",
    "_identity": {
      "molecule": "C2",
      "sap": "S2"
    }
  }
}
```

### TimeSeries

**ODSL:**

```js
ts = TimeSeries("BUSINESS")
ts.identity.molecule = "TS-MOL-01"
ts.identity.sap = "GAS-UK-TS-01"

MY_OBJECT = Object()
MY_OBJECT.PRICE = ts
save MY_OBJECT
```

**REST:**

```json
POST https://api.opendatadsl.com/api/object/v1
Authorization: Bearer {{token}}

{
  "_id": "MY_OBJECT",
  "PRICE": {
    "_type": "VarTimeSeries",
    "calendar": "BUSINESS",
    "_identity": {
      "molecule": "TS-MOL-01",
      "sap": "GAS-UK-TS-01"
    }
  }
}
```

### Curve

**ODSL:**

```js
ondate = CurveDate(Date("2026-03-02"), "#REOMHENG")
c = Curve(ondate)
c.add("M01", 1.4)
c.add("M02", 2.2)
c.add("M03", 3.3)
c.identity.molecule = "C1"
c.identity.sap = "S1"

AAA_IDENTITY = Object()
AAA_IDENTITY.FORWARD = c
save AAA_IDENTITY
```

**REST:**

```json
POST https://api.opendatadsl.com/api/object/v1
Authorization: Bearer {{token}}

{
  "_id": "AAA_IDENTITY",
  "FORWARD": {
    "_type": "VarCurve",
    "ondate": {
      "curveDate": "2024-01-15",
      "expiryCalendar": "#REOMHENG"
    },
    "contracts": [
      {"tenor": "M01", "value": 1.4},
      {"tenor": "M02", "value": 2.2},
      {"tenor": "M03", "value": 3.3}
    ],    
    "_identity": {
      "molecule": "FWD-MOL-01",
      "bloomberg": "NBPGBP Comdty"
    }
  }
}
```

### Smart TimeSeries

**ODSL:**

```js
st = SmartTimeSeries("MY_BASE_TS:PRICE", "BASE*1.05")
st.identity.sap = "DERIVED-SAP-ID"
st.identity.etrm = "ETRM-REF-99"

MY_OBJECT = Object()
MY_OBJECT.DERIVED = st
save MY_OBJECT
```

**REST:**

```json
POST https://api.opendatadsl.com/api/object/v1
Authorization: Bearer {{token}}

{
  "_id": "MY_OBJECT",
  "DERIVED": {
    "_type": "VarSmartTimeSeries",
    "input": "MY_BASE_TS:PRICE",
    "expression": "BASE*1.05",
    "_identity": {
      "sap": "DERIVED-SAP-ID",
      "etrm": "ETRM-REF-99"
    }
  }
}
```

:::note
The system names you use as keys — `sap`, `molecule`, `bloomberg`, `etrm` — are entirely up to you. Use names that are meaningful to your team and consistent across your data objects.
:::

---

## Reading Identities

**ODSL:**

```js
// Read the full identity map
item = ${data:"MY_OBJECT:SCURVE"}
print item.identity

// Read a specific system's identity
print item.identity.sap
```

**REST:**

```js
// Read the full data item — identity is included in the response
GET https://api.opendatadsl.com/api/data/v1/private/MY_OBJECT:SCURVE
Authorization: Bearer {{token}}
```

---

## Searching Using an Identity

You can search for data items by matching against an identity value using the `find` command:

**ODSL:**

```js
ts = find ${data} where _identity.molecule="C1"
```

**REST:**

```js
GET https://api.opendatadsl.com/api/data/v1/private?_filter={"_identity.molecule":"C1"}
Authorization: Bearer {{token}}
```

This returns all data items where the `molecule` identity key equals `"C1"`, regardless of their OpenDataDSL ID. This is particularly useful when you know the downstream system's identifier but not the native OpenDataDSL path.

---

## Retrieving a TimeSeries by Identity

You can retrieve a time series directly using an identity value as the lookup key:

**ODSL:**

```js
ts1 = ${data:"private.identity.molecule"/"test"}
```

**REST:**

```js
GET https://api.opendatadsl.com/api/data/v1/private.identity.molecule/test
Authorization: Bearer {{token}}
```

The path format is `"private.identity.<key>"/"<value>"`, where:
- `<key>` is the identity system name (e.g. `molecule`, `sap`)
- `<value>` is the identifier you are looking up

---

## Retrieving a Curve by Identity

You can retrieve a curve using an identity value, with an optional `_ondate` parameter to specify the curve date:

**ODSL:**

```js
c1a = ${data:"private.identity.molecule"/"C1","_ondate=2026-03-02"}
```

**REST:**

```js
GET https://api.opendatadsl.com/api/data/v1/private.identity.molecule/C1?_ondate=2026-03-02
Authorization: Bearer {{token}}
```

The same `"private.identity.<key>"/"<value>"` path format applies. Add `_ondate=<date>` to retrieve the curve as of a specific date.

---

## Updating Identities

**ODSL:**

To update or add an identity on an existing data item, fetch the raw data item, set the new value, and save it back:

```js
// Fetch the data item with raw flag to preserve identity data
obj = ${object:"AAA_IDENTITY", "_raw=true"}

// Add or update a single identity
obj.SCURVE.identity.bloomberg = "NEW-BLOOMBERG-ID"
save obj
```

**REST:**

To add/update an identity key, use the `identity` function with the `PUT` action:

```json
PUT https://api.opendatadsl.com/api/data/v1/private?_function=identity
Authorization: Bearer {{token}}

{
  "action":"PUT",
  "data": [
      "_id":"AAA_IDENTITY:SCURVE",
      "bloomberg":"NEW-BLOOMBERG-ID"  
  ]
}
```

**ODSL:**

To remove an identity key, use the `.remove()` method:

```js
obj = ${object:"AAA_IDENTITY", "_raw=true"}
obj.SCURVE.identity.remove("bloomberg")
save obj
```

**REST:**

To remove an identity key, use the `identity` function with the `DELETE` action:

```json
PUT https://api.opendatadsl.com/api/data/v1/private?_function=identity
Authorization: Bearer {{token}}

{
  "action":"DELETE",
  "_id": ["AAA_IDENTITY:SCURVE"],
  "identity":["bloomberg"]
}
```

---

## Identities in Downstream Delivery

When an automation delivers data to a queue, webhook, or script target, the `_identity` map is included in the message payload. The receiving system can read its own key directly, with no external mapping table required.

### Example queue message payload (JSON)

```json
{
  "_id": "AAA_IDENTITY:SCURVE",
  "_type": "VarSmartCurve",
  "expression": "BASE*1.1",
  "_identity": {
    "molecule": "C2",
    "sap": "S2"
  }
}
```

### Reading the identity in a receiving script

If your automation uses a Script target, you can access the identity directly:

```js
// input.data contains the delivered data item
item = input.data
sapId = item.identity.sap
print "Delivering to SAP with ID: " + sapId
```

### Reading the identity in a Mustache template

If your automation uses an Email, Webhook, or Queue target with a Mustache template, reference identities using standard Mustache syntax:

```html
<p>SAP ID: {{identity.sap}}</p>
<p>Molecule ID: {{identity.molecule}}</p>
```

---

## Multiple Data Items on One Object

A single master data object often holds several data items, each with their own identities:

**ODSL:**

```js
MY_OBJECT = Object()

ts = TimeSeries("BUSINESS")
ts.identity.molecule = "TS-MOL-01"
ts.identity.sap = "GAS-UK-TS-01"
MY_OBJECT.PRICE = ts

c = Curve(ondate)
c.identity.molecule = "FWD-MOL-01"
c.identity.bloomberg = "NBPGBP Comdty"
MY_OBJECT.FORWARD = c

save MY_OBJECT
```

**REST:**

```json
PUT https://api.opendatadsl.com/api/object/v1
Authorization: Bearer {{token}}

{
  "_id": "MY_OBJECT",
  "PRICE": {
    "_type": "VarTimeSeries",
    "calendar": "BUSINESS",
    "_identity": {
      "molecule": "TS-MOL-01",
      "sap": "GAS-UK-TS-01"
    }
  },
  "FORWARD": {
    "_type": "VarCurve",
    "ondate": "2024-01-15",
    "_identity": {
      "molecule": "FWD-MOL-01",
      "bloomberg": "NBPGBP Comdty"
    }
  }
}
```

Each data item carries its own independent identity map, so downstream systems receiving either item can resolve their own identifier without any additional configuration.

## Further Reading

- [Extracting Data](https://doc.opendatadsl.com/docs/poc/extracting-data) — overview of all data delivery methods including automations
- [Automation REST service](https://doc.opendatadsl.com/docs/api/rest/service/automation) — full automation reference
- [SmartCurve variable reference](https://doc.opendatadsl.com/docs/odsl/variable/smartcurve)
- [TimeSeries variable reference](https://doc.opendatadsl.com/docs/odsl/variable/timeseries)


---

<!-- source: topics/data-management/Data-Migration.md -->

## Introduction
It is sometimes beneficial to copy some data over from production into a test environment in order to test out some changes before putting them into production.

## Examples

### Master Data and Common Data References

If you are using EventCurves, EventTimeseries for data in the common database, this section will help you migrate data between environments.

The following code searches for master data records with an object query and then iterates through each object reading the raw data reference before updating the object to the test environment.

```js
set auto_follow_references false

objects = find ${object} where source='ICE' and exchange='NDEX'

for obj in objects
	print obj.id

	// Iterate through the object getting all the data items
	keys = obj.getProperties().keyNames
	for key in keys
		value = obj[key]
		if typeOf(value) == "Reference" 
			if value.service == "data"
				print value
				// Get the raw value and add it to the object
				value.addOption("_raw", true)
				raw = value.getReferencedObject()
				raw.id = key
				obj[key] = raw
			end
		end
	next

	// Switch environment
	use test

	// Save the object
	save obj

	// Go back to production
	use production
next
```


---

<!-- source: topics/data-management/Data-Modelling.md -->

Data Modelling
==============

This guide explains how to create your own data models, load data into them and utilise them in processes and reports

## Introduction

If you have read the section on [variables](https://doc.opendatadsl.com/docs/odsl/variable/variables), you will see that we have a lot of built-in data types which allow you to model your data exactly the way you want it, especially using the dynamic and flexible [Object](https://doc.opendatadsl.com/docs/odsl/variable/object) variable.

Data Modelling certainly benefits from dynamic flexibility, but it also requires the ability to define certain fixed information in order to categorise, classify or annotate your data sets and make them searchable and browseable.

This section walks through how you can create your own custom data types, create objects from these types, create relationships between the objects. We will then search, browse and use these objects with plenty of code examples.

### Naming conventions

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

## Types And Objects

A type in ODSL is a structure of properties, expressions and methods which are shared amongst any objects that are created from it.

### Root type
Certain properties are present on all types - these properties are defined on the root type known as #Object and are as follows:

```json
{
  "name": "String",
  "description": "String",
  "classification": "String",
  "geolocation": "Geometry" 
}
```

### Property types
As you can see from the root type, each property has a type, properties can be:
* A primary type:
    * String
    * Date
    * Boolean
    * Number
* A special type:
    * Dimension - a special string which can be used for filtering data 
    * Geometry - used to define a point or shape for [geo-spatial](https://doc.opendatadsl.com/docs/odsl/dm/geospatial) mapping
    * Duration - a duration of time
    * list - a list of array of other types
    * object - a generic object 
* A data type:
    * TimeSeries - a list of values mapped against date/times
    * Curve - a forward curve used for contract deliveries and forecasts
* Any other defined type

### Creating types

A type can represent any real or virtual thing, but for our example throughout this guide, I will use things around a business that we might want to make a record of, such as companies, offices, employees, computers and other assets that we would like to track.

So let’s start off with a simple company type:

```js
// Declare a type Company
Company = type
    name as String()
end
```

An explanation of the the above code:

#### Company = type

This starts the creation of a new [declared type](https://doc.opendatadsl.com/docs/odsl/variable/DeclaredType) called Company

#### name = String()

This adds a property called name of type String to the Company type

#### end

The ends the creation of the company type

Let’s add a new address type:

```js
// Declare an address type
Address = type
    street as String()
    area as Dimension()
    city as Dimension()
    county as Dimension()
    country as Dimension()
    postcode as String()
end
```

You can see that we have now created an Address type and added 6 properties of an Address.

4 of the properties we created as type Dimension, this is a special type of string property that indicates that you will want to filter objects of this type by that property.

Defining a property as Dimension informs user interfaces to generate a filtered list of possible options for this property when browsing for data. We would usually not define Dimension properties that are ‘usually unique’.

### Objects (Master Data)

We can create objects from these types using the **object as** creation syntax as follows:

```js
// Create a company
ABC = object as Company
    name = "ABC Ltd"
end

// Create an address
ABCLondon = object as Address
    street = "123 Abacus Street"
    area = "Westminster"
    city = "London"
    county = "Greater London"
    country = "England"
    postcode = "SW1 1AB"
end
```

### Object IDs
The ID of an object or master data is the variable name you give it.
If you want to use a more complex ID, i.e. with dots in it, set the id using the id property, e.g.

```js
obj = object as Company
  name = "My Company"
end

obj.id = "MY.COMPANY"
```

### Inspecting objects

To take a look at what our object looks like, use the **print** statement:

```js
print ABC
```

This displays the following:

```json
{
  "name": "ABC Ltd"
}
```

Similarly for the address:

```js
print ABCLondon
```

Produces the following:

```json
{
  "area": "Westminster",
  "city": "London",
  "country": "England",
  "county": "Greater London",
  "postcode": "SW1 1AB",
  "street": "123 Abacus Street"
}
```

### Adding non-type-defined properties

When creating an object from a type, we can also add any other variables onto the object that have not been defined on the type, e.g. in the following example, we add a name property:

```js
// Create a minimal address with a non-type-defined name property
ABCNewcastle = object as Address
    name = "Newcastle Address"
    city = "Newcastle"
end
```

You can add more complex data types, such as TimeSeries or Curves dynamically too:

```js
// Create a minimal address with a non-type-defined name and a TimeSeries property
ABCNewcastle = object as Address
    name = "Newcastle Address"
    city = "Newcastle"
    employees = TimeSeries("MONTHLY")
end

ABCNewcastle.employees.add("2020-10-01", 25)
```

If you place a breakpoint in the code just after this, then you can see the variable we have created:

We can directly access the properties also and the editor prompts us with the available properties, including the non-type-defined ones:

### Adding matching expressions

You can improve the quality of data entry by adding matching expressions on properties which validate the data entered. The matching expression is a regular [regex](https://en.wikipedia.org/wiki/Regular_expression) expression as can be shown in the creation of a person type below:

```js
Person = type
    // A person
    first as String()
    last as String()
    dob as Date()
    email as String() matches "^(\[a-zA-Z0-9_\\-\\.\]+)@(\[a-zA-Z0-9_\\-\\.\]+)\\.(\[a-zA-Z\]{2,5})$"
end
```

If we now create a person from this type with an invalid email address, you can see in the editor it highlights that the object is in error because the email address given does not match the patter we have defined and the object variable does not get created.

### Adding expressions

Expressions can be added to types to add on-the-fly data calculations and reduce repetition of data entry. For example, we can add a name expression to our person that concatenates the first and last name properties to produce a full name:

```js
Person = type
    // A person
    first as String()
    last as String()
    name as first + " " + last
    dob as Date()
    email as String() matches "^(\[a-zA-Z0-9_\\-\\.\]+)@(\[a-zA-Z0-9_\\-\\.\]+)\\.(\[a-zA-Z\]{2,5})$"
end
```

If we create our JohnDoe object as follows and print out the name:

```js
JohnDoe = object as Person
    first = "John"
    last = "Doe"
    dob = "1985-10-19"
    email = "johndoe@abc.com"
end

print JohnDoe.name
```

It produces the following output:

```json
John Doe
```

We can also add expressions to data types other than String, e.g. add this property to the person type

```js
age as (Date() - dob).years
```

This will calculate how old the person is today in years

### Saving types and objects

To store our work to the database, we need to use the active variable services for [types](https://doc.opendatadsl.com/docs/odsl/service/type) and [objects](https://doc.opendatadsl.com/docs/odsl/service/object). 
This is done using the **save** command:

:::note
You can save types and objects implicitly or explicitly, there is no difference between the 2 methods shown below
:::

```js
// Save the types

// Implicit save - service is determined by the variable type
save Company

// Explicit save using the service name
save ${type:Address}
save ${type:Person}

// Save the objects

// Implicit save - service is determined by the variable type
save ABC
save ABCLondon

// Explicit save using the service name
save ${object:ABCNewcastle}
save ${object:JohnDoe}
```

## Querying Data

We have 2 methods of getting data into our session:

*   Directly assign an object to a variable using the [Object Service](https://doc.opendatadsl.com/docs/odsl/service/object) and the object id    
*   Search for objects using the **find** command on the [Object Service](https://doc.opendatadsl.com/docs/odsl/service/object)
    

### Direct assignment

If we know the id of the object to retrieve, we can directly retrieve the object from the service as follows:

```js
jd = ${object:"JohnDoe"}
```

This uses the object service to retrieve the object with id **JohnDoe** and assign it to an object variable named jd.

### Searching for data

We can use the **find** command on the [Object Service](https://doc.opendatadsl.com/docs/odsl/service/object) to retrieve a list of objects that match the criteria we specify.

```js
people = find ${object} where first="John" and last="Doe"
print people
```

The people variable above is a List of person objects, as shown below:

```json
[
{
  "_id": "JohnDoe",
  "_type": "Person",
  "_links": {},
  "dob": "1985-10-19",
  "email": "johndoe@abc.com",
  "first": "John",
  "last": "Doe"
}
]
```

### Adding a custom data type to a type

In our example of a person type, we could directly add a reference to an address type to re-use the address type structure as follows:

```js
Person = type
    // A person
    first as String()
    last as String()
    name as first + " " + last
    dob as Date()
    age as (Date() - dob).years
    email as String() matches "^(\[a-zA-Z0-9_\\-\\.\]+)@(\[a-zA-Z0-9_\\-\\.\]+)\\.(\[a-zA-Z\]{2,5})$"
    address as Address()
end
```

In the above example, we have added an address property as type Address() which relates to our user defined type ‘Address’.

You can see in the example code below, how we can now access the address property by first creating it as an object as type Address:

```js
JaneDoe = object as Person
    first = "Jane"
    last = "Doe"
    dob = "1987-07-11"
    email = "janedoe@abc.com"
    address = object as Address
        city = "Newcastle"
    end
end
```

We could also assign an already created address to the property, e.g. :

```js
JaneDoe = object as Person
    first = "Jane"
    last = "Doe"
    dob = "1987-07-11"
    email = "janedoe@abc.com"
    address = ABCNewcastle
end
```

NOTE: this will create a copy of the ABCNewcastle object, any changes made to this object from now on will not be reflected in JaneDoe’s address.

In order to create a reference to another object rather than copying the values from an object as we saw previously, we have to reference the active variable reference for this object in a ref function as follows:

```js
JaneDoe = object as Person
    first = "Jane"
    last = "Doe"
    dob = "1987-07-11"
    email = "janedoe@abc.com"
    address = ref(${object:"ABCNewcastle"})
end
```

This is a useful way of creating independent shared information.

## Creating Relationships

Extending upon the ability to create shared information, you can create a link between 2 objects in a 2-way relationship. For example, using JaneDoe in the previous section, if we wanted to know both where she lived and conversely, who lived at that address.

The link syntax is as follows:

```js
link A as type\[/childType\] of B
```

*   The _**A**_ and _**B**_ are objects in [active variable notation](https://doc.opendatadsl.com/docs/odsl/service/object)    
*   The _**type**_ is the relationship name when viewed at the _**A**_ end (or both if childtype is not specified)
*   The optional _**childtype**_ is the relationship name when viewed from the _**B**_ end
    
We can create a link between JaneDoe and ABCNewcastle as follows:

```js
link ${object:"ABCNewcastle"} as address/residents of ${object:"JaneDoe"}
```

This creates a 2-way relationship between the Person object and the Address object as shown below:


---

<!-- source: topics/data-management/Data-Monitoring.md -->

This guide gives detailed information about best-practices for utilising the data monitoring capabilities of the OpenDataDSL platform.

## Monitoring Terminology

Monitoring of the loading of data is organised using the following entities:

### Smart Loader
A smart loader runs the defined process until all datasets are complete according to the completeness criteria.
It uses a retryStrategy to determine how often to run the process. 

### Process
A process defines the workflow or script that is run to extract, transform and load data into the platform.

### Dataset feed
A dataset feed is a configuration which defines the time window that you expect to receive all the data for a specific feed.

The identifier for a dataset feed is of the following format:

```{provider id}.{feed id}```

e.g. **ICE.IFEU**

### Dataset
A dataset is an individual product within a dataset feed, this defines the quantity of data we expect to receive on a daily basis.

The identifier for a dataset feed is of the following format:

```{provider id}.{feed id}.{product id}```

e.g. **ICE.IFEU.B**

### Dataset delivery
A dataset delivery is a record of all information regarding the process of getting the data for a dataset for a single day.
It also calculates a score which identifies how well the process worked for each day. 

The identifier for a dataset delivery is of the following format:

```{provider id}.{feed id}.{product id}:{ondate}```

e.g. **ICE.IFEU.B:2024-06-12**

### Completeness Checks
These are advanced checks that are run on the raw data to make more targeted checks to ensure completeness.

### Quality Checks
Quality checks are used to check the data to ensure it conforms to your expectations of values for this dataset.

A check consists of:
* name - The free-form name you want to give to this check, e.g. ```Check for zero values```
* script - The name of the script to use for the expression - can be empty.
* expression - The expression to use to run the check. This is usually the function call, e.g. ```zeroCheck(['SETTLEMENT_PRICE'])```

You can also define a group of checks and add this onto the dataset.

### Scoring
Every day that dataset loading occurs, a score is calculated which is a measurement used to indicate how well the process worked that day.

The scores are as follows:
* 4 - Data was loaded on time
* 3 - Data was maximum of 1 hour late
* 2 - Data was between 1 hour and 4 hours late
* 1 - Data was more than 4 hours late
* 0 - No data was expected, e.g. a holiday

## Monitoring Lifecycle

The dataset delivery record represents the lifecycle for the dataset loading for a single day.

### Initialisation

Each day begins with an initialisation of the dataset delivery record.
This sets the following default values:
* status - waiting
* score - 4 (0 if it is a holiday) ( any lateness will start reducing that number)
* complete - false
* initialised - the timestamp of when the intialisation was run

### Data loaded
When data is loaded that includes a _dsid property, the following occurs:

* Check for corrections - if the data for this data was previously loaded, it is checked to see if any of the values have changed.
* Calculate metrics
    * Determine the quantity of tenors by tenor type loaded during this update
    * Determine if all the expected tenors have been loaded - completeness
    * Run any advanced completeness checks configured on the dataset to finalise completeness
* Update the timeline and delivery information in the dataset delivery
* Update the status to either partial (if not late) or loaded

### Quality checks
Any quality checks that have been configured on the dataset are triggered after data has been loaded.
The quality checks are always performed on the entire dataset, not just the most recently loaded.

### Check for lateness
Periodically, a process runs that determines if any datasets are late according to the dataset feed time window.
If a dataset is determined to be late (or missing), the dataset delivery score is updated and the dataset **late** (or **missing**) event is raised, which you can act on with an [automation](https://doc.opendatadsl.com/docs/topics/automations/creating).
The lateness check also checks the ```missing``` time if it is configured, if the current time is beyond the missing time and it is a smart loader, the smart loader is halted and manual intervention is required.

## Using the Dataset Monitoring API
This section guides you through using the dataset monitoring features.

### Datasets and Dataset Feeds
Datasets and dataset feeds are 'managed' in 3 different locations:
* private - these are configured and managed by your company
* common - these are configured and managed by us, you will only see the common datasets you have access to
* public - these are configured and managed by us, you have access to all of these

#### Managing private datasets and dataset feeds
With private datasets, you can configure everything about the dataset and dataset feed, specifically:
* **Dataset ID**
  > As mentioned above, the dataset id comprises 3 sections: provider, feed and product. 
  > For private datasets, it is recommended to set the provider to a short-form version of your company name, e.g. for OpenDataDSL we use ODSL
* **Expected tenors**
  > You can either manually enter the list of expected tenors or if you have already loaded some data, you can get the system to calculate the minimum actual loaded tenors
* **Expected checks**
  > You can optionally add checks to determine if the dataset is complete as an addition to the simple minimum expected tenor check
* **Calendar**
  > This the the calendar that defines the days that you expect this data to be available, any non-calendar days are marked as holidays
* **Timings**
  > You can specify the time window when you expect the data to be ready to collect or be loaded into the system
* **Checks**
  > You can specify the quality checks you want to perform on the data loaded to this dataset

##### Examples

**OpenDataDSL:**

An example of creating a private dataset feed

```js
dsf = object as DatasetFeed
	dsid = "ODSL.TRADER2"
	name = "ODSL Trader2"
	calendar = "BUSINESS"
	timezone = "Europe/London"
	time = "19:00 EU1"
	late = "21:00 EU1"
end

save dsf
```

An example of creating a private dataset

```js
ds = object as Dataset
	dsid = "ODSL.TRADER2.NBP"
	name = "Trader2 NBP Prices"
	expected = SimpleObject()
end
ds.expected.set("*",12)
ds.expected.set("Month", 12)

save ds
```

An example of creating a private dataset with expected checks
```js
ds = object as Dataset
  dsid = "ODSL.DS.TEST"
  source = "private"
  name = "ODSL test dataset"
end

// Add simple expected checks
ds.expected = SimpleObject()
ds.expected.set("*", 2)
ds.expected.set("CalendarDay", 2)
ds.expected.set("Month", 2)

// Add advanced checks
cc1 = SimpleObject()
cc1.type = "function"
cc1.script = "#DatasetCompletenessChecks"
cc1.name = "Check for M01"
cc1.expression = "hasTenor('M01')"

ds.expectedChecks = [cc1]

// Add quality checks
check1 = SimpleObject()
check1.type = "function"
check1.script = "#QualityCheckDatasets"
check1.name = "Check for zero values"
check1.expression = "zeroCheck('SETTLEMENT_PRICE')"

// Add a quality check group
check2 = SimpleObject()
check2.type = "group"
check2.name = "My Dataset Quality Checks"
check2.group = "MY.QUALITY"

ds.checks = [check1, check2]

save ds
```

**REST API:**

An example of creating a private dataset feed

```js
POST {{url}}/dataset/v1/feed
Authorization: Bearer {{token}}

{
  "_id": "ODSL.TRADER1",
  "calendar": "BUSINESS",
  "time": "16:00 EU1",
  "late": "19:00 EU1"
}
```

An example of creating a private dataset

```js
POST {{url}}/dataset/v1/info
Authorization: Bearer {{token}}

{
  "_id": "ODSL.TRADER1.NBP",
  "expected": {
    "*": 12,
    "Month": 12
  }
}
```

An example of creating a private dataset with expected checks

```js
POST {{url}}/dataset/v1/info
Authorization: Bearer {{token}}

{
  "_id": "ODSL.TRADER1.NBP",
  "expected": {
    "*": 12,
    "Month": 12
  },
  "expectedChecks": [
    {
      "type": "function",
      "script":"scripts-ds-quality",
      "name":"has tenor M00",
      "expression": "hasTenor('M00')"
    }
  ],
  "checks": [
    {
      "type":"function",
      "script":"#QualityCheckDatasets",
      "name":"Check for zero values",
      "expression":"zeroCheck('SETTLEMENT_PRICE')"
    },
    {
      "type":"group",
      "name":"My Quality Group",
      "group":"MY.GROUP"
    }
  ]
}
```

#### Managing public and common datasets and dataset feeds
If you want to add public and common datasets into your monitoring, you can create references to them in your private database. 

With public and common datasets, you can only configure the following:
* **Checks**
  > You can specify the quality checks you want to perform on the data loaded to this dataset
* **Category**
  > You can specify a category that you want to group datasets into
* **Priority**
  > You can specify a priority for this dataset from highest, high, medium, low, lowest to group and filter by

##### Examples

**OpenDataDSL:**

Getting a list of all the available datasets

```js
print ${dataset:"allds"}
```

Creating a reference to a common dataset

```js
ds = object as Dataset
	dsid = "ICE.NDEX.NLB"
	source = "common"
end
save ds
```

Removing a reference to a common dataset

```js
delete ${dataset:"info/ICE.NDEX.NLB"}
```

**REST API:**

Getting a list of all the available datasets, projecting the id, name and source

```js
GET {{url}}/dataset/v1/allds
  ?_project=name,source
Authorization: Bearer {{token}}
```

Creating a reference to a common dataset with our own custom quality checks

```js
POST {{url}}/dataset/v1/info
Authorization: Bearer {{token}}

{
  "_id": "ICE.NDEX.NLB",
  "source":"common",
  "checks": [
    {
      "type": "function",
      "name": "Check for negative values",
      "script": "#QualityCheckDatasets",
      "expression": "negativeCheck('SETTLEMENT_PRICE')"
    }
  ],
  "category": "OIL",
  "priority": "low"
  }
}
}
```

Removing a reference to a common dataset

```js
DELETE {{url}}/dataset/v1/info/ICE.NDEX.NLB
Authorization: Bearer {{token}}
```

### Dataset Deliveries
You can retrieve the delivery information for any datasets that you are monitoring.
Any specific quality check results that you have added will also be shown in the dataset delivery information.

##### Examples

**OpenDataDSL:**

```js
// Get all dataset deliveries for the current ondate
find ${dataset:"delivery"}

// Get all dataset deliveries for a specific ondate
find ${dataset:"delivery/2024-06-25"}
```

**REST API:**

Get all dataset deliveries for the current ondate

```js
GET {{url}}/dataset/v1/delivery
Authorization: Bearer {{token}}
```

Get all dataset deliveries for a specific ondate

```js
GET {{url}}/dataset/v1/delivery/2024-06-25
Authorization: Bearer {{token}}
```

### Dataset Quality Check Groups
You can create quality check groups containing an array of checks, this group can then be added onto a dataset.
Any specific quality check results that you have added will also be shown in the dataset delivery information.

##### Examples

**OpenDataDSL:**

```js
// Create a quality group
g = QualityGroup()
g.type = "quality"
g.category = "dataset"
g.name = "ICE.NDEX.QUALITY"
g.description = "ICE Endex Dataset Quality Checks"
g.script = "#QualityCheckDatasets"
g.addCheck("Check for zero values", "zeroCheck(['SETTLEMENT_PRICE'])")
save g
```

**REST API:**

Create a quality group

```js
POST {{url}}/group/v1
Authorization: Bearer {{token}}

{
  "type": "quality",
  "category": "dataset",
  "name": "ICE.NDEX.QUALITY",
  "description": "ICE Endex Dataset Quality Checks",
  "script": "#QualityCheckDatasets",
  "checks": [
    {
    "name": "Check for zero values",
    "expression": "zeroCheck(['SETTLEMENT_PRICE'])"
    }
  ]
}
```

Get a group

```js
GET {{url}}/group/v1/quality/ICE.NDEX.QUALITY
Authorization: Bearer {{token}}
```

Rollback changes to a quality group

```js
DELETE {{url}}/group/v1/quality/ICE.NDEX.QUALITY
Authorization: Bearer {{token}}
```

## Dataset Quality Scripts
The ODSL scripts you create contain functions to perform quality checks on datasets.

The functions have access to the following variables:

* \#DSID - The string dataset id
* \#ONDATE - The date for the dataset update
* \#LOG - A log object to place all the failure information
* \#EVENTS - A list of all the events for this dataset update

All scripts to be used for dataset quality checks need to have a category ```dataset-quality```, you can do this by adding the following to the top of your script:

```js
/**
 * @category dataset-quality
 */
```

### Example Functions

#### Check to see if any specific properties are zero

Example expression: ```zeroCheck(['value'])```

```js
function zeroCheck(properties)
	zeroCheck = "valid"
	#LOG.failures = []
	print "Checking " + #EVENTS.size() + " events for dataset: " + #DSID + " for " + #ONDATE
    for pc in #EVENTS
		for p in properties
			// Test for zero value
			v = variable(pc, p)
			tenor = variable(pc, "absolute")
			if v.isZero()
				zeroCheck = "failed"
				#LOG.failures.add("Zero value for " + tenor)
			end
		next
    next
end
```

## Dataset Completeness Check Scripts
The ODSL scripts you create contain functions to perform expected checks on datasets.

The functions have access to the following variables:

* \#DSID - The string dataset id
* \#ONDATE - The date for the dataset update
* \#LOG - A log object to place all the failure information
* \#EVENTS - A list of all the events for this dataset update

All scripts to be used for dataset completeness checks need to have a category ```dataset-completeness```, you can do this by adding the following to the top of your script:

```js
/**
 * @category dataset-completeness
 */
```

### Example Functions

#### Check to ensure a specific named tenor is present

Example expression: ```hasTenor('M00')```

```js
function hasTenor(tenor)
	hasTenor = "failed"
	#LOG.failures = ["Missing " + tenor + " tenor"]
	for ev in #EVENTS
		if ev.relative == tenor
			hasTenor = "valid"
			#LOG.failures = []
			log info tenor + " found"
		end
	next
end
```

#### Check for number of daily tenors dependent on the day of the week

This check determines the number of expected tenors based on the day of the week.
If the day matches the startday, the **max** tenors are expected, and it decreases as the week progresses, e.g.

Using the expression: ```rollingOffDayTenors('MONDAY', 14, 'BUSINESS')```

|Day of week|Expected Tenors|
|-|-|
|Monday|14|
|Tuesday|13|
|Wednesday|12|
|Thursday|11|
|Friday|10|

```js
function rollingOffDayTenors(startday, max, calendar)
	rollingOffDayTenors = "valid"
	test = Date(#ONDATE)
	dow = test.dow
	log info "Day of week = " + dow
	cal = Calendar(calendar)
	expected = max
	while dow != startday
		test = cal.previous(test)
		dow = test.dow		
		expected = expected - 1
	end
	log info "Expected number of daily tenors = " + expected
	actual = 0
	for ev in #EVENTS
		if startsWith(ev.relative,"D")
			actual = actual + 1
		end
	next
	if actual < expected
		log fatal "Actual number of daily tenors = " + actual
		#LOG.failures = ["Expecting " + expected + " daily tenors, got " + actual]
		rollingOffDayTenors = "failed"
	end
end
```


---

<!-- source: topics/data-management/Data-Versioning.md -->

Data Versioning
===============

This guide takes you through how we track changes to all elements in OpenDataDSL and how you can take advantage of multiple versions and follow an audit trail of every change made.

## Introduction

One of the most important aspects of data management is tracking changes that have been made to every aspect of the system. Some of the ways you can utilise this are as follows:

### Tracing issues

When things stop working or start producing incorrect results, you need to be able to trace through everything that has changed in order to resolve the issue.

### Utilise tagging to ensure stability

If you [tag](#tagging-a-version) versions with names, e.g. PROD, you can implement a phased approach of releasing new versions of elements into the system.
There is a dynamic tag (~LATEST) which always points to the last or current version.

## How it works

### Types of versioning

There are 2 types of versioning: automatic and user-defined.

#### Automatic versioning

Automatic versioning is always-on which means that any change to the element results in a new version. The following elements have automatic versioning:

*   Actions    
*   Calendars
*   Extractors
*   Report configurations
*   Reports
*   Scripts
*   Transformers
*   Types
*   Workflows
    

#### User defined

For objects, you have the choice, per type, of whether to use versioning or not. When you create a type, you can specify that all objects of that type are versioned by using the versioned modifier:

```js
Widget = versioned type
    name as String()
    price as Number()
end
```

If you save an object using the above Widget type, the system checks for an existing object of the same id and if it is different, it saves it as a new version.

### What is recorded

Along with the actual data, a version property is added with the following attributes:

|**Name**|**Description**|**Type**|
|-|-|-|
|version|The version number which is an incremental number, starting at 1|Integer|
|oid|The id of the object|String|
|timestamp|The date/time that the update was made|Date|
|user|The email address of the user who made the change or the tenantid if it was updated via a process|String|
|[tag](#tagging-a-version)|A list of tags that you can add to a version|List|

## Version Usage

The versioning of elements is automatic, so any update that you make that changes an element will automatically perform the versioning.

### Viewing version information

You can access the version information of an element using the version property of that element, e.g.

```js
act = ${action:"test_send_batch"}
print act.version
```

Displays:

```json
{
  "oid": "test_send_batch",
  "tag": [],
  "timestamp": {
    "_id": "timestamp",
    "value": "2020-11-06T09:23:50.568",
    "_type": "VarDate"
  },
  "user": "user@mymail.com",
  "version": "3"
}
```

### Listing versions

You can get a list of the versions of an element by adding a '*' version when retrieving an element, e.g.

```js
versions = ${action:"test_send_batch":*}
print versions
```

Displays:

```json
[
{
  "id": "test_send_batch",
  "tags": [],
  "timestamp": "2020-11-06T09:23:50.568Z",
  "type": "action",
  "user": "user@mymail.com",
  "version": "3"
},
{
  "id": "5fa51626d94c4a000711dbf3",
  "tags": [],
  "timestamp": "2020-11-03T12:23:37.799Z",
  "type": "action",
  "user": "user@mymail.com",
  "version": "2"
},
{
  "id": "5fa14bc9468199000752bbf9",
  "tags": [
    "PROD"
   ],
  "timestamp": "2020-11-03T12:14:42.995Z",
  "type": "action",
  "user": "user@mymail.com",
  "version": "1"
}
]
```

### Retrieving a specific version by number

You can retrieve a specific numbered version using the version modifier as follows:

```js
act = ${action:"test_send_batch":1}
print act.version
```

Dsiplays:

```json
{
  "oid": "test_send_batch",
  "tag": [
    "PROD"
  ],
  "timestamp": {
    "_id": "timestamp",
    "value": "2020-11-03T12:14:42.995",
    "_type": "VarDate"
  },
  "user": "user@mymail.com",
  "version": "1"
}
```

### Tagging a version

You can tag a version of an element with a name which you can use instead of a version number.

:::note
The tag name is unique across all versions of an element, so if you tag another element with the same name, the tag is moved to that element
:::

```js
// Tag version 1 as PROD
tag ${action:"test_send_batch":1} as PROD

// Retrieve the PROD version of the action
prodaction = ${action:"test_send_batch":PROD}
```

### UnTagging a version

To untag or remove a tag from an element, use the delete command as follows:

```js
// UnTag the PROD version
delete ${action:"test_send_batch":PROD}
```

:::note
This does not delete the version, only the tag
:::

### Deleting a version

There are several behaviours of the delete command depending on how you specify the version information and how many versions of the element there are.

If I have 3 versions of an action and I have tagged version 1 with the name PROD as shown in the tagging a version section, I will have the following:

```js
// Request latest version (version 3)
act = ${action:"test_send_batch"}

// Request tagged version (version 1)
act = ${action:"test_send_batch":PROD}
```

#### Default delete

If I issue a delete command without a version specifier, version 3 is deleted and version 2 now becomes the current version, effectively performing a rollback.

```js
// Default delete performs a rollback
delete ${action:"test_send_batch"}
```

#### Deleting a numbered version

If I issue a delete command with a version, that version is removed from the version stack. The current version remains the same, unless I deleted version 3, in which case it would be the same as the default delete.

```js
// Delete version 2
delete ${action:"test_send_batch":2}
```

#### Deleting a named version

If I issue a delete command with a named version or tag, only the tag is removed and the version of the element remains.

```js
// Delete a tag from a version
delete ${action:"test_send_batch":PROD}
```

#### Deleting all versions

If I want to completely remove the element and all its versions, I can specify '*' as the version number as follows:

```js
// Completely remove the element
delete ${action:"test_send_batch":*}
```


---

<!-- source: topics/data-management/Message-Queues.md -->

A message queue is used as the glue to connect services together including your own on-premise services and applications

## Introduction

Message Queues allow you to send data from a process to another remote service or application. They act as a vehicle to transport data from one place to another.

### About Queues

By default, you have 1 queue which is not being handled by anything, so if you send data to that queue, it will sit there for the default retention time (4 days) and then be removed. You can create other named queues up to your quota maximum, each one can have it’s own handler for consuming the data.

### About Handlers

Handlers consume the data placed on a queue and can be external (Outside of our cloud and managed externally by yourself or a third party) or internal (Inside our cloud and attached to a process).

#### External Handlers

Check back here for more details soon.

#### Internal Handlers

The currently supported internal handlers are shown in the table below:

|**Name**|**Description**|**Data Type Handled**|
|-|-|-|
|loader|A data loader to load data to the object service|Batch of objects|

## Managing Queues

### Creating and updating queues

To create or update a new queue in ODSL code, use the following syntax:

```js
q = Queue("myqueue")
q.description = "My Queue"
print q
save q
```

### Queue properties

The properties of a queue are decribed below:

#### queue

This is the name of the queue, it can only contain alphanumeric characters and ‘-' and '_’.

#### handler

This is the name of the internal handler or empty if it is being handled externally

#### timeout

When a consumer receives and processes a message from a queue, the message remains in the queue. OpenDataDSL doesn't automatically delete the message. Because OpenDataDSL is a distributed system, there's no guarantee that the consumer actually receives the message (for example, due to a connectivity issue, or due to an issue in the consumer application). Thus, the consumer must delete the message from the queue after receiving and processing it.

![
	            Visibility Timeout
	        ](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/images/sqs-visibility-timeout-diagram.png)

Immediately after a message is received, it remains in the queue. To prevent other consumers from processing the message again, OpenDataDSL sets a _visibility timeout_, a period of time during which OpenDataDSL prevents other consumers from receiving and processing the message. The default visibility timeout for a message is 3 minutes. The minimum is 3 minutes. The maximum is 12 hours.

The value is set as a number and either ‘s' for seconds, ‘m’ for minutes or 'h’ for hours.

#### retention

This is the length of time that OpenDataDSL retains the message before it is automatically removed. The default is 4 days, The minimum is 1 minute. The maximum is 14 days.

The value is set as a number and either ‘s' for seconds, ‘m’ for minutes, ‘h’ for hours or 'd’ for days

### Deleting Queues

If you no longer need a queue, you can delete it using the ODSL **delete** command as follows:

```js
delete ${queue:"myqueue"}
```

### Listing your queues

To get a list of your currently configured queues, use the ODSL **find** command:

```js
queues = find ${queue}
print queues
```

### Getting a specific queue

You can get a specific queue by assigning the queue active variable to a variable, e.g.

```js
// Get the default queue
def = ${queue:""}
print def

// Get a named queue
myqueue = ${queue:"myqueue"}
print myqueue
```

### Using Queues

In OpenDataDSL you send data to a queue using the **send** command using the following syntax:

```js
send var [to queue]
```

If you don’t specify the queue name, the default queue will be used

For example, to send a batch of data to the internal loader:

```js
// Create a batch
batch=PROCESS.createBatch()

// Add some objects
obj1 = Object()
obj1.name = "Hello"
batch.add(obj1)
obj2 = Object()
obj2.name = "World"
batch.add(obj2)

// Send to the default queue
send batch
```

### Using Queues Externally

You can connect to a queue using the Microsoft Azure Service Bus SDK.
Whenever you create a queue, a shared access key is created for that queue which you can use to connect to the queue.

:::info
If you use the [Java](https://doc.opendatadsl.com/docs/sdk/java) or [Matlab](https://doc.opendatadsl.com/docs/sdk/matlab) SDK's, these have helper methods to help you work with message queues 
:::

To find out the connection string to use to connect to the queue, retrieve the queue and use the property ```connection```

For example, to get details for a queue named **export**
```
GET https://api.opendatadsl.com/api/queue/v1/export
Authorization: Bearer {{token}}
```

Example output:
```json
{
  "_id": "prod/<<redacted>>/export",
  "tid": "<<redacted>>",
  "tenant": "<<redacted>>",
  "queue": "export",
  "description": "Export Queue",
  "retention": "PT480H",
  "timeout": "PT5M",
  "connection": "<<redacted>>>>",
  "messageCount": 2,
  "sizeInBytes": 1598,
  "activeMessageCount": 2,
  "deadLetterMessageCount": 0,
  "scheduledMessageCount": 0
}
```


---

<!-- source: topics/data-management/Workflows-and-Processes.md -->

This guide gives detailed instructions on creating and using actions, workflows, scripts and processes

## Introduction

This guide is concerned with automation of all your data tasks: collection, transformation, derivation and loading. You can create scripts or workflows which can be scheduled of triggered by an event in order to automate a specific process.

In here, we describe how to:

*   Create and use custom actions in workflows    
*   Define workflows
*   Create processes using workflows or scripts
    
## Actions

An action is a specific technical task written in the [OpenDataDSL language](https://doc.opendatadsl.com/docs/odsl) which is used as a component in a workflow. There is a public library of actions readily available to use which you can see examine, change and save to your private action library to create your own custom version.

### Creating an action

To create a new action, you need to use the action syntax in ODSL as follows:

```js
name = action in category
  // Optional comment used as the description
  (in varname as type (description)? (OPTIONAL)?)*
  (out varname as type (description)?)*
  (exit name (, name)*)?
  
  code...
  
end
```

### Action definition

The first line of an action defines the name and category of the action and an option comment can be added as the next line which will be used as the description of the action.

### Input variables

You can define all the input variables that your action requires in order to work, you can also add optional variables that could be used. The variables are properties of an Object called input, so for example if you have defined an input variable called ‘url’ you would access that in your code as 'input.url'.

### Output variables

You can define the output variables that your action produces, these variables can then be used as input variables to other actions. You set the output data on an object called output, so for example if you have defined an output variable called ‘data’, you would set data on the output variable, e.g. output.data = value

### Exit transitions

You can define all of the exit paths that your action supports

### Example action

The following action extracts XML data from a HTTP url:

```js
xml_url = action in "extractors"
    // Extract XML from a HTTP URL
    in url as Scalar "The URL to read the XML from"
    out xml as Object "The XML file converted to an ODSL Object"
    exit "ok", "failed"
    assertType(input.url, "Scalar")
    on error ignore
    output.xml=${xml:input.url}
    if error
        print error
        return "failed"
    else
        return "ok"
    end
end
```

In the above example action, we have (by line number):

1.  Defined our action called xml_url in the category “extractors”    
2.  Set a description of Extract XML from a HTTP URL    
3.  Defined an input variable called url which is used as the HTTP url to extract the XML from    
4.  Defined an output variable called xml which we will set out output xml data to    
5.  Defined 2 exit transitions: ok and failed    
6.  Check that the url input variable is a Scalar    
7.  Stop errors from being thrown by setting the 'on error ignore' statement    
8.  Attempt to get the xml from the url using the [XML active variable service](https://doc.opendatadsl.com/docs/odsl/service/xml) and place the result on the output xml variable    
9.  Check to see if there was an error    
10.  Print out the error - this outputs to the process log when running as a process    
11.  Exit from the action via the failed transition    
12.  else    
13.  Exit from the action via the ok transition
    

### Saving your action

To save your [action](https://doc.opendatadsl.com/docs/odsl/variable/action), you simply need to use the save statement with the [action active variable service](https://doc.opendatadsl.com/docs/odsl/service/action) as shown below:

```js
save ${action:xml_url}
```

## Workflow

A workflow is configuration of actions, data and routing which performs an automated flow to process some input data through a set of coordinated steps. You can utilise actions from the public library or your own custom private library in a workflow.

### Creating a workflow

To create a workflow in code, you use the workflow syntax as shown:

```js
name = workflow in category
  // Optional comment used as the description
  (in varname as type (description)? (OPTIONAL)?)*
  (out varname as type (description)?)*
  (exit name (, name)*)?
  
  WF_START
    "name" -> startaction
  end
  
  (phase)*
  
  (WF_EVENT name as "success"
        return "ok"
  end)*
end
```

The first section of the workflow definition is exactly the same as an action. This is because a workflow can also be used as an action in another workflow and therefore it behaves the same as an action.

### WF_START

The WF_START token is mandatory and defines the first action that is called. There can only be one defined transition in the start token, the name of the transition is ignored, but the destination must match a named action in the workflow.

### PHASE

A workflow consists of 1 or more phases which are used to break a workflow down into logical stages. A phase represents the smallest level of metric reporting in the workflow, so in the workflow dashboard, you can see the individual phases, the timing information and any log messages produced in that phase.

A phase can also be used to handle errors and retries, the syntax for a phase is as follows:

```js
phase "name" (retries r)? (delay d TIMEUNIT)? (then reschedule r TIMEUNIT)? (external)?
  workflowBody
end

TIMEUNIT: (seconds|minutes|hours)
```

An explanation of the settings in the phase command:

*   Retries - this is the number of retries that will be made if an error is thrown by any action    
*   Delay - this is the amount of time to delay retrying    
*   Then reschedule - if you run out of tries, you can reschedule the entire workflow some time later    
*   External - Indicates that this phase is executed externally, i.e. data is sent to a queue and is loaded remotely
    

Within the phase sections, we configure which actions are performed using the following syntax:

```js
workflowBody: (workflowEvent|workflowAction|workflowGateway|workflowWorkflow)

workflowEvent:
WF_EVENT name as type
  (assign|comment|return|transition)*
end

workflowAction:
WF_ACTION name input
  (assign|comment|transition)*
end

workflowGateway:
WF_GATEWAY name input
  (assign|comment|transition)*
end

workflowWorkflow:
WF_WORKFLOW name input
  (assign|comment|transition)*
end

transition: "name" -> connection
```

An explanation of each of the elements that can be added into a phase:

#### workflowEvent

A workflow event represents a change of state that the workflow moves to. This is usually the end of the workflow, but future development will add custom states that you will be able to define.

An example end state event:
```js
    WF_EVENT stopok as "success"
        return "ok"
    end
```
#### workflowAction

This is the configuration of an action in a workflow and generally prepares the action with the input data and routes the exit transitions to the next connection.

An example action configuration:

```js
WF_ACTION getxml ai
  "ok" -> transformxml
  "failed" -> stopfailed
  ai.url = input.url
  result = ${action:"xml_url"}.run(ai, output)    
end
```

#### workflowGateway

This is the configuration of a gateway junction which routes the workflow based on an input expression

#### workflowWorkflow

This is the same as a workflowAction except that a sub-workflow is run in place of an action.

### Saving the workflow

To save your [workflow](https://doc.opendatadsl.com/docs/odsl/variable/workflow), use the save command with the [workflow active variable service](https://doc.opendatadsl.com/docs/odsl/service/workflow) as shown:

```js
save ${workflow:my_workflow}
```

## Process

A process is a scheduled or event driven workflow or script which runs in the cloud.

### Creating a process

You can create a process using the Process() constructor and setting the properties on it. The valid properties are as follows:

|**Name**|**Description**|**Type**|
|-|-|-|
|service|The type of process this is - it is used to categorise your processes|String|
|name|The name of the process|String|
|description|A description for the process|String|
|enabled|A flag showing if this process is currently enabled for running|Boolean|
|processType|The type of process, either Ristretto or Lungo|String|
|workflow|The name of the workflow to run|String|
|scriptName|The name of the script to run, will be ignored if workflow is set|String|
|settings|Any configuration settings to be used|Settings|
|input|The input object to pass into the workflow|Object|
|event|The event to check for triggering the workflow or script|Object|
|cron|A valid cron expression used for time triggering this process|String|

#### Process Type

The processType property on a process can be set to Ristretto or Lungo, the difference between the 2 is as follows:

*   Ristretto - this is used for small, short running processes (less than 6 minutes) - it is a lower cost to run    
*   Lungo - this is used for longer running processes with higher resource demands, the runtime of Lungo processes is not restricted and the cost of running is higher than Ristretto
    

#### Cron Expression

This is a valid cron expression in the format “minute hour day-of-month month day-of-week year”. So, to schedule a process to run every weekday at 5pm, use the following:

```
process.cron = "0 17 ? * MON,TUE,WED,THU,FRI *"
```

### Saving your process

To save your [process](https://doc.opendatadsl.com/docs/odsl/variable/process), use the save command with the [process active variable service](https://doc.opendatadsl.com/docs/odsl/service/process) as show below:

```js
save ${process:my_process}
```

:::note
A process assumes the rights (both application and data) of the user creating the process at the point of the process creation. If more rights are added to the user afterwards, the process would need to be re-updated in order to assume those new rights.
:::

### Example process configuration

Here are a few examples of creating a process:

### Scheduling a workflow

```js
process = Process()
process.service = "ETL"
process.name = "ECB_FX_WORKFLOW"
process.description = "ETL Process for ECB_FX"
process.enabled = true
process.workflow = "#xml_data_loader"
process.input.url = "https://www.ecb.europa.eu/stats/eurofxref/eurofxref-daily.xml"
process.input.tx = "ECB_FX"
process.cron = "0 17 ? * MON,TUE,WED,THU,FRI *"
save ${process:process}
```
