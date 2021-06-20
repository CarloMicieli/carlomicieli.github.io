---
layout: post
title: Highlights From Curry-on Prague
date: 2015-07-19
#[taxonomies]
tags: ["conferences", "functional programming", "programming languages design"]
---

Here it comes a list of pseudo-random thoughts on **Curry-on**, a programming languages designers/nerds conference held in Prague.

![Curry-On badge](/assets/img/2015/07/curryon_badge.png)

**Programs Wanted: Dead or Alive** The initial keynote speech was given by Gilad Bracha who set the tone for the following two days, discussing about the tension between live and dead. He applied this to programming, making a comparison between live programming and dead programming. He gave his view on how we are basically favoring dead software because it is much easier to reason about things that are not changing. He also discussed the languages and the tools that support our daily programming. Using smalltalk as example he showed a view on how live programming should look like; taking inspiration from literate programming we should be able to make changes on the fly to our programs, refresh and see the changes. He also went forward discussing the possibilities with time traveling debugging (and this point showed up in the closing keynote too with the Elm debugger).

**Kotlin: Challenges in language design** The second talk I attended to was about Kotlin developed by JetBrains, a general purpose programming language on the JVM. The presentation was not a full tour of the language, but presented the challenges in developing a new programming language. The first challenge was mostly specific to tools maker like JetBrains: the integration between compilers and IDEs. They have different duties, one example is code navigation (find declaration, find usage, and so on) which is something the compilers don’t provide out of the box. The second (controversial) challenge is the integration between Kotlin and Java, with the 1-billion dollar mistake: null. Kotlin implements an interesting infrastructure of nullable types to try to mitigate the chance to catch the infamous NullPointException at compile time, rather than at runtime. This posed a lot of issues interacting with Java code, they tried to address this issue with Platform types, allowing a safer way to develop.

**Akka Typed** Between Session Types and the Actor Model Then it was the time for Akka and Roland Kuhn from Typesafe. This was the first “chess timed” talk ever. In the first few minutes he gave a quick overview of the actor system and the Akka implementation. Then he started to show a very early proof of concept on how typed actors will look like in Akka. This will allow the definition of protocols for the actors communication that are checked at compile time. Typed actors will also make refactoring easy and reduce the number of mistakes. The presented implementation was based on a total function that is mapping the message to a particular behavior. He also pointed out that something will remain dynamic: who’s doing what.

**Pull > Push: Please stop polluting our imperative languages with pure concepts** Ron Pressler talked about the functional programming influence on imperative languages. He used Java 8 as working example, discussing about the monads that are now implemented in standard library: Optional, Stream and CompletableFuture. He described monads like design patterns in the imperative world, or like functions with “super powers”. During the presentation he also showed examples of computations chaining, comparing the monadic way with a more imperative approach. He also discussed the basic threading model for imperative programs. During the talk he showed situation in which the imperative model looks much easier to reason about because they are not natural for imperative programmers.

**Speed at a Price** The Evolution of V8 and the Challenges of Research in a Billion User VM discussed the challenges Google is facing to implement V8: a VM for Javascript.

**It Probably Works** was nice talk about probabilistic algorithms, where exists some level of randomness. They solve problems unrealistic to solve exactley. Tyler McMullen then showed few examples like: “Count distint problem”, “Reliable broadcast” and “Bimodal Multicast”. They have in common the fact that if we are able to bound the error rate we could get fine solutions to otherwise hard to solve problems.

**Servant: a type-level DSL for web APIs** Julian Arni showed an Haskell library to write Web-api in a typesafe manner. Using a formal language the library is able to produce the documentation automatically.

**Making Embedded Domain Specific Languages a Practical Reality** Jurriaan Hage discussed a DSL to produce better compiler error for Haskell. With this implementation we could getter improved type error reporting, basically if the compiler rejected my code it should tell me the reason.

**Julia: Numerical Applications Pushing the Limits of Language Design** during the closing keynote for the first day Jeff Bezanson and Stefan Karpinski showed Julia, a programming language for numerical application with some examples.

**JS @ 20** during the initial keynote for the second day, Brendan Eich, the creator of Javascript made an amusing presentation for the past and future of the language.

**Bits of Advice for VM Writers** Cliff Click made a quick overview of the challenges he faced implementing VM. During the presentation he shared what it worked and what it wasn’t.

**QuickCheck: from invention to product** in this nice talk Thomas Arts made a quick overview of property based testing with QuickCheck. Then he discussed the challenges to push the QuickCheck adoption in the industry.

**How to be a good host: miniKanren as a case study** as I’m completely unaware of what miniKaren actually is I had hard time to follow. The Lisp syntax didn’t help… :-)

**Coding for Types: The Universe Pattern in Idris** in his talk David Christiansen made a quick introduction of dependent types with Idris, and then showed examples of its use.

![Curry-On badge](/assets/img/2015/07/curryon_phil.jpeg)

**Everything old is new again**: Quoted domain specific languages Philip Wadler in his talk showed an example of an embedded DSL: an evolution of Linq implemented in F#.

**GS Collections: Echoes of Smalltalk’s Past** Alex Iliev made a showcase of the collections implemented by Golden Sachs, comparing them to other collection libraries available for the JVM.

**Rust: A Type System You Didn’t Know You Wanted** in a talk with huge participation from the audience, Felix Klock made a quick overview of the programming language Rust, implemented by Mozilla. Rust is a system programming language with safety as first class citizen: it basically implements an infrastructure to provide both memory safety and guarantee of data race freedom.

**Pony: making it easy to write efficient, concurrent, data-race free programs** in his talk Sylvan Clebsch introduced Pony a new programming language to provide safety at all levels: memory, types, exceptions and concurrency. Pony is a programming language that is merging seamlessly different paradigms like functional programming, object oriented programming and the actor model.

**Let’s be mainstream!** User-focused design in Elm the closing keynote by Evan Czaplicki (the creator of Elm, a language that compiles to Javascript) was completely dedicated on his view on how to make a language mainstream. It took a lot of courage to replace the word “monad” with “callbacks” in a room full of academics.
