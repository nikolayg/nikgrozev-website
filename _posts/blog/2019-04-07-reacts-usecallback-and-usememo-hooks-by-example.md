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

Before we start, let's introduce a helper button component. We'll use
[React.memo](https://reactjs.org/docs/react-api.html#reactmemo)
to turn it into a memoized component. This will force React to
never re-render it, unless some of its properties change.
We'll also add a random colour as its backgfound
so we can track when it re-rerenders:

```tsx
import React, { useState, useCallback } from 'react';

// Generates random colours any time it's called
const randomColor = () => '#'+(Math.random()*0xFFFFFF<<0).toString(16);

// The type of the props
type ButtonProps = React.ButtonHTMLAttributes<HTMLButtonElement>;

// A memoized button with a random background colour
const Button = React.memo((props: ButtonProps) => 
  <button onClick={props.onClick} style={{ background: randomColor() }}> 
    {props.children}
  </button>
)
```

Now let's look at the following simple app. It displays 2 numbers - a 
counter `c` and a `delta`. One button allows the user to increment `delta` by 1.
A second button, allows the user to increment the counter by adding `delta` to it.
We'll create 2 functions `increment` and `incrementDelta` and assign them to the buttons' on-click event handlers.
Let's also keep track of how many such functions are created while the user clicks the buttons:

```tsx
import React, { useState } from 'react';

// Keeps track of all created functions during the app's life 
const functions: Set<any> = new Set();

const App = () => {
  const [delta, setDelta] = useState(1);
  const [c, setC] = useState(0);

  const incrementDelta = () => setDelta(delta => delta + 1);
  const increment = () => setC(c => c + delta);

  // Register the functions so we can count them
  functions.add(incrementDelta);
  functions.add(increment);

  return (<div>
    <div> Delta is {delta} </div>
    <div> Counter is {c} </div>
    <br/>
    <div>
      <Button onClick={incrementDelta}>Increment Delta</Button>
      <Button onClick={increment}>Increment Counter</Button>
    </div>
    <br/>
    <div> Newly Created Functions: {functions.size - 2} </div>
  </div>)
}
```

When we run the app and start clicking the buttons we observe something interesting.
For every click of a button there are 2 newly created functions!
Futhermore, both buttons re-render on every change!

<figure>
  <img src="/images/blog/React useCallback and useMemo Hooks By Example/without-use-callback.png" alt="Without useCallback" >
  <figcaption>
    For every re-render of the component, 2 new functions are created. 
    Every change causes both buttoms to re-render.
  </figcaption>
</figure>

In other words, at every re-render we're creating 2 new functions.
If we increment `c`, why do we need to recreate the `incrementDelta` function?
This is not just about memory - it causes child components to re-render
unnecessarily. 
This can quickly become a performance issue.

One solution would be to move the two functions outside of the the `App` functional component. 
Unfortunately, this wouldn't work because they use the state variables from `App`'s scope.

# Naive solution - Why dependencies matter

This is where the [useCallback](https://reactjs.org/docs/hooks-reference.html#usecallback) hook comes in.
It takes as an arguement a function and returns a cached/memoized version of it.
It also takes a second parameter which will cover later. Let's rewrite with `useCallBack`:

```tsx
const App = () => {
  const [delta, setDelta] = useState(1);
  const [c, setC] = useState(0);

  // No dependencies (i.e. []) for now
  const incrementDelta = useCallback(() => setDelta(delta => delta + 1), []);
  const increment = useCallback(() => setC(c => c + delta), []);

  // Register the functions so we can count them
  functions.add(incrementDelta);
  functions.add(increment);

  return (<div>
    <div> Delta is {delta} </div>
    <div> Counter is {c} </div>
    <br/>
    <div>
      <Button onClick={incrementDelta}>Increment Delta</Button>
      <Button onClick={increment}>Increment Counter</Button>
    </div>
    <br/>
    <div> Newly Created Functions: {functions.size - 2} </div>
  </div>)
}
```

This prevents the instantiation of new functions and unnecessary re-renders.
However, when we re-run the app, we notice that we've introduced a bug. If we 
increment `detla` to 2, and then try to increment the counter, its value increases 
by 1, not by 2:

<figure>
  <img src="/images/blog/React useCallback and useMemo Hooks By Example/without-dependencies.png" alt="Without dependencies" >
  <figcaption>
    No new functions are created regardless of delta's state changes. During the 
    initial rendering, `useCallback` created a single cached version of the 
    "increment" function, which encapsulate the detla state value 
    and reused it on every re-render.
  </figcaption>
</figure>

This is because at the point of instantiation of the `increment` function, the value
of `delta` was 1, and this was captured in the function's scope. Since we're caching
the `increment` instance, it's never recreated and it uses its original scope with
`detla = 1`.  

The `useCallback` hook has created a single cached version of `increment`, which
encapsulates the **initial value** of `delta`. When `App` re-renders with different values for
`delta`, `useCallback` returns the previous version of the `increment` function
which keeps the old value of `delta` from the first rendering.

We need to tell `useCallback` to create new cached version of `increment` 
for every change of `delta`. 

# Dependencies

This is where the second arguement of `useCallback` comes in. It is an array of values,
which represents the **dependencies** of the cache. On any two subsequent re-renders,
`useCallback` will return the same cached function instance if the values of the dependencies are equal.

We can use dependencies to solve the previous bug:

```tsx
  const incrementDelta = useCallback(() => setDelta(delta => delta + 1), []);

  // Recreate increment on every change of delta!
  const increment = useCallback(() => setC(c => c + delta), [delta]);
``` 

Now we can see that a new `increment` function is created only when `delta` changes.
Therefore, the counter button only re-renders when `delta` changes, because
a new instance of the `onClick` property is added.
In other words,
**we only create a new callback, if the part of the closure it uses (i.e. the dependencies) 
has changed since the previous rendering**.

<figure>
  <img src="/images/blog/React useCallback and useMemo Hooks By Example/with-dependencies.png" alt="With dependencies" >
  <figcaption>
    A new `increment` function is created on every change of `delta`. 
    Only the function whose dependencies change is recreated.
  </figcaption>
</figure>

A really useful feature of `useCallback` is that it returns the **same function instance**
if the depencies don't change. Hence we can use it in the dependecy lists of other hooks. 
For example, let's create a cached/memoized function which increments both numbers:

```typescript
const incrementDelta = useCallback(() => setDelta(delta => delta + 1), []);
const increment = useCallback(() => setC(c => c + delta), [delta]);

// Can depend on [c1, c2] instead, but it would be brittle
const incrementBoth = useCallback(() => {
    incrementDelta();
    increment();
}, [increment, incrementDelta]); 
```

The new `incrementBoth` function transitively depends on `delta`. 
We could write `useCallback(... ,[delta])` and that would work.
However, this is a very brittle approach! If we changed the dependencies 
of `increment` or `incrementDelta`, we would have to remember to change the 
dependencies of `incrementBoth`.

Since the references of `increment` and `incrementDelta` won't change unless their dependencies change,
we could use them instead. Transitive dependencies can be ignored! 
This makes for a straightforward rule:


> Each function declared within a functional component's scope must be memoized/cached with `useCallback`.
> If it references functions or other variables from the component scope
> it should list them in its depency list.


This rule can be enforced by a 
[linter](https://www.npmjs.com/package/eslint-plugin-react-hooks) which checks that
your `useCallback` cache dependenices are consistent.

# Two similar hooks - useCallback and useMemo

React introduces another similar hook called [useMemo](https://reactjs.org/docs/hooks-reference.html#usememo).
It has similar signature, but works differently.
Unlike `useCallback`, which caches the provided function instance, `useMemo` invokes
the provided function and caches its result.

In other words `useMemo` caches a computed value. This is usefull when the computation
requires significant resources and we don't want to repeat it on every re-render, as in this example:

```typescript
const [c, setC] = useState(0);

// This value will not be recomputed between re-renders
// unless the value of c changes
const sinOfC1: number = useMemo(() => Math.sin(c) , [c])
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
