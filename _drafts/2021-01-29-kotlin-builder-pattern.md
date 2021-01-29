---
layout: post
title: "Kotlin Builder Pattern"
date:  "2021-01-20 11:43"
description: "
The Builder Design Pattern is one of most popular and useful construction patterns in software engineering. In this post I will try to explain it and show how you can use it with Kotlin. Sadly I often see implementations that are simple translation from Java rather that utilizing cool Kotlin syntactic sugar."
permalink: "kotlin-builder-pattern"
comments: true
toc: true
tags:
- design patterns
- Kotlin
- Builder Pattern
  
categories:
- Design Patterns
      
image: /assets/posts/kotlin-builder-pattern/pkin.jpg

---

## Purpose
Builder pattern is used to simplify creating complex objects with non-trivial building logic, or with many constructor parameters. It allows making immutable objects because all properties can be set by the Builder with no need to use object setters.

Builder sort of removes from the user the need to understand the internal object create implementation and guarantees correct object setup or returning an error.

The advantage of using Builder over traditional constructor is being able to pass many `vararg` arguments, because every Builder method can take one, while the whole constructor can take only one.

Builder solves problem of telescopic constructors, when many variants of constructor are created with increasing number of arguments.
```kotlin
constructor(firstName: String): this(firstName, "", 0)
constructor(firstName: String, lastName: String): this(firstName, lastName, 0)
constructor(firstName: String, lastName: String, age: Int): this(firstName, lastName, age)
```
Technically they allow you to use the constructor with just enough arguments you want to set, but in practice adding a new field to the class forces you to modify each constructor. Fortunately in Kotlin you can use named arguments and you don't have to mimic Java.

From my personal experience, you will more often use Builder than create your own, but I believe it's worth understanding how it works and be comfortable using it when the need comes.

### Example usage
Because of my professional bias, examples are coming from Android world.

#### NotificationBuilder
```kotlin
val notificationBuilder = Notification.Builder(this, "channelId")
notificationBuilder.setContentTitle("Title")
notificationBuilder.setContentText("Content")
notificationBuilder.setSmallIcon(R.mipmap.ic_launcher)
val notification = notificationBuilder.build()
```
Most traditional usage of the Builder Pattern. Builders Constructor takes 2 arguments necessary for proper object creation, other fields are getting values through setter methods called on the Builder instance. Finally the `build()` method is called that returns desired notification object.

#### Dexter
```kotlin
DialogOnAnyDeniedMultiplePermissionsListener.Builder
        .withContext(context)
        .withTitle("Camera permission")
        .withMessage("Camera permission is needed to take pictures of your cat")
        .withButtonText(android.R.string.ok)
        .withIcon(R.mipmap.my_icon)
        .build()
```
In this example Builder methods are connected in chain. It's possible because each one of them returns Builder instance, so `this`. With proper method naming you can almost read it like a sentence.

#### AlertDialog
```kotlin
val dialog = AlertDialog.Builder(this)
        .apply {
            setTitle("Title")
            setIcon(R.mipmap.ic_launcher)
        }.show()
```
Very Kotlin style with utilizing the `apply`. Interestingly enough there is no `build()` method but `show()` that is not only returning dialog object but also displays it. Sounds like a bad idea for a method to do more than one thing, but in this case I believe it was done on purpose to avoid a common mistake of creating a dialog but forgetting to display it with a separate method.

## Elements
Builder is basically single internal helper class.

### Constructor
Surprisingly, Builders constructor is very important, even when it doesn't usually take any arguments. Constructor should require all arguments that are needed to build correct object. You can't expect Builder user will know which setters to use, or will read documentation :)
> It seems obvious, but some time ago on Android platform you could legaly build a notification that had no chance to be displayed by the system. All you had to do was to not set the title or content text or an icon - none of those things were required by the Builder constructor. There was also no exception thrown when trying to display such notification...

### Methods
Besides constructor Builder gives you methods to set the object.

In case of not setting some property with a dedicated method, default value should be used. It can also be `null`. Thanks to great nullability handling in Kotlin it is much better to use default `null` value than for example: "" (empty `String`) or magic value like `-1` when only positive number is expected. To detect if value was set or not simple null-check `.?` can be used instead of comparing to some default value used for certain type in project or class.

Methods should be allowed to be called in any given order.

Mandatory `build()` method, or other reasonably named (like `show()` for dialog) returning the desired object.

When using Builder Pattern, it's good to make objects constructor private to limit creating the object just for internal Builder.

### Verifying the arguments
When using Builder you pass wrong values, not allowed in the object, when should you be notified about it?
There are at least few approaches:
1. ASAP, when Builder method gets wrong argument it should check if its OK for the object, but:
    - what if correct argument value depends on argument set by other method? If order of calling methods matters then what is the point of even using Builder over traditional constructor?
    - checking values at Builder methods may not be enough, `build()` method should also verify arguments all together
2. Only the `build()` method should check all arguments at once, because there may be some dependencies between them, and this was the first thing why the Builder Pattern was used, so:
    - well constructed Builder allows chaining methods so validation in `build()` method isn't much later that checking in each setter method
    - dependencies between arguments can be verified
    - but if Builder is not the only way of creating the object then validations have to be copy-pasted to every place that is creating the object
3. The object itself should verify its correctness
    - minding SRP (Single-Responsibility Principle), Builder just constructs the objects with all required data, but object can verify if the data is correct
    - no need to copy-paste validations, every place creating the object will have to pass the same checks

[Interesting thread at StackExchange about it](https://softwareengineering.stackexchange.com/questions/241309/builder-pattern-when-to-fail),
where some users suggest merging approach 2 and 3. The point is for Builder to verify its contracts, and for the object to verify its own contracts. Nice example of Builder that creates a String containing number ranges like "1-2,3-4,5-6". `String` class can't verify if range edges have correct values - it was made to just handle string of characters. But Builder on the other hand, can and should check if added range makes sense, or if ranges are not overlapping if that is the requirement. Then the method `addRange(min Int,max Int)` should throw `IllegalArgumentException` when `min > max` and `build()` method should throw exception when ranges like `1-4` and `2-6` are added. Or maybe the `addRange()` method should throw, that's debatable.

In any case, I would stick to the rule that created object verifies its contracts, and Builder checks its own contracts.

[Joshua Bloch](https://pl.wikipedia.org/wiki/Joshua_Bloch) in his famous book "Effective Java"
(chapter 2, topic 2) also suggests verifying value correctness after coping values from Builder, not in Builder itself.

## Implementation
I guess this is the part you were waiting for :)

### Java style
Barebone example of the simplest possible Builder. Basically, Java translated to Kotlin without using any language fireworks.
We have here:
- private constructor in `Product` to limit creating instances only to inner Builder class
- required `requiredProperty` argument in Builders constructor
- optional field set by `optionalProperty()`, default value is `null` and that is OK for `Product`
- method `optionalProperty()` is returning `this` to allow method chaining
- `build()` method creating instance of `Product` with all fields set with values from Builder

```kotlin
class Product private constructor(
        val property: Any,
        val optionalProperty: Any?
) {

    class Builder(private val requiredProperty: Any) {
        private var optionalProperty: Any? = null

        fun optionalProperty(value: Any?): Builder {
            this.optionalProperty = value
            return this
        }

        fun build(): Product {
            return Product(requiredProperty, optionalProperty)
        }
    }
}
```
And usage
```kotlin
val product = Product.Builder("required")
        .optionalProperty("optional")
        .build()
```

### More like Kotlin
Builders constructor specifies all fields, and their possible default values. It could be a `data class` but in this case it wouldn't provide any additional value. Lack of default value in the constructor is making argument mandatory.

Using `apply` makes `optionalProperty()` to return instance of the Builder. I also used so called [single-expression function](https://kotlinlang.org/docs/reference/functions.html#single-expression-functions) meaning no explicit return type declaration.
```kotlin
class FancyProduct private constructor(
        val property: Any,
        val optionalProperty: Any?
) {

    class Builder(
            private var requiredProperty: String,
            private var optionalProperty: Any? = null,
    ) {

        fun optionalProperty(value: Any) = apply { this.optionalProperty = value }

        fun build(): FancyProduct {
            return FancyProduct(requiredProperty, optionalProperty)
        }
    }
}
```
Such Builder can be used in few ways:
- identical as in Java or Kotlin example above
- providing both arguments (required and optional) at once in Builder constructor
- providing both arguments but in random order using named arguments
```kotlin
val fancyProduct = FancyProduct.Builder("required")
        .optionalProperty("optional")
        .build()

val fancyProduct2 = FancyProduct.Builder(
        "required",
        "optional"
).build()

val fancyProduct3 = FancyProduct.Builder(
        optionalProperty = "optional",
        requiredProperty = "required"
).build()
```

### Kotlin DSL
[DSL (Domain Specific Language)](https://en.wikipedia.org/wiki/Domain-specific_language) is a domain-dedicated quasi language. Kotlin allows fairly easy and pleasant method creation, that can be later used to describe objects in a purely domain way.

Builder looks the same as in previous example.

Inside `DslProduct` class there is now a `companion object` with `dslProduct()` method that will allow us to create a desired object. This hides the Builder behind a method.
```kotlin
class DslProduct private constructor(
        val requiredProperty: Any,
        val optionalProperty: Any?
) {
    companion object {
        inline fun dslProduct(requiredProperty: Any, block: Builder.() -> Unit) =
                Builder(requiredProperty)
                        .apply(block)
                        .build()
    }

    class Builder(
            private val requiredProperty: Any,
            private var optionalProperty: Any? = null
    ) {
       fun optionalProperty(value: Any?) = apply { this.optionalProperty = value }
       fun build() = DslProduct(requiredProperty, optionalProperty)
    }
}
```
Using this simple DSL looks like this.
```kotlin
val dslProduct = dslProduct("required") {
    optionalProperty("optional")
}
```
For such a simple object using DSL doesn't look very tempting, but if object is a composition of other complex objects - each with its own builder than it start to look interesting. Example from one of my apps I created with [DslMaker](https://kotlinlang.org/docs/reference/type-safe-builders.html). It's part of Kotlin standard library (so no extra dependencies) and additionaly it takes care for inner builder scope limits. Address, Location and OpenHours are using Builders and DSL to create instances.
```kotlin
val shop = shop("ID") {
    address = address {
        cityName = "Poznań"
        streetName = "ul. Półwiejska"
        streetNumber = "123/2"
    }
    location = location {
        lat = 53.12
        lng = 23.4
    }
    openHours = openHours {
        weekDay = "6:00-22:00"
        saturday = "7:00-23:00"
        sunday = "closed"
    }
    features(
            Feature.Bakery,
            Feature.Atm
    )
}
```
Full example with DslMaker -> [tutaj](https://gist.github.com/asvid/5fed8dd3c831c2a72744e8ffe8f7dc0f) <- But still this is a fairly simple example and DSL itself in Kotlin deserves a separate post.

However, the DSL approach requires you to write some boilerplate, that doesn't look very inviting at first...

## Alternative
Named arguments in constructor and default field values inside created object may in a way give you similar effect as Builder Pattern. This can be useful for rather simple objects. It's also worth making sure that the default values create a sensible instance from domain point of view, not just so it compiles. 
```kotlin
val person = Person(
        firstName = "Adam",
        lastName = "Świderski",
        address = Address(
                cityName = "Poznań",
                streetName = "ul. Półwiejska",
                streetNumber = "123/1",
                country = "Poland",
                postalCode = "60-000"
        ),
        contact = Contact(
                workEmail = "adam@work.email",
                workPhoneNumber = "+48 123112312",
                privateEmail = "adam@private.email"
        ),
)
```
It even look kinda like DSL, but without need to use annotations and writing additional methods etc. You can notice fields like `val height: Float? = null` - I decided that this information is not required to create sensible `Person` instance so default value is set to `null`. Yup `null` and not `0.0` or `-1.0`, or some other magic `DEFAULT_HEIGHT` constant.

Inside `init` blocks fields values are verified (check `Verifying the arguments` above). Passing wrong argument type is cought at compile time, but object can have some domain requirements like: height have to be a positive number, or shoe size (if provided) can't be smaller than 4.
```kotlin
data class Person(
        val firstName: String,
        val lastName: String,
        val address: Address,
        val contact: Contact,
        val height: Float? = null,
        val shoeSize: Float? = null,
) {
    init {
        height?.let { require(0f < it) { "height is always greater than 0" } }
        shoeSize?.let { require(4f <= it) { "smallest standard shoe size is 4" } }
    }
}

data class Address(
        val country: String,
        val cityName: String,
        val streetName: String,
        val streetNumber: String,
        val postalCode: String,
        val district: String? = null,
)

data class Contact(
        val workPhoneNumber: String,
        val workEmail: String,
        val privatePhoneNumber: String? = null,
        val privateEmail: String? = null,
) {
    init {
        require(workPhoneNumber.isValidPhoneNumber())
        require(workEmail.isValidEmail())
        require(privatePhoneNumber?.isValidPhoneNumber() ?: true)
        require(privateEmail?.isValidEmail() ?: true)
    }
}
```

## Summary
Builder is quite useful Design Pattern and for sure you will come across it at some point. It's good to know how its build and when to use it - because this is not a silver bullet. Kotlin features allow reducing Builder boilerplate and using conveniences like DSL, named parameters and default values may allow you to get similar result without writing any additional code.

In Design Patterns literature you can find more complex examples of Builder using elements like Director and ConcreteBuilder. I never encountered things like that in my career, but I believe there are usecases for it. And if at same point, you need to have generic way of providing instances of objects using many types of Builders - maybe there are nicer ways of achieving this :)