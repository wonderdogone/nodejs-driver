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
 
Typically you create only 1 `Client` instance for a given Cassandra cluster and use it across your application.

<pre><code class="javascript">
var driver = require('cassandra-driver');
var client = new driver.Client({contactPoints: ['host1']});
client.connect(function (err) {
  
});
</code>
</pre>

At this point, the driver will be connected to one of the contact points and discovered the rest of the nodes in your cluster.  

Even though calling `#connect()` is not required (the execute method internally calls to connect), it is recommended you call to `#connect()` 
 on application startup, this way you can ensure that you start your app once your are connected to your Cassandra cluster.

## Retrieving data

The `#execute()` method can be used to send a CQL query to a Cassandra node, a simple way to use would be to provide a query and a callback.

<pre><code class="javascript">
client.execute("SELECT name, email, birthdate FROM users WHERE key = 'mick-jagger'", function (err, result) {
  var user = results.rows[0];
  //The row is an Object with column names as property keys. 
  console.log('My name is %s and my email is %s', user.name, user.email);
});
</code>
</pre>

### Using query parameters and prepared statements

Instead of hard coding your parameters in your query, you can use parameter markers in your queries and provide the parameters as an Array.
 
<pre><code class="javascript">
client.execute('SELECT name, email, birthdate FROM users WHERE key = ?', ['mick-jagger'], function (err, result) {

});
</code>
</pre>

This way you can reuse the query and forget about escaping / stringifying the parameters in your query. 

Additionally, if you plan to reuse a query within your application (it is generally the case, your parameter value changes 
but there is only a small number of different queries for a given schema), you can benefit from using prepared statements.
 
Using prepared statements increases performance compared to plain executes, especially for repeated queries as the query only needs to be parsed once by the cluster. 
It has the additional benefit of providing metadata of the parameters to the driver, allowing better type mapping between javascript and Cassandra without the need of additional info (hints) from the user.

<pre><code class="javascript">
//Set the prepare flag in your queryOptions
client.execute('SELECT name, email, birthdate FROM users WHERE key = ?', ['mick-jagger'], {prepare: true}, function (err, result) {

});
</code>
</pre>

If you want to know more about the how 

## Inserting data

You can use the `#execute()` method to execute any CQL query.

<pre><code class="javascript">
var params = ['mick-jagger', 'Sir Mick Jagger', 'mick@rollingstones.com', new Date(1943, 6, 26)];
client.execute('INSERT INTO users (key, name, email, birthdate) VALUES (?, ?, ?)', params, function (err) {
  //Inserted in the cluster
});
</code>

### Setting the consistency level

To specify how consistent the data must be for a given read or write operation, you can set the [consistency level][consistency] per query

<pre><code class="javascript">
client.execute(query, params, {consistency: types.consistencies.quorum}, function (err) {
  //This callback will be called once it has been written in the number of replicas
  //satisfying the consistency level specified.
});
</code>

Or you can provide a default consistency level for all your queries when creating the `Client` instance (defaults to `one`).

<pre><code class="javascript">
var client = new Client({queryOptions: {consistency: types.consistencies.quorum}});
</code>

## Authentication (optional)

Using an authentication provider on an auth-enabled Cassandra cluster:

<pre><code class="javascript">
var authProvider = new driver.auth.PlainTextAuthProvider('my_user', 'p@ssword1!');
//Set the auth provider in the clientOptions when creating the Client instance
var client = new Client({authProvider: authProvider});
</code>
</pre>

 [consistency]: http://www.datastax.com/documentation/cassandra/2.0/cassandra/dml/dml_config_consistency_c.html