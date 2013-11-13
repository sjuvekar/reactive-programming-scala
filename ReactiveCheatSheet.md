---
layout: page
title: Cheat Sheet
---

This cheat sheet originated from the forums. There are certainly a lot of things that can be improved! If you would like to contribute, you have two options:

Click the "Edit" button on this file on GitHub:
https://github.com/sjuvekar/reactive-programming-scala/edit/master/ReactiveCheatSheet.md
You can submit a pull request directly from there without checking out the git repository to your local machine.

Fork the repository https://github.com/sjuvekar/reactive-programming-scala/ and check it out locally. To preview your changes, you need jekyll. Navigate to your checkout and invoke jekyll --auto --server, then open the page http://localhost:4000/ReactiveCheatSheet.html.


<h1>Partial Functions</h1>
A subtype of trait <code>Function1</code> that is well defined on a subset of its domain.
```scala
  trait PartialFunction[-A, +R] extends Function1[-A, +R] {
    def apply(x: A): R
    def isDefinedAt(x: A): Boolean
  }
```
Every concrete implementation of PartialFunction has the usual <code>apply</code> method along with a boolean method <code>isDefinedAt</code>.

<b>Important:</b> An implementation of partialFunction can return <code>true</code> for <code>isDefinedAt </code> but still end up throwing Runtime Exception (like MatchException in pattern-matching implementation).
<h1>For-Comprehension and Pattern Matching</h1>
A general For-Comprehension is described in Scala Cheat Sheet here: https://github.com/lrytz/progfun-wiki/blob/gh-pages/CheatSheet.md One can also use Patterns inside for-expression. The simplest form of for-expression pattern looks like
```scala
for { pat <- expr} yield e
```
where <code>pat</code> is a pattern containing a single variable <code>x</code>. We translate the <code>pat <- expr</code> part of the expression to
```scala
x <- expr withFilter {
    case pat => true
    case _ => false
  } map {
    case pat => x
  }
```
The remaining parts are translated to <code> map, flatMap, withFilter</code> according to standard for-comprehension rules.
<h1>Random Generators with For-Expressions</h1>
The <code>map</code> and <code>flatMap</code> methods can be overridden to make a for-expression versatile, for example to generate random elements from an arbitrary collection like lists, sets etc. Define the following trait <code>Generator</code> to do this.
```scala
trait Generator[+T] {
  self =>
  def generate: T
  def map[S](f: T => S) : Generator[S] = new Generator[S] {
    def generate = f(self.generate)
  }
  def flatMap[S](f: T => Generator[S]) : Generator[S] = new Generator[S] {
    def generate = f(self.generate).generate
  }
}
```
Let's define a basic integer random generator as 
```scala
val integers = new Generator[Int] {
  val rand = new java.util.Random
  def generate = rand.nextInt()
}
````
With these definition, and a basic definition of <code>integer</code> generator, we can map it to other domains like <code>booleans, pairs, intervals</code> using for-expression magic
```scala
val booleans = for {x <- integers} yield x > 0
val pairs = for {x <- integers; y<- integers} yield (x, y)
def interval(lo: Int, hi: Int) : Generator[Int] = for { X <- integers } yield lo + x % (hi - lo)
```
<h1>Monads</h1>
A monad is a parametric type M[T] with two operations: <code>flatMap</code> and <code>unit</code>. 
```scala
trait M[T] {
  def flatMap[U](f: T => M[U]) : M[U]
  def unit[T](x: T) : M[T]
}
```
These operations must satisfy three important properties:
<ol>
  <li><b>Associativity:</b> <code>(x flatMap f) flatMap g == x flatMap (y => f(y) flatMap g)</code></li>
  <li><b>Left unit:</b> <code>unit(x) flatMap f == f(x)</code></li>
  <li><b>Right unit:</b> <code>m flatMap unit == m</code></li>
</ol>
Many standard Scala Objects like <code>List, Set, Option, Gen</code> are monads with identical implementation of <code>flatMap</code> and specialized implementation of <code>unit</code>. An example of non-monad is a special <code>Try</code> object that fails with a non-fatal exception because it fails to satisfy Left unit (See lectures). 
<h3>Monads and For-Expression</h3>
Monads help simplify for-expressions. 

<b>Associativily</b> helps us "inline" nested for-expressions and write something like 
```scala  
  for { x <- e1; y <- e2(x) ... }
```
<b>Right unit</b> helps us eliminate for-expression using the identity 
```scala
  for{x <- m} yield x == m
```

<h1>Pure functional programming</h1>

In a pure functional state, programs are side-effect free, and the concept of time isn't important (i.e. redoing the same steps in the same order produces the same result).

When evaluating a pure functional expression using the substitution model, no matter the evaluation order of the various sub-expressions, the result will be the same (some ways may take longer than others). An exception may be in the case where a aub-expression is never evaluated (e.g. second argument) but whose evaluation would loop forever.

<h1>Mutable state</h1>

In a reactive system, some states eventually need to be changed in a mutable fashion. An object has a state if its behavior has a history. Every form of mutable state is constructed from variables:

    var x: String = "abc"
    x = "hi"
    var nb = 42

The use of a stateful expression can complexify things. For a start, the evaluation order may matter. Also, the concept of identity and change gets more complex. When are two expressions considered the same? In the following (pure functional) example, x and y are always the same (concept of <b>referential transparency</b>):

    val x = E; val y = E
    val x = E; val y = x

But when a stateful variable is involved, the concept of equality is not as straightforward. "Being the same" is defined by the property of <b>operational equivalence</b>. x and y are operationally equivalent if no possible test can distinsuish between them.

Considering two variables x and y, if you can create a function f so that f(x, y) returns a different result than f(x, x) then x and y are different. If no such function exist x and y are the same.

As a consequence, the substitution model ceases to be valid when using assignments.

<h1> Loops </h1>
Variables and assignments are enough to model all programs with mutable states and loops in essence are not required. <b>Loops can be modeled using functions and lazy evaluation</b>. So, the expression
```scala
    while (condition) { command }
```
can be modeled using function <tt>WHILE</tt> as
```scala
    def WHILE(condition: => Boolean)(command: => Unit): Unit = 
        if (condition) {
            command
            WHILE(condition)(command)
        }
        else ()
```
<b>Note:</b> 
<ul>
    <li> Both <b> condition </b> and <b> command </b> are <b> passed by name </b></li>
    <li> <b>WHILE</b> is <b>tail recursive</b></li>
</ul>
    

<h2>For loop</h2>
The treatment of for loops is similar to the <b>For-Comprehensions</b> commonly used in functional programming. The general expression for <tt>for loop</tt> equivalent in Scala is 
```scala
    for(v1 <- e1; v2 <- e2; ...; v_n <- e_n) command
```
Note a few subtle differences from a For-expreesion. There is no <tt>yield</tt> expression, <tt>command</tt> can contain mutable states and <tt>e1, e2, ..., e_n</tt> are expressions over arbitrary Scala collections. This for loop is translated by Scala using a <b>foreach</b> combinator defined over any arbitrary collection. The signature for <b>foreach</b> over collection <b>T</b> looks like this
```scala
def foreach(f: T => Unit) : Unit
```
Using foreach, the general for loop is recursively translated as follows:
```scala
    for(v1 <- e1; v2 <- e2; ...; v_n <- e_n) command = 
        e1 foreach (v1 => for(v2 <- e2; ...; v_n <- e_n) command)
```
