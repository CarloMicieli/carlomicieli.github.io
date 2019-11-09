---
layout: post
title: Fun With Functions in Scala
date: 2015-07-25
slug = "/2015/07/25/fun-with-functions-in-scala/"
#[taxonomies]
#tags = ["scala", "functional programming", "functions"]
---

In functional programming **functions** are the basic building block. Scala is a hybrid language where _imperative_ and _functional_ programming paradigms live side by side, it's also targeting the JVM, and until Java 8 will rule the land, functions must behave nicely in the bytecode produced by the compiler.

Nonetheless, functions have first class support in Scala, and we have a nice literal syntax to define them. To see a basic example it is enough to fire up the REPL:

```
[carlo:~] $ scala
Welcome to Scala version 2.11.7 (Java HotSpot(TM) 64-Bit Server VM, Java 1.8.0_45).
Type in expressions to have them evaluated.
Type :help for more information.
```

To define a basic function that doubles a number we write:

```
scala> val doubleMe: (Int) => Int = x => x * 2
doubleMe: Int => Int = <function1>
```  

The REPL output is telling us that `doubleMe` is a function that takes an `Int` as argument and produces another `Int`.

To apply a value to the function `doubleMe` we are simpling use the usual syntax to call a function:

```
scala> doubleMe(21)
res0: Int = 42
```

`Function1[A]` is the _trait_ for all the functions `A => A`. This particular trait receives a special treatment that allows to compose functions.

```
scala> val doubleMe: (Int) => Int = x => x * 2
doubleMe: Int => Int = <function1>
```

Let's define another function, `plusOne`, in this case we will use a more traditional syntax: when we define a method in the REPL the method will be _lifted_ to a function:

```
scala> val plusOne: Int => Int = x => x + 1
plusOne: (x: Int)Int
```

At this point we can compose two functions. To apply `plusOne` and then `doubleMe` we could write:

```
scala> val f = plusOne andThen doubleMe
f: Int => Int = <function1>

scala> f(20)
res1: Int = 42
```

we are getting back another function with the type `Int => Int`.

Using the method `compose` we would apply `doubleMe` before the function `plusOne` is applied:

```
scala> val g = plusOne compose doubleMe
g: Int => Int = <function1>

scala> g(20)
res2: Int = 41
```

- `f(20)`: `doubleMe(plusOne(20)) ~> doubleMe(21) ~> 42`
- `g(20)`: `plusOne(doubleMe(20)) ~> plusOne(40) ~> 41`

REPL is acting like a big container where our code can live in, so we can define functions like normal methods:

```
scala> def doubleMe(x: Int) = x * 2
doubleMe: (x: Int)Int
```

thanks to the _local type inference_ in this simple example, we don't need to add the return type in the function definition.

If we would like to add a return type the syntax will look like:

```
scala> def doubleMe(x: Int): Int = x * 2
doubleMe: (x: Int)Int
```

Applying an argument to this version of `doubleMe` looks like the previous examples:

```
scala> doubleMe(21)
res3: Int = 42
```

In this case we can't compose this version of `doubleMe` with `plusOne` like we did before. If we try we will get an error:

```
scala> doubleMe andThen plusOne
<console>:13: error: missing arguments for method doubleMe;
follow this method with _ if you want to treat it as a partially applied function
     doubleMe andThen plusOne
     ^
```

this is happening because `doubleMe` is not a `Function1` instance anymore. Composing functions in a different way is working, what we get in this case is an example of _lifting_:

```
scala> plusOne andThen doubleMe
res4: Int => Int = <function1>
```

We can also try to convert _methods_ to _functions_:

```
scala> val h: Int => Int = doubleMe
h: Int => Int = <function1>
```

Sometimes we could reduce the code we have to write using the `_` placeholder:

```
scala> val plusOne: Int => Int = _ + 1
plusOne: Int => Int = <function1>
```

In this case, the function `plusOne` is receiving one argument: `_` will be replaced with the argument we have applied to the function.

Scala is not allowing functions with only one argument, defining `sum` is quite simple:

```
scala> val sum: (Int, Int) => Int = (a, b) => a + b
sum: (Int, Int) => Int = <function2>
```

we could simply the code even further using two `_` placeholders:

```
scala> val sum: (Int, Int) => Int = _ + _
sum: (Int, Int) => Int = <function2>
```

the values application to the `sum` function looks like the `Function1` examples:

```
scala> sum(40, 2)
res5: Int = 42
```

It's quite natural expecting an error in the case we provide less arguments that expected:

```
scala> sum(42)
<console>:12: error: not enough arguments for method apply: (v1: Int, v2: Int)Int in trait Function2.
Unspecified value parameter v2.
       sum(42)
          ^
```

Why should we need to call a function with less arguments that the expected? One of the nicest aspects of Functional Programming is to produce complex computations composing basic functions. In this case we have a trivial use case, but we have two functions `plusOne` and `sum` that they are basically doing something related. At this point we could reuse the `sum` implementation to define `plusOne`. In order to do this we will _partially apply_ the `sum` function:

```
scala> val plusOne = sum(1, _: Int)
plusOne: Int => Int = <function1>

scala> plusOne(41)
res6: Int = 42
```

The syntax looks strange at first, and we already found another use for the `_`. Type inference in Scala is not able to infer the type for function parameters, so in this case we have to help the compiler to figure out the type for the `sum` parameter we are leaving out.

`Function2` is another _trait_ that is getting special treatment from Scala. One of the common use case for functions `(A, B) => C` is the so called _"Schönfinkelisation"_ (just joking, it goes under the *currying* name these days).

Currying translates a function with more parameters to a sequence of applications of functions with a single parameter. For `Functon2`, once the function is curried, we will have that `(A, B) => C` become `A => (B => C)`. In Scala we have a special method for currying:

```
scala> val c = sum.curried
c: Int => (Int => Int) = <function1>
```

applying a value to `c` yields another function:

```
scala> c(10)
res7: Int => Int = <function1>
```

Another functionality we are getting out of `Function2` is `tupled`, to translate the list of function parameters to a `Tuple`.

```
scala> val t = sum.tupled
t: ((Int, Int)) => Int = <function1>
```

The function `t` has only one parameter, in this case a `Tuple2[Int, Int]`.

So far so good, we have seen a list of basic syntax to define *total functions* in Scala, real life is not so nice. In case we have functions that are not defined for particular argument values. The most famous example is a function for division:

```
scala> val div: (Int, Int) => Int = _ / _
div: (Int, Int) => Int = <function2>
```

what happens once we will try to divide a number by 0?

```
scala> div(42, 0)
java.lang.ArithmeticException: / by zero
  at $anonfun$1.apply$mcIII$sp(<console>:10)
  ... 33 elided
```

Exceptions are so little functional, we could probably do better. Scala provides us a _trait_ `PartialFunction[A, B]` to define partial functions:

```
scala> val div: PartialFunction[(Int, Int), Int] = {
	 | case (n, d) if d != 0 => n / d
	 | }
div: PartialFunction[(Int, Int),Int] = <function1>
```

`PartialFunction` trait is available only for functions with one parameter, so in this case we are using the _trick_ to pass a `Tuple2[Int, Int]` as argument.

If we call the `div` function with 0 as denominator the only thing we have changed is the exception we are getting at runtime:

```
scala> div((0,0))
scala.MatchError: (0,0) (of class scala.Tuple2$mcII$sp)
  at scala.PartialFunction$$anon$1.apply(PartialFunction.scala:253)
  at scala.PartialFunction$$anon$1.apply(PartialFunction.scala:251)
  at $anonfun$1.applyOrElse(<console>:10)
  at $anonfun$1.applyOrElse(<console>:10)
  at scala.runtime.AbstractPartialFunction.apply(AbstractPartialFunction.scala:36)
  ... 33 elided
```

The nice thing about `PartialFunction` is the chance to check if the function is defined before for a specific value:

```
scala> div.isDefinedAt((10,0))
res8: Boolean = false

scala> div.isDefinedAt((10,3))
res9: Boolean = true
```

The call to `isDefinedAt` is not always cheap, but it allows nice composition of partial functions.

```
scala> val plusOne: PartialFunction[Int,Int] = {
     | case i if i > 0 => i+ 1
     | }
plusOne: PartialFunction[Int,Int] = <function1>

scala> val doubleMe: PartialFunction[Int, Int] = {
     | case i if i%2==0 => i * 2
     | }
doubleMe: PartialFunction[Int,Int] = <function1>
```

We can now compose them:

```
scala> val f = plusOne orElse doubleMe
f: PartialFunction[Int,Int] = <function1>
```

in case we apply a value for which `plusOne` is not defined (ie non positive numbers) we will then try to apply the argument to `doubleMe`.

```
scala> f(-10)
res10: Int = -20

scala> f(12)
res11: Int = 13
```

in the first example `plusOne` is not defined at `-10`, so we will apply `doubleMe`. In the second example, as `plusOne` is defined at `12` we will apply this function directly and conclude the computation.

Another nice functionality is to lift a `PartialFunction` to replace `MatchError` exception with an `Option` value:

```
scala> val g = plusOne.lift
g: Int => Option[Int] = <function1>

scala> g(0)
res0: Option[Int] = None

scala> g(1)
res1: Option[Int] = Some(2)
```
