---
layout: post
title: Java 8 in a Nutshell
date: 2014-03-21 22:13:36.000000000 +11:00
type: post
published: true
status: publish
excerpt: 
    Oracle has officially released JDK 8, featuring long-awaited language features like 
    lambdas and a new Date-Time API. This post gives a brief overview 
    of the new functional programming features in Java ...
categories:
- Miscellaneous
- blog
tags:
- Java
- Jdk
- Lambda
author:
  login: nikolaygrozev
  email: nikolay.grozev@gmail.com
  display_name: nikolaygrozev
  first_name: 'Nikolay'
  last_name: 'Grozev'
---

# Introduction and Installation

Oracle has [officially released JDK 8](https://blogs.oracle.com/thejavatutorials/entry/jdk_8_is_released), 
featuring long-awaited language features like lambdas and a new Date-Time API. This post gives a brief overview 
of the new functional programming features in Java.

The latest JDK 8 can be downloaded and installed from 
[Oracle's website](http://www.oracle.com/technetwork/java/javase/downloads/index.html), or if you are running 
Ubuntu you can [use an additional PPA repository](http://www.webupd8.org/2014/03/how-to-install-oracle-java-8-in-debian.html) as follows:

```bash
sudo su -
echo "deb http://ppa.launchpad.net/webupd8team/java/ubuntu trusty main" | tee /etc/apt/sources.list.d/webupd8team-java.list
echo "deb-src http://ppa.launchpad.net/webupd8team/java/ubuntu trusty main" | tee -a /etc/apt/sources.list.d/webupd8team-java.list
apt-key adv --keyserver keyserver.ubuntu.com --recv-keys EEA14886
exit

sudo apt-get update
sudo apt-get install oracle-java8-installer
sudo update-java-alternatives -s java-8-oracle
sudo apt-get install oracle-java8-set-default
```

# Enhanced interfaces

A major problem of Java interfaces is extensibility. If you change an interface, all implementing 
classes must change as well. If this interface is a part of a public library for example, thousands of 
implementations (some of which in external code bases) may have to change. As of Java 8, interfaces can have 
default methods. Unlike standard abstract methods, a default method has an implementation and underlying classes 
do not need to implement it. Hence, default methods can be added later on, without having to modify or recompile existing classes.

Another change in Java 8 is that interfaces can have static methods just like classes. 
The following example illustrates these features:

```java
@FunctionalInterface // Will cause a complication error if a second abstract method is added
public interface Student {
    // Abstract method - all non-abstract subclasses need to implement it
    double getGPAGrade();

    // Static method
    static double getMaxGPAGrade() {
        return 7;
    }

    // Default method - no need to implement. Subclasses can still decide to override it.
    default double getPercentageGrade() {
        return (getGPAGrade() / getMaxGPAGrade()) * 100;
    }
}
```

An interface with a single abstract method (i.e. which is not static or default) is called a **functional interface**, 
since it is essentially an abstract definition of a function. Standard examples are 
[Runnable](http://download.java.net/jdk8/docs/api/java/lang/Runnable.html), 
[Callable](http://download.java.net/lambda/b78/docs/api/java/util/concurrent/Callable.html), 
[ActionListener](http://download.java.net/jdk8/docs/api/java/awt/event/ActionListener.html), and 
[Comparator](http://download.java.net/jdk8/docs/api/java/util/Comparator.html). 
The new package [java.util.function](http://download.java.net/jdk8/docs/api/java/util/function/package-summary.html) 
introduces more standard functional interfaces like 
[Predicate](http://download.java.net/jdk8/docs/api/java/util/function/Predicate.html), 
[Function](http://download.java.net/jdk8/docs/api/java/util/function/Function.html), and 
[Consumer](http://download.java.net/jdk8/docs/api/java/util/function/Consumer.html).

Java 8 introduces an informative annotation 
[@FunctionalInterface](http://download.java.net/jdk8/docs/api/java/lang/FunctionalInterface.html) 
which can be used to annotate interfaces. If an annotated interface is not a functional interface a 
compilation error is raised.


# Lambdas

Lambdas are not a new concept and should be straightforward to understand for anyone exposed to some sort of 
functional programming. If you have no experience with programming languages with functional features, you can 
think of lambda expressions as a syntax shorthand for anonymous subclasses of functional interfaces. 
The syntax for a lambda expression is:

```
(param1, param2, ... , paramN) -> expression | block of code 
```

If the parameter list has only one parameter the brackets can be omitted. Note that there are no types in the 
lambda definitions, as types are automatically inferred from the functional interface definition. Types can be 
defined explicitly, but this is not required. The body of the lambda expressions can be either a single expression 
(whose evaluation is the result of the function call) or a block of code, similar to a method definition.

The following examples demonstrate functionally equivalent definitions:

```java
Student s1 = () -> 3;               // Expression body
Student s2 = () -> {return 3;};     // Block of code body
Student s3 = new Student() {        // Java 7 style
    public double getGPAGrade() {
        return 3;
    }
};
```

```java
Comparator <Integer> cmp1 = (x, y) -> x.compareTo(y);   // Expression body. Types NOT specified
Comparator <Integer> cmp2 = (Integer x, Integer y) -> { // Block of code body. Types specified
    return x.compareTo(y);
};
Comparator <Integer> cmp3 = new Comparator<Integer> () { // Java 7 style
    public int compare(Integer x, Integer y) {
        return x.compareTo(y);
    }
};
```

```java
Runnable r1 = () -> System.out.println("Test");         // Expression body
Runnable r2 = () -> { System.out.println("Test"); };    // Block of code body
Runnable r3 = new Runnable() {                          // Java 7 style
    public void run() {
        System.out.println("Test");
    }
};
```


# Method References

Lambdas are a neat way to implement functional interfaces, but often all they do is call an existing method. 
To make things simpler Java 8 introduces method references. Method references are a shorthand for implementing 
functional interfaces by calling already defined methods.

There are four types of method references:

<table>
    <thead>
        <tr>
            <th>Reference to</th>
            <th>Syntax</th>
            <th>Lambda Equivalent</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Static method</td>
            <td>Class::staticMethod</td>
            <td>(param1, ... paramN) -> Class.staticMethod(param1, ... paramN)</td>
        </tr>
        <tr>
            <td>Specific instance's method</td>
            <td>var::instanceMethod</td>
            <td>(param1, ... paramN) -> var.instanceMethod(param1, ... paramN)</td>
        </tr>
        <tr>
            <td>Instance method</td>
            <td>Class::instanceMethod</td>
            <td>(var, param1, ... paramN) -> var.instanceMethod(param1, ... paramN)</td>
        </tr>
        <tr>
            <td>Constructor</td>
            <td>Class::new</td>
            <td>(param1, ... paramN) -> new Class(param1, ... paramN)</td>
        </tr>
    </tbody>
</table>

The following examples demonstrate equivalent definitions with method references and lambdas:


```java
Predicate <String> p1 = Boolean::getBoolean;         // Static method reference
Predicate <String> p2 = s -> Boolean.getBoolean(s);  // Equivalent lambda
```

```java
String var = "TestEquality";
Predicate <String> p1 = var::equals;         // Specific instance's method reference
Predicate <String> p2 = s -> var.equals(s);  // Equivalent lambda
```

```java
Predicate <String> p1 = String::isEmpty;     // Instance method reference
Predicate <String> p2 = s -> s.isEmpty();    // Equivalent lambda
```

```java
Predicate <String> p1 = Boolean::new;           // Constructor reference
Predicate <String> p2 = s -> new Boolean(s);    // Equivalent lambda
```

# Streams

So far we've seen that Java 8 has removed some clutter around implementing functional interfaces. 
Functional interfaces are especially useful for manipulating collections, and thus Java 8 introduces the 
[java.util.stream](http://download.java.net/jdk8/docs/api/java/util/stream/package-summary.html) package. 
The main interface in this package is [Stream](http://download.java.net/jdk8/docs/api/java/util/stream/Stream.html), 
which represents a sequence of elements supporting performing a sequence of actions - e.g. filtering, mapping, aggregation etc. 
Each of these standard stream methods takes as a parameter an instance of a functional interface.

Each standard collection in Java 8 can be converted to a stream through the **stream()** and **parallelStream()** methods. 
The difference between them, is that if possible **parallelStream()** returns a Stream instance, whose operations 
can be parallelised. Streams can also be converted to collections by the **collect()** method.

The main functionalities/methods of Stream are:

*   **filter** - given a predicate creates a new stream, whose elements match the predicate;
*   **map** - given a function f, creates a new stream, whose elements are the result of applying the function over this stream's element;
*   **reduce** - aggregates the stream elements into a single value by consecutively applying a provided binary function. 
The result is wrapped in an [Optional](http://download.java.net/jdk8/docs/api/java/util/Optional.html) instance;
*   **forEach**/**forEachOrdered** - executes a [Consumer](http://download.java.net/jdk8/docs/api/java/util/function/Consumer.html) 
instance's method for every element of the stream. The [Consumer](http://download.java.net/jdk8/docs/api/java/util/function/Consumer.html) 
functional interface represents an arbitrary block of code.

The following example demonstrates how lambdas and method references allow standard collection processing 
operations to be written in a single line, rather than using series of loops as in Java 7.


```java
List <Integer> numbers = Arrays.asList(1, 7, 15, 51, 16, 8);

// Filter all even numbers, then convert the stream to a list
List <Integer> evens = numbers.stream().filter(x -> x % 2 == 0).collect(Collectors.toList());

// Find the max even number
int maxEven = numbers.stream().filter(x -> x % 2 == 0).max(Integer::compare).get();

// Compute the sum of the squares of all even numbers in the list
int sumSquaresEven =
   numbers.stream().filter(x -> x % 2 == 0).map(x -> x * x).reduce(Integer::sum).get();

// Print all elements on a new line
numbers.stream().forEachOrdered(x -> System.out.println(x));
```

# References

*   [http://docs.oracle.com/javase/tutorial/java/javaOO/](http://docs.oracle.com/javase/tutorial/java/javaOO/)
*   [http://baddotrobot.com/blog/2014/02/18/method-references-in-java8/](http://baddotrobot.com/blog/2014/02/18/method-references-in-java8/)
*   [http://www.webupd8.org/2014/03/how-to-install-oracle-java-8-in-debian.html](http://www.webupd8.org/2014/03/how-to-install-oracle-java-8-in-debian.html)
*   [http://www.infoq.com/minibooks/emag-java-8](http://www.infoq.com/minibooks/emag-java-8)
