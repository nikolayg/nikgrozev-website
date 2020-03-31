---
layout: post
title: Improve Your TypeScript With Static Analysis
date: 2020-03-22 05:22:09.000000000
type: post
published: true
status: publish
excerpt: 
    This article describes a set of tools for static analysis and automated audits
    of TypeScript code.
categories:
- JavaScript
- TypeScript
tags:
- TypeScript
- JavaScript
- Static Analysis
- SonarQube
- ESLint
- Husky
- Node
---

# Table of Contents

- [Introduction](#introduction)
- [ESLint and Friends](#eslint)
- [Jest and Code Coverage](#jest)
- [Jest and Create React App (CRA)](#jest-cra)
- [Dependency Audit / Supply Chain Analysis](#dep-audit)
- [Local SonarQube](#sonar-qube)
- [Run SonarQube Analysis](#sonar-analysis)
- [Git Hooks with Husky](#husky)


<div id='introduction'/>
# Introduction

Recently, I've been working on a number of TypeScript projects. 
This includes both back-end Node JS APIs and front-end React user intefaces. 
I wanted to automate the code quality assessment as much as possible, 
but it turned out this is not trivial.

In this post, I'll describe a set of tools for static analysis and automated audits.
I'll demonstrate how to configure and execute them locally so you can get quick 
feedback during development. The same scripts and commands can be used in any CI/CD tool.

Most of these tools and configurations are not specific to TypeScript 
and can be adapted to plain JavaScript. However, if you're looking at automated 
code quality tooling for large code bases
[then TypeScript is your friend](https://www.youtube.com/watch?v=wYgSiFaYSSo).

There are many code snippets in this post. At any point, you can refer to 
[the GitHub repo](https://github.com/nikolayg/secure-typescript-boilerplate) 
which has all these tools and configurations ready to go. 
Just clone and play with it!

<div id='eslint'/>
# ESLint and Friends

[ESLint](https://eslint.org/) has replaced [TSLint](https://palantir.github.io/tslint/)
as the go-to static analsys tool for TypeScript. ESLint has a 
plugin architecture, where additional rules and language support can be 
installed and configured. 
You can configure ESLint's plugins and rules 
via a `.eslintrc.js` file in the root of your project.

We'll need the following ESLint plugins and auxiliary packages:

- [@typescript-eslint/parser](https://www.npmjs.com/package/@typescript-eslint/parser) - ESLint parser for TypeScript;
- [@typescript-eslint/eslint-plugin](https://www.npmjs.com/package/@typescript-eslint/eslint-plugin) - ESLint rules for TypeScript;
- [eslint-plugin-import](https://www.npmjs.com/package/eslint-plugin-import) - rules ensuring the proper use of `import` statements;
- [eslint-plugin-sonarjs](https://www.npmjs.com/package/eslint-plugin-sonarjs) - code quality ESLint rules from [Sonar Source](https://www.sonarsource.com/);

[Prettier](https://prettier.io/) is a popular code formatting tool. If you want to integrate it with ESLint, we'll need the following. 
- [prettier](https://www.npmjs.com/package/prettier) - the [Prettier](https://prettier.io/) tool itself; 
- [eslint-config-prettier](https://www.npmjs.com/package/eslint-config-prettier) - rules for the [Prettier](https://prettier.io/) code formatter; 
- [eslint-plugin-prettier](https://www.npmjs.com/package/eslint-plugin-prettier) - turns off ESLint rules which might conflict with Prettier;
- [prettier-eslint](https://www.npmjs.com/package/prettier-eslint) - allows ESLint to 
[auto-fix](https://eslint.org/docs/user-guide/command-line-interface#fix) formatting issues
in your code;


If you're using React, you'll also need:
- [eslint-plugin-react](https://www.npmjs.com/package/eslint-plugin-react) - rules for React;
- [eslint-plugin-react-hooks](https://www.npmjs.com/package/eslint-plugin-react-hooks) - enforces the [React hook rules](https://reactjs.org/docs/hooks-rules.html);

Let's install all of these via `npm` or `yarn`:

```bash
# Core TypeScript ESLint support
npm install --save-dev @typescript-eslint/parser 
npm install --save-dev @typescript-eslint/eslint-plugin

# Extra code quality rules
npm install --save-dev eslint-plugin-import 
npm install --save-dev eslint-plugin-sonarjs

# Prettier and its ESLint integration
npm install --save-dev prettier 
npm install --save-dev eslint-config-prettier 
npm install --save-dev eslint-plugin-prettier 
npm install --save-dev prettier-eslint

# React rules for ESLint (Optional)
npm install --save-dev eslint-plugin-react eslint-plugin-react-hooks
```

Now let's create the `.eslintrc.js` configuration file. 
[Here is the full file](https://github.com/nikolayg/secure-typescript-boilerplate/blob/master/.eslintrc.js), below is just a highlight of the important sections:

```js
// `.eslintrc.js
module.exports = {
  parser: '@typescript-eslint/parser', // Parse TypeScript
  parserOptions: {
    project: './tsconfig.json',
    jsx: false // True for React
  },

  rules: { 
    /* disable or configure individual rules */

    /* Will need the following for React hooks: */
    // "react-hooks/rules-of-hooks": "error",
    // "react-hooks/exhaustive-deps": "warn"
  },

  // Use the rules from these plugins 
  extends: [
    'plugin:@typescript-eslint/recommended',
    'prettier/@typescript-eslint',
    'prettier',
    'plugin:prettier/recommended',
    'plugin:sonarjs/recommended',
    // 'plugin:react/recommended' // If we need React
  ]
};
```

You can configure the behaviour or Prettier with a [.prettierrc](https://github.com/nikolayg/secure-typescript-boilerplate/blob/master/.prettierrc) and [.prettierignore](https://github.com/nikolayg/secure-typescript-boilerplate/blob/master/.prettierignore)
files in the root of the project.

Finally, let's add two additional commands to our `package.json`. They'll allow us
to run `ESLint` and automatically fix formatting issues:


```
{
  ...
  "scripts": {
    ...
    "lint": "eslint './src/**/*.{tsx,ts}'",
    "lint-fix": "eslint './src/**/*.{tsx,ts}' --fix",
  }
}
```

<div id='jest'/>
# Jest and Code Coverage

[Jest](https://jestjs.io/) has emerged as the most popular JavaScript testing framework.
To make it work with TypeScript we'll need a helper module called [ts-jest](https://www.npmjs.com/package/ts-jest).
It dynamically compiles the TypeScript code. 
We also need `jest` to generate a test coverage report. We'll feed this report
into [SonarQube](https://www.sonarqube.org/) later on for further analysis. There's a handy module for this called [jest-sonar-reporter](https://www.npmjs.com/package/jest-sonar-reporter).

We can install them and `jest` itself via `npm` or `yarn`:

```bash
npm install --save-dev jest @types/jest
npm install --save-dev ts-jest jest-sonar-reporter
```

Jest can be configured via a file called `jest.config.js` in the project root folder. 
We'll use it to transform all test files matching the 
[Jest naming convention](https://www.reactnative.guide/7-testing/7.1-jest-setup.html) 
with `ts-jest` and generate reports 
via `jest-sonar-reporter`. 
[Here is](https://github.com/nikolayg/secure-typescript-boilerplate/blob/master/jest.config.js) the full file:

```js
module.exports = {
  coverageDirectory: './coverage',
  collectCoverageFrom: ['src/**/*.ts', 'src/**/*.tsx'],
  testEnvironment: 'node',
  modulePaths: ['<rootDir>/src', 'node_modules'],
  roots: ['src'],
  transform: {
    '^.+\\.tsx?$': 'ts-jest'
  },
  testRegex: '(/__tests__/.*|\\.(test|spec))\\.(ts|tsx)$',
  coverageReporters: ['json', 'lcov', 'text'],
  coveragePathIgnorePatterns: ['.*/src/.*\\.d\\.ts', '.*/src/testUtil/.*'],
  testResultsProcessor: 'jest-sonar-reporter'

  // Use the below to set coverage goals.
  // "npm test" will fail if these metrics are violated
  coverageThreshold: {
    global: {
      statements: 80,
      branches: 80,
      functions: 80,
      lines: 80
    }
  }
};
```

Now, we need to tell `jest-sonar-reporter` where to put the coverage output. 
Edit `package.json` and add the following at the end:

```json
{
  ...
  "jestSonar": {
    "reportPath": "coverage",
    "reportFile": "test-reporter.xml",
    "indent": 4
  }
}
```

As a last step, we need to ensure we generate the coverage reports during testing.
Add the following to your scripts in `package.json`:

```json
"test": "jest --forceExit --detectOpenHandles --coverage"
```

If you run `npm test` you should see a new folder `./coverage` with the code coverage reports.

<div id='jest-cra'/>
# Jest and Create React App (CRA)

Many React projects use the [Create React App](https://create-react-app.dev/) code
generator. CRA uses Jest internally but hides many of its properties.
Also, it doesn't pick up the configuration in `jest.config.js`.

Fortunately, you can still configure the coverage output directly in the 
`package.json` script:

```json
{
  ...
  "scripts": {
    "test": "CI=true react-scripts test --silent --env=jsdom --coverage --testResultsProcessor jest-sonar-reporter",
  }
}
```

You will still need to install `jest-sonar-reporter` and add the `"jestSonar"`
configuration as in the previous section.

<div id='dep-audit'/>
# Dependency Audit / Supply Chain Analysis

A large JavaScript project can have hundreds of direct dependencies.
Each of them can have many dependencies on its own.
A security vulnerability in any of them can become a vulnerability for 
the entire project. This is known as a [Supply Chain Attack](https://devops.com/software-supply-chain-attacks-how-to-disrupt-attackers/).

To address this, both `npm` and `yarn` introduced a new command called `audit`.
It checks whether your dependencies have known vulnerabilities and provides a report.
Unfortunately, working directly with `npm audit` can be tricky. It provides
limited options to filter threats by severity and to white list modules.
It can generate a lot of noise. 

There are a few auxiliary tools which "wrap" the audit command
and give developers much more control. A popular option is
[audit-ci](https://www.npmjs.com/package/audit-ci). 
It automatically detects whether you're using `npm` or `yarn` and gives you plenty 
of configuration options. Let's install it:

```bash
npm install --save-dev audit-ci
```

Now, let's create a file `audit-ci.json` with its config:

```json
{
  "high": true,
  "package-manager": "auto",
  "report-type": "summary",
  "path-whitelist": [
    "1217|sonarqube-scanner>download>decompress",
    "1217|sonarqube-verify>sonarqube-scanner>download>decompress"
  ]
}
```

In the above, we asked `audit-ci` to fail only if there're threats classified as
`high` or more critical. We also white listed a couple of threats from development
dependencies so they will not cause failure.

Lastly, let's configure a script for `audit-ci` in `package.json`:

```json
{
  ...
  "scripts": {
    "audit-dependencies": "audit-ci --config audit-ci.json",
  }
}
```

Now, you should be able to run `npm run audit-dependencies` and analyze your
supply chain for vulnerabilities.

<div id='sonar-qube'/>
# Local SonarQube

[SonarQube](https://www.sonarqube.org/) is a popular tool for static source 
code analysis. It supports many languages including TypeScript. 
Typically, a company would have a SonarQube instance which analyses
all of its projects. The CI/CD pipeline would push your code to the 
SonarQube instance during each build.

I often find it convenient to run SonarQube locally so I can quickly analyse
my code before integrating it. This is quite easy with a 
[Docker container](https://dev.to/rafaeldias97/nodejs-express-docker-jest-sonarqube-45me).

Let's create a `docker-compose.sonar.yml` file:

```yml
version: '3'
services:
  sonarqube:
    container_name: sonarqube
    image: sonarqube:latest
    ports:
      - '9000:9000'
      - '9092:9092'
```

Then we can start/stop SonarQube with:

```bash
# Start it ...
docker-compose -f docker-compose.sonar.yml up -d

# Stop it ...
docker-compose -f docker-compose.sonar.yml down
```

After starting it, wait for Sonar to load on 
[http://localhost:9000](http://localhost:9000) (it can take 1-2mins). 
Navigate to [http://localhost:9000/dashboard](http://localhost:9000/dashboard) 
and use `admin`/`admin` to login.

Note that the local SonarQube instance uses in-memory data storage.
Any changes you make will be wiped out on restart.

<div id='sonar-analysis'/>
# Run SonarQube Analysis

Before you analyse a project in SonarQube, you need to create a config file
called `sonar-project.properties` in the root folder:

```
sonar.projectKey=secure-typescript-boilerplate
sonar.projectName=secure-typescript-boilerplate
sonar.projectVersion=1.0
sonar.language=ts
sonar.sources=src
sonar.sourceEncoding=UTF-8
sonar.exclusions=src/**/*.test.ts
sonar.test.inclusions=src/**/*.test.ts
sonar.coverage.exclusions=src/**/*.test.ts,src/**/*.mock.ts,node_modules/*,coverage/lcov-report/*
sonar.typescript.lcov.reportPaths=coverage/lcov.info
sonar.testExecutionReportPaths=coverage/test-reporter.xml
```

This file defines the project name, key, and version. It also specifies the programming
language, code location, and the code coverage report.

To run the SonarQube analysis we will need an auxiliary module 
called [sonarqube-scanner](https://www.npmjs.com/package/sonarqube-scanner):

```bash
npm install --save-dev sonarqube-scanner
```

The module expects to find a file called `sonar-project.js` in the project root.
Let's create it:

```js
const sonarqubeScanner = require('sonarqube-scanner');

sonarqubeScanner(
  {
    serverUrl: process.env.SONAR_SERVER || 'http://localhost:9000',
    token: process.env.SONAR_TOKEN || '',
    options: {}
  },
  () => {
    console.log('>> Sonar analysis is done!');
  }
);
```

Now we can run `sonarqube-scanner` with `node sonar-qube.js` and this will submit 
our code sonar server. 
By default, the local server is used, but this can be overriden via environment
variables. Go on and test it to make sure it works.

The `sonarqube-scanner` module has one shortcoming. It starts the SonarQube analysis
asynchronously and it doesn't wait for it to complete. This is problematic 
when you want to integrate the code scan into a CI/CD pipeline. You may
need to wait for the analysis to complete and either fail/proceed based on the
result.

Thus, we'll use another module called
[sonarqube-verify](https://www.npmjs.com/package/sonarqube-verify):

```bash
npm install --save-dev sonarqube-verify
```

The `sonarqube-verify` library is a wrapper of `sonarqube-scanner`.
It starts the code analsys and then checks its progress every few seconds.
On completion, it either suceeds or fails based on the analysis result.

Let's create the script to run it:

```json
{
  ...
  "scripts": {
    "sonar": "sonarqube-verify"
  }
}
```

Now we can run the analysis:

```bash
# Generate the test coverage report
npm test

# Run the analysis (remember to start SonarQube)
npm run sonar

# Visit http://localhost:9000/projects to see result 
# login with admin/admin
```

Check out the documentation of [sonarqube-verify](https://www.npmjs.com/package/sonarqube-verify)
to see how to push to a remote server via env variables.


<div id='husky'/>
# Git Hooks with Husky

[Git hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) are a 
great way to ensure code quality before your code hits the CI/CD pipeline.
They can significantly speed up the developer feedback loop and can
enforce discipline in a team. [Husky](https://www.npmjs.com/package/husky)
is a popular Node JS library for custom hooks. Let's install it:

```bash
npm install --save-dev husky
```

Now we need to create a `.huskyrc` file which defines the hook:

```json
"hooks": {
  "pre-push": "npm build && npm test && npm run lint && npm run audit-dependencies"
}
```

This ensures that the code builds, the tests pass, no ESLint errors are present, 
and there're no known vulnerabilities in the dependencies. 
Depending on the project size these commands can take a while.
I prefer to use `pre-push` instead of `pre-commit` so that local commits 
are quick, but everyone is forced to clean up their code before they integrate.
Every team has its own philosophy - use whatever works for you!

From now on, on every `git push` the above command will run. If it fails, 
no code will be pushed to the remote repository.
