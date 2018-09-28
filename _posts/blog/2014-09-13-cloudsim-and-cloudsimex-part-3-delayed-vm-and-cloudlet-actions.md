---
layout: post
title: CloudSim and CloudSimEx [Part 3] – Delayed VM and Cloudlet actions
date: 2014-09-13 03:43:30.000000000
type: post
published: true
status: publish
excerpt: 
    Many few CloudSim users have asked how to submit/destroy a VM or a cloudlet with a delay. 
    Unfortunately, this is far from trivial, as it involves launching and handling custom events 
    and ensuring the internal state of the broker is consistent - e.g. all cloudlets of a terminated 
    VM are killed. That's why CloudSimEx introduced a new broker DatacenterBrokerEX, which among 
    other new features also supports dynamic provisioning and scheduling of VMs and cloudlets ...
categories:
- CloudSimEx
- Java
tags:
- CloudSim
- CloudSimEx
- Java
- Maven
- Simulation
---

# Introduction

Many CloudSim users have asked on the mailing lists how to submit/destroy a VM or a cloudlet with a delay. 
Unfortunately, this is not straightforward in CloudSim, and users have to develop a custom broker and override 
several methods to achieve this. This is far from trivial, as it involves launching and handling custom events 
and ensuring the internal state of the broker is consistent - e.g. all cloudlets of a terminated VM are killed.

That's why CloudSimEx introduced a new broker 
[DatacenterBrokerEX](https://github.com/Cloudslab/CloudSimEx/blob/master/cloudsimex-core/src/main/java/org/cloudbus/cloudsim/ex/DatacenterBrokerEX.java), 
which among other new features also supports dynamic provisioning and scheduling of VMs and cloudlets.


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


# Prerequisites

I assume you’ve already worked through Part 1 of this series. If not, now is a good time to check it out – 
[CloudSim and CloudSimEx [Part 1]](/2014/06/08/cloudsim-and-cloudsimex-part-1/).

# Some Code Please

To demonstrate how the delayed actions work, we modify 
[CloudSimExample1](https://code.google.com/p/cloudsim/source/browse/trunk/modules/cloudsim-examples/src/main/java/org/cloudbus/cloudsim/examples/CloudSimExample1.java), 
which simply creates a VM and schedules a cloudlet on it. Our modification starts the VM with a 2 seconds 
delay, and destroys it with a 500 secs delay. Also, we delay the cloudlet submission with 10 secs. The full 
code of the example can be found in the class 
[DelayExample1](https://github.com/Cloudslab/CloudSimEx/blob/master/cloudsimex-examples/src/main/java/org/cloudbus/cloudsim/ex/DelayExample1.java). 
Below we will focus only on the usage of the functionalities.

First we need to use a broker of type 
[DatacenterBrokerEX](https://github.com/Cloudslab/CloudSimEx/blob/master/cloudsimex-core/src/main/java/org/cloudbus/cloudsim/ex/DatacenterBrokerEX.java) 
(or a subclass):

```java 
DatacenterBrokerEX broker = new DatacenterBrokerEX("Broker", 1000);
```

Note that the new broker has an additional parameter called `lifeLength`. 
Standard CloudSim brokers are terminated whenever there are no more running cloudlets/jobs. 
If the simulated system becomes idle for a time period (i.e. no cloudlets are run) the broker 
and the simulation will be terminated! To overcome this issues 
[DatacenterBrokerEX](https://github.com/Cloudslab/CloudSimEx/blob/master/cloudsimex-core/src/main/java/org/cloudbus/cloudsim/ex/DatacenterBrokerEX.java) 
introduces this new parameter called `lifeLength`, and ensures that the simulation and the broker are not terminated before 
`lifeLength` seconds have passed, even if there are no running cloudlets. In the previous code we want the 
broker to be alive for `lifeLength=1000` secs, even if there is an idle period.

After creating the broker, we can go on to instantiate the VM and the clouldlet, 
as we would in a typical CloudSim program. The interesting part is the creation and destruction of VMs with a delay:

```java
// submit vm to the broker after 2 seconds
broker.createVmsAfter(Arrays.asList(vm), 2);
// destroy VM after 500 seconds
broker.destroyVMsAfter(Arrays.asList(vm), 500);
```

The above code will submit the VM 2 seconds afterwards, and will terminate it 500 secs later. 
Analogously, the following code will submit the cloudlet with a delay of 10 secs:

```java
broker.submitCloudletList(Arrays.asList(cloudlet), 10);
```

If we look at the standard output of CloudSim, we can verify that:

```
2.0: Broker: Trying to Create VM #0 in Datacenter_0
2.1: Broker: VM #0 has been created in Datacenter #2, Host #0
10.0: Broker: Sending cloudlet 0 to VM #0
410.0: Broker: Cloudlet 0 received
500.0: Broker: Trying to Destroy VM #0 in Datacenter_0
500.0: Broker: VM #0 has been destroyed in Datacenter #2
```

Moreover, if we schedule the termination of the VM before the completion of the cloudlet, 
its status will be set appropriately to `Cloudlet.FAILED_RESOURCE_UNAVAILABLE`.

# Use Cases

So how is this useful? Well, it allows us to easily define execution scenarios in advance 
before the simulation start. We can predefine that the workload (i.e. cloudlets) will occur at 
a given time to test an autostaling policy, or that VMs will fail at specific times to test fail over policies. 
And most importantly we can do this with just a few lines of code.

[DatacenterBrokerEX](https://github.com/Cloudslab/CloudSimEx/blob/master/cloudsimex-core/src/main/java/org/cloudbus/cloudsim/ex/DatacenterBrokerEX.java) 
and its subclasses have other useful features as well, so stay tuned for future posts!
