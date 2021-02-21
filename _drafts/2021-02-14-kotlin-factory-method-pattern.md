---
layout: post
title: "Kotlin Factory Method"
date:  "2021-01-20 11:43"
description: "
TBD
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
      
image: /assets/posts/kotlin-builder-pattern/pkin.jpg

---

# Przeznaczenie

[comment]: <> (sprawdzić definicje z Design Patterns bandy 4 i Thinking in Java)

Podobnie jak Builder wzorzec Factory Method służy do oddzielenia procesu tworzenia obiektów od ich reprezentacji. Zamiast wprost wywoływać konstruktor, możemy wywołać metodę obiektu-wytwórni, który generuje implementację interfejsu. W odróżnieniu od Buildera, zazwyczaj nie będzie nas interesowało podanie wszystkich wymaganych przez obiekt argumentów i zależności — to będzie należeć do zadań Factory. Fabryka może dostarczać obiekty różnych typów implementujących ten sam interfejs, wyłącznie na podstawie dostarczonych argumentów. Dzięki całkowitemu odizolowaniu implementacji od interfejsu Fabryka pozwala nam to podmieniać implementację w runtime, a nie tylko w czasie kompilacji.

Korzystając z Fabryki, mówimy sobie mniej więcej: "wiem tylko `to` i `tamto`, daj mi poprawny obiekt danego typu". Przykład: `Locale.forLanguageTag("pl-PL")` - dostarczy nam obiekt `Locale`, z którego wyciągniemy sobie pełną nazwę kraju lub języka. W przypadku buildera, byłoby to raczej  "daj mi obiekt, który będzie miał ustawione `to` i `tamto` a resztę zostaw domyślne". Fabryki często mają wstrzykiwane zależności, które pozwalają im na tworzenie rozbudowanych obiektów z minimalną ilością informacji dostarczonych przez klienta.

Jest kilka wariantów tego wzorca:
- Static Factory Method (już opisany [tutaj]({% post_url 2021-02-14-kotlin-static-factory-method-pattern %}))
- "Typowe" Factory Method
- Abstract Factory

W tym poście chciałbym opisać Factory Method, zostawiając Abstract Factory na osobny wpis.

# Przykłady implementacji

Factory Method może występować pod kilkoma postaciami. Najważniejszą cechą jest poleganie wyłącznie na interfejsach zamiast na konkretnych implementacjach.

## Przykład bandy czworga (prosty)

Jest to przykład bardzo podstawowy, pochodzący z książki "Wzorce Projektowe" tzw. bandy czworga (Gamma, Helm, Johnson, Vlissides)

```kotlin
// Podstawowe użycie Factory Method
val creator: Creator = ConcreteCreator()
val concreteProduct: Product = creator.factoryMethod()
concreteProduct.doStuff()

// nadal można utworzyć instancję ConcreteProduct przez konstruktor
val concreteProduct2 = ConcreteProduct()
```

[comment]: <> (tutaj dać UMLa)

### Elementy

- **Product** - interface instancji dostarczanej przez fabrykę
- **ConcreteProduct** - implementacja interfejsu, czyli konkretny obiekt, który zostanie zbudowany
- **Creator** - interface fabryki, zawiera deklarację metody wytwarzającej instancję typu `Product`
- **ConcreteCreator** - implementacja fabryki budująca instancję `Product`, w tym wypadku `ConcreteProduct`

### Implementacja

```kotlin
interface Product {
    fun doStuff()
}
// "konkretny" produkt implementujący interface
internal class ConcreteProduct : Product {
    override fun doStuff() {
        println("ConcreteProduct is doing stuff")
    }
}

interface Creator {
    // metoda fabryczna dostarczająca instancję obiektu implementującego interface `Product`
    fun factoryMethod() : Product
}

// "konkretna" implementacja fabryki, wytwarzająca `ConcreteProduct`
class ConcreteCreator: Creator {
    // zwracany typ musi być ogólny, zwracanie `ConcreteProduct` który jest internal powoduje błąd kompilacji
    override fun factoryMethod() : Product {
        println("ConcreteCreator is using factory method")
        return ConcreteProduct()
    }
}
```

## Ciekawszy przykład od bandy czworga

W tej samej książce znajduje się ciekawszy przykład. Załóżmy, że mamy aplikację, w której możemy tworzyć i następnie modyfikować figury geometryczne. Każda figura będzie miała jakieś współrzędne i na przykład sposób obliczania pola. Łatwo sobie wyobrazić, że każdy typ figury będzie się inaczej skalować lub zmieniać jej położenie. Zamiast tworzyć metody wewnątrz figury, można stworzyć nowy obiekt — Manipulator, który będzie wiedział jak zmienić położenie wierzchołków danej figury. Jednak nie chcemy, żeby klient naszego API musiał znać szczegóły implementacyjne Manipulatora albo Figury.

```kotlin
// utworzenie instancji przez konstruktor
val square = Square()
// użycie manipulatora dla kwadratu, ale przez generyczny interface
square.createManipulator().drag()

val shapeFactory: ShapeFactory = ByTypeFactory()
// stworzenie koła przez fabrykę przyjmującą `enum` z oczekiwanym typem figury
val circle: Shape = shapeFactory.createShape(ShapeFactory.Type.Circle)// analogiczna modufikacja figury
circle.createManipulator().drag()
```

[comment]: <> (UML)

### Elementy

- **Shape** - interface figury implementowany przez klasy `Circle`, `Square` i `Line`
- **Manipulator** - interface obiektu umożliwiającego modyfikację figury
- **ShapeFactory** - interface fabryki tworzącej figury na podstawie enuma `Type`
- **ByTypeFactory** - implementacja `ShapeFactory`

### Implementacja

```kotlin
interface Shape {
    // Shape potrafi stworzyć dla siebie Manipulator, ale dla klienta jest on generyczny a nie konkretny
    fun createManipulator(): ShapeManipulator<out Shape> // metoda wytwórcza
}

// `internal` oznacza że klasa nie jest dostępna poza modułem - jest dostępna tylko dla klas z którymi jest kompilowana
// jest to przydatne podczas tworzenia bibliotek kiedy nie chcemy wyciekać konkretnych typów
internal class Circle: Shape {
    // w razie czego manipulator może zostać zastąpiony innym, bez konieczności aktualizacji klienta
    override fun createManipulator() = CircleManipulator(this)
}

internal class Square : Shape {
    override fun createManipulator() = SquareManipulator(this)
}

internal class Line : Shape {
    override fun createManipulator() = LineManipulator(this)
}
```

```kotlin
// wykorzystanie generyków zapewnia że konkretny Manipulator będzie umiał obsłużyć tylko konkretny typ figury
// np. CircleManipulator nie przyjmie argumentu typu Square
interface ShapeManipulator<T : Shape> {
    fun drag()
    fun resize(scale: Float)
}

// typ argumentu w konstruktorze musi zgadzać się z zadeklarowanym przez Manipulator
internal class CircleManipulator<T>(private val shape: T) : ShapeManipulator<Circle> {
    override fun drag() = println("CircleManipulator is manipulating circle $shape")
    override fun resize(scale: Float) = println("CircleManipulator is resizing circle $shape")
}

internal class SquareManipulator<T>(private val shape: T) : ShapeManipulator<Square> {
    override fun drag() = println("SquareManipulator is manipulating square $shape")
    override fun resize(scale: Float) = println("SquareManipulator is resizing square $shape")
}

internal class LineManipulator<T>(private val shape: T) : ShapeManipulator<Line> {
    override fun drag() = println("LineManipulator is manipulating line $shape")
    override fun resize(scale: Float) = println("LineManipulator is resizing line $shape")
}
```

```kotlin
interface ShapeFactory {
    // umieszczenie enuma wewnątrz klasy Factory zamiast Shape - to fabryka zna typy produkowanych przez siebie obiektów, 
    // a nie sam obiekt wie w jakich formach występuje
    enum class Type { Circle, Square, Line, Undefined }

    fun createShape(type: Type): Shape
}

// Częste wykorzystanie factory do zwracania obiektu danego typu
class ByTypeFactory : ShapeFactory {
    override fun createShape(type: ShapeFactory.Type): Shape =
        when (type) { // zastosowanie `when` spowoduje błąd kompilacji jeśli nie obsłużymy wszystkich typów
            ShapeFactory.Type.Circle -> Circle()
            ShapeFactory.Type.Square -> Square()
            ShapeFactory.Type.Line -> Line()
            // typ Undefined nie jest obsłużony, ale wpadnie w `else`
            else -> throw Exception("unknown shape, don't know how to create it")
        }
}
```

Interface fabryki mogą implementować inne obiekty, np:
```kotlin
// fabryka może też dostarczać obiekty anonimowe, dla klienta to bez znaczenia
class UndefinedShapeFactory : ShapeFactory {
    // niezależnie od parametru `type` zwrócony będzie taki sam obiekt
    override fun createShape(type: ShapeFactory.Type) = object : Shape {
        // z takim samym manipulatorem
        override fun createManipulator() = object : ShapeManipulator<Shape> {
            override fun drag() = println("UndefinedShape dragging")
            override fun resize(scale: Float) = println("UndefinedShape resizing")
        }
    }
}

// użycie
UndefinedShapeFactory()
        .createShape(ShapeFactory.Type.Circle)
        .createManipulator()
        .drag()
```
lub:
```kotlin
// w testach czasami przydaje się zastąpienie prawdziwego factory jakąś zaślepką
class FakeFactory : ShapeFactory {
    override fun createShape(type: ShapeFactory.Type): Shape {
        return FakeShape()
    }
}
// dla klienta to nadal bez znaczenia jaki dostaje obiekt, tak długo jak implementuje `Shape`
class FakeShape : Shape {
    override fun createManipulator(): ShapeManipulator<out Shape> {
        return FakeShapeManipulator()
    }
}

class FakeShapeManipulator : ShapeManipulator<FakeShape> {
    override fun drag() = println("FakeShape dragging")
    override fun resize(scale: Float) = println("FakeShape resizing")
}

// użycie
val shape = FakeFactory().createShape(ShapeFactory.Type.Circle) // to raczej nie będzie kółko...
shape.createManipulator().drag() // ale działa jak kółko :)
```

Można się nawet pokusić o losowe wybieranie implementacji fabryki. Nie ma to za bardzo sensu w przypadku figur geometrycznych, ale proceduralne generowane elementów mapy czy przeciwników w grze wydaje się już całkiem dobrym przykładem.
```kotlin
// losowo zwraca ByTypeFactory lub FakeFactory, dla klienta to bez różnicy bo i tak zna tylko interface ShapeFactory
object RandomShapeFactory {
    fun getShapeFactory(): ShapeFactory = if (Random.nextBoolean()) ByTypeFactory() else FakeFactory()
}

// użycie
RandomShapeFactory.getShapeFactory() // losowo wybrane ShapeFactory, albo Fake albo ByType
        .createShape(ShapeFactory.Type.Circle) // nigdy nie mamy pewności jaki obiekt dostaniemy, ale zawsze będzie to Shape
        .createManipulator().drag() // więc można ten obiekt zmieniać zgodnie z jego API
```

## Sealed class

Załóżmy, że mamy w aplikacji 3 bazy danych: MySQL, Realm i MongoDB. Mimo że są zupełnie różne (SQL, obiektowa, No-SQL), to udostępniamy je klientom schowane za wspólnym interfejsem `Database`. Żeby ułatwić sobie dostanie się do konkretnej bazy mamy fabrykę która da nam instancję na podstawie konfiguracji.

```kotlin
// różne konfiguracje baz danych
val mySqlConfig = MySqlConfig("dbAddress", "port")
val realmConfig = RealmConfig // object czyli Singleton nie ma konstruktora
val mongoDbConfig = MongoDbConfig("fileUri", "table")

// nie wiemy jaką instancję bazy dostaniemy z fabryki
val db: Database = DatabaseFactory.getDatabaseForConfig(mySqlConfig)
// znamy tylko generyczny interfejs
db.save("Save me!")
```

Nie ma tutaj podawania typu z `enum` jak poprzednio, tylko cały obiekt konfiguracji z indywidualnymi polami itd.

```kotlin
// wspólny interface bazy udostępniany klientowi
interface Database {
    fun save(item: Any)
    fun getItem(id: Int)
}

// konkretna implementacja bazy przyjmująca konfigurację w konstruktorze
internal class MySql(val config: MySqlConfig) : Database {
    override fun save(item: Any) = println("saving $item in MySQL")
    override fun getItem(id: Int) = println("getting item at $id from MySQL")
}

internal class Realm(val config: RealmConfig) : Database {
    override fun save(item: Any) = println("saving $item in Realm")
    override fun getItem(id: Int) = println("getting item at $id from Realm")
}

internal class MongoDB(val config: MongoDbConfig) : Database {
    override fun save(item: Any) = println("saving $item in MongoDB")
    override fun getItem(id: Int) = println("getting item at $id from MongoDB")
}
```

```kotlin
// klasa konfiguracji bazy jest `sealed` co oznacza ścisłą kontrolę nad tym jakie klasy mogą z niej dziedziczyć 
sealed class DatabaseConfig

// od Kotlin 1.1 można używać data class i tworzyć je poza sealed klasą, ale w tym samym pliku
data class MySqlConfig(val address: String, val port: String) : DatabaseConfig()
object RealmConfig : DatabaseConfig() // singletony mogą dziedziczyć z klas sealed
open class MongoDbConfig(val fileUri: String, val tableName: String) : DatabaseConfig() // po tej klasie można dziedziczyć

// Fabryka dostarczająca implementację Database pod konkretną konfigurację
object DatabaseFactory {
    // metoda przyjmuje generyczny interface i zwraca generyczną instancję Database
    fun getDatabaseForConfig(config: DatabaseConfig): Database {
        // when w połączeniu z sealed class daje pewność obsłużenia wszystkich przypadków, lub błąd kompilacji
        return when (config) {
            is MongoDbConfig -> MongoDB(config)
            is MySqlConfig -> MySql(config)
            is RealmConfig -> Realm(config)
        }
    }
}
```
Tymczasem w innym pliku:
```kotlin
// błąd, nie można rozszerzać klasy sealed poza plikiem gdzie się znajduje
data class BadConfig(val badData: String): DatabaseConfig()
// klasa MongoDbConfig jest open, więc można z niej dziedziczyć
data class BadConfig(val badData: String): MongoDbConfig("", "")
```

## Rejestrowane fabryki

W książce "Thinking in Java" jest ciekawy przykład zastosowania `Static Factory Method` (opisany [tutaj]()) w połączeniu z fabryką z rejestrem. Ogólnie chodzi o to umożliwienie dodawania obiektów z własnymi fabrykami, implementującymi jakiś wspólny interfejs, do rejestru nadrzędnej fabryki dostarczającej ich instancje - bez znajomości konkretnych typów obiektów. Poprzednio Fabryka sama definiowała te typy jak `Square, Circle, Line`, a w tym przypadku rola fabryki ogranicza się do uruchomienia `Static Factory Method` zarejestrowanego obiektu.

```kotlin
// AirFilter ma companion object z metodą `create()` czyli Static Factory Method
val airFilter: Part = AirFilter.create()
val fuelFilter: Part = FuelFilter.create()
val fuelFilter: Part = FuelFilter() // błąd, prywatny konstruktor na to nie pozwala

// rejestrowanie factory, nazwa klasy wskazuje na companion object
RandomPartCreator.registerFactory(AirFilter)
RandomPartCreator.registerFactory(FuelFilter)
RandomPartCreator.registerFactory(OilFilter)

// tworzymy instancje 10x
repeat(10) {
    // losowe tworzenie instancji, nie znamy tutaj konkretnego typu
    val randomPart = RandomPartCreator.createRandomPart()
    println(randomPart.description())
    // no chyba że sprawdzimy
    println("is it AirFilter? ${randomPart is AirFilter}")
}
```

Od środka wygląda to następująco:
```kotlin
// interface implementowany przez każdą część dostarczaną przez nadrzędną Fabrykę
interface Part {
    fun description(): String
}

// interface dla companion object, dostarczający konkretny typ części rozszerzający `Part`
interface PartFactory<T : Part> {
    fun create(): T
}

// implementacja konkretnej części z prywatnym konstruktorem
internal class FuelFilter private constructor() : Part {
    // companion object implementujący interface
    companion object Factory : PartFactory<FuelFilter> {
        // Static Factory Method dostarczająca instancję zgodną z zadeklarowanym typem
        override fun create(): FuelFilter = FuelFilter()
    }
    override fun description() = "I'm a Fuel Filter"
}

internal class AirFilter private constructor() : Part {
    companion object Factory : PartFactory<AirFilter> {
        override fun create(): AirFilter = AirFilter()
    }
    override fun description() = "I'm a Air Filter"
}

internal class OilFilter private constructor() : Part {
    companion object Factory : PartFactory<OilFilter> {
        override fun create(): OilFilter = OilFilter()
    }
    override fun description() = "I'm a Oil Filter"
}

object RandomPartCreator {
    // rejestr wewnętrznych fabryk, czyli Static Factory Method
    // Set zapewnia brak duplikatów
    // typ tworzony przez PartFactory musi rozszerzać `Part`
    private val partFactories = mutableSetOf<PartFactory<out Part>>()

    // dodanie nowej fabryki do rejestru
    fun registerFactory(factory: PartFactory<out Part>) {
        partFactories.add(factory)
    }

    fun createRandomPart(): Part { // tzw. generator - tworzy instancję bez podawania parametrów
        // losowa liczba z zakresu ograniczonego rozmiarem rejestru
        val randomFactory = Random.nextInt(partFactories.size)
        // niestety nie da się wyciągnąć n-tego elementu z Set-u bez zmiany typu na listę
        return partFactories.toList()[randomFactory].create() // wywołanie metody `create()` na wylosowanej fabryce
    }
}
```