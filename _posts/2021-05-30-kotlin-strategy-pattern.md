---
layout: post
title: "Strategy Pattern in Kotlin"
date:  "2021-06-05 11:43"
description: "
The `Strategy` pattern creates a family of algorithms, enclosing the differing logic in separate classes while hiding it from clients behind the interface. It enables the interchangeable use of implementations. The use of the strategy simplifies the customer code, avoids code duplication and conditional statements. Significantly simplifies testing - by separating client testing from strategy algorithms.
"
permalink: "kotlin-strategy-pattern"
comments: true
toc: true
tags:
- design patterns
- Kotlin
- Strategy Pattern
- behavioral design pattern

categories:
- Design Patterns
- rss

image: /assets/posts/strategy.jpg

---

# Purpose

The `Strategy` design pattern defines a family of algorithms and allows them to be used interchangeably. By `algorithm`, here I mean any logic, be it sorting, searching, or computing some value from data. It does not matter. It is, in a sense, an extension of the [Template Method](https://asvid.github.io/kotlin-template-method) pattern, but inversely to it, `Strategy` prefers composition over inheritance. Strategies do not inherit from any specific class but only implement a common interface. This allows for easy code encapsulation and algorithm replacement without the inheritance overhead.

## The Problem
An example of a problem that can be solved by the `Strategy` may be the way of calculating the price taking into account the promotion kind: 
```kotlin
data class Item(val name: String, val price: Double) // product on the bill

enum class Promotion { // enum with promotion kinds
    NoPromotion, SpecialPromotion, ChristmasPromotion
}

class Bill {
    private val items = mutableListOf<Item>() // list of producsts on the bill
    fun addItem(item: Item): Bill {
        items.add(item)
        return this
    }

	// method calculating final price from list of items and selected promotion
    fun calculateFinalPrice(promotion: Promotion): Double {
        val initialSum = items.sumOf { it.price }
		
        // checking promotion and using right algorythm to calculate price
        return when (promotion) { 
            Promotion.NoPromotion -> initialSum
            Promotion.SpecialPromotion -> when {
                initialSum > 20 -> initialSum * 0.95
                initialSum > 30 -> initialSum * 0.85
                initialSum > 40 -> initialSum * 0.75
                else -> initialSum
            }
            Promotion.ChristmasPromotion -> initialSum * 0.80
        }
    }
}
```
> I'm aware that `Double` is not the best type to use for money operations, but for the ease of use in this post examples I decided to use it.

We have 3 promotions here, one of which `NoPromotion` does not change anything. The implementation of the method calculating the promotion is in the `Bill` class - so in addition to collecting products, the class also calculates the final price, we do not have the `Single Responsibility Principle` preserved here.

If there happens to be a new promotion request, just add an `enum` and its implementation inside `Bill` class. The IDE will report that the new case is not handled if you forget. It doesn't look extremely bad, other than the lack of SRP.

Now let's assume that in addition to the standard receipt, you want to be able to issue an invoice that takes into account the same promotions.
```kotlin
class Invoice {
	...
}
```
You could copy the code that calculates the promotion, or you could artificially extract some abstract receipt class with promotion implementation from which `Invoice` and `Bill` would inherit. Unless, of course, they are not already inheriting from another class.

And then there will be some class that does not fit into this hierarchy, which will also require the implementation of promotions...

# Implementation
And then the 'Strategy' comes in, all in white [^na_bialo]. It allows you to easily transfer the method of calculating individual promotions to separate classes with a common interface. In such a way that customers do not even need to know what specific promotion are they using.

## Abstract
Let's start from abstract implementation to understand all pieces of this pattern:
```kotlin
// the client class, strategy is provided in the constructor
class Context(private val strategy: Strategy) {
    // using generic strategy interface
    fun useStrategy() = strategy.use() 
}

interface Strategy { // using interface instead of class is very important
    // abstract or concrete class would limit using the strategy only to its hierarchy
    fun use() // strategies usually have single public method
}

class StrategyA : Strategy { // first strategy
    override fun use() { // concrete algorithm implementation
        println("using strategy A")
    }
}

class StrategyB : Strategy { // second strategy
    override fun use() {
        println("using strategy B")
    }
}

fun main() {
    // using either strategy is identical
	// strategies are transparent for the client
    val contextA = Context(StrategyA())
    contextA.useStrategy()
    val contextB = Context(StrategyB())
    contextB.useStrategy()
}
```
We have here the `Context` class, so the client using the Strategy. It only knows the strategy interface, not the concrete classes. This allows you to easily expand the family of strategies with a new one, without updating the customers. Due to the use of the interface, and not the `Strategy` class, specific Strategies are loosely related to each other, while guaranteeing clients a common API.

It can be represented symbolically like this:

{% plantuml %}
@startuml

interface Strategy{
+ use()
  }

class StrategyA implements Strategy{
+ use()
  }
class StrategyB implements Strategy{
+ use()
  }

class Context{
	- strategy: Strategy
	+ useStrategy()
}

Context::strategy -> Strategy

@enduml
{% endplantuml %}

## The solution
Promotions can be encapsulated in specific classes:
```kotlin
interface Promotion {
    fun calculate(sum: Double): Double
    val name: String
}

// singleton, because this strategy doesn't need to keep its state - but it could
object ChristmasPromotion : Promotion {
    override val name = "Christmas Promotion"
    override fun calculate(sum: Double): Double {
        return sum * 0.8
    }
}

// this is a NullObject, a special case of Strategy not performing any actions
object NoPromotion : Promotion { 
    override val name = "No Promotion"
    override fun calculate(sum: Double): Double {
        return sum
    }
}

object SpecialPromotion : Promotion {
    override val name = "Special Promotion"
    override fun calculate(sum: Double): Double {
        return when {
            sum > 20 -> sum * 0.95
            sum > 30 -> sum * 0.85
            sum > 40 -> sum * 0.75
            else -> sum
        }
    }
}
```
Having such implementation of promotions, the `Bill` class gets simplified to:
```kotlin
class Bill {
    ...
	// strategies can be passed in the constructor, or in the method that uses them
    fun calculateFinalPrice(promotion: Promotion): Double {
        println("applying ${promotion.name}")
        val initialSum = items.sumOf { it.price }
        return promotion.calculate(initialSum) // the promotion object calculates the price
    }
}
```
Maybe this is not the best example, because we still don't have SRP - the `Bill` still calculates the final amount, but now at least delegates it to the` Promotion` object.
Adding a new type of promotion doesn't cause any update in the `Bill` class. The same promotion classes can be used in the `Invoice` class or any other class that needs to include them.

It is much easier to test the logic in separate classes than in the initial example with the `when` condition. Adding another promotion will not cause the need to fix the existing tests, but only add new ones for the new class.
In order not to spoil this testing awesomeness, make sure that `Strategy` gets all the values it needs in the public method or the constructor, rather than magically extracting them from some configuration.
Often the 'strategy' does not need to store its state, but only performs some actions on provided data. This is one of the few cases where the use of `Singleton` makes sense.

## Multiple strategies
OK, with single strategy it's cool, but nothing stops us from making `Bill` have many kinds of strategies. Final price may vary depending on taxes or loyalty program.
```kotlin
interface Tax {
    fun applyTaxes(sum: Double): Double
}

interface Promotion {
    fun applyPromotion(sum: Double): Double
}

interface LoyaltyProgram {
    fun applyPolicy(sum: Double): Double
}
```

Strategies can be passed as a parameter in the method where they are to be used, but they can also be passed in the constructor of a client object. Below is an example with default values (probably the simplest kind of [Builder](https://asvid.github.io/kotlin-builder-pattern) in Kotlin), which allows you to overwrite only those strategies that are actually to be different than the default ones.
```kotlin
class Bill( 
	// all strategies here are `object`s, 
	// their implementation is an irrelevant detail
    val tax: Tax = DefaultTax,
    val promotion: Promotion = NoPromotion,
    val clientPolicy: LoyaltyProgram = NewClient
)

val newClientAnarchist = Bill(
        tax = NoTax, // well it's not how it works in real life...
        clientPolicy = NewClient
)

val returningClientWithSpecialPromotionBill = Bill(
        clientPolicy = ReturningClient,
        promotion = SpecialPromotion
)
```
A [Static Factory](https://asvid.github.io/kotlin-static-factory-methods) can also become handy
```kotlin
class Bill private constructor (
        private val tax: Tax,
        private val promotion: Promotion,
        private val clientPolicy: ReturningClientPolicy
) {
    ...
    companion object Factory{
        val defaultTax = DefaultTax
        val defaultPromotion = NoPromotion
        val defaultClientPolicy = NewClient

        fun returningClient(): Bill = Bill(
                defaultTax, defaultPromotion, ReturningClient
        )

        fun returningClientWithSpecialPromotion() = Bill(
                defaultTax, SpecialPromotion, ReturningClient
        )

        fun newClientAnarchist() = Bill(
                NoTax, defaultPromotion, NewClient
        )
    }
}
```
**But now class `Bill` becomes aware of at least some concrete implementations of `Strategy`**

The calculation of the individual components of the final amount must be performed in a fixed order, you cannot charge tax from promotion, etc. Similar to the [Template Method](https://asvid.github.io/kotlin-template-method) approach, the `Bill` class is responsible for the correct sequence of steps, but the Strategy pattern allows the implementation of these steps to be replaced.
```kotlin
fun calculateFinalPrice(): Double {
	val initialSum = items.sumOf { it.price }
	return initialSum.run { 
		promotion.applyPromotion(this) // `this` is the initial sum
	}.run {
		clientPolicy.applyPolicy(this) // now it's the amount after applying the promotion
	}.run {
		tax.applyTaxes(this) // and after loyalty policy
	} // returning final amount after all the modifiers
}
```
Different taxes would apply to specific types of products rather than the entire bill. However, assuming that tax law is constantly changing, the use of the `Tax` strategy allows for quick response to new regulations without the need to update strategy clients.

I admit that I'm not a fan of such operations queuing with the `run ()` methods. Fortunately, this can be improved by using the extension functions:
```kotlin
fun Double.applyPromotion(promotion: Promotion): Double {
    return promotion.applyPromotion(this)
}

fun Double.applyPolicy(policy: LoyaltyProgram): Double {
    return policy.applyPolicy(this)
}

fun Double.applyTaxes(tax: Tax): Double {
    return tax.applyTaxes(this)
}
```
And then we have:
```kotlin
fun calculateFinalPrice(): Double {
    val initialSum = items.sumOf { it.price }
    return initialSum
            .applyPromotion(promotion)
            .applyPolicy(clientPolicy)
            .applyTaxes(tax)
	// nice :)
}
```

### Invoke
Because the `Strategy` tends to have a single public method, you can consider using the `invoke()` operator. Additionally, the `annonmous class` instead of a default implementation of the strategy, to not multiply the `NullObjects`:
```kotlin
interface Tax {
    // having this allows to use object as a method
    operator fun invoke(sum: Double): Double
}
...
class Bill(
        val tax: Tax = object : Tax {
            override fun invoke(sum: Double): Double {
                return sum
            }
        },
		...
) {
    ...
    fun calculateFinalPrice(): Double {
        val initialSum = items.sumOf { it.price }
        return initialSum.run{
            promotion(this) // calling the `invoke()`
        }.run {
            clientPolicy(this)
        }.run {
            tax(this)
        }
    }
}
// using annonmous classes allows you to create new strategies on the fly
// but not having them as a concrete class kills a lot of benefits of the pattern
val customBill = Bill(promotion = object : Promotion {
    override fun applyPromotion(sum: Double): Double {
        return sum * 0.123512
    }
})
```

# Naming
Naming conventions may vary from team to team or project to project. I personally prefer the meaningful domain name rather than using pattern-function-part modifiers.
```kotlin
// I like this more
class SpecialPromotion: Promotion{
    fun calculate(initialPrice: Double): Double{
        ...
    }
}
// than this
class SpecialPromotionStrategy: PromotionStrategy{
    fun use(initialPrice: Double): Double{
        ...
    }
}
```
Using the first style, I don't even have to think if I'm using the `Strategy Pattern`. I'm using the domain `Promotion` class that knows how to calculate the final price with its internal rules. What information does it really give me to have the `Strategy` in the name? Should we also add `Singleton` to every Kotlin `object` name?

This preference stands for all design patterns. Sadly common practise (especially in older projects) is to strictly stick to often unwritten rules. So you can immediately know that the class is part of particular pattern. Because someone (like a junior-dev with 20 years of experience) has memorized a book with patterns and is able to implement it only fallowing the template strictly.

If you are interested in this topic, I can recommend a great talk by [Kevlin Henney - Seven Ineffective Codding Habbits of Many Programmers](https://youtu.be/ZsHMHukIlJY?t=1517), where amongst others, he talks about naming.

# Summary
The `Strategy` pattern creates a family of algorithms, enclosing the differing logic in separate classes while hiding it from clients behind the interface. It enables the interchangeable use of implementations. The use of the strategy simplifies the customer code, avoids code duplication and conditional statements. Significantly simplifies testing - by separating client testing from strategy algorithms.

This pattern should be used quite often, even if initially the whole "family" of strategies will consist of 1 class. The advantages of encapsulating code outweigh the disadvantages of adding an interface and a new class. Often, sooner than later, it turns out that a given algorithm needs to be used somewhere else, or there is a need to add another one. 
However, don't overdo it. There will be places where creating the Strategy is pointless, when an algorithm is 1 line of code used in 1 place with no prospect of spreading.

`Kotlin` offers interesting possibilities for using the Strategy, thanks to named arguments, using the` invoke () `operator and the` extension functions`. However, you should pay attention to whether the syntactic sugar makes it difficult to test or to use the code elsewhere.

## Pros
- **algorithm encapsulation** - the entire algorithm is inside a separate class, ready to be used in any place of the system.
- **composition over inheritance** — no close connection between algorithm and the client.
- **anty-IF policy** - only strategy knows how to process data, client is using the generic interface, so conditional expressions are gone.
- **implementation interchangeability** - different implementations, e.g. sorting, may work better in certain cases. The strategy allows you to quickly provide an "acceptable" algorithm and then correct it without changing the client.
- **ease of testing** - independent client and strategy testing. Changes to the strategy do not force the client tests to be fixed.

## Cons
- **having more object instances** - using `object` so Singletons helps with this issue.
- **can be overused** - if there is **absolutely** no chance that the algorithm will be used anywhere else, or there is a need for an alternate version, then adding a strategy may be unnecessary..., but it will make testing easier anyway.

---

[^na_bialo]: I'm sorry for this very Polish inside joke, but it just fits too good :) origin: https://www.youtube.com/watch?v=FeZYsTVrpMY