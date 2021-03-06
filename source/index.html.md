---
title: Open Bouquet API Reference

language_tabs:
  - shell : cURL

toc_footers:
  - <a href='#'>Sign Up for a Developer Access</a>
  - <a href='https://github.com/tripit/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---


Welcome to the Open Bouquet API! You can use our API to perform analytics on your data-sources connected to your Open Bouquet server.

We have language bindings in Shell and HTML! You can view code examples in the dark area to the right, and you can switch the programming language of the examples with the tabs in the top right.

The best way to give a try to the API is to download Open Bouquet for free, install it on your laptop, plug your database, and start exploring! [Open Bouquet website](url "https://openbouquet.io") 

# Version 4.2.30

```shell
# you don't need to be authenticated to check the status
curl -X GET "http://yourserverdomain/v4.2/rs/status
```

This documentation is up to date with the **version 4.2.30** of Open Bouquet.

You can get your OB server version by calling the /rs/status API:

The /rs/status API return the `bouquet-server` version and also the list of available database plugins `bouquet-plugins`.

# Authentication

> To check if you need to authenticate, use this code for example:

```shell
# With shell, just check the analytics endpoint
curl -X GET "http://yourserverdomain/v4.2/analytics
```
> If you need to authenticate, you'll get the following error, where loginURL is the authentication end-point. You can also use the selfLoginURL to be automatically redirected to the original URL after sign-in.

```json
{"loginURL":"http://yourserverdomain/dev/auth/oauth",
"selfLoginURL":"http://yourserverdomain/dev/auth/oauth?client_id=admin_console
&redirect_uri=https://yourserverdomain/dev/v4.2/analytics?style=HTML",
"code":401,
"error":"Auth failed : invalid access_token",
"type":"InvalidTokenAPIException",
"suppressed":[]}
```

The authentication mechanism depends on the version of OB server you are using.
By default the server authentication is turned-off.

When the authentication is turned on, it uses OAuth2, so you will have to retrieve an authentication token from the authentication server.

Note that by default, if you are not authenticated and  try to access the API, it will throw a 401 error code, with the authentication URL in the error message.

OB expects for the OAuth token to be included in all API requests to the server in a header that looks like the following:

`Authorization: Bearer TOKEN`

For convenience you can also provide the TOKEN as part of the URL, using the `access_token` parameter:

`GET http://yourserverdomain/dev/v4.2/analytics&access_token=TOKEN`

<aside class="notice">
In all examples you must replace <code>TOKEN</code> with the OAuth2 token.
</aside>

# Bouquet Concepts

This is a brief overview of the various concepts use by Bouquet to organize information.

## Projects

A Project is the top level object that organize the data-source model. First of all a Project defines how to access the data-source, through a JDBC url and required authentication. The Project also defines which parts of the data-source will be made available through the API, by listing visible schemas.

A Project maintains a list of Domains.

## Domains

A Domain is associated with a table or a view in the data-source. A Domain can be created explicitly by a user but Domains are also dynamically created by Open Bouquet from tables when a new Project is created.A user can then modify a dynamic Domain to make it permanent and persist the changes to the model.

A Domain has Dimension and Metric attributes, which are dynamically mapped to the underlying data-source. A user can also modify any existing attribute, or create new ones. Those changes are then persisted in the model.

### Dimensions

A Dimension is any non-aggregated expression defined on the Domain.

### Metrics

A Metric is any expression defined on the Domain. If the metric is an aggregated expression, it will be used as is. If it is neither an aggregated expression, nor an analytics expression, it must be possible to apply an aggregation operator like SUM(), ...

Since a Metric may be any expression, it can contain conditional operator like CASE(), combined with an aggregation operator.

## Relations

A Relation defines how to join two Domains. It is defined by a left domain, a right domain, and a predicate expression between the two. It also specify the relation cardinality.

##Hierarchies

## Bookmarks

A Bookmark is a simple way to create a custom view on your model. From a Domain, you can define a default analysis, select filters, restrict the dimensions and metrics usable for pivoting. Bookmarks are also organized on their own inside folders, independently from the Projects - so you can mix in the same folder Bookmarks from different Projects.

# Analytics API

The Analytics API provides you all with you need to interact with your data:

* list available content like Projects, Bookmarks and Domains
* query a specific Bookmark or Domain
* view dataviz for a specific Bookmark or Domain (as VegaLite output)
* explore a Bookmark or Domain scope
* create a Bookmark out of a query

Note that to configureand enrich a project and its domains, you should use the Management API.

<aside class="notice">
The Analytics API treats Domains and Bookmarks as mostly the same. Although they are different kinds of objects, you can query and view them using the same operations. 
</aside>

<aside class="notice">
In order to discover the API features in a more friendly way, you can use most of the API methods with the url parameter style=HTML.
In that mode the API will output some simple HTML that allows interaction through forms. It also provides references to the API parameters and links to some of the related calls you may want to do.
</aside>

## List available content

> This is a very simple example demonstrating how you can get the top level content in your account. In that example you don't need to provide any parameter.

```shell
curl -X GET 
	--header 'Accept: application/json' 
	--header 'Authorization: Bearer TOKEN'
	"http://yourserverdomain/v4.2/analytics"
```

This method retrieves all the content available for the authenticated user.

<aside class="notice">
This API method support the style=HTML for interactive exploration.
</aside>

### HTTP Request

`GET http://yourserverdomain/dev/v4.2/analytics`

<aside class="success">
Remember to provide a valid OAuth TOKEN!
</aside>

### URL Parameters

There are no URL parameters for this method.

### Query Parameters

Parameter | Required | Description
--------- | ------- | -----------
parent | false, default=/ | This is the path of a valid parent container. If the path is not defined, or path=/, it will display the top level content. In order to view a child content, use path=<child.selfRef>. For example in order to list all projects, path=/PROJECTS. This is working like a standard Unix path, for example: /PROJECTS/MyProject/ to list MyProject's domains, or /MYBOOKMARKS/some/folder to list bookmarks and folders. 
q | false | This is a search string that you can use to filter the result. Multiple tokens can be separated by commas.
visibility | false, default=ALL | If visibility is set to ALL, will show all existing content. If visibility is VISIBLE, it will only show objects that have been made explicitly visible.
style | false, default=HUMAN | defines the output style, check API reference
envelope | false, default=RESULT | defines the output content, check API reference

### Parent parameter

> This is an example demonstrating how you can list the projects available in your account.

```shell
curl -X GET 
    --header 'Accept: application/json' 
    --header 'Authorization: Bearer TOKEN'
    "http://yourserverdomain/v4.2/analytics?parent=/PROJECTS"
```

The parent parameter is a path. Each section of the path identifies a valid container, using either its name or immutable ID.

When browsing the content, you can always use a child `selfRef` property to deep dive into it.

#### Top Level content

The top level container are immutable: `/PROJECTS`, `/MYBOOKMARK`, `/SHARED`

* PROJECTS lists all available projects for the authenticated user
* MYBOOKMARK lists all bookmarks created by the authenticated user
* SHARED lists all bookmarks available to the authenticated user

#### Project content

In order to list a project's content, you can use either its ID or its name and so to list your project content you can use both of these paths:

* path=`/PROJECTS/@MyProjectID/`
* path=`/PROJECTS/My Project/`

Note that the name can change, so if you need to make a persistent reference to the project, it is better to use its ID (remember that by using the style=HUMAN, the API will return objects by name. If you'd rather list by ID, you can use style=ROBOT. Note that in all case you can use either name or ID to reference ID). 

Note that in order to list a project content, the OB server will request access to the project's database, and update the content dynamically. So the first time you access it may take some delay, depending on the database.

#### Bookmarks

Both the `/MYBOOKMARK` and `/SHARED` paths return a list of Bookmarks and Folders. The only difference is the bookmark visibility:

* `/MYBOOKMARK` only returns the bookmarks created by the authenticated user 
* `/SHARED` returns the bookmarks shared with the authenticated user


### Reply

> This is the reply from the previous API call for listing available projects

```json
{
"parent": {
    "name": "Projects",
    "description": "list all your Projects",
    "parentRef": "/",
    "selfRef": "/PROJECTS",
    "type": "FOLDER",
    "link": "http://yourserverdomain/v4.2/analytics?parent=/PROJECTS&style=HUMAN&visibility=ALL&access_token=xxx",
    "upLink": "http://yourserverdomain/v4.2/analytics?parent=/&style=HUMAN&visibility=ALL&access_token=xxx"
    },
"children": [{
    "name": "demo project",
    "attributes": {
        "jdbc": "jdbc:postgreql://databasehost:5432/demo"
    },
    "parentRef": "/PROJECTS",
    "selfRef": "/PROJECTS/demo project",
    "type": "PROJECT",
    "link": "http://yourserverdomain/v4.2/analytics?parent=/PROJECTS/demo project&style=HUMAN&visibility=ALL&access_token=xxx",
    "objectLink": "http://yourserverdomain/v4.2/rs/projects/576002908294a27bac5a6946?style=HUMAN&access_token=xxx"
    }]
}
```

The reply has the following structure:

* The `parent` node describes the current location of the `parent` parameter. You can, amongst other things, use it to browse upward, as it also provides  a `parentRef` property (unless it is the root folder).

* The `children` node is an array of nodes, and it is the content of the `parent` node.

Both nodes have the same structure:

Property | Description
-------- | -----------
name |
description | 
parentRef | 
selfRef | 
type | 

The `link` provides you with direct navigation into the content hierarchy. If the object type is a container, the link navigates to the content. If the object type cannot be navigated, it is a link to the associated query.

The `objectLink` provides a link to the detailed definition of the object and will redirect you to a different API depending on the object type. This API is actually the metadata management API that allows you to discover but also modify the model definition.

<aside class="notice">
Navigation in your browser is easier if you install a json prettify plugin :)
</aside>

## Query a Bookmark or Domain

This method performs a query on a Bookmark or a Domain and returns the result as a paginated data table.

The result of a query is automatically cached server-side. Therefore, performing the same query repeatedly is no cause for performance issues :  no need to cache the result on the application side. 

 Still, it is a good practice to set a `limit` on the query, especially for interactive queries. If you don't, and although the server will paginate the result internally to avoid memory overflow, you may have trouble retrieving the result in JSON. If you really need to extract a large table, prefer using the /export API method which provides streaming results.

You can also paginate the results to simplify display.

It is possible to change the layout of the table using the `data` parameter.

<aside class="notice">
This API method supports the style=HTML parameter for interactive exploration.
</aside>

### HTTP Request

> Example of running the default query for a Domain

```shell
# run a default query on the domain 'Sales' in project 'Demo'
curl -X GET 
    --header 'Accept: application/json' 
    --header 'Authorization: Bearer TOKEN'
    "http://yourserverdomain/v4.2/analytics/'Demo'.'Sales'/query
```

`GET http://yourserverdomain/v4.2/analytics/<REFERENCE>/query`

<aside class="success">
Remember to provide a valid OAuth TOKEN!
</aside>

### URL Parameters

Parameter | Required | Description
--------- | -------- | -----------
REFERENCE | true | The identifier of the Bookmark/Domain to retrieve

#### Bookmark/Domain Reference

### Query Parameters

Note that you can use the API without providing any additional parameters. In that case the API will compute the default analysis for the reference.
If it's a Bookmark, it will use the Bookmark default analysis. In case of a Domain, it will do an no-op analysis.

Parameter | Required | Description
--------- | -------- | -----------
groupBy | false | A list of Expressions to group the result by. This is a multi-valued parameter. Note that the list can be empty: in that case the resulting dataset will be one row as maximum if you provide metrics to compute. Each value must be a valid expression in the Bookmark or Domain scope. See the Expression chapter for reference, but usually you will provide a dimension name you want to group the result by. Alternatively you can use a wildcard (*) as a value to replace the default reference settings. In case of a Bookmark it will be replaced by the selected dimensions; in case of a Domain it will be replaced by ALL dimensions (so use with care).
metrics | false | A list of Expressions to compute. This is a multi-valued parameter. Note that the list can be empty:  in that case only the group by is applied (However the query will still perform a group by: if you want to export raw data you need to include an unique key in the list, such as a customer identifier). Each value must be a valid expression in the Bookmark or Domain scope. See the Expression chapter for reference, but usually you will pass a metric name you want to aggregate the data upon. You can also use any valid expression (usually of numeric value) by combining dimensions, metrics and functions.
filters | false | A list of conditional expressions to filter the data on. Note that the list can be empty. Each value must be a valid expression in the Bookmark or Domain scope. See the Expression chapter for reference, but usually you can use any dimension of type conditional (also called segment), or create a simple condition, using constant values for instance. You can also use any valid conditional expression by combining dimensions, metrics and functions.
period | false | the period expression is a shorthand to filter on a timeframe. The expression type is expected to be a date or a timestamp. Note that you can use only a simple expression that must resolve to a dimension (with or without indexing). You can use any function, especially date functions. If this parameter is set, you should provide a timeframe parameters too - see next one for details.
timeframe | false | the `timeframe` parameter is used to define the date or time range of the period. You can either use explicit values (in that case you should provide both  a lower and an upper bound, though you can pass a null value to create a semi-open range) or use an alias value or a constant expression. See the [Period definition](#period-timeframe) for details. 
compareTo | false | the `compareTo` parameter is used in conjunction with the `timeframe` parameter to create an automatic comparison between present and past metrics values. Setting a value will automatically turn the compare feature on (see [compareTo section for details](comparing-present-and-past-metrics))  You can use either explicit values (in that case you should provide both a lower and an upper bound, though you can pass a null value to create a semi-open range) or an alias value or a constant expression. Note that the alias values for the `compareTo` parameter are different from the `timeframe` parameter.
orderBy | false | a list of sort Expressions to order the results. Note that the list can be empty. Each value must be a valid *sort* Expression in the Bookmark or Domain scope. A valid *sort* Expression is any Expression enclosed by the `DESC()` or `ASC()` functions. In order to define the value to sort on, you can use either an index value, the name of a column (Dimension or Metric), or any valid expression.
rollup | false | a list of rollup Expressions to define how to compute sub-totals. Note that the list can be empty. Each value must be a valid *rollup* expression identifying a query groupBy column. See [Computing Rollup](#computing-rollup) for details. 
limit | true | the `limit` parameter simply allows to limit the number of rows extracted **server side**. This is mandatory for the /query API to avoid memory overflow. If you need a full ResultSet, you should rather use the [/export API](#export-a-bookmark-or-domain) which is designed to handle a large number of rows, potentially exceeding the server internal memory. See [Query Pagination](#query-pagination) for details.
offset | false | the `offset` parameter allows to paginate the rows extracted **server side**. The default value is to start at the first row. See [Query Pagination](#query-pagination) for details.
maxResults | false | the `maxResults` parameter controls the client-side pagination, and defines the page size. Note that paginating is unexpensive since the complete query (with limit and offset) is stored in the server cache. See [Query Pagination](#query-pagination) for details.
startIndex | false | the `startIndex` parameter controls the client-side pagination, and defines the page starting row. Note that paginating is unexpensive since the complete query (with limit and offset) is stored in the server cache. See [Query Pagination](#query-pagination) for details.
lazy | false | when the `lazy` parameter is set to true, the server tries to retrieve the results from the cache only. If it is not available, no actual computation is performed **at all**. Note that the server may try to perform some transformation if it cannot lookup the exact same query from the cache (that's the smart cache). If the result is not in cache, the /query API returns an error code 404. See [Controlling Query Execution](#controlling-query-execution) for details.
timeout | false | The timeout is in milliseconds. If provided, the request execution will be interrupted if its computation exceeds the provided timeout. In that case, the API will return an ComputingInProgressAPIException. Note that the execution will continue server side, you are only interrupting the request. You can run the same request again (with or without timeout) in order to get the reply (each request is actually pooled, so performing multiple request is actually running the query only once). Note also that the error message contains a QueryID that identifies running query. You can use the /status/QueryID API in order to get information regarding the query (see [/status API](#query-status)) or cancel the execution.
state | false | The `state` parameter, if provided, is used to apply the state configuration to the current domain. It will override any other settings. You can copy an application state - it will compute the corresponding query. Note however that an application can store custom information and do additional computation, that won't be supported by the API. Also the state must be for the same domain for this to work.
data | false | This parameter defines how to layout the data output. You can use it in conjunction with style and envelope to further customize the reply. See the [Supported Data Layouts](#supported-data-layouts) for up to date information.
style | false | This parameter defines the reply style. The style can modify the response type in order to get a HTML version of the reply. It can also modify the content of the JSON reply by altering the way it references objects. If `style=HUMAN` the API will use object names, so you can read and easily understand the output;  the downside is that those outputs are fragile to name changes, so you cannot use them to build an application for instance. If you need the references to be immutable, you should to use `style=ROBOT`, the downside being that the output is going to be difficult to read. Note that you can switch between `HUMAN` and `ROBOT` easily, only the references are modified.
envelope | false | This parameter defines the content of the reply. You can use it to further customize what information you need, from a full reply including the explicit query as it is interpreted by the API, reply meta-information, header and data; or you can just ask for the data. See [Style Parameter](#style-parameter) for details.


### Default Query

If no parameter is provided, the method will try to compute a default query.

In case of a Bookmark reference, the /query API will compute the Bookmark pre-defined analysis, with default settings.

In case of a Domain, the /query API will just add the available metrics and compute totals. If a date or timestamp Dimension is defined, the /query API will also try to use it to restrict the query to the current month.

#### Using State

If a stateID is provided using the `state` parameter, the /query API will try to apply the state. Note that the state must be defined on the same Domain for this to work.

### Expressions

The Expressions are all evaluated in the current Domain scope (if the reference is a Bookmark, the scope is the Bookmark's Domain's).

See the ["Expression Reference Guide"](#expression-reference-guide) for details about how to create expressions.

### Defining Query

Group By and Metrics

### Filtering



### Period & TimeFrame

The `period` parameter defines the Expression to use as the main Period for filtering on `timeframe` and also to perform time-over-time comparison (see [Comparing Present and Past Metrics](#comparing-present-and-past-metrics)).

The `period` definition must resolve to a simple Dimension - so in practice you cannot use functions here. Note that there is two way you can reference a dimension:
 * using the Dimension name (or ID): `'customer > registration date'`
 * by combining relations: `'Customer'.'registration date'` 
 
Once the `period` is defined, you can then use the `__PERIOD` alias in the expression you are using for other parameters (for example `groupby` or `orderby`; it also work with the [/view API](#view-a-bookmark-or-domain) as `x`, `y`, `color`, ... parameters). Note that you can also use the alias `__PERIOD`  in Expressions to extract the first day of month of the date: for example use `MONTHLY('__PERIOD')` if you want to aggregate results by month.


You can also use the `filters` parameter to filter on period. This may be useful for special cases, e.g. to define a non-constant period filter. Note that if you use directly the `filter` parameter, the `__PERIOD` alias is not available.

The `timeframe` parameter is an array of values that you can use to filter the data on the `period` value.
If the `timeframe` is not defined, and even if the `period` is defined, no filter is applied.
If the `timeframe` is defined, you must also define the `period` (unless there is a default value available for it).

There are three ways to define the `timeframe`: 
 * using an explicit date range
 * using an alias
 * using a expression

#### TimeFrame defined using explicit date range

You can simply provide the date or timestamp value as a formatted string. The API supports the following formats:
 * ISO 8601 timestamp: `yyyy-MM-dd'T'HH:mm:ss.SSSZ`
 * ISO 8601 date: `yyyy-MM-dd`
 * default Java timestamp format: `EEE MMM dd HH:mm:ss zzz yyyy`

#### TimeFrame defined using an alias

Four alias are currently available :
 * `__LAST_7_DAYS` 
 * `__CURRENT_MONTH` 
 * `__PREVIOUS_MONTH`
 * `__CURRENT_YEAR`

#### TimeFrame defined using an expression

The timeframe can define a constant expression. 

### Comparing Present and Past metrics

The `compareTo` parameter accepts the same definitions as the `timeframe` parameter, except for the aliases. The `compareTo` parameter defines a different set of aliases that you can use to easily define relative comparisons:

 * `__COMPARE_TO_PREVIOUS_PERIOD` 
 * `__COMPARE_TO_PREVIOUS_MONTH` 
 * `__COMPARE_TO_PREVIOUS_YEAR`
 
#### ordering by comparison metric
 
 

### Ordering the results

#### Sort Expression

#### Defining how to sort

The easiest way to define a sorting order is to use an existing ResultSet column and provide the position of the column. For example, in order to sort on the first column, we would use `ASC(0)` .

Please note that the position is zero-based, so the first column is 0, the second is 1, etc...

<aside class="warning">
You cannot use an index based orderBy to sort by a compareTo metric. In that case (if the `compareTo` parameter is set), indexes are not taking into account the compareTo columns generated by the query. If you need to sort the result based on a compareTo column you must use the explicit column name (see <a href="#ordering-by-comparison-metric">ordering by comparison metric</a> for details)
</aside>

### Computing Rollup


### Controlling Query Execution

Different parameters allow you to control and monitor the Query execution:
 * using Lazy execution to get result only from the cache
 * using a timeout to control how long to wait for the results
 * once the Query has timed out,  it is then possible to monitor progress and cancel it, see [Query Status](#query-status) for details.

#### Lazy execution

When the `lazy` parameter is set to true, the server tries to retrieve the result from the cache only. If it is not possible, no actual computation is performed **at all**. Note that the server may try to perform some transformations if it cannot lookup the exact same query from the cache (that's the smart cache).

If the query is not in cache, the reply will return a  code 404 with an error NOT_IN_CACHE.

Note that if you use the `lazy=noError`, the /query API will return a valid reply instead of an error code, with an empty ResultSet and an error message identifying the cause (Not in cache). That may be useful in some situation!

#### Timeout

### Query Pagination

The API provides the ability to paginate results. There are two ways to paginate the results:

* limit the results returned by the database. This is controlled by the `limit` and `offset` parameters. Note that in order to prevent any memory overhead while returning the data through the API, the /Query API enforces that a limit is always defined (default is `limit=1000`). 
Internally Open Bouquet use streaming and cache pagination in order to control memory footprint so that even with a high limit value can be handled. Still, if you want to export a full data-set without any limitation, you should rather use the `/export` API which is specifically designed for that.

* paginate the data-table to limit the size of the data returned to the API client. This is controlled by the `maxResults` and `startIndex` parameters. Since the data-table is already cached server side, using the client pagination is very fast and does not imply any database round-trip (unless the cache has been invalidated).
 
#### Database Pagination

The `limit` parameter defines the maximum number of rows to retrieve from the database. The `offset` parameter defines the first row to retrieve.
For example:

* in order to retrieve the first hundred rows, use: `limit=100 & offset=0` 
* in order to retrieve the next page, use: `limit=100 & offset=100`


#### Client Pagination

The `maxResults` parameter defines the maximum number of rows to send back to the client. The `startIndex` parameter defines the first row to retrieve.
For example:

* in order to retrieve the first ten rows, use: `maxResults=10 & startIndex=0` 
* in order to retrieve the next page, use: `maxResults=10 & startIndex=10`

#### How can I paginate through a complete data-set?

You can use the database & client pagination and also the query result to easily iterate through a complete data-set.
Say for example that you are running a new query without much knowledge of the size of the output. You are using some default values for pagination: `limit=1000 & maxresults=100` in order to preview the results in a table. The /query output provides you information regarding the size of the data-set:

* `result.info.totalSize=1000`
* `result.info.complete=false`

In that case the `result.info.complete` flag turns to `false` to signal you that you can use the offset to keep retrieving data from the database. This is because the data-set is larger than the limit : here the server hits the database pagination limit.


In order to get the next page, just run the same query using `limit=1000 & offset=1000 & maxResults=10`.
Note that when `result.info.complete=true`, the `result.info.totalSize` returns the size of the current page.

You can also get information regarding the client pagination:

* `result.info.pageSize` returns the value of the `maxResults` parameter
* `result.info.startIndex` returns the value of the `startIndex` parameter
i

### Reply

#### Table Header

#### Supported Data Layouts

The `data` parameter allows to select the output layout.

Data&nbsp;Value | Table Layout
---------- | ------------
`TABLE` | returns a matrix, that is an array of rows. A row is an array of cell values.
`RECORDS` | returns the table as an array of records. Each row is a record of the form `{"columnName":value,...}`
`TRANSPOSE` | if the query has multiple metrics, transposes the table to create a row for each record/metric. It adds two additional columns `metric` and `value`
`SQL` | additionally you can ask to retrieve the SQL code for the Query instead of the results
`LEGACY` | returns a JSON compatible with legacy API - this is for internal use only, you should not use it, it will be deprecated soon.

Note that you can play with the `envelope` parameter in order to customize further the reply. For instance if you want to get only the data as the reply, you can add `envelope=DATA` parameter. If `data=SQL` is used, this will return just the SQL statement. See [Envelope Parameter](#envelope-parameter) for details.

### Examples

In these examples we will use a Project `Demo` that contains a Domain `Sales`.

In order to make the example clearer, we will use the POST syntax, and only display the json payload as example.

```shell
# run a POST query on the domain 'Sales' in project 'Demo'
curl -X POST 
    --header 'Accept: application/json' 
    --header 'Authorization: Bearer TOKEN'
    "http://yourserverdomain/v4.2/analytics/'Demo'.'Sales'/query
    --data '{
        "period":"date",
        "timeframe":["2016-01-01","2016-02-29"],
        "groupBy":["'Country'"],
        "metrics":["'Total'"]
    }'
```

#### How to compute metrics by month and display it by pivot?

How can one create a query that returns the following table ?

Country | Total | Jan 2016 | Fev 2016
------- | ----- | -------- | --------
France | 1.000 | 400 | 600
US | 2.000 | 800 | 1.200

<aside class="notice">
Note that we are using POST version of /analysis/ID/query for clarity. You can as well use the GET version.
</aside>


Let's start with the simple query that returns the total for the selected period (from 2016/01/01 to 2016/02/29):

Country | Total 
------- | ----- 
France | 1.000 
US | 2.000 

> query monthly Total by Country 

```json
{
"period":"Date",
"timeframe":["2016-01-01","2016-02-29"],
"groupBy":["'Country'","monthly(__PERIOD)"],
"metrics":["'Total'"]
}
```

We can easily add the monthly total to get the details:

Country | Monthly | Total 
------- | ------- | -----
France | 2016/01/01 | 1.000 
US | 2016/02/29 | 2.000 

<aside class="notice">
Note that we are using the `__PERIOD` alias in place of the `Date` dimension, and group by month using the `monthly()` function.
</aside>

> Query Monthly Total by Country with Rollup at Country level

```json
{
"period":"Date",
"timeframe":["2016-01-01","2016-02-29"],
"groupBy":["'Country'","monthly(__PERIOD)"],
"metrics":["'Total'"],
"orderBy":["'Country'","monthly(__PERIOD)"],
"rollup":[0]
}
```

We can also add the total by country using the rollup feature:

GROUPING_ID | Country | Monthly | Total 
----------- | ------- | ------- | -----
1 | France |  | 1.000 
  | France | 2016-01-01 | 400
  | France | 2016-02-01 | 600
1 | US | | 2.000 
  | US | 2016-01-01 | 800
  | US | 2016-02-01 | 1.200
  
> Query Total by Country for the Period and for specific months

```json
{
"period":"Date",
"timeframe":["2016-01-01","2016-02-29"],
"groupBy":["'Country'"],
"metrics":["'Total'"
          ,"'Total' ON monthly(__PERIOD)=date(\"01/01/16\") as 'JAN 2016'"
          ,"'Total' ON monthly(__PERIOD)=date(\"01/02/16\") as 'FEV 2016'"]
}
```

Now in order to pivot, we need to add additional metrics for each required column:

Country | Total | Jan 2016 | Fev 2016
------- | ----- | -------- | --------
France | 1.000 | 400 | 600
US | 2.000 | 800 | 1.200


## Query Status

This method provides the status of a running query.


## Export a Bookmark or Domain

This method is similar to the /query API but will stream the result as a data file instead of returning a JSON reply.

It supports several formats for export: CSV, XLS, XLSX

### HTTP Request

`GET http://yourserverdomain/v4.2/analytics/{REFERENCE}/export/{filename}`

<aside class="success">
Remember to provide a valid OAuth TOKEN!
</aside>

### URL Parameters

Parameter | Required | Description
--------- | -------- | -----------
REFERENCE | true | The identifier of the Bookmark or the Domain to retrieve
filename | true | The filename can be defined here to provide a filename when the user will be prompted to save the file. The name part is optional, but the file extension must be provided to define the format of the export

Filename&nbsp;extension | Format
----------------------- | ------
.csv | Comma separated file with a header for the columns
.xls | Excel file format
.xlsx | Excel file format


## View a Bookmark or Domain

This method can be used in conjunction with the /query API in order to obtain a representation of the result suitable for generating a chart.

For now it supports the Vega Lite format: [Vega Lite website](url "https://vega.github.io/vega-lite/") 

### HTTP Request

`GET http://yourserverdomain/v4.2/analytics/{REFERENCE}/view`

### HTML preview

You can interactively explore the /view API by adding the parameter `style=HTML`. In that mode, the API will directly return a HTML preview including the Vega Lite chart.
You can then adjust the specific /view parameters in order to explore the possibilities.

## Explore Scope

This method can be used to list the objects available in a given scope. The list includes:

* Dimensions
* Metrics
* Relations
* Columns (from the underlying table if the user has access)
* Functions (available in the scope or for the underlying database)

### HTTP Request

`GET http://yourserverdomain/v4.2/analytics/{REFERENCE}/scope`

### URL Parameters

Parameter | Required | Description
--------- | -------- | -----------
type | false | the type parameter allows you to filter the scope by specifying the type of Expression: `DIMENSION`, `COLUMN`, `METRIC`, `RELATION`, `FUNCTION`
values | false | the value parameter allows you to filter the scope by specifying the return value of the Expression: `DATE`, `STRING`, `CONDITION`, `NUMERIC`, `AGGREGATE`
value | false | the value parameter allows you to specify an Expression to be evaluated in the current scope. The /scope output will then be computed for the resulting scope of the Expression. If an Expression is a relation, the scope will be computed for the relation target domain. Remember that you can compose Expressions using the dot `.`operator to navigate through domains.
target | false | this optional parameter is a reference to another domain. If it is provided the scope is computed for a relation between the source and the target domain.

### Examples




## API output parameters

All API methods returning data accept some parameters to modify the output:

Parameter | Required | Description
--------- | -------- | -----------
style | false | defines the output style. Values can be HUMAN, ROBOT, HTML or LEGACY. See next chapter for details
envelope | false | defines the output content. Values can be ALL, RESULT, DATA

### Style parameter

This parameter defines the reply style.

Style&nbsp;Value | Table Layout
---------- | ------------
`HUMAN` | When the `style` parameter is set to `HUMAN`, the API will use names as object references. This allows the reply to be easily readable by human. The drawback is that the information is fragile to name changes, so you shouldn't use it statically in your apps.
`ROBOT` | When the `style` parameter is set to `ROBOT`, the API will use immutable identifiers as object references. That makes the information robust to name changes, but you may have a hard time deciphering it (unless you are NEO).
`HTML` | This special style modify the output to be a static HTML page, instead of the regular JSON data feed. This is mainly used for exploring the API in a friendly way (think of it as a custom swagger UI). The HTML pages are actually good old forms that you can use also to input parameters and quickly see the result. It also provides some additional shortcut links to related queries you may need.

### Envelope parameter

This parameter defines the content of the reply. You can use it to further customize what information you need, from a full reply including the explicit query as it is interpreted by the API, reply meta-information, header and data; or you can just ask for the data.

Envelope&nbsp;Value | Table Layout
---------- | ------------
`ALL`  | This the default value. It will return the full reply, including the original query as evaluated by the API (so it can be differ from the query you send, because it will list all the implicit settings and overrides), and the complete result, consisting of meta informations, the table header and the data.
`RESULT` | This setting will only return the result part
`DATA` | This setting will only return the data with no additional information.


# Model API

The Model API allows you to configure the meta-model, including project creation, customization of domains, creation of dimensions and metrics, configuring hierarchies and indexing, etc...

## Dimensions

### Managing Dimensions

These APIs allow to manage the Dimensions definition:

API | Description
--- | -----------
`GET http://yourserverdomain/v4.2/rs/projects/{projectId}/domais/{domainId}/dimensions` | Get all dimensions for the domain
`POST http://yourserverdomain/v4.2/rs/projects/{projectId}/domais/{domainId}/dimensions` | Create a new dimension in the domain
`GET http://yourserverdomain/v4.2/rs/projects/{projectId}/domais/{domainId}/dimensions/{dimensionId}` | Get a dimension definition
`PUT http://yourserverdomain/v4.2/rs/projects/{projectId}/domais/{domainId}/dimensions/{dimensionId}` | Modify a dimension definition
`DEL http://yourserverdomain/v4.2/rs/projects/{projectId}/domais/{domainId}/dimensions/{dimensionId}` | Delete a dimension definition

### Dimension Definition

Attribute | Description
--------- | -----------
id | the dimension identifier
name | the dimension name
type | the dimension type: `INDEX`, `CATEGORICAL`, `CONTINUOUS`
expression | the dimension expression, as defined in the domain's scope
parentId | if the dimension is part of a hierarchy, this is the parent dimension Id
attributes | attributes are alternative values you can associate to a dimension member
options | defines optional behaviors to control what users can do

### Dimension Option

The Dimension Option controls how users are allowed tp interact with the dimension.

The option's attributes are the following:

Attribute | Description
--------- | -----------
`mandatorySelection` | when set to true, it is mandatory to include a filter on that dimension in any query. If no filter is defined on the dimension, the query will fail.
`singleSelection` | when set to true, user can only perform a single selection on that dimension
`unmodifiableSelection` | when set to true, the user cannot modify the dimension selection. In that case `defaultSelection` must be set.
`defaultSelection` | this is a constant expression that is evaluated to define the dimension default selection
`hidden` | when set to true, the dimension selection is not returned by the API
`groupFilter` | this is a list of user group IDs to apply the option on
`userFilter` | this is a list of user IDs to apply the option on

#### Restricting Option to specific users / group of users

It is possible to restrict an option to a set of users groups or even specific users.

This is done by using the `groupFilter` or `userFilter`. If none is specified, the option will apply to any user.

Note that you can combine this restriction with the `defaultSelection` in order to create a user specific selection, by using the `$'USER'` parameter in the expression definition.

#### Setting the default selection

The default selection must be a constant expression. You can use:
* any constant value of type String, Date or NUmeric
* most of the built-in functions, for example to work on date
* dimension level parameters (see following)

You can use the following parameters to define it:

Parameters | Description
---------- | -----------
`$'MIN'` `$'MAX'` | If the dimension is continuous, those parameters will resolve to the range boundaries
`$'USER'` | This parameter will resolve to the user context, where you can access user defined properties, e.g.: `$'USER'.'systemAccountID'`. This can be useful to create a filter that depends on the user running the queries

#### Examples

```shell
# add a simple default selection to a dimension and make it mandatory and single
curl -X PUT 
    --header 'Accept: application/json' 
    --header 'Authorization: Bearer TOKEN'
    "http://yourserverdomain/v4.2/rs/projects/DEMO/domains/COUNTRY/dimensions/NAME"
    --data {
        
```


# Management API

This is a set of APIs to manage the OB server.

# Expression Reference Guide

This sections describes how to write Expressions and the Functions available for you to use.

## Constant Expression

Constant type | Example
------------- | -------
Numerical   | 123.4
Text   | "Hello world"
Date   | DATE("DD/MM/YY")<br><i>note that you cannot change the date format when defining a constant</i>

<aside class="success">
String constants are enclosed using <b>double quotes</b>, be careful!
</aside>

## Referencing objects

An Expression allows you to create references to model objects and  perform operations on their underlying values.

To create a reference, you can use two syntax:

 * Shorthand
 * Explicit

>examples of referencing objects using identifiers

```Shell

// always enclose the identifier in single-quotes

// using a named identifier 
// without specifying the object type, 
// so the scope will manage the order
// in case of conflicts
'this is a identifier name'

// using an ID
@'someID'

// using a typed name, 
// in case of collision
[dimension:'this is a dimension']

// using a typed ID 
// only the ID is enclosed, not the type
@bookmark:'someBookmarkID'
```

Important: The resolution of an object identifier is case sensitive, and the identifier must be enclosed in simple quotes.

<aside class="success">
Identifiers must be enclosed between <b>single quotes</b>, be careful!
</aside>

Object type | Shorthand | Explicit
----------- | --------- | --------
Column | #'CUSTOMER_NAME' | [col:'CUSTOMER_NAME'] or @col:'CUSTOMER_NAME'
Object by name<br>(Domain, Dimension, Metric or Relation) | 'Name' | [dimension:'Name'] or @metric:'Name'<br><i>note: using the explicit form can be useful to resolve name collisions
Object by ID<br>(Domain, Dimension, Metric or Relation) | @'someID' | [id:'someID']
Parameter | $'M0' | [param:'M0']

## Composing Expression

If an Expression is an Object, it is usually possible to compose it with other Expressions as long as they are valid in the target object scope.

For example if the scope is the `Sales` domain, you can create the following Expression to resolve the country of the customer: 

    `'customer'.'address'.'country'.'name'`

You can also use the special Expression `$self` to reference the Domain itself, for example: 

    `'customer'.'address'`
## Naming expressions

Since version 4.2.2, Bouquet server allows us to give a name to Expressions. Just use the AS keyword in the expression editor:

'metric1' + 'metric2' AS 'sum_of_both_metrics'

This can be useful when using custom Expressions to compute an analysis. The resulting columns in the data-table will have the Expressions names. Note however that if you define a an Expression as a simple alias for a Dimension or a Metric, the Dimension or Metric given name will take precedence over the expression's name in the datatable.

## Operators

Operator | Type | Description
-------- | ---- | -----------
expr1 &#124;&#124; expr2 | Logical | OR logical operator
expr1 && expr2 | Logical | AND logical operator
expr1 LIKE expr2  |  Logical | Search for a specified pattern in a column
expr1 = expr2 | Comparison | Checks if expr1 is equal to expr2
expr1 > expr2 | Comparison | Greater comparator
expr1 >= expr2 | Comparison | Greater or equal comparator
expr1 < expr2 | Comparison | Less comparator
expr1 <= expr2 | Comparison | Less or equal comparator
expr1 != expr2 | Comparison | Not equal comparator
expr1 RLIKE expr2 | Comparison | Executes a "LIKE" comparison using a regexp expression
n1 % n2 | Math  |  Modulo operator
n1 + n2 | Math  |  Addition operator
n1 - n2 | Math  |  Subtraction operator
n1 *n2 | Math  |  Multiplication operator
n1 / n2 | Math  |  Division operator
n1 ** n2 | Math  |  Exponential operator (n1 to n2th power)

## Math functions

Function | Definition
-------- | ----------
ABS(n) | Absolute value of number
ACOS(n) | Arc cosine of n
ASIN(n) | Arc sine of n
ATAN(n) | Arc tangent of n
COS(n) | Cosine of n
CEIL(n) | Smallest integer value that is greater than or equal to a number.
COSH(n) | Hyperbolic cosine of n
DEGREES(n) | Argument converted to degrees of n
EXP(n1,n2) | Exponential (n1 to n2th power)
FLOOR(n) | Largest integer value that is less than or equal to a number.
GREATEST(expr1, expr2, ...) |Greatest value in a list of expressions
LEAST(expr1, expr2, ...) | Smallest value in a list of expressions
LN(n) | Natural Log of n, where n>0
LOG(n1,n2) | Log of n1, base n2
MINUS(n1,n2) | Difference between n1 and n2
PI() | PI number of n
POWER(n1,n2) | n1 raised to the n2th power.
RADIANS(n) | Argument converted to radians of n
RAND() |Generate a random value
ROUND(n,[p]) |Number n rounded to a given (optionnal) precision p. p can be negative, to cause p digits left of the decimal point of the value n to become zero.
SIGN(n) |Sign of a number (-1 or 1)
SIN(n) | Sine of n
SINH(n) | Hyperbolic Sine of n
SQRT(n) | Square root of n
SUBSTRACTION(n1,n2) | Substract n2 from n1
TAN(n) | Tangent of n
TANH(n) | Hyperbolic tangent of n
TRUNCATE(n1,n2) | n1 truncated to n2 decimal places.

##Date functions

Function | Definition
-------- | ----------
ADD_MONTHS(date,num_months) | Returns date + num_months (can be negative)
CURRENT_DATE() | Get the current system date
CURRENT_TIMESTAMP() | Get the current system timestamp
DAILY(timestamp) | Converts the timestamp into a date
DATE_ADD(date1,number,period) | Add to date1 the number of period (unit in SECOND,MINUTE,HOUR,DAY,MONTH,YEAR)<br>Note: Adding two dates is not possible
DATE_INTERVAL(date1,date2,period) | Computes the difference between date1 and date2 in a particular period (unit in SECOND,MINUTE,HOUR,DAY)
DATE_SUB(date1,date2) | Removes date2 from date1
DATE_SUB(date1,number,period) | Removes from date1 the number of period (unit in SECOND,MINUTE,HOUR,DAY,MONTH,YEAR)
DATE_TRUNCATE(date,format) | Truncates the date (it can be either a Date or a Timestamp) depending on the format constant string:<br>if format="day" then the result is the Date part of date (excluding the time)<br>if format="week" then the result is the first day of the week containing date<br>if format="month" then the result is the first day of the month containing date<br>if format="quarter" then the result is the first day of the quarter containing date<br>if format="year" then the result is the first day of the year containing date<br><br>Note: this is useful to perform weekly, monthly, quarterly or yearly analysis (you can also use the shortcut functions DAILY(), WEEKLY(), MONTHLY(), QUARTERLY() or YEARLY())
DAY(date) | Extracts the day of the month from date
DAY_OF_WEEK(date) | Extracts the day of the week date, between 1 and 7. Day 1 is sunday
DAY_OF_YEAR(date) | Extracts the day of the year from the date
FROM_EPOCH(n) | Returns a timestamp based on a numeric field representing the number of seconds since 01-01-1970.
HOUR(date) | Extracts the hour from date
MINUTE(date) | Extracts minutes from date
MONTHLY(date or timestamp) | Returns the first day of month from the date
MONTHS(date) | Extracts the month in the year from date
MONTHS_BETWEEN(date1,date2) | Returns the number of months between d1 and d2
QUARTERLY(date or timestamp) | Returns the first day of the quarter from the date
SECOND(date) | Extracts seconds from date
TO_DATE(timestamp) | Converts a timestamp to a date
TO_DATE(text, format) | Converts a text to a date with a specific format
TO_EPOCH(timestamp) | Returns a numeric value based on a timestamp field representing the number of seconds since 01-01-1970.
TO_TIMESTAMP(date) | Converts a timestamp to a date
TO_TIMESTAMP(text, format) | Converts a text to a timestamp with a specific format
WEEKLY(date or timestamp) | Returns the first day of the week from the date
YEAR(date) | Extracts the year from date
YEARLY(date or timestamp) | Returns the first day of the year from the date

## Text functions

Function | Definition
-------- | ----------
CONCAT(s1,s2) | Concatenates Text1 and text2
LENGTH(s) | Returns the length of a text
LOWER(s) | Returns the text to lower case
MD5(string) | Calculates the MD5 hash of string, returning the result in hexadecimal
POSITION(s1, s2) | Returns the position of s2 within s1 starting from index 1, 0 otherwise
REPLACE(text, from, to) | Replaces within the first text from text with to text
REVERSE(string) | Returns reversed string
SPLIT_PART(string, delimiter, position) | Splits string on delimiter and return the given field (counting from one)
SUBSTRING(text,n1 [,n2]) | Substring of s, starting from index n1 optionally ending at index n2
TRANSLATE(text, from, to) | Replaces within the first text each single char specified in from text with the corresponding char specified in to
TO_CHAR(number) | Converts a number to a text
TO_CHAR(date or timestamp, format) | Converts a date or timestamp to a text with a specific format. You can use any format supported by the underlying data-source. Check next § for a quick reference guide
UPPER(s) | Uppercase text

### Date Formatting

This is just a quick reference guide for usual formats. You can use any format supported by the underlying data-source. 

Format | Definition
------ | ----------
YYYY | 4 digits year, e.g. 2016
YYY  | 3 digits year, e.g. 016
YY  | 2 digits year, e.g. 16
Y  | 1 digit year, e.g. 6
MONTH | Month name (uppercase)
MON | abbreviated month name
MM | month number (01-12)
DAY | name of the day
DDD | Day of the year (1-366)
DD | Day of the month (1-31)
D | Day of week (1-7)

## Regex functions

Function | Available | Definition
-------- | --------- | ----------
REGEXP_COUNT(string, regexp) | RedShift only | Counts the number of occurrences of the regexp within the string
REGEXP_INSTR(string, regexp) | RedShift & Oracle | Returns the position of the first occurrence found of the searched regexp within the string
REGEXP_REPLACE(string, regexp, replace) | All but MySQL | Replaces every occurrences of the substring matching a POSIX regular expression
REGEXP_SUBSTR(string, regexp) | All but MySQL | Extracts the first substring matching POSIX regular expression

## JSON functions

These functions are only supported by the Redshift plugin.

Function | Definition
-------- | ----------
JSON_ARRAY_LENGTH ('json string') | Returns the number of elements in the outer array of a JSON string
JSON_EXTRACT_ARRAY_ELEMENT_TEXT('json string', pos) | Returns a JSON array element in the outermost array of a JSON string, using a zero-based index.
JSON_EXTRACT_PATH_TEXT(json string', 'path_elem' [,'path_elem'[, …]]) | Returns the value for the key:value pair referenced by a series of path elements in a JSON string. The JSON path can be nested up to five levels deep.

## Aggregate functions

Function | Definition
-------- | ----------
COUNT(*) or COUNT([DISTINCT] expr) | Count the number of rows returned or the number of rows returned by expr
AVG([DISTINCT] value expr) | Average value of ‘n' ignoring NULL
DISTINCT(expr) | Get distinct values returned by expr
MAX(value expr) | Maximum value within an expression
MEDIAN([DISTINCT] n) | Median of n, ignoring NULLs.
MIN(value expr) | Minimum value within an expression
SUM(value expr) | Sum an expression

VARIANCE([DISTINCT] n) | Variance of n, ignoring NULLs

### Windowing / Analytics functions

You can apply an operator on a group of rows using the windowing syntax:

 * Use the partition expression to partition the query result set into groups based on one or more value expression. If you omit this clause, then the function treats all rows of the query result set as a single group
 * Use the order by expression to specify how data is sorted within a partition

Function | Definition
-------- | ----------
AVG([DISTINCT] value expr, [partition expr], [order by expr]) | Average value of ‘n' ignoring NULL
DISTINCT(expr) | Get distinct values returned by expr
MAX(value expr, [partition expr], [order by expr]) | Maximum value within an expression
MEDIAN([DISTINCT] n) | Median of n, ignoring NULLs.
MIN(value expr, [partition expr], [order by expr]) | Minimum value within an expression
RANK([partition expr], [order by expr]) | Returns the rank of a value in apartition expression. Same values within a partition have the same rank and the partition can be ordered
ROW_NUMBER([partition expr], [order by expr]) | Returns a number to each row to which it is applied in a partition expression. The number returned is unique within its partition and the partition can be ordered
SUM(value expr, [partition expr], [order by expr]) | Sum an expression

## Numeric functions

Function | Definition
-------- | ----------
TO_INTEGER(expr) | Convert a text, or a numeric field to an integer
TO_NUMBER(expr) | Convert a text, an integer or a numeric field to a float
TO_NUMBER(expr,size,precision) | Convert a text, an integer or a numeric field to a specific numeric format

## Logical functions

Function | Definition
-------- | ----------
CASE(condition1,then1,...,[else]) | Group the data into sub-sets
ISNULL(expr) | Expr is null logical function
NOT(expr1, expr2) | Expr1 not equal to expr2 logical operator
NULLIF(expr1, expr2) | Returns the first expression if the two expressions aren't equal or a null value if the two expressions are equal

## Miscellaneous functions

Function | Definition
-------- | ----------
EXISTS(expr) | Checks that a sub condition is matched. The EXISTS condition is considered "to be met" if the subquery returns at least one row.
LPAD(text1,length,text2) | Left pad with the text2 the text1 to specified length
LTRIM(text,[character]) | Remove leading characters from the text. By default, the character to remove is a whitespace
RPAD(text1,length,text2) | Right pad with the text2 the text1 to specified length
RTRIM(text,[character]) | Remove trailing characters from the text. By default, the character to remove is a whitespace
TRIM(text, [character]) | Remove leading and trailing characters from the text. By default, the character to remove is a whitespace
