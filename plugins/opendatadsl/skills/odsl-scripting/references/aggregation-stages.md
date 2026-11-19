# Aggregation stages

MongoDB-style stages for the aggregate command and aggregation pipelines.

Generated from the OpenDataDSL documentation (https://doc.opendatadsl.com/docs).


---

<!-- source: product/developer/odsl/Stages/Stages.md -->

## Aggregation Stages
The following MongoDB Aggregation Stages are fully supported within the ODSL language:
* addFields
* bucket
* bucketAuto
* count
* documents
* facet
* geoNear
* graphLookup
* group
* limit
* lookup
* match
* merge
* out
* project
* redact
* replaceRoot
* replaceWith
* sample
* skip
* sort
* sortByCount
* unwind

All other pipeline stages can be used in ODSL in JSON format, e.g.

```js
aggregate ${object}
    { "$collStats": { "storageStats": { } } }
end
```


---

<!-- source: product/developer/odsl/Stages/addFields.md -->

## Definition
The ```addFields``` stage adds new fields to documents, the output from ```addFields``` are documents that contain all existing fields
and newly added fields.

[Read the official MongoDB documentation](https://www.mongodb.com/docs/manual/reference/operator/aggregation/addFields/)

## Syntax
```addFields``` has the following form:

```js
addFields varname=expression (,varname=expression)*
```

## Behaviour
```addFields``` appends new fields to existing documents. You can include one or more ```addFields``` stages in an aggregation operation.

To add a field or fields to embedded documents (including documents in arrays) use the dot notation.

To add an element to an existing array field with ```addFields```, use with ```concatArrays```

## Examples
### Using two ```addFields``` stages 

```js
// Create a new collection in the private database called scores and add 2 documents
o1 = object as #Object
    student = "Maya"
    homework = [ 10, 5, 10 ]
    quiz = [ 10, 8 ]
    extraCredit = 0
end
o1.id = 1
save ${object:"scores"/o1}

o2 = object as #Object
    student = "Ryan"
    homework = [ 5, 6, 5 ]
    quiz = [ 8, 8 ]
    extraCredit = 8
end
o2.id = 2
save ${object:"scores"/o2}

// Use 2 addFields operations to include 3 new fields in the output documents
aggregate ${object:"scores"}
    addFields totalHomework = sum("$homework"), totalQuiz = sum("$quiz")
    addFields totalScore=add("$totalHomework","$totalQuiz","$extraCredit")
end
```

### Adding fields to an embedded document
Use dot notation to add new fields to embedded documents.
This example create a collection called vehicles and adds a new field ```fuel_type``` to the embedded document ```specs```

```js
json = `
[
      { _id: 1, type: "car", specs: { doors: 4, wheels: 4 } },
      { _id: 2, type: "motorcycle", specs: { doors: 0, wheels: 2 } },
      { _id: 3, type: "jet ski" }
   ] 
`
objects = ${json:json}
save ${object:"vehicles"/objects}

aggregate ${object:"vehicles"}
    addFields "specs.fuel_type"="unleaded" 
end
```

### Overwriting an existing field
Specifying an existing field name in an ```addFields``` operation causes the original field to be replaced.

Using the prior example data in the ```vehicles``` collection.

```js
aggregate ${object:"vehicles"}
    match _id=1
    addFields type="Motor Car" 
end
```

### Add elements to an array
Using the ```scores``` collection created earlier, you can use ```addFields``` with ```concatArrays``` to add an element to an existing array

```js
aggregate ${object:"scores"}
    match _id=1
    addFields homework=concatArrays("$homework", [7]) 
end
```


---

<!-- source: product/developer/odsl/Stages/bucket.md -->

## Definition
The ```bucket``` stage categorises incoming documents into groups, called buckets, based on a specified expression and bucket boundaries.

[Read the official MongoDB documentation](https://www.mongodb.com/docs/manual/reference/operator/aggregation/bucket/)

## Syntax
```bucket``` has the following form:

```js
bucket expression boundaries [value (,value)*] (default varname)? (output varname=expression (,varname=expression)*)?
```

## Behaviour
```bucket``` requires at least one of the following conditions to be met, or the operation throws an error:

* Each input document resolves the expression to a value within one of the bucket ranges specified by boundaries
* A default value is specified to bucket documents whose values are outside the boundaries or of a different BSON type than the values in boundaries.

## Examples

### Bucket by year and filter by bucket results

The following operation create a sample collection named ```artists``` and then groups the documents into buckets according to the ```year_born``` field and filters based on the count of documents in the buckets:

:::note
The aggregation in the example below makes use of a built-in function called ```doc``` which converts the parameters into JSON elements and returns a JSON Object
:::

```js
json = `[
  { "_id" : 1, "last_name" : "Bernard", "first_name" : "Emil", "year_born" : 1868, "year_died" : 1941, "nationality" : "France" },
  { "_id" : 2, "last_name" : "Rippl-Ronai", "first_name" : "Joszef", "year_born" : 1861, "year_died" : 1927, "nationality" : "Hungary" },
  { "_id" : 3, "last_name" : "Ostroumova", "first_name" : "Anna", "year_born" : 1871, "year_died" : 1955, "nationality" : "Russia" },
  { "_id" : 4, "last_name" : "Van Gogh", "first_name" : "Vincent", "year_born" : 1853, "year_died" : 1890, "nationality" : "Holland" },
  { "_id" : 5, "last_name" : "Maurer", "first_name" : "Alfred", "year_born" : 1868, "year_died" : 1932, "nationality" : "USA" },
  { "_id" : 6, "last_name" : "Munch", "first_name" : "Edvard", "year_born" : 1863, "year_died" : 1944, "nationality" : "Norway" },
  { "_id" : 7, "last_name" : "Redon", "first_name" : "Odilon", "year_born" : 1840, "year_died" : 1916, "nationality" : "France" },
  { "_id" : 8, "last_name" : "Diriks", "first_name" : "Edvard", "year_born" : 1855, "year_died" : 1930, "nationality" : "Norway" }
]`
objects = ${json:json}
save ${object:"artists"/objects}

aggregate ${object:"artists"}
    bucket "$year_born" boundaries [1840, 1850, 1860, 1870, 1880] default "Other" output count=sum(1), artists=push(doc(name=concat("$first_name", "$last_name"), year_born="$year_born"))
    match count > 3
end
```

### Use ```bucket``` with facet to bucket by multiple fields

You can use the facet stage to perform multiple ```bucket``` aggregations in a single stage.

The following operation creates a collection called ```artwork``` and then uses two ```bucket``` stages within a facet stage to create two groupings, one by price and the other by year

```js
json = `[
  { "_id" : 1, "title" : "The Pillars of Society", "artist" : "Grosz", "year" : 1926,
      "price" : 199.99 },
  { "_id" : 2, "title" : "Melancholy III", "artist" : "Munch", "year" : 1902,
      "price" : 280.00 },
  { "_id" : 3, "title" : "Dancer", "artist" : "Miro", "year" : 1925,
      "price" : 76.04 },
  { "_id" : 4, "title" : "The Great Wave off Kanagawa", "artist" : "Hokusai",
      "price" : 167.30 },
  { "_id" : 5, "title" : "The Persistence of Memory", "artist" : "Dali", "year" : 1931,
      "price" : 483.00 },
  { "_id" : 6, "title" : "Composition VII", "artist" : "Kandinsky", "year" : 1913,
      "price" : 385.00 },
  { "_id" : 7, "title" : "The Scream", "artist" : "Munch", "year" : 1893
      /* No price*/ },
  { "_id" : 8, "title" : "Blue Flower", "artist" : "O'Keefe", "year" : 1918,
      "price" : 118.42 }
]`
objects = ${json:json}
save ${object:"artwork"/objects}

aggregate ${object:"artwork"}
  facet price 
        bucket "$price" boundaries [0, 200, 400] default "Other" output count=sum(1), artwork=push(doc(title="$title",price="$price")), averagePrice=avg("$price")      
      ,year
        bucket "$year" boundaries [1890, 1910, 1920, 1940] default "Unknown" output count=sum(1), artwork=push(doc(title="$title",year="$year"))
  end
end
```


---

<!-- source: product/developer/odsl/Stages/bucketAuto.md -->

## Definition
The ```bucketAuto``` stage categorises incoming documents into groups, called buckets, based on a specified expression.
Bucket boaundaries are automatically determined in an attempt to evenly distribute the documents into the specified number of buckets

[Read the official MongoDB documentation](https://www.mongodb.com/docs/manual/reference/operator/aggregation/bucketAuto/)

## Syntax
```bucketAuto``` has the following form:

```js
bucketAuto expression buckets int (granularity ('R5'|'R10'|'R20'|'R40'|'R80'|'1-2-5'|'E6'|'E12'|'E24'|'E48'|'E96'|'E192'|'POWERSOF2'))? (output varname=expression (,varname=expression)*)?
```

## Behaviour
There may be less than the specified number of buckets if:

* The number of input documents is less than the specified number of buckets.
* The number of unique values of the groupBy expression is less than the specified number of buckets.
* The granularity has fewer intervals than the number of buckets.
* The granularity is not fine enough to evenly distribute documents into the specified number of buckets.

The even distribution of documents across buckets depends on the cardinality, or the number of unique values, of the expression field. 
If the cardinality is not high enough, the ```bucketAuto``` stage may not evenly distribute the results across buckets.

### Granularity

See the [MongoDB documentation](https://www.mongodb.com/docs/manual/reference/operator/aggregation/bucketAuto/#granularity) on how granularity option works

## Examples

### Aggregating artwork by price

```js
json = `[
  { "_id" : 1, "title" : "The Pillars of Society", "artist" : "Grosz", "year" : 1926,
      "price" : 199.99 },
  { "_id" : 2, "title" : "Melancholy III", "artist" : "Munch", "year" : 1902,
      "price" : 280.00 },
  { "_id" : 3, "title" : "Dancer", "artist" : "Miro", "year" : 1925,
      "price" : 76.04 },
  { "_id" : 4, "title" : "The Great Wave off Kanagawa", "artist" : "Hokusai",
      "price" : 167.30 },
  { "_id" : 5, "title" : "The Persistence of Memory", "artist" : "Dali", "year" : 1931,
      "price" : 483.00 },
  { "_id" : 6, "title" : "Composition VII", "artist" : "Kandinsky", "year" : 1913,
      "price" : 385.00 },
  { "_id" : 7, "title" : "The Scream", "artist" : "Munch", "year" : 1893
      /* No price*/ },
  { "_id" : 8, "title" : "Blue Flower", "artist" : "O'Keefe", "year" : 1918,
      "price" : 118.42 }
]`
objects = ${json:json}
save ${object:"artwork"/objects}

aggregate ${object:"artwork"}
    bucketAuto "$price" buckets 4 
end
```


---

<!-- source: product/developer/odsl/Stages/count.md -->

## Definition
The ```count``` stage passes a document to the next stage that contains a count of the number of documents input to the stage.

[Read the official MongoDB documentation](https://www.mongodb.com/docs/manual/reference/operator/aggregation/count/)

## Syntax
```count``` has the following form:

```js
count string
```

## Behaviour
```count``` returns a single document with a single property named with the argument to ```count``` and a value of the number of input documents to this stage.  

## Example

A collection named ```scores``` has the following documents:

```json
{ "_id" : 1, "subject" : "History", "score" : 88 }
{ "_id" : 2, "subject" : "History", "score" : 92 }
{ "_id" : 3, "subject" : "History", "score" : 97 }
{ "_id" : 4, "subject" : "History", "score" : 71 }
{ "_id" : 5, "subject" : "History", "score" : 79 }
{ "_id" : 6, "subject" : "History", "score" : 83 }
```

The following aggregation excludes documents that have a score less than or equal to 80 and returns the count of the number of passing scores.

```js
aggregate ${object:"scores"}
    match score > 80
    count "passing_scores"
end
```


---

<!-- source: product/developer/odsl/Stages/documents.md -->

## Definition
The ```documents``` stage returns literal documents from input values.

[Read the official MongoDB documentation](https://www.mongodb.com/docs/manual/reference/operator/aggregation/documents/)

:::note
Only available for MongoDB 5.1 or later
:::

## Syntax
```documents``` has the following form:

```js
documents arrayExpression
```

## Behaviour
```documents``` can take the following as the **arrayExpression**
* A Json array expression, e.g. ```[{x:2},{x:3}]```
* A variable which evaluates to an array, e.g. a [List](https://doc.opendatadsl.com/docs/odsl/variable/list) variable 

## Examples

### Using a static array
This example takes 3 documents and buckets them

```js
aggregate ${object}
    documents [{x:10}, {x:2}, {x:5}]
    bucketAuto "$x" buckets 4 
end
```

### Using a List variable
This example uses the objects in a list variable to perform the aggregation

```js
docs = List()
for i=1 to 10
    o = Object()
    o.size = i
    docs.add(o)
next
aggregate ${object}
    documents docs
    bucketAuto "$size" buckets 4 
end
```


---

<!-- source: product/developer/odsl/Stages/facet.md -->

## Definition
The ```facet``` stage processes multiple aggregation pipelines within a single stage on the same set of input documents. Each sub-pipeline has its own field in the output document where its results are stored as an array of documents.

[Read the official MongoDB documentation](https://www.mongodb.com/docs/manual/reference/operator/aggregation/facet/)

## Syntax
```facet``` has the following form:

```js
facet 
    name (pipelineitem)* 
    (, name (pipelineitem)*)*
end
```

## Behaviour
```facet``` can use any pipeline stages except:
* collStats
* facet
* geoNear
* indexStats
* out
* merge
* planCacheStats

Each sub-pipeline within ```facet``` is passed the exact same set of input documents. 
These sub-pipelines are completely independent of one another and the document array output by each is stored in separate fields in the output document. 
The output of one sub-pipeline can not be used as the input for a different sub-pipeline within the same ```facet stage```. 
If further aggregations are required, add additional stages after ```facet``` and specify the field name of the desired sub-pipeline output.

## Example
The following example takes a json array of objects and feeds them into an aggregation pipeline using the documents pipeline stage.
It then uses MongoDB's faceting features to provide customers with the store's inventory categorized across multiple dimensions such as tags, price, and year created. 
This ```facet``` stage has three sub-pipelines that use
* sortByCount
* bucket
* bucketAuto

to perform this multi-faceted aggregation. 

```js
json = `[{ "_id" : 1, "title" : "The Pillars of Society", "artist" : "Grosz", "year" : 1926,
  "price" : 199.99,
  "tags" : [ "painting", "satire", "Expressionism", "caricature" ] },
{ "_id" : 2, "title" : "Melancholy III", "artist" : "Munch", "year" : 1902,
  "price" : 280.00,
  "tags" : [ "woodcut", "Expressionism" ] },
{ "_id" : 3, "title" : "Dancer", "artist" : "Miro", "year" : 1925,
  "price" : 76.04,
  "tags" : [ "oil", "Surrealism", "painting" ] },
{ "_id" : 4, "title" : "The Great Wave off Kanagawa", "artist" : "Hokusai",
  "price" : 167.30,
  "tags" : [ "woodblock", "ukiyo-e" ] },
{ "_id" : 5, "title" : "The Persistence of Memory", "artist" : "Dali", "year" : 1931,
  "price" : 483.00,
  "tags" : [ "Surrealism", "painting", "oil" ] },
{ "_id" : 6, "title" : "Composition VII", "artist" : "Kandinsky", "year" : 1913,
  "price" : 385.00,
  "tags" : [ "oil", "painting", "abstract" ] },
{ "_id" : 7, "title" : "The Scream", "artist" : "Munch", "year" : 1893,
  "tags" : [ "Expressionism", "painting", "oil" ] },
{ "_id" : 8, "title" : "Blue Flower", "artist" : "O'Keefe", "year" : 1918,
  "price" : 118.42,
  "tags" : [ "abstract", "painting" ] }]`
docs = ${json:json}

aggregate ${object}
    documents docs
    facet categorizedByTage
        unwind "$tags"
        sortByCount "$tags"
    , categorizedByPrice
        match price != null
        bucket "$price" boundaries [0, 150, 200, 300, 400] default "Other" output count=sum(1), titles=push("$title") 
    , "categorizedByYears(Auto)"
        bucketAuto "$year" buckets 4 
    end
end
```


---

<!-- source: product/developer/odsl/Stages/geoNear.md -->

## Definition
The ```geoNear``` stage outputs documents in order of nearest to farthest from a specified point.

[Read the official MongoDB documentation](https://www.mongodb.com/docs/manual/reference/operator/aggregation/geoNear/)

## Syntax
```geoNear``` has the following form:

```js
geoNear options
```

Where options are:
* near - The point for which to find the closest documents.
* distanceField - the output field that contains the calculated distance
* distanceMultiplier - an optional factor that all distances are multiplied by
* includeLocs - an optional field to output the location used to calculate the distance
* key - Optional. Specify the geospatial indexed field to use when calculating the distance
* maxDistance - Optional. The maximum distance from the center point that the documents can be
* minDistance - Optional. The minimum distance from the center point that the documents can be
* query - An optional query to limit the result to match the query
* spherical - An optional boolean to indicate that you want to use spherical geometry rather than planar geometry

## Behaviour
```geoNear``` requires a geospatial index to be added to the collection you want to perform the calculation on

## Examples

### Using a static array
This example takes 3 documents and buckets them

```js
aggregate ${object}
    documents [{x:10}, {x:2}, {x:5}]
    bucketAuto "$x" buckets 4 
end
```

### Using a List variable
This example uses the objects in a list variable to perform the aggregation

```js
docs = List()
for i=1 to 10
    o = Object()
    o.size = i
    docs.add(o)
next
aggregate ${object}
    documents docs
    bucketAuto "$size" buckets 4 
end
```


---

<!-- source: product/developer/odsl/Stages/graphLookup.md -->




---

<!-- source: product/developer/odsl/Stages/group.md -->

## Definition
The ```group``` stage combines multiple documents with the same field, fields, or expression into a single document according to a group key. 
The result is one document per unique group key.

A group key is often a field, or group of fields. 
The group key can also be the result of an expression. Use the _id field in the $group pipeline stage to set the group key. 

[Read the official MongoDB documentation](https://www.mongodb.com/docs/manual/reference/operator/aggregation/group/)

## Syntax
```group``` has the following form:

```js
group _id=field (,varname=accumulator)*
```

## _id field
The _id expression specifies the group key. 
If you specify an _id value of null, or any other constant value, the ```group``` stage returns a single document that aggregates values across all of the input documents.

## Accumulators
The ```accumulator``` operators must be one of the following:

|Name|Description|
|-|-|
|addToSet|Returns an array of unique expression values for each group. Order of the array elements is undefined|
|avg|Returns an average of numerical values. Ignores non-numeric values|
|bottom|Returns the bottom element within a group according to the specified sort order|
|bottomN|Returns an aggregation of the bottom n fields within a group, according to the specified sort order|
|concatArrays|Returns a single array that combines the elements of two or more arrays|
|count|Returns the number of documents in a group|
|first|Returns the result of an expression for the first document in a group|
|firstN|Returns an aggregation of the first n elements within a group. Only meaningful when documents are in a defined order. Distinct from the $firstN array operator|
|last|Returns the result of an expression for the last document in a group|
|lastN|Returns an aggregation of the last n elements within a group. Only meaningful when documents are in a defined order. Distinct from the $lastN array operator|
|max|Returns the highest expression value for each group|
|maxN|Returns an aggregation of the n maximum valued elements in a group. Distinct from the $maxN array operator|
|median|Returns an approximation of the median, the 50th percentile, as a scalar value|
|mergeObjects|Returns a document created by combining the input documents for each group|
|min|Returns the lowest expression value for each group|
|minN|Returns an aggregation of the n minimum valued elements in a group. Distinct from the $minN array operator|
|percentile|Returns an array of scalar values that correspond to specified percentile values|
|push|Returns an array of expression values for documents in each group|
|setUnion|Takes two or more arrays and returns an array containing the elements that appear in any input array|
|stdDevPop|Returns the population standard deviation of the input values|
|stdDevSamp|Returns the sample standard deviation of the input values|
|sum|Returns a sum of numerical values. Ignores non-numeric values|
|top|Returns the top element within a group according to the specified sort order|
|topN|Returns an aggregation of the top n fields within a group, according to the specified sort order|

## Examples


---

<!-- source: product/developer/odsl/Stages/limit.md -->

## Definition
The ```limit``` stage limits the number of documents passed to the next stage in the pipeline.

[Read the official MongoDB documentation](https://www.mongodb.com/docs/manual/reference/operator/aggregation/limit/)

## Syntax
```limit``` has the following form:

```js
limit integer
```

## Behaviour
```limit``` is generally used in conjunction with skip as part of a pagination pattern, allowing you to define the page size for a list of documents.

## Example

A collection named ```scores``` has the following documents:

```json
{ "_id" : 1, "subject" : "History", "score" : 88 }
{ "_id" : 2, "subject" : "History", "score" : 92 }
{ "_id" : 3, "subject" : "History", "score" : 97 }
{ "_id" : 4, "subject" : "History", "score" : 71 }
{ "_id" : 5, "subject" : "History", "score" : 79 }
{ "_id" : 6, "subject" : "History", "score" : 83 }
```

The following aggregation skips the first 2 documents and returns only the next 2 documents

```js
aggregate ${object:"scores"}    
    skip 2
    limit 2
end
```


---

<!-- source: product/developer/odsl/Stages/lookup.md -->




---

<!-- source: product/developer/odsl/Stages/match.md -->




---

<!-- source: product/developer/odsl/Stages/merge.md -->




---

<!-- source: product/developer/odsl/Stages/out.md -->




---

<!-- source: product/developer/odsl/Stages/project.md -->




---

<!-- source: product/developer/odsl/Stages/redact.md -->




---

<!-- source: product/developer/odsl/Stages/replaceRoot.md -->




---

<!-- source: product/developer/odsl/Stages/sample.md -->




---

<!-- source: product/developer/odsl/Stages/skip.md -->

## Definition
The ```skip``` stage skips over the specified number of documents that pass into the stage and passes the remaining documents to the next stage in the pipeline.

[Read the official MongoDB documentation](https://www.mongodb.com/docs/manual/reference/operator/aggregation/skip/)

## Syntax
```skip``` has the following form:

```js
skip integer
```

## Behaviour
```skip``` is generally used in conjunction with limit as part of a pagination pattern, allowing you to skip through pages of documents.

## Example

A collection named ```scores``` has the following documents:

```json
{ "_id" : 1, "subject" : "History", "score" : 88 }
{ "_id" : 2, "subject" : "History", "score" : 92 }
{ "_id" : 3, "subject" : "History", "score" : 97 }
{ "_id" : 4, "subject" : "History", "score" : 71 }
{ "_id" : 5, "subject" : "History", "score" : 79 }
{ "_id" : 6, "subject" : "History", "score" : 83 }
```

The following aggregation excludes the first 2 documents and returns the remaining 4

```js
aggregate ${object:"scores"}
    skip 2
end
```


---

<!-- source: product/developer/odsl/Stages/sort.md -->

## Definition
The ```sort``` stage sorts all input documents and returns them to the pipeline in sorted order.

[Read the official MongoDB documentation](https://www.mongodb.com/docs/manual/reference/operator/aggregation/sort/)

## Syntax
```sort``` has the following form:

```js
sort field asc|desc (, field asc|desc)*
```

## Behaviour
### Limits
You can sort on a maximum of 32 keys.

### Sort Consistency
MongoDB does not store documents in a collection in a particular order. When sorting on a field which contains duplicate values, documents containing those values may be returned in any order.

If consistent sort order is desired, include at least one field in your sort that contains unique values. The easiest way to guarantee this is to include the _id field in your sort query.

## Example

A collection named ```scores``` has the following documents:

```json
{ "_id" : 1, "subject" : "History", "score" : 88 }
{ "_id" : 2, "subject" : "History", "score" : 92 }
{ "_id" : 3, "subject" : "History", "score" : 97 }
{ "_id" : 4, "subject" : "History", "score" : 71 }
{ "_id" : 5, "subject" : "History", "score" : 79 }
{ "_id" : 6, "subject" : "History", "score" : 83 }
```

The following aggregation sorts the documents by the score field and returns the first 2 (_id: 4 and 5) 

```js
aggregate ${object:"scores"}
    sort score asc    
    limit 2
end
```


---

<!-- source: product/developer/odsl/Stages/sortByCount.md -->




---

<!-- source: product/developer/odsl/Stages/unwind.md -->


