<h1>Partial Functions</h1>
```
  trait PartialFunction[-A, +R] extends Function1[-A, +R] {
    def apply(x: A): R
    def isDefinedAt(x: A): Boolean
  }
```
<h1>Generators</h1>
<h1>Monads</h1>
