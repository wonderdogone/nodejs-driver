---
layout: default
title: Cassandra CQL types to javascript types - DataStax Node.js driver for Apache Cassandra
active: datatypes
---

# CQL types to javascript types

There are few data types defined in the ECMAScript standard for javascript. This usually represents a problem when you are
trying to deal with data types that come from other systems.

### Note: Number type

According to the ECMAScript standard, there is only one number type: the double-precision 64-bit binary format IEEE 754 value (number between -(253 -1) and 253 -1). 
**There is no specific type for integers** (or 32-bit float). 

## Retrieving Data

When retrieving the value of a column from a Row object, the value will be typed according to the following table.

CQL3 data type | javascript type
--- | ---
ascii | String
bigint | [Long][long]
blob | Buffer
boolean | Boolean
counter | [Long][long]
decimal | Buffer
double | Number
float | Number
inet | Buffer
int | Number
list | Array
map | Object
set | Array
text | String
timestamp | Date
timeuuid | String
tuple | _not supported yet_
uuid | String
varchar | String
varint | Buffer

## Encoding data

When encoding data, on a normal execute with parameters, the driver tries to guess the
 target type based on the input type. Values of type Number will be encoded as double (as Number is double / IEEE 754 value).

Consider the following example:

<pre><code class="javascript">
var key = 1000;
client.execute('SELECT * FROM table1 where key = ?', [key], callback);
</code>
</pre>

If the key column is of type `int`, the execution will fail. There are 2 possible ways to avoid this type of problem:

### Prepare the statement (recommended)

Using prepared statements provides multiple benefits. 
A prepared statement is parsed and prepared on the Cassandra nodes and is ready for future execution.
Another benefit of prepared statements is that the driver will get the parameter types information allowing accurate mapping between a javascript type and a Cassandra type.

Using the previous example, setting the prepare flag in the queryOptions will fix it:

<pre><code class="javascript">
//prepare the query before execution
client.execute('SELECT * FROM table1 where key = ?', [key], {prepare: true}, callback);
</code></pre>

When using prepare statements, the driver will prepare the statement once on each host to execute multiple times.

### Hinting the target types

Providing `hints` in the query options is another way around it.

<pre><code class="javascript">
//Hint: the first parameter is an integer
client.execute('SELECT * FROM table1 where key = ?', [key], {hints: ['int']}, callback);
</code>
</pre>

## bigint, uuid and timeuuid CQL types <a name="uuid"></a><a name="bigint"></a>

Even though ECMAScript does not provide a way to represent these data types, the Node.js Cassandra driver provides ways to use these types.

<pre><code class="javascript">
var cassandra = require('cassandra-driver');
//generate a new v4 uuid
var id1 = cassandra.types.uuid();
//generate a new v1 uuid
var id2 = cassandra.types.timeuuid();
//big int value
var bigIntValue = new cassandra.types.Long.fromString("576460752303423511");
</code>
</pre>

Long is supported using [Long][long] module that it is a wrapper of [Google Closure's Long][closure-long].

Uuid is supported using [node-uuid][uuid] module. 

[uuid]: https://github.com/broofa/node-uuid
[long]: https://github.com/dcodeIO/Long.js
[closure-long]: http://docs.closure-library.googlecode.com/git/class_goog_math_Long.html
