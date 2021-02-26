---
layout: post
title: "Kotlin Factory Method"
date:  "2021-02-26 11:43"
description: "
After the `Static Factory Method` it's time for classic `Factory`. Factory is very useful and often used construction design pattern. Kotlin has interesting advantage thanks to `sealed` and `internal` classes, that are missing in Java.
"
permalink: "kotlin-factory-method"
comments: true
toc: true
tags:
- design patterns
- Kotlin
- Factory Method
- construction design pattern
  
categories:
- Design Patterns
- rss
      
image: /assets/posts/plyta.jpg

---

# Purpose

Just like Builder, the Factory is a creational pattern. It describes an interface used to deliver instances. Instead of calling object constructor, we can call a method of the Factory which will generate interface implementation - the concrete object. What makes it different from Builder is that usually none or very few arguments need to be passed. It's Factory's job to fulfill all required by the object dependencies.

Factory can deliver objects of various types implementing the same interface just by the passed arguments. The complete interface from implementation isolation allows effortless replacing implementation in runtime.

When using Factory you rather tell it: "I know just `this` and `that` give me instance implementing the expected interface". Example: `Locale.forLanguageTag("pl-PL")` will give us `Locale` instance with full country or language name even if they were not provided. In the case of Builder, it would rather be telling: "give me instance with `this` and `that` set and leave everything else default". Factories often have injected dependencies that allow them to create complex objects with just minimum input data from the client.

There are few variants of this pattern:
- Static Factory Method (already described [here](https://asvid.github.io/kotlin-static-factory-methods))
- "typical" Factory Method
- Abstract Factory

Here, I'd like to focus on Factory Method and leave Abstract Factory for the future one.

# Implementation examples

Factory Method pattern itself can take multiple forms. The most important common feature is relying only on the delivered object interface instead of concrete implementation.

## Example from "the gang of four" (simple)

Fairly basic example, coming from the book "Design Patterns: Elements of Reusable Object-Oriented Software"[^design_patterns] by so called 'gang of four' (Gamma, Helm, Johnson, Vlissides).

```kotlin
// basic example of Factory Method
val creator: Creator = ConcreteCreator()
val concreteProduct: Product = creator.factoryMethod()
concreteProduct.doStuff()

// instance can be still created with a constructor
val concreteProduct2 = ConcreteProduct()
```

{% plantuml %}
@startuml
interface Product
interface Creator{
+factoryMethod():Product
}

class ConcreteCreator extends Creator{
+factoryMethod():Product
}

class ConcreteProduct extends Product

note right of ConcreteCreator::factoryMethod
return ConcreteProduct()
end note

ConcreteCreator .left. ConcreteProduct : > creates
@enduml
{% endplantuml %}

### Elements

- **Product** - interface of the object created by the factory
- **ConcreteProduct** - implementation of `Product` interface, a concrete object that will be created by Factory
- **Creator** - Factory interface, it has declaration of the factory method returning `Product`
- **ConcreteCreator** - Factory implementation, building `Product` instance, in this case it's `ConcreteProduct`

### Implementation

```kotlin
// generic product interface
interface Product {
    fun doStuff()
}
// "concrete" product implementing interface
internal class ConcreteProduct : Product {
    override fun doStuff() {
        println("ConcreteProduct is doing stuff")
    }
}

interface Creator {
    // factory method that returns `Product` implementation
    fun factoryMethod() : Product
}

// "concrete" factory implementation, building `ConcreteProduct`
class ConcreteCreator: Creator {
    // returned type has to be generic interface, 
    // returning internal `ConcreteProduct` will cause compile time error
    override fun factoryMethod() : Product {
        println("ConcreteCreator is using factory method")
        return ConcreteProduct()
    }
}
```
There is nothing similar to `internal class` in Java, but it fits nicely into the above use. Using `internal` prevents leaking internal types. This is especially useful when building a library or SDK and you don't want to reveal implementation.

## More interesting example from the gang of four

The same book contains a more exciting example. Let's assume that we have an application, where you can create and then modify geometric figures. Each figure will contain information about its points coordinates and method to calculate area. It's easy to imagine that every figure will be scaled or moved differently. Instead of creating methods for that inside figure class, we can have another object the `Manipulator`, that will handle those operations for the given figure. But we don't want our client to know and understand implementation details for `Figure` or `Manipulator`

```kotlin
// creating an instance with the constructor
val square = Square()
// using square manipulator but through a generic interface
square.createManipulator().drag()

val figureFactory: FigureFactory = ByTypeFactory()
// creating a circle with a factory method that takes `enum` describing expected figure type
val circle: Figure = figureFactory.createFigure(FigureFactory.Type.Circle)
// similar figure manipulation
circle.createManipulator().drag()
```

{% plantuml %}
@startuml
interface Figure{
+createManipulator(): FigureManipulator
}
interface FigureManipulator {
+drag()
+resize(scale: Float)
}

class Client

Client -left-> Figure
Client -right-> FigureManipulator

class Circle extends Figure {
+createManipulator(): FigureManipulator
}
class Square extends Figure {
+createManipulator(): FigureManipulator
}

class CircleManipulator extends FigureManipulator {
-figure:Circle
+drag()
+resize(scale: Float)
}

class SquareManipulator extends FigureManipulator {
-figure:Square
+drag()
+resize(scale: Float)
}

Circle::createManipulator .right.> CircleManipulator
Square::createManipulator .right.> SquareManipulator
@enduml
{% endplantuml %}

### Elements

- **Figure** - figure itnerface implemented by classes `Circle`, `Square` and `Line`
- **FigureManipulator** - interface of figure manipulator
- **CircleManipulator** - implementation of `FigureManipulator` for concrete figure, like `Circle` in this case

### Implementation

#### Figure
```kotlin
interface Figure {
    // Figure can create FigureManipulator for itself, 
    // but for client it will always be generic and not concrete
    fun createManipulator(): FigureManipulator<out Figure> // metoda wytwórcza
}

// `internal` class means that its not available outside its module,
// but visible only for classes that its being compiled with.
// This is useful when building a library, when you don't want to leak internal types.
internal class Circle: Figure {
    // Manipulator can be replaced with new one without the need of updating clients
    override fun createManipulator() = CircleManipulator(this)
}

internal class Square : Figure {
    override fun createManipulator() = SquareManipulator(this)
}

internal class Line : Figure {
    override fun createManipulator() = LineManipulator(this)
}
```
#### Manipulator
`Manipulator` interface allows for Figure dragging and resizing. The concrete implementation is tightly connected with the type of Figure it will manipulate.
```kotlin
// generic here ensures that a concrete `FigureManipulator` 
// will handle just the concrete type of Figure, 
// ex. `CircleManipulator` can't handle `Square`
interface FigureManipulator<T : Figure> {
    fun drag()
    fun resize(scale: Float)
}

// constructor parameter type must be the one declared in FigureManipulator
internal class CircleManipulator<T>(private val figure: T) : FigureManipulator<Circle> {
    override fun drag() = println("CircleManipulator is manipulating circle $figure")
    override fun resize(scale: Float) = println("CircleManipulator is resizing circle $figure")
}
internal class SquareManipulator<T>(private val figure: T) : FigureManipulator<Square> {...}
internal class LineManipulator<T>(private val figure: T) : FigureManipulator<Line> {...}
```
#### Factory
```kotlin
interface FigureFactory {
    // enum types for provided by the factory Figures
    enum class Type { Circle, Square, Line, Undefined }

    fun createFigure(type: Type): Figure
}

// Common Factory implementation where it creates instances for provided enum type
class ByTypeFactory : FigureFactory {
    override fun createFigure(type: FigureFactory.Type): Figure =
        when (type) { // compilation error will happen if not all enum types are handled
            FigureFactory.Type.Circle -> Circle()
            FigureFactory.Type.Square -> Square()
            FigureFactory.Type.Line -> Line()
            // `Undefined` is not handled explicitly but it will fall in the `else`
            else -> throw Exception("unknown figure, don't know how to create it")
        }
}
```
`FigureFactory` contains an enum reflecting `Figure` types created by it. Not using directly types of delivered instances have a few benefits:
- It doesn't leak internal types of concrete implementations. Factory shares just the `enum` and clients see only the generic interface, not concrete types.
- To use enum type, you need to call it by factory interface like `FigureFactory.Type.Circle`. It minimizes the risk of using the wrong `Type` enum from some other factory or another place. Of course, IDE will highlight this error, but it can be just avoided. Using a less generic name like `FigureType` for enum can also help.
- `Figure` has no idea in what forms it can be created, but Factory knows what instances it can deliver. This is why figure type enum belongs to `FigureFactory` rather than `Figure`.
- The enum `FigureFactory.Type` makes sense only used with the factory. Putting it outside this interface may suggest that it's worth using it in a completely different context... which may then cause issues with maintaining the code in the future.

#### Other Factory implementations
Anonymous objects (no concrete class but implementing an interface) can also be delivered by the Factory:
```kotlin
// factory can create anonymous objects, for client it doesn't matter
class UndefinedFigureFactory : FigureFactory {
    // niezależnie od parametru `type` zwrócony będzie taki sam obiekt
    // whatever the `type` parameter is, the same object will be returned
    override fun createFigure(type: FigureFactory.Type) = object: Figure {
        // with the same manipulator
        override fun createManipulator() = object: FigureManipulator<Figure> {
            override fun drag() = println("UndefinedFigure dragging")
            override fun resize(scale: Float) = println("UndefinedFigure resizing")
        }
    }
}

// usage
UndefinedFigureFactory()
        .createFigure(FigureFactory.Type.Circle) // type doesn't really matter in this Factory
        .createManipulator() // but instance API is the same
        .drag()
```
Frequently when writing tests you don't need a concrete object but just a stub or a mock, that will allow you to verify code correctness. This is easily achievable with the Factory interface and implementing its variant for tests, which is then injected wherever it's needed:

```kotlin
// for testing it's sometimes useful to replace real factory with some kind of stub
class FakeFactory : FigureFactory {
    override fun createFigure(type: FigureFactory.Type): Figure {
        return FakeFigure()
    }
}
// for client it doesn't matter as long as it gets instance implementing `Figure` interface
class FakeFigure : Figure {
    override fun createManipulator(): FigureManipulator<out Figure> {
        return FakeFigureManipulator()
    }
}

class FakeFigureManipulator : FigureManipulator<FakeFigure> {
    override fun drag() = println("FakeFigure dragging")
    override fun resize(scale: Float) = println("FakeFigure resizing")
}

// usage
val figure = FakeFactory()
                .createFigure(FigureFactory.Type.Circle) // this wont be Circle for sure...
figure.createManipulator().drag() // but it will behave like one :)
```
You may even be tempted to randomly select the factory implementation. It doesn't make much sense in the case of geometric figures, but the procedural generated map elements or enemies in the game seem to be a pretty neat example.
```kotlin
// randomly picks `ByTypeFactory` or `FakeFactory`
// client won't notice any difference, because it knows only the `FigureFactory` interface
object RandomFigureFactory {
    fun getFigureFactory(): FigureFactory = if (Random.nextBoolean()) ByTypeFactory() else FakeFactory()
}

// uusage
RandomFigureFactory.getFigureFactory() // randomly picked `FigureFactory`, `Fake` or `ByType`
        .createFigure(FigureFactory.Type.Circle)
        // you can't be sure which object you get, but it always be `Figure`
        .createManipulator().drag() // so you can manipulate it like always
```

## Sealed class

Załóżmy, że mamy w aplikacji 3 bazy danych: `MySQL`, `Realm` i `MongoDB`. Mimo tego, że są zupełnie różne (SQL, obiektowa, No-SQL), to udostępniamy je klientom schowane za wspólnym interfejsem `Database`. Aby ułatwić sobie korzystanie z konkretnej bazy, użyjemy fabryki, która dostarczy nam instancję bazy tylko na podstawie konfiguracji.

Suppose we have 3 databases in the app: `MySQL`, `Realm` and `MongoDB`. Although they are completely different (SQL, object-oriented, No-SQL), we make them available for clients hidden behind a common interface `Database`. To facilitate the use of a specific database, we will use a factory that will provide us with an instance of the database only based on the configuration.

```kotlin
// various database configurations
val mySqlConfig = MySqlConfig("dbAddress", "port")
val realmConfig = RealmConfig // object so a Kotlin Singleton without a constructor
val mongoDbConfig = MongoDbConfig("fileUri", "table")

// you can't be sure which instance you will get from the factory
val db: Database = DatabaseFactory.getDatabaseForConfig(mySqlConfig)
// just the interface is known
db.save("Save me!")
```

{% plantuml %}
@startuml
scale 900 width

interface Database {
+save(item: Any)
+getItem(id: Int)
}

class DatabaseConfig << (S,orchid) Sealed Class >>{

}

class MySqlConfig extends DatabaseConfig{
-address
-port
}
class RealmConfig extends DatabaseConfig{
-dbName
}
class MongoDBConfig extends DatabaseConfig{
-fileUri
-tableName
}

class MySql implements Database{
-config:MySqlConfig
+save(item: Any)
+getItem(id: Int)
}
class Realm implements Database{
-config:RealmConfig
+save(item: Any)
+getItem(id: Int)
}
class MongoDB implements Database{
-config:MongoDBConfig
+save(item: Any)
+getItem(id: Int)
}

class Factory{
+getDatabaseForConfig(config: DatabaseConfig): Database
}

class BadConfig extends MongoDBConfig

Factory::getDatabaseForConfig-[bold]->Database
Factory::getDatabaseForConfig<-[bold]-DatabaseConfig
DatabaseConfig-[hidden]->Database

@enduml
{% endplantuml %}

```kotlin
// common interface available for clients
interface Database {
    fun save(item: Any)
    fun getItem(id: Int)
}

// concrete implementation of `Database`, taking configuration as constructor parameter
internal class MySql(val config: MySqlConfig) : Database {
    override fun save(item: Any) = println("saving $item in MySQL")
    override fun getItem(id: Int) = println("getting item at $id from MySQL")
}
internal class Realm(val config: RealmConfig) : Database {...}
internal class MongoDB(val config: MongoDbConfig) : Database {...}
```

There is no `enum` type to return database instance like previously, but a configuration object.

It was possible with Kotlin `sealed` class. It can be extended only with classes declared in the same file, so it gives strict control over subtypes. Some people even call it an "enum on steroids", even though `enum` itself is a fully functional object and also can have methods and fields, not only name.

```kotlin
sealed class DatabaseConfig

// since Kotlin 1.1 also `data class` can be used
// and subclasses can be created outside `sealed` class but still in the same file
data class MySqlConfig(val address: String, val port: String) : DatabaseConfig()
object RealmConfig : DatabaseConfig() // Singletons are often used with `sealed` classes

// other classes can extend an `open` class also outside this file
open class MongoDbConfig(val fileUri: String, val tableName: String) : DatabaseConfig()

// Factory creating Database instance according to provided configuration
object DatabaseFactory {
    // method is taking generic `DatabaseConfig` interface and returns generic `Database` instance
    fun getDatabaseForConfig(config: DatabaseConfig): Database {
        // `when` connected with `sealed class` guarantees handling every case
        // or compile time error - just like enum
        return when (config) {
            is MongoDbConfig -> MongoDB(config)
            is MySqlConfig -> MySql(config)
            is RealmConfig -> Realm(config)
            
            // `BadConfig` extends `MongoDbConfig` but if this case is missing it won't cause an error
            // because `BadConfig` is `MongoDbConfig` this case will be handled in first line
            is BadConfig -> MongoDB(config) 
        }
    }
}
```

Meanwhile in another file:

```kotlin
// error, you cant extend sealed class outside its file
data class BadConfig(val badData: String): DatabaseConfig()
// open class MongoDbConfig allows extending
data class BadConfig(val badData: String): MongoDbConfig("", "")
```

Using `open` classes should be well thought through, and in case of classes already extending `sealed` class, it looks like an antipattern, questioning the point of having `sealed` class in the first place.

## Factories with registry

Inside the book "Thinking in Java"[^thinking_in_java] Bruce Eckel described an interesting example of using `Static Factory Method` (already covered [here](https://asvid.github.io/kotlin-static-factory-methods)) connected with a Factory with the registry. The main point is to allow adding objects with their own factories, implementing a common interface, to the registry of top-level Factory creating their instances - without even knowing concrete types of the objects. In the previous examples, Factory interface defined types like `Square, Circle, Line` or we had a limited number of classes extending `sealed` one. In this case, the Top-level Factory role is limited to run `Static Factory Method` of registered type. Created object and its internal factory can come from the external module and be unknown to the Top-level Factory when it's being compiled - as long as it will implement correct interfaces.

```kotlin
// AirFilter has a companion object with method `create()` so a Static Factory Method
val airFilter: Part = AirFilter.create()
val fuelFilter: Part = FuelFilter.create()
val fuelFilter: Part = FuelFilter() // error, private constructor doesn't allow it

// registering the factory, class name points to companion object
RandomPartCreator.registerFactory(AirFilter)
RandomPartCreator.registerFactory(FuelFilter)
RandomPartCreator.registerFactory(OilFilter)
// factory not being a companion object can also be registered
RandomPartCreator.registerFactory(EngineFactory())

// let's have 10 instances
repeat(10) {
    // random instance creating, we don't know the concrete type
    val randomPart = RandomPartCreator.createRandomPart()
    println(randomPart.description())
    // unless we explicitly check
    println("is it an AirFilter? ${randomPart is AirFilter}")
}
```
On diagram it will look like that:

{% plantuml %}
@startuml
scale 900 width

interface Part

interface PartFactory<T: Part>{
+create():T
}

class AirFilter implements Part{
-constructor()
+companion object
}

class AirFilterFactory<AirFilter><< (O,#FF7700) CompanionObject >> implements PartFactory{
+create():AirFilter
}

class FuelFilter implements Part{
+companion object
}
class FuelFilterFactory<FuelFilter><< (O,#FF7700) CompanionObject >> implements PartFactory{
+create():FuelFilter
}

class Engine implements Part

class EngineFactory<Engine> implements PartFactory

class RandomPartCreator{
-partFactories: Set<PartFactory>
+registerFactory(factory: PartFactory)
+createRandomPart(): Part
}

RandomPartCreator::createRandomPart-[bold]->Part
RandomPartCreator::registerFactory<-[bold]-PartFactory
Part-[hidden]->PartFactory

@enduml
{% endplantuml %}

And can be implemented like this:

```kotlin
// interface implemented by every Part constructed by the Factory
interface Part {
    fun description(): String
}

// interface used by companion objects, building concrete `Part` type
interface PartFactory<T : Part> {
    fun create(): T
}

// concrete Part implementation with private construcotr
internal class FuelFilter private constructor() : Part {
    // companion object implementing Factory interface
    companion object Factory : PartFactory<FuelFilter> {
        // Static Factory Method creating an instance of the declared type
        override fun create(): FuelFilter = FuelFilter()
    }
    override fun description() = "I'm a Fuel Filter"
}
internal class AirFilter private constructor() : Part {...}
internal class OilFilter private constructor() : Part {...}

// class without companion object also can be used
class Engine: Part {
    override fun description() = "I'm an Engine!"
}
// with an external factory
class EngineFactory : PartFactory<Engine>{
    override fun create(): Engine {
        // Engine class can't have private constructor though
        return Engine()
    }
}
object RandomPartCreator {
    // registry of internal factories, so Static Factory Methods
    // Set for no duplicates
    // object created by Static Factory Method has to extend `Part`
    private val partFactories = mutableSetOf<PartFactory<out Part>>()

    // adding new internal factory to the registry
    fun registerFactory(factory: PartFactory<out Part>) {
        partFactories.add(factory)
    }
    
    // so called generator - creates instance without any parameters passed
    fun createRandomPart(): Part {
        // random number from the registry size range
        val randomFactory = Random.nextInt(partFactories.size)
        // calling `create()` on picked factory
        // unfortunately you can't take n-th element from Set without making it a list
        return partFactories.toList()[randomFactory].create()
    }
}
```

Nothing forbids using a more extensive registry, like a map with `enum` as key and Static Factory Method as value. 

# Summary
Factory Method is a commonly used design pattern, especially in the variant with `enum` types. Although it can take multiple forms main rule is the same: isolating object creation from its implementation hidden behind the interface. When having a whole family of objects it makes the program much more elastic, allowing effortless switching objects and easy mocking in tests.

Kotlin helps a lot with hiding implementation with `internal` classes, and with handling all the subclass cases thanks to the `sealed` class.

## Pros
- **isolating implementation from the interface** - the factory client is aware only about the interface, which allows adding new implementations without updating client.
- **limiting types visibility** - it's useful when you are developing a library and want to hide the internal implementation from the users.
- **ease of testing** - just the fact of using interfaces rather than concrete types makes creating a mock or stub easy. Connect this with injecting a whole configured for testing factory, and you can focus on what you actually want to test rather than building pieces around it.
- **reusing objects** - similar to Builder, returned object doesn't necessarily have to be a new instance. The constructor always returns a new instance, but Factory can have some sort of internal cache and if it makes sense return previously created instance. However, this may conflict with the intuitive understanding of what a "factory" does - building new products like a car factory will assemble new cars rather than renting them from a parking lot.

## Cons
- **additional boilerplate** - the need for interfaces, factories, enum types etc. This is not necessarily a problem, especially when thinking about interfaces and testing the code later. But you have to be prudent here, when you want to build single class instance without clear perspective of having more classes in the family - maybe you don't need a whole factory.

---
[^thinking_in_java]: [Thinking in Java](https://books.google.pl/books?id=bQVvAQAAQBAJ&dq=isbn:9780131872486&hl=pl&sa=X&ved=2ahUKEwjO6uawvojvAhVRtIsKHchPBwUQ6AEwAHoECAAQAg)
[^design_patterns]: [Design Patterns: Elements of Reusable Object-Oriented Software](https://books.google.pl/books?id=Mkn6uAEACAAJ&dq=isbn:0201633612&hl=pl&sa=X&ved=2ahUKEwiQgZnevojvAhXeAxAIHerSDCsQ6AEwAHoECAAQAg)