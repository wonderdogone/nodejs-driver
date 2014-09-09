---
layout: default
title: DataStax Node.js driver for Apache Cassandra
---

# DataStax Node.js driver for Apache Cassandra

Node.js driver for Apache Cassandra. This driver works exclusively with the Cassandra Query Language version 3 (CQL3) and Cassandra's native protocol.

## Basic usage

<pre><code class="javascript">
var driver = require('cassandra-driver');
var client = new driver.Client({contactPoints: ['host1', 'host2'], keyspace: 'ks1'});
var query = 'SELECT email, last_name FROM user_profiles WHERE key=?';
client.execute(query, ['guy'], function(err, result) {
  console.log('got user profile with email ' + result.rows[0].email);
});
</code>
</pre>