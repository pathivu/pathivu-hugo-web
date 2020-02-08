---
title: 'Query'
date: 2019-12-29T22:42:33+05:30
weight: 2
summary: 'Pathivu currently support query like selection, average, distinct and count. If you would like to see more query features, please feel free to open an [issue](https://github.com/pathivu/pathivu/issues)'
---

---

Queries in Pathivu can be used to aggregate log data and perform operations on it. Multiple queries can be piped up to perform a complex action. 

Pathivu listens for query requests on port `5180` with the use of an HTTP(s) server. This exposes a simple way of sending commands and queries to the Pathivu backend. It currently supports the following queries:

<br>

---

### Index

- [Selection](#selection)
	- [Fuzzy Search](#fuzzysearch)
	- [Structured Search](#structuredsearch)
- [Count](#count)
	- [Base Count](#basecount)
	- [Aggregated Count](#aggregatedcount)
- [Average](#average)
	- [Base Average](#baseaverage)
	- [Aggregated Average](#aggregatedaverage)
- [Distinct](#distinct)
	- [Base Distinct](#basedistinct)
	- [Distinct Count](#countdistinct)
- [Limit](#limit)
- [Pipe](#pipe)
- [Source](#source)

---

<br>


If you would like to see more queries, feel free to create an [issue](https://github.com/pathivu/pathivu/issues) on our source repository, and we will get back to you.

Before getting started with the various query commands supported by Pathivu, let us enumerate what all features we have while calling a query:

* **Ordering**: Ascending and descending order of log output can be decided at the query level.
* **Pagination** : You can set a maximum number of logs to show in output and an start displaying logs from a particular offset.
* **Timestamping**: We can declare a starting and ending timestamp to only output the logs between a given time-frame.


<br>


### Selection
---

Consider the following JSON

```json
{
  "data": [
    {
	"ts": 3,
	"level": "warn",
	"details": {
		"message": "APIKEY not provided",
	}
	"from": "app"
    },
    {
	"ts": 2,
	"level": "fatal"
	"details": {
		"message": "Error connecting to database",
		"error_code": "500"
	},
	"error_code": "500",
	"from": "app"
    }
   ]
}

```

Pathivu supports two types of search queries, namely fuzzy search and structured query search.

<h6 id="fuzzysearch"> Fuzzy Search </h4>

The `message` keyword is used for fuzzy searching in Pathivu. The fuzziness level is configurable. A simple example is given below:
```sh
message = "warn"
```

This query will give you the following output:

```json
{
  "data": [
    {
	"ts": 3,
	"level": "warn",
	"details": {
		"message": "APIKEY not provided",
	}
	"from": "app"
    }
   ]
}
```

[Go to index](#index)

<h6 id="structuredsearch"> Structured Search </h4>

Flattened JSON fields can be used for structured query searches and exact matches. In the following example, we are querying the logs which have error code 500 within an embedded struct.

```sh
details.error_code = "500"
```

This query will give you the following output:

```json
{
  "data": [
    {
	"ts": 3,
	"level": "fatal",
	"details": {
		"message": "APIKEY not provided",
		"error_code": "500"
	}
	"from": "app"
    }
   ]
}
```

[Go to index](#index)

<br>

### Count
---

Count query can be used to get the total number of logs pertaining to a particular key in the log structure. Count can be of two types, namely base count and aggregated count.

Consider the following log JSON:

```json
{
  "data": [
    {
      "ts": 3,
      "entry": {
        "details": {
          "error_code": "500",
          "message": "Error connecting to database"
        },
        "level": "fatal",
        "from": "backend"
      },
      "source": "demo"
    },
    {
      "ts": 2,
      "entry": {
        "details": {
          "error_code": "500",
          "message": "Error connecting to database"
        },
        "level": "fatal",
        "from": "app"
      },
      "source": "demo"
    },
    {
      "ts": 1,
      "entry": {
        "details": {
          "message": "APIKEY not provided"
        },
        "level": "warn",
        "from": "app"
      },
      "source": "demo"
    }
  ]
}
```

<h6 id="basecount"> Base Count </h4>

Base count is a powerful command that can be used for counting the number of logs that exist for a particular field. For example, the example below gives the count of all logs with `from` defined.

```sh
count(from) as src
```

Running this command will give you the following output:

```json
{
  "data": [
    {
      "src": "3"
    }
  ]
}
```

<h6 id="aggregatedcount"> Aggregated Count </h4>

Aggregations can be added in the count query for grouping fields accoring to a particular field. This can be achieved using the `by` keyword. For example, the following query will count all `level` fields and group them by the `from`. 


```sh
count(level) as level_count by from
```
Result will looks like
```json
{
  "data": [
    {
      "level_count": "2",
      "from": "app"
    },
    {
      "level_count": "1",
      "from": "backend"
    }
  ]
}

```

Structured JSON matching can also be used for counting. For example, the following command returns the count of all logs that have the `error_code` field inside `details` sub-structure, grouped by `from`.

```sh
count(details.error_code) as error_code_count by from
```

The output looks like this:

```json
{
  "data": [
    {
      "error_code_count": "1",
      "from": "backend"
    },
    {
      "error_code_count": "1",
      "from": "app"
    }
  ]
}
```

[Go to index](#index)

<br>

### Average
---

The `avg` keyword can be used to find the average of numerical fields in a structured logging scheme. It supports aggregations as well.

Let us consider the following log JSON:

```json
{
  "data": [
    {
      "ts": 3,
      "entry": {
        "country": "Afghanistan",
        "details": {
          "latency": 9.82
        },
        "level": "info"
      },
      "source": "demo"
    },
    {
      "ts": 2,
      "entry": {
        "country": "Pakistan",
        "details": {
          "latency": 6.45
        },
        "level": "info"
      },
      "source": "demo"
    },
    {
      "ts": 1,
      "entry": {
        "country": "India",
        "details": {
          "latency": 3.26
        },
        "level": "info"
      },
      "source": "demo"
    }
  ]
}
```


<h6 id="baseaverage"> Base Average </h4>

The following query will find the average latency of your service.

```sh
avg(details.latency) as average_latency
```

The output looks like this:

```json
{
  "data": [
    {
      "average_latency": "6.51"
    }
  ]
}
```

<h6 id="aggregatedaverage"> Aggregated Average </h4>

Average also supports aggregations. For example, the follwoing query will country-wise average latency.

```sh
avg(details.latency) as average_latency by country
```

The output looks like this:

```json
{
  "data": [
    {
      "average_latency": "3.26",
      "country": "India"
    },
    {
      "average_latency": "6.45",
      "country": "Pakistan"
    },
    {
      "average_latency": "9.82",
      "country": "Afghanistan"
    }
  ]
}
```


[Go to index](#index)

<br>

### Distinct
---

Distinct elements can be found, aggregated and printed using the `distinct` keyword. The distinct command also provides a feature to count the number of distinct logs matched.

<h6 id="basedistinct"> Base Distinct </h4>

Following the example from [count](#count), the following command will give you a list of all distinct levels in the logs. 

```sh
distinct(level)
```

The output will look something like this:

```json
{
  "data": [
    "fatal",
    "warn"
  ]
}
```

<h6 id="countdistinct"> Count Distinct </h4>

In order to find distinct value count, you can use `distinct_count` keyword. The following command will give you a list of all distinct levels in the logs along with their count.

```sh
distinct_count(level)
```

The output will look something like this:

```json
{
  "data": [
    {
      "fatal": 2
    },
    {
      "warn": 1
    }
  ]
}
```

Structured JSON matching can also be used here. For example, the following command will return a list of all distinct error codes along with their count.

```sh
distinct_count(details.error_code)
```

The output looks like this:

```json
{
  "data": [
    {
      "500": 2
    }
  ]
}
```

[Go to index](#index)

<br>

### Limit
---

Limit command can be used to limit the number of responses that we get out of Pathivu query results. For example, in the logs provided [here](#average), the following query can be used to limit the number of responses:

```sh
limit 1
```

The output will look like this:

```json
{
  "data": [
    {
      "ts": 3,
      "entry": {
        "country": "Afghanistan",
        "details": {
          "latency": 9.82
        },
        "level": "info"
      },
      "source": "demo"
    }
  ]
}
```

By default, limits are applied from the latest timestamp in Pathivu.

[Go to index](#index)

<br>

### Pipe
---

Pathivu supports piping as well. Here, you can combine two or more queries, one after the other. This gives Pathivu immense querying capabilities.

Below are a couple of examples as to how piping can be used to make powerful and meaningful queries. Note that all of the queries are performed on the following JSON:

```json
{
  "data": [
    {
      "ts": 3,
      "entry": {
        "country": "Afghanistan",
        "details": {
          "latency": 9.82
        },
        "level": "info",
        "transaction": "succeeded"
      },
      "source": "demo"
    },
    {
      "ts": 2,
      "entry": {
        "country": "Pakistan",
        "details": {
          "latency": 6.45
        },
        "level": "info",
        "transaction": "failed"
      },
      "source": "demo"
    },
    {
      "ts": 1,
      "entry": {
        "country": "India",
        "details": {
          "latency": 3.26
        },
        "level": "info",
        "transaction": "succeeded"
      },
      "source": "demo"
    }
  ]
}
```

<br>

* The following query will give you all of the failed transaction logs grouped by country.

```sh
transaction="failed" | distinct_count(country) as failed_transaction_country_wise
```

So the output will look something like this:

```json
{
  "data": [
    {
      "Pakistan": 1
    }
  ]
}
```

* The following command will give you the count of all info-level logs.

```sh
level="info" | count(level) as level_count
```

The output looks like this:

```json
{
  "data": [
    {
      "level_count": "3"
    }
  ]
}
```

[Go to index](#index)

<br>

### Source
---

User can specify the sources that the user would like to serach on using `source` keyword. Multiple sources are mentioned using `,`.

```sh
source=master,slave
```

This will output all logs with the source as `master` *and* `slave`.

[Go to index](#index)
