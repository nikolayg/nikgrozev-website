---
layout: post
title: Advanced Automated CloudStone Setup in Ubuntu VMs [Part 2]
date: 2014-06-02 22:31:06.000000000 +10:00
type: post
published: true
status: publish
excerpt: 
    This post covers how to achieve a more advanced CloudStone deployment with a dedicated load 
    balancer in front of the web server, so that new web servers can be added dynamically. To accommodate 
    multiple web servers we configure a central repository for the media filestore, which is shared 
    across all of them ...
categories:
- CloudStone
- blog
tags:
- AWS
- Bash
- Benchmark
- CloudSuite
- CloudStone
- Load Balancer
- Web App
author:
  login: nikolaygrozev
  email: nikolay.grozev@gmail.com
  display_name: nikolaygrozev
  first_name: 'Nikolay'
  last_name: 'Grozev'
---

# Introduction

In a [previous post](/2014/05/10/automated-cloudstone-setup-in-ubuntu-vms/) 
I described a set of scripts for automatic installation of the 
[CloudStone](http://parsa.epfl.ch/cloudsuite/web.html) web benchmark, 
part of the [CloudSuite 2.0](http://parsa.epfl.ch/cloudsuite/cloudsuite.html "CloudStone 2.0") suite. 
They set up CloudStone in its standard topology - i.e. a workload driver/client, a web/app server and 
a databse server. While this is sufficient to explore the servers' utilisation under different workloads, 
it is not enough to test its behaviour under various autoscaling and load balancing approaches, 
as it has a single web server.

This post covers how to achieve a <u>more advanced CloudStone</u> deployment with a dedicated load 
balancer "_in front of_" the web server, so that new web servers can be added dynamically. To accommodate 
multiple web servers we also need a central repository for the media <u>filestore</u>, which is shared 
across all of them. Thus we use a dedicated Network File System (NFS) server to host the _<u>filestore</u>_.

Again the installation is automated with a set of scripts, which have been tested in Amazon AWS. 
In principle, they should also work in other Infrastructure as a Service (IaaS) environments, as long 
they support 64bit Ubuntu 14.04 virtual machines.

# SetUp Overview

Unlike the standard CloudStone deployment, now we'll need two more Virtual Machines - 
(i) a load balancer and (ii) a NFS server. Thus overall we'll need the following <u>64bit Ubuntu 14.04</u> VMs:

*   Client/Driver VM - drives/runs the workload (i.e. emulated users' requests) to the load balancer.
*   Load balancer VM - redirects the requests to a set of web servers, using a HAProxy load balancer.
*   Web/App server - serves users' requests in a Nginx web server. Accesses the the _<u>filestore</u>_ of media files hosted on the NFS server.
*   NFS server - hosts the filestore on its file system, and shares it trough NFS. <u>Must have significant disk space!</u>
*   DB server - hosts a MySql server and a geocoding application.

The client uses [Faban](https://github.com/akara/faban) to set up and monitor the load balancer, DB and the NFS servers, 
through SSH and Java RMI - see the diagram. Note that in this setting the Client VM does not monitor the web server VM. 
In other words, the client does not "know" of its existence, and only sees the load balancer as its endpoint web server. 
This allows us to dynamically provision more instances of the web server VM and to transparently associate them with the load balancer.

<figure>
  <img src="/assets/images/Advanced Automated CloudStone Setup in Ubuntu VMs Part 2/cloudstoneoverviewwithloadbalancer1.png" alt="CloudStone Architecture with a load balancer and a NFS server" >
  <figcaption>CloudStone Architecture with a load balancer and a NFS server.</figcaption>
</figure>


# Prerequisites

To be on the safe side I'll repeat the **<u><span style="color:#993300;text-decoration:underline;">Security Notice!</span></u>** 
from the previous CloudStone post. CloudStone implicitly requires that all VMs can _ping_ each 
other and connect to each other at random ports. Thus you should create a separate firewall 
setting (i.e. a Security Group), which allows this and which is <u>used only for running CloudStone</u>. 
The following screenshot shows the security group used in this tutorial, which allows pinging and 
access to all TCP ports from anywhere. Furthermore, CloudStone assumes that all machines can ssh 
into each other without any prompts. Hence our scripts need to copy the key (i.e. the <u>_pem_</u> file) 
across all machines. Thus it is advised to <u>create a new pem file only for running CloudStone</u>.

<figure>
  <img src="/assets/images/Advanced Automated CloudStone Setup in Ubuntu VMs Part 2/securitygroup-1024x538.png" alt="CloudStone security Group in AWS" >
  <figcaption>CloudStone security Group in AWS.</figcaption>
</figure>


Before starting the setup, you should create 5 VM instances running **<u>64 bit Ubuntu 14.04</u>** – 
the client, the load balancer, the web, NFS, and the DB servers. They should be configured with a 
permissive Security Group and using the same dedicated _pem_ file, as described above. 
The NFS server VM should have at least <u>200GB mounted disk storage</u> in order to host the _filestore_ 
for significant workloads. In AWS EC2, you could just increase the root EBS volume size to 1TB when 
starting the NFS Server VM from the AWS web interface.

You should have one more Linux machine, which will guide the installation process. 
We will call it the “<u>_Installer_</u>” machine. This could be just a PC/laptop or a dedicated VM in the cloud. 
It uses SSH and SCP to configure appropriately and start all servers. The next diagram depicts the setup topology.

<figure>
  <img src="/assets/images/Advanced Automated CloudStone Setup in Ubuntu VMs Part 2/cloudstoneinstallationtopologywithloadbalancer.png" alt="CloudStone topology with a load balancer" >
  <figcaption>The installer Linux machine uses SSH and SCP to configure the CloudStone VMs. All CloudStone VMs run 64bit Ubuntu 14.04, have the same pem file (e.g. CloudStone.pem) and the same permissive security group (e.g. named CloudStone).</figcaption>
</figure>


# Installation

Once all of this is done you can start the installation. The installation scripts are hosted in a 
[GtiHub project called CloudStoneSetupOnUbuntuAdvanced](https://github.com/nikolayg/CloudStoneSetupOnUbuntuAdvanced), 
which is a fork of the original installation scripts project. From the installer machine you should either clone via 
git or download and extract this project. Then you need to copy your pem file (e.g. CloudStone.pem) in the scripts' directory.

Next you need to edit <u>main_installer.sh</u> 
with the DNS or IP addresses of the client, load balancer (lb), app/web server (as) and database server (db) VMs. 
Additionally, you can put the name of your pem file, if different from CloudStone.pem. 
You can also put an initial load scale, used to compute how many users the system should be populated with. 
For more info on the load scale parameter look at the [official CloudStone documentation](http://parsa.epfl.ch/cloudsuite/web.html). 
Your <u>main_installer.sh</u> should start with something like this:

```bash
clientIPAddress=ec2-XX-XX-XX-XX.ap-southeast-2.compute.amazonaws.com
lbIPAddress=ec2-XX-XX-XX-XX.ap-southeast-2.compute.amazonaws.com
asIPAddress=ec2-XX-XX-XX-XX.ap-southeast-2.compute.amazonaws.com
dbIPAddress=ec2-XX-XX-XX-XX.ap-southeast-2.compute.amazonaws.com
nfsIPAddress=ec2-XX-XX-XX-XX.ap-southeast-2.compute.amazonaws.com

pemFile=CloudStone.pem
userName=ubuntu

load_scale=100
```


Now you can start the installation by typing the following from the scripts folder:

```bash
sudo bash main_installer.sh
```

In the beginning, you may be prompted to accept the addition of the 5 VMs to the list of known ssh hosts. 
Installation may take quite some time! In the meantime you can monitor the following installation logs on the installer machine:

*   <u>~/client-setup.log</u> – the log for installing the Client VM. The first to be written.
*   <u>~/nfs-setup.log</u> – the log for installing the NFS server VM. Written after the installation of the Client VM.
*   <u>~/lb-setup.log</u> – the log for installing the load balancer VM. Written after the installation of the NFS VM.
*   <u>~/as-setup.log</u> – the log for installing the Web/AS VM. Written after the installation of the LB VM.
*   <u>~/db-setup.log</u> – the log for installing the DB VM. Written after the installation of the Web/AS VM.

If all is successful you shouldn’t see error messages in the logs and each log file should end with an “INSTALLATION SUMMARY” 
section similar to the following one:

```
======== ======== INSTALLATION SUMMARY ======== ========
 
$JAVA_HOME:     /usr/lib/jvm/java-6-openjdk-amd64
$JDK_HOME:      /usr/lib/jvm/java-6-openjdk-amd64
$OLIO_HOME:     /cloudstone/apache-olio-php-src-0.2
$FABAN_HOME:    /cloudstone/faban
$APP_DIR:       
$FILESTORE:     
$NFS_MOUNT_PATH:
$CATALINA_HOME: /cloudstone/apache-tomcat-6.0.35
$GEOCODER_HOME: /cloudstone/geocoderhome
```

You'll need these environment variables' values to run CloudStone.

# Running CloudStone

Just like in the [previous post](/2014/05/10/automated-cloudstone-setup-in-ubuntu-vms/), 
to run CloudStone you need to point your web browser to <u>http:// [ client-vm-address ] :9980</u> to see the following screen:

<figure>
  <img src="/assets/images/Advanced Automated CloudStone Setup in Ubuntu VMs Part 2/fabanoliomainscreen-1024x703.png" alt="Faban Main Screen" >
  <figcaption>Faban Main Screen.</figcaption>
</figure>


Then you need to schedule a new run, as in the previous post. The only differences are that 
(i) you need to use the load balancer address, instead of the web server and (ii) you need to specify 
the NFS server as a filestore server. The following screenshots give an overview:

<!-------------------------------------------- Image Galery -------------------------------------------->
<figure class="half">
  <a class="image-popup-fit-width" href="/assets/images/Advanced Automated CloudStone Setup in Ubuntu VMs Part 2/wizard-javaadvanced.png">
    <img src="/assets/images/Advanced Automated CloudStone Setup in Ubuntu VMs Part 2/wizard-javaadvanced.png">
  </a>
  <a class="image-popup-fit-width" href="/assets/images/Advanced Automated CloudStone Setup in Ubuntu VMs Part 2/wizard-driveradvanced.png">
    <img src="/assets/images/Advanced Automated CloudStone Setup in Ubuntu VMs Part 2/wizard-driveradvanced.png">
  </a>
  <a class="image-popup-fit-width" href="/assets/images/Advanced Automated CloudStone Setup in Ubuntu VMs Part 2/wizard-web-serveradvanced1.png">
    <img src="/assets/images/Advanced Automated CloudStone Setup in Ubuntu VMs Part 2/wizard-web-serveradvanced1.png">
  </a>
  <a class="image-popup-fit-width" href="/assets/images/Advanced Automated CloudStone Setup in Ubuntu VMs Part 2/wizard-data-serversadvanced.png">
    <img src="/assets/images/Advanced Automated CloudStone Setup in Ubuntu VMs Part 2/wizard-data-serversadvanced.png">
  </a>
  <figcaption>CloudStone overview.</figcaption>
</figure>
<!-------------------------------------------- Image Galery -------------------------------------------->


After setting up the benchmark click OK to start it. You can view the benchmark progress and 
eventually the result from the “View Results” menu.

# Scaling Up

At this point the current set-up has a single web/app server. We can easily provision a new one 
and associate with the load balancer. To achieve this, we firstly need to replicate the web/app server VM. 
In AWS EC2, you can create a VM image from it (the so called AMI) and then create new VMs out of it. 
Other infrastructure as a service (IaaS) clouds have similar functionality as well. 
Of course, the new VM should have the same security group as the others.

After the new VM is created from the image it will automatically start its web server and will 
mount its NFS storage upon boot, so you don't need to take care or that. However, you still need to 
associate it with the load balancer. Lets assume the IP/DNS address of the initial/old web/app server 
is <u>asIP1</u> and the new one's is <u>asIP2</u>. Then you need to login into the load balancer 
VM and from the home directory issue the following commands:

```bash

# Import utility functions
. functions.sh

# Load balance with Round Robin policy with 1:1 ratio
resetLoadBalancer asIP1 1 asIP2 1

# Reload the load balancer's configuration
sudo service haproxy reload
```

Of course, in the above <u>asIP1</u> and <u>asIP2</u>, should be replaced with the corresponding addresses. 
After that, the load balancer will evenly distribute the request to the two web servers. 
You can also implement weighted round robin (e.g. with 2:3 ratio) by simply replacing the 
invocation to <u>resetLoadBalancer</u> with:

```bash
....
resetLoadBalancer asIP1 2 asIP2 3
...
```

Similarly, if you provision a 3rd web/app server <u>asIP3</u> and want to load balance with ratio 2:3:4, you should call:

```bash
...
resetLoadBalancer asIP1 2 asIP2 3 asIP3 4
...
```

# Under the Cover

I will not go into the implementation details of the scripts, as I've already discussed most of it in the 
[previous post](/2014/05/10/automated-cloudstone-setup-in-ubuntu-vms/). In essence the 
<u>main_installer.sh</u> script orchestrates the installation by transferring configuration 
files and installation scripts to all VMs. On each VM the installation is performed by a separate bash script. These are:

*   <u>client-setup.sh</u> – executed on the client VM. Its log is redirected to _~/client-setup.log_ on the installer machine.
*   <u>lb-setup.sh</u> – executed on the load balancer VM. Its log is redirected to _~/lb-setup.log_ on the installer machine.
*   <u>nfs-setup.sh</u> – executed on the NFS server VM. Its log is redirected to _~/nfs-setup.log_ on the installer machine.
*   <u>as-setup.sh</u> – executed on the web/application server VM. Unlike the 
[previous scripts](/2014/05/10/automated-cloudstone-setup-in-ubuntu-vms/), it mounts the filestore as a NFS volume, 
instead of hosting it locally. Its log is redirected to _~/as-setup.log_ on the installer machine.
*   <u>db-setup.sh</u> – executed on the DB VM. Its log is redirected to _~/db-setup.log_ on the installer machine.

Common installation logic is still implemented in <u>base-setup.sh</u> and <u>base-server.sh</u>. 
A lot of common and reusable functions are stored in <u>functions.sh</u>. The script <u>as-image-start.sh</u> is set to 
automatically execute at boot time of the web/app server VM and starts it web server and NFS mounting.

A major problem in the set up of the load balancer was the stateful nature of the Olio workload. 
Olio maintains session data in the web server's memory for every user. Thus requests from the same 
user have to be redirected by the load balancer to the same server (a policy known as _<u>sticky load balancing</u>_). 
Usually, this is achieved through a mechanism called <u>ip-hashing</u>, which essentially means that HTTP requests coming 
from the same IP address are redirected to the same web server. However, in CloudStone all requests originate from the faban 
driver/client and thus have the same source IP address.

That's why we use the HAProxy load balancer. Unlike other load balancers (e.g. Nginx) it supports cookie-based 
load balancing. Upon session creation, the web server installs an identifying cookie in the client, which is later 
on used by the load balancer to dispatch the subsequent requests to the appropriate server.

# References

Official installation documentation:

*   [http://parsa.epfl.ch/cloudsuite/web.html](http://parsa.epfl.ch/cloudsuite/web.html)