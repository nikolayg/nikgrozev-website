---
layout: post
title: Fast and Simple Sampling in Pandas when Loading Data From Files
date: 2015-06-16 03:33:06.000000000 +10:00
type: post
published: true
status: publish
excerpt: 
    In this article I'll describe a simple and fast approach for sampling data in 
    Pandas as it is loaded from the data file ...
categories:
- DataScience
- Miscellaneous
- Python
- blog
tags:
- Pandas
- Python
- DataScience
author:
  login: nikolaygrozev
  email: nikolay.grozev@gmail.com
  display_name: nikolaygrozev
  first_name: 'Nikolay'
  last_name: 'Grozev'
---

# Introduction

Recently I've been exploring how Python can help me quickly analyse and explore data. 
Using a general purpose programming language like Python has a number of benefits compared 
to specialised languages like R when munging heterogeneous and messy data.

[Pandas](http://pandas.pydata.org/) is one of the most widely used python libraries for data analysis. 
It basically introduces a layer between other libraries like numpy and matplotlib, 
which makes it easier to read in, transform and plot data. Also, it introduces the concepts of DataFrames 
and Series, which are familiar to R programmers.

### The Problems

Quite often people have to work with pretty big data sets, which introduces several problems. 
Firstly, you may not have sufficient memory on your development machine. In this case, you may like to 
load a subset of the data and use it for exploratory analysis and visualisation. 
After having an idea of what you're looking for, you can run your thorough analyses on a dedicated 
server (e.g. a VM in the cloud). Secondly, loading all the data in memory can make your analyses run rather 
slowly, as all operations are performed on big DataFrames. This can be disruptive for your work process. 
Again, working with a subset of the data may be sufficient for preliminary exploratory work.

Pandas comes with a few features for handling big data sets. It allows you to 
[read big data files in chunks](http://pandas-docs/dev/io.html#iterating-through-files-chunk-by-chunk) 
or you can just load the first N lines. Neither of these approaches solves the aforementioned problems, 
as they don't give us a small randomised sample of the data straight away. A better approach would be to 
load all the data (in chunks or in whole), and then perform sampling. This is not always viable as the data 
could exceed the resources you have and the sampling & chunking code may not be trivial.

In this article I'll describe a simple and fast approach for sampling data as it is loaded from the data file.

# Solution: skiprows

In the later versions of Pandas its developers have introduced a new parameter `skiprows` 
of the [read_csv](http://pandas.pydata.org/pandas-docs/stable/io.html#io-read-csv-table) and function. 
It allows you to specify a list of line/row indices, which will not be loaded by pandas. 
In essence, what we can to do is generate the list of line ids which pandas will ignore.

## Approach 1 - select every N-th line

Often it is reasonable to select every N-th line in the file and ignore the rest. 
This makes sense if the data is already ordered by an important parameter (e.g. time of event occurrence) 
and thus such sampling will result in a representative sample. Generating the list of `skiprows` is 
trivial once we know the number of lines in the file. We can obtain the latter with a simple scan of the 
file we can just use an upper bound of the number of lines we expect in the file. The following code illustrates the approach:

```python
import pandas as pd

# The data to load
f = "my_data.csv"

# Take every N-th (in this case 10th) row
n = 10

# Count the lines or use an upper bound
num_lines = sum(1 for l in open(f))

# The row indices to skip - make sure 0 is not included to keep the header!
skip_idx = [x for x in range(1, num_lines) if x % n != 0]

# Read the data
data = pd.read_csv(f, skiprows=skip_idx, ... )
```

**<u>Note!</u>** If your data has a header you should make sure that 0 is not included in your skiprows list. 
Otherwise, the header will be ignored. **<u>Note!</u>** You may be tempted to use a generator instead of a 
list for the skiprows parameter value, in order to save some memory. Although this would work, my experiments 
show that the performance of `read_csv` degrades when `skiprows` uses generators.

## Approach 2 - random selection

In other cases you may want to select the lines randomly. Changing the previous example for this case is easy, as demonstrated below:

```python
import pandas as pd
import random

# The data to load
f = "my_data.csv"

# Count the lines
num_lines = sum(1 for l in open(f))

# Sample size - in this case ~10%
size = int(num_lines / 10)

# The row indices to skip - make sure 0 is not included to keep the header!
skip_idx = random.sample(range(1, num_lines), num_lines - size)

# Read the data
data = pd.read_csv(f, skiprows=skip_idx, ... )
```

This approach tends to be a bit slower, as the generation of the random sample of indices can take some time for files with many lines.
