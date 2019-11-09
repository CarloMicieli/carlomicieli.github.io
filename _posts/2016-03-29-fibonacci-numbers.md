---
layout: post
title: Fibonacci Numbers
date: 2016-03-29
#[taxonomies]
#tags = ["scala", "functional programming"]
---

Lately I’m getting overly obsessed over the legendary Fibonacci numbers.

They have a pretty trivial mathematical definition:

  	fib(1) = 1
  	fib(2) = 1
      fib(n) = fib(n - 1) + fib(n - 2)

Everything looks so easy, what could possibly go wrong? It takes only few lines of plain old Scala to calculate the n-th Fibonacci number:

```scala
def fib(n: Int): Int = n match {
  case 1          => 1
  case 2          => 1
  case n if n > 2 => fib(n - 1) + fib(n - 2)
  case _          => throw new IllegalArgumentException(s"undefined for $n")
}
```

This function will not make you rich, but it does its dirty work. Time to fire up a REPL and have fun:

```scala
scala> fib(1)
res1: Int = 1
scala> fib(2)
res2: Int = 1
scala> fib(10)
res3: Int = 55
scala> fib(20)
res4: Int = 6765
scala> fib(30)
res5: Int = 832040
```

Oddly the function looks __"slower-ish"__ as we are increasing its argument, anyhow it's look like we have just accomplished our mission. Last thing before a well deserved cold drink, let's try with `50`:

  	scala> fib(50)
      ...five minutes later...
      res7: Int = -298632863

Wow, we probably bent the space and time continuum (__great Scott!!__). We caused an overflow for the result because the 50th Fibonacci number doesn't fit a JVM `Int`. No problem here, simply change the return type to `Long`:

```scala
def fib(n: Int): Long = n match {
  case 1          => 1L
  case 2          => 1L
  case n if n > 2 => fib(n - 1) + fib(n - 2)
  case _          => throw new IllegalArgumentException(s"undefined for $n")
}
```

Let's try again:

    scala> fib(50)
    ...five minutes later...
    res8: Long = 12586269025L

Nothing we can stop us at this point, time to reach for the stars:

  	scala> fib(87)
  	...

What we have just built is a room heater (and we didn't even need a rasberry py in order to accomplish that). The process in the JVM is happly bringing a CPU core to 100% and it doesn't seem interested in yielding any kind of result.

Long story short, the trivial implementation for Fibonacci is really really __"trivial"__ as it's doing the same computations over and over again. According to the "bible" (Harold Abelson and Gerald Jay Sussman, **Structure and Interpretation of Computer Programs**; [exactly here](https://mitpress.mit.edu/sicp/full-text/book/book-Z-H-11.html#%_sec_1.2.2)) the number of steps performed by this particular implementation is growing exponentially with `n`.

And we teached us that **exponentially is bad**.

Not all hopes are lost. In Scala we love __"one liner"__, and we can solve the Fibonacci numbers problem with style (from the scaladoc for the `Stream` class):


    scala> val stream: Stream[Long] = 1L #:: 1L #::
     	stream.zip(stream.tail).map { n => n._1 + n._2}

    scala> stream.drop(86).take(1)
    res15: Stream[Long] = Stream(
      679891637638612258L
    )

With the chance to nest functions, we can implement the Fibonacci numbers in an iterative way as suggested in SICP:

```scala
def fibIter(n: Int): Long = {
  require(n >= 1)

  @annotation.tailrec
  def loop(i: Int, fib: (Long, Long)): Long = {
    if (i == 1) {
      fib._1
    } else {
      val (f2, f1) = fib
      loop(i - 1, (f2 + f1, f2))
    }
  }
  loop(n, (1L, 0L))
}
```

The Scala compile will happly replace our recursive function `loop` with an actual loop in the bytecode. In any case, every recursive call has already the previous 2 values and we have a hugely smaller number of recursive calls. With this implementation we can ask for the 87th Fibonacci number and we are getting an answer back pretty quickly:

  	scala> fibIter(87)
  	res9: Long = 679891637638612258L

In this particular example we are actually lucky: the Fibonacci number function is __referentially transparent__ (it means we can safely substitute its application with the result it yields). For these kind of functions we can trade time complexity with memory usage and apply an optimization called **memoization**. We are storing the function results, in order to compute them only once.

```scala
object memFib {
  private var cache: Map[Int, Long] = Map.empty[Int, Long]

  def apply(n: Int): Long = n match {
    case 1          => 1L
    case 2          => 1L
    case n if n > 2 =>
      if (cache.contains(n))
        cache(n)
      else {
        val res = apply(n - 1) + apply(n - 2)
        cache = cache + (n -> res)
        res
      }
    case _ => throw new IllegalArgumentException(s"undefined for $n")
  }
}
```

With this implementation the function is not completely pure (it produces side effects to update the cache), but this is just an optimization: the referential transparency is preserved.

> Over-engineering, son. Nothing else in the world smells like that.

At this point we can go completely rogue, and reach for Akka as I did [here](https://gist.github.com/CarloMicieli/31be8627b29563e34be031835860df55), where lies the implementation of the Fibonacci numbers using over 300 lines of crazy-actor-based code.
