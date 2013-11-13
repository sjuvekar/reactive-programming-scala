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
Using foreach, the general for loop is translated as follows:
```scala
    for(v1 <- e1; v2 <- e2; ...; v_n <- e_n) command = e1 foreach (v1 => for(v2 <- e2; ...; v_n <- e_n) command)
```
