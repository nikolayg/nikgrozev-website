---
layout: post
title: DIY Type Safe AOP in TypeScript
date: 2018-09-18 05:22:09.000000000
type: post
published: false
status: publish
excerpt: 
    In this article, we will demonstrate how to implement a simple type safe utility function 
    for audit logging in Typescript.  
categories:
- JavaScript
- TypeScript
tags:
- JavaScript
- TypeScript
- AOP
---

<div id='introduction'/>
# Introduction

Recently, I have been working on a few TypeScript projects where I had to implement audit logging
for many opeartions. For example, I had to log every request to an external API and the received response or error message.
I also had to audit log every attempt to access externally hosted resources (e.g. SMTP or WebDAV servers).

I had to wrap every function invocation, which needed to be audited, with the same logging logic.
For simplicity, let's assume the audit logging had to be done via `console.log`. 
For just 2 API calls I had to write code like this:


```typescript

try {
    console.log(`Calling API 1 ${param1}, ${param2}`);
    const resultCall1 = await apiCall_1(param1, param2);
    console.log(`Call to API 1 Succeeded with result: ${resultCall1}`);
} catch (e) {
    console.log(`Call to API 1 Failed with error: ${e}`);
    throw e;
}

// ....

try {
    console.log(`Calling API 2 ${param1}`);
    const resultCall2 = await apiCall_2(param1);
    console.log(`Call to API 2 Succeeded with result: ${resultCall2}`);
} catch (e) {
    console.log(`Call to API 2 Failed with error: ${e}`);
    throw e;
}
```

This is obviously a very brittle and verbose solution. Moreover, I had to mix essential buisiness logic (the API calls)
with [cross-cutting aspects](https://en.wikipedia.org/wiki/Aspect_(computer_programming)) (i.e. the logging). 
Unfortunately, I couldn't find a flexible, stable, and type safe library for 
[Aspect Oriented Programming (AOP)](https://en.wikipedia.org/wiki/Aspect-oriented_programming) in TypeScript.

Fortunately, I managed to roll out my own minimalistic implementation to isolate the logging cross-cutting concerns.

<div id='implementation'/>
#Minimalistic Type Safe AOP

Let's start by defining a simple asyncrhonous logging function. For the purposes of this article, we'll just write to the standard output.
You can obviously have versions of this function which write to a databse, stream, or use a 3rd party logging library.

```typescript
const log = async (message: string) => {
  // Write to a DB, stream, etc.
  console.log(message);
}
```

Now let's implement a small utility function, which converts an arbitrary object to JSON text.
We'll use the [json-stringify-safe](https://www.npmjs.com/package/json-stringify-safe) library to avoid
problems with circular references.

```typescript
import * as stringify from 'json-stringify-safe';

const txt = (o: any) => stringify(o, null, 2);
```


Now let's define a super type of all operations which we need to audit.  
In the most general sense, we want to audit asynchronous functions (e.g. API calls) which take
arbitrary number of parameters. The following type definition of `AsyncFunction` represents a *super type*
for all such functions. In other words, every asyncrhonous function can be assigned to an `AsyncFunction` reference.

```typescript
type AsyncFunction = (...args: any[]) => Promise<any>;
```

Now we can define a utility higher order function called `auditWrap`. 
It takes as a parameter the function whose behaviour we need to audit log.
The type parameter `F` is sub-type of `AsyncFunction` and captures the exact 
compile-time type of `fn`. Since `auditWrap` returns a function of type `F`, the types
of `fn` and the result are the same. Thus we can preserve the type safety from the caller's point of view.


```typescript
const auditWrap = function <F extends AsyncFunction>(fn: F): F {
  const wrapper = async (...args: any[]) => {
    const stringArgs = args.map(txt).join(",\n"); // Textualize all arguements

    try {
      await log(`Attempting to call function: "${fn.name}" with arguements: ${stringArgs}`);
      const result = await fn(...args);
      await log(`Call to function "${fn.name}" suceeded with result: ${txt(result)}`);
      return result;
    } catch (e) {
      await log(`Call to function "${fn.name}" failed with message: ${e.message}, Details: \n${txt(e)}`);
      throw e;
    }
  };
  return wrapper as any;
};
```


<div id='usage'/>
#Usage

Let's demonstrate how to use `auditWrap` in practice to audit log a couple of API Calls:

```typescript
import axios from 'axios';

async function example() {
  const call1Result = await auditWrap(axios.get)('https://jsonplaceholder.typicode.com/todos/1');
  const call2Result = await auditWrap(axios.get)('https://jsonplaceholder.typicode.com/todos/2');
}

example()
```

The produced audit log is huge - every axios parameter and response element is printed out. Below is a subset of the output
(`...` denotes omited output):

```
Attempting to call function: "wrap" with arguements: "https://jsonplaceholder.typicode.com/todos/1"
Call to function "wrap" suceeded with result: {
  "status": 200,
  "statusText": "OK",
  "headers": {...},
  "config": {...},
  "request": {...},
  "data": {
    "userId": 1,
    "id": 1,
    "title": "delectus aut autem",
    "completed": false
  }
}

Attempting to call function: "wrap" with arguements: "https://jsonplaceholder.typicode.com/todos/2"
Call to function "wrap" suceeded with result: {
  "status": 200,
  "statusText": "OK",
  "headers": {...},
  "config": {...},
  "request": {...},
  "data": {
    "userId": 1,
    "id": 2,
    "title": "quis ut nam facilis et officia qui",
    "completed": false
  }
}
```

Notice that name of the function in the audit log is `wrap`. Indeed, if you print `axios.get.name`, you'll see that this is its actual
function name.

Nevertheless, the above aproach is a significant improvement. We managed to eliminate all `try`-`catch`-`log` boilerplate code and to preserve type safety.
However, we can do a bit better. In the above example, we generated a huge amount of log. What if we want to audit log
only specific parts of the output or have more readable function names? 

We can achieve this by introducing higher level function(s), which hide the details of the underlying libraries and return
exactly what we need:

```typescript
const apiCall = (url: string, config?: AxiosRequestConfig) => axios.get(url, config).then(d => d.data)

async function example() {
  const call1Result = await auditWrap(apiCall)('https://jsonplaceholder.typicode.com/todos/1');
  const call2Result = await auditWrap(apiCall)('https://jsonplaceholder.typicode.com/todos/2');
}

example()
```

The output of the above is:

```
Attempting to call function: "apiCall" with arguements: "https://jsonplaceholder.typicode.com/todos/1"
Call to function "apiCall" suceeded with result: {
  "userId": 1,
  "id": 1,
  "title": "delectus aut autem",
  "completed": false
}

Attempting to call function: "apiCall" with arguements: "https://jsonplaceholder.typicode.com/todos/2"
Call to function "apiCall" suceeded with result: {
  "userId": 1,
  "id": 2,
  "title": "quis ut nam facilis et officia qui",
  "completed": false
}
```



