# Active variable services

The ${service:...} syntax and every service a script can read from, search, save to or delete from.

Generated from the OpenDataDSL documentation (https://doc.opendatadsl.com/docs).


---

<!-- source: product/developer/Active Variable Services/AI-Agent-Service.md -->

# AI Agent Service

The **aiagent** active variable service gives your scripts access to your Fusion AI agents. It uses the [AI Agent REST service](https://doc.opendatadsl.com/docs/api/rest/service/aiagent), so the same rules and permissions apply.

## Using the service

```js
// Read an item
item = ${aiagent:"morning-gas-check"}

// Find items
items = find ${aiagent}

// Save an item
save ${aiagent:item}

// Delete an item, or roll back to its previous version
delete ${aiagent:"morning-gas-check"}
```

See [AI Agent](https://doc.opendatadsl.com/docs/odsl/variable/ai) for the variable this service holds.


---

<!-- source: product/developer/Active Variable Services/AI-Assistant-Service.md -->

# AI Assistant Service

The **aiassistant** active variable service gives your scripts access to the Fusion AI assistants, built-in and custom. It uses the [AI Assistant REST service](https://doc.opendatadsl.com/docs/api/rest/service/aiassistant), so the same rules and permissions apply.

Items whose id starts with # are read from public, any other id from private.

## Using the service

```js
// Read an item
item = ${aiassistant:"EPR"}

// Find items
items = find ${aiassistant}
publicItems = find ${aiassistant:public}

// Save an item
save ${aiassistant:item}

// Delete an item, or roll back to its previous version
delete ${aiassistant:"EPR"}
```

See [AI Assistant](https://doc.opendatadsl.com/docs/odsl/variable/ai) for the variable this service holds.


---

<!-- source: product/developer/Active Variable Services/AI-Playbook-Service.md -->

# AI Playbook Service

The **aiplaybook** active variable service gives your scripts access to Fusion AI playbooks, your own and the library. It uses the [AI Playbook REST service](https://doc.opendatadsl.com/docs/api/rest/service/aiplaybook), so the same rules and permissions apply.

Items whose id starts with # are read from public, any other id from private.

## Using the service

```js
// Read an item
item = ${aiplaybook:"dataset-delivery-summary"}

// Find items
items = find ${aiplaybook}
publicItems = find ${aiplaybook:public}

// Save an item
save ${aiplaybook:item}

// Tag the current version
tag ${aiplaybook:"dataset-delivery-summary"} as PROD

// Delete an item, or roll back to its previous version
delete ${aiplaybook:"dataset-delivery-summary"}
```


---

<!-- source: product/developer/Active Variable Services/AI-Run-Service.md -->

# AI Run Service

The **airun** active variable service gives your scripts access to the records of Fusion AI playbook runs (read-only). It uses the [AI Run REST service](https://doc.opendatadsl.com/docs/api/rest/service/airun), so the same rules and permissions apply.

## Using the service

```js
// Read an item
item = ${airun:"6721f0c2a4b9e85d3c0f1a77"}

// Find items
items = find ${airun}
```

This service is read-only.


---

<!-- source: product/developer/Active Variable Services/AI-Task-Service.md -->

# AI Task Service

The **aitask** active variable service gives your scripts access to Fusion AI playbook tasks. It uses the [AI Task REST service](https://doc.opendatadsl.com/docs/api/rest/service/aitask), so the same rules and permissions apply.

## Using the service

```js
// Read an item
item = ${aitask:"weekly-ttf-review"}

// Find items
items = find ${aitask}

// Save an item
save ${aitask:item}

// Delete an item, or roll back to its previous version
delete ${aitask:"weekly-ttf-review"}
```


---

<!-- source: product/developer/Active Variable Services/AI-Tool-Service.md -->

# AI Tool Service

The **aitool** active variable service gives your scripts access to your Fusion AI custom tools. It uses the [AI Tool REST service](https://doc.opendatadsl.com/docs/api/rest/service/aitool), so the same rules and permissions apply.

## Using the service

```js
// Read an item
item = ${aitool:"2b8f7d1e-5c3a-4e9b-a0d2-7f6e1c9b3a45"}

// Find items
items = find ${aitool}

// Save an item
save ${aitool:item}

// Delete an item, or roll back to its previous version
delete ${aitool:"2b8f7d1e-5c3a-4e9b-a0d2-7f6e1c9b3a45"}
```

See [AI Tool](https://doc.opendatadsl.com/docs/odsl/variable/ai) for the variable this service holds.


---

<!-- source: product/developer/Active Variable Services/AI-Usage-Service.md -->

# AI Usage Service

The **aiusage** active variable service gives your scripts access to Fusion AI token usage, one record per conversation per day (read-only). It uses the [AI Usage REST service](https://doc.opendatadsl.com/docs/api/rest/service/aiusage), so the same rules and permissions apply.

## Using the service

```js
// Read an item
item = ${aiusage:"6721f0c2a4b9e85d3c0f1a77"}

// Find items
items = find ${aiusage}
```

This service is read-only.


---

<!-- source: product/developer/Active Variable Services/Action-Service.md -->

[Workflow actions](https://doc.opendatadsl.com/docs/odsl/variable/action) allow you to personalise your processes with snippets of code to perform a specialised action within a workflow.

## Creating and saving actions

Creating actions are described in detail here, and once you have created your action, saving it is as easy as calling the **save** method as follows:

```js
save ${action:action_var}
```

E.g.
```js
xml_url = action in "extractors"
    // Extract XML from a HTTP URL
    in url as Scalar "The URL to read the XML from"
    out xml as Object "The XML file converted to an ODSL Object"
    exit "ok", "failed"
    
    assertType(input.url, "Scalar")
    on error ignore
    output.xml=${xml:input.url}
    if error
        print error
        return "failed"
    else
        return "ok"
    end
end
save ${action:xml_url}
```

## Listing actions

You can get a list of actions by using the **find** command as follows:

```js
var = find ${action:source/"query"} [where conditions]
```

An example of getting all public actions in the ‘extactors’ category:

```js
actions = find ${action:public/""} where category = "extractors"

for a in actions
    print a.id
next
```

## Getting a specific action

You can get an action by assigning it to a variable, e.g.

```js
xls = ${action:public/"extract_xls"}

print xls.description
```

## Deleting an action

You can delete an action using the delete command:

```js
delete ${action:"name"}
```


---

<!-- source: product/developer/Active Variable Services/Alert-Service.md -->

# Alert Service

The **alert** active variable service gives your scripts access to alerts: problems with your data, raised by the platform or by people. It uses the [Alert REST service](https://doc.opendatadsl.com/docs/api/rest/service/alert), so the same rules and permissions apply. See [Alerts in ODSL](https://doc.opendatadsl.com/docs/extensions/alerts/odsl) for a full guide.

Ids starting `ALC-` are read from common, `ALP-` from public, and any other id from private.

## Using the service

```js
// Find your open alerts
open = find ${alert:private} where status == "open"

// Find shared alerts (common in quotes)
common = find ${alert:"common"} where status == "open"
public = find ${alert:public} where status == "open"

// Read an alert with its timeline
a = ${alert:"ALR-2026-000401"}

// Comment on it, or change it, and save
a.comment = "Settlement run repeated with the republished contracts"
save ${alert:a}

// Raise an alert
n = SimpleObject()
n.title = "NGE.BROKER.NBP prices look stale since Monday"
n.severity = "high"
n.agent = "opendatadsl"
save ${alert:n}
```

Read an alert again before each change: an alert saved after someone else changed it is refused.

See [Alert](https://doc.opendatadsl.com/docs/odsl/variable/alert) for the fields of an alert.


---

<!-- source: product/developer/Active Variable Services/Archive.md -->

# Archive Service

The **archive** active variable service keeps a copy of a source file in your OpenDataDSL file archive, so you can see later exactly what a loader read. It downloads the file at the URL (HTTP, HTTPS or SFTP), stores it in your archive, and returns the text Archive Complete.

## Using the service

```js
status = ${archive:"https://www.example.com/prices/daily.csv"}
```

The type of the file is taken from the end of the URL. To give it yourself, put it in place of the source, for example `${archive:csv/"https://www.example.com/download?id=42"}`.


---

<!-- source: product/developer/Active Variable Services/Audit-Service.md -->

The audit service allows you to find and view audit records for actions performed in the OpenDataDSL environment.

## Finding Audit Records
You use the [find](https://doc.opendatadsl.com/docs/odsl/command/data-management#find) command to find audit records in ODSL.

### Find examples
#### Getting all private audit records

```js
records = find ${audit}
print records.size
```

#### Getting all public audit records

```js
records = find ${audit:public}
print records.size
```

#### Getting all records for a specific service

```js
records = find ${audit} where service="action"
print records.size
```

#### Getting all records within a time range

```js
records = find ${audit} where timestamp > ${date:"today"} and timestamp < ${date:"tomorrow"}
print records.size
```

#### Getting all records since a specific timestamp

```js
records = find ${audit} where timestamp > "2020-11-03T12:23:40"
print records.size
```

## Summarising Audit Records

You use the [aggregate](https://doc.opendatadsl.com/docs/odsl/command/structured#aggregate) command to summarise audit records in ODSL.

### Aggregation examples

#### Summarising by service

```js
summary = aggregate ${audit}
    group _id="$service", qty=count()
    sort qty desc
end

print summary
```


---

<!-- source: product/developer/Active Variable Services/Automation-Log-Service.md -->

# Automation Log Service

The **automationlog** active variable service gives your scripts access to the log of each automation run. It uses the [Automation Log REST service](https://doc.opendatadsl.com/docs/api/rest/service/automationlog), so the same rules and permissions apply.

## Using the service

```js
// Read an item
item = ${automationlog:"6721f0c2a4b9e85d3c0f1a77"}

// Find items
items = find ${automationlog}

// Delete an item, or roll back to its previous version
delete ${automationlog:"6721f0c2a4b9e85d3c0f1a77"}
```


---

<!-- source: product/developer/Active Variable Services/Automation-Service.md -->

# Automation Service

The **automation** active variable service gives your scripts access to your automations. It uses the [Automation REST service](https://doc.opendatadsl.com/docs/api/rest/service/automation), so the same rules and permissions apply.

## Using the service

```js
// Read an item
item = ${automation:"6721f0c2a4b9e85d3c0f1a77"}

// Find items
items = find ${automation}

// Save an item
save ${automation:item}

// Delete an item, or roll back to its previous version
delete ${automation:"6721f0c2a4b9e85d3c0f1a77"}
```

See [Automation](https://doc.opendatadsl.com/docs/odsl/variable/automation) for the variable this service holds.


---

<!-- source: product/developer/Active Variable Services/Automation-Target-Service.md -->

# Automation Target Service

The **automationtarget** active variable service gives your scripts access to the automation targets, standard and your own. It uses the [Automation Target REST service](https://doc.opendatadsl.com/docs/api/rest/service/automationtarget), so the same rules and permissions apply.

Items whose id starts with # are read from public, any other id from private.

## Using the service

```js
// Read an item
item = ${automationtarget:"odsl.email"}

// Find items
items = find ${automationtarget}
publicItems = find ${automationtarget:public}

// Save an item
save ${automationtarget:item}

// Tag the current version
tag ${automationtarget:"odsl.email"} as PROD

// Delete an item, or roll back to its previous version
delete ${automationtarget:"odsl.email"}
```

See [Automation Target](https://doc.opendatadsl.com/docs/odsl/variable/automation) for the variable this service holds.


---

<!-- source: product/developer/Active Variable Services/Azure-Blob-Service.md -->

# Azure Blob Service (azblob)

The **azblob** active variable service reads files from, and writes files to, Azure Blob Storage. A file that is read is converted with the file service that matches its type, for example CSV or JSON.

## Using the service

The id is the URL of the blob: the storage account, the container and the path. Give the storage account key in the `AccountKey` option, kept in a secret:

```js
// key holds the storage account key, read from a secret (see practical-notes.md)
options = "AccountKey=" + key.value
prices = ${azblob:"https://ngestorage.blob.core.windows.net/prices/daily/ttf.csv", options}
```

To write a variable to a blob as JSON, save it with the URL in the `url` option.


---

<!-- source: product/developer/Active Variable Services/Azure-File-Service.md -->

Azure File Service (azfile)
===========================================

A service to read files from an Azure Storage Account


---

<!-- source: product/developer/Active Variable Services/Batch-Service.md -->

# Batch Service

The **batch** active variable service gives your scripts access to the summaries of batch updates. It uses the [Batch REST service](https://doc.opendatadsl.com/docs/api/rest/service/batch), so the same rules and permissions apply.

## Using the service

```js
// Read an item
item = ${batch:"a757b7cc-ed3b-41f5-b5dd-30330b967416"}

// Find items
items = find ${batch}

// Save an item
save ${batch:item}

// Delete an item, or roll back to its previous version
delete ${batch:"a757b7cc-ed3b-41f5-b5dd-30330b967416"}
```

See [Batch](https://doc.opendatadsl.com/docs/odsl/variable/batch) for the variable this service holds.


---

<!-- source: product/developer/Active Variable Services/CSV.md -->

CSV
====

A service to read CSV files as a [list](https://doc.opendatadsl.com/docs/odsl/variable/list) of [row](https://doc.opendatadsl.com/docs/odsl/variable/row) variables or as a [table](https://doc.opendatadsl.com/docs/odsl/variable/table) variable

This service is used to convert CSV files into variables to be used within an OpenDataDSL script.

## Syntax

The standard active variable syntax is followed with the **id** part being the URL of the CSV file, e.g.
```js
csvdata = ${csv:"https://domain/example.csv", options}
```

## Options

The following options affect the way that the CSV data is processed into the variable:

|**Option**|**Description**|**Example**|
|-|-|-|
|separator|The character or regex which is used to separate the columns in the CSV file. The default separator is a comma|separator= +|
|headerpos|The line which contains the header information where the column names are extracted from. The default is 1, if there are no headers, use 0|headerpos=0|
|datapos|The number of lines to skip from the header position to where the data starts. The default is 0|datapos=1|
|output|The output variable type, if it is set as **table** then the variable is of type [table](https://doc.opendatadsl.com/docs/odsl/variable/table), anything else it is a list of [row](https://doc.opendatadsl.com/docs/odsl/variable/row) variables|output=table|

## Example

The following example extracts a csv file from UCI Machine Learning Repository for [balloons](https://archive.ics.uci.edu/ml/datasets/Balloons)
```js
// Grab one of the UCI data sets for Balloons
// https://archive.ics.uci.edu/ml/datasets/Balloons

csvdata = ${csv:"https://archive.ics.uci.edu/ml/machine-learning-databases/balloons/adult+stretch.data","headerpos=0"}
print csvdata.size
for row in csvdata
    print row
next

// Get the data as a table
csvdata = ${csv:"https://archive.ics.uci.edu/ml/machine-learning-databases/balloons/adult+stretch.data","headerpos=0", "output=table"}
print csvdata.rows.size
for row in csvdata.rows
    print row
next
```


---

<!-- source: product/developer/Active Variable Services/CURL.md -->

# CURL Service

The **curl** active variable service downloads a file from a URL using the curl tool, and reads it with the file service that matches its type, in the same way as the file service. Use it for sites that the standard web download cannot read.

## Using the service

```js
data = ${curl:"https://www.example.com/prices/daily.csv"}
```


---

<!-- source: product/developer/Active Variable Services/Calendar-Service.md -->

Calendar Service
=================

The calendar service allows you to create and use all the [calendars](https://doc.opendatadsl.com/docs/odsl/calendar/calendars) in the system

## Introduction

The calendar service is the interface for the calendar repository.

Calendars are either:

*   Public - they are available to everybody and maintained by OpenDataDSL    
*   Private - they are proprietary and only available to you and your colleagues
    

All public calendar codes start with the # character in order to distinguish them from private calendar codes.

## Getting a calendar

If you know the code for a calendar, you can get it from the service using the following active variable syntax:
```js
calendar = ${calendar:code}
```
Example:
```js
// Get the public England and Wales holiday calendar
enghols = ${calendar:"#eng"}
```
This returns a new [Calendar](https://doc.opendatadsl.com/docs/odsl/calendar/Calendars) variable with the England and Wales holiday rules configured.

## Saving a calendar

If you create a new calendar or edit an existing one, you can save the calendar back to the service using the [save](https://doc.opendatadsl.com/docs/odsl/command/save) command.

Example:
```js
/** 
 Create an hourly calendar using a holiday calendar and 
 exclude hours 0-6 and 20-23
**/
ukpeak = IntradayCalendar(1h, holiday, \[0,1,2,3,4,5,6,20,21,22,23\])

// Save the calendar to the service
save ${calendar:ukpeak}
```
## Listing Calendars

To get a list of available calendars, you can use the [find](https://doc.opendatadsl.com/docs/odsl/command/find) command with the calendar service.

Example:
```js
calendars = find ${calendar:public}
for cal in calendars
    print cal.code
next
```
This gets a list of all the public calendars and prints their code to the console


---

<!-- source: product/developer/Active Variable Services/Correction-Service.md -->

# Correction Service

The **correction** active variable service gives your scripts access to corrections made to dataset data after it was delivered (read-only). It uses the [Correction REST service](https://doc.opendatadsl.com/docs/api/rest/service/correction), so the same rules and permissions apply.

## Using the service

```js
// Read an item
item = ${correction:"6721f0c2a4b9e85d3c0f1a77"}

// Find items
items = find ${correction}
```

This service is read-only.


---

<!-- source: product/developer/Active Variable Services/Currency-Service.md -->

Currency Service
=================

A service to use the publically available FX rates


---

<!-- source: product/developer/Active Variable Services/Curve-Service.md -->

Curve Service
=============

The curve service is used for interacting with Forward Curve configurations


---

<!-- source: product/developer/Active Variable Services/DHTML.md -->

DHTML
======

A service to read Dynamic HTML pages as an [object](https://doc.opendatadsl.com/docs/odsl/variable/object) variable


---

<!-- source: product/developer/Active Variable Services/Data-Service.md -->

# Data Service

The **data** active variable service gives your scripts access to timeseries, curves and other data held on your master data. It uses the [Data REST service](https://doc.opendatadsl.com/docs/api/rest/service/data), so the same rules and permissions apply.

Items whose id starts with # are read from public, any other id from private.

## Using the service

```js
// Read an item
item = ${data:"NGE.GAS.TTF:MID"}

// Find items
items = find ${data}
publicItems = find ${data:public}

// Delete an item, or roll back to its previous version
delete ${data:"NGE.GAS.TTF:MID"}
```

See [TimeSeries](https://doc.opendatadsl.com/docs/odsl/variable/timeseries) for the variable this service holds.


---

<!-- source: product/developer/Active Variable Services/Dataset-Delivery-Service.md -->

# Dataset Delivery Service

The **dataset_delivery** active variable service gives your scripts access to the delivery record of each monitored dataset for each date. It uses the [Dataset Delivery REST service](https://doc.opendatadsl.com/docs/api/rest/service/dataset_delivery), so the same rules and permissions apply.

## Using the service

```js
// Read an item
item = ${dataset_delivery:"ICE.NDEX.NLB:2026-10-23"}

// Find items
items = find ${dataset_delivery}
```

This service is read-only.


---

<!-- source: product/developer/Active Variable Services/Dataset-Service.md -->

# Dataset Service

The **dataset** active variable service gives your scripts access to dataset monitoring: the datasets and feeds you monitor and their daily deliveries. It uses the [Dataset REST service](https://doc.opendatadsl.com/docs/api/rest/service/dataset), so the same rules and permissions apply.

## Using the service

The part after the colon in a find is the section of the dataset service to read: info for datasets, feed for dataset feeds and delivery for deliveries.

```js
// Find the datasets you monitor
datasets = find ${dataset:info}

// Find the dataset feeds you monitor
feeds = find ${dataset:feed}

// Find today's late deliveries
late = find ${dataset:delivery} where status == "late"

// Create a dataset configuration and save it
ds = ${dataset}
ds.dsid = "ICE.NDEX.NLB"
save ds
```

A new dataset from `${dataset}` is a Dataset object; a dataset needs a `dsid` in the form provider.feed.product before it can be saved. Saving a DatasetFeed object saves a feed instead.


---

<!-- source: product/developer/Active Variable Services/Date-Service.md -->

Date Service
=============

Create and utilise [dates](https://doc.opendatadsl.com/docs/odsl/variable/date)

## Assignment - creating a date

To use the date service to create a new date, use the Active Variable assign statement syntax as shown below:
```js
var = ${date:"\[calendar\]/key"}
```
Where:

*   var is the name of the variable to create    
*   date is the name of the service    
*   calendar is an optional calendar to use defaulting to daily    
*   key is the key which defines what date to assign to the variable
    

The following assignment keys are available (case-insensitive):

|**Key**|**Description**|
|-|-|
|now|The current day and time|
|today or \{empty\}|The current backwards aligned day using the supplied calendar|
|yesterday|The day prior to today according to the supplied calendar|
|tomorrow|The day after today according to the supplied calendar|
|named, e.g. monday|The next or current named day|
|ISO Date|The supplied parsed absolute date i.e. doesn’t use the supplied calendar|
|ISO Date Time|The supplied parsed absolute date time|
|Period Code|The start date of the supplied period code|

:::note
All dates in OpenDataDSL are stored in the UTC timezone
:::


---

<!-- source: product/developer/Active Variable Services/Documentation-Service.md -->

# Documentation Service

The **documentation** active variable service gives your scripts access to documentation for items and services. It uses the [Documentation REST service](https://doc.opendatadsl.com/docs/api/rest/service/documentation), so the same rules and permissions apply.

Items whose id starts with # are read from public, any other id from private.

## Using the service

```js
// Read an item
item = ${documentation:"NGE.GAS.TTF"}

// Find items
items = find ${documentation}
publicItems = find ${documentation:public}

// Save an item
save ${documentation:item}

// Tag the current version
tag ${documentation:"NGE.GAS.TTF"} as PROD

// Delete an item, or roll back to its previous version
delete ${documentation:"NGE.GAS.TTF"}
```

See [Documentation](https://doc.opendatadsl.com/docs/odsl/variable/documentation) for the variable this service holds.


---

<!-- source: product/developer/Active Variable Services/Environment-Service.md -->

Environment Service
===================

This service allows you to manage your data environments

## Creating a new data environment

To create a new data environment, you simply need to create an Object with the same name you want to have for the environment and then save it with the environment active variable, here is an example:
```js
test = Object()
save ${environment:test}
```
This creates a data environment called test.

You can add other properties to the object which will get stored along with the environment, e.g. if you want to add a description, you can do the following:
```js
test = Object()
test.description = "My test environment"
save ${environment:test}
```
## Listing environments

To list your environments, use the **find** command:
```js
env = find ${environment:""}

for e in env    
    print json(e)
next
```
This will output the details about your environment as something like the following:
```json
{
  "_id": "test",
  "_createdBy": "user@company.com",
  "_size": "163840",
  "_timestamp": "2020-09-17T14:16:15.847Z",
  "description": "My test environment"
}
```
The JSON object contains the following properties:

*   _id - This is the name of the environment    
*   _createdBy - This is the email address of the user that created the environment    
*   _size - This is the current storage size of the environment in bytes    
*   _timestamp - This is the date and time of when the environment was first created    

It will also include any dynamic properties you added when you created/updated it

## Adding more information

You can add or change any dynamic properties on the environment by re-saving it, e.g. using the example above, if I wanted to add a department name:
```js
test = ${environment:"test"}
test.department = "Operations"
save ${environment:test}
```
If you re-read the environment, you will now see the department property added:
```js
test = ${environment:"test"}
print json(test)
```
```json
{
  "_id": "test",
  "_createdBy": "user@company.com",
  "_size": "163840",
  "_timestamp": "2020-09-17T14:16:15.847Z",
  "department": "Operations",
  "description": "My test environment"
}
```

## Deleting an environment

You can delete an environment using the **delete** command as follows:

```js
delete ${environment:"name"}
```
:::danger
Deleting an environment, deletes all the data saved in that environment!
:::


---

<!-- source: product/developer/Active Variable Services/Event-Service.md -->

Event Service
=================

The event service allows you to read all the [events](https://doc.opendatadsl.com/docs/odsl/variable/event) in the system

## Introduction

The event service is the interface for the event repository.

Events are either:

*   Public - they are available to everybody and maintained by OpenDataDSL
*   Private - they are proprietary and only available to you and your colleagues

An event is always connected to a Master Data Record (Object)

## Finding events
You use the find command to find events in the system.

### Using the event property
You would usually use the ```event``` property in the query to filter to the specific event list you want to find, e.g.

```js
find ${event:public} where event = "#ABN.FX:SPOT"
```

### Using a date range
You can filter the objects by eventtime - the time the event is recorded for.

```js
find ${event:public} 
    where event = "#ABN.FX:SPOT" 
    and eventtime >= "2024-06-03" 
    and eventtime <= "2024-06-07"
```

You can also filter the objects by eventstart and eventend - the time period the event represents.

```js
find ${event:public} 
    where event = "#DCE.CN.Y.SOYBEAN_OIL:FUTURES" 
    and eventstart >= "2024-01-01" 
    and eventend <= "2024-01-31"
```

## Creating events
An example of creating some metrics, adding onto a Master Data Record and saving

```js
// Create the temperature metric
evtemp = Event(${date:"now"})
evtemp.eventtype = "#Metric"
evtemp.type = "temperature"
evtemp.value = 14
evtemp.id = ObjectId()

// Create the humidity metric
evhum = Event(${date:"now"})
evhum.eventtype = "#Metric"
evhum.type = "humidity"
evhum.value = 25
evhum.id = ObjectId()

// Add the events to an object and save it
s9754 = new Object()
s9754.MESSAGE = [evtemp, evhum]
save ${object:s9754}
```


---

<!-- source: product/developer/Active Variable Services/Expiry-Calendar-Service.md -->

Expiry Calendar Service
=======================

The expiry calendar service allows you to manage and use [expiry calendars](https://doc.opendatadsl.com/docs/odsl/calendar/expiry) in your scripts

## Introduction

The expiry calendar service is the interface to the expiry calendar repository.

Expiry calendars are either:

*   Public - they are available to everybody and maintained by OpenDataDSL    
*   Private - they are proprietary and only available to you and your colleagues
    

All public expiry calendar codes start with the # character in order to distinguish them from private expiry calendar codes.

## Getting an expiry calendar

To use an expiry calendar, you assign the active variable to a local variable in your script using this syntax:
```js
ecal = ${expiry:code}
```
Example:
```js
// Get the end of month expiry calendar with business frequency trading calendar
eombus = ${expiry:"#EOMBUSINESS"}
```
This returns a new [expiry calendar](https://doc.opendatadsl.com/docs/odsl/calendar/expiry) with the expiry rules configured.

## Saving an expiry calendar

If you create a new expiry calendar or edit an existing one, you can save it back to the service using the **save** command.

Example:
```js
eombus = ExpiryCalendar(BusinessCalendar())
eombus.addRule("go to the end of the previous month")
save ${expiry:eombus}
```

## Listing expiry calendars

To get a list of available expiry calendars, you can use the **find** command with the expiry calendar service.

Example:
```js
calendars = find ${expiry:public/""}
print calendars
```
This gets a list of all the public expiry calendars and prints them to the console


---

<!-- source: product/developer/Active Variable Services/Extension-Service.md -->

[Extensions](https://doc.opendatadsl.com/docs/odsl/variable/extension) allow you to create or use application extensions which extend the GUI and Excel Add-in.

## Creating and saving extensions

Creating extensions are described in detail here, and once you have created your extension, saving it is as easy as calling the **save** method as follows:

```js
save ${extension:e}

// or simply
save e
```

E.g.
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

## Listing extensions

You can get a list of extensions by using the **find** command as follows:

```js
var = find ${extension:source/"query"} [where conditions]
```

An example of getting all public extensions:

```js
ext = find ${extension:public}

for e in ext
	print e.name
next
```

An example of getting all private (installed) extensions:

```js
ext = find ${extension}

for e in ext
	print e.name
next
```

## Getting a specific extension

You can get an extension by assigning it to a variable, e.g.

```js
ext = ${extension:public/"odsl.example"}
print ext.description
```

## Publishing an extension

```js
e = ${extension:"odsl.example"}
publish e to "public"
```

## Installing an extension

```js
e = ${extension:public/"odsl.example"}
publish e to "private"
```

## Uninstalling an extension

```js
delete ${extension:"odsl.example"}
```

:::warning
If you run the command to uninstall an extension that has been created by you, it will be completely removed and would need to be re-created if needed
:::


---

<!-- source: product/developer/Active Variable Services/Extractor-Service.md -->

Extractor Service
==================

The extractor service allows you to manage and use data extractors


---

<!-- source: product/developer/Active Variable Services/FTP.md -->

FTP
====

A service to read files from an FTP site


---

<!-- source: product/developer/Active Variable Services/File.md -->

File
=====

A service to read files from the local file system


---

<!-- source: product/developer/Active Variable Services/GZIP.md -->

GZIP
===========

A service to handle the extraction of data from a compressed zip file


---

<!-- source: product/developer/Active Variable Services/Group-Service.md -->

# Group Service

The **group** active variable service gives your scripts access to groups of items, such as favourites, curve build groups and export groups. It uses the [Group REST service](https://doc.opendatadsl.com/docs/api/rest/service/group), so the same rules and permissions apply.

## Using the service

```js
// Read an item
item = ${group:"6721f0c2a4b9e85d3c0f1a77"}

// Find items
items = find ${group}

// Save an item
save ${group:item}

// Tag the current version
tag ${group:"6721f0c2a4b9e85d3c0f1a77"} as PROD

// Delete an item, or roll back to its previous version
delete ${group:"6721f0c2a4b9e85d3c0f1a77"}
```

See [Group](https://doc.opendatadsl.com/docs/odsl/variable/group) for the variable this service holds.


---

<!-- source: product/developer/Active Variable Services/HTML.md -->

HTML
====================

A service to read a HTML page as an [object](https://doc.opendatadsl.com/docs/odsl/variable/object) variable


---

<!-- source: product/developer/Active Variable Services/Index-Service.md -->

# Index Service

The **index** active variable service gives your scripts access to the database indexes on your services. It uses the [Index REST service](https://doc.opendatadsl.com/docs/api/rest/service/index), so the same rules and permissions apply.

## Using the service

```js
// Read an item
item = ${index:"region_1"}

// Find items
items = find ${index}

// Save an item
save ${index:item}

// Delete an item, or roll back to its previous version
delete ${index:"region_1"}
```

See [Index](https://doc.opendatadsl.com/docs/odsl/variable/index) for the variable this service holds.


---

<!-- source: product/developer/Active Variable Services/Introduction.md -->

Introduction
============

Detailed information on Active Variables and the services used by them

Active variables are the defining feature of OpenDataDSL which allows the user to interact directly with the Data Management System through the use of a well-defined, simple to use variable construct.

In OpenDataDSL, active variables provide a way to interact with the database and services to help you collect, validate, manage and report on your data.

The syntax of an active variable is as follows:
```js
${service (:source)? }
${service: (source/)? id (:version)? (,option)* } (for range)?
```
Examples:
```js
actvar = ${action:"#name"}
save ${action:actvar}
tag ${action:actvar} as PROD
actvarprod = ${action:"#name":PROD}

objects = find ${object:public} where dataset == "BOE_FX"
```

## External Data Collection Services

A list of services that help you collect data from various locations and various file formats

### Azure File Service (azfile)
A service to read files from an Azure Storage Account
    
### CSV
A service to read CSV files as a [list](https://doc.opendatadsl.com/docs/odsl/variable/list) of [row](https://doc.opendatadsl.com/docs/odsl/variable/row) variables or as a [table](https://doc.opendatadsl.com/docs/odsl/variable/table) variable
    
### DHTML
A service to read Dynamic HTML pages as an [object](https://doc.opendatadsl.com/docs/odsl/variable/object) variable
    
### File
A service to read files from the local file system
    
### FTP
A service to read files from an FTP site
    
### GZIP
A service to handle the extraction of data from a compressed zip file
    
### HTML
A service to read a HTML page as an [object](https://doc.opendatadsl.com/docs/odsl/variable/object) variable
    
### JSON
A service to read a JSON format file as an [object](https://doc.opendatadsl.com/docs/odsl/variable/object) variable
    
### ODS
A service to read an ODS (OpenDocument) format file as a [spreadsheet](https://doc.opendatadsl.com/docs/odsl/variable/spreadsheet) variable
    
### PDF
A service to read a PDF format file as an [object](https://doc.opendatadsl.com/docs/odsl/variable/object) variable
    
### S3
A service to read files from an AWS S3 bucket
    
### XLS
A service to read XLS spreadsheet files as a [spreadsheet](https://doc.opendatadsl.com/docs/odsl/variable/spreadsheet) variable
    
### XML
A service to read XML format files as an [object](https://doc.opendatadsl.com/docs/odsl/variable/object) or [list](https://doc.opendatadsl.com/docs/odsl/variable/list) variable
    

### Archive
A service to keep a copy of a source file in your file archive

### Azure Blob Service (azblob)
A service to read and write files in Azure Blob Storage

### CURL
A service to download files with the curl tool

### Kafka
A service to read JSON messages from Apache Kafka topics

### Mail
A service to read emails and their attachments from a mailbox

## OpenDataDSL Remote Services

This is a list of active variable services that interact with the OpenDataDSL remote services

### Action Service
Workflow actions allow you to personalise your processes with snippets of code to perform a specialised action within a workflow

### Audit Service
The audit service allows you to view all the changes that have happened in your environment
    
### Calendar Service
The calendar service allows you to create and use all the [calendars](https://doc.opendatadsl.com/docs/odsl/calendar/calendars) in the system
    
### Currency Service
A service to use the publically available FX rates
    
### Curve Service
The curve service is used for interacting with Forward Curve configurations
    
### Environment Service
This service allows you to manage your data environments
    
### Expiry Calendar Service
The expiry calendar service allows you to manage and use [expiry calendars](https://doc.opendatadsl.com/docs/odsl/calendar/expiry) in your scripts
    
### Extractor Service
The extractor service allows you to manage and use data extractors
    
### Object Service
The object service allows you to manage, browse, search for and use stored objects
    
### Process Service
The process service allows you to manage your scheduled and event driven remote processes

### Secret Service
The secret service allows you to securely store secrets such as passwords or security tokens

### Script Service
The script service allows you to store and retrieve ODSL scripts
    
### Tenant Service
A service to view and record your company information and tenancy configuration
    
### Transformer Service
The transformer service allows you to store and manage data transformers
    
### Type Service
The type service allows you to store and maintain data types used in [data modelling](https://doc.opendatadsl.com/docs/odsl/dm/modelling)
    
### User Service
A service to view and record your user information

### Workflow Service
The workflow services allows you to store and manage [workflows](https://doc.opendatadsl.com/docs/odsl/dm/workflow)
    
    

### AI Agent Service
Your Fusion AI agents

### AI Assistant Service
The Fusion AI assistants, built-in and custom

### AI Playbook Service
Fusion AI playbooks

### AI Run Service
The records of Fusion AI playbook runs

### AI Task Service
Fusion AI playbook tasks

### AI Tool Service
Your Fusion AI custom tools

### AI Usage Service
Fusion AI token usage

### Alert Service
Alerts about your data and the shared data you use

### Automation Service
Your automations

### Automation Target Service
Automation targets

### Automation Log Service
The log of each automation run

### Batch Service
The summaries of batch updates

### Correction Service
Corrections to dataset data

### Data Service
Timeseries, curves and other data

### Dataset Service
Dataset monitoring

### Dataset Delivery Service
Dataset deliveries

### Documentation Service
Documentation for items and services

### Group Service
Groups of items

### Index Service
Database indexes

### Metric Service
Usage metrics

### Policy Service
Security policies

### Process Execution Service
Process execution logs

### Queue Service
Message queues

### Queue Log Service
The log of queue messages

### Report Service
Stored reports

### Report Configuration Service
Report configurations

### Report Creator Service
Report creators

### Script Log Service
The log of each script run

### Task Service
Tasks assigned to users

### Validation Service
Validation records

### Validation Config Service
Validation configurations

## Variable Services

These services help you work with OpenDataDSL variables

### Date Service
Create and utilise dates
    
### Memory Service
Recreate the current session by saving all variables in memory, then re-loading them later


---

<!-- source: product/developer/Active Variable Services/JSON.md -->

JSON
====================

A service to read a JSON format file as an [object](https://doc.opendatadsl.com/docs/odsl/variable/object) variable


---

<!-- source: product/developer/Active Variable Services/Kafka.md -->

# Kafka Service

The **kafka** active variable service reads JSON messages from Apache Kafka topics and returns them as a [list](https://doc.opendatadsl.com/docs/odsl/variable/list) of [objects](https://doc.opendatadsl.com/docs/odsl/variable/object), the same shape the JSON service returns for a JSON array.

## Using the service

```js
messages = ${kafka:"prices-*","bootstrap=b-1.example.com:9196;b-2.example.com:9196,group=nge_prices,username=nge_reader"}
```

The id is a topic name, several topic names separated by `;`, or a pattern: an id with `*` is a wildcard pattern, and an id starting with `^` is a regular expression.

Each read returns the messages published since the consumer group last read the topic, up to `maxRecords`. A message that is a JSON array gives one object per element.

## Options

Options are given after the id as `name=value` pairs separated by commas. Use `;` to separate several values inside one option.

|**Option**|**Description**|**Default**|
|-|-|-|
|bootstrap|The broker list, e.g. b-1:9196;b-2:9196 (required)||
|group|The consumer group id (required)||
|username|The SASL username||
|password|The SASL password. Without it, the password configured for the platform is used||
|protocol|SASL_SSL, SSL, SASL_PLAINTEXT or PLAINTEXT|SASL_SSL|
|mechanism|SCRAM-SHA-512, SCRAM-SHA-256 or PLAIN|SCRAM-SHA-512|
|maxRecords|The most messages returned by one read|1000|
|maxWait|The most milliseconds one read can take|30000|
|idleTimeout|Stop after this many milliseconds with no new messages|3000|
|offsetReset|earliest or latest: where a new consumer group starts|earliest|
|commit|true or false: whether to record that the returned messages have been read|true|
|metadata|true adds _topic, _partition, _offset, _key and _timestamp to each object|false|
|failOnBadMessage|true fails the read on a message that is not JSON, instead of skipping it|false|
|kafka.name|Any Kafka consumer property, e.g. kafka.fetch.max.bytes=52428800||

:::caution
With commit=true, the messages are marked as read before the read returns. If your script fails afterwards, those messages are not read again. Use commit=false while you test.
:::


---

<!-- source: product/developer/Active Variable Services/Mail.md -->

# Mail Service

The **mail** active variable service reads emails from a mailbox over IMAP or POP3, and reads their attachments with the file service that matches each attachment's type, for example CSV or XLS. Zip and gzip attachments are unpacked. The result is a list of the files read.

## Using the service

The id is the mail server host:

```js
login = "user=prices@northgate-energy.com,password=" + pwd
files = ${mail:"imap.example.com", login + ",sender=settlements@example-broker.com,subject=Daily settlement prices"}
```

## Options

|**Option**|**Description**|**Default**|
|-|-|-|
|user|The mailbox user||
|password|The mailbox password. Keep it in a secret||
|agent|imap or pop3|imap|
|port|The server port|993 for IMAP, 995 for POP3|
|folder|The folder to read|INBOX|
|seen|true reads emails that have been read, false reads unread emails|false|
|sender|Only read emails from these senders||
|subject|Only read emails with this subject||
|fromDateTime|Only read emails received on or after this date, as dd-MM-yyyy||
|toDateTime|Only read emails received on or before this date, as dd-MM-yyyy||
|filter_files|Only read the attachments with these file names, separated by ;||


---

<!-- source: product/developer/Active Variable Services/Memory-Service.md -->

Memory Service
==============================

Recreate the current session by saving all variables in memory, then re-loading them later

## Saving the current session

At any point in a scripting or CLI session, you can save the current state of all the variables using the following syntax:
```js
save ${memory:location/"name"}
```
For example, to save to a local file named session.json in the /temp directory:
```js
save ${memory:"/temp/session.json"}
```

## Loading a saved session

To load a saved session, you need to do the following:
```js
var=${memory:location/"name"}
```
The returned var is ignored and the entire memory is replaced with the saved session

For example, to retrieve the session.json memory from the /temp directory:
```js
x=${memory:"/temp/session.json"}
```


---

<!-- source: product/developer/Active Variable Services/Metric-Service.md -->

# Metric Service

The **metric** active variable service gives your scripts access to the usage metrics gathered for billing. It uses the [Metric REST service](https://doc.opendatadsl.com/docs/api/rest/service/metric), so the same rules and permissions apply.

## Using the service

```js
// Read an item
item = ${metric:"6721f0c2a4b9e85d3c0f1a77"}

// Find items
items = find ${metric}
```

This service is read-only.

See [Metric](https://doc.opendatadsl.com/docs/odsl/variable/metrictimeseries) for the variable this service holds.


---

<!-- source: product/developer/Active Variable Services/ODS.md -->

ODS
===================

A service to read an ODS (OpenDocument) format file as a [spreadsheet](https://doc.opendatadsl.com/docs/odsl/variable/spreadsheet) variable


---

<!-- source: product/developer/Active Variable Services/Object-Service.md -->

Object Service
==============

The object service allows you to manage, browse, search for and use stored objects

## Creating Objects

All objects have a [type](https://doc.opendatadsl.com/docs/odsl/variable/declaredtype) - this is a structure that defines the properties that you would expect on an object. Individual objects can also have dynamic properties that are not included in the type structure. If you create an object without specifying a type, ODSL will add a #Object type to it.

### Creating a basic object

You can create a simple object using the basic Object constructor, e.g.
```js
myobject = Object()
```
This creates a dynamic object of type #Object which has the following standard properties:

*   name    
*   description    
*   classification    
*   geolocation
    

### Creating a typed object

You can create an object from a type in 2 different ways:

#### Using a constructor
```js
london = Object("#Location")
```
#### Using an object structure
```js
london = object as #Location
    name = "London"
end
```
The only difference between the 2 methods is that with the object structure you can directly set some properties.

## Saving Objects

To save an object, you use the save command as shown in the code snippet below:
```js
myobject = Object()
myobject.name = "My Test"

save ${object:myobject}
```
This will save a private object called myobject of type #Object to the database.

## Reading Objects

You can find objects using the [find](https://doc.opendatadsl.com/docs/odsl/command/find) command and searching for objects is covered in this [article](https://doc.opendatadsl.com/docs/tutorials/searching). If you know the id of the object you want, you can directly read the object using the active variable, e.g.
```js
myobject = ${object:"myobject"}
```

## Deleting Objects

To delete an object from the database, you use the delete command.
```js
delete ${object:"myobject"}
```
If [versioning](https://doc.opendatadsl.com/docs/odsl/dm/versioning) is enabled on the object type you are deleting, then the delete command performs a roll back. To completely delete a versioned object, add a * version modifier to the delete command, e.g.

```js
delete ${object:"myobject":*}
```


---

<!-- source: product/developer/Active Variable Services/PDF.md -->

PDF
===================

A service to read a PDF format file as an [object](https://doc.opendatadsl.com/docs/odsl/variable/object) variable


---

<!-- source: product/developer/Active Variable Services/Policy-Service.md -->

# Policy Service

The **policy** active variable service gives your scripts access to your security policies. It uses the [Policy REST service](https://doc.opendatadsl.com/docs/api/rest/service/policy), so the same rules and permissions apply.

## Using the service

```js
// Read an item
item = ${policy:"NGE.READ.GAS"}

// Find items
items = find ${policy}

// Save an item
save ${policy:item}

// Delete an item, or roll back to its previous version
delete ${policy:"NGE.READ.GAS"}
```

See [Policy](https://doc.opendatadsl.com/docs/odsl/variable/policy) for the variable this service holds.


---

<!-- source: product/developer/Active Variable Services/Process-Execution-Service.md -->

# Process Execution Service

The **exec** active variable service gives your scripts access to the process execution logs. It uses the [Process Execution REST service](https://doc.opendatadsl.com/docs/api/rest/service/process-execution), so the same rules and permissions apply.

## Using the service

```js
// Read an execution log
log = ${exec:"6090915ad6a48d0b3c4f663d"}

// Find the failed executions of a process
failed = find ${exec} where name == "ECB_FX" and status == "failed"

// Stop a running execution
delete ${exec:"6090915ad6a48d0b3c4f663d"}
```

You cannot save to this service. To run a process, use the process service.


---

<!-- source: product/developer/Active Variable Services/Process-Service.md -->

Process Service
===============

The process service allows you to manage your scheduled and event driven remote processes


---

<!-- source: product/developer/Active Variable Services/Queue-Log-Service.md -->

# Queue Log Service

The **queuelog** active variable service gives your scripts access to the log of messages sent to your queues. It uses the [Queue Log REST service](https://doc.opendatadsl.com/docs/api/rest/service/queuelog), so the same rules and permissions apply.

## Using the service

```js
// Read an item
item = ${queuelog:"6721f0c2a4b9e85d3c0f1a77"}

// Find items
items = find ${queuelog}

// Delete an item, or roll back to its previous version
delete ${queuelog:"6721f0c2a4b9e85d3c0f1a77"}
```


---

<!-- source: product/developer/Active Variable Services/Queue-Service.md -->

# Queue Service

The **queue** active variable service gives your scripts access to your message queues. It uses the [Queue REST service](https://doc.opendatadsl.com/docs/api/rest/service/queue), so the same rules and permissions apply.

## Using the service

```js
// Read an item
item = ${queue:"etrm"}

// Find items
items = find ${queue}

// Save an item
save ${queue:item}

// Delete an item, or roll back to its previous version
delete ${queue:"etrm"}
```


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

<!-- source: product/developer/Active Variable Services/S3.md -->

S3
==================

A service to read files from an AWS S3 bucket


---

<!-- source: product/developer/Active Variable Services/Script-Log-Service.md -->

# Script Log Service

The **scriptlog** active variable service gives your scripts access to the log of each script run. It uses the [Script Log REST service](https://doc.opendatadsl.com/docs/api/rest/service/scriptlog), so the same rules and permissions apply.

## Using the service

```js
// Read an item
item = ${scriptlog:"6721f0c2a4b9e85d3c0f1a77"}

// Find items
items = find ${scriptlog}

// Delete an item, or roll back to its previous version
delete ${scriptlog:"6721f0c2a4b9e85d3c0f1a77"}
```


---

<!-- source: product/developer/Active Variable Services/Script-Service.md -->

Script Service
==============

The script service allows you to store and retrieve ODSL scripts


---

<!-- source: product/developer/Active Variable Services/Task-Service.md -->

# Task Service

The **task** active variable service gives your scripts access to tasks assigned to users. It uses the [Task REST service](https://doc.opendatadsl.com/docs/api/rest/service/task), so the same rules and permissions apply.

## Using the service

```js
// Read an item
item = ${task:"64255eae35f43713c1afa927"}

// Find items
items = find ${task}

// Save an item
save ${task:item}

// Tag the current version
tag ${task:"64255eae35f43713c1afa927"} as PROD

// Delete an item, or roll back to its previous version
delete ${task:"64255eae35f43713c1afa927"}
```

See [Task](https://doc.opendatadsl.com/docs/odsl/variable/task) for the variable this service holds.


---

<!-- source: product/developer/Active Variable Services/Tenant-Service.md -->

Tenant Service
==============

A service to view and record your company information and tenancy configuration

## Description

The tenant service is a private only service that allows you to record information about your company and view information about your tenancy configuration such as service quotas etc.

## Saving your company information

The following information is recorded about your company:

*   name    
*   address1    
*   address2    
*   city    
*   postcode    
*   email    
*   phone    
*   accountContact    

To add this information in OpenDataDSL code, you need to create a basic Object and save it to the tenant service. Copy the code below, fill in the blanks and run it:
```js
company = Object()
company.name = ""
company.address1 = ""
company.address2 = ""
company.city = ""
company.postcode = ""
company.email = ""
company.phone = ""
company.accountContact = ""
save ${tenant:company}
```

## Viewing your tenant information

To look at your tenant information, run the following code:
```js
print ${tenant:""}
```
This will show in JSON the information you entered about your company, plus service quota and various other information.


---

<!-- source: product/developer/Active Variable Services/Transformer-Service.md -->

Transformer Service
====================

The transformer service allows you to store and manage data transformers


---

<!-- source: product/developer/Active Variable Services/Type-Service.md -->

Type Service
=============

The type service allows you to store and maintain data types used in [data modelling](https://doc.opendatadsl.com/docs/odsl/dm/modelling)


---

<!-- source: product/developer/Active Variable Services/User-Service.md -->

User Service
=============

A service to view and record your user information

## Description

The user service is a private only service that allows you to record information about you and view information about your colleagues.

## Saving your User Information

You can save the following information about yourself in the system:

*   name    
*   address1    
*   address2    
*   city    
*   postcode    
*   email    
*   phone   

To add this information using ODSL, you need to create an Object, set properties on it and save it to the user service. Copy the code below, fill in the blanks and run it.
```js
user = Object()
user.name = ""
user.address1 = ""
user.address2 = ""
user.city = ""
user.postcode = ""
user.email = ""
user.phone = ""
save ${user:user}
```

## Viewing your User Information

To look at your user information, run the following code:
```js
print ${user:"me"}
```
This will show in JSON the information you entered about you and various other information.

## Viewing your Colleagues

To get a list of your colleagues that have connected to OpenDataDSL, run the following code:

```js
colleagues = find ${user}
print colleagues
```


---

<!-- source: product/developer/Active Variable Services/Validation-Config-Service.md -->

# Validation Config Service

The **validationconfig** active variable service gives your scripts access to the validation checks configured for each curve and timeseries. It uses the [Validation Config REST service](https://doc.opendatadsl.com/docs/api/rest/service/validationconfig), so the same rules and permissions apply.

## Using the service

```js
// Read an item
item = ${validationconfig:"NGE.GAS.TTF:MID"}

// Find items
items = find ${validationconfig}

// Save an item
save ${validationconfig:item}

// Tag the current version
tag ${validationconfig:"NGE.GAS.TTF:MID"} as PROD

// Delete an item, or roll back to its previous version
delete ${validationconfig:"NGE.GAS.TTF:MID"}
```


---

<!-- source: product/developer/Active Variable Services/Validation-Service.md -->

# Validation Service

The **validation** active variable service gives your scripts access to validation records: the result of each check on your curves and timeseries. It uses the [Validation REST service](https://doc.opendatadsl.com/docs/api/rest/service/validation), so the same rules and permissions apply.

## Using the service

```js
// Read an item
item = ${validation:"NGE.GAS.TTF:MID:2026-10-23:M01"}

// Find items
items = find ${validation}
```

This service is read-only.


---

<!-- source: product/developer/Active Variable Services/Workflow-Service.md -->

Workflow Service
================

The workflow services allows you to store and manage [workflows](https://doc.opendatadsl.com/docs/odsl/dm/workflow)


---

<!-- source: product/developer/Active Variable Services/XLS.md -->

XLS
===================

A service to read XLS spreadsheet files as a [spreadsheet](https://doc.opendatadsl.com/docs/odsl/variable/spreadsheet) variable


---

<!-- source: product/developer/Active Variable Services/XML.md -->

XML
===================

A service to read XML format files as an [object](https://doc.opendatadsl.com/docs/odsl/variable/object) or [list](https://doc.opendatadsl.com/docs/odsl/variable/list) variable

This service is used to convert XML files into variables to be used within an OpenDataDSL script.

## Syntax

The standard active variable syntax is followed with the **id** part being the URL of the XML file, e.g.
```js
xmldata = ${xml:"https://domain/example.xml", options}
```

## Options

The following options affect the way that the XML data is processed into the variable:

|**Option**|**Description**|**Example**|
|-|-|-|
|selector|This is an XML selector which allows you to filter the xml data. It is a CSS-like element selector, that finds elements matching a query. If a selector is used, the output variable is a [list](https://doc.opendatadsl.com/docs/odsl/variable/list)|```selector=[type=”test”]```|
|ignoreContentType|Set this to true if the document content type is causing exceptions|ignoreContentType=true|
|xslt|Transforms the input XML data using the supplied XSLT transformer|"xslt="+xslt|

## Example

This example uses the following XML format file of FX Rates from the European Central Bank obtainable from

[https://www.ecb.europa.eu/stats/eurofxref/eurofxref-daily.xml](https://www.ecb.europa.eu/stats/eurofxref/eurofxref-daily.xml)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<gesmes:Envelope xmlns:gesmes="http://www.gesmes.org/xml/2002-08-01" xmlns="http://www.ecb.int/vocabulary/2002-08-01/eurofxref">
	<gesmes:subject>Reference rates</gesmes:subject>
	<gesmes:Sender>
		<gesmes:name>European Central Bank</gesmes:name>
	</gesmes:Sender>
	<Cube>
		<Cube time='2021-06-03'>
			<Cube currency='USD' rate='1.2187'/>
			<Cube currency='JPY' rate='133.81'/>
			<Cube currency='BGN' rate='1.9558'/>
			<Cube currency='CZK' rate='25.448'/>
			<Cube currency='DKK' rate='7.4365'/>
			<Cube currency='GBP' rate='0.85955'/>
			<Cube currency='HUF' rate='345.82'/>
			<Cube currency='PLN' rate='4.4520'/>
			<Cube currency='RON' rate='4.9220'/>
			<Cube currency='SEK' rate='10.1145'/>
			<Cube currency='CHF' rate='1.0961'/>
			<Cube currency='ISK' rate='146.30'/>
			<Cube currency='NOK' rate='10.1501'/>
			<Cube currency='HRK' rate='7.5013'/>
			<Cube currency='RUB' rate='89.2163'/>
			<Cube currency='TRY' rate='10.5650'/>
			<Cube currency='AUD' rate='1.5792'/>
			<Cube currency='BRL' rate='6.1894'/>
			<Cube currency='CAD' rate='1.4710'/>
			<Cube currency='CNY' rate='7.7910'/>
			<Cube currency='HKD' rate='9.4551'/>
			<Cube currency='IDR' rate='17420.91'/>
			<Cube currency='ILS' rate='3.9598'/>
			<Cube currency='INR' rate='88.8755'/>
			<Cube currency='KRW' rate='1357.75'/>
			<Cube currency='MXN' rate='24.3300'/>
			<Cube currency='MYR' rate='5.0241'/>
			<Cube currency='NZD' rate='1.6915'/>
			<Cube currency='PHP' rate='58.208'/>
			<Cube currency='SGD' rate='1.6141'/>
			<Cube currency='THB' rate='37.938'/>
			<Cube currency='ZAR' rate='16.5218'/>
		</Cube>
	</Cube>
</gesmes:Envelope>
```

### Retrieving the XML

Running the following code:
```js
xmldata=${xml:"https://www.ecb.europa.eu/stats/eurofxref/eurofxref-daily.xml"}
print xmldata
```
Produces an object looking something like this:
```json
{
  "_id": "xmldata",
  "Cube": {
    "_id": "Cube",
    "Cube": {
      "_id": "Cube",
      "Cube": \[
        {
          "_id": "Cube",
          "content": "",
          "currency": "USD",
          "rate": 1.2187
        },
        {
          "_id": "Cube",
          "content": "",
          "currency": "JPY",
          "rate": 133.81
        }
        
        ...
        
      \],
      "content": "",
      "time": "2021-06-03"
    },
    "content": ""
  },
  "Sender": {
    "_id": "Sender",
    "content": "European Central Bank",
    "name": {
      "_id": "name",
      "content": "European Central Bank"
    }
  },
  "subject": {
    "_id": "subject",
    "content": "Reference rates"
  }
}
```

### Using a selector
If you use a selector to only select nodes that have a currency attribute:
```js
xmldata=${xml:"https://www.ecb.europa.eu/stats/eurofxref/eurofxref-daily.xml", "selector=\[currency\]"}
print xmldata
```
You get a list of objects containing only the currencies and rates like this:
```json
[
{
  "content": "",
  "currency": "USD",
  "rate": 1.2187
}
{
  "content": "",
  "currency": "JPY",
  "rate": 133.81
}

...

]
```

### Using an XSLT

You can create an XSLT that transforms the XML, see the example below:

```js
xslt=`<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
<xsl:output method="xml" encoding="utf-8" indent="no"/>

<xsl:template match="/">
<xsl:variable name="time" select="//@time"/>
<currencies time="{$time}">
<xsl:for-each select="//@currency">
<currency code="{.}" rate="{../@rate}" />
</xsl:for-each>
</currencies>
</xsl:template>

</xsl:stylesheet>
`

xmldata=${xml:"https://www.ecb.europa.eu/stats/eurofxref/eurofxref-daily.xml","xslt="+xslt}

print json(xmldata)
```
