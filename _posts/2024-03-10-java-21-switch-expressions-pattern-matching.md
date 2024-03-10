---
layout: post
title: "Java 21 - Switch expressions and pattern matching"
date: 2024-03-10
categories: learning java
tags: java21
---

In this last post in the Java 21 language feature series, I'll take a look at how the pattern matching and switch expressions integrate with the new class types I covered in [Java 21 - New class types](https://brydonleonard.github.io/learning/java/2024/03/04/java-21-new-class-types.html) and take a quick look at text blocks and sequenced collections.
## Pattern matching

Prior to Java 21, checking the type of an object was a little frustrating, since you had to both perform the check _and_ manually cast the object to the type that you just checked:

```java
if (obj instanceof Shape) { // First, check the type
    ((Shape)obj).calculateArea(); // then manually cast it to the type
}
```

Java 21's pattern matching removes that requirement. You can now pass a type _pattern_ as the second operand of `instanceof` (the right-hand one) and write the conditional as:

```java
//                 ┌─────┬─── this is the type pattern           
if (obj instanceof Shape s) {
	// s (of type Shape) is now in scope
    s.calculateArea();
}
```

### Destructuring patterns

When you use pattern matching with record types, you can destructure record types into their components:

```java
record Point(int x, int y);

if (obj instanceof Point(x, y)) {
    // x and y are now in scope
}
```

You can even do it recursively with nested record types!

```java
record Point(int x, int y);
record Line(Point p1, Point p2);

if (obj instance Line(Point(x1, y1), Point(x2, y2))) {
    // x1, y1, x2, and y2 are all in scope here
}
```

## Switch expressions

All of the pattern matching features are also available in switch statements' case labels. You can even add new _guard_ statements to your cases (which use the `when` keyword), which act as extra restrictions on the cases:

```java
switch (obj) {
    case String s when s == "foo": System.out.println("Got foo!");
	    break;
	case String s: System.out.println("Got a string");
		break;
	default:
		 throw IllegalArgumentException("Got an unexpected type");
}
```

Even with guard statements, switch statements are pretty verbose, though. Java 21 now lets you avoid all of those `break`s with the new switch expressions, which don't fall through between cases. All you need to do is switch from `:` to `->`:

```java
switch (obj) {
	case String s when s == "foo" -> System.out.println("Got foo!");
	case String s -> System.out.println("Got a string");
	default -> throw IllegalArgumentException("Got an unexpected type");
}
```

As their name suggests, switch expressions are *expressions* and evaluate to a value. You could, for example, assign them to variables:

```java
String stringToPrint = switch (obj) {
	case String s when s == "foo" -> "Got foo!";
	case String s -> "Got a string";
	default -> throw IllegalArgumentException("Got an unexpected type");
}
```

### Case label evaluation

Switch expressions (and switch statements that use pattern matching) must always be exhaustive; every possible value of the object passed to the switch _must_ have behaviour defined for it. In most cases, you can cover those additional behaviours with a `default` case (as in the examples above). When you're using a sealed type or an enum, however, the compiler *knows* about every possible type that the object could have, so if you cover all those cases, you don't need the default case any more:

```java
sealed interface Shape {
	int getArea();
}
record Square(int width) implements Shape {
	public int getArea() { return width * width; }
}
record Rectangle(int width, int height) implements Shape {
	public int getArea() { return width * height; }
}

Shape obj = getAShape(); // Get some shape
// This switch covers all possible values, so no default case
switch (obj) { 
	case Square square -> System.out.println("Square with width " + square.width);
	case Rectangle rect -> System.out.("Rect with size " + rect.width + " by " rect.height);
}
```

There are a few additional new behaviours that come with expression-style (`->`) case labels:
- If there's a `null` case, the switch expression/statement won't throw a null pointer exception if the selector expression's value is null
- Only one case is executed, so if an earlier case is true for *every* value that would match some later case, it is said to *dominate* that later case since the later one will never run. You'll get a compile-time error if you've got dominating cases.
### Yield (if you want to, I guess)

If you want to use the old statement-style (`:`) case labels but also want to return a value from the switch (so you want it to be an expression), you can do that with the `yield` keyword:

```java
String stringToPrint = switch (obj) {
	case String s when s == "foo": yield "Got foo!";
	case String s: yield "Got a string";
	default: throw IllegalArgumentException("Got an unexpected type");
}
```

## The others!

The last two new non-preview features in Java 21 are text blocks and sequenced collections.
### Text blocks

If you've used text blocks or multi-line strings in other languages, Java's will behave the way you expect. You can define strings that wrap over multiple lines without manually concatenating them:

```java
String html = """
	<html>
		<body>
			<p>Hello World.</p>
		</body>
	</html>
	""";
```

The strings that it creates are regular Java strings and you can use them as you would any other. The interesting details of text blocks' behaviour really lie in how they treat whitespace. You'll have to look at [the documentation](https://docs.oracle.com/en/java/javase/21/text-blocks/index.html) for the full list, but here are some of the highlights:
- Leading whitespace is trimmed until the line with the _least_ whitespace has none at all. That means you can indent your text block for readability and it'll be trimmed.
- The line terminators are always `\n`, regardless of what they are in the rest of the source file
- `String::formatted` is a new method that you can chain on strings to perform substitution:

```java
String output = """
    Name: %s
    Phone: %s
    Address: %s
    Salary: $%.2f
    """.formatted(name, phone, address, salary);
```
### Sequenced collections

Sequenced collections is the name for the new interfaces in the Java Collections Framework:
- `SequencedList`
- `SequencedSet`
- `SequencedMap`

They don't really add any new functionality, but give you *easier* access to:
- the first/last element of a collection (through `getLast()`, `getFirst()` , `addFirst()`, and `addLast()`)
- a reversed view of the collection (through `reversed()`)

It's worth keeping in mind that the addition of `addFirst()` doesn't let the collections do magic. The time complexity is still the same as if you'd called `list.add(0, obj)`.

## Closing out

That's it for Java 21's (non-previews) language features. I think they've done a great job making Java *feel* more like a modern language. At some point, I'll also write up a post on the changes to ZGC included in the release, but that's a job for later!

## References

- [The Java 21 language updates docs](https://docs.oracle.com/en/java/javase/21/language/java-language-changes.html#GUID-C9246BC1-4785-4F3B-997D-58C4319E21C2)

[![Hits](https://hits.seeyoufarm.com/api/count/incr/badge.svg?url=https%3A%2F%2Fbrydonleonard.github.io%2Flearning%2Fjava%2F2024%2F03%2F10%2Fjava-21-switch-expressions-pattern-matching.html&count_bg=%2379C83D&title_bg=%23555555&icon=&icon_color=%23E7E7E7&title=hits&edge_flat=false)](https://hits.seeyoufarm.com)