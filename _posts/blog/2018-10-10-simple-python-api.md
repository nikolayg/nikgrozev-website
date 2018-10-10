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

Recently, I have been considering picking up Python as language to develop APIs.
It has been a few years since I started working mostly with Node JS/TypeScipt.
The Node ecosystem has evolved quite well and it's nice to use the same language
on the front end and the back end. However, when it comes to machine learning and
statistics, the Python ecosystem is much more mature. On such projects, I would
consider writing parts of the backend in Python, so I can use libraries like
[NumPy](http://www.numpy.org/), [Pandas](https://pandas.pydata.org/), and [SciPy](https://www.scipy.org/). 


This article will describe how to get started with the python ecosystem so 
we can write APIs. We'll leave out application specific aspects like authentication
and database access, and will focus on the basics. As an example, we'll build a 
simple REST-ful API. You can find the full code [here](https://github.com/nikolayg/sample-python-api).


# Pyenv

Almost all modern programming languages have tools that allow us to quickly install
and switch between versions. Just like Node's [NVM](https://github.com/creationix/nvm) and  Java's [SDKMan](https://sdkman.io/), Python as [Pyenv](https://github.com/pyenv/pyenv).

If you don't already have it, you can follow the 
[Official installation instructions](https://github.com/pyenv/pyenv#installation).
On OS X, you can just `brew` it

```bash
brew install pyenv
```

Once you have pyenv, you can install and configure the desired python version as follows:

```bash
# Install the desired python version - e.g. 3.4.3
pyenv install 3.4.3
# Set it up as a global version - pyenv will reconfigure your PATH accordingly
pyenv global 3.4.3
```

# Pyenv Project Configuration

After installing Pyenv, let's create a folder for our sample API:

```bash
# Create a folder for the project
mkdir sample-python-api && cd sample-python-api
```

Whenver you enter a folder, Pyenv looks for a special file called `.python-version`.
It specifies the Python version for the project. If `.python-version` exists,
Pyenv will automatically switch to the respective version.

Let's create the file:

```bash
# Define project specific Python versions
echo "3.6.0" > .python-version
```

Now we can check that Pyenv detects the file and we can install the 
respective Python version.

```bash
# Should print the version from ".python-version".
pyenv local

# Installs the version from ".python-version" if not installed 
# Can take some time.
pyenv install
```

# Pipenv

Python's most popular package manager is called [PIP](https://github.com/pypa/pip).
[Pipenv](https://pipenv.readthedocs.io/en/latest/) is a new package and 
environment manager which uses `PIP` under the hood. It provides more
advanced features like version locking and dependency isolation between projects.

After installing `pyenv`, you should be able to add `pipenv` with the following command: 

```bash
pip install --user pipenv
```

If that doesn't work for you check the [Pipenv's installation documentation](https://pipenv.readthedocs.io/en/latest/install/#installing-pipenv)
for alternative installation instructions for your platform.

Now we can install a dependency/library for our sample API:

```bash
pipenv install flask
```

This will create two files `Pipfile` and `Pipfile.lock` in the project's directory.
The first one defines the dependencies and the second specifies the exact installed versions.
Whenever we add more dependncies, they'll be added to these two files.

`Pipenv` has the concept of a shell/environment which encapsulates the python process.
We can start a `pipenv` shell via:

```bash
# Initialises common environment variables
pipenv shell
```

This will initialise common environment variables in the current shell.
As a result, when you run `python` the shell will start the REPL interpreter
with all `pipenv` libraries loaded at your disposal:

```bash
# Initialises common environment variables and dependencies
pipenv shell

# Start the shell
python

# We have all dependencies loaded, so we can import them
> from flask import Flask
```

# PYTHONPATH

`PYTHONPATH` is a special environment variable, which tells the Python 
interpreter where to search for modules. Let's create two
source folders `src` and `tests`. By now your project should look like this:

```
sample-python-api/
 |
 ├──── Pipfile
 ├──── Pipfile.lock
 ├──── src/
 └──── tests/
```

Now we'll need to tell Pipenv the rigth `PYTHONPATH` so the Python interpreter
can include the files `src` and `tests`. 

Whenver Pipenv starts a shell, it looks
for a special file `.env` and loads all environment variable definitions from there.
Thus let's create a `.env` file with the following:

```
PYTHONPATH="src:test"
```

Now restart the Pipenv shell, for the change to take effect.

# Environments and Externalisation

In Node JS, developers use the special environment variable `NODE_ENV`
to specify whether the code is running in *production* or *development* mode.
Depending on the "mode", different configuration can be used.

To follow this pattern in Python we can set the `PYTHON_ENV` environment variable.
So for example:

```bash
# Will run in production mode.
PYTHON_ENV=production python <my_script.py>
```

Now lets create a file `src/environment/instance.py`, which will exemplify how to
load different configurations depending on the environment:

```python
import os

# Load the development "mode". Use "developmen" if not specified
env = os.environ.get("PYTHON_ENV", "development")

# Configuration for each environment
# Alternatively use "python-dotenv"
all_environments = {
    "development": { "port": 5000, "debug": True, "swagger-url": "/api/swagger" },
    "production": { "port": 8080, "debug": False, "swagger-url": None  }
}

# The config for the current environment
environment_config = all_environments[env]
```

Now our other modules can import `environment_config` to access environment specific
configuration. Note that the values in the above configuration are specific to 
`Flask`, which we will discuss next.

# Flask and Flask-RESTPlus

[Flask](http://flask.pocoo.org/) lightweight web server and framework.
We can spin a Web API directly with `flask`. However, the 
[Flask-RESTPlus](https://flask-restplus.readthedocs.io/en/stable/) 
extension makes it much easier to get started. It also automatically configures
a [Swagger UI](https://swagger.io/tools/swagger-ui/) endpoing for the API. 

If you have been following so far, we already installed `flask` by:

```bash
pipenv install flask
```

Now let's add `flask-restplus` too:

```bash
pipenv install flask-restplus
```

# Models and Resources

# Swagger

# Unit Tests

```bash
echo PYTHONPATH="$PWD/src" python -m pytest
```

# VS Code

# Cloud Foundry

- https://www.youtube.com/watch?v=yh-28ksEXwY