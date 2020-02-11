---
layout: post
title: FPiS Getting Started
date: 2016-04-29
#[taxonomies]
#tags = ["scala", "functional programming", "types", "functions"]
---

The first two chapters of **Functional Programming in Scala** (_"FPiS"_ from now on) contain introduction material to both Scala and functional programming.

Glossing over the introduction about the Scala language, I found much interesting and enlightening the introduction the reader is getting about functional programming.

In the first two chapters, we are making our acquaintance with two of the pillars in FP: **pure functions** and **referential transparency**. Before we can talk about functions, we need to introduce another relevant component: **types** (at least, we will use them in the context of statically typed languages, like Scala).

*Types* are collection (or _Set_) of values. Scala is strongly typed programming language and basically every expression has a type. The compiler is checking whether a given program is correctly typed, and eventually discard unsound programs with one (or more) type errors.

The jury is still deciding whether types are an unnecessary complication or a nice helper in writing correct programs. In my opionion, the idea to discover type errors before the program is actually executed is a great feature to have, as it reduce the number of things that can eventually go wrong. On the other hand, from time to time we have to overcome the frustation of weird compiler error messages.

With types, we can define a function `f` from `A` to `B`:

```scala
val f: A => B
```

`f` is the function which associates with each element of `A` a unique member of type `B`.

In FP functions are values, we call them *High-order functions* when they receive functions as input and/or produce functions as output. The function `both` is a function that receive a function `f` as input and returns a new function that it will apply `f` twice:

```scala
def both[A](f: A => A): A => A = f andThen f
```

```
scala> val doubleMe = (x: Int) => x * 2
doubleMe: Int => Int = <function1>

scala> both(doubleMe)
res0: Int => Int = <function1>

scala> both(doubleMe)(2)
res1: Int = 8
```

Scala has a shorted syntax to define functions inline, the only issue is with the type inference machinery. Providing the `both` function with a sensible type we can then define the same expression in this way:

```
scala> val fourTimes = both[Int](_ * 2)
fourTimes: Int => Int = <function1>
```

Functions are true values, and as such we can do something like store them in a list:

```
scala> val l: List[Int => Int] = List(_ * 2, _ + 42)
l: List[Int => Int] = List(<function1>, <function1>)

scala> val l2 = (1 to 10).map(v => (x: Int) => x + v).toList
l2: List[Int => Int] = List(<function1>, <function1>, <function1>, <function1>, <function1>, <function1>, <function1>, <function1>, <function1>, <function1>)
```

now the list `l2` contains 10 functions (`[(+1), (+2), (+3), ...]`), with a couple of list methods (`zip` and `collect`) we can first create a list of pairs `(Int, Int => Int)`. With `collect` we apply the first value in the pair to the corresponding function:

```
scala> (List.fill(10)(42) zip l2).collect { case (v, f) => f.apply(v) }
res2: List[Int] = List(43, 44, 45, 46, 47, 48, 49, 50, 51, 52)
```

after the values are applied to the functions we are back to a list of `Int`.

Another interesting example is the **Newton's method** for finding the square root of a number. With this procedure we are trying to improve our guess until the result is good enough.

```
sqrt(n) ≥ 0
  abs(sqrt(x)^2 - x) < eps
```

`eps` represents formally our expectation about the result precision.

```scala
def abs(x: Double): Double = if(x < 0) -x else x

def until[A](cond: A => Boolean)(op: A => A, x: A): A = {
  if (cond(x))
    x
  else
    until(cond)(op, op(x))
}

def sqrt(x: Int): Double = {
  val eps: Double = 0.00001
  def satis(y: Double): Boolean = {
    abs(math.pow(y, 2) - x) < eps
  }
  def improve(y: Double): Double = (y + x / y) / 2.0
  until(satis)(improve, x)
}
```

If we now try this function we will get an aproximation for the square root of 2:

```
scala> sqrt(2)
res3: Double = 1.4142156862745097
```

One of the benefit of the FP mindset is the chance to reduce the overall problem (ie computing the square root) to smaller and simpler subproblems. We are modelling these subproblems with short functions that have a specific task to perform. Even in this small example, the juice of the `sqrt` function reads like an English sentence:

	 until satis improve x

We basically trade loops and indexes with a more abstract and high level description of the approach we are applying in the problem solution.

And we can make this function even more general, the Netwon's method has more broad use in finding approximations for a function root.

```scala
def deriv(f: Double=>Double, x: Double): Double = {
  val dx: Double = 0.0001
  (f(x + dx) - f(x)) / dx
}

def newton(f: Double => Double): Double => Double = {
  val eps: Double = 0.00001
  def satis(y: Double) = abs(f(y)) < eps
  def improve(y: Double) = y - (f(y) / deriv(f, y))

  until(satis)(improve, _: Double)
}

def sqrt(x: Double): Double = newton(y => math.pow(y, 2) - x)(x)
```

If we run this function we will get a similar result:

```
scala> sqrt(2.0)
res4: Double = 1.414215778910855

scala> sqrt(9.0)
res5: Double = 3.000000002944079
```

Using only primitive types in our functions leave us in a sort of _blindness state_. The relation between **functions** and **types** is somehow based on false assumptions or even worse pure lies. The `sqrt` function is not a **total function**, in fact it is undefined for negative numbers. This constraint was specified in the function specification, but we left out this important aspect of `sqrt` from the type system. In real world programs this information (if we are lucky, or not) will end up in a fancy comment, but is this enough?

In general, we have a strong preference over total functions because it's easier reasoning about them. In case the function is not total, not all hope is lost. We could encode this function property in the type system.

If we go back to `sqrt`, this function has the following type:

```scala
sqrt: Double => Double
```

As alredy mentioned, `sqrt` will fail orribily when applied to negative numbers, and it will fail in the worst way possible (ie the recursion never terminates). In literature, this kind of result from a function is called a **bottom value**. We could receive an error (like a JVM `Exception`), or the function never completes its computation: both these situations are hidden in the function type.

Types can be _compilable documentation_ when they are used in the right way. Let's define a function called `divideBy`:

```
scala> def divideBy(x: Int) = (y: Int) => y / x
divideBy: (x: Int)Int => Int
```

the type of `divideBy` is inferred to be `Int => Int`. And this looks fine until we try something acceptable:

```
scala> val divBy4 = divideBy(4)
divBy4: Int => Int = <function1>

scala> divBy4(16)
res6: Int = 4

scala> divBy4(32)
res7: Int = 8
```

It is quite easy to get back a function that is not total, like when we apply the value `0`:

```
scala> val divBy0 = divideBy(0)
divBy0: Int => Int = <function1>
```

the type for the function `divBy0` is `Int => Int`, no matter what number we are applying to this function we are not getting back an `Int`:

```
scala> divBy0(42)
java.lang.ArithmeticException: / by zero
  at $anonfun$divideBy$1.apply$mcII$sp(<console>:11)
  ... 32 elided

scala> divBy0(4)
java.lang.ArithmeticException: / by zero
  at $anonfun$divideBy$1.apply$mcII$sp(<console>:11)
  ... 32 elided
  ```

We could make an attempt to make the types better. We can start defining a new value type. In Scala value types have a little overhead over normal primitive types:

```scala
case class NonZeroNum private(val x: Int) extends AnyVal

object NonZeroNum {
  implicit def nonZeroNum2Int(n: NonZeroNum): Int = n.x
  def fromInt(x: Int): NonZeroNum = {
    if(x == 0) throw new Exception("invalid value")
    new NonZeroNum(x)
  }
}
```

we still have the nasty `Exception` around, but this time we have only the chance to receive this error when we create new values for `NonZeroNum`.

With this new type we can redefine our function `divideBy`:

```
scala> def divideBy(x: NonZeroNum) = (y: Int) => y / x
divideBy: (x: NonZeroNum)Int => Int
```

the new signature is:

```
NonZeroNum => (Int => Int)
```

using the function, the only difference is we need a `NonZeroNum` value:

```
scala> val one = NonZeroNum.fromInt(1)
one : NonZeroNum = NonZeroNum(1)

scala> divideBy(one)(12)
res8: Int = 12
```

## References

- _Paul Chiusano, Rúnar Bjarnason_. **Functional Programming in Scala**. 2014. Manning Publications.
- _Richard Bird, Philip Wadler_. **Introduction to Functional Programming (1st edition)**. 1988. Prentice Hall International.
- _John Hughes_. **Why functional programming matters**. The computer journal 32.2 (1989): 98-107.
- _Scott Wlaschin_. **Functional programming design patterns** (YouTube video). 2014. Retrieved from [here](https://www.youtube.com/watch?v=E8I19uA-wGY).
