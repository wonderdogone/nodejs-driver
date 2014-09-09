---
layout: default
title: Getting started with DataStax Node.js driver for Apache Cassandra
active: start
---


# Getting started

Getting started with the DataStax Node.js driver for Apache Cassandra.

## Connecting to a cluster

To connect to a Cassandra cluster, you need to provide at least 1 node of the cluster, 
 if there are more nodes than the ones provided, the driver will add it for you once it connects to the first node.

<pre><code class="javascript">
var driver = require('cassandra-driver');
var client = new driver.Client({contactPoints: ['host1']});
client.connect(function (err) {
  
});
</code>
</pre>

Even though calling connect is not required (the execute method internally calls connect), it is recommended you call to `connect` 
 on application startup, this way you can ensure that you start your app only if your Cassandra cluster is up.

## Executing your first query

_Work in progress: check [project README](https://github.com/datastax/nodejs-driver) for more info._

## Using query parameters and prepared statements

_Work in progress: check [project README](https://github.com/datastax/nodejs-driver) for more info._

## Authentication (optional)

Using an authentication provider on an auth-enabled Cassandra cluster:

<pre><code class="javascript">
var authProvider = new driver.auth.PlainTextAuthProvider('my_user', 'p@ssword1!');
//Setting the auth provider in the clientOptions
var client = new Client({authProvider: authProvider});
</code>
</pre>
