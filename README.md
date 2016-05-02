# completable-futures
[![Build Status](https://img.shields.io/travis/spotify/completable-futures/master.svg)](https://travis-ci.org/spotify/completable-futures)
[![Test Coverage](https://img.shields.io/codecov/c/github/spotify/completable-futures/master.svg)](https://codecov.io/github/spotify/completable-futures?branch=master)
[![Maven Central](https://img.shields.io/maven-central/v/com.spotify/completable-futures.svg)](https://maven-badges.herokuapp.com/maven-central/com.spotify/completable-futures)
[![License](https://img.shields.io/github/license/spotify/completable-futures.svg)](LICENSE)

completable-futures is a set of utility functions to simplify working with asynchronous code in
Java8.

## Usage

Using `completable-futures` requires Java 8 but has no additional dependencies. It is meant to be
included as a library in other software. To import it with maven, add this to your pom:

```xml
<dependency>
    <groupId>com.spotify</groupId>
    <artifactId>completable-futures</artifactId>
    <version>0.1.0</version>
</dependency>
```

## Features

### Combining more than two things

The builtin CompletableFuture API includes `future.thenCombine(otherFuture, function)` but if you
want to combine more than two things it gets trickier. The `CompletableFutures` class contains the
following APIs to simplify this use-case:

#### allAsList

If you want to join a list of futures of uniform type, use `allAsList`. This returns a future which
completes to a list of all values of its inputs:

```java
List<CompletableFuture<String>> futures = asList(completedFuture("a"), completedFuture("b"));
CompletableFuture<List<String>> joined = CompletableFutures.allAsList(futures);
```

#### joinList

`joinList` is a stream collector that combines multiple futures into a list. This is handy if you
apply an asynchronous operation to a collection of entities:

```java
collection.stream()
    .map(this::someAsyncFunction)
    .collect(CompletableFutures.joinList())
    .thenApply(this::consumeList)
```

#### combine

If you want to combine more than two futures of different types, use the `combine` method:

```java
CompletableFutures.combine(f1, f2, (a, b) -> a + b);
CompletableFutures.combine(f1, f2, f3, (a, b, c) -> a + b + c);
CompletableFutures.combine(f1, f2, f3, f4, (a, b, c, d) -> a + b + c + d);
CompletableFutures.combine(f1, f2, f3, f4, f5, (a, b, c, d, e) -> a + b + c + d + e);
```

### Missing parts of the CompletableFuture API

The `CompletableFutures` class includes utility functions for operating on futures that is missing
from the builtin API.

#### handleCompose

Like `CompletableFuture.handle` but lets you return a new `CompletionStage` instead of a
direct value.

```java
CompletionStage<String> composed = handleCompose(future, (value, throwable) -> completedFuture("hello"));
```

#### exceptionallyCompose

Like `CompletableFuture.exceptionally` but lets you return a new `CompletionStage` instead of a
direct value.

```java
CompletionStage<String> composed = CompletableFutures.exceptionallyCompose(future, throwable -> completedFuture("fallback"));
```

#### dereference

Unwrap a `CompletionStage<CompletionStage<T>>` to a plain `CompletionStage<T>`.

```java
CompletionStage<CompletionStage<String>> wrapped = completedFuture(completedFuture("hello"));
CompletionStage<String> unwrapped = CompletableFutures.dereference(wrapped);
```

#### exceptionallyCompletedFuture

Creates a new future that is already exceptionally completed with the given exception.

```java
return CompletableFutures.exceptionallyCompletedFuture(new RuntimeException("boom"));
```

## License

Copyright 2016 Spotify AB.
Licensed under the [Apache License, Version 2.0](LICENSE).

## Code of Conduct

This project adheres to the [Open Code of Conduct][code-of-conduct]. By participating, you are
expected to honor this code.

[code-of-conduct]: https://github.com/spotify/code-of-conduct/blob/master/code-of-conduct.md
