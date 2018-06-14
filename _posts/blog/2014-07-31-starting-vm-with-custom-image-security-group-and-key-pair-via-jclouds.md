---
layout: post
title: Starting VM with Custom Image, Security Group and Key-Pair via JClouds
date: 2014-07-31 02:25:32.000000000
type: post
published: true
status: publish
excerpt: 
    Recently I needed to programatically start EC2 instances from a private 
    AMI (VM Image) as a part of a new 
    provisioning algorithm I was working on. I also wanted to specify my predefined security group and 
    key-pair, so that the new VM is usable right away. Lastly, I wanted to implement it as EC2-independent 
    as possible, so that I can include multiple clouds later on. After struggling a bit with both 
    Apache LibCloud and Apache JClouds I could finally implement this in JClouds ...
categories:
- Java
tags:
- AWS
- Java
- JClouds
- Virtual Machine
- VM Image
---

# Introduction

Recently I needed to programatically start EC2 instances from a private 
[AMI](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html) (VM Image) as a part of a new 
provisioning algorithm I was working on. I also wanted to specify my predefined security group and 
key-pair, so that the new VM is usable right away. Lastly, I wanted to implement it as EC2-independent 
as possible, so that I can include multiple clouds later on.

There are a few Multi-Cloud libraries out there, whose purpose is to provide just that - a cloud 
provider agnostic API for managing cloud resources. After struggling a bit with both 
[Apache LibCloud](https://libcloud.apache.org/) and 
[Apache JClouds](https://jclouds.apache.org/) I could finally implement this in JClouds.

# JClouds Implementation

The easiest way to set up JClouds is by using it as a 
[Maven dependency](https://jclouds.apache.org/start/install/). 
The following method implements the provisioning:

```java
public static List<List<String>> launchInstances(
        String providerId,
        String userName,
        String password,
        String locationId,
        String imageId,
        String hardwareId,
        String securityGroupName,
        String keyPairName,
        String groupName,
        int numVMs,
        Properties imageOwnerIdFilter) {
 
    // Get the Compute abstraction for the provider.
    // Override the available VM images
    ComputeService compute = ContextBuilder.
            newBuilder(providerId).
            credentials(userName, password).
            overrides(imageOwnerIdFilter).
            buildView(ComputeServiceContext.class).getComputeService();
 
    // Create a template for the VM
    Template template = compute.
            templateBuilder().
            locationId(locationId).
            imageId(imageId).
            hardwareId(hardwareId).build();
 
    // Specify your own security group
    TemplateOptions options = template.getOptions();
    options.securityGroups(securityGroupName);
 
    // Specify your own keypair if the current provider allows for this
    try {
        Method keyPairMethod = options.getClass().getMethod("keyPair", String.class);
        keyPairMethod.invoke(options, keyPairName);
    } catch (NoSuchMethodException | SecurityException | IllegalAccessException | IllegalArgumentException
            | InvocationTargetException e) {
        throw new IllegalStateException("Provider: " + providerId + " does not support specifying key-pairs.", e);
    }
 
    final List<List<String>> launchedNodesAddresses = new ArrayList<>();
    try {
        // Launch the instances...
        Set<? extends NodeMetadata> launchedNodesMetadata = compute.createNodesInGroup(groupName, numVMs, template);
 
        // Collect the addresses ...
        for (NodeMetadata nodeMetadata : launchedNodesMetadata) {
            launchedNodesAddresses.add(new ArrayList<>(nodeMetadata.getPublicAddresses()));
        }
    } catch (RunNodesException e) {
        throw new IllegalStateException("Nodes could not be created.", e);
    }
 
    return launchedNodesAddresses;
}
```

The input parameters of the method are:

*   `providerId` - a JClouds specific ID of the provider (AWS, RightScale etc.);
*   `userName` - the user name for the provider;
*   `password` - the password or secret key;
*   `locationId` - a JClouds specific ID of the location of the cloud data centre site;
*   `imageId` - the ID of the VM Image or AMI in AWS;
*   `hardwareId` - the type of the VM (e.g. m1.small);
*   `securityGroupName` - the name of the predefined security group;
*   `keyPairName` - the name of the predefined key-pair;
*   `groupName` - a name pattern for the newly created VMs. **Must be lower case**;
*   `numVMs` - how many VMs you want to create at once. If 1 - a single VM is created;
*   `imageOwnerIdFilter` - by default some providers do not list private VM images. Thus, you need to specify a custom filter/query that will make them list your private images.

In the next section we'll see how all these parameters can be specified.

The above code is pretty self-evident and follows the standard JClouds templates. 
The first interesting point is that we need to override the list of available VM images 
(line 19) with a custom query, so that we can use our own private VM image. 
Secondly JClouds does not support specifying key-pairs for all cloud providers and hence the 
"keyPair" method is not present in any base class. To avoid using provider specific `ComputeService` 
implementations we dynamically inspect if the current provider supports key-pairs (lines 35-36).

As a result the method returns a list of the addresses for each instantiated VM.

# Running in AWS

The above method is generic and is not bound to AWS EC2. 
In order to run it on EC2, we need to specify the appropriate parameters as in the following code sample:

```java
String providerId = "aws-ec2";
String accesskeyid = "...";
String secretkey = "...";
String imageOwnerId = "...";
String locationId = "ap-southeast-2a";
String imageId = "ap-southeast-2/ami-XXX";
String hardwareId = org.jclouds.ec2.domain.InstanceType.T1_MICRO;
String securityGroupName = "SecGroupName";
String keyPairName = "KeyPairName";
String groupName = "groupname"; // Must be lower case
int numVMs = 1;
 
Properties imageOwnerIdFilter = new Properties();
imageOwnerIdFilter.setProperty(
    "jclouds.ec2.ami-query", "owner-id=" +
    imageOwnerId +
    ";state=available;image-type=machine");
 
List<List<String>> launchedNodesAddresses = launchInstances(providerId,
    accesskeyid,
    secretkey,
    locationId,
    imageId,
    hardwareId,
    securityGroupName,
    keyPairName,
    groupName,
    numVMs,
    imageOwnerIdFilter);
 
System.out.println(launchedNodesAddresses);
```

Note that in the above code we use the AWS region as a part of the image ID (line 6). 
This is a requirement for EC2. Also, we constructed an AWS specific image query with 
the image owner id as a parameter (line 14).
