---
layout: post
title: Functional Programming and Category Theory [Part 2] – Applicative Functors
date: 2016-04-11 23:54:06.000000000 +10:00
type: post
published: true
status: publish
categories:
- Functional programming
- Scala
excerpt: 
    This post introduces a type of functors called Applicative Functors. 
    Unlike the ordinary functors, applicatives allow 
    us to work with multi-arguement functions and thus turn out to be quite useful in FP.
tags:
- Applicative Functor
- Functional programming
- Scala
- Category Theory
author:
  login: nikolaygrozev
  email: nikolay.grozev@gmail.com
  display_name: nikolaygrozev
  first_name: 'Nikolay'
  last_name: 'Grozev'
---

# Table of Contents 

- [Introduction](#introduction)
- [Motivating Example](#curried-functions)
- [Curried Functions](#curried-functions)
- [Applicative Functor: Structure and Computation](#struct-and-compute)
  - [Function "Pure"](#function-pure)
  - [Function "Apply"](#function-apply)
  - [Variations](#variations)
  - [Putting the Pieces Together](#putting-pieces-together)
- [Laws](#laws)
  - [Identity Law](#identity-law)
  - [Composition Law](#composition-law)
  - [Homomorphism Law](#homomorphism-law)
  - [Interchange Law](#interchange-law)
- [Applicative Functors are Functors](#app-functor-is-functor)
- [References](#references)



<!--- =============================== INTRODUCTION =============================== -->
<div id='introduction' />
# Introduction

In a [previous post](www.nikgrozev.org/2016/03/14/functional-programming-and-category-theory-part-1-categories-and-functors/) 
I talked about the basic concepts in Category Theory and Functional Programming (FP) - namely Categories and Functors. 
This post introduces a type of functors called *Applicative Functors*. Unlike the ordinary functors, applicatives allow 
us to work with multi-arguement functions and thus turn out to be quite useful in FP. 
 


<!--- =============================== MOTIVATING EXAMPLE =============================== -->
<div id='curried-functions' />
# Motivating Example

We can reason about a functor as a context for values of a given type **<u>A</u>** - e.g. **<u>List[A], Vector[A], Option[A]</u>**. 
A functor allows us to transfer/map a function **<u>f:A &rarr; B</u>** to that context. For example:

- **map[List]: (A &rarr; B) &rarr; (List[A] &rarr; List[B])**
- **map[Option]: (A &rarr; B) &rarr; (Option[A] &rarr; Option[B])**

But what if the function that we want to use takes multiple parameters? Functors can only map over single-arguement functions. 
The following example illustrates the problem:

```scala
def sum(x : Int)(y : Int) = x + y

val xOpt : Option[Int] = Some(1)
val yOpt : Option[Int] = Some(2)

// How can we sum xOpt and yOpt?
// val result = sum(xOpt)(yOpt) does not compile
```

We can obviously "unbox" the values in the two options, apply the function, and embed the new result in an option. 
This approach is far from elegant. It is not general and does not apply to other functor. We would like to have 
a generic solution that works with many other functors as well.

<!--- =============================== CURRIED FUNCTIONS =============================== -->
<div id='curried-functions' />
# Curried Functions

Before we discuss applicative functors, we need to introduce the concept of [currying](https://en.wikipedia.org/wiki/Currying). 
In FP, we can partially apply a function of multiple parameters by specifying a subset of its required parameters. 
For this purpose, Scala offers special syntax, allowing us to group function parameters into separate parameter lists.
 
```scala
def curriedF(x : Int) (y : Int) = x + y

// We need to use underscore after the function invocation
val partiallyApplied : Int => Int = curriedF(1)_

// Call the partially applied function. Result is 3
val result = partiallyApplied(2)
```

The ability to create partially applied functions is called **currying**. Scala also allows us to create curried 
functions from non-curried ones:

```scala
// Non-curried
def f(x : Int, y : Int) = x + y

// Make a curried version of f
val fCurried = (f _).curried

// Invoke the non-curried and curried functions
f(1, 2)
fCurried(1)(2)
```
In fact, this is a much more general idea which is not specific to Scala and FP. Unfortunately, most languages do not 
explicitly support currying. We can still easily curry every function in most modern languages as long as they support 
anonymous functions, as in the following Python example:
 
```python
# Non-curried
def f(x, y): return x + y

# Make a curried version of f
def fCurried(x): return lambda y : f(x, y) 

# Invoke the non-curried and curried functions
f(1, 2)
fCurried(1)(2)
```
Curried and non-curried functions are equivalent. As we just showed, we can convert each function to a curried one. 
The opposite is also true. In terms of notation, we'll denote the non-curried version of a function as 
**<u>f: (A<sub>1</sub> &times; A<sub>2</sub> &times; ... &times; A<sub>n-1</sub>) &rarr; A<sub>n</sub></u>**, 
or we could just write: **<u>f: (A<sub>1</sub>, A<sub>2</sub>, ...,A<sub>n-1</sub>) &rarr; A<sub>n</sub></u>**.

The curried version will be written as: **<u>f: A<sub>1</sub> &rarr; A<sub>2</sub> &rarr; ... &rarr; A<sub>n</sub></u>**.

Currying is a simple but powerful method to represent a function with multiple arguements as a single-arg function. 
This allows us to generalise everything that works for single arguement functions to the multi-arguement case.


<!--- =============================== STRUCTURE AND COMPUTATION =============================== -->
<div id='struct-and-compute' />
# Applicative Functor: Structure and Computation

An applicative functor is defined by:

- A type constructor **<u>F[ _ ]</u>** - e.g. **<u>List</u>**, **<u>Option</u>**;
- A function **<u>pure : A &rarr; F[A]</u>**, where **<u>A</u>** can be any type (including types of functions);
- A function **<u>apply: F[A &rarr; B] &rarr; (F[A] &rarr; F[B])</u>**, where **<u>A</u>** and **<u>B</u>** can 
be any types (including types of functions);

<div id='function-pure' />
## Function "Pure"

The **<u>pure</u>** function is also known as **<u>point</u>** in some languages and libraries. Essentially, 
it is a constructor which can convert a value of any type **<u>A</u>** to a value of **<u>F[A]</u>**. 
We often say that **<u>pure</u>** "lifts" its arguement to the corresponding type defined by the type constructor 
(e.g. from **<u>Int</u>** to **<u>List[Int]</u>**). Here is a sample implementation for **<u>Option</u>**.

```scala
// Implementation for Options
def pure[A](a : A) : Option[A] = Some(a)

pure("Text") // Some("Text")

// We can also embed a function in an Option
pure(x:Int => x.toString) //Some(Int => String)
```

Similarly, an implementation of **<u>pure</u>** for **<u>List</u>** would just wrap it in a list:

```scala
// Implementation for List
def pure[A](a : A) : List[A] = List(a)

// We can embed a function in a list
pure(x:Int => x + 1) //List(Int => Int)
```

<div id='function-apply' />
## Function "Apply"

The **<u>apply</u>** function takes a function, which is embedded into the context defined by the type 
constructor (e.g. **<u>Option[A &rarr; B]</u>**), and lifts/converts it to a function in the realm of the type 
constructor (e.g. **<u>Option[A] &rarr; Option[B]</u>**). Here is a sample implementation for **<u>Option</u>**:

```scala
  // Implementation for Option
  def apply[A, B](oFunc : Option[A => B]) : Option[A] => Option[B] = {
    // Returns an anonymous function
    oA: Option[A] => oA match {
      case None => None
      case Some(a) => oFunc.map(f => f(a))
    }
  }
```

Implementing "apply" for List is somewhat less straigthforward. Given a list of functions 
(i.e. **<u>List[A &rarr; B]</u>**), we need to return a function that converts from one list to another 
(i.e. **<u>List[A] &rarr; List[B]</u>**). One approach is to return a function, which applies all functions 
in the given list to all elements in its input parameter:

```scala
  // Implementation for List
  def apply[A, B](lFunc : List[A => B]) : List[A] => List[B] = {
    // Create an inner result function
    def resultF (lA: List[A]) : List[B] = {
      lA match {
        case List() => List()
        case head :: tail => lFunc.map(f => f(head)) ::: resultF(tail)
      }
    }
    return resultF
  }
```

<div id='variations' />
## Variations

The **<u>apply</u>** function returns a function as a result. One would usually take this result and invoke 
it with some parameter. Hence, it often makes sense to use an alternative signature for the **<u>apply</u>** function:

```scala
def apply[A,B](f: F[A=>B], fa: F[A]): F[B]
``` 

This definition does both at the same time - it converts/maps the lifted function **<u>f: F[A=&gt;B]</u>** 
to a function of type **<u>F[A]=&gt;F[B]</u>** and then applies it to the second arguement **<u>fa</u>**. 
It is a convenience "wrapper" for the previous definition of **<u>apply</u>**. In the rest of the article 
we'll use both definitions interchangeably in accordance with the needs of each example. 

<div id='putting-pieces-together' />
## Putting the Pieces Together

Let's consider a multi-argument curried function **<u>f: A<sub>1</sub> &rarr; A<sub>2</sub> &rarr; ... &rarr; A<sub>n</sub></u>**. 
The function **<u>pure</u>** can embed **<u>f</u>** into an instance of **<u>F[A<sub>1</sub> &rarr; 
A<sub>2</sub> &rarr; ... &rarr; A<sub>N</sub>]</u>**. 

This sets the stage for an invocation of the **<u>apply</u>** function which transforms 
**<u>F[A<sub>1</sub> &rarr; A<sub>2</sub> &rarr; ... &rarr; A<sub>N</sub>]</u>** to **<u>F[A<sub>1</sub>] &rarr; F[A<sub>2</sub> &rarr; ... &rarr; A<sub>N</sub>]</u>**. We can further use **<u>apply</u>** repeatedly to transform this to **<u>F[A<sub>1</sub>] &rarr; F[A<sub>2</sub>] &rarr; ... &rarr; F[A<sub>N</sub>]</u>**, and hence we have been able to *"lift"* the **<u>f</u>** function 
into the applicative functor. This computation is depicted in the next figure.

<figure>
  <img src="/assets/images/Functional Programming and Category Theory Part 2 - Applicative Functors/applicativefunctor1.jpg" alt="Computational process of applying an Applicative functor." >
  <figcaption>Computational process of applying an Applicative functor.</figcaption>
</figure>


The following diagram depicts how an Applicative Functor acts as an endofunctor in the **Hask** category. 
It shows how the generic function **<u>pure</u>** maps each type (i.e **<u>A<sub>1</sub> ... A<sub>n</sub></u>**) 
to the corresponding type defined by the type constructor (i.e **<u>F[A<sub>1</sub>] ... F[A<sub>n</sub>]</u>**). 
By composing the **<u>apply</u>** and **<u>pure</u>** functions, 
we can map any function **<u>f: A<sub>1</sub> &rarr; ... &rarr; A<sub>n</sub></u>** to a function with the 
following form **<u>f ': F[A<sub>1</sub>] &rarr; ... &rarr; F[A<sub>n</sub>]</u>**.


<figure>
  <img src="/assets/images/Functional Programming and Category Theory Part 2 - Applicative Functors/applicativefunctorhask.jpg" alt="An Applicative Functor in Hask" >
  <figcaption>An Applicative Functor in Hask.</figcaption>
</figure>


Going back to the example from the motivation section, we can implement the desired logic as:

```scala
def sum(x : Int)(y : Int) = x + y
val xOpt : Option[Int] = Some(1)
val yOpt : Option[Int] = Some(2)

// The result is Some(3)
apply(apply(pure(sum), xOpt), yOpt)
```

For convenience, some languages and libraries alias the **<u>apply</u>** function with the left associative **<u>&lt;*&gt;</u>** operator, which allows for the more succinct expression:

```scala
pure(sum) <*> xOpt <*> yOpt
```

<!--- =============================== Laws =============================== -->
<div id='laws' />
# Laws

This section is a bit less intuitive and you may wish to skip it when reading for the first time. To qualify as an Applicative Functor, a pair of **<u>apply</u>** and **<u>pure</u>** functions must obey several laws. In this section, we'll formulate and demonstrate them using the aforementioned Applicative Functor for Option.
 
<div id='identity-law' />
## Identity Law

The identity law states that **<u>pure(id) &lt;*&gt; v = v</u>** for every **<u>v</u>**, where **<u>id</u>** is an identity function. This rule implies that **<u>pure</u>** preserves the identity function. The following snippet exemplifies this law for the Option's applicative functor:

```scala
// Identity Function
def id[A](a: A) = a

// Any v - for example Some(42)
val v: Option[Int] = Some(42)

// The following should be true for every v. In other words,
// pure(id[Int]) should be an identity with respect to <*>.
pure(id[Int]) <*> v == v
```

<div id='composition-law' />
## Composition Law

The composition law states that **<u>pure(f) &lt;*&gt; pure(x)=pure(f(x))</u>** meaning that for every function **<u>f</u>** and value **<u>x</u>**, applying the lifted/mapped function to the lifted value is the same as lifting the function's result, as in the following snippet: 

```scala
// Any function f. For example: x+1
def f(x: Int) = x + 1

// Any x - for example Some(42)
val x: Option[Int] = Some(42)

// The following should be true for every f and x. In other words, "lifting" 
// f and applying x, should be the same as lifting f(x)
pure(f) <*> x == pure(f(x))
```

<div id='homomorphism-law' />
## Homomorphism Law

The homomorphism law states that **<u>pure(&#8728;) &lt;*&gt; u &lt;*&gt; v &lt;*&gt; w =  u &lt;*&gt; (v &lt;*&gt; w)</u>**. Here, "&#8728;" is function composition, which is a function of two arguements (the functions it composes). This law states that function composition is preserved by the applicative functor. The following snippet illustrates:

```scala
// Composition Function
def compose[A,B,C](f: A=>B, g: B=>C) = 
  a: A => g(f(a))

// Any two functions u and v. For example:
def u(x: Int) = x+1
def v(x: Int) = x+2

// Any w - for example Some(42)
val w: Option[Int] = Some(42)

// The following should be true. In other words, pure(compose) 
// should compose u and v with respect to <*>.
pure(compose) <*> u <*> v <*> w == u <*> (v <*> w)
```

<div id='interchange-law' />
## Interchange Law

The interchange law states that **<u>u &lt;*&gt; pure x = pure (f =&gt; f(x)) &lt;*&gt; u</u>**. Therefore, we should be able to change the order of **<u>apply</u>**'s the parameters. To do so, however, we need to convert the second parameter to a higher order function, as in the following snippet:

```scala
// Any function u. For example: x+1
def u(x: Int) = x + 1

// Any u - for example Some(42)
val x: Option[Int] = Some(42)

// The following should be true. In the following code, f is an anonymous
// higher order function that applies its arguement (a function) to x.
u <*> x == pure(f:(Option[Int] => Option[Int]) => f(x)) <*> u
```

<!--- =============================== Applicative Functors are Functors =============================== -->
<div id='app-functor-is-functor' />
# Applicative Functors are Functors

So far we have defined an applicative functor, as a pair of two functions - **<u>pure</u>** and **<u>apply</u>**. 
However, we have not yet shown that it is indeed a type of functor. We can do so by 
implementing a **<u>map</u>** function as follows:

```scala
def map[F[]]map(f: A => B, fa:F[A]) =
	pure(f) <*> fa
```

The previous laws ensure that this implementation conforms to the functor laws.

In the [previous post](www.nikygrozev.org/2016/03/14/functional-programming-and-category-theory-part-1-categories-and-functors/) 
we saw that a Functor can be defined as a trait representing the type constructor and defining the **<u>map</u>** function as a method. 
Analogously, we can define the **<u>AppFunctor</u>** trait as follows: 

```scala
trait AppFunctor[A] {
    def pure(a: A): AppFunctor[A]

    // OOP version of apply[A,B](f: AppFunctor[A => B], fa: AppFunctor[A]): AppFunctor[B] 
    // We'll use "this" instead of the second parameter "fa".
    def apply[A,B](f: AppFunctor[A => B]): AppFunctor[B] 

    // We can also implement map (from the definition of a Functor) 
    // through the other two methods:
    def map[A,B](f: A => B): AppFunctor[B] =
        apply(pure(f))
}
```

<!--- =============================== References =============================== -->
<div id='references' />
# References

- [https://github.com/dcapwell/scala-tour/blob/master/Applicative%20Functor.md](https://github.com/dcapwell/scala-tour/blob/master/Applicative%20Functor.md)
- [https://drboolean.gitbooks.io/mostly-adequate-guide/content/ch10.html](https://drboolean.gitbooks.io/mostly-adequate-guide/content/ch10.html)
- [http://808fabrik.com/scala/functor-monad-in-scala-handout.pdf](http://808fabrik.com/scala/functor-monad-in-scala-handout.pdf)
