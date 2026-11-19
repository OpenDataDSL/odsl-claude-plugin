# Calendars, period codes and timeseries ranges

Calendar types and how to use them.

Generated from the OpenDataDSL documentation (https://doc.opendatadsl.com/docs).


---

<!-- source: product/developer/Calendars/Business-Calendar.md -->

Business Calendar
=================

A business calendar is a basic holiday calendar that represents a standard working week with every day from Monday to Friday

## Introduction

A business calendar represents the basic working week for most of the western world, with every calendar day except Saturday and Sunday. It is actually a holiday calendar with the following 2 holiday rules pre-applied:

*   Every Saturday    
*   Every Sunday


---

<!-- source: product/developer/Calendars/Calendars.md -->

Calendars
=========

A detailed explanation of all the supported calendars in OpenDataDSL

## Introduction

The calendars in OpenDataDSL are one of the fundamental components necessary for a smart data management platform. 
They provide time-based intelligence for understanding when things should happen and enable you to highlight issues when things don’t happen. 
The key to the way calendars work in ODSL is in the flexibility and ease of use, this section describes all the built-in calendar types and how you can customise them to suit your needs.

## Contents

### Daily Calendar
A daily calendar is a simple calendar that represents every calendar day

### Holiday Calendar
A holiday or trading calendar is a daily calendar with rules that determine which days are non-working or holiday days

### Business Calendar
A business calendar is a basic holiday calendar that represents a standard working week with every day from Monday to Friday

### Intraday Calendar
An intraday calendar represents all regular calendars that have time intervals of less than a whole day, e.g. Hourly

### Sparse Calendar
A sparse calendar is used when there is no pre-defined period between intervals, each index can represent any point in time

### Expiry Calendar
An expiry calendar is used to calculate the last trading date and the conversion of absolute and relative futures contracts

### Period Code
A period code represents an absolute or relative delivery period for some commodity or financial future

### Custom Calendar Examples
A set of examples that showcase the flexibility of the calendars in OpenDataDSL

### Timeseries Scaling
In-depth information about how to rescale TimeSeries

### Timeseries Ranges
In-depth information about how to use time ranges with Timeseries


---

<!-- source: product/developer/Calendars/Custom-Calendar-Examples.md -->

Custom Calendar Examples
========================

A set of examples that showcase the flexibility of the calendars in OpenDataDSL

## Only Weekends

This calendar is the opposite of a business calendar where only the weekends are included days

```js
// Create a weekend only calendar
WEEKEND = HolidayCalendar()
WEEKEND.name = "Weekends only"
WEEKEND.addRule("Every Monday")
WEEKEND.addRule("Every Tuesday")
WEEKEND.addRule("Every Wednesday")
WEEKEND.addRule("Every Thursday")
WEEKEND.addRule("Every Friday")
save ${calendar:WEEKEND}
```

## Half-Hourly Weekend Only

This example builds on the example above by breaking the weekend only into 1/2 hourly intervals

```js
// Create a Half Hourly, weekend only calendar
HHWEEKEND = IntradayCalendar(30m, WEEKEND)
HHWEEKEND.name = "Half-hourly calendar on weekends only" 
save ${calendar:HHWEEKEND}
```


---

<!-- source: product/developer/Calendars/Daily-Calendar.md -->

Daily Calendar
==============

A daily calendar is a simple calendar that represents every calendar day

## Introduction

This daily calendar is the base calendar for every calendar that handles observations from 1 day up to a month. This includes:

*   Business Calendar
    
*   Holiday Calendar
    

## Creating a daily calendar

To create a daily calendar use the DailyCalendar function as shown below:

```js
// Create a daily calendar
daily = DailyCalendar()
```

For a description of the properties and methods on a calendar see [here](https://doc.opendatadsl.com/docs/odsl/variable/calendar)


---

<!-- source: product/developer/Calendars/Expiry-Calendar.md -->

Expiry Calendar
===============

An expiry calendar is used to calculate the last trading date, delivery dates and the conversion of absolute and relative futures contracts

## Introduction

The expiry calendar is used with forward curves to calculate the last trading day and the delivery period of a financially traded [contract](https://doc.opendatadsl.com/docs/odsl/variable/contract). It uses expiry rules for date calculation which can be different for specific period code types.

## Expiry Rules

An expiry rule is a set of instructions in English that are used to determine a date from a period code. There are a number of pre-defined words that can be used to describe the actions to take and these are described in detail below.

Each rule starts with the word **go** and the starting point for each calculation is the start of delivery for the period code the calculation is for.

### Expiry rule syntax

The basic syntax for an expiry rule is as follows:

```js
expiry: move (move|align)* (exception THEN (move|align)*)? ("for" periodcode)? (timelimit)?

move: (moverel|moveabs)
```

### Relative movement

The syntax for a relative movement is as follows:

```js
("then") "go" ("back"|"forward") t TIME ("using calendar")

TIME: ('day'|'days'|'week'|'weeks'|'month'|'months'|'quarter'|'quarters'|'season'|'seasons'|'year'|'years')
```

An example instruction to move back 1 day from the start of delivery using the trading calendar:

```js
go back 1 day using calendar
```

### Absolute movement

The syntax for an absolute movement is as follows:

```js
("then") "go to the" ("beginning"|"end"|number suffix|lastday|"penultimate") "of the" ("previous"|"current"|"next")? TIME
```

#### lastday rule

```js
last ([Mm]'onday'|[Tt]'uesday'|[Ww]'ednesday'|[Tt]'hursday'|[Ff]'riday'|[Ss]'aturday'|[Ss]'unday'|trading day)
```

:::note
'trading day' can only be used for delivery rules, e.g.

```
delivery starts on last trading day of the current month then go forward 2 days then align forwards
```

:::

An example instruction to go to the end of the previous month:

```js
go to the end of the previous month
```

### Alignment

After each movement, you can optionally align the current calculated date using the following syntax:

```js
("then")? ("align"|"align forwards")
```

An example instruction to go to the end of the previous month and then align to the trading calendar:

```js
go to the end of the previous month then align
```

### Rules for specific period codes

You can specify a period code type in the rule that will specify to use this rule specifically for those types of period codes, e.g

```js
go to the end of the previous month then align for quarters
```

### Time limited rules

You can specify that a rule is applicable after or before a certain point in time

```js
go to the end of the previous month then align for quarters after 2021-01-01
```

### Exceptions and complex rules

Sometimes, the trading rules for a calendar include some complex rules that determine when trading of a certain product ceases. These can be mostly handled using an **except if** or an **unless** clause in the expiry rule. 
The except clause syntax is as follows

```js
except if (exception information) then
```

The unless clause syntax is as follows

```js
unless (exception information) then
```

The difference between an **except if** and an **unless** clause is the starting date that the exception clause uses:

##### Except if
The date tested in the exception clause is the calculated date after any preceeding clauses have been executed.

##### Unless
The date tested in the exception clause is the original start-of-delivery date, therefore ignoring any preceeding clauses.
If the **unless exception** is true, then the date is reset to the original start-of-delivery date.

##### Example of except if versus unless

Consider the following 2 rules:

**Unless Rule**

go back 1 days using calendar *unless* there is a non-business day within 1 day before then go to the end of the current day using calendar

** Except if Rule**

go back 1 days using calendar *except if* there is a non-business day within 1 day before then go to the end of the current day using calendar

Using a contract with a start date of Monday 23rd October 2023
* The unless clause will create an expiry date of the 23rd October 2023. 
* The except if clause will create an expiry date of the 20th October 2023.

Using a contract with a start date of Tuesday 24th October 2023
* The unless clause will create an expiry date of the 23rd October 2023.
* The except if clause will create an expiry date of the 23rd October 2023.

#### Proximity to a holiday date

The exception information can involve proximity to a holiday day:

```js
there is a holiday (within)? t time (before|after)
```

This checks to see if there is a holiday day either within or specifically on a number of days/weeks before or after.

e.g.

```js
... except if there is a holiday within 7 days after then ...
```

#### Proximity to a non-business date

The exception information can involve proximity to a non-business day:

```js
there is a non-business day (within)? t time (before|after)
```

This checks to see if there is a non-business day either within or specifically on a number of days/weeks before or after.

e.g.

```js
... except if there is a non-business day within 7 days after then ...
```
#### The week/month/quarter/season/year ends on a specific day

You can make an exception for when the period ends on a specific day:

```js
the last day of the (week|month|quarter|season|year) is a DOW
```

e.g.

```js
... except if the last day of the month is a Monday then ...
```

#### The week/month/quarter/season/year starts on a specific day

You can make an exception for when the period starts on a specific day:

```js
the first day of the (week|month|quarter|season|year) is a DOW
```

e.g.

```js
... except if the first day of the month is a Monday then ...
```

## Delivery Rules
You can override the logic for calculating the start and end of delivery for specific period codes using the delivery rule syntax:

```js
delivery (starts|ends) (on)? (at)? (the)? moveperiod (move|align)* (for periodcode)? (timelimit)?;

moveperiod: ("beginning"|"end"|date suffix) "of the" ("previous"|"current"|"next")? TIME
```

An example for ICE Low Sulphur Gasoil Futures where the start of delivery is always the 16th of the delivery month:
```js
delivery starts on the 16th of the current month
```

Delivery rules use the delivery calendar if provided, otherwise they use the trading calendar.

## Creating an expiry calendar

To create an expiry calendar and add your specific expiry rules to it, use the ExpiryCalendar function with a [daily](https://doc.opendatadsl.com/docs/odsl/calendar/daily), [business](https://doc.opendatadsl.com/docs/odsl/calendar/business) or [holiday](https://doc.opendatadsl.com/docs/odsl/calendar/holiday) calendar as the trading calendar.

The trading calendar is used to calculate the expiry and delivery dates for any period code.

```js
eombus = ExpiryCalendar(BusinessCalendar())
eombus.addRule("go to the end of the previous month then align")
```

The example above creates an expiry calendar using a business calendar where the expiries for all period codes is the last business day of the month prior to delivery.

You can also specify a separate delivery calendar which will be used to calculate the delivery start and end dates.

```js
eombus = ExpiryCalendar(BusinessCalendar(), "#HENG")
```

## Saving and re-using your calendar

Once you have created your expiry calendar, you can save it to the expiry calendar service using the save command on the [expiry service](https://doc.opendatadsl.com/docs/odsl/service/expiry) as follows:

```js
save ${expiry:my_calendar}
```

Now, whenever you need to use your calendar in a script, you can simply use the active variable which will read your calendar from the expiry calendar service as follows:

```js
cal = ${expiry:"my_calendar"}
```


---

<!-- source: product/developer/Calendars/Holiday-Calendar.md -->

Holiday Calendar
================

A holiday or trading calendar is a daily calendar with rules that determine which days are non-working or holiday days

## Introduction

The holiday calendar is a daily calendar with holiday rules.

## Holiday Rules

A holiday rule is an algorithm that determines if a specific day is a holiday. There are a number of built in standard algorithms which cater for all possible holiday dates. All rules can be written in plain English following a specific syntax for each rule.

### Actual Rule

The **actual** rule defines a specific absolute day of a specific month like the 5th of May.

#### Syntax
```
DOM (OF)? MONTHNAME (named 'rule name')? (OPTION)*
```
#### Examples
```js
5th of May named 'Labour Day'
8th of may named 'VE Day' from 2020 to 2020
25th of december named 'Christmas Day' on saturday shift 3 days and on sunday shift 2 days
```
### Last Rule

The **last** rule represents the last specific named day of a specific month.

#### Syntax
```
last DOW in MONTHNAME (named 'rule name')? (OPTION)*
```
#### Examples
```js
last monday in may named 'Spring Bank Holiday'
last monday in august named 'Summer Bank Holiday'
```
### Every Rule

The **every** rule represents every named day line every Saturday

#### Syntax
```
every DOW (OPTION)*
```
#### Examples
```js
every Saturday
every Sunday
```
### Ordinal Rule

The **ordinal** rule is defined by the 1st, 2nd, 3rd or 4th specific named day of a specific month

#### Syntax
```
(FIRST|SECOND|THIRD|FOURTH) DOW IN MONTHNAME (named 'rule name')? (OPTION)*
```
#### Examples
```js
first monday in may named 'Early May' to 2019
1st of January named 'New Years Day' on saturday shift 2 days and on sunday shift 1 day
```
### Named Rule

The **named** rule represents a set of hard coded holidays such as Easter

:::note
Options are also supported on named rules
::: 

#### Syntax
```
(
Easter Sunday|
Easter Monday|
Good Friday|
Ascension Day|
Corpus Christi|
Pentecost|
Victoria Day|
Whit Monday
)
```

### Supported Options

Some of the holiday rules support options which you can use to customise or add offsets, here is a list of the supported options in those rules:

|**Option**|**Description**|**Example**|
|-|-|-|
|on Saturday shift|If the holiday falls on a Saturday, shift it by x days|on Saturday shift 2 days|
|on Sunday shift|If the holiday falls on a Sunday, shift it by x days|on Sunday shift 1 day|
|on Monday shift|If the holiday falls on a Monday, shift it by x days|on Monday shift 1 day|
|from|Specifies that the holiday is valid from the specified year|from 2012|
|to|Specifies that the holiday is valid to the specified year|to 2020|
|repeating every|The holiday normally repeats every year, but specifying this enables you to only repeat a holiday every x years|repeating every 5 years|
|starting|Along with repeating, starting states when the repeating starts|starting 2012|

## Creating a calendar with your own rules

To create a new holiday calendar, use the constructor for a holiday calendar and add your rules. An example is shown below:

```js
my_calendar = HolidayCalendar()
my_calendar.addRule("Every Saturday")
my_calendar.addRule("Every Sunday")
my_calendar.addRule("Good Friday")
my_calendar.addRule("Easter Monday")
my_calendar.addRule("First Monday in May named 'Early May'")
my_calendar.addRule("Last Monday in May named 'Spring Bank Holiday'")
my_calendar.addRule("Last Monday in August named 'Summer Bank Holiday'")
my_calendar.addRule("25th of december named 'Christmas Day' on saturday shift 3 days and on sunday shift 2 days")
my_calendar.addRule("26th of december named 'Boxing Day' on saturday shift 2 days and on sunday shift 1 day")
```

## Saving and re-using your calendar

Once you have created your holiday calendar, you can save it to the calendar service using the save command on the calendar active variable service as follows:

```js
save ${calendar:my_calendar}
```

Now, whenever you need to use your calendar in a script, you can simply use the active variable which will read your calendar from the calendar service as follows:

```js
cal = ${calendar:"my_calendar"}
```


---

<!-- source: product/developer/Calendars/Intraday-Calendar.md -->

Intraday Calendar
=================

An intraday calendar represents all regular calendars that have time intervals of less than a whole day, e.g. Hourly

## Introduction

The intraday calendar is used for medium frequency data (between 1 second and 12 hour intervals) and has the following features:

*   Observations are stored at regular intervals between 1 seconds and 12 hours using a [Duration](https://doc.opendatadsl.com/docs/odsl/variable/Duration)    
*   A Holiday Calendar can be applied which determines the days when observations are recorded    
*   You can exclude specific hours of the day, essentially limiting the range of intraday observations
    

## Creating an Intraday Calendar

There are 3 constructors for an Intraday Calendar:

```js
cal = IntradayCalendar(Duration)
cal = IntradayCalendar(Duration, HolidayCalendar)
cal = IntradayCalendar(Duration, HolidayCalendar, excludeHours)
```

Here are some examples of creating Intraday Calendars:

```js
// Create an hourly calendar
hourly = IntradayCalendar(1h)

// Create a 15 minutely calendar
fifteenmin = IntradayCalendar(15min)

/** 
 Create an hourly calendar using a holiday calendar and 
 exclude hours 0-6 and 20-23
**/
ukpeak = IntradayCalendar(1h, "HENG", [0,1,2,3,4,5,6,20,21,22,23])
ukpeak.name = "UK Electricity Peak Hours"
```


---

<!-- source: product/developer/Calendars/Period-Code.md -->

<PeriodCode />


---

<!-- source: product/developer/Calendars/Sparse-Calendar.md -->

Sparse Calendar
===============

A sparse calendar is used when there is no pre-defined period between intervals, each index can represent any point in time

## Introduction

A sparse calendar is a useful calendar for the recording of non-regular time spaced observations and there is no requirement for checking for missing values in a TimeSeries.

A sparse calendar is only really useful when used in conjunction with a TimeSeries.

## Creating a sparse timeseries

### Simple constructor

To create a TimeSeries with a sparse calendar, you can use the TimeSeries constructor with a calendar as shown below:

```js
// Creates an empty TimeSeries with the sparse calendar
ts = TimeSeries("SPARSE")
```

### Single value constructor

You can also create a sparse TimeSeries with a single observation as follows:

```js
// Creates a TimeSeries with a single observation
ts = TimeSeries("2020-10-01", "SPARSE", 12.5)
```

### Data type constructor

You can also specify a data type for the TimeSeries which ensures only values of that data type are accepted into the observations (See [TimeSeries](https://opendatadsl.atlassian.net/wiki/spaces/DOCUMENTAT/pages/2785484/TimeSeries) for more details):

```js
// Creates a TimeSeries with a single observation and specifies a datatype
ts = TimeSeries("2020-10-01", "SPARSE", 12.5, "Numeric")
```

### Multiple value constructor

You can also create a TimeSeries with multiple observations as shown below:

```js
// Creates a TimeSeries with multiple observations
ts = TimeSeries(["2020-11-21T08:40:12","2020-09-11T19:27:22"], "SPARSE", [12.5,15.5])
```

:::note
You don’t need to add the observations in date/time order, the TimeSeries will sort them.
:::

## Adding observations

You can add observations to a TimeSeries with a sparse calendar in the same way as any other calendar, e.g.

```js
// Add some values at random timepoints
ts.add("2020-10-01", 12.5)
ts.add("2020-11-21", 15.5)
ts.add("2020-09-11", 18.5)
```


---

<!-- source: product/developer/Calendars/TimeSeries-Scaling.md -->

This guide explains how to rescale [timeseries](https://doc.opendatadsl.com/docs/odsl/topics/timeseries) to a different calendar.

## Introduction
Rescaling timeseries is the process of changing the changing the calendar of the TimeSeries and fitting the data using that new calendar.

Rescaling from a high frequency calendar to a low frequency calendar, e.g. Daily to Monthly is called `aggregation`

Rescaling from a low frequency calendar to a high frequency calendar, e.g. Monthly to Daily is called `distribution`

## Aggregation
When you rescale data from a high frequency calendar to a low frequency calendar, we need to reduce or aggregate the data.
The way the data is aggregated is determined by the `observed` setting.

### Observed setting

#### beginning
An observation made at the beginning of the period

#### end
An observation made at the end of the period.

This is typically used for end of day settlement prices where the settlement value is either the last traded price or a calculation using the last tradded prices.

#### averaged
An average of the values observed throughout the period

#### summed
A sum of the values observed throughout the period

This is typically used for volumes, e.g. the number of traded items in a day, so aggregating to a lower frequency will sum up these values to give you a total volume for the period.

#### high
The highest value observed throuhout the period

#### low
The lowest value observed throughout the period

#### delta
The change of value from the start to the end of the period.

This is typically used for metered data where the values are meter readings which always increase in value.
Scaling using delta allows you to see the amount used per period.  

#### none
A point-in-time observation

### Implicit observed
When you scale a TimeSeries without specifying the observed option, OpenDataDSL first checks the global observed setting which is set using:

```js
set observed value
```

If this hasn't been set, it uses the observed attribute of the TimeSeries which defaults to `end`.

## Distribution
When you rescale data from a low frequency calendar to a high frequency calendar, we need to distribute the observed value across a range of values.
The way the data is distributed is determined by the `distribution` setting.

### Distribution setting

#### constant
The value is constant across the whole period.

This can be seen like a bar chart where the tops of the bars are flat and the value jumps at each source period start.

#### linear 
The values are interpolated using a linear spline between the source values.

This can be seen as a simple line chart where the values are points on the line

#### cubic
The values are interpolated using a cubic spline between the source values

This can be seen as a smoothed line chart.

#### none
A single value is used at the source index, the rest of the values are filled in with NaN or Missing

### Implicit distribution
When you scale a TimeSeries without specifying the distribution method, OpenDataDSL first checks the global distribution setting which is set using:

```js
set distribution value
```

If this hasn't been set then `constant` distribution is used.

### Distribution value
The value used to calculate the distributed values is the observed value in the source TimeSeries except for the following cases:

#### Summed observed
If the TimeSeries is observed as `summed` then the distributed values are divided by the number of observations so that the sum of those observations matches the original observed value.

## Scaling
To rescale a TimeSeries, you using the `scale` function which has 3 signatures:
```js
ts = scale(input, calendar)
ts = scale(input, calendar, observed)
ts = scale(input, calendar, observed, distribution)
```

### Examples

#### Monthly to Daily TimeSeries

```js
// Create a monthly TimeSeries
mts = TimeSeries("2021-01-01", MonthlyCalendar(), [1,3,7,5,4,5])

// Scale to daily using a cubic spline
cts = scale(mts, DailyCalendar())
print cts.values
```

#### Monthly to Daily TimeSeries - cubic spline

```js
// Create a monthly TimeSeries
mts = TimeSeries("2021-01-01", MonthlyCalendar(), [1,3,7,5,4,5])

// Scale to daily using a cubic spline
cts = scale(mts, DailyCalendar(), "beginning", "cubic")
print cts.values
```

## Auto-scaling
By default, when the number of observations requested for a timeseries exceed a threshold, the timeseries is autoscaled according to the default observed setting on the timeseries.

The autoscaling thresholds depend on where the request comes from as shown in the table below:

|Request Source|Observation Threshold|
|-|-|
|REST API and sdk's|25,000|
|Excel Add-in|25,000|
|ODSL in VS Code|25,000|
|Web Portal|5,000|
|Mobile Application|5,000|

### Detection
To determine if a timeseries has been autoscaled, there is a ```scaling``` property on the timeseries, if this is anything but the value ```0``` then it has been autoscaled.

### Overriding
In order to tell the server not to autoscale a timeseries, you need to:

|Request Source|Overriding|
|-|-|
|REST API|Add query parameter ```_scaling=0```|
|Excel Add-in|Uncheck the option for autoscale|
|ODSL in VS Code|Use the command ```set autoscale off``` or add the option ```_scaling=0``` to your data request|
|Web Portal|Check the scaling box and move the slider to the left|


---

<!-- source: product/developer/Calendars/daily-offset-calendar.md -->

A daily offset calendar is a simple calendar that represents every calendar day but with a predetermined start point of the day

## Introduction

Standard daily calendars always start at 00:00:00, this calendar allows you to provide an hour of the day which represents the start time of the day.

## Creating a daily offset calendar

To create a daily offsey calendar use the OffsetDailyCalendar function as shown below:

```js
cal = OffsetDailyCalendar(timezone, dayoffset, hour)
```

## Examples

### Constructing

A UK Power calendar which starts at 23:00 on the previous day using the timezone 'Europe/London'
```js
UKPOWER = OffsetDailyCalendar("Europe/London", -1, 23)
```

A European GASDAY calendar which starts at 06:00 on the previous day using the timezone 'Europe/Paris'
```js
GASDAY = OffsetDailyCalendar("Europe/Paris", 0, 6)
```

### Scaling

Scaling a half-hourly series to average daily with the start of the day at 23:00 on the previous day

```js
UKPOWER = OffsetDailyCalendar("Europe/London", -1, 23)
e = ${data:"EPEX.GB.CONT.ID.RPDHH.HALFHOURLY:OFFPEAK_VOL"}
s = scale(e, UKPOWER, 'averaged')
print s.data
```

## Further Information
* Properties and methods on a [calendar](https://doc.opendatadsl.com/docs/odsl/variable/calendar)
* [Timeseries timescaling](https://doc.opendatadsl.com/docs/odsl/calendar/scaling)


---

<!-- source: product/developer/Calendars/timeseries-ranges.md -->

## Introduction

When specifying ranges for [timeseries](https://doc.opendatadsl.com/docs/odsl/topics/timeseries) , curveseries, reports etc. you use the *_range* query parameter.

This can have one of the following values:

* from(date)
* from(date, tz)
* last(number)
* between(date, date)
* between(date, date, tz)
* date or within(date)
* within(date, tz)

### From function

The **from** function produces a date range that is from the specified date to the end of the timeseries you are requesting.

#### Example:

**OpenDataDSL:**

```js
ts = ${data:"DR_EV_TEST:HOURLY"} for from("2024-12-20")
```

**REST API:**

```html
GET https://api.opendatadsl.com/api/data/v1/private/DR_EV_TEST:HOURLY?_range=from(2024-12-20)
Authorization: Bearer {{token}}
```

### Last function

The **last** function doesn't use a date range, instead it returns the last n observations from the timeseries

#### Example:

**OpenDataDSL:**

```js
ts = ${data:"DR_TEST:DAILY"} for last(10)
```

**REST API:**

```html
GET https://api.opendatadsl.com/api/data/v1/private/DR_EV_TEST:DAILY?_range=last(10)
Authorization: Bearer {{token}}
```

### Between function

The **between** function creates a bounded date range where the first date is start of the range and the second date is the end of the range.

#### Example:

**OpenDataDSL:**

```js
ts = ${data:"DR_TEST:DAILY"} for between("2024-12","2024-12")
```

**REST API:**

```html
GET https://api.opendatadsl.com/api/data/v1/private/DR_EV_TEST:DAILY?_range=between(2024-12,2024-12)
Authorization: Bearer {{token}}
```

### Within function

The **within** function produces a date range covering the start and end of the date period provided.
Note, it can also be used as just the date period without the within() function.

#### Example:

**OpenDataDSL:**

```js
ts1 = ${data:"DR_TEST:HOURLY"} for within("2024-12-01")
ts2 = ${data:"DR_TEST:DAILY"} for "2024-12"
```

**REST API:**

```html
GET https://api.opendatadsl.com/api/data/v1/private/DR_EV_TEST:HOURLY?_range=within(2024-12-01)
Authorization: Bearer {{token}}

GET https://api.opendatadsl.com/api/data/v1/private/DR_EV_TEST:DAILY?_range=2024-12
Authorization: Bearer {{token}}
```

## Date Bounds

Dates in these functions use the following standard rules according to the format provided to determine the actual date used.

* Start and from dates always use the start of the period according to the format provided
* End dates always use the end of the period according to the format provided.

The following table explains how the dates are determined and transformed:

|Format|Example|Used Start Date|Used End Date|
|-|-|-|-|
|`yyyy`|2024|2024-01-01T00:00:00|2024-12-31T23:59:59|
|`yyyy-MM`|2024-10|2024-10-01T00:00:00|2024-10-31T23:59:59|
|`yyyy-MM-dd`|2024-10-12|2024-10-12T00:00:00|2024-10-12T23:59:59|
|`yyyy-MM-ddT00`|2024-10-12T11|2024-10-12T11:00:00|2024-10-12T11:59:59|
|`yyyy-MM-ddTHH:mm`|2024-10-12T11:36|2024-10-12T11:36:00|2024-10-12T11:36:59|
|`yyyy-MM-ddTHH:mm:ss`|2024-10-12T11:36:24|2024-10-12T11:36:24|2024-10-12T11:36:24|
|`yyyy-MM-ddTHH:mm:ss[timezone]`|2024-10-12T11:36:24[Europe/Amsterdam]|2024-10-12T09:36:24[UTC]|2024-10-12T09:36:24[UTC]|
|`Any absolute tenor`|2024M10|2024-10-01T00:00:00|2024-10-31T23:59:59|

## Date Rules
[Date rules](https://doc.opendatadsl.com/docs/kb/daterules) can be used in date ranges and will be calculated based on a reference date point of now.

### Example:

**OpenDataDSL:**

```js
ts = ${data:"DR_TEST:DAILY"} for between("T-1Y","T")
```

**REST API:**

```html
GET https://api.opendatadsl.com/api/data/v1/private/DR_EV_TEST:DAILY?_range=between(T-1Y,T)
Authorization: Bearer {{token}}
```

## Timezones
If you pass a timezone as an argument into the function, it uses that as the timezone of the passed in date(s).

### Example:

**OpenDataDSL:**

```js
ts = ${data:"DR_TEST:HOURLY"} for between("2024-01-01","2024-01-31","Europe/Amsterdam")
```

**REST API:**

```html
GET https://api.opendatadsl.com/api/data/v1/private/DR_EV_TEST:HOURLY
    ?_range=between(2024-01-01,2024-01-31,Europe/Amsterdam)
Authorization: Bearer {{token}}
```
