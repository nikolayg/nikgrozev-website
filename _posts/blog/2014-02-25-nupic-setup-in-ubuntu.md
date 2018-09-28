---
layout: post
title: NuPIC Setup in Ubuntu
date: 2014-02-25 23:34:01.000000000
type: post
published: true
status: publish
excerpt: 
    NuPIC (Numenta Platform for Intelligent Computing) is a biologically 
    inspired library for machine learning in the loose sense of the term. It is an implementation of 
    the HTM (Hierarchical Temporal Memory) model which mimics the structure and organisation of the neocortex. 
    In this post I'll overview a setup procedure for Ubuntu (tested with version 14.04) ...
categories:
- NuPIC
tags:
- C++
- NuPIC
- Python
- Data Science
---


# Introduction

[NuPIC](https://github.com/numenta) (Numenta Platform for Intelligent Computing) is a biologically 
inspired library for machine learning in the loose sense of the term. It is an implementation of 
the [HTM](http://en.wikipedia.org/wiki/Hierarchical_temporal_memory) (Hierarchical Temporal Memory) 
model which mimics the structure and organisation of the [neocortex](http://en.wikipedia.org/wiki/Neocortex) 
following the ideas from [Jeff Hawkin](http://en.wikipedia.org/wiki/Jeff_Hawkins)'s book 
"[On Intelligence](http://en.wikipedia.org/wiki/On_Intelligence)".

NuPIC is not the only open source implementation of HTM, but it is the "cutting edge" in this area. 
Unfortunately it is quite difficult to set up and test. In this post I'll overview a setup procedure for 
Ubuntu (tested with version 14.04).


<div class="mid-page-ads in-body-ads ad-secion">
    <div class="ad-header ad-header-body">Related Topics</div>
    <script id="mNCC" language="javascript">
        if (window.innerWidth >= 1024) {
          medianet_width = "600";
          medianet_height = "250";
          medianet_crid = "459711728";
        } else {
          medianet_width=Math.min(250, window.innerWidth).toString();
          medianet_height = "250";
          medianet_crid = "318234500";
        }
        medianet_versionId = "3111299"; 
      </script>
    <script src="//contextual.media.net/nmedianet.js?cid=8CU4WBM36"></script>
</div>


# Setup Procedure

> **<u style="color:red;">UPDATE 2</u>**: The Nupic team has changed the set-up procedure again, 
so the script below is not longer valid with the latest code. Please follow the 
[latest instructions](https://github.com/numenta/nupic#build-and-test-nupic) on GitHub instead. 
<u>It is however still important to ensure you have sufficient RAM and CPU resources on your machine, as described below</u>.

> **<u style="color:red;">UPDATE 1</u>**: I've updated the installation instructions below, as 
the NuPIC team has made significant changes during the last 2-3 months. The following instructions 
are tested on 64 bit Ubuntu 14.04 with the NuPIC trunk code from <u>11-Jun-2014</u>.

Before starting, you should have **python 2.7** installed which comes by default in most Ubuntu installations. 
It is important to ensure that you have **sufficient resource on your PC!** If you don't you may be 
getting strange and completely unrelated errors during the build process - e.g. compile or configuration errors. 
This may cause you to waste hours or even days in futile debugging. For example, running the installation procedure 
completed successfully on a VirtuaBox Virtual Machine with 4 dedicated CPU cores and 2GB RAM. Running the same procedure 
on a clone of the same VM with 1GB and 1 CPU core resulted in runtime errors of the unit tests about missing dependencies. 
Unfortunately, I couldn't find any documentation or guidelines about how much CPU and RAM capacity is typically needed.

After setting up python 2.7 you'll be ready to go with the installation script given below. 
You can also run the commands one by one to check for error messages.

The script firstly installs all needed packages. Then it clones the [NuPIC](https://github.com/numenta/nupic)'s 
code and installs all required python dependencies. **<u>NOTE</u>**: The script installs the python dependencies as 
global python-pip packages. If you are using python-pip for purposes other than running NuPIC you may prefer to install 
them in [virtualenv](http://www.virtualenv.org/en/latest/) instead. Eventually, the script builds NuPIC's code and runs 
all tests. The command `make -j3` (line 25) builds using 3 CPU cores. Depending on your system you may like to increase 
or decrease this - e.g. `make -j4` if you can use 4 cores. After the installation and tests are over you should be able 
to verify that all tests have completed successfully.

```bash
#!/bin/bash

# Install GIT, python etc.
sudo apt-get -y install git python2.7 gcc automake libtool python-pip python-dev libssl-dev cmake

# Clone the NUPIC code
git clone https://github.com/numenta/nupic.git

sudo pip install --allow-all-external --allow-unverified PIL --allow-unverified psutil -r external/common/requirements.txt

# Env. variable needed by all build scripts
pyVersion=`python -c 'import sys; print(".".join(map(str, sys.version_info[:2])))'`
export NUPIC=`pwd`/nupic
export NTA=$NUPIC/build/release
export NTA_ROOTDIR=$NTA
export PYTHONPATH=$PYTHONPATH:$NTA/lib/python$pyVersion/site-packages

# Configure and generate build files
mkdir -p $NUPIC/build/scripts
cd $NUPIC/build/scripts
cmake $NUPIC

# Build...
cd $NUPIC/build/scripts
make -j3

# Go to scripts folder to run tests
cd $NUPIC/build/scripts

# All C++ tests
make tests_everything

# C++ HTM Network API tests
make tests_cpphtm

# Python HTM Network API tests
make tests_pyhtm

# Python OPF unit and integration tests (requires mysql) - Skip
# make tests_run_all

# Run all tests!
make tests_all
```

The above environment variables `NUPIC`, `NTA`, `NTA_ROOTDIR`, and `PYTHONPATH` need to be set before running NuPIC. 
You may wish to save them in `/etc/profile` or `/etc/bash.bashrc` so that they are always loaded. You may have 
to reboot afterwards so that the changes can take effect.

```bash

# Variable definitions at the end of /etc/default ...

pyVersion=`python -c 'import sys; print(".".join(map(str, sys.version_info[:2])))'`
export NUPIC=[path-to-nupic-directory]/nupic
export NTA=$NUPIC/build/release
export NTA_ROOTDIR=$NTA
export PYTHONPATH=$PYTHONPATH:$NTA/lib/python$pyVersion/site-packages
```


After completing the installation, you can run some examples.

```bash
# Run the "Online" hotgym  example
python $NUPIC/examples/opf/clients/hotgym/simple/hotgym.py
```

# Resources

*   [https://github.com/numenta/nupic/blob/master/README.md#installation](https://github.com/numenta/nupic/blob/master/README.md#installation)
*   [https://github.com/numenta/nupic/wiki/Installing-NuPIC-on-Ubuntu](https://github.com/numenta/nupic/wiki/Installing-NuPIC-on-Ubuntu)
