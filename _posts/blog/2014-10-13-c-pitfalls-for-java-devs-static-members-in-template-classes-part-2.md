---
layout: post
title: C# pitfalls for Java Devs - Static Members in Template Classes [Part 2]
date: 2014-10-13 01:47:03.000000000
type: post
published: true
status: publish
excerpt: 
    C# and Java generics differ significantly in terms of the underlying semantics, which can be quite confusing 
    for a Java programmer. In this post we'll look into it ...
categories:
- C#
tags:
- C#
- Generics
- Java
---

# Introduction

Previously I discussed one of the major differences between C# and Java - 
[method overriding](/2014/10/12/c-pitfalls-for-java-devs-overriding-part-1/). 
Because of it Java and C# code, which look almost identical, can have very different semantics. 
Similarly, C# and Java generics differ significantly in terms of the underlying semantics, which can 
be quite confusing for a Java programmer.

# Example

Let us consider the following seemingly analogous code snippets implemented in both languages.

C# code:

```csharp
public class Animal { }
public class Lion:Animal { }
public class Bear:Animal { }
public class Cage<T>;
{
    private static int instanceCount = 0;
    public Cage()
    {
        instanceCount++;
    }
    public void PrintInstanceCount()
    {
        System.Console.WriteLine(instanceCount);
    }
}
```

Java code:

```java
public class Animal { }
public class Lion extends Animal { }
public class Bear extends Animal { }
public class Cage<T>;
{
    private static int instanceCount = 0;
    public Cage()
    {
        instanceCount++;
    }
    public void PrintInstanceCount()
    {
        System.out.println(instanceCount);
    }
}
```

Now let us consider the following invocations:

```java
Cage <Animal> ca = new Cage<Animal>();
Cage <Lion> cl = new Cage<Lion>();
Cage <Bear> cb = new Cage<Bear>();
ca.PrintInstanceCount();
cl.PrintInstanceCount();
cb.PrintInstanceCount();
```

Many will be surprised to see that the results are different, as per the following table:

<table style="width:20em">
    <thead>
        <tr>
            <th>Command / Language</th>
            <th>C#</th>
            <th>Java</th>
        </tr>
    </thead>

    <tbody>
        <tr>
            <td>ca.PrintInstanceCount();</td>
            <td>1</td>
            <td>3</td>
        </tr>
        <tr>
            <td>cl.PrintInstanceCount();</td>
            <td>1</td>
            <td>3</td>
        </tr>
        <tr>
            <td>cb.PrintInstanceCount();</td>
            <td>1</td>
            <td>3</td>
        </tr>
    </tbody>
</table>

Java implements [type erasure](http://docs.oracle.com/javase/tutorial/java/generics/erasure.html), 
which means that type parameters are erased at compile time and at a runtime there is a single class 
for each template. Thus, in the previous example at runtime there is a single static member variable for 
all instances of Cage, independent of the type parameter.

Conversely, the C# compiler (i.e. the Just In Time compiler - JIT) ensures that all closed types 
(e.g. Cage<Lion>and Cage<Bear>) have their own static fields. Thus, to the programmer they seem as different types/classes.

# Consequences

As a result C# does not let you access the static members of a template class without specifying the template parameters:

```java
Cage.instanceCount = 5;         // Invalid in C#, Valid in Java
Cage<Lion>.instanceCount = 5;   // Valid in C#, Invalid in Java
Cage<Bear>.instanceCount = 5;   // Valid in C#, Invalid in Java
```

Also, C# does not let you have an entry point (i.e. a Main method) in a template class.
