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
  - [Parallel Programming Introduction](#parallel_programming_introduction)
  - [Parallelism on the JVM](#parallelism_on_the_jvm)
  - [Complexity of Parallel Algorithms](#complexity_of_parallel_algorithms)
  - [Benchmarking](#benchmarking)
- [Week 2](#week2)
- [Week 3](#week3)
- [Week 4](#week4)

<div id='introduction'/>
# Introduction

[EPFL](http://www.epfl.ch/index.en.html) has released a new Coursera course on [Parallel Programming](https://www.coursera.org/learn/parprog1),
which is a part of the [Functional Programming in Scala Specialisation](https://www.coursera.org/specializations/scala). 
This post summarises the course and can be used as a quick ref-card on the topic.

<div id='week1'/>
# Week 1

<div id='parallel_programming_introduction'/>
## Parallel Programming Introduction

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
* [Instruction level parallelism](https://en.wikipedia.org/wiki/Instruction-level_parallelism) - 
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

<div id='parallelism_on_the_jvm'/>
## Parallelism on the JVM

On the operating system (OS) level, a 
[process](https://en.wikipedia.org/wiki/Process_(computing)) is an 
instance of a program being executed and has its own 
[address space](https://en.wikipedia.org/wiki/Address_space). 
Processes can not access each others' address spaces and 
[are isolated](https://en.wikipedia.org/wiki/Process_isolation).
[Inter-process communication](https://en.wikipedia.org/wiki/Inter-process_communication) 
techniques like sockets and pipelines can be used to communicate between processes.

OS processes can be expensive to create, as each one has its own address space, 
program code, open file handles etc. 
Furthermore, the inter-process communication can be inefficient and cumbersome to program.

Enter [threads](https://en.wikipedia.org/wiki/Thread_(computing)) (a.k.a. lightweight processes). 
A thread is a separate line of execution (a sequence of instructions) within a process. 
A process can have multiple threads, all of which share its address space, file handles etc. 
Starting a thread is cheaper, as fewer resources need to be allocated. 
The threads within a process are concurrent and can execute in parallel. 
Each thread maintains its own programming stack. Alike processes, the OS scheduler preemptively 
schedules all threads.

Scala uses the JVM classes for threads, and hence creating and starting a thread is very similar 
to Java. One approach to creating a thread is to extend from the Thread class:

```scala
// Subclassing class Thread
class CustomThreadClass extends Thread {

  // Override the run method - its code will
  // be run in a separate thread
  override def run = println("From CustomThreadClass")
}
```
Then we can `start` a thread from that class, and wait for its completion (`join`):

```scala
val thread = new CustomThreadClass()

// Start the thread
thread.start()

// Block the current thread until it's finished
thread.join()
```

It is often important to ensure that a piece of code is atomic - i.e. it can not be
execute simultaneosly from multiple threads. To achieve this, the JVM provides
`synchronized` code blocks. Each such block is associated with a "monitor object", 
and only one thread can execute the block associated with it. Here's an example:

```scala
// Thread class - takes the monitor as a constructor arg
class Atom(name : String, monitor: AnyRef) extends Thread {
  override def run = {
    // Synch by the monitor given in the constructor
    monitor.synchronized {
      // This code can not run smultaneously for a given monitor
      println(s"Atomic Operation Part 1 in $name")
      println(s"Atomic Operation Part 2 in $name")
      println(s"Atomic Operation Part 3 in $name")
    }
  }
}
```

Then we can run the following experiment:

```scala
// We'll use this reference to implement atomicity
val monitor = ""

// Pass the same monitor to the threads
val thread1 = new Atom("Atom 1", monitor)
val thread2 = new Atom("Atom 2", monitor)

// Start the threads. Their executions will not overlap.
thread1.start()
thread2.start()

// Block the current thread until they're finished
thread1.join()
thread2.join()
```

This overview of JVM threads only scratches the surface. For more details you can refer to
the [official tutorial](http://docs.oracle.com/javase/tutorial/essential/concurrency/),
which covers more advanced topics on threads and processes.
<!--This basic overview should be sufficient to follow the rest of the article.-->

<div id='complexity_of_parallel_algorithms'/>
## Complexity of Parallel Algorithms

With serial algorithms, we often use the *big-O* notation to evaluate
their computational complexity. Ideally, we would like to have the same 
with parallel algorithms. However, in the parallel case we've got another
parameter - the number of processing units.

Let us denote by \\( W(e) \\) the number of steps needed
by the computation/algorithm \\( e \\). We'll call \\( W(e)) \\)
the **work** required for \\( e \\), and it's a measure of the time
it would take if executed serially. \\( W(e)) \\) is an upper
bound of the execution time if run in a parallel fashion.

Now we can introduce a lower bound as well. With \\( D(e) \\) we denote
the execution time of \\( e \\) if given infinite number of processing
units. We call \\( D(e) \\) the **depth** of \\( e \\).

Assuming our hardware has capacity to run \\( P \\) number of threads 
simultaneously, we can approximate the execution time as:

\\[ 
  D(e) + \frac{W(e)}{P} 
\\]

The rationale is that when \\( P \\) grows, this expression 
is asymptotically equivalent to the depth \\( D(e) \\). If \\( P \\)
is a relatively small number, then the expression has the same
complexity as the work \\( W(e) \\).

But how can we estimate the effect of adding more resources to a computation
(i.e. increasing \\( P \\))? Amdahl’s law comes to the resque. Lets assume 
that the computation \\( e \\) has 2 parts:

 * **Part 1** which is not parallelisable;
 * **Part 2** which is absolutely paralelisable - i.e. can run in as many
 independent threads that our hardware can have.

Lets denote by \\( f\\) the fraction of the serial execution time 
(i.e. \\( W(e)) \\)) that is dedicated to running **Part 1**. Then we can approximate
the execution time as:

\\[ 
  f \cdot W(e) + \frac{(1-f) \cdot W(e)}{P}
\\]

Which means that adding more resources will only speed-up the parallelizable
part of the computation. From this formula, we can deduct the original statment
of the law that the speed-up of adding more resources to a computation
\\( e \\) is:

\\[ 
  \frac{1}{ f + \frac{1-f}{P} } 
\\]


<div id='benchmarking'/>
## Benchmarking

The asymptotic complexity of a parallel algorithm gives us a rough estimate 
of the actual running time. In practice, there are many other factors
that affect the actual execution time.
The CPU speed, the number of processing units, the memory access speed, 
and the cache size and policies are just a few of hardware factors
affecting the execution time. Furthermore, the operating system (OS) 
scheduling of threads and processes and the execution environment 
behaviour in terms of garbage collection and just-in-time (JIT) 
compilation can have dramatic effect. Hence, benchmarking is an
invaluable tool for assessing performance.

Running the same benchmark execution multiple times can give us very 
different results. Hence, we should take aggregate descriptive
statistics of repeated runs (e.g. the mean or the interquartile range) 
as evidences of the actual performance. Furthermore, it makes
sense to eliminate outliers, as they are most likely a result of
extreme events (e.g. garbage collection or swapping). 

Another trick is to start measuring the application performance 
after it has been running for some time. This is called **warm-up period**.
The idea is, that during warm-up period, the JIT compilation have been
complete and the caches should be populated with commonly accessed data.
Therefore, measuring the application performance afterwards should
give more represantative results.

A popular library for benchmarking is [ScalaMeter](https://scalameter.github.io/).
Here's how to benchmark a piece of code:

```scala
// Import all from the ScalaMeter library
import org.scalameter._

// Runs a block and returns how long it took in millisecons
val time = measure {
  // The code we want to benchmark
  (0 until 100000000).map(math.pow(_, 5)).sum
}

println(s"The operation took: $time ms")
```

Now lets see how we can add warm-up to our measurment:

```scala
// Use the default ScalaMeter warmer
val time = withWarmer(new Warmer.Default).measure {
  // The code we want to benchmark
  (0 until 1000000).map(math.pow(_, 5)).sum
}
```

We can also config the warmenr parameters (e.g. min and max number
of runs):

```scala
// Custom config of the warmer
val time = config(Key.exec.minWarmupRuns -> 30,
  Key.exec.maxWarmupRuns -> 60).withWarmer(new Warmer.Default).measure {
  // The code we want to benchmark
  (0 until 1000000).map(math.pow(_, 5)).sum
}
```

ScalaMeter has many more functionalities. It can measure memory
consumption, ignore Garbage collection periods, count method 
invocation and so on. Consult the documentation to learn more :). 
