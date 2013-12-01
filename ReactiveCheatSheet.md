---
layout: page
title: Reactive Cheat Sheet
---

This cheat sheet originated from the forums. There are certainly a lot of things that can be improved! If you would like to contribute, you have two options:

Click the "Edit" button on this file on GitHub:
[https://github.com/sjuvekar/reactive-programming-scala/blob/master/ReactiveCheatSheet.md](https://github.com/sjuvekar/reactive-programming-scala/edit/master/ReactiveCheatSheet.md)
You can submit a pull request directly from there without checking out the git repository to your local machine.

Fork the repository [https://github.com/sjuvekar/reactive-programming-scala/](https://github.com/sjuvekar/reactive-programming-scala/) and check it out locally. To preview your changes, you need jekyll. Navigate to your checkout and invoke `jekyll serve --watch` (or `jekyll --auto --server` if you have an older jekyll version), then open the page `http://localhost:4000/ReactiveCheatSheet.html`.


## Partial Functions

A subtype of trait `Function1` that is well defined on a subset of its domain.

    trait PartialFunction[-A, +R] extends Function1[-A, +R] {
      def apply(x: A): R
      def isDefinedAt(x: A): Boolean
    }

Every concrete implementation of PartialFunction has the usual `apply` method along with a boolean method `isDefinedAt`.

**Important:** An implementation of PartialFunction can return `true` for `isDefinedAt` but still end up throwing RuntimeException (like MatchError in pattern-matching implementation).

A concise way of constructing partial functions is shown in the following example:

    trait Coin {}
    case class Gold() extends Coin {}
    case class Silver() extends Coin {}
    
    val pf: PartialFunction[Coin, String] = {
      case Gold() => "a golden coin"
      // no case for Silver(), because we're only interested in Gold()
    }
    
    println(pf.isDefinedAt(Gold()))   // true 
    println(pf.isDefinedAt(Silver())) // false
    println(pf(Gold()))               // a golden coin
    println(pf(Silver()))             // throws a scala.MatchError


## For-Comprehension and Pattern Matching

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


## Random Generators with For-Expressions

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


## Monads

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


## Monads and For-Expression

Monads help simplify for-expressions. 

**Associativily** helps us "inline" nested for-expressions and write something like

    for { x <- e1; y <- e2(x) ... }

**Right unit** helps us eliminate for-expression using the identity

    for{x <- m} yield x == m


## Pure functional programming

In a pure functional state, programs are side-effect free, and the concept of time isn't important (i.e. redoing the same steps in the same order produces the same result).

When evaluating a pure functional expression using the substitution model, no matter the evaluation order of the various sub-expressions, the result will be the same (some ways may take longer than others). An exception may be in the case where a sub-expression is never evaluated (e.g. second argument) but whose evaluation would loop forever.


## Mutable state

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


## Loops

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


## For loop

The treatment of for loops is similar to the <b>For-Comprehensions</b> commonly used in functional programming. The general expression for <tt>for loop</tt> equivalent in Scala is

    for(v1 <- e1; v2 <- e2; ...; v_n <- e_n) command

Note a few subtle differences from a For-expreesion. There is no `yield` expression, `command` can contain mutable states and `e1, e2, ..., e_n` are expressions over arbitrary Scala collections. This for loop is translated by Scala using a **foreach** combinator defined over any arbitrary collection. The signature for **foreach** over collection **T** looks like this

    def foreach(f: T => Unit) : Unit

Using foreach, the general for loop is recursively translated as follows:

    for(v1 <- e1; v2 <- e2; ...; v_n <- e_n) command = 
        e1 foreach (v1 => for(v2 <- e2; ...; v_n <- e_n) command)


## Monads and Effect

Monads and their operations like flatMap help us handle programs with side-effects (like exceptions) elegantly. This is best demonstrated by a Try-expression. <b>Note: </b> Try-expression is not strictly a Monad because it does not satisfy all three laws of Monad mentioned above. Although, it still helps handle expressions with exceptions. 


#### Try

The parametric Try class as defined in Scala.util looks like this:

    abstract class Try[T]
    case class Success[T](elem: T) extends Try[T]
    case class Failure(t: Throwable) extends Try[Nothing]

`Try[T]` can either be `Success[T]` or `Failure(t: Throwable)`

    import Scala.util.{Try, Success, Failure}

    def answerToLife(nb: Int) : Try[Int] = {
      if (nb == 42) Success(nb)
      else Failure(new Exception("WRONG"))
    }

    answerToLife(42) match {
      case Success(t)           => t        // returns 42
      case failure @ Failure(e) => failure  // returns Failure(java.Lang.Exception: WRONG)
    }


Now consider a sequence of scala method calls:

    val o1 = SomeTrait()
    val o2 = o1.f1()
    val o3 = o2.f2()

All of these method calls are synchronous, blocking and the sequence computes to completion as long as none of the intermediate methods throw an exception. But what if one of the methods, say `f2` does throw an exception? The `Try` class defined above helps handle these exceptions elegantly, provided we change return types of all methods `f1, f2, ...` to `Try[T]`. Because then, the sequence of method calls translates into an elegant for-comprehension:

    val o1 = SomeTrait()
    val ans = for {
        o2 <- o1.f1();
        o3 <- o2.f2()
    } yield o3

This transformation is possible because `Try` satisfies 2 properties related to `flatMap` and `unit` of a **monad**. If any of the intermediate methods `f1, f2` throws and exception, value of `ans` becomes `Failure`. Otherwise, it becomes `Success[T]`.


## Monads and Latency

The Try Class in previous section worked on synchronous computation. Synchronous programs with side effects block the subsequent instructions as long as the current computation runs. Blocking on expensive computation might render the entire program slow!. **Future** is a type of monad the helps handle exceptions and latency and turns the program in a non-blocking asynchronous program.


#### Future

Future trait is defined in scala.concurrent as:

    trait Future[T] {
        def onComplete(callback: Try[T] => Unit)
        (implicit executor: ExecutionContext): Unit
    }

The Future trait contains a method `onComplete` which itself takes a method, `callback` to be called as soon as the value of current Future is available. The insight into working of Future can be obtained by looking at its companion object:

    object Future {
        def apply(body: => T)(implicit context: ExecutionContext): Future[T]
    }

This object has an apply method that starts an asynchronous computation in current context, returns a `Future` object. We can then subsribe to this `Future` object to be notified when the computation finishes.

    import scala.concurrent._
    import ExecutionContext.Implicits.global

    // The function to be run asyncronously
    val answerToLife: Future[Int] = future {
      42
    }

    // These are various callback functions that can be defined  
    answerToLife onComplete {
      case Success(result) => result
      case Failure(t) => println("An error has occured: " + t.getMessage)
    }

    answerToLife onSuccess {
      case result => result
    }
  
    answerToLife onFailure {
      case t => println("An error has occured: " + t.getMessage)
    }
    
    answerToLife.now    // only works if the future is completed


#### Combinators on Future

A `Future` is a `Monad` and has `map`, `filter`, `flatMap` defined on it. In addition, Scala's Futures define two additional methods:

    def recover(f: PartialFunction[Throwable, T]): Future[T]
    def recoverWith(f: PartialFunction[Throwable, Future[T]]): Future[T]

These functions return robust features in case current features fail. 

Finally, a `Future` extends from a trait called `Awaitable` that has two blocking methods, `ready` and `result` which take the value 'out of' the Future. The signatures of these methods are

    trait Awaitable[T] extends AnyRef {
        abstract def ready(t: Duration): Unit
        abstract def result(t: Duration): T
    }

Both these methods block the current execution for a duration of `t`. If the future completes its execution, they return: `result` returns the actual value of the computation, while `ready` returns a Unit. If the future fails to complete within time t, the methods throw a `TimeoutException`.

`Await` can be used to wait for a future with a specified timeout, e.g.

    userInput: Future[String] = ...
    Await.result(userInput, 10 seconds)   // waits for user input for 10 seconds, after which throws a TimeoutException


#### async and await

Async and await allow to run some part of the code aynchronously. The following code computes asynchronously any future inside the `await` block

    import scala.async.Async._

    def retry(noTimes: Int)(block: => Future[T]): Future[T] = async {
      var i = 0;
      var result: Try[T] = Failure(new Exception("Problem!"))
      while (i < noTimes && result.isFailure) {
        result = await { Try(block) }
        i += 1
      }
      result.get
    }


#### Promises

Another way to create future is through the Promise monad:

    trait Promise[T]
      def future: Future[T]
      def complete(result: Try[T]): Unit
      def tryComplete(result: Try[T]): Boolean
    }

It is used as follows:

    val p = Promise[T]  // defines a promise
    p.future            // returns a future that will complete when p.complete() is called
    p.complete(Try[T])  // completes the future
    p success T         // successfully completes the promise

## Observables
Observables are asynchronous streams of data. Contrary to Futures, they can return multiple values.

    trait Observable[T] {
      def Subscribe(observer: Observer[T]): Subscription
    }

    trait Observer[T] {
      def onNext(value: T): Unit            // callback function when there's a next value
      def onError(error: Throwable): Unit  // callback function where there's an error
      def onCompleted(): Unit              // callback function when there is no more value
    }

    trait Subscription {
      def unsubscribe(): Unit    // to call when we're not interested in recieving any more values
    }

Observables can be used as follows:

    import rx.lang.scala._

    val ticks: Observable[Long] = Observable.interval(1 seconds)
    val evens: Observable[Long] = ticks.filter(s => s%2 == 0)
    
    val bugs: Observable[Seq[Long]] = ticks.buffer(2, 1)
    val s = bugs.subscribe(b=>println(b))

    s.unsubscribe()

Some observable functions (see more at http://rxscala.github.io/scaladoc/index.html#rx.lang.scala.Observable):

- `Observable[T].flapmap(T => Observable[T]): Observable[T]` merges a list of observables into a single observable in a non-deterministic order
- `Observable[T].concat(T => Observable[T]): Observable[T]` merges a list of observables into a single observable, putting the results of the first observable first, etc.
- `groupBy[K](keySelector: T=>K): Observable[(K, Observable[T])]` returns an observable of observables, where the element are grouped by the key returned by `keySelector`

#### Subscriptions

Subscriptions are returned by Observables to allow to unsubscribe. With hot observables, all subscribers share the same source, which produces results independent of subscribers. With cold observablesm each subscriber has its own private source. If there is no subscriber no computation is performed.

Subscriptions have several subtypes: `BooleanSuscription` (was the subscription unsubscribed or not?), `CompositeSubscription` (collection of subscriptions that will be unsubscribed all at once), `MultipleAssignmentSubscription` (always has a single subscription at a time)

    val subscription = Subscription { println("Bye") }
    subscription.unsubscribe() // prints the message
    subscription.unsubscribe() // doesn't print it again

#### Creating Rx Streams

Using the following constructor that takes an Observer and returns a Subscription

    object Observable {
      def apply[T](subscribe: Observer[T] => Subscription): Observable[T]
    }

It is possible to create several observables. The following functions suppose they are part of an Observable type (calls to `subscribe(...)` implicitely mean `this.subscribe(...)`):

    // Observable never: never returns anything
    def never(): Observable[Nothing] = Observable[Nothing](observer => { Subscription {} })

    // Observable error: returns an error
    def apply[T](error: Throwable): Observable[T] =
        Observable[T](observer => {
        observer.onError(error)
        Subscription {}
      }
    )

    // Observable startWith: prepends some elements in front of an Observable
    def startWith(ss: T*): Observable[T] = {
        Observable[T](observer => {
        for(s <- ss) observer onNext(s)
        subscribe(observer)
      })
    })

    // filter: filters results based on a predicate
    def filter(p: T=>Boolean): Observable[T] = {
      Observable[T](observer => {
        subscribe(
          (t: T) => { if(p(t)) observer.onNext(t) },
          (e: Thowable) => { observer.onError(e) },
          () => { observer.onCompleted() }
        )
      })
    }

    // map: create an observable of a different type given a mapping function
    def map(f: T=>S): Observable[S] = {
      Observable[S](observer => {
        subscribe(
          (t: T) => { observer.onNext(f(t)) },
          (e: Thowable) => { observer.onError(e) },
          () => { observer.onCompleted() }
        )
      }
    }

    // Turns a Future into an Observable with just one value
    def from(f: Future[T])(implicit execContext: ExecutionContext): Observable[T] = {
        val subject = AsyncSubject[T]()
        f onComplete {
          case Failure(e) => { subject.onError(e) }
          case Success(c) => { subject.onNext(c); subject.onCompleted() }
        }
        subject
    }
    

#### Blocking Observables

`Observable.toBlockingObservable()` returns a blocking observable (to use with care). Everything else is non-blocking.

    val xs: Observable[Long] = Observable.interval(1 second).take(5)
    val ys: List[Long] = xs.toBlockingObservable.toList

#### Schedulers

Schedulers allow to run a block of code in a separate thread. The Subscription returned by its constructor allows to stop the scheduler.

    trait Observable[T] {
      def observeOn(scheduler: Scheduler): Observable[T]
    }

    trait Scheduler {
      def schedule(work: => Unit): Subscription
      def schedule(work: Scheduler => Subscription): Subscription
      def schedule(work: (=>Unit)=>Unit): Subscription
    }

    val scheduler = Scheduler.NewThreadScheduler
    val subscription = scheduler.schedule { // schedules the block on another thread
      println("Hello world")
    }


    // Converts an iterable into an observable
    // works even with an infinite iterable
    def from[T](seq: Iterable[T])
               (implicit scheduler: Scheduler): Obserable[T] = {
       Observable[T](observer => {
         val it = seq.iterator()
         scheduler.schedule(self => {     // the block between { ... } is run in a separate thread
           if (it.hasnext) { observer.onNext(it.next()); self() } // calling self() schedules the block of code to be executed again
           else { observer.onCompleted() }
         })
       })
    }
