---
layout: post
title: Await and Async Explained with Diagrams and Examples
date: 2017-10-01 05:22:09.000000000
type: post
published: true
status: publish
excerpt: 
    Explains JavaScript Async/Await syntax and semantics with diagrams and examples ... 
categories:
- JavaScript
- Node
- Async
- Await
- Promise
- blog
tags:
- JavaScript
- Node
- Async
- Await
- Promise
author:
  login: nikolay.grozev@gmail.com
  email: nikolay.grozev@gmail.com
  display_name: nikolay.grozev@gmail.com
  first_name: 'Nikolay'
  last_name: 'Grozev'
---

# Table of Contents

- [Introduction](#introduction)
- [Promises](#promises)
- [The Problem - Composing Promises](#composite-promises)
- [Async Functions](#async-functions)
- [Await](#await)
- [Error Handling](#error-handling)
- [Discussion](#discussion)

<div id='introduction'/>
# Introduction

The [async/await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)
syntax in JavaScript ES7 makes it easier to coordinate asynchronous promises.
If you need to asynchronously fetch data from multiple databases or APIs in a 
certain order, you can end up with a spaghetti of promises and callbacks.
The `async`/`await` construct allows us to express such logic more succinctly
with more readable and maintainable code.

This tutorial explains JavaScript `async`/`await` syntax and semantics with diagrams and simple examples. 

Before we dive in, let's start with a brief overview of promises. 
Feel free to skip this section if you already know about JS promises.

<div id='promises'/>
# Promises

In JavaScript, a promise represents an abstraction of non-blocking asynchronous execution.
JS promises are similar to [Java's `Future`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Future.html) or
[C#'s `Task`](https://msdn.microsoft.com/en-us/library/system.threading.tasks.task(v=vs.110).aspx),
if you have come across them.

Promises are typically used for network and I/O operations - e.g. reading from a file, or
making HTTP requests. Instead of blocking the current "thread" of execution, we can spawn
an asynchronous promise, and use the `then` method to attach a 
callback which will be triggered when the promise completes. The callback itself can return 
a promise, and thus we can effectively chain promises. 

For the sake of simplicity, in all examples we'll assume that the 
[request-promise](https://github.com/request/request-promise) library
has been installed and loaded as:

```javascript
var rp = require('request-promise');
```

Now we can make a simple HTTP GET request that returns a promise as 

```javascript
const promise = rp('http://example.com/')
```

Now, let's look at an example:

```javascript
console.log('Starting Execution');

const promise = rp('http://example.com/');
promise.then(result => console.log(result));

console.log("Can't know if promise has finished yet...");
```

We spawned a new `Promise` on **line 3**, and then attach
a callback function to it on **line 4**. The `promise` is asynchronous, so when
we reach **line 6**, we can't know if the promise has completed or not. 
If we run the code multiple times, we may get different results each time.
More generally, the code after any promise is spawn runs concurrently with the promise. 

**There is no reasonable way to block the current sequence of operations until `promise` has finished**.
This is different from [Java's `Future.get`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Future.html#get--), 
which allows us to block the current thread until a `Future` has completed.
In JavaScript, **we can't easily wait for a promise**. The only way to schedule code after `promise`,
is to attach a callback via `then`.

The following diagram depicts the computational process of the example:

<figure>
  <img src="/images/blog/async-await/SimplePromiseExample.png" alt="Simple Promises Example." >
  <figcaption>
    Computational process of a promise. The calling "thread" can't wait for the promise.
    The only way to schedule code after a promise is to specify a callback via the "then" method.
  </figcaption>
</figure>

The callback attached via `then` executes only if the promise is successful.
If it fails (e.g. due to a network error), the callback will not execute.
To handle failed promises, you can attach another callback via `catch`:

```javascript
rp('http://example.com/').
    then(() => console.log('Success')).
    catch(e => console.log(`Failed: ${e}`))
```

Finally, for test purposes we can easily create "dummy" promises that succeed or fail
with the `Promise.resolve` and `Promise.reject` methods:

```javascript
const success = Promise.resolve('Resolved');
// Will print "Successful result: Resolved"
success.
    then(result => console.log(`Successful result: ${result}`)).
    catch(e => console.log(`Failed with: ${e}`))


const fail = Promise.reject('Err');
// Will print "Failed with: Err"
fail.
    then(result => console.log(`Successful result: ${result}`)).
    catch(e => console.log(`Failed with: ${e}`))
```

For a more detailed tutorial on promises, check 
[this article](/2017/01/22/node-js-cheatsheet-part-1/#promises).


<div id='composite-promises'/>
# The Problem - Composing Promises

Using a single promise is straightforward. However, when we need to program complicated
asynchronous logic, we may end up combining a few promises. Writing all the `then` clauses
and anonymous callbacks can easily get out of hand.

For example, let's assume we need to write a program which:

1. Makes an HTTP call, waits for it to complete, and prints the result;
2. Then makes other two parallel HTTP calls;
3. When both of them complete, prints their result. 

The following snippet demonstrates how this can be done: 

```javascript
// Make the first call
const call1Promise = rp('http://example.com/');

call1Promise.then(result1 => {
    // Executes after the first request has finished
    console.log(result1);

    const call2Promise = rp('http://example.com/');
    const call3Promise = rp('http://example.com/');

    return Promise.all([call2Promise, call3Promise]);
}).then(arr => {
    // Executes after both promises have finished
    console.log(arr[0]);
    console.log(arr[1]);
})
```

We start by making the first HTTP call and scheduling a callback to run when it 
finishes (**lines 1-3**). In the callback, we spawn two other promises 
for the subsequent HTTP request (**lines 8-9**). These two promises run
concurrently and we need to schedule a callback, when **both of them complete**.
Hence, we need to combine them into a single promise via `Promise.all` (**line 11**),
which resolves when both of them complete. The result of the callback is a promise,
so we need to chain yet another `then` callback that prints the results (**lines 12-16**). 

The following diagram depicts the computational flow:

<figure>
  <img src="/images/blog/async-await/CombinedPromises.png" alt="Combined Promises Example." >
  <figcaption>
    Computational process of a combination of promises. We use "Promise.all" to combine two
    concurrent promises into a single promise.
  </figcaption>
</figure>

For such a simple example, we ended up with 2 `then` callbacks and had to use 
`Promise.all` to synchronise concurrent promises. What if we had to run a few more
asynchronous operations or add error handling? 
This approach can easily end up in a spaghetti of `then`-s, 
`Promise.all` calls, and callbacks.  


<div id='async-functions'/>
# Async Functions

An async function is a shortcut for defining a function which returns a promise.

For example, the following definitions are equivalent:

```javascript
function f() {
    return Promise.resolve('TEST');
}

// asyncF is equivalent to f!
async function asyncF() {
    return 'TEST';
}
```

Similarly, async functions which throw exceptions are equivalent to functions 
which return rejecting promises:

```javascript
function f() {
    return Promise.reject('Error');
}

// asyncF is equivalent to f!
async function asyncF() {
    throw 'Error';
}
```

<div id='await'/>
# Await

When we spawn a promise we can't synchronously wait for it to finish.
We can only pass a callback via `then`. Waiting for a promise is disallowed to encourage the
development of non-blocking code. Otherwise, developers would be tempted to perform 
blocking operations because it's easier than working with promises and callbacks. 

However, in order to synchronise promises we need to allow them to wait for each other.
In other words, if an operation is asynchronous (i.e. encapsulated in a promise) it should
be able to wait for another asynchronous operation to finish. But how would the
JavaScript interpreter know if an operation is running within a promise or not? 

The answer is in the `async` keyword. Every `async` function returns a promise.
Thus, the JavaScript interpreter knows that all operations in `async` functions 
will be encapsulated in promises and run asynchronously. Therefore, it can allow
them to wait for other promises to finish.

Enter the `await` keyword. **It can only be used in `async` functions**, and allows
us to synchronously wait on a promise. If we use promises outside of `async` functions
we will still need to use `then` callbacks:

```javascript
async function f(){
    // response will evaluate as the resolved value of the promise
    const response = await rp('http://example.com/');
    console.log(response);
}

// We can't use await outside of async function.
// We need to use then callbacks ....
f().then(() => console.log('Finished'));
```

Now, let's look at how we can solve the problem from the [previous section](#composite-promises):

```javascript
// Encapsulate the solution in an async function
async function solution() {
    // Wait for the first HTTP call and print the result
    console.log(await rp('http://example.com/'));

    // Spawn the HTTP calls without waiting for them - run them concurrently
    const call2Promise = rp('http://example.com/');  // Does not wait!
    const call3Promise = rp('http://example.com/');  // Does not wait!

    // After they are both spawn - wait for both of them
    const response2 = await call2Promise;
    const response3 = await call3Promise;

    console.log(response2);
    console.log(response3);
}

// Call the async function
solution().then(() => console.log('Finished'));
```

In the above snippet, we encapsulate the solution in an `async` function.
This allows us to directly `await` for the promises, thus avoiding the need
for `then` callbacks. Finally, we invoke the `async` function which simply
spawns a promise which encapsulates the logic of invoking the other promises.

Indeed in the first example (without `async`/`await`), the promises will be fired in parallel.
In this case we do the same (lines 7-8). Notice that we don't use `await` until **lines 11-12**, 
when we block the execution until both promises have resolved. Afterwards, we know that both promises 
have resolved (similarly to using `Promise.all(...).then(...)`) in the previous example.

The underlying computational process is equivalent to the one depicted in 
[previous section](#composite-promises). The code, however, is much more readable
and straightforward.

Under the hood, `await`/`await` actually translate to promises and `then`
callbacks. In other words, it's syntactic sugar for working with promises.
Every time we use `await`, the interpreter spawns a promise, and puts the
rest of the operations from the `async` function in a `then` callback.

Let's consider the following example:

```javascript
async function f() {
    console.log('Starting F');
    const result = await rp('http://example.com/');
    console.log(result);
}
```

The underlying computational process of the `f` function is depicted below.
Since `f` is `async`, it will also run in parallel with its caller:

<figure>
  <img src="/images/blog/async-await/AsyncAwaitExample.png" alt="AsyncAwaitExample." >
  <figcaption>
    Computational process of "await".
  </figcaption>
</figure>

The function `f` starts and spawns a promise. At that moment, the rest of
the function is encapsulated in a callback, and scheduled to execute after the
promise is complete.

<div id='error-handling'/>
# Error Handling

In most previous examples, we assumed that the promises resolve successfully.
Hence, `await`-ing on a promise returned a value. If a promise we
`await` for fails, this will result in an exception within the async function.
We can use standard `try`/`catch` to handle it:

```javascript
async function f() {
    try {
        const promiseResult = await Promise.reject('Error');
    } catch (e){
        console.log(e);
    }
}
``` 

If the `async` function does not handle an exception, whether it is caused by
a rejected promise or another bug, it will return a rejected promise:

```javascript
async function f() {
    // Throws an exception
    const promiseResult = await Promise.reject('Error');
}

// Will print "Error"
f().
    then(() => console.log('Success')).
    catch(err => console.log(err))

async function g() {
    throw "Error";
}

// Will print "Error"
g().
    then(() => console.log('Success')).
    catch(err => console.log(err))
```

This gives us handy way to work with rejected promises via well the known
exception handling mechanism.

<div id='discussion'/>
# Discussion

Async/await is a language structure that complements promises. It allows us to work
with promises with less boilerplate. However, `async`/`await` **do not replace
the need for plain promises**. For example, if we call an `async` function from
a normal function or the global scope, we won't be able to use `await` and will resort to 
vanilla promises:

```javascript
async function fAsync() {
    // actual return value is Promise.resolve(5)
    return 5;
}

// can't call "await fAsync()". Need to use then/catch
fAsync().then(r => console.log(`result is ${r}`));
```

I usually try to encapsulate most of my asynchronous logic in one or few `async` 
functions, which I call from the non-async code. This minimises the amount of
`then`/`catch` callbacks that I need to write.

The `async`/`await` constructs are syntactic sugar for working with promises more
coincisely. Every `async`/`await` construct can be rewritten with plain promises.
Ultimately, it's a matter of style and brevity.

Academics like pointing out that there's a difference between **concurrency** and **parallelism**.
Check out [Rob Pike's talk](https://vimeo.com/49718712) on the topic or my 
[previous post](/2015/07/14/overview-of-modern-concurrency-and-parallelism-concepts/).
Concurrency is about **composing independent processes** (in the general meaning of the term process) 
to work together, while parallelism is about **actually executing multiple processes simultaneously**. 
Concurrency is about the application design and structure, while parallelism is about the actual execution.

Let’s take a multi-threaded application as an example. The separation of the application into 
threads defines its concurrent model. The mapping of these threads on the available cores defines 
its level or parallelism. A concurrent system may run efficiently on a single processor, 
in which case it is not parallel.

<figure>
  <img src="/images/blog/Overview of Modern Concurrency and Parallelism Concepts/concurrent_vs_parallel.png" alt="Concurrent vs. Parallel" >
  <figcaption>Concurrent vs. Parallel.</figcaption>
</figure>

In that sence, promises allow us to break up a program into **concurrent** modules
which may or may not run in parallel. Whether the actual JavaScript execution is 
**parallel** depends on the implementation. For example, Node Js is single threaded and
if a promise is CPU bound you won't see much parallelism. However, if you compile 
your code to java bytecode via something like
[Nashorn](http://www.oracle.com/technetwork/articles/java/jf14-nashorn-2126515.html),
in theory you may be able to map CPU bound promises on different cores and achieve
parallelism. Hence in my opinion, promises (either vanilla or expressed via `async`/`await`) constitute
the concurrency model of a JavaScript app.

> **Update**: updated based on constructive comments from the forum.

