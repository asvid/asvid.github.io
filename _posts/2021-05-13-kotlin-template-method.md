---
layout: post
title: "Kotlin Template Method"
date:  "2021-05-18 11:43"
description: "The template method is a very simple design pattern, that separates shared class parts from changing ones. The core idea is to have an abstract parent class containing the algorithm steps and allowing inheriting classes to overwrite individual steps, but not the algorithm that uses those steps itself."
permalink: "kotlin-template-method"
comments: true
toc: true
tags:
- design patterns
- Kotlin
- Template Method
- behavioral design pattern

categories:
- Design Patterns
- rss

image: /assets/posts/pizza.jpg

---

# Purpose
The template method is a very simple design pattern, that separates the shared class parts from distinctive ones. The core idea is to have an abstract parent class containing the algorithm steps and allowing inheriting classes to overwrite individual steps, but not the algorithm that uses those steps itself.

Think about Pizza - steps to make it are more-less the same, despite the type of pizza. You need to make a dough, apply sauce and ingredients, and finally bake it. Different types of pizza may have a different dough, sauce, or baking time (I guess, I don't know how to make pizza :) ), but the order and steps would be always the same. So we can agree that there is an abstract `Pizza` with a method `make()`, but inheriting classes like `Pepperoni` or `Hawaiian` will override methods applying ingredients, without changing the algorithm of making pizza.

We are having here the inversion of control, because parent class `Pizza` is calling overridden methods from inheriting class, and not the other way around (child class calling abstract parent methods).

Template methods may be seen in libraries when the creator allows extending library classes and overriding some methods, but maintain control over calling those methods.

# Przykłady implementacji
## Podstawowy wzorzec
Let's start with a generic example, picturing what this pattern is about:

```kotlin
abstract class AbstractClass { // parent class

	// you cant override this method because it's not `open`
    fun templateMethod() { // the Template Method, so the algorithm with ordered steps
        println("running template method")
        primitiveOperation1() // calling algorithm steps
        primitiveOperation2()
        primitiveOperation3()
    }

    abstract fun primitiveOperation1() // algorithm step to be overridden by concrete class
    
    private fun primitiveOperation2() { // algorithm step not to be overridden
        println("doing abstract operation 2")
    }

    abstract fun primitiveOperation3() // another step
}

class ConcreteClass : AbstractClass() { // concrete class
    override fun primitiveOperation1() { // step implementation for concrete class
        println("doing concrete operation 1")
    }

    override fun primitiveOperation3() {
        println("doing concrete operation 3")
    }
}

class AnotherConcreteClass : AbstractClass() { // another concrete class
    override fun primitiveOperation1() {
        println("doing concrete operation 1") // not cool duplicated implementation
    }

    override fun primitiveOperation3() {
        println("doing another concrete operation 3")
    }
}

```
We have here a `templateMethod()` in the abstract class, so our algorithm or rather list of steps, where each step is a method call representing a primitive operation. Those operations may have a default implementation already in the abstract class or require providing an implementation in inheriting classes. The term "primitive" is used for operations to suggest they are just steps and not the algorithm, and that their implementation can be replaced in concrete classes.

The Template method can't be overridden by inheriting classes. This would be possible if `templateMethod()` would be `open`, but Kotlin encourages encapsulation so by default methods and classes are `final`. Overriding `templateMethod()` would mean changing the algorithm, and its constant form is the reason for the Template Method pattern to be used. If there is a need to change the algorithm, maybe it's better to use a pattern like `Strategy`.

## Pizza
Now let's look into more graphic example: **Pizza**
```kotlin
abstract class Pizza { // base class for all pizza types

    fun make() { // steps are the same for every pizza
        makeDough()
        applySauce()
        addIngredients()
        bake()
    }

    open fun bake() { 
		// default implementation for each step
		// concrete classes needs to override only the distinctive ones
        println("baking for 20 minutes")
    }

    open fun addIngredients() {
        println("adding cheese")
    }

    open fun applySauce() {
        println("applying tomato sauce")
    }

    open fun makeDough() {
        println("making 30cm dough")
    }
}

class Pepperoni : Pizza() { // concrete type of pizza
    override fun addIngredients() { // overriding ingredients according to recipe
        println("adding salami")
        println("adding onion")
        println("adding cheese")
    }
	// but it's not controlling the process of making a pizza
	
	// all other methods are left with default implementation
}
class BigPepperoni : Pizza() { // same as previous but bigger

    override fun addIngredients() { // alas, duplicated implementation from previous class
        println("adding salami")
        println("adding onion")
        println("adding cheese")
    }

    override fun makeDough() { // the only difference from standard pepperoni
        println("making 50cm dough")
    }
}

```
Class `Pizza` contains shared steps for all types of pizza. Concrete types of pizza override only the distinctive steps, for example, pizza size, ingredients, or sauce. There is a problem though with sharing method implementation between `Pepperoni` and `BigPepperoni` classes - they have the same ingredients but `BigPepperoni` has a bigger dough size. It may seem that `BigPepperoni` should just extend the `Pepperoni` class and override just the `makeDough()` method. Then `Pepperoni` would need to be `open` and it would create another level of inheritance. You can easily imagine an ongoing class explosion where every pizza type has 3 sizes and 2 sauces to choose from. It may look like the reason to use the `Factory` pattern...

### Pizza Lambdas
Instead of overriding methods of base class `Pizza` interface, we can try to make use of passing lambdas in the constructor:
```kotlin
abstract class Pizza( // base class constructor is taking lambdas but provides default implementation
    private val makeDough: () -> Unit = {
        println("making 30cm dough")
    },
    private val applySauce: () -> Unit = {
        println("applying tomato sauce")
    },
    private val addIngredients: () -> Unit = {
        println("adding cheese")
    },
    private val bake: () -> Unit = {
        println("baking for 20 minutes")
    }
) {

    fun make() { // unchanged template method
        makeDough() // calling lambda parameter from constructor that can be replaced
        applySauce()
        addIngredients()
        bake()
    }
}

class Pepperoni : Pizza( // concrete class
    addIngredients = { // passing lambda overriding default implementation
        println("adding salami")
        println("adding onion")
        println("adding cheese")
    }
)

class BigPepperoni : Pizza(
    addIngredients = { // duplicated implementation, again
        println("adding salami")
        println("adding onion")
        println("adding cheese")
    },
    makeDough = { // dough size lambda replaced
        println("making 50cm dough")
    }
)
```
Looks way more Kotlin style :) In the book [^effective_java] Joshua Bloch mentions that after Java incorporated lambdas, the `Template Method` pattern became irrelevant. It is more convenient to inject behavior as lambdas in the base class constructor than create concrete classes overriding selected methods.

There is still a problem with duplicated implementation. It's possible to keep common implementation inside a variable and pass it to all classes that require it.
```kotlin
val pepperoniAddons: () -> Unit = { // functions are first class citizens
    println("adding salami")
    println("adding onion")
    println("adding cheese")
}

class Pepperoni : Pizza(
    addIngredients = pepperoniAddons // passing the lambda
)

class BigPepperoni : Pizza(
    addIngredients = pepperoniAddons, // passing the same lambda
    makeDough = {
        println("making 50cm dough")
    }
)
```
It works. But passing in every parameter the generic lambda `() -> Unit` is not really safe and allows stupid errors to happen, like wrong argument order. It can be partially avoided with using named arguments.

### Pizza interfaces
Assuming primitive operations are not returning anything, and just changing the Pizza instance, it's safer to use concrete interfaces corresponding to algorithm steps.
```kotlin
interface DoughMaker { // algorithm step interface
	operator fun invoke() { // this allows to use interface instance as lambda
		println("making dough") // default implementation, Kotlin allows it inside interfaces
	}
}

interface SauceApplier {
	operator fun invoke() {
		println("applying sauce")
	}
}

interface AddonsApplier {
	operator fun invoke() {
		println("applying addons")
	}
}

interface Baker {
	operator fun invoke() {
		println("baking")
	}
}
```
Having such interfaces, abstract class `Pizza` may look like this:
```kotlin
abstract class Pizza( 
	// distinctive types makes passing arguments easy, no need to remember order
    private val makeDough: DoughMaker = object : DoughMaker { 
        override fun invoke() {
            println("making 30cm dough") // overridden default implementation
        }
    },
    private val applySauce: SauceApplier = object : SauceApplier {
        override fun invoke() {
            println("applying tomato sauce")
        }
    },
    private val addIngredients: AddonsApplier = object : AddonsApplier {
        override fun invoke() {
            println("adding cheese")
        }
    },
    private val bake: Baker = object : Baker {
        override fun invoke() {
            println("baking for 20 minutes")
        }
    }
) {

    fun make() { // the same Template Method
        makeDough() // here we don't run lambda, but the `invoke()` method of the interface
        applySauce()
        addIngredients()
        bake()
    }
}

```
Concrete classes may have the same objects implementing interface injected, resolving problem with duplicated code:
```kotlin
object PepperoniAddonsApplier : Pizza.AddonsApplier { // common object for all Pepperoni pizza sizes
    override fun invoke() {
        println("adding salami")
        println("adding onion")
        println("adding cheese")
    }
}

object BigPizzaDoughMaker : Pizza.DoughMaker { // sizing object can be used with all Pizza types
    override fun invoke() {
        println("making 50cm dough")
    }
}

class Pepperoni : Pizza(
    addIngredients = PepperoniAddonsApplier
)

class BigPepperoni : Pizza(
    addIngredients = PepperoniAddonsApplier,
    makeDough = BigPizzaDoughMaker
)
```
This allows you to create totally custom Pizza, without creating a concrete class:
```kotlin
val customPizza = object : Pizza(
	applySauce = object : SauceApplier {
		override fun invoke() {
			println("adding super sauce")
		}
	},
	makeDough = object : DoughMaker {
		override fun invoke() {
			println("making super dough 48cm")
		}
	},
	addIngredients = object : AddonsApplier {
		override fun invoke() {
			println("no addons, its super by itself")
		}
	},
	bake = object : Baker {
		override fun invoke() {
			println("I like it raw")
		}
	}
) {}
customPizza.make()
```
> Is it still a `Template Method` or rather a `Strategy`?

## Active Record
This example is taken from the world of Ruby (that I'm not familiar with), where it came from a pattern proposed by Martin Folwer in book [^fowler]. In general, you have a class - a model in MVC understanding, that contains data and also methods to manipulate and save this object in database. More about [^active_record].

In `Ruby on Rails` that kind of model also has a lot of callbacks [^ruby_active_record] or so-called hooks, that are called before saving to DB or in case of an error. This is exactly where `Template Method` fits perfectly. In Kotlin, it can be implemented like that:
```kotlin
abstract class ActiveRecord { // base ActiveRecord class
	
	// template method calling hooks in right order
    fun save() { // the generic save to DB method
        println("saving record $this")
        this.beforeSave() // calling hook
        val isSuccess = DB.save(this) // saving to DB may be success or fail
        if (isSuccess) {
            this.afterSave() // calling hook on success
        } else {
            this.failedSave() // and another one for error
        }
    }

    open fun beforeSave() { // hook with default implementation, open to be overridden
        // NOOP
        println("NOOP beforeSave()")
    }

    open fun afterSave() {
        // NOOP
        println("NOOP afterSave()")
    }

    open fun failedSave() {
        // NOOP
        println("NOOP failedSave()")
    }
}

object DB { // some stub of DB
    fun save(record: ActiveRecord): Boolean {
        println("DB is saving record: $record")
        // here would be saving entity in DB and returning if it was successful or not
        return true
    }
}

class User(private var username: String) : ActiveRecord() { // concrete model extending ActiveRecord
	
    override fun beforeSave() { // hook is overridden
        println("$this beforeSave()")
        sanitizeRecord() // calling fixing the data before its saved in DB
    }

    private fun sanitizeRecord() { // fixing model data
        println("$this sanitizeRecord()")
        username.trim()
        username = username.filter { it.isLetter() }
    }
}

class Post(val text: String) : ActiveRecord() // another model, without hooks
```
Having `ActiveRecord` constructed with hooks in mind, you can easily plugin for example logging, error handling, or fixing data before it's saved in DB. A very similar idea is used in `JUnit` tests, where you have methods annotated with `@BeforeEach` or `@AfterAll`, to run some actions before each test, or clean up after all of them are done.

`ActiveRecord` itself is sometimes called as [^active_record_antipatern] and there are good reasons for it: SRP violation (single class to keep data, DB operations, validate model, etc.), direct mapping DB structure to model fields, troubles with testing. But as I mentioned - Ruby is not my world, and this post is not about `ActiveRecord`, just `Template Method` fits nicely for this case.

# Summary
The `Template Method` pattern is simple in design, and I think I was using it without even knowing that it has a name. The pattern creates a template of the algorithm, and at the same time requires (or just allows to) override its steps in concrete class (see Pizza examples).

Despite its simple construction and a bit archaic approach it still has its place in modern projects, especially libraries. It has some shortcomings, that can be overcome with using lambdas or injecting whole objects, but then it becomes more a `Strategy` than `Template Method`. It's not necessarily bad, but it's something you should be aware of. This might not be the best pattern to use in every case when you have a family of classes with some shared behavior. But for implementing hooks it seems perfect.

## Pros
- fairly simple implementation for a class family with only some distinctive behaviors but shared core
- interesting use-case with `ActiveRecord`, well suited for hooks
- maintaining control over method call order, even when objects are being extended, useful for library creators

## Cons
- potential class explosion
- inheriting class needs to know a bit about the parent to know which methods to override, where to call `super()` etc.
- inheritance instead of composition, change to injecting lambdas in the constructor is creating a `Strategy`-like construction
---
[^effective_java]:["Effective Java"](https://books.google.pl/books/about/Effective_Java.html?id=ka2VUBqHiWkC&redir_esc=y) 
[^fowler]:["Patterns of Enterprise Application Architecture"](https://books.google.pl/books?id=FyWZt5DdvFkC&q=active+record&pg=PT187&redir_esc=y)
[^active_record]:[Active Record](https://en.wikipedia.org/wiki/Active_record_pattern)
[^ruby_active_record]:[Active Record in Ruby](https://guides.rubyonrails.org/active_record_callbacks.html)
[^active_record_antipatern]:[antipattern](https://www.mehdi-khalili.com/orm-anti-patterns-part-1-active-record)