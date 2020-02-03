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
- [Count](#count)
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

Pathivu now supports both fuzzy search and structured json style search. 

For fuzzy serach, in pathivu message keyword is used.

In the below example, we are finding the log line which containing warn.
```sh
message = "warn"
```

For structured query, flattend json field is used.

In the below example, we are finding all the user from the Kumari Kandam.

```sh
user.location = "Kumari Kandam"
```

<br>

### Count
---

The below query will gives you the count of country field in the log lines.

```sh
count(country) as country_count
```

You can aggregate by adding `by`keyword.

The below query will count all the log line with the transaction field and aggregates by country.
```sh
count(transaction) as transaction_count by region
```
Result will looks like
```json
{
    "data":[
        {
            "transaction_count": 100,
            "country": "Kumari Kandam"
        },
        {
            "transaction_count": 100,
            "country": "Atlantis"
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
