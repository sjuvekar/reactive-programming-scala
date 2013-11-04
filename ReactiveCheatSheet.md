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
trait Generator[T+] {
  self =>
  def generate: T
  def map[S}(f: T => S) : Generator[S] = new Generator[S] {
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
Many standard Scala Objects like <code>List, Set, Option, Gen</code> are monads with identical implementation of <code>flatMap</code> and specialized implementation of <code>unit</code>. An example of non-monad is a special <code>Try</code> object that fails with a non-fatal exception (See lectures). 
<h2>Monads and For-Expression</h2>
<h1>Futures</h1>
<h1>Observables</h1>
