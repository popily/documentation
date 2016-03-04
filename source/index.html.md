---
title: Popily API Reference

language_tabs:
  - python
  - javascript
  - http

toc_footers:
  - API Version 1.0
  - API Docs Version 1.0

includes:

search: true
---

# Introduction

Popily provides a powerful REST API for integrating embedded analytics into your application.

This API reference provides information on interacting with Popily's API endpoints.

# Authentication

Popily's API supports authentication through an API key. [API keys](#api-key) can be obtained through an account with Popily.

## API key

```python
import requests
r = requests.get('https://popily.com/api/sources/',
                    headers={'Authorization': 'Token ' + YOUR TOKEN})
```

You authenticate with Popily using a token passed in the Authorization HTTP header. If you haven't been assigned a token, please contact us and we'll hook you up.

Authentication includes an HTTP header `Authorization: Token 12345`, so Popily knows who is making the request. We strongly recommend keeping this token to yourself, so think twice about making requests directly from your front-end code (even though that is technically possible).

# Interacting with the API

## Status codes

Status Code | Description
--------- | -------
200 | OK
201 | Resource Created
204 | No Content
304 | Not Modified
400 | Bad Request
401 | Unauthorized
403 | Forbidden
404 | Not Found
429 | Too Many Requests
500 | Internal Server Error
503 | Service Unavailable

## Making requests

Popily follows RESTful API patterns.

- `GET` - Retrieve resources
- `POST` - Create new resources
- `PUT` - Modify existing resources
- `DELETE` - Remove resources

`GET` arguments must be passed as params and `POST` arguments must be passed as JSON with correct Content-Type headers.

## Pagination

Popily uses pagination when `GET` requests returns a list of objects (e.g. insights, sources). To get all objects in a paginated list of objects, you will need to paginate through the list of objects.

# Sources endpoint

The sources endpoint can be used to create new data sources or interact with existing sources. Resources in this endpoint are bound to user accounts for the sake of authentication.

## Create source

```python
import popily

popily_api = popily.Popily('YOUR API TOKEN')

# Give it a title (optional)
title = 'DC Comics Data'

# Define the column types
columns = [
    {
        'column_header': 'name',
        'data_type': 'rowlabel'
    },
    {
        'column_header': 'ALIGN',
        'data_type': 'category'
    },
    {
        'column_header': 'EYE',
        'data_type': 'category'
    },
    {
        'column_header': 'HAIR',
        'data_type': 'category'
    },
    {
        'column_header': 'SEX',
        'data_type': 'category'
    }
]

# Example connection to a database column called 'characters'
connection_string = 'mysql://username:password@host:port/database'
query = 'SELECT * FROM characters'
source = popily_api.add_source(
                        connection_string=connection_string,
                        query=query, columns=columns,
                        title=title)
```
```javascript
var popily = require('popily')('YOUR API TOKEN');

var sourceData = {};

// Give it a title (optional)
sourceData['title'] = 'DC Comics Data';

// Define the column types
sourceData['columns'] = [
    {
        'column_header': 'name',
        'data_type': 'rowlabel'
    },
    {
        'column_header': 'ALIGN',
        'data_type': 'category'
    },
    {
        'column_header': 'EYE',
        'data_type': 'category'
    },
    {
        'column_header': 'HAIR',
        'data_type': 'category'
    },
    {
        'column_header': 'SEX',
        'data_type': 'category'
    }
];

# Example connection to a database column called 'characters'
sourceData['connection_string'] = 'mysql://username:password@host:port/database';
sourceData['query'] = 'SELECT * FROM characters';

popily.addSource(sourceData, function(err, source) {
    // process the source
});

```

A (data) source resource is data a user has added to Popily and can be populated from a file, a database connection, or a JSON API. When you add a source to Popily, we'll automatically create a resource, inspect the resource, perform some calculations, and make a sample space of insight resources that you can browse in the application UI (or retrieve via the insights endpoint).

Parameter | Format | Description
--------- | ------- | -----------
url | string | Full URL where the data file or API endpoint can be accessed.
title | string | Human-readable name of the source.
columns | JSON object | [A list of column headers and their data types](#data-types).
type | string | The type of data store providing data to the source: _e_csv_ (CSV file accessed via URL) _e_xls_ (Excel file accessed via URL) _e_json_ (JSON accessed via URL) _sql_ (results from SQL query). When absent, type is inferred.
sheetname | string | Used when the data source is an Excel file. Denotes the name of the sheet to use.
connection_string | string | Full connection string to your MySQL or Postgres database.
query | string | The SQL query you'd like to run on your database.

## Source data types

> Here is an example of a JSON object for creating a new source. Note that the JSON object contains column header and column data type for every row in the data source.

```JSON
{
    "url": "https://example.com/sales.csv",
    "title": "Sales Data",
    "columns": [
        {
            "column_header": "Date",
            "data_type": "datetime"
        },
        {
            "column_header": "Manager",
            "data_type": "category"
        },
        {
            "column_header": "Store ID",
            "data_type": "rowlabel"
        },
        {
            "column_header": "Sales",
            "data_type": "currency"
        },
    ]
}
```

All source resources in Popily must contain a description of the type of data in each column. Popily understands more than just integers and strings, it can also work with higher-level concepts like categories, US states or currency. We’re always adding to this list, but these are the available types as of today.

Note: You only need to define the columns you care about. Popily will ignore everything else.

Data Type | Description | Example
--------- | ------- | -----------
category | A column containing categorical values. | Apple, Orange, Pear
numeric | A column of numeric values. | 320, 2, 23.10
currency | A column of numeric values as currencies. | $320, $2, $23.10
datetime | A column of datetime or timestamps. | 12/12/2015, 2/3/15, January 6, 2015
country | A column of countries. | Spain, France, England
state | A column of US states. | Arizona, Maine, Oregon
coordinate | A column of latitude and longitude coordinate pairs. | [43.522, 23.521]
zipcode | A column of US Zip codes. | 42351
rowlabel | A column of non-categorical uniquely or semi-uniquely identifying each individual row | Steve Jobs, Dunder Muffin, Wendy Jones

## Source status

When a source resource is added to Popily, our Mechanical Tukey technology starts processing the data. You can check the status of any source resource by looking at the source status:

Status | Description
--------- | -------
inspecting | Examining new source.
calculating | Processing new source.
reinspecting | Reprocessing source after change.
Adding | Merging a source.
error | Something went wrong.

## Explore source

> Example API response after creating a new source

```JSON
{
    "id": 432,
    "explore_path": "/explore/source/dc-wikia-data-csv-3/discoveries/"
}
```

We’re exploring this data through the API, but once a data source has been added, you can also explore it through the Popily user interface. In the API response after you create a new data source, you’ll see an explore_path property, which is where you’ll find your new source via the interface.

## List sources

```python
import popily_api

# Get source list
popily_api.get_sources()
```
```javascript
var popily = require('popily')('YOUR API TOKEN');

// Get source list
popily.getSources(function(err, sources) {
  if(err)
    throw new Error(err);
  sources.results.forEach(function(err, source) {
    // source
  });
});
```

> Example API response

```JSON
{"count": 1,
 "next": "http://popily.com/api/sources/?page=2",
 "previous": None,
 "results": [{"columns": [{"column_header": "SEX",
             "data_type": "category",
             "id": 29413,
             "score": 0.0},
            {"column_header": "HAIR",
             "data_type": "category",
             "id": 29412,
             "score": 0.0},
            {"column_header": "EYE",
             "data_type": "category",
             "id": 29411,
             "score": 0.0},
            {"column_header": "ALIGN",
             "data_type": "category",
             "id": 29410,
             "score": 0.0},
            {"column_header": "name",
             "data_type": "rowlabel",
             "id": 29409,
             "score": 0.0}],
             "created_by": 3,
             "description": None,
             "explore_path": "/explore/source/dc-wikia-data-3/discoveries/",
             "id": 2287,
             "sheetname": None,
             "slug": "dc-wikia-data-3",
             "status": "inspecting",
             "title": "DC Comics Data",
             "url": None}]
}
```

Lists all source resources created by a user.

### API response objects
- **count:** Number of source resources
- **next:** Pagination marker
- **previous:** Pagination marker
- **results:** (see [get_source](#get_source) for full explanation)

# Insights endpoint

When `source` resources are created in Popily, our Mechanical Turk technology creates a large sample space of relationships between columns. In Popily, each of these relationships is called an `insight` and is represented by a `chart`.

For most users, the workflow will be:

1. [Retrieve the insight](#get-insight) for the desired relationship(s)
2. [Customize the insight's chart](#customize-chart) (e.g. removing values, datetime grouping)

## List insights

```python
import popily_api

# Get insights about Column A and Column B from sourcev 34242
popily.get_insights(324442, columns=['Column A','Column B'])
```
```javascript
var popily = require('popily')('YOUR API TOKEN');
popily.getInsights(324442, {
  'columns': ['Column A','Column B']
  }, function(err, insights) {
    // insights
  });

```

> Example API response

```JSON
{
    "count": 1,
    "previous": null,
    "results": [{
        "title": "Number of ALIGN grouped by SEX",
        "x_label": "ALIGN",
        "y_label": "Number of Records",
        "filters_key": "",
        "source": 429,
        "insight_type_category": "count",
        "id": 249985,
        "z_label": "",
        "insight_type": "count_by_category_by_category"
    }],
    "next": null
}
```

Retrieves a list of insights for a source resource.

Parameter | Format | Description
--------- | ------- | -----------
source | integer | id for the source.
columns | list | List of column names in a source.
insight_types | list | The type of analysis used in the insights.

## Get insight

```python
import popily_api

# Get insight 249985
popily.get_insight(249985,height=500,width=500)
```
```javascript
var popily = require('popily')('YOUR API TOKEN');

var filters = [];

// Get insight 249985
popily.getInsight(249985, {filters: filters}, {
  height: 500,
  width: 500
  }, function(err, insight) {
    var iframe = document.createElement('iframe');
    iframe.src = insight.embed_url;
  });
```

> Example API response

```JSON
{
    "embed_url": "https://popily.com/widget/chart/align-hair/-/?style=compact-500-500&app_id=18&sig=8b7c8e557a9ab1bcf4d5d762dda773d2&timestamp=1456113107",
    "x_label": "ALIGN",
    "y_label": "Number of Records",
    "filters_key": "",
    "source": 429,
    "title": "Number of ALIGN grouped by SEX",
    "insight_type_category": "count",
    "id": 249985,
    "z_label": "",
    "insight_type": "count_by_category_by_category"
}
```

Retrieves details on a single insight.

Parameter | Format | Default | Description
--------- | ------- | ----------- | -----------
id | string | Insight UID
height | integer| Height in pixels
width | integer | Width in pixels
filters | JSON object | Data filters applied to insight. See [insight filters](#filtering-chart-data).

If you want to visualize the data yourself you can get the x, y and z values (see the insight documentation for more details), or you can use the charts Popily generates for you via the insight’s embed_url property. Just drop it in an iframe:

`<iframe frameborder="0" height="500" width="500" src="https://popily.com/widget/chart/hair-align-2/-/?style=compact-500-500&app_id=18&sig=63bb87883e20548076aefac271911c13&timestamp=1456114892"></iframe>`


## Customize chart

```python
import popily_api

# Customize insight 249985
popily.customize_insight(249985,
                         title='This Chart is Awesome',
                         x_label='Goodness',
                         y_label='Genders',
                         category_order='z-a')
```
```javascript
var popily = require('popily')('YOUR API TOKEN');

// Customize insight 249985
var filters = [];

popily.customize_insight(249985, {filters: filters}, {
         title: 'This Chart is Awesome',
         x_label: 'Goodness',
         y_label: 'Genders',
         category_order: 'z-a'
 }, function(err, insight) {
 });
 
```

Customize the display of a single insight. Note, currently customizations are bound to an insight and thus an insight can only have a single customization.

Parameter | Format | Description
--------- | ------- | -----------
title | string | Custom chart title
x_label | string | Custom x label
y_label | string | Custom y label
category_order | string | Custom ordering of categories. Allowed values: 'a-z', 'z-a', 'asc', and 'dsc'

## Filtering chart data

```python
import popily_api

filters = [
    {
        'column': 'SEX',
        'values': ['Genderless Characters','Transgender Characters']
    }
]

insight = popily.get_insight(249985, filters=filters)
```
```javascript
var popily = require('popily')('YOUR API TOKEN');

var filters = [
    {
        'column': 'SEX',
        'values': ['Genderless Characters','Transgender Characters']
    }
];

popily.getInsight(249985, {filters: filters}, function(err, insight) {
  // here process insight
});

```


Sometimes you want to tell a more specific story. With Popily you can apply filters to any insight or list or insights, and then assign customizations that are specific to the filters you’ve applied. Filters allow you to control the data within a relationship insight.

Example filter param to include only 'Genderless Characters' and 'Transgender Characters' in a column titled `SEX`:

`[
    {
        'column': 'SEX',
        'values': ['Genderless Characters','Transgender Characters']
    }
]`


# Users endpoint

You can create user accounts to associate source and insight resources with specific users of your application. Your authentication token allows you to access the source and insight objects of any users you create.

## Create users

```python
import requests

r = requests.post('https://popily.com/api/users/',
                    json={'username':'testing'},
                    headers={'Authorization': 'Token ' + <YOUR API TOKEN>})
```
Create a user resource.

Parameter | Format | Description
--------- | ------- | -----------
json | JSON object | Arguments for user creation.

Arguments are passed via a json object:

- **username:** Username
- **email:** Email address. Optional for API, required for Popily.com.
- **groups_member:** List of groups to which the user belongs. Any user you create automatically belongs to your default group.

## List users

```python
r = requests.get('https://popily.com/api/users/',
                  headers={'Authorization': 'Token ' + <YOUR API TOKEN>})
```

List all user resources associated with an API token.

### API response objects

- **id:** UID of the User object.
- **username:** We'll force this to be unique when we create the User object.
- **email:** Email address. Optional for API, required for Popily.com.
- **groups_member:** List of groups to which the user belongs.

# Python client

We love Python so we couldn't help but make an official Popily client for Python. You can check it out on [pypi](https://pypi.python.org/pypi/popily-api/0.0.6) and [GitHub](https://github.com/popily/popily-api).

## Install client

```python
pip install popily-api
```
```javascript
npm install popily
```

Installing the Popily API client is easy with the package manager [pip](https://pypi.python.org/pypi/pip).

## Upgrade client

```python
pip install popily-api --upgrade
```
```javascript
npm install popily
```

Upgrading the python client can be accomplished with the `-upgrade` argument.

## add_source

```python

# Three examples of add_source

import popily_api

popily = popily_api.Popily('YOUR API TOKEN')

# Give it a title (optional)
title = 'DC Comics Data'

# Define the column types
columns = [
    {
        'column_header': 'name',
        'data_type': 'rowlabel'
    },
    {
        'column_header': 'HAIR',
        'data_type': 'category'
    },
]

# If the data is in a CSV file locally
file_data = {'data': open('/path/to/the/file.csv')}
source = popily.add_source(file_obj=file_data,
                                columns=columns, title=title)

# If the dataset is remote
url = 'https://example.com/dataset.csv'
source = popily.add_source(url=url,
                                columns=columns, title=title)

# If the data is from an SQL query
connection_string = 'mysql://username:password@host:port/database'
query = 'SELECT * FROM TableA'
source = popily.add_source(
                        connection_string=connection_string,
                        query=query, columns=columns,
                        title=title)
```
```javascript

// Three examples of add_source

var popily = require('popily')('YOUR API TOKEN');

# Give it a title (optional)
var title = 'DC Comics Data';

# Define the column types
var columns = [
    {
        'column_header': 'name',
        'data_type': 'rowlabel'
    },
    {
        'column_header': 'HAIR',
        'data_type': 'category'
    },
]

# If the data is in a CSV file locally
var filePath = {'data': '/path/to/the/file.csv'}
popily.addSource({
  title: title
  columns: columns, 
  data: filePath
  }, function(err, source) {
    // ..
  });

# If the dataset is remote
var url = 'https://example.com/dataset.csv'
popily.addSource({
    title: title
    columns: columns, 
    url: url
  }, function(err, source) {
    // ..
  });

# If the data is from an SQL query
var connectionString = 'mysql://username:password@host:port/database'
var query = 'SELECT * FROM TableA'
popily.addSource({
  title: title
  columns: columns, 
  connection_string: connectionString,
  query: query
  }, function(err, source) {
    // ..
  });

```

Creates a new data source resource in Popily.

Parameter | Format | Description
--------- | ------- | -----------
url | string | Full URL where the data file or API endpoint can be accessed.
title | string | Human-readable name of the source.
columns | JSON object | [A list of column headers and their data types](#data-types).
type | string | The type of data store providing data to the source: _e_csv_ (CSV file accessed via URL) _e_xls_ (Excel file accessed via URL) _e_json_ (JSON accessed via URL) _sql_ (results from SQL query). When absent, type is inferred.
sheetname | string | Used when the data source is an Excel file. Denotes the name of the sheet to use.
connection_string | string | Full connection string to your MySQL or Postgres database.
query | string | The SQL query you'd like to run on your database.

## get_sources

```python
# Get sources
sources = popily_api.get_sources()
```
```javascript
// Get sources
popily.getSources(function(err, sources) {
  // ..
});
```

> Example API response

```JSON
{"count": 1,
 "next": "http://popily.com/api/sources/?page=2",
 "previous": None,
 "results": [{"columns": [{"column_header": "SEX",
             "data_type": "category",
             "id": 29413,
             "score": 0.0},
            {"column_header": "HAIR",
             "data_type": "category",
             "id": 29412,
             "score": 0.0},
            {"column_header": "EYE",
             "data_type": "category",
             "id": 29411,
             "score": 0.0},
            {"column_header": "ALIGN",
             "data_type": "category",
             "id": 29410,
             "score": 0.0},
            {"column_header": "name",
             "data_type": "rowlabel",
             "id": 29409,
             "score": 0.0}],
             "created_by": 3,
             "description": None,
             "explore_path": "/explore/source/dc-wikia-data-3/discoveries/",
             "id": 2287,
             "sheetname": None,
             "slug": "dc-wikia-data-3",
             "status": "inspecting",
             "title": "DC Comics Data",
             "url": None}]
}
```

Lists all source resources created by a user.

### API response objects
- **count:** Number of source resources
- **next:** Pagination marker
- **previous:** Pagination marker
- **results:** (see [get_source](#get_source) for full explanation)

## get_source

```python
# Get source with id 34242
popily_api.get_source(34242)
```
```javascript
// Get source with id 34242
popily.getSource(34242, function(err, source) {
  // ..
})
```

> Example API response

```JSON
{
  "columns": [{"column_header": "HAIR",
              "data_type": "category",
              "id": 29382,
              "score": 0.183804993880959},
              {"column_header": "ALIGN",
               "data_type": "category",
               "id": 29380,
               "score": 0.17228792581526},
              {"column_header": "SEX",
               "data_type": "category",
               "id": 29383,
               "score": 0.157287131801953},
              {"column_header": "EYE",
               "data_type": "category",
               "id": 29381,
               "score": 0.113107718497747},
              {"column_header": "name",
               "data_type": "rowlabel",
               "id": 29379,
               "score": 0.0}],
  "created_by": 3,
  "description": None,
  "explore_path": u"/explore/source/dc-wikia-data/discoveries/",
  "id": 2279,
  "sheetname": None,
  "slug": "dc-wikia-data",
  "status": "finished",
  "title": "DC Comics Data",
  "url": None
}
```

Retrieves details on an individual source resource.

Parameter | Format | Description
--------- | ------- | -----------
source | integer | id for the source.

### API response objects
- **id:** Source UID
- **url:** Full URL where the data file or API endpoint can be accessed)
- **slug:** Unique slug of the source resource
- **title:** Human-readable name
- **description:** Human-readable description
- **columns:** List of columns
  - **column_header:** Title of column
  - **data_type:** Type of data in column ([see source data types](#source-data-types))
  - **id:** Column UID
  - **score:** "Interestingness" score (used in some rankings)
- **created_by:** UID of creating user
- **status:** Status of source resource
  - **inspecting:** Initial inspection
  - **calculating:** Processing underway
  - **reinspecting:** Secondary inspection
- **type:** Type of data used to create source resource
  - **e_csv:** CSV file accessed via URL
  - **e_xls:** Excel file accessed via URL
  - **e_json:** JSON accessed via URL
  - **sql:** Results from SQL query
- **sheetname:** Used when working from Excel files with more than one sheet
- **connection_string:** Full connection string to your MySQL or Postgres database
- **query:** SQL query you'd like to run on your database

## get_insights

```python
# Get insights about Column A and Column B from sourcev 34242
popily.get_insights(324442, columns=['Column A','Column B'])
```
```javascript
// Get insights about Column A and Column B from sourcev 34242
popily.getInsights(324442, {
    columns: ['Column A','Column B']
  }, function(err, insights) {
    // ..
  });
```

> Example API response

```JSON
{
    "count": 1,
    "previous": null,
    "results": [{
        "title": "Number of ALIGN grouped by SEX",
        "x_label": "ALIGN",
        "y_label": "Number of Records",
        "filters_key": "",
        "source": 429,
        "insight_type_category": "count",
        "id": 249985,
        "z_label": "",
        "insight_type": "count_by_category_by_category"
    }],
    "next": null
}
```

Retrieves a list of insights for a source resource.

Parameter | Format | Description
--------- | ------- | -----------
source | integer | id for the source.
columns | list | List of column names in a source.
insight_types | list | The type of analysis used in the insights.

## get_insight

```python
# Get insight 249985
popily.get_insight(249985,height=500,width=500)
```
```javascript
// Get insight 249985
popily.getInsight(249985, {}, {
  height: 500, 
  width=500
  }, function(err, insight) {
    // ,,
  });
```

> Example API response

```JSON
{
    "embed_url": "https://popily.com/widget/chart/align-hair/-/?style=compact-500-500&app_id=18&sig=8b7c8e557a9ab1bcf4d5d762dda773d2&timestamp=1456113107",
    "x_label": "ALIGN",
    "y_label": "Number of Records",
    "filters_key": "",
    "source": 429,
    "title": "Number of ALIGN grouped by SEX",
    "insight_type_category": "count",
    "id": 249985,
    "z_label": "",
    "insight_type": "count_by_category_by_category"
}
```

Retrieves details on a single insight.

Parameter | Format | Default | Description
--------- | ------- | ----------- | -----------
id | string | Insight UID
height | integer| Height in pixels
width | integer | Width in pixels
filters | JSON object | Data filters applied to insight. See [insight filters](#filtering-chart-data).

If you want to visualize the data yourself you can get the x, y and z values (see the insight documentation for more details), or you can use the charts Popily generates for you via the insight’s embed_url property. Just drop it in an iframe:

`<iframe frameborder="0" height="500" width="500" src="https://popily.com/widget/chart/hair-align-2/-/?style=compact-500-500&app_id=18&sig=63bb87883e20548076aefac271911c13&timestamp=1456114892"></iframe>`

## customize_insight

```python
# Customize insight 249985
popily.customize_insight(249985,
                         title='This Chart is Awesome',
                         x_label='Goodness',
                         y_label='Genders',
                         category_order='z-a')
```
```javascript
// Customize insight 249985
popily.customizeInsight(249985, {}, {
    title: 'This Chart is Awesome',
    x_label: 'Goodness',
    y_label: 'Genders',
    category_order: 'z-a'
  }, function(err, insight) {
    // ..
  });
```

Customize the display of a single insight. Note, currently customizations are bound to an insight and thus an insight can only have a single customization.

Parameter | Format | Description
--------- | ------- | -----------
title | string | Custom chart title
x_label | string | Custom x label
y_label | string | Custom y label
category_order | string | Custom ordering of categories. Allowed values: 'a-z', 'z-a', 'asc', and 'dsc'

# Getting help

We love developers because we _are_ developers, so we made sure you can get help in the ways you want:

- **Slack:** Ask a question in our slack community room
- **Twitter:** [@popilyteam](https://twitter.com/popilyteam), [@jonathonmorgan](https://twitter.com/jonathonmorgan), [@chrisalbon](https://twitter.com/chrisalbon)
- **Email:** &#97;&#119;&#101;&#115;&#111;&#109;&#101;&#64;&#112;&#111;&#112;&#105;&#108;&#121;&#46;&#99;&#111;&#109;, &#106;&#111;&#110;&#97;&#116;&#104;&#111;&#110;&#64;&#112;&#111;&#112;&#105;&#108;&#121;&#46;&#99;&#111;&#109;, &#99;&#104;&#114;&#105;&#115;&#64;&#112;&#111;&#112;&#105;&#108;&#121;&#46;&#99;&#111;&#109;

Code on!
