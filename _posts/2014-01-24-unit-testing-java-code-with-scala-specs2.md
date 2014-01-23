---
layout: post
title: "Unit testing Java code with Scala Specs2"
category: Scala
tags: [scala, specs2, TDD]
author: "Mehdi Mollaverdi"
---
{% include JB/setup %}

# Unit testing Java code with Scala Specs2

Here at Bombora Technologies, we do a lot of Java programming and we are big fans of TDD. Traditionally we've been using JUnit to write unit tests, recently though we started using the Scala [specs2](http://etorreborre.github.io/specs2/) library to test our Java code.

The team has had a mixture of opinions about Scala and Specs2.  Here is an aggregation of different pros of cons that different developers in the team have come up with:

## Pros

### More Readable Tests

* __Nicer assertions__: TBC
* __More readable mocking__: TBC
* __Mock definitions, stubbing and verification__: TBC
* __Test names as proper sentences rather than method names__: TBC
* __Nicer use of argument matchers__: TBC
* __Infix operators and parenless method calls__: TBC
* __TBC__

### Scala Collection API

TBC

### Better Test Organization with Scope Traits

TBC

### More Reusability with Scope Traits

TBC

### Running Tests in Parallel

TBC

### Function Composition

TBC

### Interop with Java

TBC

## Cons

### IDE Support

We use Intellij IDEA for almost all of our day-to-day programming work. The IntelliJ support for Scala has improved over the last year, but it just still isn't there. Some of the difficulties/deficiencies that we've faced are:

* TBC <<Inability to create Java classes or add methods to an existing class when TDDing>>
* TBC

### Slow to Compile

Scala is slow to compile. Specially when doing TDD, for each test case, you would write a failing test first, run the test, provide the implementation and finally re-run the test to see if it passes or not. You would probably then also refactor your code and re-run the test to make sure it still passes. And then you would repeat the same process for more tests.

It can be frustrating if each time you run a single test inside the IDE, it takes a few seconds to compile.

TBC <<The fact that intellij and the mandatory external build made it worth>>

### Interop with Java

TBC

## Conclusion

TBC
