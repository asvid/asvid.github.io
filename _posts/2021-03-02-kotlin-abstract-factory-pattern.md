---
layout: post
title: "Kotlin Abstract Factory"
date:  "2021-03-08 11:43"
description: "
The factory of factories, 'Abstract Factory' makes creating objects that are part of some 'family' easy. It is another layer over concrete factories delivering client the factory instance for creating objects of a certain variant.
"
permalink: "kotlin-abstract-factory"
comments: true
toc: true
tags:
- design patterns
- Kotlin
- Abstract Factory
- construction design pattern

categories:
- Design Patterns
- rss

image: /assets/posts/abstract_factory.jpg

---

# Purpose

The name of this pattern doesn't suggest directly how it differs from other construction patterns, like `Builder` or `Factory Method`. In the `Abstract Factory` it's not about creating a single object instance, but the whole family of connected objects. It still sounds like an ordinary `Factory` that can produce GUI controls for example. To have the `Abstract Factory` you should add another layer of abstraction and have a mechanism for creating GUI controls in few variants, like for Linux, Windows, or MacOS, but still have a generic API for the pattern client.

`Abstract Factory` is the **factory of factories**, it creates a version of the factory building objects with a common interface. Object families are bound with the factory that builds them. Still being in the GUI area: you can have control families for Windows, Linux, and MacOS - this will prevent situations like using MacOS button next to Windows text field. Factory client is not concerned about which family factory is being used, because the API of delivered objects is the same. And this is the `Abstract` of this Factory - **sharing the object family factory interface**.

Some useful posts before further reading :
- [Static Factory Method](https://asvid.github.io/kotlin-static-factory-methods)
- ["Ordinary" Factory Method](https://asvid.github.io/kotlin-factory-method)

# Implementation examples

I had some trouble myself understanding this pattern before I saw some code examples and diagrams, so let's jump into it now.

## Basic

The book example from "Design Patterns"[^design_patterns] from "the gang of four". It is very `abstract` but it shows nicely the idea behind this pattern.

The interface of the factory producing family of objects. Family consists two products: `ProductA` and `ProductB`.
```kotlin
interface Factory {
    fun createProductA(): ProductA
    fun createProductB(): ProductB

    companion object // won't hurt and can be useful
}
```

Each of the products has two variants: `1` i `2`.
```kotlin
// products
interface ProductA
interface ProductB

// variant 1
class Product1A : ProductA
class Product1B : ProductB

// variant 2
class Product2A : ProductA
class Product2B : ProductB
```

Each variant has a separate specific factory, supplying both products in a suitable variant. The `internal` ensures that the class does not leak outside the module. Concrete factories are a good candidate for using Singleton, hence the `object`. 
```kotlin
// variant 1 factory
internal object ConcreteFactory1 : Factory {
    override fun createProductA() = Product1A()
    override fun createProductB() = Product1B()
}

// variant 2 factory
internal object ConcreteFactory2 : Factory { // 
    override fun createProductA() = Product2A()
    override fun createProductB() = Product2B()
}
```
The `Abstract Factory`, returning the appropriate concrete factory (for the appropriate variant) but being hidden behind
generic `Factory` interface. This way, customers don't even need to know about the specific factory existence. The client
just needs to know that they will get a `Factory` and that it will be able to build` ProductA` and `ProductB`. The `getFactory ()` methods
here is the so-called `top-level function` because it is outside any class, but can be part of e.g.` companion object`
Factory interface. 

However, the Factory interface itself should not necessarily be aware of its implementations, fortunately it can
an empty `companion object`, which will be given in the appropriate place with the` extension function`. 
I wrote more about it [here](https://asvid.github.io/pl/kotlin-builder-pattern).
```kotlin
// naive but working implementation with ordinary Int
fun getFactory(whichOne: Int): Factory {
    return when (whichOne) {
        1 -> ConcreteFactory1
        2 -> ConcreteFactory2
        else -> throw IllegalArgumentException("value: $whichOne is not available")
    }
}

// variants as enum
enum class FactoryVariant {
    Variant1,
    Variant2
}

// less naive but similar implementation with enums
fun getFactory(whichOne: FactoryVariant): Factory {
    return when (whichOne) {
        FactoryVariant.Variant1 -> ConcreteFactory1
        FactoryVariant.Variant2 -> ConcreteFactory2
        else -> throw IllegalArgumentException("value: $whichOne is not available")
    }
}

// extension function for the Factory companion object
fun Factory.Companion.getFactory(whichOne: Int): Factory {
    return when (whichOne) {
        1 -> ConcreteFactory1
        2 -> ConcreteFactory2
        else -> throw IllegalArgumentException("value: $whichOne is not available")
    }
}
```

Usage:

```kotlin
val factory: Factory = getFactory(2)

factory.createProductA() //  Product2A
factory.createProductB() //  Product2B

val factory2: Factory = getFactory(FactoryVariant.Variant1)
factory2.createProductA() // Product1A
factory2.createProductB() //Product1B

// or as Factory interface companion object method
Factory.getFactory(1).createProductA()

```

On diagram it looks like this:

{% plantuml %} 
@startuml 

interface Factory{
+ createProductA() : ProductA
+ createProductB() : ProductB 
  }

interface ProductA 
interface ProductB

class Product1A<< (C,#FF7700) Variant1 >> implements ProductA 
class Product1B<< (C,#FF7700) Variant1 >> implements ProductB

class Product2A<< (C,#FF0077) Variant2 >> implements ProductA 
class Product2B<< (C,#FF0077) Variant2 >> implements ProductB

class ConcreteFactory1<< (O,#FF0077) Variant1 >> implements Factory
class ConcreteFactory2<< (O,#FF7700) Variant2 >> implements Factory

Factory::createProductA-right[bold]-|>ProductA 
Factory::createProductB-right[bold]-|>ProductB

class AbstractFactory{ 
+getFactory(variant) : Factory 
}

AbstractFactory::getFactory-->Factory

@enduml 
{% endplantuml %}

## GUI elements family

A concrete example using the already mentioned GUI controls families. It still seems to me not quite "real" but
personally, it helped me the most to understand what the `Abstract Factory` can actually be used for.

So we have few GUI controls, like:

```kotlin
interface View
abstract class Button : View
abstract class Image : View
abstract class GridView : View
```

And interface of the factory that supplies them.
```kotlin
interface ViewFactory {
    // variants known to the Factory
    enum class OS {
        Linux, Windows
    }

    // methods creating generic controls, independent from the OS
    fun createButton(): Button
    fun createImage(): Image

    companion object {
        // "static" method that supplies concrete factory
        fun createFactory(os: OS): ViewFactory {
            return when (os) {
                OS.Linux -> LinuxViewViewFactory
                OS.Windows -> WindowsViewViewFactory
            }
        }
    }
}
```

`ViewFactory` interface is implemented by concrete factories for Windows and Linux
```kotlin
object LinuxViewViewFactory : ViewFactory {
    // concrete types that are created for specific OS
    // internal ensures they wont leak outside module
    internal class LinuxButton : Button()
    internal class LinuxImage : Image()

    // the clients will see only generic control interface
    override fun createButton(): Button = LinuxButton()
    override fun createImage(): Image = LinuxImage()
}

object WindowsViewViewFactory : ViewFactory {
    internal class WindowsButton : Button()
    internal class WindowsImage : Image()

    override fun createButton(): Button = WindowsButton()
    override fun createImage(): Image = WindowsImage()
}
```

It can be used like this:
```kotlin
val winViewFactory = ViewFactory.createFactory(ViewFactory.OS.Windows)
val button = winViewFactory.createButton()
```

But this way, the abstract factory interface needs to know its implementations - that is, the specific control factories. You can approach it differently and not create a `companion object` in` ViewFactory` that returns factories, but create a new class that will take care of delivering specific controls using each available control factory.
```kotlin
// class hiding concrete factories, but implementing the same interface
class ViewCreator(os: ViewFactory.OS) : ViewFactory {

    // instance of `ViewCreator` can provide controls only for provided in the constructor OS 
    private val factory = when (os) {
        ViewFactory.OS.Linux -> LinuxViewViewFactory
        ViewFactory.OS.Windows -> WindowsViewViewFactory
    }

    // methods returning generic controls from the factory
    // created on the constructor argument base
    override fun createButton(): Button = factory.createButton()
    override fun createImage(): Image = factory.createImage()
}

// usage
val linuxButton = ViewCreator(ViewFactory.OS.Linux).createButton()
```

Clients still don't know the specific control factory, but now the `ViewFactory` interface does not need to know them either. No need for extending `companion object` also. We just create an instance of the factory for the specific OS that is underneath is using specific factories compatible with the provided OS type.

It's easy to see that as the number of controls increases, a lot of code will look almost identical, essentially just proxy method calls to specific factories. An interesting and unorthodox idea is to use one `make <Type> ()` method instead of separate methods for each type of control. Then the `ViewCreator` would look like this:
```kotlin
// not implementing the `ViewFactory` interface anymore
class ViewCreator(os: ViewFactory.OS) {
    // this field must be public because it's used in the `inline` method
    val factory = when (os) {
        ViewFactory.OS.Linux -> LinuxViewViewFactory
        ViewFactory.OS.Windows -> WindowsViewViewFactory
    }

    // `inline` and `reified` allow using unknown generic types before compilation
    inline fun <reified T : View> make(): T = when (T::class) {
        Button::class -> factory.createButton() as T
        Image::class -> factory.createImage() as T
        else -> throw IllegalArgumentException()
    }
}

// usage
val windowsButton = ViewCreator(ViewFactory.OS.Windows).make<GridView>()
```

The keywords `inline` and` reified` are easily a topic for a separate post. Using them here allows the passing of a type extending `View`, and causes the `make()` method to return just such a type. This saves you from having to add a new `createButton()` method for each new control. It is enough for the new control to extend the `View` and handling the new case in the `when`. 

What exactly does the `inline` do? It copies the code to the place where it is called, **literally** :) and that is why all fields used by the `inline` methods must be publicly available. With this implementation of the `make()` method, we need to use `reified` to be able to use the **class of the passed type <T>** to find the correct method in the specific `ViewFactory`. 
The `reified` can only be used in` inline` methods, because "pasting" the method code where it is called allows you to know the exact type during the compilation. After calling the appropriate method, e.g. `createButton()`, the result should be cast to `T`, because `T` is the expected type of the returned object - not `Button`, even if `T :: class == Button: : class`.

Copying code causes compiled files to explode in size, e.g. calling `make()` twice:
```kotlin
val windowsButton = ViewCreator(ViewFactory.OS.Windows).make<Button>()

// sadly you can also pass the `internal` types within the same module
// so the returned type wont be generic `Button` but concrete `WindowsButton`
val windowsButton2 = ViewCreator(ViewFactory.OS.Windows).make<WindowsViewViewFactory.WindowsButton>()
```

This will generate `bytecode` that will be equivalent of Java:
```java
// the beginning of the first Button
ViewCreator this_$iv=new ViewCreator(OS.Windows);
int $i$f$make=false;
KClass var4=Reflection.getOrCreateKotlinClass(Button.class);
Button var10000;
View var9;
Image var10;
if(Intrinsics.areEqual(var4,Reflection.getOrCreateKotlinClass(Button.class))){
var10000=this_$iv.getFactory().createButton();
if(var10000==null){
throw new NullPointerException("null cannot be cast to non-null type abstract_factory.family.Button");
}

var9=(View)var10000;
}else{
if(!Intrinsics.areEqual(var4,Reflection.getOrCreateKotlinClass(Image.class))){
throw(Throwable)(new IllegalArgumentException());
}

var10=this_$iv.getFactory().createImage();
if(var10==null){
throw new NullPointerException("null cannot be cast to non-null type abstract_factory.family.Button");
}

var9=(View)((Button)var10);
}
// the end of the first Button
Button windowsButton=(Button)var9;

// ------------------------------------------------------------------------------------------

// the beginning of the second Button
ViewCreator this_$iv=new ViewCreator(OS.Windows);
int $i$f$make=false;
KClass var5=Reflection.getOrCreateKotlinClass(WindowsButton.class);
if(Intrinsics.areEqual(var5,Reflection.getOrCreateKotlinClass(Button.class))){
var10000=this_$iv.getFactory().createButton();
if(var10000==null){
throw new NullPointerException("null cannot be cast to non-null type abstract_factory.family.WindowsViewViewFactory.WindowsButton");
}

var9=(View)((WindowsButton)var10000);
}else{
if(!Intrinsics.areEqual(var5,Reflection.getOrCreateKotlinClass(Image.class))){
throw(Throwable)(new IllegalArgumentException());
}

var10=this_$iv.getFactory().createImage();
if(var10==null){
throw new NullPointerException("null cannot be cast to non-null type abstract_factory.family.WindowsViewViewFactory.WindowsButton");
}

var9=(View)((WindowsButton)var10);
}
// the end of the second Button
WindowsButton windowsButton2=(WindowsButton)var9;
```

And so, anywhere where the `inline` method is used. So maybe this is not the best solution for frequently used methods, and creating GUI controls in an app that need as much as `Abstract Factory` for it, sounds like one of those cases.

This approach is also from the book "Design Patterns" [^ design_patterns] and was rather intended for Objective C or Smalltalk languages, but I'll come back to it later.

## Factory with registry

In the previous post about [Factory Method](https://asvid.github.io/en/kotlin-factory-method#factories-with-registry) I mentioned a factory with a registry that allows you to flexibly add new factories. A similar thing can be done with the `Abstract Factory`. I will use the same example as before, i.e. factory building GUI controls for a specific operating system.

```kotlin
// interface of the Factory that will be registered
interface ViewFactory {
    fun createButton(): Button
    fun createImage(): Image
}
// types supplied by the Factory
interface View
abstract class Button : View
abstract class Image : View
```
Concrete Factories are the same as before.
```kotlin
object LinuxViewViewFactory : ViewFactory {
    internal class LinuxButton : Button()
    internal class LinuxImage : Image()

    override fun createButton(): Button = LinuxButton()
    override fun createImage(): Image = LinuxImage()
}

object WindowsViewViewFactory : ViewFactory {
    internal class WindowsButton : Button()
    internal class WindowsImage : Image()

    override fun createButton(): Button = WindowsButton()
    override fun createImage(): Image = WindowsImage()
}
```
The Factory with a registry itself:
```kotlin
object ViewFactoryMaker {
    // registry for concrete Factories, key is just a String
    private val registry = mutableMapOf<String, ViewFactory>()

    fun register(name: String, factory: ViewFactory) {
        registry[name] = factory
    }
  
    // returning concrete Factory under provided key
    // if key is missing it will return `null`
    fun getFactory(name: String): ViewFactory? {
        return registry[name]
    }
}
```
The usage is as follows:
```kotlin
// in this case `xpButton1` will be `null` - there is no Factory under "XP" key
val xpButton1 = ViewFactoryMaker.getFactory("XP")?.createButton()

// but it can be created as anonymous object and registered
ViewFactoryMaker.register("XP", object: ViewFactory {
  override fun createButton(): Button {
    // anonymous object extending Button class
    return object: Button() {}
  }

  override fun createImage(): Image {
    return object: Image() {}
  }
})

// `xpButton2` will not be `null` this time
val xpButton2 = ViewFactoryMaker.getFactory("XP")?.createButton()
```

The registry allows you to easily add new factories if the need arises. The Abstract Factory basically provides just the interface for the specific factories and objects types created by them.

## The `make` method once again
By using the `make()` approach and the factory registry, you can achieve an interesting effect.
```kotlin
val linuxButton = ViewFactory.getFactory(ViewFactory.OS.Linux).make<Button>()
```
Pozwala to na łatwe rozszerzanie możliwości fabryk, bez konieczności pisania proxy-metod. Aby to osiągnąć, należy rozbudować `ViewFactory` o companion object i metodę `inline`. Każda konkretna fabryka będzie rozszerzać abstrakcyjne `ViewFactory` i nadpisywać metody tworzące kontrolki GUI.

This allows you to easily expand the capabilities of factories, without having to write proxy-methods. To achieve this, add a companion object and an `inline` method to the `ViewFactory`. Any concrete factory will extend the abstract `ViewFactory` and override the methods creating the GUI controls.
```kotlin
abstract class ViewFactory {
    // variants known to the Factory
    enum class OS {
        Linux, Windows
    }

    // methods to override by concrete factory
    protected abstract fun createButton(): Button
    protected abstract fun createImage(): Image

    // registry now keeps method references, and not concrete factories
    // the registry key is generic type returned by the method
    protected val registry = mutableMapOf<KClass<out View>, () -> View>(
        Button::class to ::createButton,
        Image::class to ::createImage
    )

    // this method cannot be overriden
    inline fun <reified T : View> make(): T? {
        // the `registry` field is protected, 
        // so `inline` method can't access it directly
        return `access$registry`[T::class]?.invoke() as T?
    }

    companion object {
        fun getFactory(os: OS): ViewFactory {
            return when (os) {
                OS.Linux -> LinuxViewFactory // Singleton
                OS.Windows -> WindowsViewFactory() // standard obiekt
            }
        }
    }

    // a bit nasty way to share registry with `make()` method - suggested by IDE
    // it's internal so after compilation it won't be available outside its module
	// the weird name is supposed to scare out the clients I guess
    @PublishedApi
    internal val `access$registry`: MutableMap<KClass<out View>, () -> View>
        get() = registry
}
```

So now, having a specific factory, provided by the method `getFactory()` from companion object, instead of calling a specific method on it, e.g. `createButton()` we use `make<Button>()` and the registry will find the appropriate method looking on the return type and invoke it by using `invoke()`.

Concrete Factory may look like that:
```kotlin
// standard class instead of Singleton allows `internal` classes klasa
class WindowsViewViewFactory : ViewFactory() {

    // private inner ensures the type is not visible outside
    private inner class WindowsButton : Button()
    private inner class WindowsImage : Image()

    // overriding methods creating GUI controls
    // their references will be added to the factory registry
    override fun createButton(): Button = WindowsButton()
    override fun createImage(): Image = WindowsImage()
}
```
Or like so, with a new `GridView` control added that only` LinuxViewFactory` can create
```kotlin
object LinuxViewViewFactory : ViewFactory() {
    internal class LinuxButton : Button()
    internal class LinuxImage : Image()

    //  GridView implementation exists only for Linux
    internal class LinuxGridView : GridView()

    // and the Factory has only those methods in the interface
    override fun createButton(): Button = LinuxButton()
    override fun createImage(): Image = LinuxImage()

    // adding a new control just for this one Factory
    init {
        // using `LinuxGridView` as argument will return `null`
        registry[GridView::class] = {
            LinuxGridView()
        }
    }
}
```
Some usage showcases:
```kotlin
val linuxFactory = ViewFactory.getFactory(ViewFactory.OS.Linux)
val windowsFactory = ViewFactory.getFactory(ViewFactory.OS.Windows)

val linuxButton = linuxFactory.make<Button>()
val winButton = windowsFactory.make<Button>()

val gridView = linuxFactory.make<GridView>() // OK
val winGridView = windowsFactory.make<GridView>() // null

// sadly `access$registry` is public in the same module scope
val registry = linuxFactory.`access$registry`
// and nothing will stop us from registering new control
// as long as it's implementing `View` from Abstract Factory
class NewView: View
registry[NewView::class] = { NewView() }

// legal use of the `make()` method, but will return `null`
// `View` is not present in the registry
val view = linuxFactory.make<View>()
```

Adding a new control for all specific factories only means adding an abstract method to `ViewFactory` and registering it. All derived classes will have to implement it. Adding a control only for a specific factory is also relatively simple, and the implementation is limited to the code of a specific factory. In this case, the `inline` method will not generate a huge amount of code wherever it's used, because all it does is calling the method from the registry.

Generated code for the `make<Button>()` method.
```kotlin
// `var10000` is a reference for the method returning `Button`
Function0 var10000 = (Function0)linuxFactory.getAccess$registry().get(Reflection.getOrCreateKotlinClass(Button.class));
Button linuxButton = (Button)((View)((Button)(var10000 != null ? (View)var10000.invoke() : null)));
```

So it's an interesting and quite useful hybrid of a factory with a registry and the single method for creating objects. An unquestionable advantage is the flexibility of this implementation, but public access to the method registry, and the lack of certainty whether the selected factory can create the desired type may raise doubts.

# Summary
The `Abstract Factory` pattern is useful for creating objects that can be combined into meaningful families. It constitutes a layer above the standard `Factory Method`, serving specific factories depending on the client needs. It can come in several variants, e.g. with the single `make()` method  or with a registry of specific factories, but in each case it defines the interface implemented by the supplied concrete factories. It makes no difference to the client which specific implementation it gets. The pros and cons are very close to those of the Factory Method

## Pros  
- **easy "families" of objects management** - relying solely on the factory interface, the customer can be sure that the delivered objects come from the same "family" and will work properly with each other
- **complete separation of the implementation from the interface** - from the client of the factory level, you are only interested in the object interface, allowing you to easily add new implementations without changing the client.

## Cons
- **additional classes creation** - interfaces, factories, enums, etc. This is not necessarily a disadvantage, especially when it comes to the interfaces, because it significantly helps to test the code later. You have to be wise here, and if you are providing single class instances and there is no prospect of increasing the number of types, then maybe you don't need all this boilerplate.
- **`inline` methods** - be careful with them, because they can generate a lot of code that is copied multiple times in the project
- **extra work when adding a new type** - adding a new type that is created by factories requires implementation in each specific factory. Adding a new specific factory also requires all methods to be created or overwritten. This is not necessarily a disadvantage, because the design pattern requires a fully operational factory to be delivered to the client, but it is additional work beyond adding only the classes of objects from the new family.

---
[^design_patterns]: [Wzorce projektowe - Elementy oprogramowania obiektowego wielokrotnego użytku](https://books.google.pl/books?id=Mkn6uAEACAAJ&dq=isbn:0201633612&hl=pl&sa=X&ved=2ahUKEwiQgZnevojvAhXeAxAIHerSDCsQ6AEwAHoECAAQAg)