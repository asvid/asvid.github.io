---
layout: post
title: "Adapter Pattern in Kotlin"
date:  "2021-06-26 11:43"
description: "
The Adapter or Wrapper Pattern allows you to "translate" one interface into another, expected by the client class. It is especially useful when the adapted object comes from 3rd party library, and you do not want to make your system depending on that interface, creating the so-called `anticorruption layer`. Adaptee interface changes will only affect the `Adapter` and not the rest of the code.
"
permalink: "kotlin-adapter-pattern"
comments: true
toc: true
tags:
- design patterns
- Kotlin
- Adapter Pattern
- structural design pattern

categories:
- Design Patterns
- rss

image: /assets/posts/adapter.jpg

---

# Purpose
As the name suggests, the `Adapter` pattern transforms the class interface to another one requested by the client. Using the `Adapter` allows incompatible classes to interact with each other. Another term for this pattern is `Wrapper`.

The adapter allows you to "map" an adapted interface (`Adaptee`) to the expected interface (`Target`) by the client class without adding another level of inheritance. Such inheritance would not always be possible if `Target` was a class rather than an interface. The new class would have to be both `Target` and` Adaptee` at the same time.

Instead of creating a new `Adapter` class, you may also add a method in the client that takes an object with an `Adaptee` interface, but it will cause the class to expand with a lot of similar methods. The client class can also come from 3rd party dependency, and then it will not be possible to change it.

# Implementation
The `adapter` is essentially a single class, but it's important to understand its surroundings.

{% plantuml %}
@startuml
skinparam groupComposition 1
class Client{
- useTarget()
}
interface Target{
+ method()
}
class Adapter extends Target{
+ method()
- adaptee: Adaptee
}
interface Adaptee{
 + otherMethod() 
}

Adapter::adaptee -right-* Adaptee
Client-right->Target
Adapter::method..>Adaptee::otherMethod

@enduml
{% endplantuml %}

- **Client** - class that uses the `Target` object
- **Target** - interface required by the `Client` class
- **Adapter** - class with the `Target` interface that adapts the` Adaptee` object, most often the adapted object will be assigned to the field in this class
- **Adaptee** - an adapted class with an incompatible interface that we want to use with `Client`

## Abstract
In the code, the diagram above might look like this:
```kotlin
class Client(private val target: Target) {
    private val argument = BigDecimal(10.0)

    fun doWork() {
        target.method(argument)
    }
}
// the client will accept only this interface
interface Target {
    fun method(argument: BigDecimal): Double
}
// and this is the interface we want to use with the client
interface Adaptee {
    fun originalMethod(argument: CustomArgument): CustomResult
}
// the Adapter implementing Target interface and taking the Adaptee in the constructor
class Adapter(private val adaptee: Adaptee) : Target {
    override fun method(argument: BigDecimal): Double {
        val stringArgument = argument.toCustomArgument()
		// calling method from adapted interface
        return adaptee.originalMethod(stringArgument).toDouble()
    }
}

// usage
fun main() {
    val target1: Target = TargetImpl()
    val adaptee: Adaptee = AdapteeImpl()
    val adapter: Target = Adapter(adaptee)

    val client1: Client = Client(target1)
    val client2: Client = Client(adaptee) // error! wrong interface
    val client3: Client = Client(adapter) // using the Adapter with Adaptee instance
}
```
You can see here how the `Adapter` class uses the `Adaptee` object within the `Target` interface. The `Adapter` encapsulates the logic of mapping one interface to another. This avoids unnecessary extending or modifying the client or `Adaptee` only for a specific client.

Different clients can use different adapters of the same `Adaptee` class. Thanks to this, there is no need to adapt `Adaptee` to a specific client or several clients. Just look at this monster:
```kotlin
// Adaptee implements all interfaces expected by all clients
class Adaptee: Target1, Target2, Target3{
	// the only actual method of the Adaptee
    fun originalMethod(argument: CustomArgument): CustomResult
    // interface methods
    override fun target1Method()
    override fun target2Method()
    override fun target3Method()
}
```
And with Adapters:
```kotlin
// unchanged Adaptee class
class Adaptee{
    fun originalMethod(argument: CustomArgument): CustomResult
}
// Adapters can be put in packages closer to the Client than to the Adaptee
class Adapter1(val adaptee: Adaptee) : Target1{
    override fun target1Method()
}
class Adapter2(val adaptee: Adaptee) : Target2{
    override fun target2Method()
}
class Adapter3(val adaptee: Adaptee) : Target3{
    override fun target3Method()
}
```

## List
The adapter will work well for a list of elements on which we want to perform an operation for which they were not created.
```kotlin
// lets assume that this class is comming from 3rd party library
// `data class` cannot be extended
data class Item(val name: String)

// and we have this interface in our system
interface PrettyPrintableItem {
    fun prettyPrint(): String
}
// Adapter/Wrapper providing `PrettyPrintableItem` functionality for the `Item` object
class ItemAdapter(private val item: Item) : PrettyPrintableItem {
    override fun prettyPrint(): String {
        return "hello, my name is: ${item.name}"
    }
}

// usage
fun main() {
	// list of items returned from 3rd party lib, that we want to "pretty print"
    val list: List<Item> = listOf(
            Item("Adam"),
            Item("Not Adam"),
            Item("Adam Maybe"),
            Item("Yes"),
    )

    list.forEach {
		// adapting item
        val adapted = ItemAdapter(it)
        println(adapted.prettyPrint())
    }
}
```
Using the `Adapter` allows a clean link between classes that we have no control over (from external libraries) with our code with different interfaces. It is a good practice not to use the 3rd party interfaces in the whole system, if possible, but only at the point where the library meets our code. That provides the exchangeability of libraries, easy version update, and protects your code from the forced changes dictated by API alterations in independent software pieces.

I realize that this is not always possible, and sometimes it is even not worth creating an intermediate interface. However, it would be stupid in a long-lived project that multiple teams are working on to have a problem because some silly utils library changed its API after update.

## Duck Typing
If something quacks like a duck, swims like a duck and flies like a duck, it must be a duck. Contrary to `strong typing` where we know for 100% that the object is a duck because it inherits from the class `Duck`, here we are interested in the available behavior of the object. You can see that in Python or JavaScript. Kotlin is strongly typed language, and does not allow for multi-inheritance, but extension functions do allow you to "append" functionality to a class. So we can have a species-fluid dog:
```kotlin
class York: Dog{
    fun bark(){}
}
// adding `quacking` to York class
fun York.quack(){}

York().bark()
// and now it quacks
York().quack()
```
![York duck](assets/posts/york_duck.jpg){: .center-image }

This York doesn't look happy. And it's definitely not a duck. But if it can quack, and that's all we need, it should be good enough?

Going back to the list example, instead of creating an `Adapter`, it would be enough to add an extension function for `Item`:
```kotlin
fun Item.prettyPrint(): String {
    return "hello, my name is: ${this.name}"
}

// Item now is able to "pretty print" itself without additional Adapter	
list.forEach {
	println(it.prettyPrint())
}
// previous version with Adapter
list.forEach {
    val adapted = ItemAdapter(it)
    println(adapted.prettyPrint())
}
```
`Extension functions` are generally a great feature, and in such a simple example, they'll probably do a better job than the additional `Adapter` class. They can also be added to classes that we have no control over, such as those from libraries.

The problem arises when we start adding more and more of these functions to a specific class, in various places in the code. The IDE is great at suggesting possible methods, you can easily jump to implementation, but during the code review, it can be hard to figure out where the class has a given method from - because it is not in the class implementation. It can also obscure the class interface or implicitly override methods.

```kotlin
interface Item

class FirstItem : Item
class AnotherItem : Item

// extension function for generic interface `Item`
fun Item.prettyPrint(): String = "hello, I'm: $this"
// extension function for concrete class `AnotherItem`
fun AnotherItem.betterPrint(): String = "greetings, I'm: $this"
// extension function overriding `prettyPrint` from the `Item` interface
fun AnotherItem.prettyPrint(): String = "yo, I'm: $this"

fun main() {

    val item1: Item = FirstItem()
    val item2: Item = AnotherItem()
    val item3: AnotherItem = AnotherItem()

    item1.prettyPrint()
    item2.prettyPrint()
    item3.prettyPrint() // which method will be called here?
    item3.betterPrint()
}
```
In this example, I have added 3 extension functions. The case of `prettyPrint` is especially interesting, where the implementation is different for the generic` Item` interface and for the specific `AnotherItem` class. The IDE and the compiler don't see any problem. The second method will be just used with `AnotherItem`. The `extension functions` can be written anywhere in your code, even in very distant places from where the class is declared.

Imagine a situation where you created the `extension function` to a generic interface and used it for a long time without any issues with all the different types implementing this interface. At one point a completely different team needed an `extension function` for a concrete type, so someone wrote a method with the same name and signature at a convenient place in code (for them), overriding your method for the generic interface. Without any `override` keyword :) tests may catch it, but they don't have to. Code Review was probably done by someone on the other team, so until something starts behaving in an unexpected way, you probably won't find out about the entire operation. And then looking for the cause of the error may not be pleasant...

We have a few problems here:
- dependency on `extension functions` for an interface that can be easily overwritten
- spreading `extension functions` all over the system
- code review limited to members of 1 team (this is a topic for a separate post)

Zamknięcie całej "rozszerzonej" funkcjonalności w np. `ItemPrinter` pozwoliłoby uniknąć nieporozumień i ułatwić code review. W Git można łatwo sprawdzić, kto jest autorem lub modyfikował ostatnio tę klasę i również dodać do pull requesta. W przypadku `extension functions` taka opcja też istnieje, ale metody mogą być rozsiane po systemie, co utrudnia znalezienie autorów, a jeśli coś jest trudniejsze niż bezproblemowe, to nikt tego nie będzie robił. 

## Shapes
In post about [Factory Method](https://asvid.github.io/kotlin-factory-method) I used geometrical shapes example, that fits nicely here:

For the record:
```kotlin
interface Shape {
    fun draw()
    fun createManipulator(): ShapeManipulator<out Shape>
}

interface ShapeManipulator<T : Shape> {
    fun drag()
    fun resize(scale: Float)
}

internal class Circle : Shape {
    override fun draw() {}
    override fun createManipulator() = CircleManipulator(this)
}

internal class CircleManipulator<T>(private val shape: T) : ShapeManipulator<Circle> {
    override fun drag() = println("CircleManipulator is manipulating circle $shape")
    override fun resize(scale: Float) = println("CircleManipulator is resizing circle $shape")
}
```
We have a `Shape` interface which is implemented by e.g.` Circle`. Additionally, each shape has its own 'ShapeManipulator', an object that knows how to modify the size, position, etc. of a specific shape.
The `Window` class displays the figures on the screen
```kotlin
class Window() {
    fun drawShape(shape: Shape) {
		// magic
    }
}
```
And now we want to be able to display the text on the screen, next to the geometric shapes. However, the `TextView` class is not a shape, it has a different interface and comes from 3rd party library, for example. It is also such a complex class that there is no chance of rewriting it using the `Shape` interface, or changing the behavior of the `Window` class.
```kotlin
class TextView {
    fun displayText() {}
    fun changeSize() {}
    fun changePosition() {}
}
```
Let's use `Adapter`
```kotlin
class TextViewAdapter(val textView: TextView) : Shape {
    override fun draw() {
        textView.displayText()
    }
	
	// anonymous object instead of separate class
    override fun createManipulator(): ShapeManipulator<out Shape> {
        return object : ShapeManipulator<Shape> {
            override fun drag() {
                textView.changePosition()
            }

            override fun resize(scale: Float) {
                textView.changeSize()
            }

        }
    }
}
```
Which for `Window` behaves like any other figure
```kotlin
fun main() {
    val window = Window()
    val circle = Circle()
    window.drawShape(circle)

    val textView = TextView()
    window.drawShape(textView) // error! wrong interface
    window.drawShape(TextViewAdapter(textView)) // using Adapter
}
```
I have intentionally omitted here return types or display logic of elements. Such an 'Adapter' in a real project would be much more complicated. But I hope the idea here is clear.

# Naming
Here I have mixed feelings about whether adding `Adapter` in the name is necessary. On the one hand, it's clear information that the object only maps one interface to another. On the other hand, this information may not be needed at all, clients are only interested in the interface. Does `ItemAdapter` or `ItemWrapper` say more than `ItemWithPrettyPrint`? Moreover, if we create more adapters for different clients for `Item`, naming them `ItemForClient1Adapter` doesn't look great.

Therefore, I tend to name adapters from the object they adapt (`Adaptee`), and the interface they implement (`Target`).

# Summary
The Adapter or Wrapper Pattern allows you to "translate" one interface into another, expected by the client class. It is especially useful when the adapted object comes from 3rd party library, and you do not want to make your system depending on that interface, creating the so-called `anticorruption layer`. Adaptee interface changes will only affect the `Adapter` and not the rest of the code.

Kotlin allows, through the `extension functions`, to provide` Adapter`-like functionality without having to create an entirely new class. This will make sense when you are not interested in the type of object but its capabilities, which is often referred to as `Duck Typing`. However, `extension functions` can obscure the actual class interface, override one another, and cause chaos in general. By limiting their scope, you can deal with it, but if their number starts growing for a specific class, it may be worth setting up a separate wrapper class to organize them.

## Consequences
- **single responsibility principle** - you do not need to change the class adapted for a specific client, only add an `Adapter` with the required interface. The adapted class can change independently from the clients, and it is the job of the `Adapter` to reconcile these changes with the client interface.
- **anticorruption layer** - separates the interface you have no control over and "translates" it into your own. Changes to the interface coming from a library, won't affect the system
- **available for subclasses** - as in the abstract example, creating an adapter for a generic interface allows the use of any class that implements that interface.
- **be careful with extension functions** - at times it may seem that the `extension function` will provide sufficient functionality, but in large projects with multiple teams working on the same code base, this may have unforeseen consequences. This problem can be partially solved by limiting the scope of `extension functions`.