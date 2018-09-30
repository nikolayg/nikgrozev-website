---
layout: post
title: Simple Python API
date: 2018-10-10 05:22:09.000000000
type: post
published: false
status: publish
excerpt: 
    Simple Python API.  
categories:
- Python
- API
tags:
- Python
- API
- pyenv
---

# Introduction

...

# Pyenv

```bash
# Create a folder for the project
mkdir sample-python-api && cd sample-python-api

# Define project specific Python versions
echo "3.6.0" > .python-version

# Should print the version from ".python-version".
pyenv local

# Installs the version from ".python-version". Can take some time.
pyenv install
```

# Pipenv

After installing `pyenv`, you cann add `pipenv` with the following command: 

```bash
pip install --user pipenv
```

If that doesn't work for you check the [Pipenv's installation documentation](https://pipenv.readthedocs.io/en/latest/install/#installing-pipenv)
for alternative installation instructions for your platform.

Now we can install a dependency/library:

```bash
pipenv install flask
```

This will create two files `Pipfile` and `Pipfile.lock` in the project's directory.

After initialising `pipenv`, you need to activate it via:

```bash
# Initialises common environment variables
pipenv shell
```

This will initialise common environment variables in the current shell.
As a result, when you run `python` the shell will start the REPL interpreter
with all `pipenv` libraries loaded at your disposal:

```bash
# Initialises common environment variables
pipenv shell

# Start the shell
python

# We have all dependencies loaded, so we can import them
from flask import Flask
```


# Flask

# Swagger

# Unit Tests

# VS Code

# Cloud Foundry