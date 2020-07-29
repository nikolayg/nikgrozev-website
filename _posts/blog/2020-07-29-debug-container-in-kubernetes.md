---
layout: post
title: How To Start A Debug Container in Kubernetes
date: 2020-07-29 05:22:09.000000000
type: post
published: true
status: publish
excerpt: 
    This post demonstrates how to 
    start a temporary debug container in a K8S cluster 
    and open up a terminal sessions into it. This will allow us
    to test various network issues without leaving any permanent
    pods behind.
categories:
- Kubernetes
- Docker
tags:
- Kubernetes
- Docker
- kubectl
---

# Background

On a recent project, I've been troubleshooting some connectivity issues in a Kubernetes cluster. 
The pods were failing to talk
to external on-prem systems and I had to prepare a
[Minimal, Reproducible Example (MRE)](https://stackoverflow.com/help/minimal-reproducible-example) for 
the network administrators. 

In this post, I'll demonstrate how to 
start a temporary debug container in a K8S cluster 
and open up a terminal sessions into it. This will allow us
to test various network issues without leaving any permanent
pods behind.

# The Test Pod & Container

Let's start by creating a YAML file which defines the test pod:

```yml
apiVersion: v1
kind: Pod
metadata:
  name: testcurl
spec:
  containers:
  - name: curl
    image: curlimages/curl 
    command: [ "sleep", "600" ]
```

In the above, we're defining a new pod with a single container
based on the `curlimages/curl` image. This is a minimalistic
image (about 11MB) which includes `curl` - you can obviously choose to use a bigger image with more networking tools.

**The container will complete in 10 minutes and the pod will
be die/exit.** If you need more time for your troubleshooting
please increase the sleep interval in the above config. 

Let's create the pod in the cluster:

```bash
kubectl apply -f pod.yaml
```

After the pod starts, you should be able to open a terminal 
into the container and execute `curl` commands:

```bash
# Will open up a terminal session into the container
kubectl exec -it testcurl -- sh

# We can now curl external addresses or internal services:
> curl http://example.com/
> curl myservice/health
```
