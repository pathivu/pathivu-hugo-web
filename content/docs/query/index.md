---
title: 'Query'
date: 2019-12-29T22:42:33+05:30
weight: 2
summary: 'Pathivu currently support query like selection, average, distinct and count. If you would like to see more query features, please feel free to open an [issue](https://github.com/pathivu/pathivu/issues)'
---

---

Queries in Pathivu can be used to aggregate log data and perform operations on it. Multiple queries can be piped up to perform a complex action. 

Pathivu listens for query requests on port `5180` with the use of an HTTP(s) server. This exposes a simple way of sending commands and queries to the Pathivu backend. It currently supports the following queries:


- [Selection](#selection)
	- [Fuzzy Search](#fuzzysearch)
	- [Structured Search](#structuredsearch)
- [Count](#count)
	- [Base Count](#basecount)
	- [Aggregated Count](#aggregatedcount)
- [Average](#average)
- [Distinct](#distinct)
- [Pipe](#pipe)
- [Source](#source)

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
	"source": "app"
    },
    {
	"ts": 2,
	"level": "fatal"
	"details": {
		"message": "Error connecting to database",
		"error_code": "500"
	},
	"error_code": "500",
	"source": "app"
    }
   ]
}

```

Pathivu supports two types of search queries, namely fuzzy search and structured query search.

<h6 id="fuzzysearch"> Fuzzy Search </h4>

The `message` keyword is used for fuzzy searching in Pathivu. A simple example is given below:
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
	"source": "app"
    }
   ]
}
```

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
	"source": "app"
    }
   ]
}
```

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
        "source": "backend"
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
        "source": "app"
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
        "source": "app"
      },
      "source": "demo"
    }
  ]
}
```

<h6 id="basecount"> Base Count </h4>

Base count is a powerful command that can be used for counting the number of logs that exist for a particular field. For example, the example below gives the count of all logs with `source` defined.

```sh
count(source) as src
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

Aggregations can be added in the count query for grouping fields accoring to a particular field. This can be achieved using the `by` keyword. For example, the following query will count all `level` fields and group them by the `source`. 


```sh
count(level) as level_count by source
```
Result will looks like
```json
{
  "data": [
    {
      "level_count": "2",
      "source": "app"
    },
    {
      "level_count": "1",
      "source": "backend"
    }
  ]
}

```

Structured JSON matching can also be used for counting. For example, the following command returns the count of all logs that have the `error_code` field inside `details` sub-structure, grouped by source.

```sh
count(details.error_code) as error_code_count by source
```

The output looks like this:

```json
{
  "data": [
    {
      "error_code_count": "1",
      "source": "backend"
    },
    {
      "error_code_count": "1",
      "source": "app"
    }
  ]
}
```

<br>

### Average
---

you can find the average of the numerical fields. 

This below query will find the average latency of your service.

```sh
avg(latency) as average_latency
```

You can apply grouping in the average quries as well.

The below query will find the averge latency by country wise.
```sh
avg(latency) as average_latency by country
```

<br>

### Distinct
---

You can find the distict elements by distinct key word

The below query will give you all the distict country value.
```sh
distinct(country) as dictinct_country
```

In order to find distinct value count, you can use `distinct_count` keyword.

```sh
distinct_count(country) as country_count
```

Result will looks like
```json
{
    "data":[
        {
            "country_count": 100,
            "country": "Kumari Kandam"
        },
        {
            "country_count": 100,
            "country": "Atlantis"
        }
    ]
}
```

<br>

### Pipe
---

User can combine two quries using `|` operator

The below query find the count of all the failed transaction country wise.
```sh
transaction="failed" | distinct_count(country) as failed_transaction_country_wise
```

<br>

### Source
---

User can specify the sources that the user would like to serach on using `source` keyword. Multiple sources are mentioned using `,`.

```sh
source=transaction_service,refund_service
```
