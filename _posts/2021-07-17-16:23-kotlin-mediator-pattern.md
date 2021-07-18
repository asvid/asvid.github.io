---
layout: post
title: "Mediator in Kotlin"
date:  "2021-07-17 16:23"
description: "
The Mediator's job is to organize communication between close classes. The `Mediator` pattern cuts out dependencies between components. It takes over the interaction between them, becoming the main communication hub for a group of classes. There is a reverse of the controls because components are now just telling 'what happened' instead of telling others to 'do something'. It can be found e.g. in the form of `ViewModel` in Android, where it separates UI interactions from data model changes.
"
permalink: "kotlin_mediator_pattern"
comments: true
toc: true

tags:
- kotlin
- Mediator Pattern
- design patterns

categories:
- Design Patterns
- rss

image: /assets/posts/mediator.jpg

---

# Purpose
We define a `Mediator` as an object encapsulating interactions between other objects (`components`) from a given set. The pattern limits, or even cuts off completely, direct dependencies between classes. Components can only communicate with each other through the `Mediator`, which becomes the central hub for information and control. The `Mediator` controls the flow of information using its internal logic.

This can be compared to the air traffic control tower (ATC). Pilots of all planes from the area communicate with it, and it is the tower that decides the order of take-offs and landings. However, the ATC does not control the entire flight in all aspects, but only organizes the planes in relation to each other. Its easy to imagine the chaos, when the pilots of all machines had to communicate directly with each other to determine who was to land next.

For this reason, the `Mediator` pattern seems to me **to fit rather for refactoring** close-by classes, the number of interactions of which has already begun to disturb their maintenance, but have well-established responsibilities. In some cases, the use of `Mediator` will make sense from the very beginning of code development, but it can also be an overkill.

A fairly common use of this pattern is [View Model](https://developer.android.com/topic/libraries/architecture/viewmodel) known from `Android Architecture Components`. It intercepts interactions with the UI and, according to its logic, informs the data model and the other way around - it passes changes in the model to the UI.

# Implementation
The mediator has a very simple structure, there are 2 types of objects:
- **Mediator** - the main communication hub, known to all `components`
- **Component** - doesn't know other components, doesn't depend on them (though it may depend on other classes). It communicates only with the `Mediator` who is calling appropriate`components`

{% plantuml %}
@startuml

interface Mediator{
	+ notify(sender, event, context)
}
class ConcreteMediator implements Mediator{
	- componentA: ComponentA
	- componentB: ComponentB
	- componentC: ComponentC
	+ notify(sender, event, context)
}

class ComponentA{
	- mediator: Mediator
}
class ComponentB{
	- mediator: Mediator
}
class ComponentC{
	- mediator: Mediator
}

ComponentA --* ConcreteMediator::componentA
ComponentB --* ConcreteMediator::componentB
ComponentC --* ConcreteMediator::componentC

ComponentA::mediator-left->ConcreteMediator::notify
ComponentB::mediator-right->ConcreteMediator::notify
ComponentC::mediator-up->ConcreteMediator::notify

@enduml
{% endplantuml %}

It is worth noting that the only interface needed is `Mediator`, which all the` components` know. Each `component` is a separate and independent object that probably already inherits from another class or implements the needed interfaces. You may be tempted to introduce an additional `Component` interface, but it would not do much, because the `Mediator` needs to know the specific interfaces of its `components` to be able to call specific methods after receiving the message.

The `mediator` object can get components in the constructor, via DI, or even create the necessary instances and manage their lifecycle.

## Abstract implementation
Let's see how it may look in code:
```kotlin
interface Mediator {
    // mediator declares methods for communication
    fun method(sender: Any, args: Any? = null)
}

// components don't have to share any interface, but they all have to have reference to the Mediator
// they shouldn't have dependencies to eachother, it kills the point of having a Mediator in between
class ComponentA(private val mediator: Mediator) {
    fun operationA() {
        mediator.method(this, "arg A")
    }
}

class ComponentB(private val mediator: Mediator) {
    fun operationB() {
        mediator.method(this, 10.34)
    }
}

class ComponentC(private val mediator: Mediator) {
    fun operationC() {
        mediator.method(this, println("print me!"))
    }
}
// Mediator encapsulates relationships between components
// it has references to all components it manages
// and sometimes it can even manage component lifecycle
class Concretemediator : Mediator { 
    // components may be injected or passed in constructor
    // but Mediator reference must be passed to each component
    val componentA = ComponentA(this)
    val componentB = ComponentB(this)
    val componentC = ComponentC(this)

    override fun method(sender: Any, args: Any?) {
        // checking which object is sender
        when (sender) {
            // Mediator knows how to handle the incoming message
            is ComponentA -> println("arg from A: $args")
            is ComponentB -> println("arg from B: ${args as Float * 3}")
            is ComponentC -> (args as () -> Any).invoke()
        }
    }
}
```
Instead of the `notify(sender, event, context)` method, I used `method(sender, args)` here, but the principle is the same, the `Mediator` interface determines the form of communication. The `event` in the former case can be an object inheriting from some generic interface, or a `sealed class`, which will create a nice closed set of events that is easier to navigate through.

`Context` can indicate where the message is coming from, e.g. whether the product was added to the shopping cart from the list of recommended additional products or directly from the product page. The arguments of the method, or even the `Mediator` methods, will depend heavily on the specific implementation, and the information needed.

## Dialog
A common example of using `Mediator` is dialog boxes that contain UI elements such as buttons and text inputs. Oftentimes, the state of user interface elements depends on one another. The "OK" button should be inactive, if not all required form fields have been filled, etc. If there are many view elements, the number of such dependencies between them may increase and be difficult to control.

Therefore, it is a good idea to have a central communication hub that will collect events from the UI elements and adjust the state of the displayed elements accordingly. The `Dialog` itself can be such a `Mediator`.

```kotlin
// view manager, controlling UI elements inside it
interface UiDirector {
    // `event` is naive String for simplifying the example
    // `sender` has to be UI element extending `UiElement` class
    fun notify(sender: UiElement, event: String)
}

// `UiElement` class is an UI component inside `UiDirector`
abstract class UiElement(val uiDirector: UiDirector)

// button is UI element
class Button(uiDirector: UiDirector) : UiElement(uiDirector) {
    fun click() {
        // that informs `uiDirector` about being clicked
        // but has no other logic regarding this
        uiDirector.notify(this, "click")
    }
}

class TextBox(uiDirector: UiDirector) : UiElement(uiDirector) {
    // TextBox is also an UI element
    // lets assume it has inner logic refreshing value of the `text` field
    // that takes care of pasting the text or manual typing it
    // using `observable` makes sure that after each change the `uiDirector` gets notified
    // there is no need to remember about calling `notify()` in every place that changes `text`
    val text: String by Delegates.observable("") { property, oldValue, newValue ->
        uiDirector.notify(this, "text_changed")
    }
}

// Dialog window, handles view state according to UI components events
class FancyDialog : UiDirector {

    private val title = "fancy dialog"
    // dialog creates instances of its `components`
    private val okButton = Button(this)
    private val cancelButton = Button(this)
    private val input = TextBox(this)

    private var inputText = ""

    override fun notify(sender: UiElement, event: String) {
        when (sender) {
            // after each UI component change it gets update
            input -> if (event == "text_change") inputText = input.text
            // and has logic called after buttons are clicked
            okButton -> if (event == "click") submit()
            cancelButton -> if (event == "click") dismiss()
        }
    }
    private fun dismiss() {}
    private fun submit() {}
}
```

This allows reusing the implementation of the UI controls as they don't have any specific logic other than managing their state. The `Delegates.observable` in the `TextBox` class makes it easy to notify the view manager of a change. No need to do this in every place that updates the `text` field.

## Chat
An interesting example of the 'Mediator' pattern is chat. We have participants who send messages publicly or directly to another participant, but it is the `Chat` object that decides who gets which message.

```kotlin
// helper alias
typealias ParticipantName = String

// Mediator interface
interface Chatroom {
    fun registerParticipant(participant: Participant)
    // message recipient is optional, when left empty the message is public
    fun send(message: String, from: ParticipantName, to: ParticipantName? = null)
}
// Participant class has very simple API, limited to sending and receiving the message
// no logic controlling where the message actually goes
class Participant(val name: String, private val chatroom: Chatroom) {

    init {
        // the participant is registering itself in the chatroom
        chatroom.registerParticipant(this)
    }

    // is able to send the message
    fun send(message: String, to: ParticipantName? = null) {
        chatroom.send(message, this.name, to)
    }

    // and receive it
    fun receive(message: String) {
        println("[$name] gets: $message")
    }
}
// `object` is here just to make it easier to use in `main()`
object SuperChat : Chatroom {
    // chatroom modes
    enum class Mode { 
        PUBLIC, // every message is delivered to every participant
        PRIVATE, // only messages with recipient will be delivered
        MIXED // both direct messages and public ones will be delivered
    }

    var mode: Mode = Mode.PRIVATE

    private val participants = mutableListOf<Participant>()

    override fun registerParticipant(participant: Participant) {
        participants.add(participant)
    }

    // handling messages
    override fun send(message: String, from: ParticipantName, to: ParticipantName?) {
        when (mode) {
            // depending on the mode they will be handled differently
            Mode.PUBLIC -> participants.forEach { it.receive("$from says: $message") }
            Mode.PRIVATE -> participants.find { it.name == to }?.receive("$from says: $message")
            Mode.MIXED -> {
                if (to == null) participants.forEach { it.receive("$from says: $message") }
                else participants.find { it.name == to }?.receive("$from says: $message")
            }
        }
    }
}

fun main() {
    val alice = Participant("Alice", SuperChat)
    val bob = Participant("Bob", SuperChat)
    val charlie = Participant("Charlie", SuperChat)

    alice.send("hi all!")
    bob.send("hi Alice", "Alice") // only this message will be delivered in PRIVATE mode
    charlie.send("hi Alice")

    SuperChat.mode = SuperChat.Mode.PUBLIC
}
```
Since chat mode is initially `PRIVATE`, only the message with the recipient will be printed `[Alice] gets: Bob says: hi Alice`. The rest of the messages will go to the `Mediator` but not to the other participants. Changing the message delivery policy will have this effect for `MIXED`:

```
[Alice] gets: Alice says: hi all!
[Bob] gets: Alice says: hi all!
[Charlie] gets: Alice says: hi all!
[Alice] gets: Bob says: hi Alice
[Alice] gets: Charlie says: hi Alice
[Bob] gets: Charlie says: hi Alice
[Charlie] gets: Charlie says: hi Alice
```
> There's a bug here because Alice is getting a message from herself. Her `hi all` had no recipient, so it went to all participants - including herself. Just like Charlie answering `hi Alice` but without a recipient.

Participants do not need to know about each other to send messages to each other, this is handled by the `Mediator` in the form of a` SuperChat` object. Additionally, it has its internal messaging policy, it is not just a proxy between objects. The chat could have a list of muted participants who, e.g. temporarily and as a punishment, cannot send messages. But instead of blocking messages from being sent, you can block messages from being delivered without alerting the participant about the `shadowban.

# Naming
`Mediator` is a very simple pattern in terms of structure, it's basically a single interface. There is no need to add this to the name of objects that act as Mediator. Depending on the implementation, `Mediator` may not even be visible outside, unlike the `Facade`, that by definition, is an entry interface to some functionality.
For this reason, I would definitely avoid using the name Mediator in class names. Even the `notify()` method can and should refer to the domain of the problem being solved, as in the `send()` chat. This pattern should be transparent outside the class group and be only an implementation detail, and organizational improvement.

# Summary
The Mediator's job is to organize communication between close classes. The `Mediator` pattern cuts out dependencies between components. It takes over the interaction between them, becoming the main communication hub for a group of classes. There is a reverse of the controls because components are now just telling "what happened" instead of telling others to "do something". It can be found e.g. in the form of `ViewModel` in Android, where it separates UI interactions from data model changes.

The mediator, using internal logic, decides what to do with the information coming from one of his components. It can call a method on one or more components. Therefore, it needs to know the specific interfaces of the components, not some generics derived from the pattern. Theoretically, a `Component` may also have only the `notify()` method and decide for itself how to react to a given message from the `Mediator` (two-way mediation), but this significantly complicates the matter and starts to be a different pattern.

Be careful, so the 'Mediator' does not turn into a so-called **god object**, expanded and complex, containing much of the system logic. The mediator should be used locally rather than as an access interface to functionality, unlike Facade.

It's well suited for refactoring. If you have a group of related classes that are cumbersome to reuse by the number of dependencies. It also improves testing, because there is no need to create mocks for a group of classes, but only the Mediator. If all interactions between components are contained in the Mediator, using the same components but in a different context will only require writing a new Mediator.

## Pros
- **no dependencies** - The mediator takes over the communication between the components, so they don't need to know about each other.
- **testing** - Mediator mock/stub is enough to test the class, no need to create a whole dependency tree.
- **easy code reuse** - lack of dependencies makes using the class in new places easy. You can even reuse whole functionality in a new context by creating a new Mediator object with no component changes.
- **refactoring** - Mediator allows to untangle the mutual dependencies of a "mature" code, where class responsibilities are already established, but the amount of mutual interactions starts to be a problem.

## Cons
- **god object** - can expand to enormous dimensions, "swallowing" subsequent components and become an access point to functionality, instead of just an organizer for communication.
- **covers the problem instead of solving it** - shifting all communication and dependencies between components to Mediator may cause objects to still depend on each other, but implicitly. Instead of having references to themselves in the code, they will still expect certain Mediator calls to do their job properly.