---
title: Popily API Reference

language_tabs:
  - python
  - javascript
  - json

toc_footers:
  - API Version 1.0
  - API Docs Version 1.0

includes:

search: true
---

# Introduction

Popily makes it it easy to embed customer-facing analytics into your application. We have client libraries in Python and JavaScript (more coming soon!) that interact with a REST API, so you can connect to your data, ask the API for the information you want, and drop cool interactive visualizations into your application -- all with just a few lines of code. 

# Quick start

Getting started is easy. Just send some data, pick a column to visualize, and embed the results. 

```python
from popily_api import Popily

popily = Popily('YOUR API KEY')

# Add a source
columns = [
    {
        'column_header': 'Column 1',
        'data_type': 'number'
    },
    {
        'column_header': 'Column 2',
        'data_type': 'category'
    }
]
source = popily.add_source('http://url-of-your-csv.csv', columns=columns)

# Get some insights
insights = popily.get_insights(source['id'])

# Embed the insight with an iframe
embed_url = insights['results'][0]['embed_url']
```
```javascript
var popily = new Popily('YOUR API KEY')

// Add a source
columns = [
    {
        'column_header': 'Column 1',
        'data_type': 'number'
    },
    {
        'column_header': 'Column 2',
        'data_type': 'category'
    }
]
popily.addSource('http://url-of-your-csv.csv', {columns:columns}, function(err, source) {
    console.log(source); 
});

// Get some insights
popily.get_insights(source.id, {}, function(err, insights) {
    var iframe = document.createElement('iframe');
    iframe.src = insights.results[0].embed_url;
    document.body.appendChild(iframe);     
});
```

# Concepts

We have four big concepts in Popily that you'll use throughout your journey with our API.

1. Data sources - Wherever your data comes from (a database, CSV file, etc).
2. Columns - Each source has columns, which are like database columns. (If you are working with JSON, we'll flatten the properties into something we can treat like columns)
3. Insights - A relationship between columns. Like average employee salary over time.
4. Filters - A "slide" of the data (like a `WHERE` clause in a SQL query). Like average salary for employee John Smith over time.

With that out of the way, let's get started.

# Authentication

You authenticate each request an API key. Soon you'll get an API get as soon as you create your account. For now though, you need to email us at awesome@popily.com to as for one.

## API key

```python
from popily_api import Popily
popily = Popily('YOUR API KEY')
```
```javascript
// Node.js
var popily = require('popily')('YOUR API TOKEN');

// With the browser client
var popily = new Popily('YOUR API KEY');
```
You authenticate with Popily using a token passed in the Authorization HTTP header. If you haven't been assigned a token, just contact us and we'll hook you up. **We strongly recommend keeping this token to yourself.**

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

`GET` arguments must be passed as params and `POST` arguments must be passed as JSON with correct Content-Type headers. Most of the time you won't need to worry about any of this, as the actual request formatting is handled by the client library in your language of choice. 

**If you want to use a language we haven't written a client for yet, let us know!** We'd love to help, either by supporting you in building an API client, or creating it ourselves.

## Pagination

Popily uses pagination when `GET` requests returns a list of objects (e.g. insights, sources). To get all objects in a list of objects, you will need to paginate through the list. Every list response has a `prev` and `next` property just for this purposes. For example:

```JSON
{
  // list of response objects
  "results": [], 
  "next": "https://popily.com/path/to/next/",
  "previous": "https://popily.com/path/to/prev/"
}
```

# Sources

The `sources` endpoint can be used to create new data sources or interact with existing sources. Each `source` is associated with a `user` by its `created_by` property. By default each `source` you create is associated with you, but you can also assign `source` objects to `user` objects you create. 

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
// Node.js
var popily = require('popily')('YOUR API TOKEN');

// Browser
var popily = new Popily('YOUR API TOKEN');

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

// Example connection to a database column called 'characters'
sourceData['connection_string'] = 'mysql://username:password@host:port/database';
sourceData['query'] = 'SELECT * FROM characters';

popily.addSource(sourceData, function(err, source) {
    // do something with the source
});
```

As soon as you add a `source`, Popily will automatically start performing the calculations required to generate lots of `insight` objects. **Note: sources are processed asynchronously**, so depending on the size of your data, this might take a few seconds to a few minutes. You can browse your `source` via the API (more on that below), or through the user interface at [popily.com](https://popily.com). 

Once you add a source you can tell Popily to refresh the data on a schedule (like every hour or every day), so the charts you embed in your application stay up to date.

Parameter | Format | Description
--------- | ------- | -----------
url | string | Full URL where the data file or API endpoint can be accessed.
title | string | Human-readable name of the source.
columns | JSON object | [A list of column headers and their data types](#data-types).
sheetname | string | Used when the data source is an Excel file. Denotes the name of the sheet to use.
connection_string | string | Full connection string to your MySQL or Postgres database.
query | string | The SQL query you'd like to run on your database.

## Sending files

```python
import popily

popily_api = popily.Popily('YOUR API TOKEN')

your_file = open('/path/to/your/file.csv')
source = popily_api.add_source(file_obj=your_file)
```
```javascript
// Node.js 
// Note file upload isn't allowed from the browser
var popily = require('popily')('YOUR API TOKEN');

filePath = '/path/to/your/file.csv'
var sourceData = {
  data: filePath
};

popily.addSource(sourceData, function(err, source) {
    // do something with the source
});
```

In addition to creating data `sources` from file URLs and databases, you can also upload files directly to Popily. 

## Source data types

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

When a `source` is added to Popily, our "Mechanical [Tukey](https://en.wikipedia.org/wiki/John_Tukey)" technology starts processing the data. You can check the status of any `source` by retrieving it from the API.

```python
from popily_api import Popily
popily = Popily('YOUR TOKEN')
# Get by slug
popily.get_source('your-source-slug')
# or get by id
popily.get_source(12345)
```
```javascript
// Node.js
var popily = require('popily')('YOUR API TOKEN');

// Browser
var popily = new Popily('YOUR API TOKEN');

// Get by slug
popily.getSource('your-source-slug')
// or get by id
popily.getSource(12345)
```

Status | Description
--------- | -------
inspecting | Examining new source.
calculating | Processing new source.
reinspecting | Reprocessing source after change.
Adding | Merging a source.
error | Something went wrong.

## Exploring your data sources

> Example API response after creating a new source

```JSON
{
    "id": 432,
    "explore_path": "/explore/source/dc-wikia-data-csv-3/discoveries/"
}
```

We’re exploring this data through the API, but once a data `source` has been added, you can also explore it through the Popily user interface. In the API response after you create a new data source, you’ll see an `explore_path` property, which is where you’ll find your new source via the interface.

## List sources

```python
from popily_api import Popily
popily = Popily('YOUR TOKEN')

# Get source list
popily.get_sources()
```
```javascript
// Node.js
var popily = require('popily')('YOUR API TOKEN');

// Browser
var popily = new Popily('YOUR API TOKEN');

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

Of course if you prefer the API, you can explore the columns and `insight` objects associated with your `source` that way as well. If you'd like to get a list of all your `source` objects, here's how:

### API response objects
- **count:** Number of source resources
- **next:** Pagination marker
- **previous:** Pagination marker
- **results:** (see [get_source](#get_source) for full explanation)

# Insights

Our Mechanical Tukey engine performs a bunch of calculations that represent the relationships between columns in your `source`. In Popily, each of these relationships is called an `insight`.

## List insights

You can get lists of `insight` objects for a given `source`.

```python
from popily_api import Popily

popily = Popily('YOUR TOKEN')

# Get all insights for a source by its id
popily.get_insights(324442)

# Get all insights for a source by its slug
popily.get_insights('some-source-slug')
```
```javascript
// Node.js
var popily = require('popily')('YOUR API TOKEN');

// Browser
var popily = new Popily('YOUR API TOKEN');

// Get all insights for a source by its id
popily.getInsights(324442, {}, function(err, insights) {
    // insights
});

// Get all insights for a source by its slug
popily.getInsights('some-source-slug', {}, function(err, insights) {
    // insights
});
```

> Example API response

```JSON
{
    "count": 30,
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
    "next": "https://popily.com/api/insights/?source=429&page=2"
}
```

Retrieves a list of insights for a source resource.

Parameter | Format | Description
--------- | ------- | -----------
source | integer or string | id or slug for the source.
columns | list | List of column names in a source.
insight_actions | list | The type of analysis used in the insights.

## Selecting specific column relationships

Most of the time you'll want to filter your list of `insights` to those that represent the relationship between specific columns of interest.

```python
from popily_api import Popily

popily = Popily('YOUR TOKEN')

# Get all insights that represent the relationship between Column A and Column B
columns = [
  'Column A',
  'Column B'
]
popily.get_insights('some-source-slug', columns=columns)
```
```javascript
// Node.js
var popily = require('popily')('YOUR API TOKEN');

// Browser
var popily = new Popily('YOUR API TOKEN');

var columns = [
  'Column A',
  'Column B'
];

// Get all insights that represent the relationship between Column A and Column B
popily.getInsights('some-source-slug', {columns: columns}, function(err, insights) {
    // insights
});
```

## Limiting by analysis

You can further limit your list of `insight` objects by the type of analysis Popily performed. We call this analysis an `insight_action`.

An `insight_action` is an aggregation (like a count, sum or average) or other analysis (like a comparison) performed for a particular column combination. 

Parameter | Description
--------- | -----------
average | Average of a number
sum | Total of a number
count | Number of rows, used for categories and dates
ratio | Percentage of rows, used for categories and dates
comparison | Compares two numbers, produces scatterplots
geo | Geospatial information, produces maps


```python
from popily_api import Popily

popily = Popily('YOUR TOKEN')

# Get all insights that represent the relationship between Column A and Column B
columns = [
  'Column A',
  'Column B'
]

# Let's say Column A is a number. We only want insights that look at the average
# of Column A.
insight_actions = [
  'average'
]

popily.get_insights('some-source-slug', columns=columns, average=average)
```
```javascript
// Node.js
var popily = require('popily')('YOUR API TOKEN');

// Browser
var popily = new Popily('YOUR API TOKEN');

// Get all insights that represent the relationship between Column A and Column B
var columns = [
  'Column A',
  'Column B'
];

// Let's say Column A is a number. We only want insights that look at the average
// of Column A.
var insightActions = [
  'average'
];

var params = {columns: columns, insight_actions: insightActions};
popily.getInsights('some-source-slug', params, function(err, insights) {
    // insights
});
```

## Filtering your data

You can also request "slices" of the returned `insight` objects by applying filters to the columns you request from the API. 

This is a powerful feature that gives you fine-grained control over how your data is presented. At the moment filters are limited to category columns, but coming soon we'll have support for datetime and number columns as well -- for example filtering only to show only values that fall between March 1, 2016 and March 7, 2016 or filtering to show only values that are greater than 0. 

The format of a filter is always the same:

`{ column: 'column name', values: ['value 1', 'value 2'], op: 'eq' }`

The `op` property stands for "operation" and is optional. It defaults to 'eq' (which stands for "equals"). Support for additional operations like 'gt' (greater than), 'lt' (less than), etc are coming soon.

```python
from popily_api import Popily

popily = Popily('YOUR TOKEN')

# Get all insights that represent the relationship between Column A and Column B
columns = [
  'Column A',
  'Column B'
]

# Let's say Column A is a number. We only want insights that look at the average
# of Column A.
insight_actions = [
  'average'
]

# Let's say Column B is a category and it's values are colors. We only want to 
# show values associated with red and green. 
filters = [
  {
    'column': 'Column B',
    'values': ['red', 'green']
  }
]

popily.get_insights('some-source-slug', columns=columns, average=average)
```
```javascript
// Node.js
var popily = require('popily')('YOUR API TOKEN');

// Browser
var popily = new Popily('YOUR API TOKEN');

// Get all insights that represent the relationship between Column A and Column B
var columns = [
  'Column A',
  'Column B'
];

// Let's say Column A is a number. We only want insights that look at the average
// of Column A.
var insightActions = [
  'average'
];

// Let's say Column B is a category and it's values are colors. We only want to 
// show values associated with red and green. 
var filters = [
  {
    'column': 'Column B',
    'values': ['red', 'green']
  }
];

var params = {
    columns: columns, 
    insight_actions: insightActions,
    filters: filters
};

popily.getInsights('some-source-slug', params, function(err, insights) {
    // insights
});
```

## Getting single insights

Sometimes you just want one `insight`, not a list. This can be achieved in two ways. The first builds on the list examples from above. There is only one `insight_action` (aka type of analysis) performed on any given combination of columns. So, say you had two columns, Column A, a number, and Column B, a category. Popily will calculate the average of Column A for each category in Column B. If you ask for these two columns with the `insight_action` of "average," you can be sure you'll get one result.

In this situation, you can include `single` in your request to let Popily know that you only want one object, not a list. **Note**: you can add `single` to any request, even if you're not sure that request will only return one object. If more than one `insight` would normally be returned, Popily will choose the one it thinks is most interesting. 

```python
from popily_api import Popily

popily = Popily('YOUR TOKEN')

# Get all insights that represent the relationship between Column A and Column B
columns = [
  'Column A',
  'Column B'
]

# Let's say Column A is a number. We only want insights that look at the average
# of Column A.
insight_actions = [
  'average'
]

popily.get_insights('some-source-slug', columns=columns, average=average, single=True)
```
```javascript
// Node.js
var popily = require('popily')('YOUR API TOKEN');

// Browser
var popily = new Popily('YOUR API TOKEN');

// Get all insights that represent the relationship between Column A and Column B
var columns = [
  'Column A',
  'Column B'
];

// Let's say Column A is a number. We only want insights that look at the average
// of Column A.
var insightActions = [
  'average'
];

var params = {
    columns: columns, 
    insight_actions: insightActions,
    single: true
};

popily.getInsights('some-source-slug', params, function(err, insights) {
    // insights
});
```

Of course, just like with `source` objects, you can also retrieve `insight` objects by their slug or id. The slug can be found in the URL of the `insight` when viewed through the UI. It's usually something like "title-of-insight-1."

```python
import popily_api

popily = Popily('YOUR API KEY')

# Get by id
popily.get_insight(249985)

# Get by slug
popily.get_insight('some-slug')
```
```javascript
// Node.js
var popily = require('popily')('YOUR API TOKEN');

// Browser
var popily = new Popily('YOUR API KEY')


// Get insight by id
popily.getInsight(249985, {}, function(err, insight) {
  console.log(insight);
});

// Get insight by slug
popily.getInsight('some-slug', {}, function(err, insight) {
  console.log(insight);
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

Parameter | Format | Description 
--------- | ------- | ----------- 
id or slug | string | Insight unique id
height | integer| Height in pixels (for displaying on a web page)
width | integer | Width in pixels (for display on a web page)
filters | JSON object | Filters applied to insight. See [insight filters](#filtering-chart-data)
full | string | If included will return x, y and z values

## Displaying insights

Once you have retrieved an insight, you can add it to your web application by dropping it in an iframe. **Note**: a JavaScript library that you can use to attach responsive visualizations directly to your DOM elements is coming soon. 

```html
<iframe frameborder="0" height="500" width="500" src="https://popily.com/widget/chart/hair-align-2/-/?style=compact-500-500&app_id=18&sig=63bb87883e20548076aefac271911c13&timestamp=1456114892"></iframe>
```

## Customize display

When you retrieve an `insight` you can also customize its display. Almost any aspect of a Popily visualization can be customized, including:

Property | Description 
--------- | -------
title | The title of the chart
x_label | Text along the x axis
y_label | Text along the y axis
category_order | Order of bars in a bar chart. Options are: auto, asc, dsc, a-z, z-a
time_interval | minute, hour, day, week, month, year

```python
import popily_api

popily = Popily('YOUR API KEY')

kwargs = {
  'title': 'Your title',
  'x_label': 'Your X Label',
  'category_order': 'z-a'
}

popily.get_insight('some-slug', **kwargs)
```
```javascript
// Node.js
var popily = require('popily')('YOUR API TOKEN');

// Browser
var popily = new Popily('YOUR API KEY')

var options = {
  title: 'Your title',
  x_label: 'Your x label',
  category_order: 'z-a'
};

// Get insight by slug
popily.getInsight('some-slug', options, function(err, insight) {
  console.log(insight);
});
```

## Filtering insight data

Just like with lists of `insight` objects, you can filter the data of a single `insight`. See the [Filtering your data](#filtering-your-data) section for details.

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

# API Clients

## Python

You can check out the official Popily client on [pypi](https://pypi.python.org/pypi/popily-api/0.0.7) and [GitHub](https://github.com/popily/popily-api).

## JavaScript

We've written JavaScript clients for Node.js and the browser. You can check out the official Popily Node.js client on [npm](https://www.npmjs.com/package/popily) and [GitHub](https://github.com/popily/popily-node), and the browser client on [GitHub](https://github.com/popily/popily-js).

# Getting help

We love developers because we _are_ developers, so we made sure you can get help in the ways you want:

- **Slack:** Ask a question in [our slack community room](https://gentle-shore-82359.herokuapp.com/)
- **Twitter:** [@popilyteam](https://twitter.com/popilyteam), [@jonathonmorgan](https://twitter.com/jonathonmorgan), [@chrisalbon](https://twitter.com/chrisalbon)
- **Email:** &#97;&#119;&#101;&#115;&#111;&#109;&#101;&#64;&#112;&#111;&#112;&#105;&#108;&#121;&#46;&#99;&#111;&#109;, &#106;&#111;&#110;&#97;&#116;&#104;&#111;&#110;&#64;&#112;&#111;&#112;&#105;&#108;&#121;&#46;&#99;&#111;&#109;, &#99;&#104;&#114;&#105;&#115;&#64;&#112;&#111;&#112;&#105;&#108;&#121;&#46;&#99;&#111;&#109;

Code on!
