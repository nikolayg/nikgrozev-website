---
layout: post
title: Shell Cheatsheet [Part 2] – Working with Docker
date: 2016-06-18 15:22:09.000000000
type: post
published: true
status: publish
excerpt: 
    This post is a cheat sheet of commonly used 
    Docker shell commands ...
categories:
- Bash
- Docker
tags:
- Bash
- Docker
- Cheatsheet
- Shell
---

# Table of Contents

- [Introduction](#introduction)
- [Docker 101 - Managing a Single Container](#docker-101-managing-a-single-container)
  - [Inventory of Containers and Images](#inventory-of-containers-and-images)
  - [Start Containers, Run Commands and Shell (SSH-style)](#start-containers-run-commands-and-shell-ssh-style)
  - [Networking and Persistent Volumes](#networking-and-persistent-volumes)
  - [Restart, Stop, and Delete Containers](#restart-stop-and-delete-containers)
  - [Diff a Container and Create an Image](#diff-a-container-and-create-an-image)
- [Dockerfile - Create a New Image](#dockerfile-create-a-new-image)
  - [Sample Dockerfile](sample-dockerfile)
  - [Working with Dockerfile](#working-with-dockerfile)
- [Docker Compose](#docker-compose)
  - [Sample docker-compose.yml File](#sample-docker-compose-yml-file)
  - [Run Docker Compose](#run-docker-compose)

<div id='introduction'/>
# Introduction

Docker used to run "out of the box" only on Linux-based systems, but now it is available for 
[Windows and OS X](https://blog.docker.com/2016/03/docker-for-mac-windows-beta/) as well, without 
the need for third party tools. This makes it worthwhile to create a quick reference for often used 
docker commands and tools. This post is a cheat sheet of commonly used 
[Docker](https://www.docker.com/) shell commands. It is meant to serve as a quick reference that you 
can consult when developing scripts or debugging. It is not a tutorial - if you haven't used docker 
before please refer to the [official learning materials and documentation](https://docs.docker.com/). 


<div id='docker-101-managing-a-single-container'/>
# Docker 101 - Managing a Single Container

<div id='inventory-of-containers-and-images'/>
## Inventory of Containers and Images
```bash
# Check Docker hub for images with a given name
docker search ubuntu

# Pull a container from Docker hub. To use an image tag - use [image-name]:[tag]
docker pull hello-world
docker pull ubuntu
docker pull ubuntu:12.10

# List all "pulled" images on your local system
docker images

# List all running containers
docker ps

# List all containers (not running as well)
docker ps -a

# List what is running in a container. Id/Name is taken from ps.
docker top 32c5a521c664
```

<div id='start-containers-run-commands-and-shell-ssh-style'/>
## Start Containers, Run Commands and Shell (SSH-style)
```bash
# Start a container - pull it from Docker hub if not present. You can name it.
# Standard output/error will be redirected to the current terminal
docker run hello-world
docker run --name=test-container-name hello-world

# Start in detached mode - standard output/error will NOT be redirected to the current terminal
docker run -d hello-world

# Start a container and run a specific command from within
docker run busybox echo "Test\n"
docker run ubuntu /bin/echo hello ubuntu container
run ubuntu /bin/bash -c "ls -l"

# Start a container and run a bash sessions interactively (SSH-style)
docker run -it ubuntu bash
docker run -it ubuntu /bin/bash
docker run -it busybox sh

# Attach a shell into a running container (SSH-style). Id/Name is taken from ps.
sudo docker exec -it b3a04a93f46f bash
sudo docker exec -it b3a04a93f46f sh

# Detach from a container's shell without killing it (keyboard shortcut):
Ctrl + p + q

# Reattach/reconnect to running shell (if doconnected with the above shortcut)
docker attach 32c5a521c664

# Run an arbitrary command within a running container
docker exec b3a04a93f46f /bin/echo "Hello again"
```

<div id='networking-and-persistent-volumes'/>
## Networking and Persistent Volumes
```bash
# Start a web server in a container; map its port (80) to a localhost port (8080)
docker run -p 8080:80 nginx

# If containers can't access the hosts networking (run on host)
sysctl -w net.ipv4.ip_forward=1

# Force the container to use a specific DNS server
docker run --dns 8.8.8.8 ubuntu

# Mount a host folder '~/test-vol' to a container as '/host-test-vol'
docker run -it -v ~/test-vol:/host-test-vol ubuntu /bin/bash
```

<div id='restart-stop-and-delete-containers'/>
## Restart, Stop, and Delete Containers
```bash
# Start/restart a stopped/running container
docker start 33712928d6c5
docker restart 33712928d6c5

# Gracefully stop a running container by.
# Sends SIGTERM to the root process.
docker stop b3a04a93f46f

# Forcefully stop a running container (SIGKILL)
docker kill b3a04a93f46f

# Remove a stopped container
docker rm b3a04a93f46f

# Remove a container regardless of its state (i.e. running, stopped)
docker rm -f b3a04a93f46f
```

<div id='diff-a-container-and-create-an-image'/>
## Diff a Container and Create an Image
```bash
# Check the changes in a container, since it was started from an image
docker diff 60b4f89dfc7f

# Create an image from a container - it will include all new changes
docker commit 60b4f89dfc7f test-image-name
```

<div id='dockerfile-create-a-new-image'/>
# Dockerfile - Create a New Image

<div id='sample-dockerfile'/>
## Sample Dockerfile

```bash
# Define base docker image - will be pulled from the repo if not present
FROM ubuntu:16.04

# Metadata - Let people know who build this
MAINTAINER John Dockerson john.dockerson@dockeremail.com

# Install and set-up the current image - run only when creating the image
RUN apt-get update
RUN apt-get install -y git nginx

# Default working directory of all containers from this image
WORKDIR /var

# Set an env. variable 'APP-VER' for all containers from this image
ENV APP-VER 1.0

# Exposes port 80 - only to other containers! Can't be accessed from localhost.
EXPOSE 80

# Mount a volume from the host - less flexible that '-v' option.
# Can't specify custom mount point.
VOLUME /var/log

# Copy file(s) from the host to the image. If a folder - copies the content!
ADD folder /usr/local/
COPY folder /usr/local/
ADD folder /usr/local/folder
ADD file.txt /usr/local/

# Commands that will run when starting a container - e.g. start web/db server
# If the caller passes a command - this will not run! You can have only 1 CMD command.
CMD service nginx start

# Like CMD, but can't be overridden if the caller passes a command!
ENTRYPOINT service nginx start
```

<div id='working-with-dockerfile'/>
## Working with Dockerfile

```bash
# Build a docker image from Dockerfile in the current folder
docker build .

# Build a docker image from Dockerfile and name it
docker build -t nikolay/testimage .
docker build -t nikolay/testimage:ver1 .
```

<div id='docker-compose'/>

# Docker Compose

Example borrowed from the official [documentation](https://docs.docker.com/compose/django/):

<div id='sample-docker-compose-yml-file'/>

## Sample docker-compose.yml File
```yml
# Use the latest version of docker compose
version: '2'

# Define each container in the services section
services:
# A container from an existing image
  db:
    image: postgres

# A container from a custom image (defined in Dockerfile)
  web:
#   Build the image, map ports, volumes, etc. Specify a command
    build: .
    command: python manage.py runserver 0.0.0.0:8000
    volumes:
      - .:/code
    ports:
      - "8000:8000"
    depends_on:
      - db

```

<div id='run-docker-compose'/>

## Run Docker Compose

```bash
# From the same folder - start all containers defined in docker-compose.yml.
# If some of the containers are running - starts only the needed ones
docker-compose up

# Force all containers to be restarted
docker-compose up --force-recreate

# Stop all containers
docker-compose stop
```
