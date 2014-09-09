---
layout: default
title: FAQ - DataStax Node.js driver for Apache Cassandra
active: faq
---


# FAQ

## Which Cassandra versions does this driver support?
The beta version of this driver supports any Cassandra version greater than 2.0 and above.
On future versions, any Cassandra version from 1.2 will be supported.

## Which CQL version does this driver support?
It supports [CQL3](http://cassandra.apache.org/doc/cql3/CQL.html).

## Should I create a `Client` instance per module in my app?
Normally you should use 1 client instance per application domain, you should share that instance between modules within your application.

## Should I shutdown the pool after executing a query?
No, you should only call `client.shutdown` once in your application lifetime.