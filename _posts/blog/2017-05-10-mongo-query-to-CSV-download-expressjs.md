---
layout: post
title: Mongo Query to CSV Download (Express JS) 
date: 2017-05-10 05:22:09.000000000
type: post
published: true
status: publish
excerpt: 
    Streaming a large Mongo query to a CSV download via Express JS ...
categories:
- JavaScript
- Node
- NPM
- Express
- Mongo
- CSV
- blog
tags:
- JavaScript
- Node
- NPM
- Mongo
- CSV
author:
  login: nikolay.grozev@gmail.com
  email: nikolay.grozev@gmail.com
  display_name: nikolay.grozev@gmail.com
  first_name: 'Nikolay'
  last_name: 'Grozev'
---


<div id='introduction'/>
# The Problem

I recently worked on an app which uses [MongoDB](https://www.mongodb.com/) and
[Express JS](https://expressjs.com/). 
The app had to allow for the
export of a large Mongo query result (50K+ of documents). In this post, we'll
see how to do it succinctly and "in-flight" with two popular CSV libraries.

<div id='query-cursor'/>
# Query Cursor

To keep things simple, I used the 
[Standard MongoDB driver](https://mongodb.github.io/node-mongodb-native/). 
My ultimate goal was to stream documents from a query to a CSV formatter,
which can then stream the output to the HTTP response.
As a first step we need to acquire a dabatase `cursor`. Assuming the Mongo DB driver
instance is called `mongodb` our code should be similar to this:

```javascript
// Your collection name
const collectionName = ...

// Your Mongo query object - e.g. {name: "john"}
const query = ... 

// Your Mongo query selection object - e.g. {name: 1, _id: 0}
// Select as few fields as possible!
const selection = ...

// Every query (e.g. find/aggregate) returns a cursor
const cursor = mongodb.collection(collectionName).find(query, selection)
```

Cursors can be `pipe`-d into other streams and are handy for "in-flight" processing.

<div id='transformation-function'/>
# Transformation Function

We'll also need a function which transforms the JSON documents, returned by the cursor,
into objects with a suitable schema/object model. The function should look like this:

```javascript
transform(doc) {
    // Return an object with all fields you need in the CSV
    // For example ...
    return {
        Address: doc.address,
        State: doc.state.abbreviation
    };
}
```

<div id='fast-csv'/>
# Using Fast-CSV 

Let's see how to transform the cursor's data with the [fast-csv](https://github.com/C2FO/fast-csv) library.

First we obtain the cursor and the transformation function. Then we need to set the appropriate 
download headers, which also include the name of the downloaded file, and send/flush them
to the response. Next, we create a CSV transformer sream, and use it to "pipe" the cursor's documents 
the response. The following snippet demonstrates the idea:

```javascript
const fastCsv = require('fast-csv');

downloadCSV(request, response) {
    // Obtain your cursor ...
    const cursor = ...
    // The transformer function
    const transformer = ...;

    // Name of the downloaded file - e.g. "Download.csv"
    const filename = ...;

    // Set approrpiate download headers
    response.setHeader('Content-disposition', `attachment; filename=${filename}`);
    response.writeHead(200, { 'Content-Type': 'text/csv' });

    // Flush the headers before we start pushing the CSV content
    response.flushHeaders();

    // Create a Fast CSV stream which transforms documents to objects
    var csvStream = fastCsv
        .createWriteStream({headers: true})
        .transform(transformer)

    // Pipe/stream the query result to the response via the CSV transformer stream 
    cursor.stream().pipe(csvStream).pipe(response);
}
```

<div id='csv'/>
# Using Node-CSV 

The [fast-csv](https://github.com/C2FO/fast-csv) library worked pretty well, but I was
also willing to try the [node-csv](https://github.com/wdavidw/node-csv) library. 
I was hoping to get better performance without much extra efforts. Its interface is similar, so
without much change the code looked like this:

```javascript
var csv = require('csv');

download(request, response) {
    // Obtain your cursor ...
    const cursor = ...
    // The transformer function
    const transformer = ...;

    // Name of the downloaded file - e.g. "Download.csv"
    const filename = ...;

    // Set approrpiate download headers
    response.setHeader('Content-disposition', `attachment; filename=${filename}`);
    response.writeHead(200, { 'Content-Type': 'text/csv' });

    // Flush the headers before we start pushing the CSV content
    response.flushHeaders();

    // Pipe/stream the query result to the response via the CSV transformer stream 
    cursor.stream()
        .pipe(csv.transform(transformer))
        .pipe(csv.stringify({header: true}))
        .pipe(response)
}
```

However, the performance of the two libraries turned out to be very similar.
