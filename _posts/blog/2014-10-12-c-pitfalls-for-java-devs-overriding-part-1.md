---
layout: post
title: C# pitfalls for Java Devs - Overriding [Part 1]
date: 2014-10-12 22:07:10.000000000 +11:00
type: post
published: true
status: publish
excerpt: 
    Last week I finally had the time to learn a bit of C#, after a number of years of mostly Java coding. 
    Professional C# developers have described it to me as a "better Java" or "Java with syntactic sugar", 
    and have claimed that Java programmers should not have problems picking it up. 
    It is true that both languages have a lot in common - i.e. the C-like syntax and single inheritance. 
    However, there are some quite fundamental philosophical differences, which can be mind-boggling for a Java programmer ...
categories:
- C#
- blog
tags:
- C++
- Java
- C#
author:
  login: nikolaygrozev
  email: nikolay.grozev@gmail.com
  display_name: nikolaygrozev
  first_name: 'Nikolay'
  last_name: 'Grozev'
---

# Introduction

Last week I finally had the time to learn a bit of C#, after a number of years of mostly Java coding. 
Professional C# developers have described it to me as a "better Java" or "Java with syntactic sugar", 
and have claimed that Java programmers should not have problems picking it up. 
It is true that both languages have a lot in common - i.e. the C-like syntax and single inheritance. 
However, there are some quite fundamental philosophical differences, which can be mind-boggling for a Java programmer.

One of these is method overriding, which is fundamentally different in Java and C#.

# Example

Let us consider the following seemingly analogous code snippets implemented in both languages.

C# code:

```csharp
public class Base
{
    public virtual void F()
    {
        System.Console.WriteLine("Base::F()");
    }
    public void G()
    {
        System.Console.WriteLine("Base::G()");
    }
}
public class Der : Base
{
    public override void F()
    {
        System.Console.WriteLine("Der::F()");
    }
    public void G()
    {
        System.Console.WriteLine("Der::G()");
    }
}
```


Java code:

```java
public class Base
{
    public void F()
    {
        System.out.println("Base::F()");
    }
    public void G()
    {
        System.out.println("Base::G()");
    }
}
public class Der extends Base
{
    public void F()
    {
        System.out.println("Der::F()");
    }
    public void G()
    {
        System.out.println("Der::G()");
    }
}
```

Now let us consider the following invocations:

```java
Base d = new Der();
d.F();
d.G();
((Der)d).F();
((Der)d).G();
```

Many will be surprised to see that the results are different, as per the following table.

<table style="width:30em;">
    <thead>
        <tr>
            <th>Command / Language</th>
            <th>C#</th>
            <th>Java</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>d.F();</td>
            <td>Der::F()</td>
            <td>Der::F()</td>
        </tr>
        <tr>
            <td>d.G();</td>
            <td>Base::G()</td>
            <td>Der::F()</td>
        </tr>
        <tr>
            <td>((Der)d).F();</td>
            <td>Der::F()</td>
            <td>Der::F()</td>
        </tr>
        <tr>
            <td>((Der)d).G();</td>
            <td>Der::G()</td>
            <td>Der::G()</td>
        </tr>
    </tbody>
</table>

Java **always** invokes the method of the **actual class** of the object. 
In other words, the actual method is not known by the compiler - it is decided on at runtime. 
C# has this behaviour only if the method is declared as virtual (e.g. `F()`). 
The invocation of non-virtual methods (e.g. `G()`) is decided by the compiler, before the execution.

In other words, Java keeps it simple, by always deferring the decision until runtime. 
This is called *Dynamic binding*, and causes a certain performance penalty, because of 
the runtime object inspection. In contrast, C# does it the C++ way, and supports both dynamic 
and static binding. By default, methods are bound statically at compile time, unless they are marked as virtual.

# Syntax specifics

In Java, it is a good practice to annotate overriding methods with the `@Override` annotation. 
This signals the compiler to validate that the method indeed overrides properly. 
Even if the annotation is not specified, the behaviour remains the same.

In C#, the `override` keyword has a similar role - it tells the compiler that an 
overriding takes place. However, if the programmer does not specify the `override` keyword, 
the method can not be dynamically invoked from a base class reference! In other words, in the 
previous example if we had omitted the override keyword, the invocation of `d.F()` would 
call `Base::F()`, even though it is a virtual method and the actual instance type is `Der`.

In C# you can explicitly mark methods with the `new` keyword, to indicate that they are redefining, 
rather than overriding a base class method. Specifying `new`, is semantically the same as not specifying 
override (as in the previous paragraph). The `new` keyword just makes it clearer for the reader.

# A huge difference

Java's default dynamic binding behaviour can be implemented in C# by specifying all instance 
methods as `virtual`, and annotating all their redefinitions with `override`. 
The opposite is not possible. Java can't implement static binding. The closest you can achieve is 
to define a method as `final`, which forbids subclasses from re-implementing it altogether.

This is a huge philosophical difference. In Java an object behaves identically, no matter what the 
type of the reference actually is. Conversely, in C# the behaviour of a single object may change 
depending on the declared type of the reference.

In Java, you can extend any class from any library and override any accessible method 
(unless explicitly forbidden with final). This sometimes proves invaluable, as it allows you to 
work-around bugs or unwanted behaviour in external code bases. In C#, you can only override what the 
original developer has marked as virtual. This in turns allows you to be more specific about which 
parts of your code are extensible (i.e. can be overriden).
