---
layout: post
title: Common Types for Redux Actions and Reducers
date: 2017-09-16 05:22:09.000000000
type: post
published: true
status: publish
excerpt: 
    In my past React/Redux projects which use Flow or Typescript, I ended up writing similar 
    type definitions for the Redux actions and reducers. 
    I decided to extract these as generic type definitions,
    which can be reused in the future ... 
categories:
- JavaScript
- Node
- NPM
- TypeScript
- Flow
- React
- Redux
- blog
tags:
- JavaScript
- Node
- NPM
- TypeScript
- Flow
- React
- Redux
author:
  login: nikolay.grozev@gmail.com
  email: nikolay.grozev@gmail.com
  display_name: nikolay.grozev@gmail.com
  first_name: 'Nikolay'
  last_name: 'Grozev'
---


<div id='introduction'/>
# Introduction

I have been working on a few [React](https://facebook.github.io/react/)/
[Redux](http://redux.js.org/) projects recently.
Having experience with many statically typed languages, I found it natural to
use the [Flow](https://flow.org/) type checker or [TypeScript](https://www.typescriptlang.org).
Static types can significantly increase code maintainability - watch 
[Jared Forsyth on React Conf](https://www.youtube.com/watch?v=V1po0BT7kac) if you
are not a believer in static types.

In all projects, I ended up writing very similar type definitions for the
Redux actions and reducers. I decided to extract these as reusable generic 
type definitions. 

Fortunately, the syntaxes of [Flow](https://flow.org/) and [TypeScript](https://www.typescriptlang.org)
are very similar. The following type definitions are valid for both.

<div id='actions'/>
# Actions

In Redux, an [action](http://redux.js.org/docs/basics/Actions.html) is an object 
dispatched from the application code to the redux store. Most actions
have a string property called `type` and a `payload` property. The following generic type
denotes this: 

```javascript
export interface Action<Payload> {
  type: string;
  payload: Payload;
}
``` 

Many actions represent API calls to a server. In this case, they `payload` would be
a promise that encapsulates the HTTP request. We can use a 
[redux middleware](https://github.com/pburtchaell/redux-promise-middleware)
that automatically resolves the payload promises before firing the reducers.
Here is a type for this:

```javascript
export type APIAction<Payload> = Action<Promise<Payload>>;
```

<div id='action_cretors'/>
# Action Creators

[Action creators](http://redux.js.org/docs/basics/Actions.html#action-creators)
are functions that return actions. A generic action creator can be defined as:

```javascript
export type ActionCreator<Params, Payload> = (params: Params) => Action<Payload>;
```

A creator which returns an `APIAction`:

```javascript
export type APIActionCreator<Params, Payload> = (params: Params) => APIAction<Payload>;
```

<div id='reducers'/>
# Reducers

A [reducer](http://redux.js.org/docs/basics/Reducers.html) is a function that
combines the state and an action to produce the next state. It can be defined as:

```javascript
export type Reducer<State, Payload> = (state: State, action: Action<Payload>) => State;
```

<div id='examples'/>
# Example

Let's look at an example of using the type definitions for an action and a
reducer which retrieve a student record from an API:

```javascript
// Student is the model type. The redux state is a just list of students
type Student = { id: string, gpa: number };
type State = Student[];

// An API action creator - the app will dispatch the returned action
const getStudent: APIActionCretor<string, Student> = (id) => {
    return {
        type: "GET_STUDENT", 
        // assuming the API call returns Promise<Student>
        payload: callApi(`http://endpoint/${id}`)
    };
}

// Reducer - will merge the state and the resolved promise
const studentReducer: Reducer<State, Student> = (state, action) => {
    switch (action.type) {
        case "GET_STUDENT":
            return [action.payload, ...state];
        default:
            return state
    }
}
``` 

<div id='npm_module'/>
# NPM Modules

I have published the type definitions to NPM, so they can be used from multiple projects.

If you're using [Flow](https://flow.org/), then you can install the `redux-common-types-flow` module:

```bash
npm install --save redux-common-types-flow
```

Then you can import the types:

```javascript
import {type APIActionCreator} from 'redux-common-types-flow';
import {type Action} from 'redux-common-types-flow';
// ....
```

If TypeScript is your cup of tea, then install `redux-common-types-ts`:

```bash
npm install --save redux-common-types-ts
```

Now you can import as:

```javascript
import { APIActionCreator, Action, /*..*/} from "redux-common-types-ts";
```

