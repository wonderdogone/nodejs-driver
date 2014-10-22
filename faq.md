---
layout: default
title: FAQ - DataStax Node.js driver for Apache Cassandra
active: faq
redirect_url: http://www.datastax.com/documentation/developer/nodejs-driver/1.0/nodejs-driver/faq/njdFaq.html
---


# FAQ - Node.js Cassandra

## Which Cassandra versions does this driver support?
The beta version of this driver supports any Cassandra version greater than 2.0 and above.
On future versions, any Cassandra version from 1.2 will be supported.

## Which CQL version does this driver support?
It supports [CQL3](http://cassandra.apache.org/doc/cql3/CQL.html).

## How to generate a uuid and a time-based uuid

[You can use `uuid()` and `timeuuid()` functions in the types module](datatypes#uuid).

## Should I create a `Client` instance per module in my app?
Normally you should use 1 client instance per application, you should share that instance between modules within your application.

## Should I shutdown the pool after executing a query?
No, you should only call `client.shutdown` once in your application lifetime.
