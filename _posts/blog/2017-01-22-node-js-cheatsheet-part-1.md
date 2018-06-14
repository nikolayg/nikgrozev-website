---
layout: post
title: Node.js Cheatsheet [Part 1]
date: 2017-01-22 05:22:09.000000000
type: post
published: true
status: publish
excerpt: 
    A cheatsheet of common Node.js tools, commands, concepts, and techniques  ...
categories:
- Node
- JavaScript
- Cheatsheet
tags:
- Node
- JavaScript
- Cheatsheet
---


# Table of Contents

- [Introduction](#introduction)
- [Setup (Node, NVM, and NPM)](#setup-node-nvm-npm)
    - [Node Version Manager (NVM)](#nvm)
    - [The Node Package Manager (NPM)](#npm)
    - [Run, Debug, and Reload](#run-debug-and-reload)
- [Module System](#module-system)
    - [Pre-ES6 Modules](#pre-es6-modules)
    - [ES6 Modules](#es6-modules)
- [Environment Variables](#environment-variables)
- [Compiling with Babel](#babel)
- [The Flow Typechecker](#flow)
- [Promises](#promises)


<div id='introduction'/>
# Introduction

[Node.js](https://nodejs.org/) (Node) is an open source runtime environment 
backed by a huge developer community. It is often used 
to prototype web services and backends. Using Node, developers can reuse
front-end skills and libraries for the backend. This post is a cheatsheet of common 
Node tools, commands, concepts, and techniques.

<div id='setup-node-nvm-npm'/>
# Setup (Node, NVM, and NPM)

<div id='nvm'/>
## Node Version Manager (NVM)

Node evolves quickly and sometimes new releases break backward compatibility. 
Thus, you may need to maintain and switch between different versions. 
The [Node Version Manager (NVM)](https://github.com/creationix/nvm) comes to the rescue.
Like Ruby's [rvm](https://rvm.io/) and Python's [pyenv](https://github.com/yyuu/pyenv),
NVM allows us to work with multiple Node environments. You can install it from 
[here](https://github.com/creationix/nvm).


The following commands list all available versions:

```bash
# List all locally installed Node-s
nvm ls

# List all available node releases
nvm ls-remote
```

Here is how to install new Node versions:

```bash
# Install the latest long term support (LTS) version
nvm install --lts

# Install the latest release (may not be LTS)
nvm install node

# Install a specific version
nvm install 7.0.0
```

When installing a new Node version, you may want to migrate existing
global Node packages. This will ensure they're compatible wit the new version.
To do so, use the `--reinstall-packages-from=node` option:

```bash
# Installs 6.9.0 and migrates global packages
nvm install 6.9.0 --reinstall-packages-from=node
```

Switching to another installed version goes like this:

```bash
nvm use 7.4.0
```

<div id='npm'/>
## The Node Package Manager (NPM)

[NPM](https://www.npmjs.com/) is Node's default package manager and is similar to Ruby's
[Bundler](http://bundler.io/) and Python's [PIP](https://pip.pypa.io/en/stable/).
It comes preinstalled with Node. To create a new project in a folder:

```bash
npm init
```

This will create a `package.json` file which contains all project dependecies and metadata. 
It can also include a `scripts` section which predefines command 
aliases. Here is an example of a `package.json` file:

```json
{
  "name": "project-name",
  "version": "1.0.0",
  "description": "Example project",
  "main": "server.js",
  "scripts": {
    "test": "echo \"We do not have tests yet!\" && exit 1"
  },
  "author": "me",
  "license": "ISC",
  "dependencies": {
    "request": "^2.60.0",
    "yargs": "^3.15.0"
  },
  "devDependencies": {
    "babel-cli": "^6.22.2"
  }
}
```

Given this sample, we can run the `test` command alias as follows:

```bash
npm test
```

You can modify the `package.json` file manually, or add new dependencies 
via the `npm install` command with the `--save` flag:

```bash
# Install the latest version of the request package 
# and save the dependency in package.json 
npm install request --save

# Install a specific version of the
# package and save the dependency in package.json 
npm install request@2.60.0 --save
```

We can also install a library as a "dev" dependency with the `--save-dev` flag.
This is useful for tools which we won't need in 
production -- e.g. testing libraries. Such dependencies are defined in the
`devDependencies` section of the `packages.json`.

```bash
npm install babel-cli --save-dev 
```

All modules are stored in the `./node_modules` folder and it should
be in your `.gitignore`. You can reinstall all modules with: `npm install`.

NPM alllows the installation of *global packages*. They are not placed in 
`./node_modules` and are shared across all Node environments. 
In general, you should avoid global packages except for global utilities
like debuggers, compilers, and performance monitoring tools. The `-g` flag denotes
that a package is installed globally:

```bash
# Install nodemon globally
npm install nodemon -g
``` 

<div id='run-debug-and-reload'/>
## Run, Debug, and Reload

A Node interpreter (REPL) can be started with the `node` command.
This opens up a REPL for running Node commands interactively. To quit we need 
to type `.exit`:

```bash
> node # start a Node REPL
> console.log('Hey node!');
Hey node!
undefined
> .exit //Quit
```

Within a Node project folder, you can execute a Node script, which is a JavaScript file.
To debug, use the `debug` parameter, which starts the script in debug mode. 
Then you can step through the
code via the *continue* (`c`), *step over/next*(`n`), and *step into* (`s`)
commands. The `repl` command allows you to check the values of
variables and expressions in the respective context. Breakpoints
are put directly in the code as `debugger` statements:

```bash
# Run a file called script.js in a Node proj folder
node script.js

# Run script.js in debug mode
node debug script.js
```

Node does not automatically refresh/reload upon code changes
and needs to be restarted. Hence, it is convenient to use the `nodemon`
tool which automatically reloads the Node environment. First you need to
install it globally:

```bash
npm install nodemon -g
```

Then you can use it in place of the `node` command:

```bash
# Runs server.js and reloads on change
nodemon server.js
```

<div id='module-system'/>
# Module System

Node does not have logical modular constructs like packages or namespaces. 
Modules are either files in the project folder, or external NPM packages.
**The latest version of Node does not natively support the ES6 `import` and
`export` functionalities**. To use them, you need to set-up [Babel](#babel) compilation. 
Furthermore, most existing JavaScript code uses
pre-ES6 syntax. Thus, we will overview both ways of working with modules.

<div id='pre-es6-modules'/>
## Pre-ES6 modules

Before ES6, modules could export a single variable called `module.exports`. 
Client modules would use the `require` function to load its value:

For example, we could define a module in a file `./friends.js`:

```javascript
let johny = { name: 'johny' };
let mary = { name: 'mary' };

// This is what client modules will see
// when they include/require this module
module.exports = {
  friends: [johny, mary]
};
```

Now lets import/include `friends.js` and a 3rd party module/package `moment`:

```javascript
// Load the moment library - installed with  NPM
let moment = require('moment');

// Load the friends.js module. You need to specify its relative path.
// The value of "myFriends" will be equal to the above "module.exports"
let myFriends = require ('./friends.js');
```

<div id='es6-modules'/>
## ES6 modules

In ES6, a module can export multiple elements with the `export` command.
A module can optionally have a **default export**, which is automatically
imported when you include the module:

```javascript
// You can export at the time of definition
export const exportedVar = { name: 'I am exported' };
export function exportedFunc() { 
  console.log('I am exported'); 
};

// You can export after definition
const exportMeLater = { name: 'Will be exported' };
export { exportMeLater };

// You can have a single default export
const defaultExport = { name: 'I am a default export' };
export default defaultExport;
``` 

External modules are loaded via the `import` command which has several variations.
Assuming the previous example is in a file called `importExample.js`, we could work 
with it as follows:

```javascript
// The default export does not need to be in "{}". The named (i.e. non-default) 
// exports need to be and you can cherry-pick what you import.
import defaultExport { exportedVar, exportMeLater } from './importExample.js'
```


<div id='environment-variables'/>
# Environment Variables

Application configuration is often provided in the form of environment variables.
Examples of such configuration are: environment identifier (e.g. dev/staging/prod) and database
connection url. In Node, environment variables can be accessed through the 
`proces.env` object. For example:

```bash
# Start node with an environment variable
someVar=value node 
>
> // access env var
> console.log(process.env.someVar);
value
```

If Node is running a script (i.e. not in REPL mode), you can access the special variables
`__dirname` and `__filename` which denote the paths to the currently executing script/module
and its directory. This comes handy when accessing resources from relative folders.

<div id='babel'/>
# Compiling with Babel

The latest versions of Node support ES6 almost fully. However, some features (e.g. 
ES6-style import/export) are not yet supported. If we want to use them,
we'll have to compile our code to an earlier JavaScript version with 
[Babel](https://babeljs.io). 

We need to install it as a dev dependency:

```bash
npm install --save-dev babel-cli
```

Babel uses presets, which define what code transformations will be applied.
Let's install a basic ES6 preset:

```bash
npm install babel-preset-env --save-dev
```

Babel loads the preset from its config file `.babelrc`. Hence, we should specify it there as:

```json
{
  "presets": ["env"]
}
```

In the most typical use case, Babel would take a source folder with ES6 code and generate
the compiled output in another folder. If all our code is in the `src/` 
sub-folder, we can compile to the `build/` folder as follows:

```bash
./node_modules/.bin/babel src/ -d build/
```

We can then run the newly generated code in `build/` as normal Node code by using the
`node` or `nodemon` commands. Babel can monitor for changes, and compile
automatically with the `--watch` option:

```bash
./node_modules/.bin/babel --watch src/ -d build/
```

The `babel-node` command combines compilation and execution in one step. 
Given a file `srct/test.js` you can compile and run as:

```bash
./node_modules/.bin/babel-node src/test.js
```

So how do we run this with `nodemon`? Here it is:

```bash
nodemon --exec ./node_modules/.bin/babel-node src/test.js
```

It is convenient to define command aliases in `package.json`.
Assuming a file `src/test.js` exists, we can use:  

```json
"scripts": {
  "package": "babel src/ -d build/",
  "runDev": "babel-node src/test.js",
  "debugDev": "babel-node debug src/test.js",
  "monitorDev": "nodemon --exec babel-node src/test.js"
}
```

And then we can invoke them with:

```bash
npm run package    # Compiles all the code
npm run runDev     # Compiles and runs
npm run debugDev   # Compiles and debugs
npm run monitorDev # Compiles, runs, and monitors
```

**NOTE:** We could have installed Babel globally, like we did with nodemon.
In this case, we would have all babel utilities on the Path, and we would not
have to use relative paths (i.e. `./node_modules/.bin`). Both approaches are
possible.  

<div id='flow'/>
# The Flow Typechecker

Many developers prefer statically typed languages like Scala, Java, and C++.
While there are compilers from such languages to JavaScript (e.g. 
[TypeScript](https://www.typescriptlang.org/), [GWT](http://www.gwtproject.org/), and 
[Scala.js](https://www.scala-js.org/)), incorporating type safety into existing JavaScript
code bases is far from trivial. 

Enter [Flow](https://flowtype.org/)! It is a static type checker for JavaScript. You
can incrementally add Flow type annotations to your JavaScript code instead of rewriting 
it all in another language. 

First, lets install Flow globally:

```bash
npm install flow-bin -g
```

This will add the `flow` command to your terminal environment. Then we need to initialise
Flow within the node project directory:

```bash
flow init
```

This will create a `.flowconfig` file with Flow settings.

JavaScript syntax does not support type annotations and adding them would result in 
syntax errors. One workaround is to add the types as comments which Flow will interpret:

```javascript
//@flow
const c /*: number*/ = 'Not a number';
```

Note the `//@flow` comment at the beginning of the file. It tells Flow to process the file --
otherwise it will ignore it. To analyse the code for type violations, just run `flow` from
the project's folder. This should report all type errors.

Defining type information in comments is not very convenient. We can use Babel to remove 
the type annotations for us. Given the Babel setup from the previous secion, we also need 
to install a plugin for Flow:

```bash
npm install babel-plugin-transform-flow-strip-types --save-dev
```

And then the `.babelrc` file must be configured to use it:

```json
{
  "presets": ["env"],
  "plugins": ["transform-flow-strip-types"]
}
```

Now we can use types in the code:

```typescript
// @flow
function f(name: string): string {
    return `Hello ${name}`;
}

console.log(f('Bob'));    // Types match - all Good
console.log(f(5));        // Will generate Flow error
```

We can also enable Flow to type check how we use 3rd party libraries. There is a central
repository of type definitions for many popular libraries. It can be accessed via the
`flow-typed` module:

```bash
npm install flow-typed -g
```

This will enable Flow to dynamically pull library type definitions.

Finally, don't forget to install a Flow plugin for your favorite editor/IDE to get type
hints as you code.

<div id='promises'/>
# Promises

Promises are available in JavaScript and are not specific to Node.
However, they are quite important for many Node libraries for async operations, 
and thus we'll review them quickly. A promise allows us to work succinctly with 
async operations without nested boilerplate callbacks.

A promise is an instance of class `Promise`. It is a wrapper of a higher order function
which takes two callback functions as parameters called `resolve` and `reject`. The function
calls them when the encapsulated operation succeeds or fails respectively. The
following example shows a promise which succeeds if a random number is less than 0.5:

```javascript
// A Promise wraps a higher order function.
// "resolve" and "reject" are callback functions.
function promiseFunction(resolve, reject) {
  const rn = Math.random();

  if(rn < 0.5) {
    // Signal for success
    resolve(rn);
  } else {
    // Signal for failure
    reject('Bad luck');
  }
};

// Wrap the function in a Promise
let promise = new Promise(promiseFunction);
```

We can "chain" actions after a promise with the `then` method, which is also a higher order function.
It takes two parameters -- a function which is called "on success" and one for failure.
Alternatively, we can provide only the "on success" function, and then chain a call 
to the `catch` method:

```javascript
// If called, the "number" parameter is the arguement of the 
// resolve function call from the promise
const successF = number => {
  console.log(`Resolve was called with ${number}`);
}

// If called, the "err" parameter is the arguement of 
// the reject function call from the promise
const errF = err => {
  console.log(`Reject was called with "${err}"`);
}

// The same as "promise.then(successF).catch(errF);"
promise.then(successF, errF);
```

Often, we need to run multiple async operations in a sequence. For example, we can
call a web service, and if successfull make an SQL query, and if successfull ...
In other words, we need to chain promises. We can easily achieve this by returning
a promise from our success-handling function:

```javascript
// A success handling function that reuturns a promise itself
const handleSuccessWithAnotherPromise = number => {
  return new Promise(promiseFunction);
}

// Prints a number if two consequtive invocations of
// "Math.random" returned < 0.5
promise.
  // Handle with a function returning a promise
  then(handleSuccessWithAnotherPromise). 

  // Now we can chain. "console.log" will be run
  // only if both promises have succeeded
  then(console.log).

  // Catch errors of any promise from the chain
  catch(console.log);
```


