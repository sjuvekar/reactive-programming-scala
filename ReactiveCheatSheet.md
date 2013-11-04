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
With these definition, and a basic definition of <code>integer</code> generator, we can map it to other domains like 
<h1>Monads</h1>
<h1>Futures</h1>
<h1>Observables</h1>
