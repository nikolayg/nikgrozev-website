---
layout: post
title: Functional Programming and Category Theory [Part 1] - Categories and Functors
date: 2016-03-14 17:18:19.000000000 +11:00
type: post
published: true
status: publish
excerpt: 
    This series of tutorials defines and illustrates the Category Theory concepts which are most widely adopted in FP. 
    We will use simple Scala and pseudocode examples to illustrate the new terms. In this post we will look
    into Categories and Functors ...
categories:
- Category Theory
- Functional programming
- Scala
- blog
tags:
- Category Theory
- Functional programming
- Scala
author:
  login: nikolaygrozev
  email: nikolay.grozev@gmail.com
  display_name: nikolaygrozev
  first_name: 'Nikolay'
  last_name: 'Grozev'
---

# Table of Contents 

- [Introduction](#introduction)
- [Categories](#categories)
    - [Examples](#categories-examples)
    - [The Hask Category](#categories-hask)
- [Functors](#functors)
    - [Functors in FP](#functors-in-fp)
    - [Functors in code](#functors-in-code)
    - [Examples](#functors-examples)



<div id='introduction' />
# Introduction

Category Theory is a mathematical discipline with a wide range of applications in theoretical computer science. 
Concepts like *Category*, *Functor*, *Monad*, and others, which were originally defined in Category Theory, 
have become pivotal for the understanding of modern Functional Programming (FP) languages and paradigms. 
The meaning and applications of these terms in FP can be understood without in-depth knowledge of the corresponding 
mathematical definitions and axiomatic. However, a common knowledge of the underlying theory can help FP programmers 
understand the design and structure of commonly used libraries and tools and be more productive. 

This series of tutorials defines and illustrates the Category Theory concepts which are most widely adopted in FP. 
We will use simple Scala and pseudocode examples to illustrate the new terms.  



<div id='categories' />
# Categories

A **category** is a simple algebraic structure for modelling objects and their relationships. 
A category **<u>C</u>** consists of a collection of objects **<u>ob(C)</u>** and a collection of 
arrows/morphisms **<u>hom(C)</u>** connecting the objects. In other words, every arrow **<u>f</u>** 
can be defined as a pair **<u>[a,b]</u>** of the objects it connects. We write **<u>f: a &rarr; b</u>**.

A category also defines an operation for composing arrows, such that for every 
**<u>f: a &rarr; b</u>** and **<u>g : b &rarr; c</u>**, their composition **<u>g &#8728; f</u>** 
is also an arrow, which connects **<u>a</u>** and **<u>c</u>** - i.e. **<u>g &#8728; f: a &rarr; c</u>**. 

A collection of objects and arrows qualifies as a category only if:

- The composition is associative. More formally, **<u>h &#8728; (g &#8728; f) = (h &#8728; g) &#8728; f</u>** for every three arrows, and;
- For every object **<u>a</u>** there is an identity arrow (i.e. a loop) **<u>i<sub>a</sub></u>** that connects it to itself: **<u>i<sub>a</sub>: a &rarr; a</u>**. 
- The identities should have the obvious property that for every **<u>f: a &rarr; b</u>** the following is true **<u>i<sub>b</sub> &#8728; f = f = f &#8728; i<sub>a</sub></u>**. In other words, identities are *neutral* to composition.

[//]: # (====================================================================== Category.pdf)
<figure>
  <img src="/assets/images/Functional Programming and Category Theory Part 1 - Categories and Functors/category.jpg" alt="A Category." >
  <figcaption>A category of 4 objects. For every object there must be an identity arrow (i.e. loop). The order of composition should not matter.</figcaption>
</figure>

<div id='categories-examples' />
## Examples

An intuitive example of a category is the inter-city road infrastructure. 
A portion of this category is depicted in the diagram below. In it, the objects are all cities around the world. 
We consider two cities to be connected with an arrow if one is reachable from the other. 
We assume each city is reachable from itself which is represented with the identity arrows. 
If we can travel from city **<u>A</u>** to **<u>B</u>**, and from **<u>B</u>** to **<u>C</u>**, 
then we can do so from **<u>A</u>** to **<u>C</u>**. Hence, we can compose arrows.

[//]: # (====================================================================== Category_Roads.pdf)
<figure>
  <img src="/assets/images/Functional Programming and Category Theory Part 1 - Categories and Functors/category_roads.jpg" alt="The road network forms a category." >
  <figcaption>The road network forms a category.</figcaption>
</figure>


More generally, every directed graph forms a category whose objects are the graph nodes. 
In it, two nodes/objects **<u>a</u>** and **<u>b</u>** are connected with an arrow only if there is 
*path* between them in the graph. We also consider that every node in the graph has an arrow to itself - i.e. it is reachable from itself.

In Object Oriented Programming (OOP), a class hierarchy also forms a category. 
The category objecta are the types (e.g. classes, traits, interfaces). We consider two 
types **<u>A</u>** and **<u>B</u>** to be connected with an arrow if **<u>A</u>** is a subtype of **<u>B</u>**. 
These arrows are composable, because if **<u>A</u>** is a subtype of **<u>B</u>**, and **<u>B</u>** of **<u>C</u>**, 
then **<u>A</u>** is a subtype of **<u>C</u>**. Finally, each type is a subtype of itself and thus we have 
identity arrows as well. This is depicted in the following figure.


[//]: # (====================================================================== Category_OOP.pdf)
<figure>
  <img src="/assets/images/Functional Programming and Category Theory Part 1 - Categories and Functors/category_oop.jpg" alt="Category of types." >
  <figcaption>Category of types.</figcaption>
</figure>


<div id='categories-hask' />
## The Hask Category

In Functional Programming, the **<u>Hask</u>** category has a special role. 
The objects of **<u>Hask</u>** are all types of the Haskel programming language - i.e. **<u>Ob(Hask) = {Int, String, ...}</u>**. 
We can generalise that to the types of other languages, like Scala, as well. 
For every function converting one type to another, in **<u>Hask</u>** there is an arrow between the two types. 
Arrow composition is just function composition. The identity arrows correspond to the identity functions. 
The following diagram depicts a portion of the **<u>Hask</u>** category.

[//]: # (====================================================================== Hask.pdf)
<figure>
  <img src="/assets/images/Functional Programming and Category Theory Part 1 - Categories and Functors/hask.jpg" alt="The Hask Category." >
  <figcaption>The Hask Category.</figcaption>
</figure>


<div id='functors' />
# Functors

In category theory, a Functor **<u>F</u>** is a transformation between two categories 
**<u>A</u>** and **<u>B</u>**. We write **<u>F : A &rarr; B</u>**. **<u>F</u>** must map every 
object and arrow from **<u>A</u>** to **<u>B</u>**. In other words, if **<u>a &isin; ob(A)</u>** 
then **<u>F(a) &isin; ob(B)</u>**, and if **<u>f &isin; Hom(A)</u>** then **<u>F(f) &isin; Hom(B)</u>**. 

We also require that **<u>F</u>** preserves the structure (i.e. identity arrows and composition) of the source category. 
More formally:

- If **<u>f : a &rarr; b</u>** is an arrow in **<u>A</u>** then **<u>F(f):F(a) &rarr; F(B)</u>** is an arrow in **<u>B</u>**.
- **<u>F(id<sub>X</sub>) = id<sub>(F(X))</sub></u>**, which means that each identity arrow in **<u>A</u>** is transformed to an identity arrow of the corresponding object in **<u>B</u>**.
- **<u>F(g &#8728; f) = F(g) &#8728; F(f)</u>**, which means that the mapping of arrows' composition in **<u>A</u>** is a composition of their mappings in **<u>B</u>**.
 
[//]: # (====================================================================== Functor.pdf)
<figure>
  <img src="/assets/images/Functional Programming and Category Theory Part 1 - Categories and Functors/functor.jpg" alt="A Functor from A to B." >
  <figcaption>A Functor from A to B.</figcaption>
</figure>


When a functor **<u>F</u>** transforms a category **<u>A</u>** into itself, we call it an *endofunctor* and we write **<u>F:A &rarr; A</u>**.


[//]: # (====================================================================== EndoFunctor.pdf)
<figure>
  <img src="/assets/images/Functional Programming and Category Theory Part 1 - Categories and Functors/endofunctor.jpg" alt="An Endofunctor." >
  <figcaption>An Endofunctor.</figcaption>
</figure>


<div id='functors-in-fp' />
## Functors in FP

Before we delve into Functors and FP, we need to introduce the concept of a *<u>type constructor</u>*. 
Essentially, a type constructor is a generic type definition, which takes another type as a parameter. 
For example, in Scala **<u>List[T]</u>**, **<u>Vector[T]</u>**, and **<u>Option[T]</u>** are type constructor. 
You need to specify the value of the type parameter **<u>T</u>** in order to produce a concrete type. 
For example, **List[String]** is a type, while **List** itself is not - it is a type constructor. 
We will write **TC[ _ ]** to denote that **<u>TC</u>** is a type constructor. 

Functors in Category Theory are a much more general concept than in functional programming (FP). 
<u>All functors in FP are just endofunctors in Hask</u>. Furthermore, each functor **<u>F</u>** is 
associated with a type constructor **TC[ _ ]**. Each type **<u>A</u>** in **<u>Hask</u>** is 
transformed to **<u>TC[A]</u>**. For example, if **<u>TC = List</u>** then **<u>F: Int &rarr; List[Int]</u>**. 
In other words, the type constructor uniquely defines the mapping of **<u>Hask</u>** objects. 

In order to define a functor, we also need to define the arrow mapping. The arrows in **<u>Hask</u>** are just functions. 
Hence, we need to provide a function called **<u>map</u>** with the following signature: 
**<u>map: (A &rarr; B) &rarr; (TC[A] &rarr; TC[B])</u>**. For every arrow/function **<u>f: A &rarr; B</u>** it 
returns its projection/mapping which is also a function - **<u>F(f): TC[A] &rarr; TC[B]</u>**.

To summarise, a functor in FP is uniquely defined by a type constructor **TC[ _ ]** and a map function with the aforementioned signature. 
The following diagram depicts a functor, whose type constructor is **List[_]**.

[//]: # (====================================================================== HaskFunctor.pdf)
<figure>
  <img src="/assets/images/Functional Programming and Category Theory Part 1 - Categories and Functors/haskfunctor1.jpg" alt="The List functor." >
  <figcaption>The List functor.</figcaption>
</figure>

<div id='functors-in-code' />
## Functors in code

Following the previous definitions, in Scala a functor can be defined as:

```scala
trait Functor[TC[_]] {
	def map[A,B] (f: A => B): (TC[A] => TC[B])
}
```

Note that the return type of **<u>map</u>** is a function. 
Often the caller just wants to apply/run this function to a given arguement, 
instead of passing it on as an arguement or reusing it later on. 
Thus, it is convenient to use another analogous definition of a functor, which also applies the resulting function:

```scala
trait Functor[TC[_]] {
	def map[A,B] (f: A => B)(param : TC[A]) : TC[B]
}
```
This is essentially a *shortcut* for mapping a function and applying it in one go. 
We can also define the **<u>map</u>** function as a method of the type constructor **TC** itself:

```scala
trait TC[A] {
	def map[B] (f: A => B): TC[B]
}
```

For simplicity, in this case we can just say that **TC** itself is a functor, 
because it is a *type constructor* and has a *map* function. To make it more readable we can just write:

```scala
trait Functor[A] {
	def map[B] (f: A => B): Functor[B]
}
```

This trait can then be implemented from all concrete type constructors (e.g. **List** and **Vector**) thus making them functors.

<div id='functors-examples' />
## Examples

In FP, functors are usually applied to map/convert instances of generic data structures 
(i.e. type constructors) in a way which preserves their inner structure. For collections 
like **List** and **Vector** we would implement the *map* function by just applying it to every element, as in the following pseudocode:

```scala
class List[A] extends Functor[A] {
    // ... 
    def map[B] (f: A => B): List[B] = {
        this match {
            case List() => this
            case h::t => f(h)::(t.map(f))
        }
    }
    // ... 
}
```

For type constructors which encode *"success-or-failure"* state like **Option** or **Either** 
the map function can be applied only if there is a contained value:

```scala
class Option[A] extends Functor[A] {
    // ... 
    def map[B] (f: A => B): Option[B] = {
        this match {
            case None => None
            case Some(x) => Some(f(x))
        }
    }
    // ... 
}
```
Given that all these functors have a common supertype allows us to write functions which will work with all of them. For example:

```scala
def useAnyIntFunctor(functor: Functor[Int]) =
     functor.map(_ + 5).map(_ * 2) 
```
This will work *"as expected"* with all functors - Option, List, etc. However, 
in this example the return type is **Functor**, not the actual compile-time type of the parameter. 
This can be amended by defining a method type parameter for the arguement's type.


> **<u>Update</u>**: updated with comments from [Ken Scambler](https://twitter.com/kenscambler) and other collegues from [REA](http://www.rea-group.com/IRM/content/default.aspx)

