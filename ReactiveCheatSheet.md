---
layout: page
title: Reactive Cheat Sheet
---

This cheat sheet originated from the forums. There are certainly a lot of things that can be improved! If you would like to contribute, you have two options:

Click the "Edit" button on this file on GitHub:
[https://github.com/sjuvekar/reactive-programming-scala/blob/master/ReactiveCheatSheet.md](https://github.com/sjuvekar/reactive-programming-scala/edit/master/ReactiveCheatSheet.md)
You can submit a pull request directly from there without checking out the git repository to your local machine.

Fork the repository [https://github.com/sjuvekar/reactive-programming-scala/](ttps://github.com/sjuvekar/reactive-programming-scala/) and check it out locally. To preview your changes, you need jekyll. Navigate to your checkout and invoke jekyll --auto --server, then open the page http://localhost:4000/ReactiveCheatSheet.html.


# Partial Functions #

A subtype of trait `Function1` that is well defined on a subset of its domain.

    trait PartialFunction[-A, +R] extends Function1[-A, +R] {
      def apply(x: A): R
      def isDefinedAt(x: A): Boolean
    }

Every concrete implementation of PartialFunction has the usual `apply` method along with a boolean method `isDefinedAt`.

**Important:** An implementation of partialFunction can return `true` for `isDefinedAt` but still end up throwing RuntimeException (like MatchException in pattern-matching implementation).

# For-Comprehension and Pattern Matching#

A general For-Comprehension is described in Scala Cheat Sheet here: https://github.com/lrytz/progfun-wiki/blob/gh-pages/CheatSheet.md One can also use Patterns inside for-expression. The simplest form of for-expression pattern looks like

    for { pat <- expr} yield e

where `pat` is a pattern containing a single variable `x`. We translate the `pat <- expr` part of the expression to

    x <- expr withFilter {
      case pat => true
      case _ => false
    } map {
      case pat => x
    }

The remaining parts are translated to ` map, flatMap, withFilter` according to standard for-comprehension rules.

# Random Generators with For-Expressions #

The `map` and `flatMap` methods can be overridden to make a for-expression versatile, for example to generate random elements from an arbitrary collection like lists, sets etc. Define the following trait `Generator` to do this.

    trait Generator[+T] { self =>
      def generate: T
      def map[S](f: T => S) : Generator[S] = new Generator[S] {
        def generate = f(self.generate)
      }
      def flatMap[S](f: T => Generator[S]) : Generator[S] = new Generator[S] {
        def generate = f(self.generate).generate
      }
    }
    
Let's define a basic integer random generator as 

    val integers = new Generator[Int] {
      val rand = new java.util.Random
      def generate = rand.nextInt()
    }

With these definition, and a basic definition of `integer` generator, we can map it to other domains like `booleans, pairs, intervals` using for-expression magic

    val booleans = for {x <- integers} yield x > 0
    val pairs = for {x <- integers; y<- integers} yield (x, y)
    def interval(lo: Int, hi: Int) : Generator[Int] = for { X <- integers } yield lo + x % (hi - lo)

# Monads #

A monad is a parametric type M[T] with two operations: `flatMap` and `unit`. 

    trait M[T] {
      def flatMap[U](f: T => M[U]) : M[U]
      def unit[T](x: T) : M[T]
    }

These operations must satisfy three important properties:

1. **Associativity:** `(x flatMap f) flatMap g == x flatMap (y => f(y) flatMap g)`
2. **Left unit:** `unit(x) flatMap f == f(x)`

3. **Right unit:** `m flatMap unit == m`

Many standard Scala Objects like `List, Set, Option, Gen` are monads with identical implementation of `flatMap` and specialized implementation of `unit`. An example of non-monad is a special `Try` object that fails with a non-fatal exception because it fails to satisfy Left unit (See lectures). 

# Monads and For-Expression #

Monads help simplify for-expressions. 

**Associativily** helps us "inline" nested for-expressions and write something like

    for { x <- e1; y <- e2(x) ... }

**Right unit** helps us eliminate for-expression using the identity

    for{x <- m} yield x == m

# Pure functional programming #

In a pure functional state, programs are side-effect free, and the concept of time isn't important (i.e. redoing the same steps in the same order produces the same result).

When evaluating a pure functional expression using the substitution model, no matter the evaluation order of the various sub-expressions, the result will be the same (some ways may take longer than others). An exception may be in the case where a sub-expression is never evaluated (e.g. second argument) but whose evaluation would loop forever.

# Mutable state #

In a reactive system, some states eventually need to be changed in a mutable fashion. An object has a state if its behavior has a history. Every form of mutable state is constructed from variables:

    var x: String = "abc"
    x = "hi"
    var nb = 42

The use of a stateful expression can complexify things. For a start, the evaluation order may matter. Also, the concept of identity and change gets more complex. When are two expressions considered the same? In the following (pure functional) example, x and y are always the same (concept of <b>referential transparency</b>):

    val x = E; val y = E
    val x = E; val y = x

But when a stateful variable is involved, the concept of equality is not as straightforward. "Being the same" is defined by the property of **operational equivalence**. x and y are operationally equivalent if no possible test can distinsuish between them.

Considering two variables x and y, if you can create a function f so that f(x, y) returns a different result than f(x, x) then x and y are different. If no such function exist x and y are the same.

As a consequence, the substitution model ceases to be valid when using assignments.

# Loops #

Variables and assignments are enough to model all programs with mutable states and loops in essence are not required. <b>Loops can be modeled using functions and lazy evaluation</b>. So, the expression

    while (condition) { command }

can be modeled using function <tt>WHILE</tt> as

    def WHILE(condition: => Boolean)(command: => Unit): Unit = 
        if (condition) {
            command
            WHILE(condition)(command)
        }
        else ()

**Note:**
* Both **condition** and **command** are **passed by name**
* **WHILE** is **tail recursive**

## For loop ##

The treatment of for loops is similar to the <b>For-Comprehensions</b> commonly used in functional programming. The general expression for <tt>for loop</tt> equivalent in Scala is

    for(v1 <- e1; v2 <- e2; ...; v_n <- e_n) command

Note a few subtle differences from a For-expreesion. There is no `yield` expression, `command` can contain mutable states and `e1, e2, ..., e_n` are expressions over arbitrary Scala collections. This for loop is translated by Scala using a **foreach** combinator defined over any arbitrary collection. The signature for **foreach** over collection **T** looks like this

    def foreach(f: T => Unit) : Unit

Using foreach, the general for loop is recursively translated as follows:

    for(v1 <- e1; v2 <- e2; ...; v_n <- e_n) command = 
        e1 foreach (v1 => for(v2 <- e2; ...; v_n <- e_n) command)
        
### Monads and Effect ###
