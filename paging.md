---
layout: default
title: Fetching large result sets - Node.js Cassandra
active: start
redirect_url: http://www.datastax.com/documentation/developer/nodejs-driver/1.0/nodejs-driver/reference/paging.html
---


# Fetching large result sets from Cassandra

The single-threaded nature of Node.js should be taken into consideration when dealing with retrieving large amount of rows.
 If an asynchronous method uses large buffers, it could cause large memory consumption and a poor response time.
 
On the Node.js driver for Cassandra we addressed these concerns by exposing the `#eachRow()` and `#stream()` methods.
 When using these methods, the driver will parse the rows and yield them to the user as they come through the network.
 All methods use a default `fetchSize` of 5000 rows, retrieving only first page of results up to a maximum of 5000 rows.

## Retrieving the following pages

### Automatic paging

If you want to retrieve the next pages, you can do it by setting `autoPage: true` in the `queryOptions` to automatically request the next page,
 once it finished parsing the previous page.

Example:
<pre><code class="javascript">
//Imagine a column family with millions of rows
var query = 'SELECT * FROM largetable';
client.eachRow(query, [], {autoPage: true}, function (n, row) {
  //This function will be called per each of the rows in all the table  
}, callback);
</code></pre>

### Manual paging

If you want to retrieve the next page of results only when you ask for it (ie: web pager), you can use the `pageState` 
 made available by the end callback in `#eachRow()`. 

<pre><code class="javascript">
client.eachRow(query, [], {prepare: 1, fetchSize: 1000}, function (n, row) {
    //row callback
}, function (err, result) {
    //end callback
    //store the paging state
    pageState = result.meta.pageState;
});
</code></pre>

And in the following requests, use the page state.

<pre><code class="javascript">
//Use the pageState in the queryOptions to continue where you left it.
client.eachRow(query, [], {pageState: pageState, prepare: 1, fetchSize: 1000}, function (n, row) {
    //row callback
}, function (err, result) {
    //end callback
    //store the next paging state
    pageState = result.meta.pageState;
});
</code></pre>