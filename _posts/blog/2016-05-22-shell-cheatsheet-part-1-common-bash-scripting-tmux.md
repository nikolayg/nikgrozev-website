---
layout: post
title: Shell Cheatsheet [Part 1] - Common Bash Scripting, Tmux
date: 2016-05-22 08:51:23.000000000
type: post
published: true
status: publish
excerpt: 
    This post is a cheat sheet of commonly used shell commands and tools. 
    It is meant to serve as a quick reference guide that you can consult when developing scripts or when working 
    extensively with the Bash shell ...
categories:
- Bash
- blog
tags:
- Bash
- Cheatsheet
- Shell
- Tmux
author:
  login: nikolay.grozev@gmail.com
  email: nikolay.grozev@gmail.com
  display_name: nikolay.grozev@gmail.com
  first_name: 'Nikolay'
  last_name: 'Grozev'
---

# Table of Contents

- [Introduction](#introduction)
- [Common Bash Commands](#common-bash-commands)
    - [Finding Files (find, egrep)](#finding-files-find-egrep)
    - [Interacting with Servers (curl)](#interacting-with-servers-curl)
    - [Inspect Ports and File Handles (lsof)](#inspect-ports-and-file-handles-lsof)
    - [Pattern Matching and Replacing (sed)](#pattern-matching-and-replacing-sed)
    - [Working with Tabular Data (awk)](#working-with-tabular-data-awk)
    - [Working with JSON (jq)](#working-with-json-jq)
    - [Working with Archives (tar, zip, unzip)](#working-with-archives-tar-zip-unzip)
    - [File Permissions (ls, chmod, chown)](#file-permissions-ls-chmod-chown)
    - [SSH Connection (ssh, scp)](#ssh-connection-ssh-scp)
    - [Miscellaneous Commands](#miscellaneous-commands)
- [The Bash Language](#the-bash-language)
    - [Data Structures (array, hash)](#data-structures-array-hash)
    - [Control Follow](#control-follow)
    - [Functions](#functions)
    - [Invoking a Script](#invoking-a-script)
    - [Writing a Script](#writing-a-script)
- [Managing Services in systemd](#managing-services-in-systemd)
    - [Service Lifecycle (systemctl, journalctl)](#service-lifecycle-systemctl-journalctl)
    - [Creating a new Service (script in /etc/init.d)](#creating-a-new-service-script-in-etc-init-d)
- [Tmux Commands](#tmux-commands)
    - [Manage Sessions](#manage-sessions)
    - [Windows/Tabs](#windowstabs)
    - [Panes/Splits](#panessplits)

<div id='introduction' />
# Introduction

This post is a cheat sheet of commonly used shell commands and tools. 
It is meant to serve as a quick reference guide that you can consult when developing scripts or when working 
extensively with the Bash shell. It refers to tools which are usually present in all Linux distributions and OS X, 
or which can be easily installed with the usual package managers. The only exception is **systemd** which is 
present on most modern Linux distributions (with a few exceptions).

<div id='common-bash-commands' />
# Common Bash Commands

<div id='finding-files-find-egrep' />
## Finding Files (find, egrep)

```bash
# Find file recursively
find . -name test.txt

# -iname makes it case insensitive
find . -iname Test.txt

# All files and direcotries that match the pattern
find . -name "*.css"

# The same, but only files/directories this time
find . -type f -name "*.css"
find . -type d -name "*.css"

# Using posix/standard regex in the path (not just name)
find . -regex ".*[A-Z]+.*.md"

# Find files by size - use k,M and G for kilo/mega/gigabyte
find . -size 50M
find . -size +50M -size -100M

# Avoid recursiveness by limiting the depth - i.e. like ls
find . -maxdepth 1 -name "*.css"

# Use (a/c/m)time to search by last access/change/modify time (measured in days)
# Files modified 30-60 days ago:  
find . -mtime +30 –mtime -60

# Use (a/c/m)min to serach by minutes
# Files accessed during the last 45 mins:
find . -amin -45

# Finding files by content - prints all matches
find . -type f -exec egrep "^.*pattern" /dev/null {} \;

# The same, but without the matches - i.e. just file names
find . -type f -exec egrep -l "^.*pattern" /dev/null {} \;

# The same as the above two with the grep command
egrep -r '^.*pattern'
egrep -lr '^.*pattern'
```

<div id='interacting-with-servers-curl' />
## Interacting with Servers (curl)
```bash
# Downloads the content. Tries to guess the protocol - e.g. HTTP, FTP
curl example.com > example.txt
curl ftp://myftpsite.com --user myname:mypassword  > example.txt

# Curl plays well with the tidy command, which pretty-prints HTML:
curl example.com | tidy -qi

# Use verbose mode to see headers
curl -v example.com

# Add headers to your request:
curl -H "Content-Type: application/json" -H "authToken: secret" example.com

# Web sites like google.com often respond with redirections (302/301)
# To follow the redirection use -L
curl -L google.com

# Resuming a canceled/failed download
curl -C - example.com

# HTTP POST, PUT, DELETE
curl -X POST example.com -d "Request Body"

# To use HTTP Basic Authentication
curl -u 'username:password' example.com/auth

# To access insecure HTTPS connections
curl -k https://example.com/secure.php
```

<div id='inspect-ports-and-file-handles-lsof' />
## Inspect Ports and File Handles (lsof)

```bash
# List all current network connections
lsof -i

# List processes listening on a given port
lsof -i :25

# Inspect what files/network resources a process or command is using
lsof -p 10075
lsof -c ls
```

<div id='pattern-matching-and-replacing-sed' />
## Pattern Matching and Replacing (sed)

```bash
# Removing lines (1st and 3rd) from a file
sed -e '1d' -e '3d' 'test.txt' > 'result.txt'

# Replace all occurrences of a word with another
sed -e 's/cat/dog/g' 'input.txt'

# The same but case insensitive
sed -e 's/cat/dog/ig' 'input.txt'

# Replace by regex
sed -r 's/(\s+|,|\.)/-/g' 'test.txt'

# Replace by regex using groups
sed -r 's/(\w+)/ (\1) /g' 'test.txt'
```

<div id='working-with-tabular-data-awk' />
## Working with Tabular Data (awk)

```bash
# Getting columns from tabular data
awk '{ print $2, $4 }' text.txt
ls -l | awk '{ print $2, $4 }'

# Using custom separator
echo "1,2,3,4,5" | awk -F, '{ print $2, $4 }'

# Arithmetic and formatting
echo "1 2 3 4 5" | awk '{ printf "%.2f - %.2f - %.2f\n", $2, $2 + $3, $4 }'

# Skip initial rows (e.g. a header)
ls -l | awk 'FNR > 1 { print $1 }'

# Select a specific row
ls -l | awk 'FNR == 2 { print $1 }'
```

<div id='working-with-json-jq' />
## Working with JSON (jq)

```bash
# Sample JSON data
read -r -d '' jsonTxt << EOM
{"employees":[
    {"firstName":"John", "lastName":"Doe"},
    {"firstName":"Anna", "lastName":"Smith"},
    {"firstName":"Peter", "lastName":"Jones"}
]}
EOM

# Pretty print some JSON
echo $jsonTxt | jq .
jq . jsont.txt

# Get an object's property
echo $jsonTxt | jq .employees

# Get an array element
echo $jsonTxt | jq .employees[1]

# Unpack an array of objects
echo $jsonTxt | jq .employees[]

# Get a property from all objects
echo $jsonTxt | jq .employees[].firstName

# Filter an array by value
echo $jsonTxt | jq '.employees[] | select(.firstName == "John")'

# Filter an array by regex
echo $jsonTxt | jq '.employees[] | select(.firstName | match(["jo.+", "gi"]))'
```

<div id='working-with-archives-tar-zip-unzip' />
## Working with Archives (tar, zip, unzip)
```bash
# Archive a folder
tar cvf archive_name.tar dirname/

# Archive and compress a folder
tar cvzf archive_name.tar.gz dirname/

# Extract a tar
tar xvf archive_name.tar
tar xvf archive_name.tar -C ./destination
tar xvzf archive_name.tar.gz

# Zip a single file
zip file.zip filepath

# Make a zip from a folder
zip -r archive.zip dirname

# Zip a file/folder without storing the path to it
zip -j file.zip filepath

# Unzip/Extract
unzip file.zip
unzip file.zip ./destination
```

<div id='file-permissions-ls-chmod-chown' />
## File Permissions (ls, chmod, chown)
```bash
# Check permissions - prints r/w/x flags for owner-user/group/others
# Also prints the owner user and groups
ls -l path

# Change file owner
chown owner-user file
chown owner-user:owner-group file

# Change owner recursively
chown owner-user -R folder-path

# Giver permissions (i.e. r/w/x) to user/group/others/all (u/g/o/a)
chmod g+w file
chmod a+r file
chmod ug+wx file

# Remove permissions (i.e. r/w/x) to user/group/others/all (u/g/o/a)
chmod g-w file
chmod o-x file

# Chmod recursively
chmod -R g+w directory
```

<div id='ssh-connection-ssh-scp' />
## SSH Connection (ssh, scp)
```bash
# Connect as a user
ssh user@server-address

# Connect with a pem file
ssh -i myfile.pem server-address

# Set appropriate pem file permissions
chmod 400 mykey.pem

# Upload a file over SCP
scp ~/local_file username@server.org:~/destination

# Download over SCP
scp username@server.org:remote_file local_file
```

<div id='miscellaneous-commands' />
## Miscellaneous Commands
```bash
# Move back to the previously visited folder
cd -

# Exit status of the last command
echo $?

# The pid of the last async command
echo $!

# The pid of the current process
echo $$

# Human readable folder details
ls -lah

# Similar, but also specifies the levels to be displayed (-L)
tree -h -L 1

# Inspect the file system - mounted volumes etc.
df -h

```

<div id='the-bash-language' />
# The Bash Language

<div id='data-structures-array-hash' />
## Data Structures (array, hash)

```bash
# Define an array
arr=(1 "two" 3 'four' 5 'six')

# Get an element by index
echo ${arr[0]}

# Set an element by index
arr[1]='TWO'

# Get the length
echo ${#arr[@]}

# Array slicing in the form [start_after_index]:[slice_length]
echo ${arr[@]:1:2}

# Append
arr+=(7 'eight')

# Define a hash/dictionary/associative array:
declare -A h=( ["one"]=1 ["two"]=2 )

# Get from the hash
echo ${h["one"]}

# Set and delete from hash
h["one"]="Uno"
unset h["one"]
```

<div id='control-follow' />
## Control Follow

```bash
# If-Then-Else - watch your spaces!
if [[ 3 > $((1 + 1)) || 'a' == 'b' ]]
then
  echo "Correct"
else
  echo "Incorrect"
fi

# Create file if it does not exists
if [[ ! -e "test.txt" ]] ; then
    touch "test.txt"
fi

# If variable is undefined/empty
if [[ ! $x ]]
then
  echo "x is empty"
fi

# For loop with sequence/range of numbers
for i in $(seq 1 10);
do
  if [[ $i < 5 ]]
  then
    echo $i
  else
    break
  fi
done

# Loop over all files in a folder with a pattern
for f in $(ls | egrep ".*\.md")
do
  echo $f
done

# Loop over an array - i.e. for-each
for x in ${arr[@]}
do
  echo $x
done

# Iterate/loop over a dictionary/hash
for k in "${!h[@]}"
do
  echo "$k = ${h[$k]}"
done
```

<div id='functions' />
## Functions

```bash
# Example functions with params
function prettyPrint {
  #Parameters are indexed from 1
  local x=$1
  local y=$2
  printf "%.2f$ Revenue and %.2f$ Loss \n" $x $y
}

function customEcho {
  # Number of params
  echo "$# arguements passed"

  # print all params
  for p in $*
  do
    echo "$p"
  done
}

# Invoke functions
prettyPrint 12.5 13.4
customEcho 1 2 3
```

<div id='invoking-a-script' />
# Bash Scripts

<div id='writing-a-script' />
## Invoking a Script
```bash
# Run a script in a new child process
bash script.sh
./script.sh       # If executable

# Source a script - runs commands and definitions in present process
source script.sh
. script.sh

# Run script with two predefined environment vars
var1=1 var2=2 bash script.sh

# Run a script and print each command - use for debugging
bash -x script.sh

# Run in verbose mode - prints everything including comments
bash -v script.sh

# Fails immediately if an undefined variable is used in the script
bash -u script.sh

# Fails immediately if a command returns an error code
bash -e script.sh
```

<div id='writing-a-script' />
## Writing a Script

```bash
#!/usr/bin/env bash

# Display commands and their args before running them
set -x

# Verbose mode
set -v

# Fails if an undefined variable is used
set -u

# Fails if a command returns an error code
set -e

# Position params
echo "$1 $2"

# Array of position params
echo $*
echo $@

# Number of params
echo $#
```

<div id='managing-services-in-systemd' />
# Managing Services in systemd

<div id='service-lifecycle-systemctl-journalctl' />
## Service Lifecycle (systemctl, journalctl)
```bash
# List all services
systemctl list-unit-files --type=service

# Start/stop/restart/reload/enable a Services
systemctl start some.service
systemctl stop some.service
systemctl restart some.service
systemctl reload some.service
systemctl enable some.service
systemctl disable some.service

# Check the config of a service
systemctl show some.service

# Check the log/journal of systemd events
journalctl

# Events for a service
journalctl --unit=some.service

# Events since last boot or from today
journalctl -b
journalctl --since=today

# View error events
journalctl -p err
```

<div id='creating-a-new-service-script-in-etc-init-d' />
## Creating a new Service (script in /etc/init.d)

To create a service, place its script in - **"/etc/init.d"** (on some distros in **"/etc/rc.d/init.d/"**). 
The code of the script should handle start, stop, restart, etc. The following example 
(taken from [here](http://unix.stackexchange.com/questions/20357/how-can-i-make-a-script-in-etc-init-d-start-at-boot)) 
demonstrates a skeleton of such a script:

```bash
#!/usr/bin/env bash
# === Sample Script ===

# Source function library.
. /etc/init.d/functions

start() {
    # code to start app comes here
    # example: daemon program_name &
}

stop() {
    # code to stop app comes here
    # example: killproc program_name
}

case "$1" in
    start)
       start
       ;;
    stop)
       stop
       ;;
    restart)
       stop
       start
       ;;
    status)
       # code to check status of app comes here
       # example: status program_name
       ;;
    *)
       echo "Usage: $0 {start|stop|status|restart}"
esac

exit 0
```

<div id='tmux-commands' />
# Tmux Commands

<div id='manage-sessions' />
## Manage Sessions
```bash
# Start a new session
tmux new -s session_name

# Attach to a running session
tmux attach -t session_name

# List all running sessions
tmux ls

# Connect/switch to another session
tmux switch -t session_name

# Disconnect from session (also "ctrl+b+d")
tmux detach

# Kill/terminate a sessions
tmux kill-session -t session_name
```

<div id='windowstabs' />
## Windows/Tabs

```
# Create window/tab (shortcut)
ctrl+b+c

# Rename a window/tab (shortcut)
ctrl+b+,

# Switch window/tab (shortcut)
ctrl+b+w

# Close window/tab (shortcut)
ctrl+b+&
```

<div id='panessplits' />
## Panes/Splits

```
# Split window vertically (shortcut)
ctrl+b+%

# Split window horizontally (shortcut)
ctrl+b+"

# Close current pane (shortcut)
ctrl+b+x

# Navigate to next pane (shortcut)
ctrl+b+o

# Navigate to next pane with arrow key (shortcut)
ctrl+b+[arrow-key]
```
