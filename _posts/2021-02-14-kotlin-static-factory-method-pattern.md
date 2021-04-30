---
layout: post
title: "Kotlin Static Factory Methods"
date:  "2021-02-15 11:43"
description: "
`Static Factory Methods` known from Java have their place also in Kotlin, although they look and behave a bit different because there is no `static` word in Kotlin. Here I'll try to show how to use `companion object` for `Static Factory Methods` and more.
PS: This whole post was supposed to be about `Factory Method` with just a short mention about static factory methods, but the topic becomes more interesting than I thought :)
"
permalink: "kotlin-static-factory-methods"
comments: true
toc: true
tags:
- design patterns
- Kotlin
- Static Factory Methods
- construction design pattern
  
categories:
- Design Patterns
- rss
      
image: /assets/posts/lodz.jpg

---

# Purpose

There is a concise error in the title, there are no static methods in Kotlin. But there are ways to achieve similar behavior to proposed by Joshua Bloch in Effective Java book - using static factory methods instead of constructors. This is also completely different from the Factory Method design pattern, don't confuse those.

Long story short, these are methods that create object instances based on supplied arguments (or even without them) and that you can call from anywhere without the need of having an instance of a class that contains them. To do so in Java, you would use the `static` keyword, which means that method is part of a class (understood as a type) rather than an object, and you can call it without creating an instance. We don't have this possibility in Kotlin, but we can get a similar effect with `companion object`.

# Example usage
Even if the phrase "static factory method" sounds alien, I'm sure you saw it being used in code. Typical static factory methods may look like this:
```kotlin
val locale = Locale.forLanguageTag("PL")
val today = LocalDate.now(ZoneId.of("GMT"))
val time = LocalDateTime.of(2021, 1, 1, 12, 34, 23, 123)
val someNumber = Double.fromBits(0x10000000000000L)
val formattedPi = String.format("%.2f", 3.14159265358979323)
```
Also, every primitive type in Kotlin is created using one, not with a constructor.

# Elements
The main rule here will be to use well-named methods instead of constructor, that should be private just like in Builder Pattern. Popular factory method names are:
- `from` or `valueOf` - type conversion method, it takes a single argument and returns another object with the same value
    ```kotlin
    val instant = Instant.now()
    val date = Date.from(instant)
  
    val prime = BigInteger.valueOf(Long.MAX_VALUE)
    ```
- `of` - the aggregating method, that takes multiple arguments and returns instance containing all of them
    ```kotlin
    val cutlery = EnumSet.of(Fork, Spoon, Knife)
    ```
- `instance`, `getInstance` or `instanceOf` - a method returning instance, often seen in Singletons (so not really in Kotlin :) ) it may take arguments and return the same instance for provided values but it's not guaranteed
    ```kotlin
    val luke: StackWalker = StackWalker.getInstance(options)
    ```
- `create` or `newInstance` - similar to `getInstance` but this time a new instance is guaranteed each time
    ```kotlin
    val array = Array.newInstance(String::class.java, 10)
    ```
- `get<<Type>>` - similar to `getInstance` but returns the same instance of the object when the factory method is in a separate class. `<<Type>>` is the type of returned object
    ```kotlin
    val path = Path.of("path", "to/file/store")             // already familiar method 'of()'
    val fileStore: FileStore = Files.getFileStore(path)     // <<Type>> to FileStore
    // in this case for the same `Path` instance, new `FileStore` instance will be created each time
    ```
- `new<<Type>>` - analogically to the previous example but for getting a new instance each time
    ```kotlin
    val br: BufferedReader = Files.newBufferedReader(path)
    ```

The above-mentioned names are fairly generic, but nothing should stop you from using more descriptive like `forLanguageTag` or `fromBits` like in previous examples. The most important thing is for the static factory method name to go along with already known name patterns for this use-case. There is no other way to distinguish it from "normal" methods (that don't return an instance) at the first sight.

## Companion Object

So in Java, such methods had to be static members of a class, but in Kotlin we have `companion object`. It allows us to call the method without explicitly creating an instance of a class (but in fact, `object` is an instance). Programmers deeply anchored in Java often use it just as a container for keeping constants and methods that they want to be `static`, but it has many interesting features:
- `companion object` is a special case of an inside-class `object` so a Kotlin Singleton
- Its fields and methods behave similar to `static` ones in Java, but are still members of an instance with all its consequences, like access modifiers.
- It can implement an interface or extend a class.
- Class having `companion object` has access to its fields, even private ones. 
- Interfaces also can have `companion object`. 
- It's not being inherited, subclasses don't have access to parents object.
- It's not accessible from the instance, just like the inner `object`.
- It can have a name, but it's optional - by default it can be reached with the `Companion` name.
- There can single `companion object` inside a class, but any number of normal `objects`.
- Unlike `inner` classes it doesn't have access to fields and methods of the parent object, just like inside `object.
- Parent class name already points to `companion object`, there is no need to call it like `<<Class>>.Companion.<<method()>>`.

```kotlin
// the companion object in InterestingObject class is extending this abstract class
abstract class Printer { 

    abstract fun getName(): String

    fun print() {
        val name = getName()
        println(name)
    }
}
// private constructor so instance of this class can't be created in traditional way
class InterestingObject private constructor() { 

    object Factory : Printer() { // ordinary inside object
        @JvmStatic // generating true static methods for Java interoperability
        fun create() = InterestingObject()

        // only companion object override methods can be @JvmStatic
        override fun getName(): String {
            return "Interesting Object"
        }
    }

    companion object CompanionFactory : Printer() { // companion object with a custom name
        private val secret = "No one can know this" // private companion object field, not accessable outside the class
        val publicInfo = "***** ***" // publicly available field
        
        fun create() = InterestingObject()
        @JvmStatic 
        override fun getName(): String {
            return "Interesting Object from Companion"
        }
    }
    // instance method has access to private companion object field
    fun getSecret() = secret
}

val obj0 = InterestingObject() // error, private constructor doesn't allow to create instance this way
val obj1 = InterestingObject.create() // method from CompanionFactory so from the companion object
val obj2 = InterestingObject.Factory.create() // method from Factory object
val obj3 = InterestingObject.CompanionFactory.create() // companion object method again but with unnecessary companion object name

val secret = InterestingObject.CompanionFactory.secret // error, no access to the private field in companion object
val secret2 = InterestingObject.create().getSecret() // but you can get it from the instance method
val publicInfo = InterestingObject.publicInfo // this field is accessable from companion object

// calling abstract class methods
val print1 = InterestingObject.print()
val print2 = InterestingObject.Factory.print()
val print3 = InterestingObject.CompanionFactory.print()

// error, you can't call in-class objects from the instance
InterestingObject.create().Factory.getName()
InterestingObject.create().CompanionFactory.getName()
```

The code example below shows incrementing variable `counter` each time when a new `CountMe` instance is created. Like I already wrote subclass `ReallyCountMe` doesn't have access to its parents `companion object`. Nonetheless, when a new `ReallyCountMe` instance is created `counter` is also being incremented. 

It happens because parent constructor and `init` bloc is called each time, and `counter` field is part of Singleton - the `companion object`, so each time instance is created (parent or subclass) the same variable `counter` is incremented.

```kotlin
open class CountMe(val objectName: String) {
    init {
        count++  // incrementing counter in companion object
    }
    companion object {
        var count = 0 // every CountMe instance have access to this field
        fun printCount() = println("count = $count")
    }   
}

class ReallyCountMe(private val name: String): CountMe(name)

// usage
CountMe.printCount() // prints 0
CountMe("1")
CountMe("2")
CountMe("3")
CountMe.printCount() // prints 3

ReallyCountMe("1")
ReallyCountMe("2")
ReallyCountMe("3")

CountMe.printCount() // prints 6
```
Every `CountMe` instance has a reference to the same (and the only) `companion object` instance.

Another example from library `Fiel` (simple HTTP client), where `Result` subclasses are created with `companion object` methods:
```kotlin
sealed class Result<out V : Any?, out E : Exception> {
...
companion object {
        // Factory methods
        fun <E : Exception> error(ex: E) = Failure(ex)

        fun <V : Any?> success(v: V) = Success(v)

        fun <V : Any?> of(value: V?, fail: (() -> Exception) = { Exception() }): Result<V, Exception> =
                value?.let { success(it) } ?: error(fail())

        fun <V : Any?, E: Exception> of(f: () -> V): Result<V, E> = try {
            success(f())
        } catch (ex: Exception) {
            error(ex as E)
        }
    }
```

### Companion Object Extensions
There are times where you don't have the chance to modify class `companion object`, like when it comes from 3rd party library. But if the class has the `companion object` then you can use `extension functions` on it.

```kotlin
// --- external module, like 3rd party library ---
class ExternalClass {
    companion object
}

class ExternalClassWithoutCompanion

interface Car{
    companion object
}

// --- internal module ---
class Ferrari: Car // internal class implementing interface from the library
fun Car.Companion.getFerrari():Ferrari = Ferrari() // library class extension method creating instance of internal class

// usage
val ferrari = Car.getFerrari()

// other extensions function examples
fun ExternalClass.Companion.newMethod() = "Wow! Extending class companion"
fun Car.Companion.newMethod() = "Wow! Extending interface companion"
fun ExternalClassWithoutCompanion.Companion.newMethod() = "Wow!" // error, this class has no companion object
fun ExternalClassWithoutCompanion.newMethod() = "Wow!" // extension method for the class instance

// usage
ExternalClass.newMethod() // new extension method of ExternalClass companion object 
ExternalClassWithoutCompanion.newMethod() // error, newMethod() is instance method, not companion object
ExternalClassWithoutCompanion().newMethod() // this will work, extension method is used on the instance
```

Again using `Fuel` library, where `Result` class has `companion object`. `Result` class itself is `sealed` so you cant extend it outside its module.
```kotlin
fun Result.Companion.awersomeNewMethod(){
    println("This wasn't originally here")
}

//usage
Result.awersomeNewMethod()
```

Using `companion object` can make clients of your APIs lives much easier, even if you put an empty one in a public class or interface. It allows them to extend functionality without building wrappers or extending classes - and this is why a lot of Kotlin features exist in the first place.

# Alternative - Top-level functions

Another interesting way of creating instances are `top-level functions`, known for example from:
```kotlin
val intList = listOf(1, 2, 3)
val someSet = setOf("1", "2", "3")
val someMap = mapOf(1 to "1", 2 to "2")
```
They are especially useful for creating simple but often used objects like lists or maps. Like the name may suggest those functions exist outside (above?) of the classes, and for that reason, they are publicly available (ok, they can be also private but whats the point then?). You should be aware of accidental function shadowing, for example:
```kotlin
// adding such function in the project
fun listOf(vararg item: Int) = item.asList()

// we are shadowing function from kotlin.collections without any trace in code it self (no additional imports)
val intList = listOf(1, 2, 3)   // our top-level function
val intList2 = listOf("1", "2") // function from kotlin.collections
```
It's not necessarily evil, but it may surprise you. So it's smarter to avoid too generic names, and not abuse `top-level functions`. Also to not litter IDE suggestions, because they will be popping out there.

# Summary
Kotlin with its syntactic sugar, such as named arguments, makes writing code much more pleasant than Java. Still using methods to create instances instead of constructors is often a good idea. Static Factory Methods are often used in Kotlin itself or by 3rd party library creators.

I tried to show interesting ways of using `companion object` which **is not just a container for constants**, or is emulating the `static` keyword from Java. Kotlin creators are extensively using it, for example in Coroutines - so I guess we should too.

## Pros
- **methods have descriptive names**
  
  Some languages allow naming constructors, unfortunately not Kotlin or Java. Static Factory Methods are a way of achieving this functionality. 
  
- **methods can take same the argument types in the same order**

  As long as they have different names, the list of arguments may be identical. You can't get this with a barebone constructor. Named arguments and default values may cover this issue just a bit.

- **you don't always need a new instance**

  The constructor will always create a new instance, but method may first check some cache and return instance that is already created.

- **any subtype can be returned**

  This subtype may not even be public. You can create a `companion object` inside an interface and return instances of the internal types that implement it. Client will see only the public interface.

## Cons
- **you can't extend a class that have only a private constructor**

  But this can always be a strong suggestion to use composition over inheritance :)

- **no easy way to distinguish Static Factory Methods from usual methods**

  Using constructor is clearly visible in the code, but the factory method looks like any other method. The automated documentation generation also won't be picking those methods as ways to create instance. Using common nomenclature for naming static factory methods may help, examples are [here](#elements)