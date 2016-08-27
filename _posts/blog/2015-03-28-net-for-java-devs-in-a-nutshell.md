---
layout: post
title: ".NET for Java Devs in a nutshell"
date: 2015-03-28 00:15:28.000000000 +11:00
type: post
published: true
status: publish
excerpt: 
    .Net is a software development platform, which allows code written in different languages to interoperate. 
    C# and the .Net model have some resemblances with some java-based technologies. This article aims to briefly 
    explain some of its core concepts by comparing and contrasting the two “worlds” ...
categories:
- C#
- blog
tags:
- C#
- .NET
- Java
author:
  login: nikolaygrozev
  email: nikolay.grozev@gmail.com
  display_name: nikolaygrozev
  first_name: 'Nikolay'
  last_name: 'Grozev'
---

# Introduction

.Net is a software development platform, which allows code written in different languages to interoperate. 
C# and the .Net model have some resemblances with some java-based technologies. This article aims to briefly 
explain some of its core concepts by comparing and contrasting the two “worlds”.

# Specifications and Implementations

There are several main specifications which define what .Net actually is:

1.  The **Common Intermediate Language** (CIL or just IL) is a language independent representation of 
compiled code. Similarly, to java byte code, which can be compiled from several of languages 
(Java, Scala, Jython etc.), CIL can be compiled from a multitude of languages.
2.  The **Common Language Runtime** (CLR) is an execution environment (like the JVM), which executes 
CIL code. It manages threads, garbage collection, just-in-time (JIT) compilation etc.
3.  The **Common Type System** (CTS) defines all data types and programming constructs supported by the CLR, 
and which can be described in CIL. In java terms, you can think of it as a specification of the 
JVM's types and constructs.
4.  The **Common Language Specification** (CLS) is a subset of CTS. Not all languages support all 
features of the CLR runtime (i.e. CTS). For example a language may not have unsigned long. The CLS defines 
a subset of CTS, which should be supported by all .Net languages. If some code needs to be integrated with 
other languages, it's externally visible interface (function signatures, member variables etc.) must only 
use CLS types. There are ways to make the compiler enforce that. There is no similar concept in the java world, 
and all languages must support all JVM types in order to interoperate.
5.  The **Framework Class Library** (FCL) and its subset the **Base Class Library** (BCL) define the standard 
library functionalities, which can be used by all languages. In a java environment, the alternative would be the standard java library.
6.  The **Common Language Infrastructure** (CLI) is a “master” specification, which includes all previous 
ones (CIL, CLR, CTS, CLS etc.) and defines their relationships. .Net is the most famous implementation of CLI. 
[Mono](http://www.mono-project.com/) is a major multi-platform open source alternative. In November 2014, 
Microsoft announced it will open source .Net core (the fundamental part of .Net) to GitHub and will start working to make 
.Net multi-platform (as of now it works on Windows only).


<figure>
  <img src="/assets/images/NET for Java Devs in a nutshell/dot_net.jpg" alt="Relationship between specifications" >
  <figcaption>Relationship between specifications.</figcaption>
</figure>


# Assemblies

In .Net assemblies are files (*.dll or *.exe) which contain compiled CIL code and a manifest. 
The manifest defines the assembly's name, version and dependencies/references to other assemblies. 
An assembly is basically packaged compiled code, and a similar java concept would be a JAR file. 
A JAR file also has a manifest, but is not required to contain name, version and dependencies/references. 
Some component models (e.g. [OSGI](http://en.wikipedia.org/wiki/OSGi)) demand that jar files' manifests contain such information.

A Jar file is in fact a zip archive, and can be easily opened and explored. 
A .Net assembly on the hand can be explored with a tool called *ildasm.exe*, which ships with .Net.

Assemblies can be built with the respective compiler for the used language (e.g. *csc* for C#), similar to 
the *javac* and *jar* utilities. A response file (*.rsp) can contain multiple build instructions 
and configurations. Assemblies can also be built with an IDE like Visual Studio, SharpDevelop or MonoDevelop.

The **Global Assembly Cache** (GAC) is a repository, which contains assemblies reusable between 
multiple applications – mostly system binaries. Users can install assemblies in GAC using system tools, 
which ship with .Net, although this is not advisable.

Big projects need tools to manage the dependencies on components (assemblies or JARs) and to automate 
the build. In the java domain, tools like [Maven](http://maven.apache.org/) and [Gradle](http://gradle.org/) 
can be used. In the .Net world, [NuGet](https://www.nuget.org/) can be used to manage assembly dependencies. 
Integration with Mono on non-Windows systems seems to be in progress. 
[MsBuild](http://en.wikipedia.org/wiki/MSBuild) seems to be the preferred automated build system for .Net. 
It also allows you to build Visual Studio solutions and projects without Visual Studio.

In .Net, assemblies are not only deployment artefacts – they are also logical entities. 
For example C# defines an access modifier called `internal`, which denotes that the respective member 
can only be accessed from the same assembly. Similarly, members with modifier `protected internal` 
can be accessed either from the assembly or through inheritance. There does not seem to be an alternative 
to this in Java. In OSGI you can achieve similar access restriction by specifying which JAR/bundle packages 
are visible to the outside world.
