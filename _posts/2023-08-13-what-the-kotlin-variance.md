---
layout: post
title:  "What the Kotlin? Variance!"
date:   2023-08-16
categories: learning kotlin
---

# What the Kotlin? Variance!

I like Kotlin, so I thought it'd be fun to write a series of posts on some of its more interesting aspects (or at least, some of the things I find fun about it). My goal isn't to re-write the Kotlin reference docs (which are excellent), but will just try to bridge the gaps that a dev may have when coming from another language. I'll include links to the Kotlin docs wherever appropriate.

For today, I'll be looking at [variance](https://en.wikipedia.org/wiki/Covariance_and_contravariance_(computer_science)). It's not a Kotlin-specific topic, but I'll spend some time looking at how you can managed the variance of your generic types in the language.

Through this post, there will be occasions where code snippets are _intentionally_ broken to show you why certain behaviours cannot exist in Kotlin. Those code snippets will have an `// Intentionally broken` comment at the top to warn you.

## The two types of variance

### Covariance

Kotlin, like many other object-oriented programming languages, supports subtyping. One of the core properties of subtyping is that a subtype is able to be used anywhere that its supertype is. For example, `printWeight` below could take either a `Vehicle` or `Car` as an argument.

```kotlin
open class Vehicle
class Car : Vehicle()

fun main() {
    val vehicle: Vehicle = Car()
}
```

That's fairly straightforward, but what if we're working with _lists_ of vehicles?

```kotlin
val listOfVehicles: List<Vehicle> = listOf<Car>(Car())
```

That works, but there's more going on than you might think. When we started this section, we defined `Car : Vehicle` (I'll be using this notation to indicate "Car is a subtype of vehicle"), but the fact that we could assign a `List<Car>` to a variable of type `List<Vehicle>` means that we've somehow ended up with `List<Car> : List<Vehicle>` too! This property, where complex types have the same subtype relationship as their components, is called _covariance_. Because `List<T>` follows the same subtype relationship as `T` in Kotlin, we say that Kotlin's lists are _covariant in `T`_.

> If I'm honest, I find the preposition for variance being "in" (so, covariant _in_ `T`) a little weird, but that seems to be the standard, so it's what I'll use.

#### Why lists can be covariant in Kotlin

Lists aren't covariant in all languages; notably, Java's lists are not covariant:

```bash
jshell> List<Object> list = new ArrayList<String>();
|  Error:
|  incompatible types: java.util.ArrayList<java.lang.String> cannot be converted to java.util.List<java.lang.Object>
|  List<Object> list = new ArrayList<String>();
|                      ^---------------------^
``` 
Kotlin's able to support list covariance because its lists are _immutable_. Kotlin does have a mutable list type (`MutableList`), which is _not_ covariant. To see why, let's pretend that mutable lists are covariant and consider the example below. 

```kotlin
// Intentionally broken
open class Vehicle
class Car : Vehicle() {
    fun openDoor(): { ... }
}
class Motorbike : Vehicle()

fun main() {
    val mutableList: MutableList<Car> = mutableListOf<Car>()
    mutableList.add(Car())
    mutableList.add(Motorbike())

    mutableList.forEach { car ->
        car.openDoor
    }
}
```

If mutable lists were covariant, we could treat a `MutableList<Car>` as a `MutableList<Vehicle>`. Since `Motorbike` and `Car` are both subtypes of `Vehicle`, it would be valid to add either type to a list of `Vehicle`s, but what we've then ended up with is a `List<Car>` that contains a `Motorbike`! If a list is immutable, the type of all of its elements is known and valid when it's created and there's no risk of its covariance causing issues.

> There are still ways that `List`s' covariance can cause problems in Kotlin. See the appendix for more info.

### Contravariance

So, when complex types have the same subtype relation as their components, they're called covariant. When we flip that and the complex types have a subtype relationship that's the _opposite_ of their components, they're _contravariant_. The idea of the subtype relationship being "backwards" is pretty strange and trying to imagine use-cases for that behaviour can be difficult. However, it does get little easier if you know that the only place that contravariance can really exist is in functions that are contravariant in the types of their parameters. In fact, _every_ function is contravariant in its parameters:

```kotlin
open class Vehicle {
    fun checkOil() {}
}
class Car : Vehicle()

fun runChecks(vehicle: Vehicle) {
    vehicle.checkOil()
}

fun main() {
    runChecks(Vehicle())
    runChecks(Car())
}
```
You'd expect that to work perfectly fine and it does! `Car : Vehicle`, so it makes sense that you can call `runChecks` (which takes a `Vehicle`) with either one. But, you can also do:

```
val runVehicleChecks: (Vehicle) -> Unit = ::runChecks
val runCarChecks: (Car) -> Unit = runVehicleChecks
```

because if a method takes in a `Vehicle` as a parameter, it can also take in a `Car` as a parameter. What we've ended up with here is `Car : Vehicle` and `(Vehicle) -> Unit : (Car) -> Unit`. The subtype relationship of these functions is the reverse of the parameters types in which they're covariant!

#### But why can contravariance only apply to input types?

The reason that the types of arguments going _in_ to the function can never be contravariant, while the functions themselves can be, is that you'd be able to make a call to a function that requires some aspect of a subtype, but pass in a supertype that is missing that functionality instead. For example, you could pass in a `Vehicle` to method with a `Car` parameter:

```kotlin
// Intentionally broken
open class Vehicle
class Car(
    var tyrePressure: Double
) : Vehicle()

fun pumpTyres(car: Car) {
    car.tyrePressure += 50.0
}

fun main() {
    val vehicle = Vehicle()
    pumpTyres(vehicle)
}
```


### Producers and consumers

We've discussed that fact that a complex type can only be contravariant when their component is being passed _in_ to the complex type. What hasn't been mentioned explicitly is that complex types can only ever be *co*variant when the component is being read _out_ of the complex type (so, the opposite of contravariance).

As such:
- covariant complex types are often called _producers_ because values come out of them
- contravariant complex types are often called _consumers_ because values are passed into them\*

### Paint a picture

So, in summary:

![Variance diagram](/assets/images/what-the-kotlin-variance/variance.png)

## In, out, and all about

In Kotlin, generic types are invariant by default (they're neither covariant nor contravariant) because without additional help, the compiler can't know whether a given generic type is a producer or consumer. If it made assumptions, you could end up with a covariant consumer or contravariant producer (which are the two illegal configurations). As such, Kotlin supports _declaration-site variance_, which means that you can decide on the variance of generic types when you define them.

> To re-iterate, _generic types_ are invariant by default, but not functions, which are contravariant. If functions were invariant , you could never pass a subtype in the place of a supertype and would need to create function overloads for the whole inheritence tree.

To make a generic type:
- covariant, use the `out` keyword
- contravariant, use the `in` keyword

But, if invariance is the default and you have be explicit if you want the compiler to act any differently, why is `List<Car> : List<Vehicle>` by defult? That's because [the definition of the generic list in Kotlin is explicitly covariant](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-list/):

```kotlin
public interface List<out E> : Collection<E>
//                     ^
//              The interesting bit
```

Likewise, types that are contravariant out of the box, like `Comparable<T>` [are explicitly defined as contravariant](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-comparable/):

```kotlin
public interface Comparable<in T>
//                          ^
//                  The interesting bit
```

### On using `in`

You're much more likely to see `out` in the wild than `in`, not least because the place that you're going to need contravariance (in functions and their parameter types) has it by default. You can, however, use contravariance to create types that can be scoped down from taking less specific types to taking more specific ones. For example, if you have a `Comparable<Number>`, but for whatever reason you want to scope it down and use it as if it's specifically made for `Double`s, its contravariance means you can do that:

```kotlin
fun getNumberComparable(): Comparable<Number> { ... }

val scopeDownDoubleComparable: Comparable<Double> = getNumberComparable()
```

> If you, dear reader, have a nice example of other better use-cases, please let me know and I'll add them here.

## Closing out

That's covariance. This was a slower post than most of these will be, just because variance can be confusing (at least contravariance was for me at first). In the next post I'll run through everything you need to know to figure out what's going on here:

```kotlin
inline operator fun <reified T : Component> Entity.get(clazz: KClass<T>): T
```

## Gotcha - MutableList<T> is a subtype of List<T>

Because [`MutableList<T>` is actually a subtype of `List<T>` in Kotlin](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-mutable-list/), it's possible to abuse  the immutable list's covariance and the functions' default contravariance to wind up in exactly the bad situation we described in **Why lists can be covariant in Kotlin**.

Because `MutableList<T>` is a subtype of `List<T>`, it can be assigned to a `List<S>`, where `T : S` because `MutableList<T> : List<T> : List<S>`. If we then cast the resultant list to `MutableList<S>`, we've ended up with a list that'll allow us to insert objects of any subtype of `S`, including those that are _not_ `T`. For example:

```kotlin
interface Vehicle
class Car : Vehicle
class Motorbike : Vehicle

fun main() {
    val list = mutableListOf<Car>(Car())
    val superList: List<Vehicle> = list

    (superList as MutableList<Vehicle>).add(Motorbike())

    for (v: Vehicle in list) {
        println(v::class)
    }
}
```

results in:

```kotlin
class Car
class Motorbike
```


You'll have noticed that the example above uses a `for` loop, rather than `Iterable<T>::foreach`. That's because `foreach` casts each element in the list to the generic type of the list while iterating through them. So, swapping out the for loop for 

```kotlin
list.forEach { println(v::class) }
```

results in 

```kotlin
java.lang.ClassCastException: class Motorbike cannot be cast to class Car
```