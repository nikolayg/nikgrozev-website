---
layout: post
title: Node.js Cheatsheet [Part 2] - Yarn
date: 2017-01-30 05:22:09.000000000
type: post
published: true
status: publish
excerpt: 
    Introduces the Yarn package manager and lists common commands in a cheatsheet ...
categories:
- Node
- JavaScript
- Cheatsheet
- blog
tags:
- Node
- JavaScript
- Yarn
- Cheatsheet
---


# Table of Contents

- [Introduction](#introduction)
- [Setup](#setup)
- [Install and Remove Packages](#install-remove-packages)
- [Upgrade Packages](#upgrade-packages)
- [Cache](#cache)
- [Scripts](#scripts)
- [Resources](#resources)

<div id='introduction'/>
# Introduction

In a [previous post](/2017/01/22/node-js-cheatsheet-part-1/) we looked
at Node.js and its package manager NPM. However, NPM has several shortcomings
in terms of performance and reproducibility.

Facebook's [Yarn](https://yarnpkg.com/) is a JavaScript package manager, 
which resolves NPM's problems and is a bit more user-friendly.
It parallelises better and caches all installed modules.
Most importantly, it works with NPM's repositories and preserves
the structure of the `package.json` file which makes it easy to migrate
existing NPM projects.

In this post, we will review Yarn's basic commands and use cases.

<div id='setup'/>
# Setup

Yarn can be installed as a global NPM module:

```bash
npm install -g yarn
```

This will give us the `yarn` command on our terminal path. Starting a
new NPM/Yarn project is as easy as:

```bash
# Very similar to "npm init"
# Will create a package.json
yarn init
```

If you already have an existing NPM project with a `package.json`, 
there is no need to initialise Yarn. Yarn commands will work right away.

<div id='install-remove-packages'/>
# Install and Remove Packages

Given an existing NPM/Yarn project, you can install all packages as:

```bash
# Similar to "npm install"
yarn install
```

This is similar to `npm install` but achieves two additional things. 
Firstly, it caches the new modules outside of `/node_modules`
so subsequent installations can be faster.
Secondly, it creates the `yarn.lock` file with the exact versions of all used
packages. In other words, even if a module has a range of version in its
`package.json`, subsequent yarn commands will only use the version 
specified in `yarn.lock`. This is similar to the role of `Gemfile.lock`
in Ruby's package manager [Bundler](http://bundler.io/).

Adding and removing new packages/modules is similar to NPM, but by default
Yarn saves all new modules in the `package.json`. This is equivalent to
the using the `--save` flag in NPM:

```bash
# Adds the latest Express to package.json"
# Similar to "npm install express --save"
yarn add express

# The same, but with a version
yarn add express@3.0.0

# Adds the latest Express as a "dev" dependency
# Similar to "npm install express --save --dev"
yarn add express

# Adds the latest Express to package.json"
# Similar to "npm install express --save"
yarn add express

# Remove a package
yarn remove express
```

Each of thes commands modifies the `yarn.lock` and `package.json` files
and populates the Yarn cache.

Working with global packages is pretty easy as well:

```bash
# Add a package globally
yarn globall add nodemon
yarn globall add nodemon@1.11.0

# Remove a global package
yarn globall remove nodemon
```

<div id='upgrade-packages'/>
# Upgrade Packages

Yarn allows you to easily upgrade and downgrade packages. 
It has a special command for inspecting your dependencies
and identifying if any of them is outdated:

```bash
# Prints if any of your dependencies is outdated
yarn outdated
```

To upgrade/downgrade a package, we can simply use:

```bash
# Upgrades Express to the newest version
yarn upgrade express

# Upgrades/Downgrades Express to a specific version
yarn upgrade express@3.0.0
```

The `upgrade` command modifies both the `yarn.lock` and `package.json` files.

<div id='cache'/>
# Cache

Yarn keeps all modules it installed in cache outside of the `/node_modules`
folder. When needed, these modules can be quickly retrieved from the cache.
For the most parts, we should not be concerned with the caching at all.
However, in some rare occasions (e.g. an internal module changed without 
increasing its version) we may need to clean the cache:

```bash
# Force Yarn to clean its cache
yarn clean cache
```

<div id='scripts'/>
# Scripts 

We can specify scripts in the `package.json` file like this:

```json
"scripts" : {
  "start": "node server.js",
  "test": "echo MISSING TESTS"
}
```

With NPM, we could run them with the `npm run` command. Yarn allows for this as well:

```bash
# Run a script from "package.json"
yarn run start
yarn run test

# Lists all scripts and lets you choose
yarn run
``` 

<div id='resources'/>
# Resources

Here are some nice resources on Yarn:

- [Andrew Mead's short course](http://www.mead.io/yarn/)
- [Different ways to install Yarn](https://yarnpkg.com/en/docs/install)
