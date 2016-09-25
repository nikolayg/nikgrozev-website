---
layout: post
title: Summary of the Parallel Programming with Scala MOOC
date: 2016-9-15 15:22:09.000000000
type: post
published: true
status: publish
excerpt: 
    MOOC ...
categories:
- Bash
- Docker
- blog
tags:
- Bash
- Docker
- Cheatsheet
- Shell
author:
  login: nikolay.grozev@gmail.com
  email: nikolay.grozev@gmail.com
  display_name: nikolay.grozev@gmail.com
  first_name: 'Nikolay'
  last_name: 'Grozev'
---

# Table of Contents

- [Introduction](#introduction)
- [Week 1](#week1)
  - [Introduction to Parallel Programming](#introduction_to_parallel_programming)
- [Week 2](#week2)
- [Week 3](#week3)
- [Week 4](#week4)

<div id='introduction'/>
# Introduction

[EPFL](http://www.epfl.ch/index.en.html) has releases a new Coursera course on [Parallel Programming](https://www.coursera.org/learn/parprog1),
which is a part of the [Functional Programming in Scala Specialisation](https://www.coursera.org/specializations/scala). 
This post summarises the course and can be used as a quick ref-card on the topic.

<div id='week1'/>
# Week 1

<div id='introduction_to_parallel_programming'/>
## Introduction to Parallel Programming

**Parallel computing** is concerned with the simultaneous execution of multiple computations. 
Its goal is faster execution (i.e. *speed-up*) than
traditional synchronous/serial programs. It is enabled by parallel hardware.

Parallel computing is related to the concept of Concurrent computing.
In a [previous post](/2015/07/15/overview-of-modern-concurrency-and-parallelism-concepts/#concurrency-vs-parallelism),
I overviewed the differences between the two concepts. In essence,
concurrency is about composing independent computations to work together, 
while parallelism is about actually executing them simultaneously on suitable
hardware. Concurrency is about the application design and structure, 
while parallelism is about the actual execution. Concurrency is about
modularity and separation of concerns, while parallelism is about efficiency
and execution speed-up.

At the most basic level, there are three types of parallelism:

* [Bit level parallelism](https://en.wikipedia.org/wiki/Bit-level_parallelism) - by increasing
 the CPU word size (e.g. from 32bit to 64bit), the processor can act on bigger chunks 
 of data with fewer operations. For example, two 64 bit integers could be added with a 
 single instruction on 64bit processor.   
* [Instruction level parallelism](https://en.wikipedia.org/wiki/Instruction-level_parallelism - 
some chips can run multiple instructions simultaneosly - 
e.g. [processor pipelines](https://en.wikipedia.org/wiki/Instruction_pipelining). 
* [Task level paralparallelismlism](https://simple.wikipedia.org/wiki/Task_parallelism) - running
 separate instruction streams/series simultaneously.

We'll focus on *Task level parallelism*, because it is on the "software level".

Parallel hardware can be:

 * [Multi-core processors](https://en.wikipedia.org/wiki/Multi-core_processor) - multiple
 processing units (cores) on the same chip. They can potentially share caches.
 * [Symmetric multiprocessors (SMP)](https://en.wikipedia.org/wiki/Symmetric_multiprocessor_system) - 
 multiple independent and identical processors. They can't share caches, but use the same bus.   
 * [General purpose graphics processing unit (GPGPU)](https://en.wikipedia.org/wiki/General-purpose_computing_on_graphics_processing_units) -
 co-processers used together with CPUs. A GPGPU can have thousands of cores executing the
 **same** instruction at the same time, which is suitable for some applications.
 * [Field-programmable gate arrays (FPGA)](https://en.wikipedia.org/wiki/Field-programmable_gate_array) - 
a programmable integrated circuit.   
 * [Computer clusters](https://en.wikipedia.org/wiki/Computer_cluster) - a set of computers
 connected in a high speed network. They work together and appear to the user as a 
 single computer.

We'll focus on *multi-core* and *symmetric multiprocessor* systems.
