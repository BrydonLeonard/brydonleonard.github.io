---
layout: post
title: "Java 21 - New class types"
date: 2024-03-04
categories: learning java
tags: java21
---

Java 21 introduces `sealed` and `record` classes. Let's take a look at what those can do for you.

## Sealed classes

Prior to Java 21, `final` was the only modifier you could apply to classes to restrict their inheritance (by preventing anything from extending them at all). Now we also have `sealed` which can be applied to both classes and interfaces and only allows explicitly permitted classes to extend/implement them. Having a list of all of the types that extend a sealed class is useful because you can *know* at compile-time that your code handles all of them.

To specify the types that can extend or implement your `sealed` type, either define all of the subtypes in the same file as the parent or specify them in the new `permits` clause of the class definition:

```java
// Sealed class
public sealed class Shape
	permits Circle, Square {
}

// Sealed interface
public sealed interface BinaryOperation
	permits Add, Subtract {
	int eval(int a, int b);
}
```

Given the `Shape` class above, we could write a method like the one below and be confident that our code can find the area of any shape passed to it (because we know that `Square` and `Circle` are the only children of `Shape`)[^1] :

```java
public double area(Shape shape) {
	switch (shape) {
		case Square: 
			return Math.pow(((Square) shape).length, 2.0);
		case Circle:
			return Math.PI * Math.pow(((Circle) shape).length, 2.0);
	}
}
```

### Requirements of subtypes

All of the subtypes of a sealed type *must* be one of:
- `final` - works just like before. Nothing can further extend the subtype.
- `sealed` - these subtypes can have their own sealed hierarchy of subtypes
- `non-sealed` - This opens up the sealed hierarchy and allows the child to be extended without restriction.


![A sealed class hierarchy](/assets/images/2024-03-04-java-21-new-class-types/sealed-class-hierarchy.JPG)

## Record classes

The other new type of class is the `record`. Java's verbosity is a pretty common complaint and `record`s attempt to address some of it; you can now define simple classes to hold immutable data in a single line[^2]!

```
record Point(int x, int y) {}
```

The compiler takes care of all the boilerplate that usually comes along with creating a class:
- `hashCode`, `equals`, `toString` are all implemented for you
- there's an implicit *canonical* constructor, which takes all components from the header
- each component in the header gets its own `private final` field (so they're all immutable)
- those components all get accessors. For a field `fieldName`, the accessor is `fieldName()`

### Overriding record class defaults

Any of the implicit members in record classes can be overridden by re-defining them explicitly. If you'd like to add validation to your record class' canonical constructor, for example, you can do so:

```java
record Point(int x, int y) {
	public Point(int x, int y) {
		if (x < 0 || y < 0) {
			throw new IllegalArgumentException("Can't have a point < 0");
		}
		this.x = x;
		this.y = y;
	}
}
```

If re-writing the whole canonical constructor's signature isn't to your tastes (or you don't want to have to worry about a typo in your custom constructor), you can use a *compact* constructor. The assignment to the *implicit formal parameters* (`this.x` and `this.y`) happens *after* the invocation of the compact constructor.

```java
record Point(int x, int y) {
	public Point {
		// x and y are in scope here
		// this.x and this.y haven't been set yet
	}
}
```

Component accessors, `hashCode`, `equals`, and `toString` can also all be customized by re-implementing them explicitly:

```java
recort Point(int x, int y) {
	public int x() { // Override the default component accessor for x
		System.out.println("X is " + x);
		return x;
	}
}
```

### Restrictions

Record classes behave almost exactly like regular classes under the hood. That means you can use them like any other class. They can:
- have generic types
- be part of inheritance hierarchies (including sealed ones)
- be annotated
- be declared locally within a method
- have static members and initializers
- have nested types
- have instance methods

Unlike regular classes, however, they can't:
- have any non-static fields (besides those derived from the header)
- have custom serialization/deserialization

## Next time

Sealed and record classes are interesting on their own, but they really become useful when used in conjunction with Java 21's new enhanced pattern matching and switch expressions. I'll take a look at those in the next post.

## References

- [The Java 21 language updates docs](https://docs.oracle.com/en/java/javase/21/language/java-language-changes.html#GUID-C9246BC1-4785-4F3B-997D-58C4319E21C2)

## Footnotes

[^1]: This could be tidier if we used pattern matching, but I'll avoid doing so until the next post, which will discuss that in detail.
[^2]: Record classes are very similar to Kotlin's data classes. The biggest differences are that the components of a record are _always_ immutable (where data types can have mutable components) and that records can't have any non-component instance fields (where data classes can).

[![Hits](https://hits.seeyoufarm.com/api/count/incr/badge.svg?url=https%3A%2F%2Fbrydonleonard.github.io%2Flearning%2Fjava%2F2024%2F03%2F04%2Fjava-21-new-class-types.html&count_bg=%2379C83D&title_bg=%23555555&icon=&icon_color=%23E7E7E7&title=hits&edge_flat=false)](https://hits.seeyoufarm.com)