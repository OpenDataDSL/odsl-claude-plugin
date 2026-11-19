# ODSL commands, statements, expressions and conditions

Syntax and examples for every ODSL command.

Generated from the OpenDataDSL documentation (https://doc.opendatadsl.com/docs).


---

<!-- source: product/developer/odsl/Commands/Commands.md -->

Commands
========

A summary of the commands in the OpenDataDSL language

## Statements
All the general OpenDataDSL statement commands

## Control Statements
Statements that control the flow of execution

## Data Management Commands 
These commands help you manage the resources in your data

## Structured Commands 
A list of complex commands that create structures or perform more complex tasks

## Expressions
Expressions usage within the OpenDataDSL language

## Conditions
The usage of conditions in the OpenDataDSL Language


---

<!-- source: product/developer/odsl/Commands/Statements.md -->

General Statements
==================

All the general OpenDataDSL statement commands

## api-key
Command to manage your user API Keys

#### Syntax
```js
apikey: api-key (create (start="string")? (expires="string")? |delete) "name";
```
#### Description

An API Key is used as a password to authenticate you in OpenDataDSL from an external application that cannot use OAUTH2 or Azure Active Directory.   

##### Explanation of the syntax

*   The **name** is a string that is used to identify what you are going to use the key for
*   The **create** directive creates a key
    * You can add an optional start date for the key
    * You can add an optional expires date for the key
*   The **delete** directive deletes (revokes) the named key

:::info
After running the create command, the key is output into the debug console, make sure you copy the **key** as this is the only time you will see it.
:::

#### Examples

Create a key that starts now and never expires
```js
api-key create "mykey" 
```

Create a key that starts and a date and expires
```js
api-key create start="2024-05-01" expires="2024-12-31T23:59:59" "mykey"
```

Revoke a key
```js
api-key delete "mykey"
```

List all your keys
```js
print ${user:"me"}.keys
```

## import
Imports functions from a script into the current script

#### Syntax
```js
import ${script:"name"}
```
#### Description

The import command gets the named script from the script service and makes all the user defined functions in that script available in your current script.

#### Example

If we have a script with a function, e.g. this functions.odsl script saved in the server:
```js
// functions.odsl
function agricultural(input)
    // Agricultural curve forward filled and extended to end of 3 years
    filled = forwardFillCurve(input)
    extended = extendCurve(filled, 3)
    agricultural = shape(extended)
end
```
We can import that script and call the agricultural function, e.g.
```js
// Import the functions we defined in functions.odsl
import ${script:"functions"}

inputcurve = ${data:"inputcurve"}
curve = agricultural(inputcurve)
```

## log
Outputs a message to the console and to a process execution log

#### Syntax
```js
log LEVEL message
```
#### Description

If the script is running in a process, the message is output into the process execution log. The message is also output to the console.

The **LEVEL** option allows you to specify the log level of the message from one of:

*   debug    
*   info    
*   warn    
*   fatal
    

#### Examples
```js
log debug "This is a debug message"
log info "This is an info message"
log warn "This is a warning message"
log fatal "This is a fatal message"

a = 12.2
log debug "The value is " + a
```

## logout
Used to log out the current user

#### Syntax
```js
logout
```
#### Description

This command is used in a script to log out the current user

## onerror
Determines how errors are handled

#### Syntax
```js
on error (exit|ignore|return "transition")
```
#### Description

The on error statement instructs OpenDataDSL on how you want to handle errors. You have 3 options:

*   exit - this is the default action which stops the currently running script and outputs the error message    
*   ignore - this option doesn’t stop the script running, but it places the error message into a scalar variable called **error**
*   return "transition" - only in a workflow action: when an error happens, the workflow follows the named transition instead of failing
    

#### Examples

Using on error exit:
```js
on error exit

// Create a date variable
d = Date()

// Call an unknown method that will force throwing an error
print d.unknown()
```

Using on error ignore:
```js
on error ignore

// Create a date variable
d = Date()

// Call an unknown method that will force throwing an error
print d.unknown()

// Script continues and we can see the error message in the variable called error
print error
```

## wait
Pauses execution of a script for a certain amount of time

#### Syntax
```js
wait number ('second'|'seconds'|'minute'|'minutes'|'hour'|'hours')
```

#### Description

Pauses the script for the time given, for example to give a remote process time to finish before reading its results.

#### Examples
```js
// Wait 30 seconds
wait 30 seconds
```

## print
Prints a message to the console

#### Syntax
```js
print message
```
#### Description

The print command prints a message to the console. The message can be a string, variable or expression.

#### Examples
```js
a = "example"
print a
```
```
example
```
```js
print "This is an " + a
```
```
This is an example
```

## publish
Publish data or an extension

#### Syntax
```js
publish {var} to {location} (log {reason})?
```
#### Description

The publish command can do the following:
* Publish data to another tenant (if you have the rights given by the receiving tenant)
* Publish an extension to the public repository or install a public extension to your private environment

#### Examples
```js
// Publishing an extension to the public repository
e = ${extension:"odsl.example"}
publish e to "public"
```

```js
// Installing an extension to an environment
e = ${extension:public/"odsl.example"}
publish e to "private"
```

## return
Terminates the currently running script and returns a variable

#### Syntax
```js
return expression
```
#### Description

The return statement terminates running of a script and returns a variable to the calling application

#### Examples
```js
return "Finished"
```

## run
Runs a process remotely

#### Syntax
```js
run (service)? processname (:version)? (with input)? (log reason)? (in delay TIMEUNIT)?
```
#### Description

The run command remotely triggers a manual run of a process. You can optionally provide a variable with input run parameters, a reason log message that will be added to the audit log and a delay for when to run the process.

#### Options

##### With input

You can optionally provide a variable with some input information which will override or provide specific context to the process, e.g.
```js
// Run a curve
ondate = Object()
ondate.date = "2021-04-26"
run CURVE_TEST with ondate
```

##### Log reason

You can optionally give a reason for manually triggering a process, this will be logged in the audit log

##### Run on another service

Some services run their own functions with run: give the service name before the id, e.g.
```js
// Run the validation checks of a curve
result = run validation "ICE.IFEU.B:SETTLE" with "2026-10-19"
```
See [Validation: ODSL](https://doc.opendatadsl.com/docs/extensions/validation/reference#odsl) for the validation functions.

##### In delay

You can optionally delay the starting of the process, e.g.
```js
run MY_PROCESS in 1 hour
```
The timeunit parameter can be one of:

*   second    
*   seconds    
*   minute    
*   minutes    
*   hour    
*   hours

## send
Send a batch of objects to a message queue

#### Syntax
```js
send variable (to queue)? (size INT)? (-noevent)?
```
#### Description

The send command sends a [batch](https://doc.opendatadsl.com/docs/odsl/variable/batch) variable to a [message queue](https://doc.opendatadsl.com/docs/odsl/dm/queues) in order to be processed by either the data loader or an external application.

If no queue name is specified, the batch is sent to your default queue.

##### Options

**TO queue**

You can optional specify the name of the queue to send the batch to.

**SIZE INT**

You can override the default number of items sent per message using the size command

**-noevent**

Add this option if you don’t want to trigger downstream processes, exports etc. from the sending of this message. This would be usually done if you are back-filling some historic data and only require that the data is added.

#### Examples

```js
// Create a batch
batch=PROCESS.createBatch()

// Add some objects to the batch
...

// Send to the default queue
send batch

// Send the batch to the queue named 'myqueue'
send batch to myqueue

// Send the batch to the queue in batches of 10 objects at a time
send batch size 10 

// Send the batch to update but not trigger downstream events
send batch -noevent
```

## set
Used to set a session option

#### Syntax
```js
set (precision INT|
    rounding ROUNDING_METHOD|
    missing (ignore|number)?|
    crs (earth|planar)|
    observed (default|beginning|end|averaged|summed|high|low|delta)|
    distribution (none|constant|linear|cubic)|
    autoscale (on|off)|
    credentials for source as user password|
    auto_follow_references (true|false)|
    while_limit number|
    minimal interval number TIMEUNIT|
    default_property (object)?
    )?
```
#### Description

The set command is used to set an option within the current script session. The following options are supported:

*   decimal precision    
*   rounding method    
*   missing value treatment    
*   coordinate reference system
*   timeseries scaling methods
*   credentials for interacting with external systems
*   options for following referenced data
*   while loop iteration limit
    

##### Setting decimal precision

The decimal precision determines the number of decimal places to use with real numbers, e.g.
```js
set precision 2
```
Sets the decimal places to 2 for all numbers

##### Setting the rounding method

The rounding method determines how numbers are rounded when shortening to meet the decimal precision value, you can select from one of the following methods:

*   ceiling    
*   down    
*   up    
*   floor    
*   half_down    
*   half_up    

##### Setting missing value treatment

Missing values can affect the way calculations are performed and therefore you can set the way that you want missing values to be treated. You can instruct ODSL to ignore missing values using the statement:
```js
set missing ignore
```
Or you can tell ODSL to use a specific value to replace missing values, e.g. if you are summing up a list of values, you can set missing as 0 so that it would not take them into account.

##### Setting the coordinate reference system (CRS)

You can set the CRS to either:

*   earth    
*   planar
    

This affects the way [geospatial data](https://doc.opendatadsl.com/docs/odsl/dm/geospatial) is handled

##### Timeseries scaling methods

There are 3 set commands which allow you to define how timeseries are scaled:

* autoscale on|off 
> By default, all timeseries are autoscaled once they reach a certain threshold in terms of number of observations.
> You can turn this off using the following command:

```js
set autoscale off
```

See [this page](https://doc.opendatadsl.com/docs/odsl/calendar/scaling#auto-scaling) for more information on autoscaling.

* observed (default|beginning|end|averaged|summed|high|low|delta)
> This defines how numbers are aggregated when a timeseries is scaled from high frequency e.g. hourly to a lower frequency e.g. daily
> 
> See [this page](https://doc.opendatadsl.com/docs/odsl/calendar/scaling#observed-setting) for more information on the observed setting.

* distribution (none|constant|linear|cubic)
> This defines how numbers are distributed when a timeseries is scaled from low frequency e.g. daily to a higher frequency e.g. hourly
> 
> See [this page](https://doc.opendatadsl.com/docs/odsl/calendar/scaling#distribution) for more information on the distribution setting.

##### Credentials

If you need to define personal credentials to an external system that the OpenDataDSL platform interacts with you use the set credentials command.
This securely stores the credentials in an Azure Vault and only uses them when needed to interact with the external system.

##### Automatically follow references

This command enables or disables automatically retrieving the underlying data from a referenced entity.

For example, if we take the master data record ```#ABN_FX.EURAED```, it has a SPOT reference to the SPOT timeseries data.
In the following example (the default), the information output is the actual timeseries data.

```js
set auto_follow_references true

o = ${object:"#ABN_FX.EURAED"}
print o.SPOT
```

In this example, the information output is the information about the reference link

```js
set auto_follow_references false

o = ${object:"#ABN_FX.EURAED"}
print o.SPOT
```

#### While loop iteration limit

While loops can iterate infinitely causing performance issues, so we have an artificial limit on while loops of ```1000``` iterations.

If this limit is reached, a ```6036 While loop iteration``` exception is thrown.

You can override the 1000 iterations limit using the set while_limit command, e.g.

```js
set while_limit 100
```

##### minimal interval

Sets the shortest period an intraday calendar can have (1 minute by default), so you can work with data at a finer granularity, e.g.

```js
set minimal interval 15 minutes
```

##### default_property

By default, reading a property that doesn't exist on an object gives null. Use `set default_property object` to create a new empty object for a missing property instead, so you can set nested properties directly, e.g. `obj.settings.colour = "blue"`. Use `set default_property` to go back to the default.

##### set on its own

`set` with no option prints the current settings: the stage, environment, runtime, precision, observed, distribution and coordinate reference system.

## use
Instructs OpenDataDSL to use a specific named environment

#### Syntax
```js
use environmentname
```
#### Description

You can create multiple environments such as dev and test which are separate from your production environment, this command instructs OpenDataDSL that from this point it should use that named environment for the rest of the running script.

NOTE: For more information on environments - see [here](https://doc.opendatadsl.com/docs/odsl/service/environment)

#### Example

If you have created an environment called test, you can use it with the following command:
```js
use test
```


---

<!-- source: product/developer/odsl/Commands/control.md -->

Control Statements
========

Statements that control the flow of execution

## for
A control statement that loops through a range of numbers of a list of elements

#### Syntax

The for statement comes in 2 flavours:

##### Looping through a range of numbers
```js
for varname = expression TO expression (step number)?
  (statement)* 
next
```
The optional step operator is the number to iterate by which defaults to 1

##### Looping through a list of elements
```js
for varname in expression
  (statement)* 
next
```
#### Description

The for statement is used to iterate through numbers or elements and execute some code.

#### Examples

Iterating through a range of values
```js
// Print out the numbers 1 to 10
for i=1 to 10
  print i
next
```
Iterating through a range of values using a step value
```js
// Print the number 2, 4, 6, 8 and 10
for i=2 to 10 step 2
  print i
next
```
Iteration through values in an array
```js
for i in [1,7,2,10]
  print i ^ 2
next
```
Getting a list of available currencies and iterating through them printing our the currency pair
```js
currencies = find ${currency:public}
for fx in currencies
  print fx.currency + "/" + fx.base
next
```

## if
Used to test a boolean condition and take appropriate actions

#### Syntax
You can use the if structure like the below:
```js
if condition
  (statement)* 
(elseif condition (statement)*)* 
(else (statement)* )?
end
```

Or you can use a single line assignment conditional expression using a ```?``` operator:

```
varname = condition ? true_expression : false_expression
```

#### Description

The if statement is a control statement - it controls the flow of the application based on the value of a boolean condition. An if statement can have an multiple elseif blocks which will be sequentially checked to see if there condition is true and an optional else block of statements that execute if the if or elseif condition does not match.

The single line assignment conditional expression can be used to test a condition and assign a value to a variable based on the trueness of the condition.
This can also be used in SMART data expressions, e.g.

```js
BASE.size() < 10 ? BASE : OTHER
```
#### Examples

A simple condition that checks the value of a variable
```js
if a==1
  print "a is 1"
end
```
A multiple condition if statement
```js
if a==1 and b>5
  a=b
end
```
An example of using an else block that will execute if a is something other than the value 1
```JS
if a==1
  a=a+1
else
  a=a-1
end
```
An example showing 2 if conditions (if and elseif) and a else block that will execute if either of the if conditions don’t match
```js
if a==1
  a=a+1
elseif a>5
  a=a+10
else
  a=a-1
end
```
An example showing how to use a single line variable assignment example:
```js
a = score>85 ? "Score is high" : "Score is low"
```

## while
Iterates through some lines of code whilst a condition remains true

#### Syntax
```js
while condition
  (statement)*
end
```
#### Description

The while command executes some code whilst a condition remains true

Warning - ensure that the condition can at some point become false, otherwise the code will execute endlessly

#### Examples
```js
a=99
while a>1
  a = a/2
end
```

## throw
Throw a custom exception

#### Syntax
```js
throw expression
```

#### Description
The throw command allows you to throw a custom exception that will return to the calling application.

#### Examples
```js
throw "Invalid value for category [" + cat + "] must be 'x' or 'y'"
```


---

<!-- source: product/developer/odsl/Commands/data-management.md -->

Data Management
========

These commands help you manage the resources in your data

## find
Used to search the database

#### Syntax
```js
varname = find 
  (top n | unique field | profile profilename (for range)? | for range | info field)?
  (from)?
  (activevar|avservice)
  (where (condition)+)?
  (order by field (asc|desc)? (, field (asc|desc)?)*)?
  (project (!)?field (, (!)?field)*)?
  (end)?
```

##### Layout

The find command can be used in a single line or multi-line format for better readability, e.g.
```js
// Single line search of private audit records
records = find ${audit} where timestamp > ${date:"today"} and timestamp < ${date:"tomorrow"}

// Multi-line version of the same search
records = find ${audit} where 
    timestamp > ${date:"today"} 
    and timestamp < ${date:"tomorrow"}
end
```

##### Options

###### top

The top option allows you to find a smaller sample of data, e.g.

```js
// Retrieve the first 15 objects
objects = find top 15 ${object} where dataset="ARGUS_DEL"
```

###### unique

The unique option allows you to get a list of unique values for a specific field in a resource

###### profile

The profile option is only used when searching through objects. It allows you to search objects, but return a list of data objects linked to the object where the profile name matches the passed in profilename. This option also allows you to specify a **range**, which can be a single date or a range of dates in the following syntax:

*   Single date as a Date or a String, e.g. “2021-07-16”
*   Single date using the date active variable service, e.g. $\{date:”yesterday”\}
*   From a date, e.g. from(“2021-01-01”)
*   The last number of days, e.g. last(3)
*   A range of dates using between, e.g. between(“2021-01-01”,$\{date:”yesterday”\})
    

###### condition

The condition option is used to filter the results of the find command. The syntax of the conditions is as follows:
```js
expression (<|<=|>|>=|=|==|!=|like|intersects|within) expression
    | condition (and|or) condition
    | ( condition )
```

####### Examples of conditions

```js
category = "extractors"
timestamp > "2020-11-03T12:23:40"
timestamp > ${date:"today"} and timestamp < ${date:"tomorrow"}
name like "ch"
location within Sphere([ 51.72961, 0.47612 ], 20 / 3963.2)
```

#### Description

The find command is a powerful way of searching a resource for the data you require. It returns a [virtual list](https://doc.opendatadsl.com/docs/odsl/variable/virtuallist) of items that match the specified conditions.

#### Examples

##### Find all

In its simplest form, the find command can be used to list all items from a service, e.g.
```js
// Get a list of all public calendars
calendars = find ${calendar:public}
```

##### Simple filtering
```js
// Get a list of public actions for a specific category
pactions = find ${action:public} where category = "extractors"
```

##### Filtering by date
```js
// Get a list of private audit records for today
records = find ${audit} where timestamp > ${date:"today"} and timestamp < ${date:"tomorrow"}
```

##### A list of data using the object profile option
```js
data = find profile SPOT for yesterday from ${object:public} where dataset == "ECB_FX"
```

##### Various date range queries
```js
// Data in get query
data = ${data:"#ECB_FX.EURZAR:SPOT"} for ${date:"yesterday"}
data = ${data:"#ECB_FX.EURZAR:SPOT"} for yesterday
data = ${data:"#ECB_FX.EURZAR:SPOT"} for last(3)
data = ${data:"#ECB_FX.EURZAR:SPOT"} for from(yesterday)
data = ${data:"#ECB_FX.EURZAR:SPOT"} for between("2021-05-01",yesterday)
```

##### Geospatial
```js
// Define a polygon and search for objects within it
london = Polygon([[51.5386, -0.4956],[51.6445, -0.0753],[51.5205, 0.1753],[51.3479, -0.1163],[51.5386, -0.4956]])
items = find ${object:"TestGeometry"} where location within london
```

###### order by

Sorts the results, e.g.

```js
recent = find ${audit} where service == "object" order by timestamp desc
```

###### project

Returns only the named fields. Put `!` before a field to leave it out instead, e.g.

```js
names = find ${object} where dataset == "ARGUS_DEL" project name, description
```

###### info

Counts the items by the values of a field instead of returning them, like the `_fieldinfo` REST parameter, e.g.

```js
bytype = find info _type from ${object}
```

## save
Used to save items to the database

#### Syntax
```js
save varname   (log reason) (replace)
save activevar (log reason) (replace)
```
#### Description

The save command saves an item to the database.

There are 2 forms:
* Simply save using the variable name and let OpenDataDSL determine the service to use, e.g. ```save x```
* Explicitly state the service using an activevar, e.g. ```save ${object:x}```

##### Log
The optional log reason is to give a human-readable reason why this item is being saved and this is stored in the audit log.

##### Replace
The optional replace statement is used with the object service to signify that you want to replace the stored object with the passed in object. This overrides the default action which is to merge the contents of the supplied object with the stored object.

##### Services
You can save to any [active variable service](https://doc.opendatadsl.com/docs/odsl/service/services) that stores items, for example action, aiagent, aiassistant, aiplaybook, aitask, aitool, alert, automation, automationtarget, batch, calendar, curve, dataset, documentation, environment, event, expiry, extractor, group, index, object, policy, process, queue, report, reportconfig, reportcreator, script, secret, task, transformer, type, user, validationconfig and workflow. The Azure File (azfile) and Azure Blob (azblob) services can also write files, and the memory service saves your variables.

#### Example
```js
// Create private type
TypeExample = type
    name as String() default "test"
end

// Save the type
save TypeExample

// Create a private object
ObjExample = object as TypeExample
    name = "Hello"
end

// Save the object
save ObjExample log "Created the new object"

// Change the object
ObjExample.name = "Hello World"

// Save the object using replace option to replace the stored object
save ObjExample log "Changed the object" replace
```

## bulk update
Used to bulk update items in the database

#### Syntax
```js
bulk update avservice (where condition)? (upsert)?
    (log string)?
    ( (updateoperator|comment)* | (pipelineOperator|comment)* )
end
```

#### Description
Only the following services are supported:
*   object
*   event

The optional **where condition** limits the documents that are updated, if ommitted, all documents are updated

The optional **upsert** option will insert a document if there is no matching document

The optional **log** option allows you to specify a reason for the update which is added to the audit log

#### Examples
Example using update operators
```js
bulk update ${object}
    set(name="test",type="real")
    set(description=Hello)
    inc(count=1,test=5)    
    rename(temp="real")
    unset(test)    
end
```

Example using pipeline operators
```js
bulk update ${object}
    addFields status="Modified", comments=["$misc1", "$misc2"]
    project misc1, misc2
end
```

Example using upsert
```js
bulk update ${object} where _id=1 upsert
    set(item="apple")
    setOnInsert(defaultQty=100)
end
```

Example using a log message
```js
bulk update ${object} where name="b4"
    log "Audit record 123456"
    set(test="Hello Again")
    inc(count=1)
end
```

## link
Links two objects together

#### Syntax
```js
link activevar (as)? parent(/child)? (of)? activevar
```

#### Description

Adds a named link between two of your private objects: the second object gets a link called **parent** to the first, and, if you give a child name, the first object gets a link with that name to the second. Both objects are saved.

Both objects must be private, and their types must allow links.

#### Examples
```js
// The site is the parent of the meter, and the meter is a child of the site
link ${object:"NGE.SITE.EEMSHAVEN"} as site/meter of ${object:"NGE.METER.EEM01"}
```

## rename
Used to rename a resource in the database

#### Syntax
```js
rename ${service:"resource"} as newname (log "reason")?
```
#### Description

The rename command is used to change the unique id of a specific resource in a service.

#### Examples
```js
// Rename a privat eobject
rename ${object:"OBJ1"} as UK_OBJ

// Rename a calendar id and add an audit log message
rename ${calendar:"mycal"} as UKHOURLY log "Changed to correct id"
```

#### See Also
* [Renaming Reports](https://doc.opendatadsl.com/docs/topics/reporting/basics#renaming-reports)

## tag
Tags a version of an item with a name

#### Syntax
```js
tag var AS name
```
#### Description

The tag command is used to name a specific version of an item so that it can be read using the supplied name, e.g. you can tag versions of your actions with PROD or TEST to ensure that you use a specific version of an action in a workflow.

#### Examples

```js
tag ${action:"ReadStuff":1} as PROD

prodaction = ${action:"ReadStuff":PROD}
```

## delete
Used to rollback versions or delete items from the database

#### Syntax
```js
delete activevar (log string)?
```

#### Description
In the standard form, it rolls back the version of an item from the database: the previous version becomes the current one, or the item is deleted if it has only one version. Services that do not keep versions delete the item.

You can use any [active variable service](https://doc.opendatadsl.com/docs/odsl/service/services) that stores items, for example action, aiplaybook, alert, automation, calendar, curve, data, dataset, documentation, environment, event, expiry, extractor, group, index, object, policy, process, queue, report, reportconfig, script, secret, task, transformer, type, validationconfig and workflow. Your access policies decide what you can delete. To stop a running process, delete its execution with the [exec service](https://doc.opendatadsl.com/docs/odsl/service/exec).

You can specify the version number to delete with the activevar syntax: ```${service:"resource":version}```

The version can also be *, in which case it will fully delete the item from the database

The optional **log** option allows you to specify a reason for the deletion which is added to the audit log

#### Examples
```js
// To rollback to the previous version
delete ${object:"TEST"}

// To rollback to the previous version, specifying an audit log message 
delete ${script:"Test Script"} log "Removed TEST object"

// To delete a specific version
delete ${object:"TEST":1}

// To delete all versions
delete ${object:"TEST":*}
```

## bulk delete
Bulk delete can remove multiple records from certain supported services

#### Syntax
```js
bulk delete avservice (log string)? (where condition)?
```

In bulk delete form, the following services are supported for delete and restore, which means once the records are deleted, they can be restored using the [bulk restore](#bulk-restore) command.
* event
* object

:::note
Both the current and archived versions of the items are deleted/restored.
:::

The following services can be used to bulk delete only - no restore
* audit
* automationlog
* batch
* dataset_delivery
* exec
* metric
* queuelog
* scriptlog
* validation

## bulk restore
Bulk restore is used to ```undo``` a [bulk delete](#bulk-delete)

#### Syntax
```js
bulk restore avservice (log string)? (where condition)?
```

The following services can use the bulk restore command:
* event
* object

#### Examples
```js

// To delete all object that are of type="MyType"
bulk delete ${object} where _type="MyType" 

// To restore objects of type="MyType"
bulk restore ${object} where _type="MyType" 

// To completely remove audit records prior to 1st June 2025
bulk delete ${audit} where timestamp < "2025-06-01"

```


---

<!-- source: product/developer/odsl/Commands/structured.md -->

Structured Commands
====

A list of commands that create structures or perform more complex tasks.

## Introduction
A structured command simplifies the creation of the more complex elements in OpenDataDSL.
A structure generally takes the form:

```
varname = structure
    (config)*
    (statement)*
end
``` 

## action
Command to create actions that can be used in workflows

#### Syntax
```js
varname = (action|gateway) in "category"
  (comment)? 
  (actionInput|actionOutput)* 
  (actionExit)? 
  (statement)* 
end

actionInput: in varname as declaredType ("desc")? (optional)?
actionOutput: out varname as declaredType ("desc")?
actionExit: exit "name" (, "name")*
```
#### Description

A workflow action is a small block or snippet of code that performs a specific task and can be used in your own custom workflows. They can have input and output data and 1 or more exit transitions which can be configured in a workflow.

##### Explanation of the syntax

*   **action** or **gateway** \- This command can create actions or gateways:    
    *   An action takes optional inputs, performs a task and creates optional outputs        
    *   A gateway takes an input and based on that input takes a specific exit transition        
*   The **category** is a string that is used to place the action in a specific category    
*   The **comment** on the first line of the action is used as the description of the action    
*   Action inputs can have an **optional** directive    
*   The action **exits** are simply string exit names, such as “ok” or “failed”
    

#### Examples

A simple send batch action
```js
test_send_batch = action in "loaders"
    // Send a batch of data to the server to be updated
    in batch as Object "The batch to upload"
    exit "ok","failed"
    
    on error ignore
    send input.batch
    if error
        return "failed"
    else
        return "ok"
    end
end

// Save the action to the server
save ${action:test_send_batch}
```

An action to read an object from the object service

```js
read_object = action in "general"
    // Read an object
    in id as Scalar "The ID of the object to read"
    out obj as Object "The ODSL Object"
    exit "ok", "failed" 

    on error ignore 
    output.obj=${object:input.id}
    if error
        print error
        return "failed"
    else
        return "ok"
    end        
end
```

## aggregate
Create an aggregation pipeline to group and summarise data

#### Syntax
```js
varname = (aggregate|pipeline) service (save)? (allowDiskUse)?
  (pipelineItem)* 
end

pipelineItem: (addFields|bucket|bucketAuto|count|documents|facet|geoNear|graphLookup
    |group|limit|lookup|match|merge|out|project|redact|replaceRoot|replaceWith
    |sample|skip|sort|sortByCount|unwind|jsonObject|comment|NL);
pipelineMatch: match (condition)+;
pipelineGroup: group assign (, assign)*;
pipelineSort: sort sortItem (, sortItem)*;
sortItem: (assign|varname (asc|desc)?);
```

Each stage is described in [Aggregation Stages](https://doc.opendatadsl.com/docs/odsl/stages). You can also write a stage as a MongoDB JSON stage, e.g. `{"$count": "total"}`. **pipeline** is another name for **aggregate**. **allowDiskUse** lets large aggregations use disk space on the database server, and **save** also saves the pipeline as a database view named after the output variable.
#### Description

The aggregate command creates an aggregation pipeline to group and summarise data from any service.

##### Explanation of the syntax

*   The **match** pipeline item allows you to filter out data using a condition clause    
*   The **group** pipeline item groups or summarises data using one or more of the following summarising functions:    
    *   count() - counts the number of occurances        
*   The **sort** pipeline item sorts the results according to the specified field and asc(ending) or desc(ending) order
    

#### Examples

The following example summarises the status of process executions filtered for a specific service
```js
summary = aggregate ${exec}
    match service="ETL"
    group _id="$status", qty=count()
    sort qty desc
end
```

#### More about aggregation
* [More details about aggregation stages](https://doc.opendatadsl.com/docs/odsl/stages)

## find
Used to search the database

#### Syntax
```js
varname = find 
  (top n | unique field | profile profilename (for range)? | for range | info field)?
  (from)?
  (activevar|avservice)
  (where (condition)+)?
  (order by field (asc|desc)? (, field (asc|desc)?)*)?
  (project (!)?field (, (!)?field)*)?
  (end)?
```

##### Layout

The find command can be used in a single line or multi-line format for better readability, e.g.
```js
// Single line search of private audit records
records = find ${audit} where timestamp > ${date:"today"} and timestamp < ${date:"tomorrow"}

// Multi-line version of the same search
records = find ${audit} where 
    timestamp > ${date:"today"} 
    and timestamp < ${date:"tomorrow"}
end
```

##### Options

###### top

The top option allows you to find a smaller sample of data, e.g.

```js
// Retrieve the first 15 objects
objects = find top 15 ${object} where dataset="ARGUS_DEL"
```

###### unique

The unique option allows you to get a list of unique values for a specific field in a resource

###### profile

The profile option is only used when searching through objects. It allows you to search objects, but return a list of data objects linked to the object where the profile name matches the passed in profilename. This option also allows you to specify a **range**, which can be a single date or a range of dates in the following syntax:

*   Single date as a Date or a String, e.g. “2021-07-16”
*   Single date using the date active variable service, e.g. $\{date:”yesterday”\}
*   From a date, e.g. from(“2021-01-01”)
*   The last number of days, e.g. last(3)
*   A range of dates using between, e.g. between(“2021-01-01”,$\{date:”yesterday”\})
    

###### condition

The condition option is used to filter the results of the find command. The syntax of the conditions is as follows:
```js
expression (<|<=|>|>=|=|==|!=|like|intersects|within) expression
    | condition (and|or) condition
    | ( condition )
```

####### Examples of conditions

```js
category = "extractors"
timestamp > "2020-11-03T12:23:40"
timestamp > ${date:"today"} and timestamp < ${date:"tomorrow"}
name like "ch"
location within Sphere([ 51.72961, 0.47612 ], 20 / 3963.2)
```

#### Description

The find command is a powerful way of searching a resource for the data you require. It returns a [virtual list](https://doc.opendatadsl.com/docs/odsl/variable/virtuallist) of items that match the specified conditions.

#### Examples

##### Find all

In its simplest form, the find command can be used to list all items from a service, e.g.
```js
// Get a list of all public calendars
calendars = find ${calendar:public}
```

##### Simple filtering
```js
// Get a list of public actions for a specific category
pactions = find ${action:public} where category = "extractors"
```

##### Filtering by date
```js
// Get a list of private audit records for today
records = find ${audit} where timestamp > ${date:"today"} and timestamp < ${date:"tomorrow"}
```

##### A list of data using the object profile option
```js
data = find profile SPOT for yesterday from ${object:public} where dataset == "ECB_FX"
```

##### Various date range queries
```js
// Data in get query
data = ${data:"#ECB_FX.EURZAR:SPOT"} for ${date:"yesterday"}
data = ${data:"#ECB_FX.EURZAR:SPOT"} for yesterday
data = ${data:"#ECB_FX.EURZAR:SPOT"} for last(3)
data = ${data:"#ECB_FX.EURZAR:SPOT"} for from(yesterday)
data = ${data:"#ECB_FX.EURZAR:SPOT"} for between("2021-05-01",yesterday)
```

##### Geospatial
```js
// Define a polygon and search for objects within it
london = Polygon([[51.5386, -0.4956],[51.6445, -0.0753],[51.5205, 0.1753],[51.3479, -0.1163],[51.5386, -0.4956]])
items = find ${object:"TestGeometry"} where location within london
```

###### order by

Sorts the results, e.g.

```js
recent = find ${audit} where service == "object" order by timestamp desc
```

###### project

Returns only the named fields. Put `!` before a field to leave it out instead, e.g.

```js
names = find ${object} where dataset == "ARGUS_DEL" project name, description
```

###### info

Counts the items by the values of a field instead of returning them, like the `_fieldinfo` REST parameter, e.g.

```js
bytype = find info _type from ${object}
```

## function
Create your own user definable function (UDF)

#### Syntax
```js
function name ( ((byref)? param (, (byref)? param)*)? )
  (comment)?
  (statement)*
end
```
#### Description

The function command allows you to create a custom function which can be used in your OpenDataDSL scripts - either in the same script or you can write functions in a script which can then be imported into a script.

The function is called using the name of the function and the parameters passed in the same order as declared.

To return a value from a function, you need to create a variable in the function as the same name as the function.

##### Documentation
Any comments that are added above the function becomes the function description visible when hovering over the function when it is called.

Multi-line comments can also include '@ variables' which can describe parameters and information about the script itself:
* **@category** - when placed in a comment block at the top of the script, this is used as the category for the file which is used in the GUI to provide categorised lists of scripts.
* **@param** - this is used to provide a description of a function parameter

##### Parameter modifiers
By default, parameters are passed 'by value', which means that changing those values in the function does not change the variable that was passed in.
Adding the **byref** parameter modifier means that the value is passed 'by reference' so the actual variable itself is passed into the function.

Example of script comment:
```java
/**
 * @category report
 * Functions for creating reports
 */
```

Example of function comment:
```java
/**
 * Bootstrap an input curve
 * to create a monthly arbitrage-free curve
 * @param input The input curve
 */
```

#### Examples

An example that creates a bootstrapped and shaped curve

```js
/**
 * Create an arbitrage free monthly curve from the input curve and use simple shaping
 * @param input The input curve
 */
function bootstrapAndShape(input)
    boot = bootstrapCurve(input)
    bootstrapAndShape = shape(boot)
end
```

## transform
Used in an ETL process to map data from an input format to output objects

#### Syntax
```js
name = transform inputVariable into (declaredType|object|rows) as varname 
  (comment)? 
  transformOptions 
  transform 
end

transformOptions: (transformCreate)? (transformUnique)? (on error ignore)? (transformIgnore)?

transformCreate: create with varname (, varname)* (transformClear)?
transformUnique: unique var=value
transformIgnore: ignore condition (, condition)*
transformClear: clear varname (, varname)*

transform: (assign|method|transformIf|transformFor|print|comment|NL)*
transformFor: for varname in expression
  transform
next
transformIf: if condition
  transform (elseif condition transform)* 
  (else transform)?
end
```
#### Description

The transform command creates a transformer that can be used to convert an input object usually read from a file or web URL into one or more output objects

##### Explanation of the syntax

*   The **inputVariable** is the input object that you want to transform from    
*   The **into** declaration can be one of:    
    *   A named public or private declaredType        
    *   A generic object        
    *   A rows object, which is effectively rows or properties like a CSV file or spreadsheet        
*   The **as varname** part defines the temporary object variable holding the current value as the input variable is iterated through    
*   The comment on the first row of the transformer is used as the description of the transformer    
*   The options section at the top of the transformer can contain the following:    
    *   **create** \- this defines which column or property name in the input variable which when the value changes defines what creates an output object, e.g. if there is a _name_ property in the input variable which is different for each element that you want to create an object for you would use the option: **create with name**.        
        *   **clear** \- this works with create and allows you to clear some property values in between creating a new object, this means that you don’t get properties repeating from other objects if their value isn’t supplied.            
    *   **unique** \- this defines the unique id for the output variables and enables you to concatenate properties and clean up vairable names etc.        
    *   **on error ignore** \- this allows the transformer to complete ignoring and transformation errors - sometimes this is necessary when formats cause execution errors        
    *   **ignore** \- this allows you to skip elements that match any supplied condition, for example if a proeprty is null which would cause issues you can specify that condition here
        

#### Examples

The following example creates a transformer and then runs the transformer using input data from an xml file on the ECB web site.
```js
// Create the transformer
ECB_FX = transform xml into #ForeignExchange as fx
    create with Cube
    unique id = "ECB\_FX\_EUR" + fx.currency
    SPOT = TimeSeries(xml.Cube.Cube.time, "BUSINESS", fx.rate)
    category = "Foreign Exchange"
    product = "ECB_FX"
    provider = "European Central Bank"
    model = "EUR" + fx.currency
    description = "European Central Bank Euro FX rates EUR/" + fx.currency
    base = "EUR"
    currency = fx.currency
end

// Test
xml=${xml:"https://www.ecb.europa.eu/stats/eurofxref/eurofxref-daily.xml"}
models = ECB_FX.run(xml)
for model in models
    print model.id
next
```

The following example shows some static JSON content being transformed in 2 different ways and also introduces using metadata to decorate the output results.

```js
// The static input JSON
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

// Convert to an object
input = ${json:json}

// Create a type to convert the JSON into
example = type
    name as String()
    description as String()
    value as Scalar()
end

// Create some metadata to decorate the output data
metadata = Object()
metadata.A = "A description"
metadata.B = "B description"

// A transformer to convert the JSON into the example type
tx = transform input into example as x
    create with data
    unique name = x.identity
    on error ignore
    ignore x.identity == "C"
    name = x.identity
    description = metadata.get(x.identity)
    value = x.value
end

// Run the transformer and output the results as JSON
result = tx.run(input)
print json(result)

// Another transformer that outputs the results as rows
rowtx = transform input into rows as x
    create with data
    unique name = x.identity
    on error ignore
    ignore x.identity == "C"
    name = x.identity
    description = metadata.get(x.identity)
    value = x.value
end

// Run the transformer and print out the results and size of the results
rowresult = rowtx.run(input)
print json(rowresult)
print rowresult.size
```

## type
Create a custom data type for your private data

#### Syntax
```js
declaredType = (versioned)? type (extends declaredType)?
  (comment)?
  (typeProperty | typeExpression | typeMethod )*
end

typeProperty: propertyName as type ( (qualifier)? )
  (matches regex)?
  (default defaultValue)?

typeExpression: propertyName as expression

typeMethod: propertyName as function( (param (,param)*)? )
  functionBody
end
```
#### Description

An explanation of the syntax:

*   **declaredType** is the id of the type    
*   The **versioned** option makes all objects of this type versioned    
*   The **extends** option allows you to extend another type, creating a specialisation of that type    
*   The **comment** on the first line after the declaration becomes the description of this type    
*   A property on the type can be one of:    
    *   property        
    *   expression        
    *   method        
*   The type of the property can be one of:    
    *   dimension - This is a special property type that is used by GUI applications, e.g. the web portal to filter objects based on a value of this property. A dimension is generally used as a property that has a very low uniqueness, e.g. like country or source        
    *   string - A simple string property        
    *   number - A generic number, can be integer or decimal        
    *   scalar - A generic scalar value, can be string, numeric or boolean        
    *   date - A date or datetime value        
    *   boolean - A true/false value        
    *   list - A list of objects - the type of object can be restricted using a qualifier        
    *   object - A generic object        
    *   duration - A timespan value, e.g. 4 days        
    *   geometry - A geometric shape        
    *   timeseries - A timeseries value        
    *   curve - A curve value        
    *   specific type - Any public or private type declaration        
*   The property can have a **qualifier** used to limit the type, e.g. if the type is a list and you want to restrict the list elements to dates
    

##### Versioning

If you specify the **versioned** option when defining the type, then all objects of this type will be versioned. This means any updates to that object where a value of a property has changed will cause a new version to be created and the existing version is archived. More details about data versioning can be found [here](https://doc.opendatadsl.com/docs/odsl/dm/versioning).

##### Extending

By default, all types extend the #Object type which has the following properties defined:

*   name - string    
*   description - string    
*   classification - dimension    
*   geolocation - geometry
    

However, you can elect to extend any public or your own private type in order to create a specialised version of that type, which will give you a type hierarchy. A simple example of why you would want to do this is shown below:

##### Type Hierarchy

The diagram below shows a simple type hierarchy, starting with #Object we create a generic Person type which adds the properties PhoneNumber and Email. We then extend the Person object into 2 specialised types: Student and Professor which add some more specific properties.

With this type hierarchy, we can add Student objects and Professor objects, but we can also list all Person objects showing all students and professors, so if I knew someones email address and seached in Person, it would find the person irrespective of whether they are a student or a professor.

For more examples and in-depth descriptions, see the section [Data Modelling](https://doc.opendatadsl.com/docs/odsl/dm/modelling).

#### Examples

Example of a type hierarchy:
```js
Widget = versioned type
    price as Number()
end

Cable = versioned type extends Widget
    length as Number()
end

Switch = versioned type extends Widget
    ports as Number()
end
```

## workflow
Create a workflow

#### Syntax
```js
varname = workflow IN "category"
  (comment)? 
  (actionInput|actionOutput)* 
  (actionExit)?
  workflowStart (phase|workflowEvent|NL|comment)*
end

workflowStart: WF_START
  workflowTransition
end
workflowBody: (workflowEvent|workflowAction|workflowGateway|workflowWorkflow)
workflowEvent: WF_EVENT varname AS "string"
  (assign|comment|returnstmt|workflowTransition)* 
end
workflowAction: WF_ACTION varname input
  (assign|comment|workflowTransition)* 
end
workflowGateway: WF_GATEWAY varname input
  (assign|comment|workflowTransition)* 
end
workflowWorkflow: WF_WORKFLOW varname input
  (assign|comment|workflowTransition)* 
end
workflowTransition: "name" -> connection

// Workflow phases
phase: phase "name" (retries INT)? (delay INT TIMEUNIT)? (then reschedule)? (external)?
  (statement|workflowBody)* 
end
reschedule: reschedule INT TIMEUNIT
fail: fail expression
abort: abort expression
```
#### Description

The configuration of a workflow is best done in the web portal with the [Workflow Manager](https://doc.opendatadsl.com/docs/extensions/workflow-manager/overview), which lets you draw a workflow as a diagram and writes this code for you, but it can also be done in OpenDataDSL code.

##### Anatomy of a workflow

A workflow has some input, output and exit configuration at the start - just like an action. The input information is passed in via a process or as an object if running the workflow manually.

###### Workflow Blocks

All workflows have building blocks in them:

*   WF_START - There must be exactly 1 of these, which indicates the start point and it only contains a transition which is the first workflow element that is called    
*   WF_ACTION - This is a block which configures a workflow [action](https://doc.opendatadsl.com/docs/odsl/variable/action). You can:    
    *   define the action transition routing, i.e. the route to take given the transition information when the action completes        
    *   assign the action input variables from the workflow input or any previous action outputs        
    *   run the action using the input variables        
*   WF_GATEWAY - This block configures a workflow gateway. It is configured in the same way as an action block and is generally used to route workflows according to an expression    
*   WF_WORKFLOW - This block configured a sub-workflow, it is configured in the same way as an action - NOTE than any workflow can be used as a sub-workflow    
*   WF_EVENT - This is generally used as an end point of a workflow and is used to return the transition information back to the calling application.
    

###### Workflow Phase

A workflow can nest its WF_ACTION, WF_GATEWAY and WF_WORKFLOW blocks in a phase.

It is recommended that you use phases in a workflow for the following reasons:

*   It breaks a workflow into distinct sections which get reported in real-time whilst the workflow is executing    
*   It allows you to time sections of the workflow    
*   Each phase allows for custom configuration of retries, retry delay and rescheduling
    

#### Examples

```js
wf_xml_extract = workflow in "data-loaders"
    // Extract some data
    in url as Scalar
    exit "success", "failed"
    
    WF_START
        "ok" -> act_extract_xml
    end

    phase "EXTRACT"
        WF_ACTION act_extract_xml ai
            "ok" -> stopok
            "failed" -> stopfailed
            ai.url = input.url
            result = ${action:"#extract_xml"}.run(ai, output)    
        end
    end

    WF_EVENT stopok as "success"
        return "ok"
    end
    WF_EVENT stopfailed as "failed"
       return "failed"
    end
end
```

As a line-by-line breakdown of this workflow

1. Define a workflow called wf_xml_extract in the category data-loaders
2. Set a description for this workflow
3. Define an input variable called url which is a [Scalar](https://doc.opendatadsl.com/docs/odsl/variable/scalar)
4. Define exit transitions for the workflow as success and failed
6. Define the workflow start point
7. Transition to the action named act\_extract\_xml (the transition name is ignored)
10. Define a workflow phase called EXTRACT
11. Define an action block called act\_extract\_xml with an input variable called ai
12. Route the “ok” transition to stopok
13. Route the “failed” transition to stopfailed
14. Set the url on the action input to be the input url (passed in by the [process](https://doc.opendatadsl.com/docs/odsl/dm/workflow))
15. Run the #extract_xml action passing in the ai variable and the global output variable
19. Define the stopok event as a success transition for the whole workflow
22. Define the stopfailed event as a failed transition for the whole workflow

## check
Runs the quality checks of an object

#### Syntax
```js
(results =)? check object (save)?

(results =)? check object (save)?
    profile for checkname (parameter (, parameter)*)?
end
```

#### Description

Runs the quality check constraints declared on the object's [type](https://doc.opendatadsl.com/docs/odsl/variable/declaredtype) and returns the results. With **save**, the object is saved with the results. In the second form, you list the checks to run on the object's data instead: each line names a data property (profile), the check to run and its parameters.

## extract
Extracts data from a web page

#### Syntax
```js
varname = extract page (, selector)? into (object|rows) as name
    (fill selector with value)*
    (param selector with value)*
    (select selector)*
    (check selector | uncheck selector)*
    (click selector)*
    (assign)*
end
```

#### Description

Reads a web page, usually a dynamic page from the [DHTML service](https://doc.opendatadsl.com/docs/odsl/service/dhtml), and extracts data from it into an object or a list of rows. Before the data is read, you can interact with the page: fill in fields, set parameters, select options, tick or untick boxes and click buttons, each found with a CSS selector. The assignments then set the properties of the extracted data.

## index
Defines a database index

#### Syntax
```js
varname = index
    (unique|sparse|hidden|ttl seconds|partial condition|collation name=value (, name=value)*)*
    field (asc|desc|text|2dsphere)
    (field (asc|desc|text|2dsphere))*
end
```

#### Description

Creates an [Index](https://doc.opendatadsl.com/docs/odsl/variable/index) variable named after the variable, which you then save with the index service. Indexes make queries on the indexed fields faster.

|**Option**|**Description**|
|-|-|
|unique|No two items can have the same values of the indexed fields|
|sparse|Only index the items that have the indexed fields|
|hidden|Keep the index but don't use it for queries|
|ttl seconds|Delete items automatically this many seconds after the date in the indexed field|
|partial condition|Only index the items that match the condition|
|collation|The collation of the index, e.g. locale="en", strength=2|

#### Examples
```js
region_type = index
    region asc
    _type asc
end
save ${index:region_type}
```

## report
Creates a report configuration from a report creator

#### Syntax
```js
varname = report with creator
    (property = value)*
end
```

#### Description

Creates a [report configuration](https://doc.opendatadsl.com/docs/odsl/variable/report) from a [report creator](https://doc.opendatadsl.com/docs/odsl/variable/reportcreator), given as a variable or by its id. Set the report's properties, such as name, description, category and script, and the values of the creator's parameters.

#### Examples
```js
summary = report with "PRICE_SUMMARY"
    name = "TTF price summary"
    category = "Gas"
end
save ${reportconfig:summary}
```


---

<!-- source: product/developer/odsl/Commands/Expressions.md -->

Expressions
===========================

Expressions usage within the OpenDataDSL language

## Syntax

Expressions can take various forms, so here is a list of the syntax for expressions:
```js
// A simple variable (either existence or true)
variable

// Not variable (either non-existence or false)
not variable

// Use of braces for explicit precedence
( expression )

// Array operators
[ expression (, expression)* ]

// Mathematical Operators
expression * expression
expression / expression
expression + expression
expression - expression
expression ^ expression

// Logical Operators
expression == expression
expression = expression
expression != expression
expression < expression
expression <= expression
expression > expression
expression >= expression

// Geospatial and Special
expression like expression
expression intersects expression
expression within expression

// Filter creation (usually used with event time series or event curves)
? condition
```

### Single line if

You can choose between two values with a condition in an assignment:

```js
label = price > 100 ? "high" : "normal"
```

## Use within the language

Expressions are used in various places within the ODSL language as shown in the following table:

|**Command**|**Description**|
|-|-|
|Assignment|When assigning a value to a variable|
|Printing|When printing a message to the terminal|
|Logging|When logging a message|
|Return|When returning a value from a script|
|Fail and Abort|Creating a message when instructing a workflow to fail or abort|
|Type declaration|Adding an expression as a property on a type|
|Extractor|Used as a selector, check, click etc. in an extractor|
|For|Used in defining a for statement|
|Active variable|Used as a name and options for an active variable|
|Method and function calls|Used as a parameter in method and function calls|

## Examples

### Assignment examples
```js
result = a + 1
```

#### Creating a filter on an event series
```js
event_series.filter = ? side = "ASK" and relative = "M01"
```

### Printing example
```js
print “Hello “ + name
```

### Logging example
```js
log info “Value is “ + value
```

### Return example
```js
return a + 1
```

### Fail example
```js
fail “Value is “ + value
```

### Type declaration example
```js
name as first + “ “ + last
```

### For example
```js
for i=a+10 to a+20
next

for i in [1,2,3]
next
```

### Methods and function calls
```js
l.addAll([1,2,3])
```


---

<!-- source: product/developer/odsl/Commands/Conditions.md -->

Conditions
==========================

The usage of conditions in the OpenDataDSL Language

## Syntax

The syntax for a condition:
```JS
variable
    | '!' variable
    | functioncall
    | expression ('<'|'<='|'>'|'>='|'='|'=='|'!='|'in'|'nin'|'like'|'matches'|'intersects'|'within') expression
    | condition ('and'|'or') condition
    | '(' condition ')'
```

|**Operator**|**Meaning**|
|-|-|
|in, nin|The value is (or is not) in a list|
|like|The value matches a regular expression, ignoring case|
|matches|The value matches a regular expression, with case|
|intersects, within|Geospatial: the shape intersects, or is within, another shape|

## Use within the language

Conditions can be used at various places within the OpenDataDSL language as shown in the following table:

|**Command**|**Description**|
|-|-|
|Aggregate|Used in a match directive in an [aggregate](https://doc.opendatadsl.com/docs/odsl/command/aggregate) command|
|Find|Used as a condition in a [find](https://doc.opendatadsl.com/docs/odsl/command/find) command, searching for something|
|Transform|Used within a [transform command](https://doc.opendatadsl.com/docs/odsl/command/transform) in both ‘if’ statements and ‘ignore’ directives|
|If|Used in the [if](https://doc.opendatadsl.com/docs/odsl/command/if) control statement|
|While|Used in the [while](https://doc.opendatadsl.com/docs/odsl/command/while) control statement|

## Examples

### Aggregation Example
```js 
summary = aggregate ${exec}
    match service="ETL"
    group _id="$status", qty=count()
    sort qty desc
end
```

### Find Example
```js
records = find ${audit} where 
    timestamp > ${date:"today"} 
    and timestamp < ${date:"tomorrow"}
end
```
### Transform Example

```js
ignoe contains(ag.column\[0\], "Date")
unit="MT"
if contains(input.product, "Potato")
  unit="100KGS"
end 
```

### If Example
```js
if a==1 and b==2
    print 1
elseif (a==2 or b==2)
    print 2
elseif (a>2 and b==3) or c
    print 3
end
```

### While Example
```js
X=1
while X<5
   X=X+1
end
```
