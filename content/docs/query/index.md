---
title: 'Query'
date: 2019-12-29T22:42:33+05:30
weight: 2
summary: 'Pathivu currently support query like selection, average, distinct and count. If you would like to see more query features, please feel free to open an [issue](https://github.com/pathivu/pathivu/issues)'
---

Pathivu currently support query like selection, average, distinct and count. If you would like to see more query features, please feel free to open an [issue](https://github.com/pathivu/pathivu/issues)


### Selection

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

### Count

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

### Average

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

### Distinct

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

### Pipe

User can combine two quries using `|` operator

The below query find the count of all the failed transaction country wise.
```sh
transaction="failed" | distinct_count(country) as failed_transaction_country_wise
```

### Source

User can specify the sources that the user would like to serach on using `source` keyword. Multiple sources are mentioned using `,`.

```sh
source=transaction_service,refund_service
```