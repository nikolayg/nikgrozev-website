---
layout: post
title: Switch Between Multiple Kubernetes Clusters With Ease
date: 2019-10-03 05:22:09.000000000
type: post
published: true
status: publish
excerpt: 
    This article describes how to work with multiple Kubernetes clusters
    in a structured and convenient way ...
categories:
- Kubernetes
tags:
- Kubernetes
- kubectl
- context
---

# Background

I often need to switch between Kubernetes clusters for different projects.
Managing the `kubectl` configuration for each of them and their files can be cumbersome.

By default, the `kubectl` command line client uses the `~/.kube/config` file to store
the Kubernetes endpoint and credentials. If you use 
[minikube](https://minikube.sigs.k8s.io/)
or [Docker Desktop](https://www.docker.com/products/docker-desktop)'s local Kubernetes, 
you should be able to see their configurations in that file.

When you work with a cloud based Kubernetes instance, the cloud console will provide
the configuration as a `yml` file. You then need to specify the file
as the value of the environment variable `KUBECONFIG`, which is used by `kubectl`.
This can quickly become cumbersome and hard to maintain.

# Multiple Contexts

To solve this problem, I've started storing all cloud cluster configuration `yml` files
in a new folder `~/.kube/custom-contexts/`. Usually, I'd make a folder for each cloud
Kube instance, which contains the `yml` config and other auxiliary files (e.g. `pem` certificates). 

Here's the content of my `~/.kube` folder:

```
~/.kube/
 ├── cache
 │   └── ...
 ├── config
 ├── custom-contexts
     └── NikTest-Kube-Config
         ├── ca-hou02-NikTest.pem
         └── kube-config-hou02-NikTest.yml
```  

It has a single Kubernetes configuration in the `NikTest-Kube-Config` folder which
contains the `yml` config and `pem` certificate downloaded from the IBM Cloud console.

Now we need to load all this configuration in the `KUBECONFIG` environment variable
so that `kubectl` can switch between them. The following script does the trick:


```bash
# Set the default kube context if present
DEFAULT_KUBE_CONTEXTS="$HOME/.kube/config"
if test -f "${DEFAULT_KUBE_CONTEXTS}"
then
  export KUBECONFIG="$DEFAULT_KUBE_CONTEXTS"
fi

# Additional contexts should be in ~/.kube/custom-contexts/ 
CUSTOM_KUBE_CONTEXTS="$HOME/.kube/custom-contexts"
mkdir -p "${CUSTOM_KUBE_CONTEXTS}"

OIFS="$IFS"
IFS=$'\n'
for contextFile in `find "${CUSTOM_KUBE_CONTEXTS}" -type f -name "*.yml"`  
do
    export KUBECONFIG="$contextFile:$KUBECONFIG"
done
IFS="$OIFS"
```

I put the above code snippet in my `~/.bashrc` so that it loads on every new terminal session. 
This way, if I add or delete configuration files from `~/.kube/custom-contexts`, this will be 
reflected in every subsequent terminal session.

# Switching Between Contexts

Now we can list all preconfigured contexts and see which one is active:

```bash
kubectl config get-contexts

# CURRENT NAME           CLUSTER        AUTHINFO
#         NikTest        NikTest        <****>   
#         docker-desktop docker-desktop docker-desktop         
# *       minikube       minikube       minikube 
```

We can also get the name of the active context/cluster:

```bash
# Prints the current config nema - e.g. minikube
kubectl config current-context
```

Finally, we can switch between the predefined contexts:

```bash
# Switch to a context/cluster
kubectl config use-context NikTest
```