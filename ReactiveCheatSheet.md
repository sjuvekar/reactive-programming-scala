<h1>Partial Functions</h1>
A subtype of trait <code>Function1</code> that is well defined on a subset of its domain.
```scala
  trait PartialFunction[-A, +R] extends Function1[-A, +R] {
    def apply(x: A): R
    def isDefinedAt(x: A): Boolean
  }
```
Every concrete implementation of PartialFunction has the usual <code>apply</code> method along with a boolean method <code>isDefinedAt</code>.

<b>Important:</b> An implementation of partialFunction can return <code> true </code> for <code>isDefinedAt </code> but still end up throwing Runtime Exception (like MatchException in pattern-matching implementation).
<h1>For-Comprehension and Pattern Matching</h1>
A general For-Comprehension is described in Scala Cheat Sheet here: https://github.com/lrytz/progfun-wiki/blob/gh-pages/CheatSheet.md One can also use Patterns inside for-expression. The simplest form of for-expression pattern looks like
```scala
for { pat <- expr} yield e
```
where <code>pat</code> is a pattern containing a single variable <code>x</code>.
<h1>Generators</h1>
<h1>Monads</h1>
<h1>Futures</h1>
<h1>Observables</h1>
