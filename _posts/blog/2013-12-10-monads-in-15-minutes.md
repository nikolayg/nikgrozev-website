---
layout: post
title: Monads in 15 minutes
date: 2013-12-10 11:28:29.000000000
type: post
published: true
status: publish
excerpt: 
    This tutorial explains the 
    intuition behind Monads and demonstrates them with a few simple and short python examples. 
    Its goal is to explain Monads simply in less than 15 minutes and thus it refrains from making 
    insightful philosophical and theoretical reflections ...
categories:
- Functional programming
- Python
tags:
- Functional programming
- Monad
- Python
---

# Introduction

At the [YOW! 2013](http://yowconference.com.au/) conference one of the designers of Haskell 
[prof. Philip Wadler](http://homepages.inf.ed.ac.uk/wadler/) illustrated how Monads enable a 
purely functional programming language to perform inherently imperative operations like I/O 
and exception handling. Not surprisingly, there has been a lot of interest in the topic resulting 
in almost exponential growth of Monads tutorials and resources on the web. It is unfortunate 
that most tutorials exemplify Monads with functional languages, given that most "Monad newbies" 
are not experienced functional programmers. However, Monads are not specific to Haskell and other 
pure functional languages and can be used and exemplified in imperative languages as well, which 
is the goal of this tutorial.



**So how is this tutorial different from the rest out there?** This tutorial explains the 
intuition behind Monads and demonstrates them with a few simple and short python examples. 
Its goal is to explain Monads simply in less than 15 minutes and thus it refrains from making 
insightful philosophical and theoretical reflections about 
[burritos](http://blog.plover.com/prog/burritos.html), 
[space suits](http://web.archive.org/web/20081206204420/http://www.loria.fr/~kow/monads/index.html), 
[writing desks](http://www.infoq.com/presentations/Why-is-a-Monad-Like-a-Writing-Desk) and endofunctors.

# Motivating Examples

We will look at 3 example problems for function composition. We will solve each of them in two ways - 
the standard imperative way and the monad way. Then we'll compare the approaches.

## 1. Logging

Let's assume we have the unary (single argument) functions `f1`, `f2` and `f3`, each of which returns 
an increment of its integer parameter. Additionally, each of them generates a readable log message, 
representing its arithmetic operation:

```python
def f1(x):
    return (x + 1, str(x) + "+1")

def f2(x):
    return (x + 2, str(x) + "+2")

def f3(x):
    return (x + 3, str(x) + "+3")
```

Now we would like to chain the functions `f1`, `f2` and `f3` given a parameter `x` - i.e. we want to compute `x+1+2+3`. 
Additionally we want to have a readable description of all applied functions.

One way to achieve this is:

```python
log = "Ops:"

res, log1 = f1(x)
log += log1 + ";"

res, log2 = f2(res)
log += log2 + ";"

res, log3 = f3(res)
log += log3 + ";"

print(res, log)
```

This solution is far from perfect, as we have repeated "glue code", which accumulates the overall result and 
prepares the input of the functions. If we add a new function `f4` to the sequence, we'll have to repeat the 
glue code again. Moreover, manipulating the state of the variables `res` and `log` makes the code 
less readable, and is not essential to the program logic.

Ideally, we would like to have something as simple as the chain invocation `f3(f2(f1(x)))`. 
Unfortunately, the return types of `f1` and `f2` are incompatible with the input parameter types of `f2` and `f3`. 
To solve the problem we introduce two new functions:

```python
def unit(x):
    return (x, "Ops:")

def bind(t, f):
    res = f(t[0])
    return (res[0], t[1] + res[1] + ";")
```

Hence, we can solve the problem with a single chained function invocation:

```python
print( bind(bind(bind(unit(x), f1), f2), f3) )
```

The following diagram depicts the computational process when `x=0`. 
By `v1`, `v2` and `v3` we denote the interim values resulting from calling unit and bind:


<figure>
  <img src="/images/blog/Monads in 15 minutes/example11.png" alt="Example 1 – Computational process of the monadic solution" >
  <figcaption>Example 1 – Computational process of the monadic solution.</figcaption>
</figure>


The `unit` function converts the input parameter `x` into a tuple/pair of 
an integer and a string. The subsequent invocations of `bind` call their parameter 
function `f` and accumulate its result with the interim value represented by the `t` 
formal parameter.

This avoids the shortcomings of our previous approach because the `bind` function implements 
all the glue code and we don't have to repeat it. We can add a new function `f4` by just including 
it in the sequence as `bind(f4, bind(f3, ... ))` and we won't have to do other changes.

## 2. List of Interim Values

In this example we assume that we have three simple unary functions:

```python
def f1(x): return x + 1

def f2(x): return x + 2

def f3(x): return x + 3
```

As in the previous example we would like to compose them in order to compute `x+1+2+3`. 
Additionally we would like to generate a list of all interim and final values - i.e. `x`, `x+1`, `x+1+2`, and `x+1+2+3`.

Unlike the previous example, in this one the functions are composable as their parameter and result types match. 
Therefore the simple invocation `f3(f2(f1(x)))` will give us the desired value of `x+1+2+3`. 
However, this won't generate the interim values.

A straightforward approach is:

```python
lst = [x]

res = f1(x)
lst.append(res)

res = f2(res)
lst.append(res)

res = f3(res)
lst.append(res)

print(res, lst)
```

Again, this is not a very good solution, as we have glue code that takes care of aggregating the 
interim values in a list. If we add a new function f4 to the sequence, we'll have to repeat the glue 
code for updating the list.

To overcome these issues, just like in our previous example we introduce two auxiliary functions:

```python
def unit(x):
    return (x, [x])

def bind(t, f):
    res = f(t[0])
    return (res, t[1] + [res])
```

Now we can write the entire program as a single sequence of invocations:

```python
print( bind(bind(bind(unit(x), f1), f2), f3) )
```

The following diagram depicts the computational process when `x=0`. Again `v1`, `v2` and `v3` are 
the interim values resulting from calling `unit` and `bind`:

<figure>
  <img src="/images/blog/Monads in 15 minutes/example21.png" alt="Example 2 – Computational process of the monadic solution" >
  <figcaption>Example 2 – Computational process of the monadic solution.</figcaption>
</figure>


## 3. Nulls/Nones

Lets try something a bit more interesting with classes and objects. 
Lets assume we have a class Employee with two methods:

```python
class Employee:
    def get_boss(self):
        # Return the employee's boss

    def get_wage(self):
        # Compute the wage
```

Each employee instance has a boss of type `Employee` and a wage, which can be accessed through 
the two methods. Both of these methods can return None (i.e. the wage is unknown, or the employee does not have a boss). 
In this example we want to develop a program, which given an `Employee` instance `john` returns his boss's wage. 
If the wage can no be determined or if `john` is `None`, we should return `None`.

Ideally we would want to write something as simple as:

```python
print(john.get_boss().get_wage())
```

However, as the methods can return `None`, this can result in an error. A simple workaround is:

```python
result = None
if john is not None and john.get_boss() is not None and john.get_boss().get_wage() is not None:
    result = john.get_boss().get_wage()

print(result)
```

However, in this solution we are unnecessarily calling multiple times the `get_boss` and `get_wage` methods. 
If they are computationally expensive (e.g. if they access a database) this may be undesirable. Hence, our solution should look like:

```python
result = None
if john is not None:
    boss = john.get_boss()
    if boss is not None:
        wage = boss.get_wage()
        if wage is not None:
            result = wage

print(result)
```

This obviously is not pretty, since we had to write 3 bloated if statements to get to our result. 
To solve the problem, we resort to the same trick we used previously. We define the following two functions:

```python
def unit(e):
    return e

def bind(e, f):
    return None if e is None else f(e)
```

Now we can write the program in one line:

```python
print( bind(bind(unit(john), Employee.get_boss), Employee.get_wage) )
```

You probably noticed that we didn't actually need to call `unit(john)`, as it simply returns 
its input parameter. We did it to keep to the previous framework/pattern, so we can define a generalised 
solution later on. Also, note that in python methods can be used as simple functions. That is, instead of 
`john.get_boss()` we can write `Employee.get_boss(john)`, which allows us to write the code above.

The following diagram illustrates the computational process if `john` does not have a boss (i.e. `john.get_boss()` returns `None`):

<figure>
  <img src="/images/blog/Monads in 15 minutes/example31.png" alt="Example 3 – Computational process of the monadic solution" >
  <figcaption>Example 3 – Computational process of the monadic solution.</figcaption>
</figure>


# Generalisation - Monads

Lets assume we want to compose the functions f1, f2, ... fn. If all input parameters match all return types, 
we can simply use `fn(... f2(f1(x)) ...)`. The following diagram depicts the underlying computational process. 
`v1`, `v2` ... `vn` represent the interim results of the function invocation.

<figure>
  <img src="/images/blog/Monads in 15 minutes/dierct_composition1.png" alt="Direct Composition – Computational model" >
  <figcaption>Direct Composition – Computational model.</figcaption>
</figure>

However, often this approach is not applicable. For instance in the Logging example the types of the 
input parameters and the returned values are different. In the 2nd and 3rd examples the functions were 
composable, but we wanted to "inject" additional logic between the function invocations - in example 2 
we wanted to aggregate the interim values and in example 3 we wanted to insert None/Null checks.

## 1\. Imperative Solution

In the above examples we first defined a straightforward imperative approach, which is depicted below:

<figure>
  <img src="/images/blog/Monads in 15 minutes/imperativecomposition2.png" alt="Imperative Composition - Computational model" >
  <figcaption>Imperative Composition - Computational model.</figcaption>
</figure>


Before calling f1, we execute some initialisation code. For instance, in Example 1 and 2 we initialised variables to store the aggregated log and interim values. 
After that, we call the functions f1, f2... fn and between the invocations we put some glue code. 
In Example 1 and 2 the glue code aggregates the log and the interim values. 
In Example 3, the glue code checks if the interim values are Null/None.

## 2\. Enter Monads

As we saw in the examples, this straightforward approach has some unpleasant effects - repeated glue code, multiple Null/None checks etc. 
In order to achieve more elegant solutions, in the examples above we used a design pattern with two functions (`unit` and `bind`). 
This pattern is called **Monad**. 
In essence, the `bind` function implements the glue code and `unit` implements the initialisation code. 
This allows us to solve the problem in one line:

```python
bind(bind( ... bind(bind(unit(x), f1), f2) ... fn-1), fn)
```

The following diagram displays the computational process:


<figure>
  <img src="/images/blog/Monads in 15 minutes/monad1.png" alt="Monad - Computational Model" >
  <figcaption>Monad - Computational Model.</figcaption>
</figure>

The `unit(x)` invocation generates an initial value `v1`. Then `bind(v1, f1)` generates a new interim value `v2`, 
which is then used in the subsequent call to bind - `bind(v2, f2)`. 
This continues until the final result is generated. Using this pattern, by using different `unit` and `bind` functions we can achieve various types of function compositions. 
Standard Monad libraries provide predefined sets of ready to use monads (`unit` and `bind` functions), which can be used "out of the box" to implement different kinds of composition.

In order to compose the `bind` and `unit` functions, the return types of `unit` and `bind`, and the type of `bind`'s first parameter must be compatible. 
This is called the Monadic Type. In terms of the previous computational diagram, the types of all interim values `v1`, `v2` ... `vn` must be Monadic.

Lastly, repeating the calls to `bind` again and again can be tedious and should be avoided. For the purpose we define an auxiliary function:

```python
def pipeline(e, *functions):
    for f in functions:
        e = bind(e, f)
    return e
```

Then instead of:

```python
bind(bind(bind(bind(unit(x), f1), f2), f3), f4)
```

We can use the shorthand:

```python
pipeline(unit(x), f1, f2, f3, f4)
```

# Conclusion

Monads are a simple and powerful design pattern for function composition. In a declarative language they can be used to implement features of imperative languages like logging and I/O. 
In an imperative language, they can reduce and isolate the bloated glue code between function invocations. 
This article only scratches the surface and builds an intuitive understanding of Monads. To learn more you can explore:

*   [Wikipedia](http://en.wikipedia.org/wiki/Monad_%28functional_programming%29)
*   [Monads in Python](http://www.valuedlessons.com/2008/01/monads-in-python-with-nice-syntax.html)
*   [List of Monad tutorials](http://www.haskell.org/haskellwiki/Monad_tutorials_timeline)
