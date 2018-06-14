---
layout: post
title: Summary of the Oracle JDK8 MOOC
date: 2015-07-30 19:49:35.000000000
type: post
published: true
status: publish
excerpt: 
    Oracle has recently released a 3-week 
    JDK 8 Massive Open and Online Course called "Lambdas and Streams”, 
    which discusses the new JDK8 functional features in much more details. It's a great course and if 
    you're interested in Java and new JDK8 it can help you get started. This article summarises the course 
    and my previous post and can be used as a quick ref-card ...
categories: 
- Java
tags:
- Java
- Jdk
- MOOC
- Oracle
---

# Introduction

In a [previous post](/2014/03/21/java-8-in-a-nutshell/), 
I summarised the new functional features of Java 8. Oracle has recently released a 3-week 
[JDK 8 Massive Open and Online Course called "Lambdas and Streams"](https://blogs.oracle.com/thejavatutorials/entry/jdk_8_massive_open_and), 
which discusses the new JDK8 functional features in much more details. It's a great course and if 
you're interested in Java and new JDK8 it can help you get started. This article summarises the course 
and my previous post and can be used as a quick ref-card.




# Lecture 1: Lambda Expressions

## Lambda Syntax

To understand lambdas, first we need to understand functional interfaces. In Java 8, an interface is called functional, 
if it has only one abstract method. In previous Java versions, all interface methods were abstract. 
In Java 8 interfaces can have default and static methods as well. In order for an interface to be function it 
should have exactly one abstract method regardless if any default or static methods are defined. 
It is recommended to annotate functional interfaces with the 
[@FunctionalInterface](https://docs.oracle.com/javase/8/docs/api/java/lang/FunctionalInterface.html) 
annotation so that the compiler can check if they meet the requirement.

In Java, a lambda function can be thought of as a syntactic short-cut for defining anonymous instances of 
functional interfaces. Lambda expressions are defined with the following pattern:

```
(param1, param2, ... , paramN) -> expression | block of code
```

If the parameter list has only one parameter, the brackets can be omitted. Note that there are no types in the 
lambda definitions, as types are automatically inferred from the functional interface definition. 
Types can be defined explicitly, but this is not required. The body of the lambda expressions can be 
either a single expression (whose evaluation is the result of the function call) or a block of code, 
similar to a method definition. The following examples demonstrate functionally equivalent definitions:

```java
Comparator <Integer> cmp1 = (x, y) -> x.compareTo(y);   // Expression body. Types NOT specified
Comparator <Integer> cmp2 = (Integer x, Integer y) -> { // Block of code body. Types specified
    return x.compareTo(y);
};
Comparator <Integer> cmp3 = new Comparator<Integer>() { // Java 7 style
    public int compare(Integer x, Integer y) {
        return x.compareTo(y);
    }
};

Runnable r1 = () -> System.out.println("Test");         // Expression body
Runnable r2 = () -> { System.out.println("Test"); };    // Block of code body
Runnable r3 = new Runnable() {                          // Java 7 style
    public void run() {
        System.out.println("Test");
    }
};
```

Alike anonymous classes, lambdas can use the variables of their environment. 
When using local variables in lambdas, they must be **effectively final**, 
which means they are not assigned to from anywhere, regardless if they are actually marked as final.

## Method References

Lambdas are a neat way to implement functional interfaces, but often all they do is 
call an existing method. To make things simpler Java 8 introduces method references. 
Method references are a shorthand for implementing functional interfaces by calling already defined methods.

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
        <td>(param1, ... paramN) -&gt; Class.staticMethod(param1, ... paramN)</td>
    </tr>
    <tr>
        <td>Specific instance's method</td>
        <td>var::instanceMethod</td>
        <td>(param1, ... paramN) -&gt; var.instanceMethod(param1, ... paramN)</td>
    </tr>
    <tr>
        <td>Instance method</td>
        <td>Class::instanceMethod</td>
        <td>(var, param1, ... paramN) -&gt; var.instanceMethod(param1, ... paramN)</td>
    </tr>
    <tr>
        <td>Constructor</td>
        <td>Class::new</td>
        <td>(param1, ... paramN) -&gt; new Class(param1, ... paramN)</td>
    </tr>
    </tbody>
</table>

The following examples demonstrate equivalent definitions with method references and lambdas:

```java
Predicate <String> p1 = Boolean::getBoolean;         // Static method reference
Predicate <String> p2 = s -> Boolean.getBoolean(s);  // Equivalent lambda

String var = "TestEquality";
Predicate <String> p1 = var::equals;         // Specific instance's method reference
Predicate <String> p2 = s -> var.equals(s);  // Equivalent lambda

Predicate <String> p1 = String::isEmpty;     // Instance method reference
Predicate <String> p2 = s -> s.isEmpty();    // Equivalent lambda

Predicate <String> p1 = Boolean::new;           // Constructor reference
Predicate <String> p2 = s -> new Boolean(s);    // Equivalent lambda
```

## Functional Interfaces in the standard library

The standard library has always had a bunch of functional interfaces – Runnable, Callable, Comparator etc. 
Java 8 introduces a whole new package of functional interfaces called 
[java.util.function](https://docs.oracle.com/javase/8/docs/api/java/util/function/package-summary.html). 
In Java there are two kinds of types – referential and primitives. 
Java generics can only be used with 
referential types – e.g. `List<int>` is invalid. 
Thus, the `java.util.function` package contains 
multiple versions of each interface – a generic version for referential types, and specialised versions 
for the primitives. For example we've got 
[Consumer\<T\>](https://docs.oracle.com/javase/8/docs/api/java/util/function/Consumer.html) and 
[IntConsumer](https://docs.oracle.com/javase/8/docs/api/java/util/function/IntConsumer.html). 
In the rest of the article we'll look at the generic interface only.

The main functional interfaces in the package are:

*   [Consumer\<T\>](https://docs.oracle.com/javase/8/docs/api/java/util/function/Consumer.html) - takes an argument of type T returns void;
*   [BiConsumer\<T,U\>](https://docs.oracle.com/javase/8/docs/api/java/util/function/BiConsumer.html) - a consumer with 2 arguments;
*   [Supplier\<T\>](https://docs.oracle.com/javase/8/docs/api/java/util/function/Supplier.html) - takes no argument and returns T;
*   [Function\<T,R\>](https://docs.oracle.com/javase/8/docs/api/java/util/function/Function.html) - takes an argument of type T and returns R;
*   [BiFunction\<T,U,R\>](https://docs.oracle.com/javase/8/docs/api/java/util/function/BiFunction.html) - a function of two arguments;
*   [UnaryOperator\<T\>](https://docs.oracle.com/javase/8/docs/api/java/util/function/UnaryOperator.html) - shorthand for Function<T,T>;
*   [BinaryOperator\<T\>](https://docs.oracle.com/javase/8/docs/api/java/util/function/BinaryOperator.html) - shorthand for BiFunction<T,T>;
*   [Predicate\<T\>](https://docs.oracle.com/javase/8/docs/api/java/util/function/Predicate.html) - given an argument of type T returns a boolean;
*   [BiPredicate\<T,U\>](https://docs.oracle.com/javase/8/docs/api/java/util/function/BiPredicate.html) - given arguments of types T and U, returns a boolean.

The consumer interfaces in the above list have a default method [andThen](https://docs.oracle.com/javase/8/docs/api/java/util/function/Consumer.html#andThen-java.util.function.Consumer-), which takes as an argument another consumer. The result is a new consumer instance, which runs the two consumers one after the other, as in the example:

```java
//Define two consumers
Consumer <String> helloConsumer = name -> System.out.println("Hello, " + name);
Consumer <String> byeConsumer = name -> System.out.println("Bye, " + name);

//Create and call a composite consumer which “chains” the previous two
Consumer <String> helloByeConsumer = helloConsumer.andThen(byeConsumer);

// Prints "Hello, Guest\nBye, Guest"
helloByeConsumer.accept("Guest");
```

The `andThen` default method of the `Function` interfaces works similarly – it chains the function invocation as 
in `f(g(x))`. The `compose` method does the same, but swaps the functions – 
i.e. `g(f(x))` instead of `f(g(x))`. The following example demonstrates this:

```java
UnaryOperator <Integer> plus1 = x -> x + 1;
UnaryOperator <Integer> mult2 = x -> x * 2;

//Create a new function with andThen
Function <Integer, Integer> plus1Mult2 = plus1.andThen(mult2);
System.out.println(plus1Mult2.apply(1));// Prints 4

//Create a new function with compose
Function <Integer, Integer> mult2Plus1 = plus1.compose(mult2);
System.out.println(mult2Plus1.apply(1)); // Prints 3
```

**Note**: in the above example you may wish to use 
[IntUnaryOperator](https://docs.oracle.com/javase/8/docs/api/java/util/function/IntUnaryOperator.html) instead of UnaryOperator<Integer> to avoid excessive boxing/unboxing.

Similarly, the predicate interfaces have default methods `and`, `or`, and `negate`, which 
can be used to create new predicates with combined logic.

## New Methods in JDK 8

The Java 8 standard collections library introduces several new methods which use functional interfaces:

*   [Iterable.forEach(Consumer c)](https://docs.oracle.com/javase/8/docs/api/java/lang/Iterable.html#forEach-java.util.function.Consumer-) – iterates and applies the consumer to each element;

*   [Collection.removeIf(Predicate p)](https://docs.oracle.com/javase/8/docs/api/java/util/Collection.html#removeIf-java.util.function.Predicate-) – removes all elements for which the predicate returns true;

*   [List.replaceAll(UnaryOperator o)](https://docs.oracle.com/javase/8/docs/api/java/util/List.html#replaceAll-java.util.function.UnaryOperator-) – replaces all elements with the result of the argument operator;

*   [List.sort(Comparator c)](https://docs.oracle.com/javase/8/docs/api/java/util/List.html#sort-java.util.Comparator-) - replaces [Collections.sort](https://docs.oracle.com/javase/8/docs/api/java/util/Collections.html#sort-java.util.List-).

Also, many methods of the standard [Logger](https://docs.oracle.com/javase/8/docs/api/java/util/logging/Logger.html) 
have been overloaded to take as an argument a _Supplier\<String\>_ instance, instead of string. For example, 
when the [Logger.fine(Supplier\<String\>)](https://docs.oracle.com/javase/8/docs/api/java/util/logging/Logger.html#fine-java.util.function.Supplier-) 
method is invoked it will only call the provided supplier if the logging level is below or equal to `Fine`. Otherwise, the supplier will not be called leading to fewer operations like string concatenation and formatting.



# Lecture 2: The Streams API

## Streams

JDK 8 defines a stream as a sequence of elements. Streams of referential elements inherit from the 
[Stream\<T\>](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html) interface, and 
there are specialised interfaces for streams of primitives. Once again we'll only consider the generic types, 
as the primitive streams offer analogous functionalities.

Streams are usually used in a pipeline of operations, where each operation produces or transforms a stream 
until a result is derived. Hence, each stream operation can be can be classified as:

*   Stream source – generates a stream;

*   Inermediate operation – processes a stream;

*   Terminal operation – given a stream, yields the final result.


<figure>
  <img src="/images/blog/Summary of the Oracle JDK8 MOOC/streamoverview.png" alt="Working with streams." >
  <figcaption>Working with streams.</figcaption>
</figure>


Sources of streams can be collections and arrays through the following methods and their overloads:

*   [Collection.stream()](https://docs.oracle.com/javase/8/docs/api/java/util/Collection.html#stream--) - converts the collection to a stream;

*   [Collection.parallelStream()](https://docs.oracle.com/javase/8/docs/api/java/util/Collection.html#parallelStream--) - converts to a stream, which can be subjected to parallel processing;

*   [Arrays.stream(](https://docs.oracle.com/javase/8/docs/api/java/util/Arrays.html#stream-T:A-)[T[]](https://docs.oracle.com/javase/8/docs/api/java/util/Arrays.html#stream-T:A-)[)](https://docs.oracle.com/javase/8/docs/api/java/util/Arrays.html#stream-T:A-) - utility method converting the provided array to a stream;

Other typical sources of streams are random number generators 
(see [Random](https://docs.oracle.com/javase/8/docs/api/java/util/Random.html)), 
lists of file paths (see <a>Files</a>) and list of lines in a text 
(see [BufferedReader](https://docs.oracle.com/javase/8/docs/api/java/io/BufferedReader.html)).

The generic [Stream\<T\>](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html) interface and 
its primitive counterparts provide several utility source functions for creating or transforming streams:

*   [Stream.concat(Stream, Stream)](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#concat-java.util.stream.Stream-java.util.stream.Stream-) - returns the concatenation of the two streams.

*   [Stream.empty()](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#empty--) - creates an empty stream;

*   [Stream.of(T... values)](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#of-) - a new stream with the specified values;

*   [IntStream.range(int, int)](https://docs.oracle.com/javase/8/docs/api/java/util/stream/IntStream.html#of-), [Int](https://docs.oracle.com/javase/8/docs/api/java/util/stream/IntStream.html#rangeClosed-int-int-)[Stream.rangeClosed(int, int)](https://docs.oracle.com/javase/8/docs/api/java/util/stream/IntStream.html#rangeClosed-int-int-) – a new integer stream representing a range.


The intermediate operations are usually implemented as methods of the 
[Stream\<T\>](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html) interface and its 
primitive counterparts. The most important are:

*   [Stream.distinct()](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#distinct--) - returns a stream with the unique elements.

*   [Stream.filter(Predicate p)](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#filter-java.util.function.Predicate-) - returns a stream with the elements for which the predicate returns true.

*   [Stream.map(Function f)](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#map-java.util.function.Function-) - returns a stream with the result of the function application on each element.

*   [Stream.flatMap(Function f)](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#flatMap-java.util.function.Function-) – if the return type of the function is another stream, then `map(f)` will return a stream of streams, which may not be convenient. The `flatMap` method resolves this problem by flattening/combining the results into a single result stream.

*   [Stream.](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#sorted-java.util.Comparator-)[sorted](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#sorted-java.util.Comparator-)[(](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#sorted-java.util.Comparator-)[Comparator c](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#sorted-java.util.Comparator-)[)](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#sorted-java.util.Comparator-) – returns a sorted stream.

*   [Stream.](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#skip-long-)[skip](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#skip-long-)[(](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#skip-long-)[long n](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#skip-long-)[)](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#skip-long-) – returns a new stream, which does not contain the first n elements.

*   [Stream.](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#limit-long-)[limit](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#limit-long-)[(](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#limit-long-)[long n](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#limit-long-)[)](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#limit-long-) – returns a new stream, which contains the first n elements.

The methods [mapToInt](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#mapToInt-java.util.function.ToIntFunction-), [mapToDouble](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#mapToDouble-java.util.function.ToDoubleFunction-), and [mapToLong](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#mapToLong-java.util.function.ToLongFunction-) are handy when converting/mapping a stream of objects to a stream of primitives.


The terminal operations are special, as they cause the pipeline to be evaluated. 
Prior a terminal operation, the stream operations are only “accumulated” or delayed. 
This allows the terminal operation to execute effectively (e.g. in parallel) on the aggregated stream.

Most often you would probably use one of the following terminal operations:

*   [Stream.collect(Collector c)](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#collect-java.util.stream.Collector-) - collects all elements - e.g. in a list or concatenates them in a string.

*   [Stream.toArray()](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#toArray--) - converts the stream to array.

*   [Stream.reduce(BinaryOperator accumulator)](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#reduce-U-java.util.function.BiFunction-java.util.function.BinaryOperator-) – consequently applies the accummulator over all elements until it yields a single value. Example: multiply all integers.

Other frequently used terminal methods are `count`, `max`, and `min,` which return 
[Optional](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html) instances.

## Optionals

In Java 8, [Optional\<T\>](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html)
is a container which holds a reference to an object or `null`. It is used to avoid excessive null checks throughout the code. It can also be thought of as a stream which has either 0 or 1 elements. 
In that sense, it is a generalisation of the rule that you should prefer to return empty 
collections and arrays from methods, rather than null.

The count, max and min methods from the previous section all return optionals, whose 
embedded value is null if the stream is empty.

Instead of doing a `null` check, you can use the `ifPresent(Consumer c)` 
method to run code if the optional contains a value:

```java
Optional <String> opt = …

// Will print only if opt refers to a non-null value
opt.ifPresent(System.out::println);
```

You can also use the `filter` and `map` methods, as you would do with streams.



# Lecture 3: More Advanced Lambdas and Stream Concepts

The [Stream.reduce(BinaryOperator)](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#reduce-java.util.function.BinaryOperator-) 
method is a terminal operation which aggregates the stream to a single value. 
The provided operator is applied to `null` and the first element, then its result and the 
second element are fed in the operator and so on … until a single value is achieved. There is no guarantee 
as to the actual sequence of execution, but the result is always as if the aforementioned procedure is executed. 
This can be used to implement things like computing the min, max or sum of all elements. In fact, streams 
already have predefined short-cut methods for these functionalities. The reduction mechanism is a 
generalisation of this concept, and the reduce method has overloads providing additional parameters.

Given an infinite stream, the 
[Stream.findFirst](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#findFirst--) and 
[Stream.findAny](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#findAny--) methods are 
typically used to terminate it and return a value matching the specified predicate wrapped in an Optional. 
The `findFirst` method returns the first match, while `findAny` can return any match thus allowing 
for parallelism behind the scenes.

```java
// Find a random positive and even integer
int r = Random.ints().findFirst(i -> i > 0 && i % 2 == 0);

int[] a = new int[] {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
// Can return either 2, 4, 6, 8, or 10
int i = Arrays.stream(a).findAny(i -> i % 2 == 0);
```


Another terminal operation is 
[Stream.collect(Collector)](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#collect-java.util.stream.Collector-) 
which converts the stream to a container – usually this is a collection or a string. 
The collector parameter defines how the stream elements will be converted to the resulting container. 
The [Collectors](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collectors.html) class defines 
utility methods for instantiating and combining common collectors.

```java
Stream <Integer> s = Arrays.stream (new Integer[] {1, 2, 3, 4});

// To list and set
List <Integer> lst = s.collect(Collectors.toList());
Set <Integer> set = s.collect(Collectors.toSet());

// Map of same keys and values
Map <Integer, Integer> map = s.collect(Collectors.map(Function.identity(), Function.identity()));

// A string of all values with a separator
String toString = s.collect(Collectors.joining(","));
```

Streams can be serial or parallel. All operations on a sequential stream are run in sequence on a single thread. 
Operations on a parallel streams are run in a thread pool using the 
[Fork Join framework](https://docs.oracle.com/javase/tutorial/essential/concurrency/forkjoin.html). 
The size of the thread pool is determined by the number of available processors, but could be overridden by setting a system property.

Parallel streams can be created directly from collections through the `parallelStream` method. 
Every stream can be made parallel or sequential with the respective stream methods.