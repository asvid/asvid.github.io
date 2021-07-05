---
layout: post
title: "Decorator in Kotlin"
date:  "2021-07-03 10:25"
description: "
The `Decorator` pattern is used where creating separate classes which are a combination of all possibilities would result in their explosion. This pattern focuses on creating object layers to transparently and dynamically complement objects with new tasks. The decorator provides an object with the same interface as the decorated object.
"
permalink: "kotlin-decorator-pattern"
comments: true
toc: true
tags:
- kotlin
- Decorator Pattern
- design patterns
- extension methods
- Wrapper

categories:
- Design Patterns
- rss

image: /assets/posts/decorator.jpg

---

# Purpose

The `Decorator` allows you to dynamically add or change the behavior of a specific object of a given class, without affecting other objects of the same class. In some cases, this allows you to significantly reduce the number of classes by moving the shared behavior to the `Decorator`, rather than extending the inheritance structure.

The decorator's job is to "wrap" (hence another name: "Wrapper") the original object and modify or overwrite its behavior. The decorator class has the same interface as the decorated object, so it doesn't matter to the client whether the object has been decorated or not. As a rule, `Decorator` is unlikely to add new public methods, as the client using the original object's interface would not have access to them anyway. Having the same interface as the original object allows decorators to nest and wrap each other freely.

Decorating is done dynamically while the program is running, not at the compile time. The decorated object is usually passed in the Decorator constructor because the 'Decorator' instance makes no sense by itself, without the object it wraps around.

# Implementation
There are few key pieces in the `Decorator` pattern:

{% plantuml %}
@startuml
skinparam linetype ortho
hide members
show Decorator members

class Client
interface Component
class ConcreteComponent implements Component
abstract class Decorator implements Component{
# component: Component
}
class ConcreteDecorator extends Decorator

Client -right- Component : using

Decorator o-- Component

@enduml
{% endplantuml %}

Elements:

- **Client** - a class that uses the `Component` object, knows only generic interface, not concrete classes, so it does not need to know about the existence of a decorator
- **Component** - interface of the object that `Decorator` decorates
- **ConcreteComponent** - a specific implementation of the `Component`, which then is passed in the decorator's constructor
- **Decorator** - an abstract class that implements the `Component` interface and takes a `Component` object in the constructor. Using an abstract class, rather than an interface allows forcing the constructor on inheriting classes without the possibility of creating an instance. If `Decorator` accepts objects with a generic` Component` interface rather than a concrete class, it can decorate entire families of objects.
- **ConcreteDecorator** - a specific decorator implementation

## Abstract
The above diagram can look like this in code:
```kotlin
interface Component {
	// example methods of decorated interface
    fun methodA()
    fun methodB()
}
// a single concrete implementation but there can be many
class ConcreteComponent : Component {
    override fun methodA() {}
    override fun methodB() {}
}
// Decorator `is-a` Component and `has-a` Component
// field `component` is `protected` which makes it available to inheriting classes
abstract class Decorator(protected val component: Component) : Component

// concrete `Decorator` implementation with forced constructor requiring the `Component` instance
class ConcreteDecorator1(component: Component) : Decorator(component) {
    // methods has to be overridden
	// in this case, `Decorator` is calling wrapped instance methods without any changes
	// so it's basically a Proxy
    override fun methodA() = component.methodA()
    override fun methodB() = component.methodB()
}
// another implementation of `Decorator`
class ConcreteDecorator2(component: Component) : Decorator(component) {
    override fun methodA(){
		// in this implementation you can't use methodA()
		// it may be related to checking `Component` parameters for example
        throw Exception("you can't do this")
    }
    override fun methodB(){
        println("running methodB")
        component.methodB()
    }
}

fun main(){
	// "naked" `Component``
    val component: Component = ConcreteComponent()
	// first Decorator wrapping a component
    val dec1: Component = ConcreteDecorator1(component)
	// second Decorator, wrapping already wrapped component
    val dec2: Component = ConcreteDecorator2(dec1)
}
```
You can see here how easy it is to nest Decorators. In this implementation, `ConcreteDecorator1` is basically` Proxy` because it simply calls the methods from the passed `Component` object. Note that the `ConcreteDecorator2.methodA()` throws an exception. The decorator can also return a constant and not use the passed `Component` object at all. The 'decorator' decides for himself how to behave. Based on the properties of the wrapped object, it may decide to raise an exception instead of returning some value.

{% plantuml %}
@startuml
title Nested Decorators
node Decorator1{
node Decorator2{
node Component{

		}
	}
}
@enduml
{% endplantuml %}


## Delegates
Kotlin has built-in support for [the delegation pattern](https://kotlinlang.org/docs/delegation.html), another pattern that sets composition above inheritance. The general idea is to **delegate** the task resulting from the class interface to some component object, passed in the constructor, or injected by DI. This way class can be composed of reusable delegates instead of duplicating the implementation or creating strange inheritance structures. A bit like the template method, only that for the class - a class template :)

Delegates can be used to implement the `Decorator` pattern. In Kotlin, delegates are created using the 'by' keyword:
```kotlin
// basic interface to be decorated
interface Component {
    fun sayHello(): String
}
class ConcreteComponent1 : Component {
    override fun sayHello() = "hello from ${javaClass.simpleName}"
}

// delegating the `Component` interface behavior to the `component` instance passed in constructor
abstract class Decorator(protected val component: Component): Component by component

class Decorator1(component: Component) : Decorator(component) {
	// component method overridden by decorator, extending the message
    override fun sayHello() = "${component.sayHello()} and from ${javaClass.simpleName}"
}
// no need to override `sayHello()` every time
// it was "delegated" to the `component` instance from constructor
class Decorator2(component: Component) : Decorator(component)

fun main() {
    val first: Component = ConcreteComponent1()
    val decOne: Component = Decorator1(first)
    val decTwo: Component = Decorator2(decOne)
    println(first.sayHello())  // hello from ConcreteComponent1
    println(decOne.sayHello()) // hello from ConcreteComponent1 and from Decorator1
    println(decTwo.sayHello()) // hello from ConcreteComponent1 and from Decorator1 and from Decorator2
}
```
Alternatively, `component` may not even be a protected field in the abstract `Decorator`. Then the inheriting specific decorators can use `super`, which will delegate the method call to the `comm` object from the constructor.
```kotlin
abstract class AltDecorator(component: Component): Component by component

class Decorator1(component: Component) : Decorator(component) {
    override fun sayHello() = "${super.sayHello()} and from ${javaClass.simpleName}"
}
```

You can delegate as many interfaces as you want, but **only interfaces** not classes:
```kotlin
interface Component {
    fun sayHello(): String
}
interface Component2{
    fun sayBye(): String
}

abstract class Decorator(
        protected val component: Component,
        protected val component2: Component2
) :
        Component by component,
        Component2 by component2
```
By using delegates, a particular `Decorator` can only contain methods that it actually wants to change. The abstract base class `Decorator` could also contain proxy calls to all methods of the decorated object, in which case concrete decorators would overwrite only the methods they require. But if you can do the same thing with a single word, why overpay? :)

## Communication Interfaces
Let's take a system that sends text messages. Messages can be sent via Bluetooth or TCP. We want to be able to send pure text, JSON and JSON with the Base64 encoded message. Any type of message should be transferable by any means. And from the client level, it shouldn't matter what type of message and medium has been selected.

Ok lets start from communication means:
```kotlin
// generic communication interface
interface Comm {
    fun sendMessage(text: String): Result
}

// class handling Bluetooth communication with the use of passed BtModule
class BtComm(val bt: BtModule) : Comm {
    override fun sendMessage(text: String): Result {
        return bt.send(text)
    }
}
// class handling TCP communication with the use of passed TcpModule
class TcpComm(val tcpModule: TcpModule) : Comm {
    override fun sendMessage(text: String): Result {
        return tcpModule.send(text)
    }
}

class TcpModule {
    fun send(text: String): Result {
        println("sending message via TCP: $text")
        return Result.Success() // sealed class
    }
}
class BtModule {
    fun send(text: String): Result {
        println("sending message via BT: $text")
        return Result.Success()
    }
}
```

Clients will only know the general `Comm` interface, without knowing exactly how the message is processed. Separate modules will handle the details of sending via Bluetooth and TCP. From the point of the communication interface, its a facade for potentially complicated messaging, framing, device pairing, address finding, etc.

`BtComm` and` TcpComm` are specific classes that implement the interface to be decorated, not decorators.

Using it with Bluetooth may look like that:
```kotlin
val btModule = BtModule()
val message = "hello"
val btComm: Comm = BtComm(btModule)
btComm.sendMessage(message) // sending message via BT: hello
```
Which causes sending simple text message via Bluetooth. Same story with TCP.


### JsonDecorator
We'd like to send JSON formatted messages with BT and TCP. A decorator like this can be used:
```kotlin
// `comm` passed in constructor is not a class field, so inheriting classes can't access it`
abstract class CommDecorator(comm: Comm) : Comm by comm

class JsonDecorator(comm: Comm) : CommDecorator(comm) {
    override fun sendMessage(text: String): Result {
		// with `super` method from abstract `CommDecorator` is called, 
		// that then delegates the call to the `comm` instance
        return super.sendMessage("{\"message\":\"$text\"}")
    }
}
```
And usage:
```kotlin
val tcpModule = TcpModule()
val btModule = BtModule()

val message = "hello"
// wrapping TCP communication in JSON decorator
val tcpJsonComm: Comm = JsonDecorator(TcpComm(tcpModule))
tcpJsonComm.sendMessage(message) // sending message via TCP: {"message":"hello"}
// for BT:
val btJsonComm: Comm = JsonDecorator(BtComm(btModule))
btJsonComm.sendMessage(message) // sending message via BT: {"message":"hello"}
```
If there was another way of communication now (Websockets, Firebase, carrier pigeon) and the messages should have the same format, no problem. You just decorate the new `Comm` implementation with an already made decorator. Likewise, if a new message format is created, familiar transfer methods can be used with the decorator.

### Base64Decorator
Last decorator will help creating Base64 encoded message:
```kotlin
class Base46Decorator(comm: Comm) : CommDecorator(comm) {
    override fun sendMessage(text: String): Result {
        return super.sendMessage(prepareMessage(text))
    }

    private fun prepareMessage(text: String): String {
        return Base64.getEncoder().encodeToString(text.toByteArray())
    }
}
```
It has a new private method to prepare messages. It does not extend the original interface, so it makes no difference whether the client knows about `Comm` or `Base46Decorator` - it has access to the same methods, so will rather choose to use `Comm` due to the easily interchangeable implementation. Such a subliminal encouragement of good behavior.

### Decorating order
All together is used like:
```kotlin
val tcpBase64JsonComm: Comm = JsonDecorator( // put message in JSON structure
        Base46Decorator( // encode everything in Base64
                TcpComm( // send using TCP
                        tcpModule
                )
        )
)
tcpBase64JsonComm.sendMessage(message) // sending message via TCP: eyJtZXNzYWdlIjoiaGVsbG8ifQ==

// different decorating order
val tcpJsonBase64Comm: Comm = Base46Decorator( // encode just the message with Base64
        JsonDecorator( // put it in JSON structure
                TcpComm( // send using TCP
                        tcpModule
                )
        )
)
tcpJsonBase64Comm.sendMessage(message) // sending message via TCP: {"message":"aGVsbG8="}
```
As you can see, the order of the decorators is of colossal importance. What exactly happened?
- **tcpBase64JsonComm**
	1. wrap the message in a JSON structure
	2. encode everything with Base64
	3. Send via TCP
- **tcpJsonBase64Comm**
	1. encode just the message with Base64
	2. wrap everything in JSON structure
	3. Send via TCP
The deeper the decorator is, the later it will be called with the result of processing data from earlier decorators.

{% plantuml %}
@startuml

Client -> JsonDecorator : sendMessage("hello")
JsonDecorator -> Base46Decorator : sendMessage({"message": "hello"})
Base46Decorator -> TcpComm : sendMessage("eyJtZXNzYWdlIjoiaGVsbG8ifQ==")
TcpComm -> TcpModule : send(body: "eyJtZXNzYW...)
TcpModule -> TcpComm : Success/Fail
TcpComm -> Base46Decorator : Success/Fail
Base46Decorator -> JsonDecorator : Success/Fail
JsonDecorator -> Client : Success/Fail

@enduml
{% endplantuml %}

As I wrote earlier, one of the decorators could also throw an exception or not use the method from the decorated object at all and return a constant. This can be helpful, for example, when we want to obscure some data depending on where we send it. For example, application logs using the class 
```kotlin
data class LogEntry (val timestamp: Timestamp, val tag: String, val message: String)
```
- if they are stored locally, they should contain all the information
- if they are sent to our website via HTTPS, only sensitive data should be obfuscated
- if for some external monitoring, we send a minimum of the information, maybe even just the TAG and timestamp.

There is no point in extending the implementation of logging or sending data with these functionalities. Decorators on `LogEntry` can be created which, using internal logic, would clean up the message before its sent. Changing the logging policy or adding new aggregators will not cause the necessity to change the interfaces of the classes responsible for collecting or sending logs, but only the replacement of decorators.

### Extension Methods
It seems that instead of adding more classes and playing with delegates, you can add few `extension methods` to the interface and thus prepare the message to be sent as JSON and/or Base64.
```kotlin
fun String.toJsonMessage(): String = "{\"message\":\"$this\"}"
fun String.toBase64(): String = Base64.getEncoder().encodeToString(this.toByteArray())

btComm.sendMessage(message.toBase64().toJsonMessage()) // sending message via BT: {"message":"aGVsbG8="}
```
But now it is the clients who need to know what form of a message to use, rather than using a decorated object with a generic interface that can be injected into them. Additionally, the `extension method` can be implicitly overridden elsewhere in the project, so you can never be sure that calling `toJsonMessage()` will produce the same result. I have already written about this in the context of the [Adapter Pattern](https://asvid.github.io/kotlin-adapter-pattern#duck-typing). In this case, I'm using the `String` class, but if `toJsonMessage()` would return some JSON object, the `toBase64()` method would have to take this into account - when using decorators, the interface is always the same and the decorators don't need to know about each other.

Code using `extension methods` is much simpler than using a series of decorators. However, if extension methods become used in multiple places and with multiple interfaces, maintaining them can be cumbersome, as well as testing. And if they will not be used anywhere else, then maybe the usual methods in the class will suffice instead of `extension methods`. Although I will admit that I sometimes prefer the `instance.extMethod()` syntax over `extMethod(instance)`.

# Naming
I usually don't like suffixes derived from design patterns in class names, but in the case of `Decorator` it is good to have clear information about the purpose of the class. Although the class interface is the same as the object being decorated, an instance of 'Decorator' itself does not make sense, unlike the decorated object.

# Summary
The `Decorator` pattern is used where creating separate classes which are a combination of all possibilities would result in their explosion. This pattern focuses on creating object layers to transparently and dynamically complement objects with new tasks. The decorator provides an object with the same interface as the decorated object.

While it is possible to add new public methods in the decorator, it may encourage clients to cast up or use the interface of a concrete decorator instead of a general component. The main advantage of the Decorator is its transparency to the customer, which is achieved by using the generic interface of the wrapped component.

In the example with communication interfaces, if not for the decorator, you would probably need to extend clients with new functionalities or introduce all variations of message processing and the way of sending it in separate classes.

Kotlin allows you to elegantly create decorators with delegates. Extension methods also kind of decorate the original class with new features, but are not a replacement or alternative to this pattern. Instead, they are an alternative to using the simple Java `Wrappers`, which add new public methods to objects that we don't want or can't extend.

# Consequences

- **changing object behavior without inheritance** - inheritance is appropriate only in cases where the derived class is a subtype of the base class (relation on the principle of generalization-> specialization) - [^effective_java]. You could probably find such a connection between the `String` and the `TCP` packet, but it would be a bit of a stretch and inflexible. Especially adding JSON formatting and Base64 encoding along the way. It would also be difficult to re-use the code for a message over Bluetooth. By arranging layers, the decorator allows you to flexibly change the behavior of an object, without changing what the object is.
- **dynamic changes in the behavior of the object** - the object can be decorated while the program is running because the new behaviors do not change the interface, but only fit into it. The same object can be decorated with one class in one place and with another in different one.
- **SRP** - a large monolithic class with multiple responsibilities, can be transformed into a set of decorators used only where they are needed. Unfortunately, this can also lead to upcasting to the decorator interface or calling an internal decorated object from the client (Demeter law violation).
- **the order in which the decorators are applied is important** - example with sending a message. The order is important, and at the same time the decorators themselves don't know anything about other decorators. The responsibility for creating the correct set of decorators should rest with some, for example, `Factory` or `Builder` - as long as there is an extensive hierarchy.


---
[^effective_java]:["Effective Java"](https://books.google.pl/books/about/Effective_Java.html?id=ka2VUBqHiWkC&redir_esc=y) 