Scala Integration   ![Build Status](https://travis-ci.org/kamon-io/kamon-scala.svg?branch=master)
==========================

[![Gitter](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/kamon-io/Kamon?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

***kamon-scala*** [![Maven Central](https://maven-badges.herokuapp.com/maven-central/io.kamon/kamon-play-25_2.11/badge.svg)](https://maven-badges.herokuapp.com/maven-central/io.kamon/kamon-scala_2.11)

Automatic TraceContext Propagation with Futures
===============================================

The `kamon-scala` module provides bytecode instrumentation for both Scala, Scalaz and Twitter Futures that automatically
propagates the `TraceContext` across the asynchronous operations that might be schedulede for a given `Future`.

The <b>kamon-scala</b> module require you to start your application using the AspectJ Weaver Agent. Kamon will warn you
at startup if you failed to do so.


### Future's Body and Callbacks ###

In the following piece of code, the body of the future will be executed asynchronously on some other thread provided by
the ExecutionContext available in implicit scope, but Kamon will capture the `TraceContext` available when the future
was created and make it available while executing the future's body.

```scala
Tracer.withNewContext("sample-trace") {
    // The same TraceContext available here,

    Future {
      // is available here as well.
      "Hello Kamon"

    }.map(_.length)
      .flatMap(len => Future(len.toString))
      .map(s => Tracer.currentContext)
      .map(println)
      // And through all async callbacks, even while
      // they are executed at different threads!
  }

```

Also, when you transform a future by using map/flatMap/filter and friends or you directly register a
onComplete/onSuccess/onFailure callback on a future, Kamon will capture the `TraceContext` available when transforming
the future and make it available when executing the given callback. The code snippet above would print the same
`TraceContext` that was available when creating the future, during it's body execution and during the execution of all
the asynchronous operations scheduled on it.
