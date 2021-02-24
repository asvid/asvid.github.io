---
layout: post
title: "Kotlin Factory Method"
date:  "2021-02-24 11:43"
description: "
Po `Static Factory Method` nadeszła pora na klasyczne `Factory`. Fabryka jest bardzo użytecznym i często stosowanym wzorcem konstrukcyjnym. Kotlin daje nam ciekawe możliwości dzięki klasom `sealed` oraz `internal`, których odpowiedników brakuje w Javie.
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
      
image: /assets/posts/plyta.jpg

---

# Przeznaczenie

Podobnie jak Builder wzorzec Factory Method służy do oddzielenia procesu tworzenia obiektów od ich reprezentacji. Zamiast wprost wywoływać konstruktor, możemy wywołać metodę obiektu-wytwórni, który generuje implementację interfejsu. W odróżnieniu od Buildera, zazwyczaj nie będzie nas interesowało podanie wszystkich wymaganych przez obiekt argumentów i zależności — to będzie należeć do zadań Factory. 

Fabryka może dostarczać obiekty różnych typów implementujących ten sam interfejs, wyłącznie na podstawie dostarczonych argumentów. Całkowitemu odizolowanie implementacji od interfejsu pozwala na podmianę implementację w runtime, a nie tylko w czasie kompilacji.

Korzystając z Fabryki, mówimy mniej więcej: "wiem tylko `to` i `tamto`, daj mi poprawny obiekt implementujący dany interface". Przykład: `Locale.forLanguageTag("pl-PL")` - dostarczy nam obiekt `Locale`, z którego wyciągniemy sobie pełną nazwę kraju lub języka. W przypadku buildera, byłoby to raczej: "daj mi obiekt, który będzie miał ustawione `to` i `tamto` a resztę zostaw domyślne". Fabryki często mają wstrzykiwane zależności, które pozwalają im na tworzenie rozbudowanych obiektów z minimalną ilością informacji dostarczonych przez klienta.

Jest kilka wariantów tego wzorca:
- Static Factory Method (już opisany [tutaj](https://asvid.github.io/pl/kotlin-static-factory-methods) )
- "Typowe" Factory Method
- Abstract Factory

W tym poście chciałbym opisać Factory Method, zostawiając Abstract Factory na osobny wpis.

# Przykłady implementacji

Factory Method może występować pod kilkoma postaciami. Najważniejszą cechą jest poleganie wyłącznie na interfejsach zamiast na konkretnych implementacjach.

## Przykład bandy czworga (prosty)

Jest to przykład bardzo podstawowy, pochodzący z książki "Wzorce Projektowe"[^design_patterns] tzw. bandy czworga (Gamma, Helm, Johnson, Vlissides)

```kotlin
// Podstawowe użycie Factory Method
val creator: Creator = ConcreteCreator()
val concreteProduct: Product = creator.factoryMethod()
concreteProduct.doStuff()

// nadal można utworzyć instancję ConcreteProduct przez konstruktor
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
    // zwracany typ musi być ogólny, 
    // zwracanie `ConcreteProduct` który jest internal powoduje błąd kompilacji
    override fun factoryMethod() : Product {
        println("ConcreteCreator is using factory method")
        return ConcreteProduct()
    }
}
```
W Javie brak odpowiednika `internal class`, ale wydaje się dobrze pasować do powyższego zastosowania. Użycie`internal` nie pozwala wyciekać wewnętrznym typom, co jest przydatne, jeśli tworzymy bibliotekę i nie chcemy zdradzać konkretów implementacji.

## Ciekawszy przykład od bandy czworga

W tej samej książce znajduje się ciekawszy przykład. Załóżmy, że mamy aplikację, w której możemy tworzyć i następnie modyfikować figury geometryczne. Każda figura będzie miała jakieś współrzędne i na przykład sposób obliczania pola. Łatwo sobie wyobrazić, że każdy typ figury będzie się inaczej skalować lub zmieniać jej położenie. Zamiast tworzyć metody wewnątrz figury, można stworzyć nowy obiekt — Manipulator, który będzie wiedział jak zmienić położenie wierzchołków danej figury. Jednak nie chcemy, żeby klient naszego API musiał znać szczegóły implementacyjne Manipulatora albo Figury.

```kotlin
// utworzenie instancji przez konstruktor
val square = Square()
// użycie manipulatora dla kwadratu, ale przez generyczny interface
square.createManipulator().drag()

val figureFactory: FigureFactory = ByTypeFactory()
// stworzenie koła przez fabrykę przyjmującą `enum` z oczekiwanym rodzajem figury
val circle: Figure = figureFactory.createFigure(FigureFactory.Type.Circle)
// analogiczna modyfikacja figury
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

### Elementy

- **Figure** - interface figury implementowany przez klasy `Circle`, `Square` i `Line`
- **FigureManipulator** - interface obiektu umożliwiającego modyfikację figury
- **CircleManipulator** - implementacja `FigureManipulator` dla konkretnej figury, np `Circle`

### Implementacja

#### Figure
```kotlin
interface Figure {
    // Figure potrafi stworzyć dla siebie FigureManipulator, 
    // ale dla klienta jest on generyczny a nie konkretny.
    fun createManipulator(): FigureManipulator<out Figure> // metoda wytwórcza
}

// `internal` oznacza że klasa nie jest dostępna poza modułem, 
// jest dostępna tylko dla klas z którymi jest kompilowana.
// Jest to przydatne podczas tworzenia bibliotek kiedy nie chcemy wyciekać konkretnych typów
internal class Circle: Figure {
    // w razie czego manipulator może zostać zastąpiony innym, bez konieczności aktualizacji klienta
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
Interface `Manipulatora` pozwala na przeciąganie i zmianę rozmiaru. Konkretna implementacja tego interfejsu jest ściśle związana z typem figury.
```kotlin
// wykorzystanie generyków zapewnia, że konkretny FigureManipulator będzie umiał obsłużyć 
// tylko konkretny typ figury np. CircleManipulator nie przyjmie argumentu typu Square
interface FigureManipulator<T : Figure> {
    fun drag()
    fun resize(scale: Float)
}

// typ argumentu w konstruktorze musi zgadzać się z zadeklarowanym przez FigureManipulator
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
    // umieszczenie enuma wewnątrz klasy Factory zamiast Figure,
    // to fabryka zna typy produkowanych przez siebie obiektów, 
    // a nie sam obiekt wie w jakich formach występuje
    enum class Type { Circle, Square, Line, Undefined }

    fun createFigure(type: Type): Figure
}

// Częste wykorzystanie factory do zwracania obiektu danego typu z wykorzystaniem enuma
class ByTypeFactory : FigureFactory {
    override fun createFigure(type: FigureFactory.Type): Figure =
        when (type) { // wystąpi błąd kompilacji jeśli nie obsłużymy wszystkich typów
            FigureFactory.Type.Circle -> Circle()
            FigureFactory.Type.Square -> Square()
            FigureFactory.Type.Line -> Line()
            // typ Undefined nie jest obsłużony, ale wpadnie w `else`
            else -> throw Exception("unknown figure, don't know how to create it")
        }
}
```
FigureFactory zawiera w sobie `enum` odpowiadający typom figur, jakie dostarcza. Nie są to bezpośrednio typy samych obiektów, a jedynie pomocniczy typ wyliczeniowy. Umieszczenie jej wewnątrz FigureFactory ma kilka zalet:
- aby użyć typu wyliczeniowego, trzeba się do niego odwołać przez interface np. `FigureFactory.Type.Circle`. Zmniejsza ryzyko użycia złego typu, jeśli korzystamy z wielu fabryk w jednym miejscu i każda ma swój enum `Type` zadeklarowany w pliku, a nie w interfejsie. Takiego problemu można również uniknąć nazywając `enum` mniej ogólnie, np. `FigureType`, ale nadal inna fabryka może chcieć korzystać z takiej nazwy.
- Figure nie wie pod jakimi postaciami może występować, ale Fabryka wie jakie instancje może dostarczać.
- Poza tym, enum `FigureType` ma sens tylko użyty z `FigureFactory`. Umieszczenie go poza tą klasą, może sugerować, że warto go również użyć w zupełnie innym kontekście... co może prowadzić do problemów z utrzymaniem kodu w przyszłości.

#### Inne implementacje Factory
Obiekty anonimowe (bez dokładnej klasy, ale implementujące interface) również mogą być dostarczane przez Fabrykę:
```kotlin
// fabryka może dostarczać obiekty anonimowe, dla klienta to bez znaczenia
class UndefinedFigureFactory : FigureFactory {
    // niezależnie od parametru `type` zwrócony będzie taki sam obiekt
    override fun createFigure(type: FigureFactory.Type) = object: Figure {
        // z takim samym manipulatorem
        override fun createManipulator() = object: FigureManipulator<Figure> {
            override fun drag() = println("UndefinedFigure dragging")
            override fun resize(scale: Float) = println("UndefinedFigure resizing")
        }
    }
}

// użycie
UndefinedFigureFactory()
        .createFigure(FigureFactory.Type.Circle) // w tym przypadku typ nie ma znaczenia
        .createManipulator() // ale API obiektu jest takie samo
        .drag()
```
Często podczas testowania nie potrzebujemy konkretnego obiektu, a jedynie zaślepkę lub Mock, który pozwoli nam zweryfikować poprawność działania programu. Można to łatwo osiągnąć, jeśli mamy interfejs Fabryki i możemy zaimplementować jej wersję pod testy, a następnie wstrzyknąć tam, gdzie jest używana:
```kotlin
// w testach czasami przydaje się zastąpienie prawdziwego factory jakąś zaślepką
class FakeFactory : FigureFactory {
    override fun createFigure(type: FigureFactory.Type): Figure {
        return FakeFigure()
    }
}
// dla klienta to nadal bez znaczenia jaki dostaje obiekt, tak długo jak implementuje `Figure`
class FakeFigure : Figure {
    override fun createManipulator(): FigureManipulator<out Figure> {
        return FakeFigureManipulator()
    }
}

class FakeFigureManipulator : FigureManipulator<FakeFigure> {
    override fun drag() = println("FakeFigure dragging")
    override fun resize(scale: Float) = println("FakeFigure resizing")
}

// użycie
val figure = FakeFactory()
                .createFigure(FigureFactory.Type.Circle) // to raczej nie będzie kółko...
figure.createManipulator().drag() // ale działa jak kółko :)
```
Można się nawet pokusić o losowe wybieranie implementacji fabryki. Nie ma to za bardzo sensu w przypadku figur geometrycznych, ale proceduralne generowane elementów mapy czy przeciwników w grze wydaje się już całkiem dobrym przykładem.
```kotlin
// losowo zwraca ByTypeFactory lub FakeFactory, 
// dla klienta to bez różnicy bo i tak zna tylko interface FigureFactory
object RandomFigureFactory {
    fun getFigureFactory(): FigureFactory = if (Random.nextBoolean()) ByTypeFactory() else FakeFactory()
}

// użycie
RandomFigureFactory.getFigureFactory() // losowo wybrane FigureFactory, albo Fake albo ByType
        .createFigure(FigureFactory.Type.Circle)
        // nie mamy pewności jaki obiekt dostaniemy, ale zawsze będzie to Figure
        .createManipulator().drag() // więc można ten obiekt zmieniać zgodnie z jego API
```

## Sealed class

Załóżmy, że mamy w aplikacji 3 bazy danych: `MySQL`, `Realm` i `MongoDB`. Mimo tego, że są zupełnie różne (SQL, obiektowa, No-SQL), to udostępniamy je klientom schowane za wspólnym interfejsem `Database`. Aby ułatwić sobie korzystanie z konkretnej bazy, użyjemy fabryki, która dostarczy nam instancję bazy tylko na podstawie konfiguracji.

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

Nie ma tutaj podawania typu bazy z `enum` jak poprzednio, tylko cały obiekt konfiguracji z właściwymi tylko dla siebie polami.

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
internal class Realm(val config: RealmConfig) : Database {...}
internal class MongoDB(val config: MongoDbConfig) : Database {...}
```
Udało się to osiągnąć dzięki Kotlinowym klasom `sealed`. Mogą po niej dziedziczyć wyłącznie klasy zadeklarowane w tym samym pliku, mamy więc ścisłą kontrolę nad typami pochodnymi. Niektórzy nazywają to nawet "enumem na sterydach", chociaż sam `enum` jest pełnoprawną klasą i również może mieć pola i metody, a nie tylko nazwę.

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
// klasa konfiguracji bazy jest `sealed` co oznacza ścisłą kontrolę nad tym,
// które klasy mogą z niej dziedziczyć 
sealed class DatabaseConfig

// od Kotlin 1.1 można używać `data class` i tworzyć je poza `sealed` klasą, 
// ale tylko w tym samym pliku
data class MySqlConfig(val address: String, val port: String) : DatabaseConfig()
object RealmConfig : DatabaseConfig() // z klasy `sealed` często dziedziczą Singletony

// po tej klasie `open` można dziedziczyć również poza plikiem z klasą `sealed`
open class MongoDbConfig(val fileUri: String, val tableName: String) : DatabaseConfig()

// Fabryka dostarczająca implementację Database pod konkretną konfigurację
object DatabaseFactory {
    // metoda przyjmuje generyczny interface i zwraca generyczną instancję Database
    fun getDatabaseForConfig(config: DatabaseConfig): Database {
        // when w połączeniu z sealed class daje pewność obsłużenia wszystkich przypadków,
        // lub błąd kompilacji
        return when (config) {
            is MongoDbConfig -> MongoDB(config)
            is MySqlConfig -> MySql(config)
            is RealmConfig -> Realm(config)
            // brak tego przypadku nie spowoduje błędu kompilacji, 
            // bo `BadConfig` nie dziedziczy bezpośrednio z klasy `sealed`
            is BadConfig -> MongoDB(config) 
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
Jednak samo używanie klas `open` powinno być dobrze przemyślane, a w przypadku klas już rozszerzających klasę `sealed` wygląda na antywzorzec, podający w wątpliwość sens użycia `sealed`.

## Rejestrowane fabryki

W książce "Thinking in Java"[^thinking_in_java] Bruce Eckel opisał ciekawy przykład zastosowania `Static Factory Method` (opisany [tutaj](https://asvid.github.io/pl/kotlin-static-factory-methods)) w połączeniu z fabryką z rejestrem. Ogólnie chodzi o to umożliwienie dodawania obiektów z własnymi fabrykami, implementującymi jakiś wspólny interfejs, do rejestru nadrzędnej fabryki dostarczającej ich instancje — bez znajomości konkretnych typów obiektów. Poprzednio Fabryka sama definiowała te typy jak `Square, Circle, Line` lub mieliśmy skończoną liczbę klas dziedziczących z `sealed class`, a w tym przypadku rola fabryki ogranicza się w zasadzie do uruchomienia `Static Factory Method` zarejestrowanego typu. Sam obiekt i jego wewnętrzna fabryka może pochodzić z dowolnego miejsca i nie być znany nadrzędnej fabryce w momencie kompilacji — tak długo, jak implementowane są odpowiednie interfejsy.

```kotlin
// AirFilter ma companion object z metodą `create()` czyli Static Factory Method
val airFilter: Part = AirFilter.create()
val fuelFilter: Part = FuelFilter.create()
val fuelFilter: Part = FuelFilter() // błąd, prywatny konstruktor na to nie pozwala

// rejestrowanie factory, nazwa klasy wskazuje na companion object
RandomPartCreator.registerFactory(AirFilter)
RandomPartCreator.registerFactory(FuelFilter)
RandomPartCreator.registerFactory(OilFilter)
// zarejestrować można też fabrykę nie będącą companion object
RandomPartCreator.registerFactory(EngineFactory())

// tworzymy 10 instancji
repeat(10) {
    // losowe tworzenie instancji, nie znamy tutaj konkretnego typu
    val randomPart = RandomPartCreator.createRandomPart()
    println(randomPart.description())
    // no chyba że sprawdzimy
    println("is it AirFilter? ${randomPart is AirFilter}")
}
```
Od środka wygląda to następująco:

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
internal class AirFilter private constructor() : Part {...}
internal class OilFilter private constructor() : Part {...}

// klasa bez companion object nadal może być użyta
class Engine: Part {
    override fun description() = "I'm an Engine!"
}
// fabryka poza obiektem
class EngineFactory : PartFactory<Engine>{
    override fun create(): Engine {
        // klasa Engine nie może mieć jednak prywatnego konstruktora
        return Engine()
    }
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
        // wywołanie metody `create()` na wylosowanej fabryce
        // niestety nie da się wyciągnąć n-tego elementu z Set-u bez zmiany typu na listę
        return partFactories.toList()[randomFactory].create()
    }
}
```
Nic nie stoi na przeszkodzie, żeby rejestr w Fabryce był bardziej rozbudowany, np. do postaci mapy gdzie kluczem będzie typ wyliczeniowy a zawartością Statyczna Metoda Fabryczna.

# Podsumowanie
Fabryka jest często używanym wzorcem, zwłaszcza w formie ze zwracaniem obiektów na podstawie `enuma`. Mimo, że występuje w wielu formach, zasada jest zawsze ta sama: odizolowanie tworzenia obiektu od jego implementacji schowanej za interfejsem. W przypadku posiadania rodziny obiektów znacząco to uelastycznia program, pozwalając sprawnie podmieniać obiekty i wygodnie tworzyć mocki w testach.

Kotlin bardzo pomaga w chowaniu implementacji przez `internal` klasy, oraz obsłudze wszystkich podklas dzięki klasom `sealed`.

## Zalety
- **całkowite oddzielenie implementacji od interfejsu** - z poziomu klienta fabryki interesuje nas wyłącznie interfejs obiektu, co pozwala na łatwe dodawanie nowych implementacji bez konieczności zmian klienta.
- **ograniczenie widoczności typów** - jest to przydatne, jeśli tworzymy bibliotekę i chcemy schować wewnętrzną implementację przed użytkownikiem.
- **łatwość testowania** - już sam fakt polegania na interfejsach, zamiast konkretnej implementacji, pozwala łatwo stworzyć Mocka lub Stuba. W połączeniu z możliwością wstrzykiwania całej, skonfigurowanej na potrzeby testu fabryki, ułatwia to wyizolowanie testowanej logiki i skupienie się na tym, co faktycznie chcemy przetestować.

## Wady
- **konieczność tworzenia dodatkowych klas** - interfejsów, fabryk, typów wyliczeniowych itd. Niekoniecznie jest to wada, zwłaszcza jeśli chodzi o interfejsy, bo znacząco pomaga to potem testować kod. Należy wykazać się tutaj rozsądkiem, jeśli dostarczamy instancje pojedynczej klasy i nie ma perspektyw na zwiększenie liczby typów, to może nie ma potrzeby tworzyć ten cały boilerplate.

---
[^thinking_in_java]: [Thinking in Java](https://helion.pl/ksiazki/thinking-in-java-edycja-polska-wydanie-iv-bruce-eckel,thi4vv.htm#format/d)
[^design_patterns]: [Wzorce projektowe - Elementy oprogramowania obiektowego wielokrotnego użytku](https://ksiegarnia.pwn.pl/Wzorce-projektowe,68608777,p.html)