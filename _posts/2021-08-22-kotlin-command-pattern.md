---
layout: post 
title: "Command Pattern in Kotlin"
date:  "2021-08-22 10:53"
description: "
The Command pattern wraps the request into a specific object that has all the information necessary to perform its task. You can think of it as the next stage of refactoring, where at first we extract the code to a separate method, and then to a separate object, taking the arguments needed to execute the request in the constructor.
"
permalink: "kotlin-command-pattern"
comments: true 
toc: true 
tags:
- kotlin
- Command Pattern
- design patterns

categories:
- Design Patterns
- rss

image: /assets/posts/command.jpg

---
# Purpose
The Command pattern wraps the request into a specific object that has all the information necessary to perform its task. You can think of it as the next stage of refactoring, where at first we extract the code to a separate method, and then to a separate object, taking the arguments needed to execute the request in the constructor.

Since the request is an object, it can be sent to a separate object (`CommandProcessor`) for execution, which allows for their queuing and facilitates logging events. The same command can be used in different places in the system, encapsulating the entire logic of the request execution, which prevents code duplication. It does not have to be the same instance, but the class.

The request object, in addition to the standard `execute()` method, may contain a method like `undo()`, i.e. undoing the changes made by the request. Virtually all graphics programs or word processors have this option. They could be storing each change in the form of `Command` in some buffer with limited capacity (hence the possibility of e.g. only 3 undos), and if the user wants to undo the last change made, the`Command` is pulled from the buffer and the changes made by it are undone.

# Implementation

## Abstract
As already mentioned, by default the `Command` object has an `execute()` (or equivalent) method. There are several other classes in this pattern:
- **Receiver** - object used by the `Command` to complete its task. If the `Command` is to download the weather from the internet,` Receiver` will be an HTTP client, for example. It can be any class in your system.
- **Invoker** - the class using the `Command`
- **Client** - creates the `ConcreteCommand` instance and binds it with the `Receiver` and sets it in the`Invoker`
- **ConcreteCommand** - specific command. It has all the other objects needed to complete the task.

{% plantuml %}
@startuml
hide Client members
  skinparam linetype ortho

interface Command{
	+ execute()
}

class ConcreteCommand implements Command{
	- receiver: Receiver
	- params
+ execute()
}

class Receiver{
	+ action()
}

class Client
class Invoker{
- command: Command
+ setCommand(command)
+ executeCommand()
}

Invoker *-->Command
Client --> Receiver
ConcreteCommand::execute --> Receiver::action
note "creates command instance\npassing the Receiver\nand command parameters" as N1
Client .. N1
N1 ..> ConcreteCommand

note "sets command instance" as N2
Client .. N2
N2 ..> Invoker

note right of Invoker::executeCommand
	calls command
	when its needed
end note

note right of ConcreteCommand::execute
	calls Receiver action
end note
@enduml
{% endplantuml %}

The order of interactions looks like this:

{% plantuml %}
@startuml

participant aReceiver
aClient --> aReceiver : posiada
aClient --> aCommand: Command(aReceiver)
activate aClient
aClient -> aInvoker : setCommand(aCommand)
deactivate aClient
aInvoker -> aCommand : execute()
aCommand -> aReceiver : action()

@enduml
{% endplantuml %}

1. The `Client` creates the instance of the specific `Command`, passing the receiver `Receiver`.
2. `Invoker` gets a specific instance of the command.
3. The `Invoker` uses the `execute()` method of the command instance to do its own thing. For example, this could be to perform an action "on click" if `Invoker` is a UI button.
4. The `Command` calls the appropriate methods of its `Receiver`.

In such generic form it can be implemented like this:

```kotlin
// generic Command interface
// binding the Receiver class in the generic interface allows grouping commands in a way
// e.g. related to text editing, graphic editor or HTTP requests
// it's not required by the pattern itself
abstract class Command(val receiver: Receiver) {
    abstract fun execute()
}
// concrete Command taking Refceiver in the constructor
class ConcreteCommand(receiver: Receiver) : Command(receiver) {
    override fun execute() {
        println("executing ConcreteCommand")
        // calling action on Recveiver to perform a task
        receiver.action()
    }
}
// the "true" action performer, knowing implementation details
// e.g. text editor or HTTP client
class Receiver {
    fun action() {
        println("performing action in Receiver")
    }
}
// class using the passed Command
// e.g. UI button calling Command when clicked
class Invoker {
    private lateinit var aCommand: Command

    fun setCommand(command: Command) {
        aCommand = command
    }

    fun performAction() {
        aCommand.execute()
    }
}
// class connecting Receiver, Command and Invoker
class Client(invoker: Invoker) {
    private val receiver = Receiver()

    init {
		// setting the Invoker command
        val concreteCommand = ConcreteCommand(receiver)
        invoker.setCommand(concreteCommand)
    }
}

fun main() {
    val invoker = Invoker()
	
    Client(invoker)
    invoker.performAction()
}
```

### Return Result
The command may return some value. However, this will not always be a good practice. While returning `Success/Failure` will usually be OK to inform the calling class about the command execution status, returning some specific data will conflict with the `CQRS` - command query responsibility segregation approach. The point is that the command either changes something or returns data.

Let's take this situation: you display a list of names, each list item can be edited and the name is changed. A name change is encapsulated in a command that operates on some data repository. Should such a command return anything, and if so, what exactly? The whole list of names with the updated name? What if list is paged, do you return just the page with the updated item, or whole list of items up to this item? Or should the command return the changed name itself, which then `Invoker` has to put into the displayed list, but then you have 2 representations of the names list: one in the repository and second one in the view, even if values are the same (but there is nothing to guarantee this). It's better if the command returns simple `Success`, which will cause the view to download the current list of names from the repository or show errors message. Or better yet, if the view always keeps its list up-to-date with repository by some sort of `data-binding` or `Observer`. After executing the command, the list will update itself, and you don't even need to return any `Success/Failure` from the command, unless you need to handle the error.

Another way may be to pass some callback in the `execute()`, but maybe let's not go that way :) 

```kotlin
// command result nicely fits with `sealed class`
sealed class CommandResult {
	// returned values could be `object` if they don't contain any data
    class Success : CommandResult()
    class Fail : CommandResult()
}
// `interface` instead of `abstract class` this time, no forced `Receiver` type in the constructor
interface Command {
    fun execute(): CommandResult
}
// `Receiver` appears just here
class ConcreteCommand(private val receiver: Receiver) : Command {
	// executing the command returns a result
    override fun execute(): CommandResult {
        println("executing ConcreteCommand")
		// that depends on Receiver response
        return if (receiver.action()) {
            CommandResult.Success()
        } else {
            CommandResult.Fail()
        }
    }
}
// `Receiver` that returns random Boolean
class Receiver {
    fun action(): Boolean {
        println("performing action in Receiver")
        return Random.nextBoolean()
    }
}

fun main() {
    val receiver = Receiver()
    // no `Invoker` in this example, invoking is simply in `main()`
	val result = ConcreteCommand(receiver).execute()
    println("Command result is: $result")
}
```

### CommandProcessor
Closing the entire command in an object allows it to be passed to the external `Processor` instead of being executed immediately. The `Processor` can queue and execute commands according to its internal logic, but from the `Invoker` point of view, it doesn't matter. The `Receiver` can be moved from a command to the `Processor`, thus the commands will only contain parameters, and the rest will be handled by `Processor` when callin the`execute()`. However, make sure that the `execute()` method always has the same signature, and you do not have to implement special handling inside the `Processor` due to the type of the command. As long as `Command` and `Processor` are in the same domain (e.g. HTTP requests to a specific API, sending messages via Bluetooth), there should be no problems with that. If a given `Processor` needs to pass different `Receiver` type when executing commands, it may suggest that they should go to different `Processors`.

```kotlin
// util to generate random delays in suspended methods
fun randomDelay() = Random.nextLong(1000, 3000)

interface Command {
	// Receiver is passed at the execute() call, not sooner
    suspend fun execute(receiver: Receiver)
}
// commands take only parameters, without Receiver
class FirstCommand(private val param: Int) : Command {
    override suspend fun execute(receiver: Receiver) {
        println("executing FirstCommand $param")
        receiver.action()
    }
}

class SecondCommand(private val param: String) : Command {
    override suspend fun execute(receiver: Receiver) {
        println("executing SecondCommand $param")
        receiver.action()
    }
}

class Receiver {
	// running Receivers action may take a while, thus `suspend` and `delay()` to mimic that
    suspend fun action() {
        println("performing action in Receiver")
        delay(randomDelay())
        println("action finished!")
    }
}
// the most important class in this example
// it has the Receiver instance used by the commands
// commands are stored in FIFO queue in the form of `Channel`
object CommandProcessor {
    private val commands = Channel<Command>()
	// using separate Scopes for adding and executing commands solves blocking one by another
    private val processScope = CoroutineScope(Executors.newSingleThreadExecutor().asCoroutineDispatcher())
    private val executeScope = CoroutineScope(Executors.newSingleThreadExecutor().asCoroutineDispatcher())
    private val receiver = Receiver()

    fun process(command: Command) {
        processScope.launch {
            println("adding $command to the queue")
            commands.send(command)
        }
    }

    init {
		// waiting for new commands in the queue and executing them as soon as they come
        executeScope.launch {
            for (command in commands) {
                command.execute(receiver)
            }
        }
    }
}
// Invoker without changes
class Invoker {
    private lateinit var aCommand: Command

    fun setCommand(command: Command) {
        aCommand = command
    }

    fun performAction() {
        CommandProcessor.process(aCommand)
    }
}

fun main() {
    val firstInvoker = Invoker()
    val secondInvoker = Invoker()

	// no Receiver here, just command parameters
    firstInvoker.setCommand(FirstCommand(1))
    secondInvoker.setCommand(SecondCommand("2"))

	// invoking actions 10x
    repeat(10) {
        firstInvoker.performAction()
        secondInvoker.performAction()
    }
}
```
The `CommandProcessor` is a globally available object that processes commands sent to it. Adding new commands does not block their execution because it runs in separate Scope. In this case, commands that return nothing will work best. You just throw them on the queue, and the command effect (if needed) should come through a different channel, as in a proper `CQRS`. Queue and thread management is on the `Processor` side, the `Invoker` doesn't have to deal with it, or even know if there are `coroutines`, Java threads or some `RX`. Commands are executed sequentially, one after the other. Combined with `delay()` for the Receiver action, this has the effect of immediately adding commands to the queue, and laboriously executing them.

Queue commands execution can be done in parallel with multiple `launchProcessors` taking `Channel` as parameter: 
```kotlin
object CommandProcessor {
	// no changes here
    private val commands = Channel<Command>()
    private val processScope = CoroutineScope(Executors.newSingleThreadExecutor().asCoroutineDispatcher())
    private val executeScope = CoroutineScope(Executors.newSingleThreadExecutor().asCoroutineDispatcher())
    private val receiver = Receiver()

    fun process(command: Command) {
        processScope.launch {
            delay(randomDelay())
            println("adding $command to the queue")
            commands.send(command)
        }
    }

    init {
        executeScope.launch {
			// starting 5 internal command processors, working on the same queue
			// so there can be 5 commands executed at the same time
            repeat(5) {
                launchProcessor(commands)
            }
        }
    }
	// private `extension function` that alows multiple consumers take commands from the same queue
	// each command is executed only once
	// after execution is done, the processor is taking next item from the queue
    private fun CoroutineScope.launchProcessor(channel: ReceiveChannel<Command>) = launch {
        for (command in channel) {
            command.execute(receiver)
        }
    }
}
```

### Undo
Commands may have a method that allows you to undo the changes they have made, i.e. `undo()` known from graphic or text editors. The implementation will strongly depend on the case, but it can be assumed that the command remembers the state before the change and restores it, if necessary. Holding multiple states in history eats memory and therefore the history buffer is often limited to the last 3 commands (or similar). Undo execution of a command can also be achieved by executing a command with the opposite parameters, then there is no need to hold the state, and the execution of `undo()` itself can be saved in the history. Undoed commands may end up in a separate buffer, allowing them to be executed again with `redo()`.

Strongly simplified example of `undo()` with keeping the before-execution state:
```kotlin
// generic command of drawing "something" on the `Canvas`
abstract class DrawCommand(private val canvas: Canvas) {
	// state before executing command - the list of already drawn elements
    private var preCommandState = listOf<Shape>()

    abstract fun execute()
    // saving the state
    fun saveState() {
        preCommandState = canvas.shapes.toList()
    }

	// undoing the command, so setting Canvas state from before exectuion
    fun undo() {
        println("undo ${this}")
        canvas.shapes = preCommandState.toMutableList()
    }
}
// drawable shapes interfaces
interface Shape
data class Line(val length: Int) : Shape
data class Circle(val diameter: Int) : Shape
// commands for drawing shapes, taking params and Receiver (Canvas)
class DrawLine(private val length: Int, private val canvas: Canvas) : DrawCommand(canvas) {
    override fun execute() {
        saveState()
        canvas.draw(Line(length))
    }
}
class DrawCircle(private val diameter: Int, private val canvas: Canvas) : DrawCommand(canvas) {
    override fun execute() {
        saveState()
        canvas.draw(Circle(diameter))
    }
}

// Receiver
class Canvas {
	// current canvas state
    var shapes: MutableList<Shape> = mutableListOf()
	// drawing a shape on Canvas means adding it to the list of shapes
    fun draw(shape: Shape) {
        println("drawing a $shape")
        shapes.add(shape)
    }
}

fun main() {
    val canvas = Canvas()
    val commandsHistory = mutableListOf<DrawCommand>()

	// after executing the command its being preserved in history
    val drawLine = DrawLine(2, canvas)
    drawLine.execute()
    commandsHistory.add(drawLine)

    val drawCircle = DrawCircle(1, canvas)
    drawCircle.execute()
    commandsHistory.add(drawCircle)

    val drawLongLine = DrawLine(10, canvas)
    drawLongLine.execute()
    commandsHistory.add(drawLongLine)

    val drawBigCircle = DrawCircle(12, canvas)
    drawBigCircle.execute()
    commandsHistory.add(drawBigCircle)

    println("current shapes: ${canvas.shapes}")
    println("--- undo last 2 ---")
	// reverting last 2 commands
    commandsHistory.removeLast().undo()
    commandsHistory.removeLast().undo()
    println("current shapes: ${canvas.shapes}")
}
```

### Why not just use lambdas?
It may seem that creating a whole class and then an object to perform some action is a redundancy, and that the same functionality and readability can be achieved using lambdas.

Example using a similar processor from the previous case:
```kotlin
fun main() {
	// command uses Receiver when its executed
    val command = { receiver: Receiver ->
        println("this is a command to do stuff")
        receiver.action()
    }
    repeat(10) {
        CommandProcessor.process(command)
		
		// lambda-commands don't need to be assigned to variables
        CommandProcessor.process { receiver: Receiver ->
            println("this is a another command")
            receiver.action()
        }
    }
}
```
We say to the `CommandProcessor`: execute this block using your `Receiver` instance. However, we lose the ability to parameterize command objects. Lambda can take parameters, but they will be used at runtime, which is in the `CommandProcessor`. Of course, they can be passed in the `process ()` method, but then it is difficult to talk about command encapsulation if you have a block of code in one place and parameters need to be passed in another.
If the `action()` method were `suspend`, it would need to be handled in the lambda block, wrapping the call in some `CoroutineScope`. It could come from `CommandProcessor` just like the `Receiver` but this is another complication that occurs when Lambda is created.
To sum up: if you want to have a parameterized Lambda, passed to the processor and use it in multiple places of the system - **create a class**.

## Home Automation example
It may be my professional bias, but controlling remote devices is perfect for a real-life example of using the `Command` pattern.

Lets have devices that can be turned ON or OFF and a remote control to control these devices. The remote control is not directly connected to any specific device, it can control any of them, or multiple at the same time. The remote is also not aware of the device it's controlling, it just handles pressing its buttons.

```kotlin
// Invoker, takes commands in the constructor, but it could also use setters
class RemoteController(
    private val firstButtonAction: Command,
    private val secondButtonAction: Command,
) {
    fun firstButton() {
        firstButtonAction.execute()
    }

    fun secondButton() {
        secondButtonAction.execute()
    }
}

interface Command {
    fun execute()
}

// Receiver
class Device(private val name: String) {
    // action
    fun switch(on: Boolean) {
        println("turning device $name ${if (on) "ON" else "OFF"}")
    }
}
// concrete command thaking Receiver in the constructor
class TurnOnCommand(private val device: Device) : Command {
    override fun execute() {
		// and calling a method accorting to its task
        device.switch(true)
    }
}

class TurnOffCommand(private val device: Device) : Command {
    override fun execute() {
        device.switch(false)
    }
}

fun main() {

    // Receiver instance
    val lightBulb = Device("living room light")

    // passed to the commands
    val turnOn: Command = TurnOnCommand(lightBulb)
    val turnOff: Command = TurnOffCommand(lightBulb)

    // Invoker (the remote) gets commands for its buttons
    val remote = RemoteController(turnOn, turnOff)

	// but Invoker itself is just executing the commands, has no idea which device its controlling and how
    remote.firstButton()

    remote.secondButton()
}
```

# Naming
Adding the `Command` suffix to the names of specific commands seems to make sense. It clearly defines what the class is used for. In the case of the `Receiver` or `Invoker`, which by nature already have their specific tasks and just found themselves in this pattern kinda "by the way" would only be confusing. You can see that in the last example of *Home Automation*.

# Summary
The `Command` is one of my favorite patterns, most of the time I have used it with some form of `CommandProcessor`. It perfectly encapsulates the request and allows it to be moved and reused. It facilitates refactoring because it is easy to replace one command with another, or to change the internal implementation without affecting the clients of it.

Command objects can contain a method to undo the changes they made. This is done by keeping the state of the `Receiver` before executing the command, or by executing the command with opposite parameters. Undoed commands can be put on a separate buffer, which allows them to be redone if necessary.

## Pros
- **encapsulation** - whole logic and method calls required to perform the task are inside a single object with generic and simple interface. It allows to queue execution, reuse code, simple testing and refactorization.
- **dynamic behavior change** - passing command objects enables changing behavior of the `Invoker` in runtime, i.e. when application config is updated remotely
- **multiple calls in one** - instead of calling multiple methods of a few `Receivers`, a single `Command` can be created that will do all that
- **undo/redo** - this pattern naturally allows to undo and redo set of instructions
- **simple extending** - adding new `Command` doesn't influence previous ones, or the calling `Invoker`

## Cons
- **many similar classes** - depending on situation, using the `Command` pattern may cause having multiple classes with single line difference
- **premature complication** - using this pattern too early may end up with single `Command` class used in one oplace, but surrounded with additional interfaces, `CommandProcessor`, etc.