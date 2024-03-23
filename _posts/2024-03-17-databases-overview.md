---
layout: post
title: "Databases - A whirlwind tour"
date: 2024-03-17
categories: learning architecture
tags: databases
---

This week, I spent some time refreshing myself on various database technologies. This post won't dive particularly deep into any of them, but I found it to be a nice recap.

It's worth clarifying upfront that while we humans like things to fall neatly into categories, the categories we invent inevitably end up with huge lists of exceptions. Database technologies are no exception; the categories I'm discussing here describe broad characteristics of their databases, but many DBs in each will have behaviours that span multiple categories or break the "rules" of the category that they're in. If you decide that a specific category of DB will work well for a project, you still need to look at the options and see which of them has a feature set that fits your use-case.


## Relational databases

Plain ol' RDBMS. These are the table-based databases that have been around for decades and it's almost a certainty that any given developer will have experience with Structured Query Language (SQL), the language used to query them. Their name comes from the fact that records can have relationships with each other by referencing each others' IDs.

Relational DBs are effectively the base case[^1] today and all other types of DBs get grouped under the banner of *NoSQL*. The relational model is really versatile, so those NoSQL databases don't generally have any functionality that couldn't also be achieved with a relational database. The difference is that while relational databases are pretty good at a lot of things, each NoSQL database is optimized for specific use-cases (like full text search or easier horizontal scaling) and will perform much better than a relational database for those tasks.[^2]

**Use-cases** - If you're unsure of the access patterns that you'll need to optimize for, no single NoSQL database meets your specific combination of requirements (and you don't want to run multiple DBs), or you just want something that devs will have an easy time ramping up on, relational DBs are a safe bet.

**Examples**
- MySQL
- PostgreSQL

## Key-value databases

Key-value databases are effectively fancy hash maps; Redis (a popular key-value stores) even has it in its name: **Re**mote **Di**ctionary **S**ervice. You can retrieve values by their keys or run queries on those keys, but key-value DBs generally treat the values as opaque and don't allow you to query those directly.

Some key-value DBs allow you to specify _two_ keys. DynamoDB is one of those and calls the keys the *partition* key and *sort* key (names will vary per DB). They're so named because the partition key determines where the data ends up when it's split over multiple database partitions and the sort key determines the ordering of records within a partition. When you do have two keys, the `(partition key, sort key)` tuple must be unique.

**Use-cases** - Key-value DBs are commonly used as caches and work well for any use-case where you want really efficient access to records by their IDs and don't need to query records' values.

**Examples**
- DynamoDB
- Redis
- Memcached
- RocksDB


## Document databases

Document databases are an extension of key-value databases where the values are all *documents* (which could be XML, JSON, YAML, etc.) and queries can be run against the bodies of those documents. If the DB had to run the query against every document in the scan range for every query, those queries would be super slow. As such, document DBs also allow you to define indexes on specific properties in the documents' bodies. For example, if your DB stored data on users' reading lists (like the one below) and you frequently queried the `favouriteGenre` property, it could be indexed separately to improve query performance:

```json
{
  "userName": "John",
  "favouriteGenre": "Fantasy",
  "readingList": [
    {
      "title": "The Final Empire",
      "author": "Brandon Sanderson"
    }
  ]
}
```

Document databases are *schema-less* because they generally don't enforce a particular shape on the data that you write to them. What that really means, however, is just that there's no _explicit_ schema; the code that reads from the database will expect the data to be in *some* particular shape, which creates an *implicit* schema for the data. Schema-less approaches are also called schema-on-read because the code that's reading the data is the code enforcing expectations on the data's shape.

**Use-cases** - If you want to dump data somewhere without worrying about pre-defining a schema, want very efficient access by ID, and also want to be able to query that data, then document DBs are for you.

**Examples**
- CouchDB
- MongoDB

## Wide-column databases

Unlike relational databases, where data is stored in tables, each with a defined set of columns, wide-column databases define all the *possible* columns that a record could have and *column families*, each of which has some subset of those possible columns. Where relational databases join between their tables, wide-column databases don't support joins at all; instead, data is denormalized and each column family is query-centric and designed to have all the data necessary for some specific query.

Wide-column databases' real advantages are in their write throughput and scalability. They're designed to be extremely horizontally scalable. To achieve those, however, they do trade off consistency[^3].

**Use-cases** - If you have huge write loads, want to be able to scale really easily, and don't need consistency, then wide-column databases could work for you.

**Examples**
- Cassandra
- Hadoop/HBase


## Time-series databases

As their name suggests, time-series DBs are highly optimised for time series data like temperature readings. They store time/value pairs and tend to have fairly large but uniform data sets. I've got one of these running in my home network to store the stats from our house's inverter over time:

![My influx dashboard tracking inverter stats](/assets/images/2024-03-17-databases-overview/influx_dashboard.png)

**Use-cases** - If you want to store and query time-series data, these are the obvious solution.

**Examples**
- Prometheus
- InfluxDB
- Amazon Timestream
- kdb+


## Search engines

Search engines are another highly-specific brand of NoSQL that let you query efficiently by building up inverted indexes on their DB's contents. In doing so, they create index entries that point from every unique string (say) back to their locations. As an example of what that looks like, we could take these strings:

| ID  | Value                                       |
| --- | ------------------------------------------- |
| 0  | The quick brown fox jumps over the lazy dog |
| 1  | The rain in spain stays mainly in the plain |
| 2  | She sell sea shells on the sea shore |
| 3  | The barbarians found out about Barbara's rhubarb bar |

and build up an inverted index that looks like:

| String | Locations |
| --- | --- |
| the | 0, 1, 2, 3 |
| spain | 1 |
| sea | 2 |
| rhubarb | 3 |
| ... | ... |

Each of the "locations" in the inverted index is a string containing the indexed word.

**Use-cases** - These work really well for search engines are dashboarding.

**Examples**
- OpenSearch/Lucene
- Several other DBs (like MySQL and PostgresQL) also support full text search in text columns


## Graph databases

Like relational DBs, graph databases model the relationships between data, but those relationships are even even higher priority. Graph databases model their data as a set of nodes and edges. Relative to other databases, they support more efficient joins of arbitrary depth and use sharding strategies that support those efficient joins. Because relationships are first-class citizens, graph databases also lend themselves to exploring datasets through the connections between data. In my experience, graph databases tend to be used more for OLAP workflows like fraud detection and seeking out unexepected relationships between data than OLTP.

**Use-cases** - Anything that needs really complex and deep joins to be efficient and use-cases that focus on exploring and understanding the relationships between data.

**Examples**
- Amazon Neptune
- Neo4j
- Aerospike
- ArangoDB

## Footnotes

[^1]: (Data)base case (hah).
[^2]: Because each NoSQL database is optimized for their specific use-case, the question of "Should I use a SQL or NoSQL database" is unanswerable unless you know what NoSQL database is being considered.
[^3]: The CAP theorem made them do it.