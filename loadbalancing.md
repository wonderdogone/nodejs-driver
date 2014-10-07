---
layout: default
title: Load balancing policies - Node.js driver for Cassandra
active: start
---

# Load balancing - Node.js driver for Cassandra

Load balancing policies are used to determine which Cassandra node will the driver send the request to execute a query. The policies are responsible of yielding a group of nodes in an specific order for the driver to use (in case the first node fails, it uses the next node).

There are 3 load balancing policies implemented in the driver:

- **RoundRobinPolicy**: It yield nodes in a round-robin fashion.
- **DCAwareRoundRobinPolicy**: A datacenter aware round-robin load balancing policy. This policy provides round-robin queries over the node of the local data center. It also includes in the query plans returned a configurable number of hosts in the remote data centers, but those are always tried after the local nodes.
- **TokenAwarePolicy**: It yields replica nodes for a given partition key and keyspace. The token aware uses a child policy to retrieve the next nodes in case the replicas for a partition key are not available.

The default load balancing policy is TokenAwarePolicy with DCAwareRoundRobinPolicy as child policy. It may seem complex but it actually isn't: The policy will yield local replicas for a given key and, if not available, it will yield nodes of the local datacenter in a round-robin manner.

## TokenAwarePolicy: specifying the partition key

The Token-aware load balancing policy saves network hops by selecting the replicas before making the query request. This way the replica node will be the coordinator of the request. 

In order to select the replica node you have to specify the routing key in the query options.

There are 2 ways to set the routing key for a query.

You can either specify the routingKey in the queryOptions.

<pre><code class="javascript">
var options = { routingKey: value };
client.execute(query, params, options, callback);
</code></pre>

Being `value` a [Buffer][buffer] containing the partition key(s) in binary format.

Or you can specify which parameters in the query are part of the partition key.

<pre><code class="javascript">
//the parameter in the position 0 is the partition key
var options = { routingIndexes: [0] };
client.execute(query, params, options, callback);
</code></pre>

## Implementing a custom load balancing policy

The built-in policies in the Node.js driver for Cassandra cover most common use cases. In the rare case that you need to implement you own policy you can do it by inheriting from one of the existent policies or the abstract `LoadBalancingPolicy` class.

You will have to take in account that the same policy will be used for all queries in order to yield the hosts in correct order.

The load balancing policies are implemented using the [Iterator Protocol][iterator], a convention for lazy iteration
 allowing to produce only the next value in the series without producing a full Array of values. Under Ecmascript 6 (harmony), it enables you to use the new [generators][generator].

**Example: A policy that selects every node except an specific one**.

Note that this policy is a sample and it is not intended for production use. [Use multiple data-center Cassandra clusters instead][dc]. 

<pre><code class="javascript">
function BlackListPolicy(blackListedHost, childPolicy) {
  this.blackListedHost = blackListedHost;
  this.childPolicy = childPolicy;
}

util.inherits(BlackListPolicy, LoadBalancingPolicy);

BlackListPolicy.prototype.init = function (client, hosts, callback) {
  this.client = client;
  this.hosts = hosts;
  //initialize the child policy
  this.childPolicy.init(client, hosts, callback);
};

BlackListPolicy.prototype.getDistance = function (host) {
  return this.childPolicy.getDistance(host);
};

BlackListPolicy.prototype.newQueryPlan = function (keyspace, queryOptions, callback) {
  var self = this;
  this.childPolicy.newQueryPlan(keyspace, queryOptions, function (iterator) {
    callback(self.filter(iterator));
  });
}

BlackListPolicy.prototype.filter = function (childIterator) {
  var self = this;
  return {
    next: {
      var item = childIterator.next();
      if (item.address === self.blackListedHost) {
        //use the next one
        return this.next();
      }
      return item;
    }
  }
}
</code></pre>

[buffer]: http://nodejs.org/api/buffer.html
[dc]: http://www.datastax.com/documentation/cassandra/2.1/cassandra/initialize/initializeMultipleDS.html
[iterator]: https://developer.mozilla.org/en/docs/Web/JavaScript/Guide/The_Iterator_protocol
[generator]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function*