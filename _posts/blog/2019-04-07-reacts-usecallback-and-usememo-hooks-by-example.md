---
layout: post
title: React's useCallback and useMemo Hooks By Example
date: 2019-04-07 05:22:09.000000000
type: post
published: true
status: publish
excerpt: 
    This article demonstrates with a few simple examples why we need the useCallback and useMemo hooks
    and when and how to use them.  
categories:
- JavaScript
- TypeScript
- React
tags:
- JavaScript
- TypeScript
- Hooks
- React
---

# Introduction

I recently started learning about the [React hooks](https://reactjs.org/docs/hooks-reference.html) API and I was
amazed by how expressive it is. Hooks allow me to rewrite tens of lines of boilerplate code with
just a few lines. Unfortunately, this convenience comes at a cost. 
I found that some more advanced hooks like `useCallback` and `useMemo`
are hard to learn and appear counter-intuitive at first.

In this article, I'll demonstrate with a few simple examples why we need these hooks
and when and how to use them. This is not an introduction to hooks, and you must be familiar
with the [useState](https://reactjs.org/docs/hooks-reference.html#usestate) hook to follow. 


# The Problem

Let's look at the following simple app. It displays 2 counters and allows the user to increment them with 2 buttons.
We'll create 2 functions `increment1` and `increment2` and assign them to the buttons' on-click event handlers.
Let's also keep track of how many such functions are created while the user clicks the buttons:

```tsx
import React, { useState } from 'react';

// Keeps track of all created functions during the app's life 
const functions: Set<any> = new Set();

const App = () => {
  const [c1, setC1] = useState(0);
  const [c2, setC2] = useState(0);

  const increment1 = () => setC1(c1 + 1);
  const increment2 = () => setC2(c2 + 1);

  // Register the functions so we can count them
  functions.add(increment1);
  functions.add(increment2);

  return (<div>
    <div> Counter 1 is {c1} </div>
    <div> Counter 2 is {c2} </div>
    <br/>
    <div>
      <button onClick={increment1}>Increment Counter 1</button>
      <button onClick={increment2}>Increment Counter 2</button>
    </div>
    <br/>
    <div> Newly Created Functions: {functions.size - 2} </div>
  </div>)
}
```

Once we run the app and start clicking the buttons we observe something interesting.
For every click of a button there are 2 newly created functions!
In other words, at every re-render we're creating 2 new functions which is excessive.
If we increment the `c1` counter, why do we need to recreate the `increment2` function?
If we repeat this error many times on a big app, this may become a performance issue.

<figure>
  <img src="/images/blog/React useCallback and useMemo Hooks By Example/without-use-callback.png" alt="Without useCallback" >
  <figcaption>
    For every re-render of the component, 2 new functions are created. 
    In this example, 5 state changes result in 10 new functions.
  </figcaption>
</figure>

One solution would be to move the two functions outside of the the `App` functional component. 
Unfortunately, this wouldn't work because they use the state variables from `App`'s scope.

# Naive solution - Why dependencies matter

This is where the [useCallback](https://reactjs.org/docs/hooks-reference.html#usecallback) hook comes in.
It takes as an arguement a function and returns a cached/memoized version of it.
It also takes a second parameter which will cover later. Let's rewrite with `useCallBack`:

```tsx
const App = () => {
  const [c1, setC1] = useState(0);
  const [c2, setC2] = useState(0);

  // Cache/memoize the functions - do not create new ones on every rerender
  // Let's ignore the dependencies for now - i.e. use []
  const increment1 = useCallback(() => setC1(c1 + 1), []);
  const increment2 = useCallback(() => setC2(c2 + 1), []);

  // Register the functions so we can count them
  functions.add(increment1);
  functions.add(increment2);

  return (<div>
    <div> Counter 1 is {c1} </div>
    <div> Counter 2 is {c2} </div>
    <br/>
    <div>
      <button onClick={increment1}>Increment Counter 1</button>
      <button onClick={increment2}>Increment Counter 2</button>
    </div>
    <br/>
    <div> Newly Created Functions: {functions.size - 2} </div>
  </div>)
}
```

When we re-run the app, we notice that we've introduced a bug. We can keep clicking
the increment buttons, but the counters' values never increase beyond 1. There're no newly 
created functions, and this is the cause of the issue.

<figure>
  <img src="/images/blog/React useCallback and useMemo Hooks By Example/without-dependencies.png" alt="Without dependencies" >
  <figcaption>
    No new functions are created regardless of the counters' state changes. During the initial rendering, 
    `useCallback` created single cached versions of the functions, which encapsulate the state values, 
    and reused them on every re-render.
  </figcaption>
</figure>

The `useCallback` hook has created single cached versions of the two functions, which
encapsulate the **initial values** of `c1` and `c2`. When `App` re-renders with different values for
`c1` and `c2`, `useCallback` returns the previous versions of the functions
which keep the old values of `c1` and `c2` from the first rendering.

We need to tell `useCallback` to create new cached versions of the functions 
for every change of values of `c1` and `c2` that they depend on. 

# useCallback and Dependencies

This is where the second arguement of `useCallback` comes in. It is an array of values,
which represents the **dependencies** of the cache. On any two subsequent re-renders,
`useCallback` will return the same cached function instance if the values of the dependencies are equal:

```tsx
const App = () => {
  const [c1, setC1] = useState(0);
  const [c2, setC2] = useState(0);

  // The useCallback caches depend on the values of c1 and c2
  const increment1 = useCallback(() => setC1(c1 + 1), [c1]);
  const increment2 = useCallback(() => setC2(c2 + 1), [c2]);

  // Register the functions so we can count them
  functions.add(increment1);
  functions.add(increment2);

  return (<div>
    <div> Counter 1 is {c1} </div>
    <div> Counter 2 is {c2} </div>
    <br/>
    <div>
      <button onClick={increment1}>Increment Counter 1</button>
      <button onClick={increment2}>Increment Counter 2</button>
    </div>
    <br/>
    <div> Newly Created Functions: {functions.size - 2} </div>
  </div>)
}
``` 

Now we can see that the number of newly created functions is the same as the number of
re-renders, which is twice as good as the original example without `useCallback`. In other words,
**we only create new callbacks, if the part of the closure they use (i.e. their dependencies) 
has changed since the previous rendering**.

<figure>
  <img src="/images/blog/React useCallback and useMemo Hooks By Example/with-dependencies.png" alt="With dependencies" >
  <figcaption>
    A new function is created on every state change. Only the function whose dependencies change is recreated.
  </figcaption>
</figure>

A really useful feature of `useCallback` is that it returns the **same function instance**
if the depencies don't change. Hence we can use it in the dependecy lists of other hooks. 
For example, let's create a cached/memoized function which increments both counters:

```typescript
const increment1 = useCallback(() => setC1(c1 + 1), [c1]);
const increment2 = useCallback(() => setC2(c2 + 1), [c2]);

// Could depend on [c1, c2] instead, but it would be brittle
const incrementBoth = useCallback(() => {
    increment1();
    increment2();
}, [increment1, increment2]); 
```

The new `incrementBoth` function transitively depends on `c1` and `c2`. 
We could write `const incrementBoth = useCallback(... ,[c1, c2])` and that would work.
However, this is brittle! If we changed the dependencies of `increment1` or `increment2`,
we would have to remember to change the dependencies of `incrementBoth`.

Since the references of `increment1` and `increment2` won't change unless their dependencies change,
we could use them instead. Transitive dependencies can be ignored! 
This makes for an easy rule - *list as dependency any
function or other variable from the component scope that you're using*. 
This can be enforced by a 
[linter](https://www.npmjs.com/package/eslint-plugin-react-hooks) which checks that
your `useCallback` cache dependenices are consistent.

# useCallback vs useMemo

React introduces another similar hook called [useMemo](https://reactjs.org/docs/hooks-reference.html#usememo).
It has similar signature, but works differently.
Unlike `useCallback`, which caches the provided function instance, `useMemo` invokes
the provided function and caches its result.

In other words `useMemo` caches a computed value. This is usefull when the computation
requires significant resources and we don't want to repeat it on every re-render, as in this example:

```typescript
const [c1, setC1] = useState(0);
const [c2, setC2] = useState(0);

// This value will not be recomputed between re-renders
// unless the value of c1 changes
const sinOfC1: number = useMemo(() => Math.sin(c1) , [c1])
```

Just as with `useCallback`, the values returned by `useMemo` can be used as other hooks' dependencies. 

As an interesting aside, `useMemo` can cache a function value too.
In other words, it is a generalised version of `useCallback` and can replace it 
as in the following example

```typescript
// Some function ...
const f = () => { ... }

// The following are functionally equivalent
const callbackF = useCallback(f, [])
const callbackF = useMemo(() => f, [])
```
