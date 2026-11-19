# Report configurations and report services

The Report variable (Default, Fixed, Dynamic, Insight and Widget reports), scheduling, and the report and reportconfig services.

Generated from the OpenDataDSL documentation (https://doc.opendatadsl.com/docs).


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

<!-- source: product/developer/Active Variable Services/Report-Service.md -->

# Report Service

The **report** active variable service gives your scripts access to stored reports, by report configuration and date. It uses the [Report REST service](https://doc.opendatadsl.com/docs/api/rest/service/report), so the same rules and permissions apply.

Items whose id starts with # are read from public, any other id from private.

## Using the service

```js
// Read an item
item = ${report:"METRICS:2026-10-23"}

// Find items
items = find ${report}
publicItems = find ${report:public}

// Save an item
save ${report:item}

// Tag the current version
tag ${report:"METRICS:2026-10-23"} as PROD

// Delete an item, or roll back to its previous version
delete ${report:"METRICS:2026-10-23"}
```

See [Report](https://doc.opendatadsl.com/docs/odsl/variable/report) for the variable this service holds.


---

<!-- source: product/developer/Active Variable Services/Report-Configuration-Service.md -->

# Report Configuration Service

The **reportconfig** active variable service gives your scripts access to your report configurations. It uses the [Report Configuration REST service](https://doc.opendatadsl.com/docs/api/rest/service/reportconfig), so the same rules and permissions apply.

Items whose id starts with # are read from public, any other id from private.

## Using the service

```js
// Read an item
item = ${reportconfig:"METRICS"}

// Find items
items = find ${reportconfig}
publicItems = find ${reportconfig:public}

// Save an item
save ${reportconfig:item}

// Tag the current version
tag ${reportconfig:"METRICS"} as PROD

// Delete an item, or roll back to its previous version
delete ${reportconfig:"METRICS"}
```

See [Report Configuration](https://doc.opendatadsl.com/docs/odsl/variable/report) for the variable this service holds.


---

<!-- source: product/developer/Active Variable Services/Report-Creator-Service.md -->

# Report Creator Service

The **reportcreator** active variable service gives your scripts access to report configurations with parameters. It uses the [Report Creator REST service](https://doc.opendatadsl.com/docs/api/rest/service/reportcreator), so the same rules and permissions apply.

Items whose id starts with # are read from public, any other id from private.

## Using the service

```js
// Read an item
item = ${reportcreator:"PRICE_SUMMARY"}

// Find items
items = find ${reportcreator}
publicItems = find ${reportcreator:public}

// Save an item
save ${reportcreator:item}

// Tag the current version
tag ${reportcreator:"PRICE_SUMMARY"} as PROD

// Delete an item, or roll back to its previous version
delete ${reportcreator:"PRICE_SUMMARY"}
```

See [Report Creator](https://doc.opendatadsl.com/docs/odsl/variable/reportcreator) for the variable this service holds.
