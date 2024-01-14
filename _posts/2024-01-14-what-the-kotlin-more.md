---
layout: post
title:  "What the Kotlin? Again!"
date:   2024-01-14
categories: learning kotlin
---

# What the Kotlin? Again!

In [my last post about Kotlin](https://brydonleonard.github.io/learning/kotlin/2023/08/16/what-the-kotlin-variance.html), I mentioned that at in some future post I'd dive into this bowl of keyword soup:

```kotlin
inline operator fun <reified T : Component> Entity.get(clazz: KClass<T>): T
```

this is that future post!

## [Generic constraints](https://kotlinlang.org/docs/generics.html)
```kotlin
inline operator fun <reified T : Component> Entity.get(clazz: KClass<T>): T
                               ^
```
I'll start off with one that's straightforward for anyone making the jump from Java. Defining generics with the type parameter `<T : Foo>` constrains the type `T` to be some subtype of Foo; this constraint is called an _upper bound_ on the generic type. The Java equivalent is `<T extends Foo>`.

## [Function inlining](https://kotlinlang.org/docs/inline-functions.html)
```kotlin
inline operator fun <reified T : Component> Entity.get(clazz: KClass<T>): T
   ^
```

In Kotlin, a higher-order function is one that takes other functions as arguments, such as:

```
fun map(numbers: List<Int>, operation: (Int) -> Int): List<Int>
```

Higher-order functions can have runtime overhead, however. They're each implemented as separate objects and capture a closure over the variables that it needs to access. The `inline` keyword helps to reduce that overhead by copy/pasting the bytecode of the higher-order function (and all of the functions passed to it) to its call locations.

Here's a toy example to show inlining in action. Instead of printing "hello world" directly, this program passes a function that prints hello world to a higher-order function, `callFunc`, which then calls it:

```
fun main() {
	callFunc { print("hello world") } 
}

fun callFunc(func: () -> Unit) {
	func()
}
```

here's the bytecode for that program. Don't worry too much about grasping all of it; I'll call out the interesting bits. I've also trimmed out the unnecessary surrounding bytecode.

```
public static final void main();
Code:
    0: getstatic     #12                 // Load the lambda with the print statement
    3: checkcast     #14                 
    6: invokestatic  #18                 // Call callFunc
    9: return

public static final void callFunc(kotlin.jvm.functions.Function0<kotlin.Unit>);
Code:
    0: aload_0
    1: ldc           #22                 
    3: invokestatic  #28                 // Do a null check on the function argument
    6: aload_0
    7: invokeinterface #32,  1           // Call the function passed as an argument
    12: pop
    13: return
...
}
```

Now, if we add `inline` to `callFunc`'s definition, we get:


```
public static final void main();
Code:
    0: iconst_0
    1: istore_0
    2: iconst_0
    3: istore_1
    4: ldc           #8                  // Load the string "hello world"
    6: getstatic     #14                 // Get a reference to stdout
    9: swap
    10: invokevirtual #20                 // Call "print"
    13: nop
    14: nop
    15: nop
    16: return
...
}
```

In the first example, there's a bunch of overhead when we load the lambda function, call `callFunc`, and do null checks before even executing the lambda. All that overhead is gone in the second example, where the bytecode loads in the string and immediately prints it out. 

You do need to exercise some caution when inlining functions. The size of your program's bytecode can explode if the function you're inlining is used a lot or is really long, since its bytecode is copied to _every_ call location.

### Reified types
```kotlin
inline operator fun <reified T : Component> Entity.get(clazz: KClass<T>): T
                        ^
```

As with Java, the types of generic functions are usually unavailable at runtime (that's called _type erasure_). Unlike in Java, however, we have a way to stop type erasure in Kotlin; the `reified` keyword. You could, for example, write a method to print class names. Without reification, it would look something like:

```
fun <T> prettyPrintClass(clazz: KClass<T>) {
    // T isn't available here, so pull the information from the clazz object.
    print(clazz.simpleName)
}
```

but with reification, could be:

```
inline fun <reified T> prettyPrintClassReified() {
    // We now have access to T as if it's a pain old class.
    print(T::class.simpleName)
}
```

## [Extension functions](https://kotlinlang.org/docs/extensions.html)
```kotlin
inline operator fun <reified T : Component> Entity.get(clazz: KClass<T>): T
                                                  ^
```

A really fun bit of syntactic sugar that Kotlin offers is the _extension function_. Extension functions let you bolt on new functions to existing classes, which can often make your code read a little more naturally. 

We could either define an `isEven` function like this:

```
fun isEven(number: Int) = number % 2 == 0

// Which is called with
isEven(4)
```

or, we could use an extension function and write it like this:

```
fun Int.isEven(number: Int) = number % 2 == 0

// Which is called with
4.isEven()
```

which is a little prettier.

### Gotchas

Extension functions tidy code up, but they can also make it much more confusing; I would warn against ever using extension functions outside of the file in which they're defined. When seeing `x.y()`, most people are going to look for `y` in the definition of `x`'s type. If it's not there, and not in the current file, they're going to have to go searching. That's not a pleasant experience for readers of your code (or for future you).

An additional note on extension functions is that, unlike regular functions, they're defined on a specific type and ignore class hierarchies. You may think of tidying up 

```
fun typeOf(obj: Any) = obj::class.simpleName
```

and instead writing it as 

```
fun Any.myType() = this::class.simpleName
```

The issue is that the second function will _always_ return `Any`, regardless of the type of its receiver.

### Operator extension functions
```kotlin
inline operator fun <reified T : Component> Entity.get(clazz: KClass<T>): T
          ^
```

The last fun type of extension is operator overloads. They let you implement custom behaviour for operators like `+` or `-`. In the example above, overloading the `get` extension implements behaviour for `[]`. Again, I'd recommend being sparing in your use of operator overloads; if you throw too many in, it may just make your code more confusing to read.
