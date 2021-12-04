---
layout: post
title: New Features in Java 9 and Later Versions
date: 2021-12-04 01:22:09.000000000
type: post
published: true
status: publish
excerpt: This post summarises the new features in Java after version 9.
  It covers the most prominent features from
  a developer's perspective. It either omits or gives a high level overview
  of less commonly used features.
categories:
  - Java
tags:
  - Java, Java 9, Java 11, Java 17
---

# Table of Contents

- [Introduction](#introduction)
- [Set Up](#set-up)
- [JShell](#jshell)
- [Run Java files](#run-java-files)
- [Var - Local Variable Type Inference](#var-local-inference)
- [Unmodifiable Collections - New Factory and Utility Methods](#unmodifiable-collections)
- [Streams - New Methods](#streams-new-methods)
- [Optional - New Methods](#optional-new-methods)
- [String - New Methods](#string-new-methods)
- [Files - New Methods](#files-new-methods)
- [Switch Expressions](#switch-expressions)
- [Pattern Matching For InstanceOf](#pattern-matching-instanceof)
- [Text Blocks (Multiline Strings)](#text-blocks)
- [New Syntax for Interfaces](#interfaces)
- [Records](#records)
- [Sealed Classes](#sealed-classes)
- [New Garbage Collection Algorithms](#garbage-collection)
- [Java Modules (Overview)](#java-modules)
- [Resources](#resources)

<div id="introduction"/>

# Introduction

Java has been often referred to as a slow moving
language and platform. It took nearly 5 years for Java 7
to come out (released in 2011)
then 3 more for Java 8 (2014) and then 3 more for Java 9 (2017).

Since then, there's been a new release every 6 months!
There're many projects
[stuck with Java 8](https://www.marcobehler.com/guides/a-guide-to-java-versions-and-features#_why_are_companies_still_stuck_with_java_8).
It turns out developers are
having a hard time keeping up to speed and upgrading their code bases.

In this post, I'll summarise the new features in Java 9 and later versions.
At the time of writing the latest is Java 17 and I will keep this
article up to date as new versions are released.
If you need to catch up with Java 8, check out
[Java 8 in a Nutshell](/2014/03/21/java-8-in-a-nutshell/).

To keep this short, I'll only cover the most prominent features from
a developer's perspective. I will omit or just give a high level overview
of less commonly used features.

<div id="set-up" />

# Set Up

[SDKMAN](http://sdkman.io/) is a utility for installing Java and related tools. It's similar to
Node's [NVM](https://github.com/creationix/nvm) and Python's [PyEnv](https://github.com/yyuu/pyenv). SDKMAN can
install and switch between multiple versions of Java, Scala, Maven, Gradle, etc.

To install on Linux or Mac:

```bash
curl -s "https://get.sdkman.io" | bash
```

Then we can install Java 17:

```bash
# View all available Java distributions:
sdk list java

# Install the latest Java 17 Temurin distro
# (Temurin used to be called AdoptOpenJDK)
sdk install java 17.0.1-tem
```

To follow along, you can start a new Java project. In this example, I'll use Gradle:

```bash
# Check out what gradle versions are available
sdk list gradle

# Get the latest one (7.3 or later for Java 17):
sdk install gradle 7.3

# Make a project
mkdir javaplayground && cd javaplayground

# Set up an "application" project using the wizard
gradle init

# Make sure it runs:
./gradlew run
```

<div id="jshell" />

# JShell

Java 9 introduced a [REPL](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop), which is great for quick experimentation.

It has `tab` key autocompletion, allows you to look up JavaDoc, and displays use friendly errors. Lastly, it imports many packages by default (including `java.util.*`), so you won't have to do it yourself.

To start it, use the `jshell` command:

```bash
# Start the JShell the REPL
> jshell

# Check out which imports are included by default
jshell> /imports

# Define variables, print something, etc
jshell> String hello = "hello"
jshell> String world = "world"
jshell> System.out.println(hello + " " + world)

# No need to import List from java.util
jshell> List<String> list = List.of("a", "b")

# Tab auto completion - will list all System methods
jshell> System.<tab>

# Multiple <tabs> shows javadoc and scrolls through it
jshell> System<tab><tab>

# Quit
jshell> /exit
```

<div id="run-java-files" />
# Run Java files

As of Java 10, you can run java source files (with main methods)
without compiling them:

```bash
java ./app/src/main/java/com/nikgrozev/App.java
```

<div id="var-local-inference" />

# Var - Local Variable Type Inference

As of Java 10, local variables can be defined with the `var`
without explicitly specifying their type. The compiler
automatically infers the type of such variables:

```java
// Before, we had to write List<String> list = ...
var list = List.of("1", "2", "3");
for (var e : list) {
    System.out.println(e);
}
```

<div id="unmodifiable-collections" />

# Unmodifiable Collections - New Factory and Utility Methods

The `List`, `Set`, and `Map` interfaces
have a new factory method `of` to quickly instantiate
immutable collections.

Another new collection factory method is `copyOf`, which
creates an immutable copy of its argument.

Finally, new collectors have been introduced, which convert a
stream to an unmodifiable collections

```java
// New Helper factory methods
List<String> sampleList = List.of("a", "b", "c");
Set<String> sampleSet = Set.of("a", "b", "c");
Map<String, String> sampleMap = Map.of("a", "a-Value", "b", "b-Value");

// Immutable collections - will throw error on modification
// sampleList.set(0, "will be error");

// Prints out:
// class java.util.ImmutableCollections$ListN,
// class java.util.ImmutableCollections$SetN,
// class java.util.ImmutableCollections$MapN
System.out.printf("%s,\n%s,\n%s\n",
    sampleList.getClass(),
    sampleSet.getClass(),
    sampleMap.getClass());

// List, Set, Map have a new method "copyOf" to create an immutable copy
List<String> mutableList = new ArrayList<>(Arrays.asList("1", "2", "3"));
List<String> immutableList = List.copyOf(mutableList);

// prints java.util.ImmutableCollections$ListN
System.out.println(immutableList.getClass());

// New collectors - toUnmodifiableXXX convert a stream to immutable collection
List<String> immutableListCollected = mutableList.
    stream().
    collect(Collectors.toUnmodifiableList());
```

<div id="streams-new-methods" />

# Streams - New Methods

Java 8 introduced the `Stream.iterate` method, which creates
and infinite stream.
It has 2 parameters - a `seed` value and a generator function `f`.
The `iterate` method creates a stream, which dynamically generates the sequence of values:

`seed, f(seed), ..., f(f(...f(seed)...))`

For example the following, creates a stream of all integers, starting from 0 and incrementing by 1:

```java
Stream<Integer> infiniteN = Stream.iterate(0, i -> i + 1);
```

Java 9 introduced a new version of the `iterate` method,
which also takes a predicate. It dynamically generates values
until it reaches one that violates the predicate:

```java
// Stream.iterate - typically used to create ranges
Stream<Integer> sampleStream = Stream.iterate(0, i -> i < 10, i -> i + 1);

// Prints 0, 1, 2, 3, 4, 5, 6, 7, 8, 9
System.out.prinln(sampleStream.toArray());
```

Java 9 also introduced the `dropWhile` and `takeWhile` stream
methods, which allow you to skip or select the initial
values based on a predicate:

```java
// Stream 0 to 9
Stream<Integer> sampleStream = Stream.iterate(0, i -> i < 10, i -> i + 1);

Stream<Integer> selectedPart = sampleStream.
    dropWhile(i -> i < 2).
    takeWhile(i -> i < 5);

// Prints Range [2, 3, 4]
System.out.printf("Range %s\n", selectedPart.toList().toString());

```

<div id="optional-new-methods" />

# Optional - New Methods

Java 8 inroduced the `Optional` class as container which
either has a single value or none.
Recently, it got two new methods: `ifPresentOrElse` and `orElseThrow`:

```java
// Optional has a new method - ifPresentOrElse
Optional<Integer> sampleOption = Optional.of(123);
sampleOption.ifPresentOrElse(
    (v) -> System.out.println("Has value: " + v),
    () -> System.out.println("Has NO value"));

Optional<Object> opt = Optional.empty();
try {
    // same as opt.get(), but throws if missing
    var value = opt.orElseThrow();
} catch (NoSuchElementException e) {
    System.out.println("Option was empty");
}
```

<div id="string-new-methods" />

# String - New Methods

`String` got a few convenient methods:

```java
System.out.println("".isBlank()); // true
System.out.println(" ".isBlank()); // true
System.out.println(" \n ".isBlank()); // true
System.out.println(" s ".isBlank()); // false

// unicode strip
System.out.println(" x ".strip());
System.out.println(" x ".stripLeading());
System.out.println(" x ".stripTrailing());

// Prints " x  x  x "
System.out.println(" x ".repeat(3)); // unicode strip

// Make a stream of lines!
"\ntest\ntest2\n\nstest".
    lines().
    forEachOrdered((var s) -> System.out.println(">> " + s));

// Manage indentation:
var multiline = "  line1\n  line2";

// Adds 2 spaces at each line's start. Normalises new lines (\n)
System.out.println(multiline.indent(2));

// Removes up to 3 spaces on each line's start and normalises new
// lines symbol. Can be less if a line has fewer lead spaces
System.out.println(multiline.indent(-2));

// Normalises new lines (\n) - nothing else changes
System.out.println(multiline.indent(0));

// Removes the common indentation - e.g. if all lines have
// between 2 and 5 leading spaces stripIndent will remove 2.
System.out.println(multiline.stripIndent());
```

<div id="files-new-methods" />

# Files - New Methods

With the new `Files`, developers can read, write, compare files more easily:

```java
// Creating and reading text files with a single line!
Path path =
    Files.writeString(Files.createTempFile("test", ".txt"), "Demo");
System.out.println(path);
String s = Files.readString(path);
System.out.println(s); //Prints "Demo"

// "mismatch" compares two files efficiently - i.e. first by size,
// then by content. Returns -1 if they're equal
System.out.println(Files.mismatch(path, path)); // prints -1
```

<div id="switch-expressions" />

# Switch Expressions

This new syntax avoids the pitfalls of
[switch fall through](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/switch.html):

```java
var text = "A";
var index = switch (text) {
    case "A" -> 1;
    case "B" -> 2;
    default -> throw new IllegalArgumentException("Unknown letter");
};
System.out.println(index);
```

<div id="pattern-matching-instanceof" />

# Pattern Matching For InstanceOf

Type guards with `instanceOf` are common in Java code.
In older versions, you'd still need to explicitly cast within the
guarded code, but now there's a shortcut:

```java
Object o = "some text";

// In older versions we had to cast expicitly
if (o instanceof String) {
    // Need to cast, although we're type guarding
    String oAsString = (String) o;
    System.out.println(oAsString);
}

// New feature - type guard and cast together
if (o instanceof String oString) {
    System.out.println(oString);
}
```

<div id="text-blocks" />

# Text Blocks (Multiline Strings)

Text Blocks (a.k.a. multiline strings) surrounded by
triple double quotes are a godsend for everyone writing SQL:

```java
var sql = """
        SELECT * FROM
        MY_TABLE
        WHERE X > 1
        """;
```

The lines' common indentation and trailing spaces are stripped.
To add leading indentation either use the `indent` method
or move the closing quotes at the beginning of the line:

```java
var sqlExplicitlyIndented = """
    SELECT * FROM
    MY_TABLE
    """.indent(2);

var sqlOriginalIndentation = """
    SELECT * FROM
    MY_TABLE
"""; // <----- closing bracket is at the line's beginning
```

Each line's trailing spaces will be ignored.

<div id="interfaces" />

# New Syntax for Interfaces

Prior to Java 8, interfaces could only defined static constants
and abstract methods. Since Java 8, interfaces can also
define public overrideable methods with default implementation.
Java 9 allow interfaces to have private methods as well:

```java
interface IStudent {
    // Abstract method - all non-abstract subclasses must implement it
    public double getGPAGrade();

    // New In Java 8: Public static method
    public static double getMaxGPAGrade() {
        return 7;
    }

    // New in Java 8: Default method - no need to implement, but can override
    default double getPercentageGrade() {
        validateGPA(); // Call the private method
        return (getGPAGrade() / getMaxGPAGrade()) * 100;
    }

    // New in Java 9: private interface methods
    private void validateGPA() {
        validateGPAScore(this.getGPAGrade());
    }

    // New in Java 9: private static methods
    private static void validateGPAScore(double gpa){
        if (gpa < 0 || gpa > getMaxGPAGrade()) {
            throw new IllegalArgumentException("Invalid GPA");
        }
    }
}

class Student implements IStudent {
    double gpaGrade = 0;
    public Student(double gpaGrade) {
        this.gpaGrade = gpaGrade;
    }
    public double getGPAGrade() {
        return gpaGrade;
    }
}
```

<div id="records" />

# Records

Java is known for it's verbosity. Consider all the boilerplate needed for
a simple data-only class - get methods, `equals`, `hashCode`, `toString`, etc.
Developers often use their IDEs to auto generate all of that.

As of Java 17, there's a shortcut to creating _immutable data-only classes_ via the
`record` keyword. Records are final and can not be extended. They get  
automatic accessor methods, constructor, and the
aforementioned `equals`, `hashCode`, `toString`:

```java
// Can not be extended
record Name(String first, String last) {};

// Default constructor
var name = new Name("john", "doe");

// equals, hashCode, toString for free
System.out.println(name); // prints Name[first=john, last=doe]
System.out.println(name.equals(new Name("john", "doe")));

// default accessors
System.out.println(name.first() + " " + name.last());
```

Optionally, you can implement additional constructors and methods:

```java
record ComplexName(String first, String last) {
    public ComplexName(String fullName) {
        this(fullName.split("\s+")[0], fullName.split("\s+")[1]);
    }
    public String funnyPrint() {
        return ":) first=" + first() + ", last=" + last();
    }
};

// Default constructor
var complexName1 = new ComplexName("john", "doe");

// Custom constructor
var complexName2 = new ComplexName("john doe");

// Call the custom method
complexName1.funnyPrint();
```

Special care must be taken to ensure all record's member variables are
immutable. Otherwise, the record won't be really immutable.

```java
record BadRecord(List<String> list){};

// Member variable isn't immutable ...
var bad = new BadRecord(new ArrayList<>(List.of("1", "2", "3")));

// We can modify the record
bad.list().remove(0);
```

<div id="sealed-classes" />

# Sealed classes

Sealed classes are a new feature which allows you to control how a class hierarchy is extended.

Previously, a class could be either `final` or freely extendable.
What if you need to have your own class hierarchy, but want to prevent client programmers
(e.g. library users) extending it or part of it?

For example, you can create a class `ImmutableSet` and a subclass `ImmutableOrderedSet`.
You don't want client programmers extending `ImmutableSet` because they can make it mutable.
However, it can't be `final`, because `ImmutableOrderedSet` needs to inherit from it.

You can do so by making it a `sealed` class that `permits` only `ImmutableOrderedSet` to extend it.

```java
// Not final - can be extended by one class only (in this example)
sealed class ImmutableSet permits ImmutableOrderedSet { /*...*/ }
final class ImmutableOrderedSet extends ImmutableSet { /*...*/ }
```

Java requires that a permitted subclass must be marked as either:

- `final` - can't be extended;
- `sealed` - the subclass is itself sealed and permits limited extension;
- `non-sealed` - the subclass can be freely extended.

The following example demonstrates how to use sealed classes:

<figure>
  <img src="/images/blog/java-9-to-now/sealed-classes.png" alt="Sealed classes demonstration" >
  <figcaption>Example sealed class hierarchy.</figcaption>
</figure>

And here is a complete example:

```java
// A sealed class can only be extended from the classes it permits
sealed abstract class Loan permits Mortgage, CarLoan, PersonalLoan {
    protected long annualIncome;
    abstract boolean isApproved();
}

// A final sub class
final class Mortgage extends Loan {
    public long housePrice;
    public boolean isApproved() {
        return housePrice / (float)annualIncome < 10;
    }
}

// A sealed subclass - allows one more sub-sub-class
sealed class CarLoan extends Loan permits TaxiCarLoan {
    public long carValue;
    public long carMaintenancePerYear;
    public boolean isApproved() {
        return carValue / annualIncome < 2 &&
                carMaintenancePerYear < 0.1 * annualIncome;
    }
}

final class TaxiCarLoan extends CarLoan {
    public long taxiRegoFee;
    public boolean isApproved() {
        return super.isApproved() && taxiRegoFee < 1000;
    }
}

// non-sealed class - can be freely extended
non-sealed class PersonalLoan extends Loan {
    public long loanValue;
    public boolean isApproved() {
        return loanValue / (float) annualIncome < 0.1;
    }
}

// External code can extend PersonalLoan without restrictions
class LoanSharkProduct extends PersonalLoan {
    public boolean isApproved() {
        return true;
    }
}
```

<div id="garbage-collection" />

# New Garbage Collection Algorithms

Java is one of the few platforms to allow developers fine grained
control over which Garbage Collection (GC) algorithm is used and its
parameters. In recent years, Java changed the default GC algorithm to
[G1](https://www.redhat.com/en/blog/part-1-introduction-g1-garbage-collector)
and introduced the [ZGC](https://hub.packtpub.com/getting-started-with-z-garbage-collectorzgc-in-java-11-tutorial/)
and [Shenandoah](https://wiki.openjdk.java.net/display/shenandoah/Main)
algorithms for low latency applications. In this section, I'll cover
the main GC algorithms (old and new) and how they compare.

**Serial GC** is the simplest GC algorithm, tailored
for single threaded applications.
It works on a single thread and stops the app while running.
It uses a mark-compact collection method, which clears the heap and
then compacts it into a contiguous memory space.

<!-- Hence, allocating new memory blocks is faster. -->

**Parallel GC** was the default before Java 9. It's a _generational_ algorithm,
meaning that it divides the heap into sections called "generations".
The idea is that new objects are more likely to require collection
than older objects. Hence, younger generations are cleaned more often
and the surviving objects are gradually moved to the old generations.
Although the algorithm is called Parallel, it doesn't run
in parallel with the application. It pauses the app, just like **Serial GC**.
However, the garbage collection itself should be much faster since it uses
multiple threads.

The **CMS (Concurrent Mark Sweep)** algorithm is now deprecated. It's similar
to the **Parallel GC**, but it attempts not to pause the application. It consumes
more CPU than other algorithms as it needs to keep track of which heap areas
the application threads are using. It may still pause the application threads
while cleaning the oldest heap generations.

As aforementioned **Garbage-first (G1)** is now the default. It divides the
heap into equal sized regions - usually a few megabytes each.
The regions are then classified into a few generations (youngest to oldest).
New objects are allocated in the youngest generation regions and
over time they're either garbage collected or moved to older regions.
G1 uses multiple threads to scan the regions and collects from the regions
with the most dead objects. Because it works on selected regions, G1
minimises the amount of time the app is paused.

The **Epsilon Garbage Collector** doesn't actually collect any garbage.
It's used for low latency apps where developers are well aware
of the memory consumption. It's also useful for short lived jobs
where the upper limit of allocated memory is small.

The **Z Garbage Collector (ZGC)** runs an analysis of the heap known as
marking. For each object reference ZGC stores the object state
(e.g. ready for collection, or being relocated) in
unused bits of the reference itself - a technique known as
colouring.

ZGC uses `load barriers` which are callbacks executed by the JVM every time
a thread loads a reference. This allows ZGC to inspect the reference's state
flags, and if it's to be relocated, it can return a different
reference to the calling thread.
Hence ZGC can move around objects and compact the heap in parallel with
the app without stopping it and using additional data structures.

In general, ZGC pauses the app very rarely (e.g. scanning for root references)
and runs in parallel with it.

The **Shenandoah GC** algorithm also runs in parallel with the app,
compacts the heap in real time, and minimises app pauses.
Unlike ZGC, it uses additional data
structures to keep track of objects' state and hence its
memory and CPU footprint are much higher.

<div id="java-modules" />
# Java Modules (Overview)

Java modules are new in Java 9. A `module` is a collection of java packages
and can be distributed as a jar file. A module declares its dependencies
on other modules and exposes some of its packages via a manifest file.
Only the exposed packages can be used by other modules.

Most importantly, the JDK itself has been broken into modules. Hence, you can
cherry pick which part of the JDK you need to package with your app.
This leads to smaller and more secure executable and runtime
(e.g. Docker images).

Unfortunately, some popular libraries haven't migrated to modules yet.
Also, there is no interoperability with [OSGI](https://www.osgi.org/)
which was the de facto standard for app modularisation before Java 9.

For more in-depth discussion of the syntax of modules, please
check out this overview of [Java Modules](http://tutorials.jenkov.com/java/modules.html).

<div id="resources" />

# Resources

- [Java 9 - 17 Features with Examples](https://www.journaldev.com/13121/java-9-features-with-examples)
- [Java Versions and Features](https://www.marcobehler.com/guides/a-guide-to-java-versions-and-features)
- [Seven Types of Java Garbage Collectors](https://medium.com/@hasithalgamge/seven-types-of-java-garbage-collectors-6297a1418e82)
- [JVM Garbage Collectors](https://www.baeldung.com/jvm-garbage-collectors)
- [Getting started with Z Garbage Collector (ZGC) in Java 11](https://hub.packtpub.com/getting-started-with-z-garbage-collectorzgc-in-java-11-tutorial/)
- [Garbage Collection in Java – What is GC and How it Works in the JVM](https://www.freecodecamp.org/news/garbage-collection-in-java-what-is-gc-and-how-it-works-in-the-jvm/)
- [Java Modules](http://tutorials.jenkov.com/java/modules.html)
- [Fight ambiguity and improve your code with Java 17’s sealed classes](https://blogs.oracle.com/javamagazine/post/java-sealed-classes-fight-ambiguity)
