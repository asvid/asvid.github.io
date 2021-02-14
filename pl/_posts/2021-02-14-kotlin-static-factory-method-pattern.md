---
layout: post
title: "Kotlin Static Factory Methods"
date:  "2021-01-20 11:43"
description: "
Znane z Javy `Statyczne Metody Wytwórcze` w Kotlinie jak najbardziej mają swoje zastosowanie, pomimo trochę innego zachowania i braku słówka `static`. Postaram się przybliżyć czym jest `companion object` i jak go można używać.
PS: Ten post miał byc o wzorcu Factory Method z tylko krótką wzmianą o statycznych metodach fabrycznych, ale ten temat okazał się ciekawszy, niż sądziłem :)
"
permalink: "kotlin-static-factory-methods"
comments: true
toc: true
tags:
- design patterns
- Kotlin
- Static Factory Methods
- construction design pattern
  
categories:
- Design Patterns
      
image: /assets/posts/lodz.jpg

---

# Przeznaczenie

W tytule jest świadomy błąd, w Kotlinie nie ma czegoś takiego jak statyczne metody. Są podobne mechanizmy pozwalające na tworzenie instancji zgodnie z propozycją Joshua Blocha z książki Effective Java — używaj statycznych metod fabrycznych zamiast konstruktora. Jest to zupełne co innego niż wzorzec Factory Method.

W skrócie są to metody, które tworzą żądane obiekty na podstawie podanych argumentów (lub nawet bez nich) i które można wywołać w dowolnym miejscu bez konieczności tworzenia instancji klasy, w której się znajdują. Żeby móc wywołać taką metodę w Javie używane jest słówko `static`, co oznacza, że metoda należy do klasy (rozumianej jako typ), a nie obiektu i można ją wywołać bez tworzenia instancji. W Kotlinie nie mamy takiej możliwości, ale taki sam efekt da się osiągnąć dzięki `companion object`.

# Przykłady użycia
Nawet jeśli fraza "static factory method" brzmi obco, z pewnością trafiliście  w kodzie na ich użycie. Typowe statyczne metody wytwórcze wyglądają tak:
```kotlin
val locale = Locale.forLanguageTag("PL")
val today = LocalDate.now(ZoneId.of("GMT"))
val time = LocalDateTime.of(2021, 1, 1, 12, 34, 23, 123)
val someNumber = Double.fromBits(0x10000000000000L)
val formattedPi = String.format("%.2f", 3.14159265358979323)
```
Również wszystkie typy proste w Kotlinie są tworzone w ten sposób, a nie przez konstruktor.

# Elementy
Główną zasadą tutaj będzie wykorzystanie dobrze nazwanych metod zamiast konstruktora, który powinien być prywatny jak w przypadku Buildera. Popularne nazwy metod fabrycznych:
- `from` lub `valueOf` - metoda konwersji typu, przyjmuje jeden argument i zwraca obiekt o tej samej wartości
    ```kotlin
    val instant = Instant.now()
    val date = Date.from(instant)
  
    val prime = BigInteger.valueOf(Long.MAX_VALUE)
    ```
- `of` - metoda agregująca, przyjmująca wiele argumentów i zwracająca instancję obiektu zawierającą wszystkie podane argumenty
    ```kotlin
    val cutlery = EnumSet.of(Fork, Spoon, Knife)
    ```
- `instance`, `getInstance` lub `instanceOf` - metoda zwracająca instancję obiektu, często widziana w Singletonach. Może przyjmować argumenty i wtedy zwracać tę samą instancję dla podanych parametrów, ale nie jest to gwarantowane.
    ```kotlin
    val luke: StackWalker = StackWalker.getInstance(options)
    ```
- `create` lub `newInstance` - podobna do `getInstance` ale gwarantuje utworzenie nowej instancji obiektu
    ```kotlin
    val array = Array.newInstance(String::class.java, 10)
    ```
- `get<<Type>>` - podobnie do `getInstance` zwraca tę samą instancję obiektu, ale kiedy metoda fabryczna znajduje się w osobnej klasie. `<<Type>>` wskazuje na typ zwracanego obiektu
    ```kotlin
    val path = Path.of("path", "to/file/store")             // znana już metoda 'of()'
    val fileStore: FileStore = Files.getFileStore(path)     // <<Type>> to FileStore
    // w tym przypadku dla tego samego obiektu Path każdorazowo będzie tworzona nowa instancja FileStore
    ```
- `new<<Type>>` - analogicznie do poprzedniego przykładu, ale dla przypadku tworzenia nowych instancji
    ```kotlin
    val br: BufferedReader = Files.newBufferedReader(path)
    ```
  
Powyższe nazwy metod są dość ogólne, ale nic nie stoi na przeszkodzie, żeby stosować bardziej opisowe, np `forLanguageTag` czy `fromBits` jak w przykładach. Najważniejsze, żeby nazwa metody wytwórczej nawiązywała do już utartych schematów, bo nie ma innej możliwości, na pierwszy rzut oka, odróżnić jej od zwykłych metod.

## Companion Object

W Javie takie metody musiały być statycznymi elementami klas, ale w Kotlinie mamy `companion object`. Pozwala on na wywołanie metod bez konieczności tworzenia instancji klasy (choć sam `object` jest instancją). Zakotwiczeni w Javie programiści często traktują go jak kontener do przechowywania stałych i metod, które w Javie byłyby oznaczone jako `static`, ale ma on sporo ciekawych właściwości:
- `companion object` jest szczególnym przypadkiem zagnieżdżonego `object`-u czyli Singletona.
- Jego pola i metody zachowują się podobnie to `static` odpowiedników w Javie, jednak nadal są częścią instancji obiektu ze wszystkimi tego konsekwencjami (np. modyfikatory dostępu).
- Może implementować interface oraz rozszerzać klasę.
- Klasa posiadająca `companion object` ma dostęp do jego pól.
- Interfejsy też mogą mieć `companion object`.
- Nie jest dziedziczony, tzn subklasy nie mają dostępu do obiektu rodzica.
- Jest niedostępny z poziomu instancji obiektu, podobnie jak zagnieżdżony `object`.
- Może mieć nazwę, ale nie musi — wtedy można się do niego odnieść przez użycie nazwy `Companion`.
- W klasie może być tylko 1 `companion object`, ale dowolna ilość zwykłych `object`-ów.
- Podobnie jak zagnieżdżony `object` nie ma dostępu do pól i metod obiektu, w przeciwieństwie do `inner` klas.
- Nazwa klasy wskazuje już na `companion object`, nie trzeba wywoływać jego metod przez `<<Obiekt>>.Companion.<<metoda()>>`.

```kotlin
// klasa abstrakcyjna po której dziedziczy companion object w klasie InterestingObject
abstract class Printer { 

    abstract fun getName(): String

    fun print() {
        val name = getName()
        println(name)
    }
}
// prywatny konstruktor nie pozwala tworzyć instancji w tradycyjny sposób
class InterestingObject private constructor() { 

    object Factory : Printer() { // zwykły zagnieżdżony obiekt
        @JvmStatic // generowanie prawdziwie statycznych metod, potrzebne jeśli kod jest używany w Javie
        fun create() = InterestingObject()

        // nadpisana metoda może być @JvmStatic tylko w companion object
        override fun getName(): String {
            return "Interesting Object"
        }
    }

    companion object CompanionFactory : Printer() { // nazwany companion object
        private val secret = "No one can know this" // prywatne pole companion object, niedostępne poza klasą
        val publicInfo = "***** ***" // dostępne publicznie pole
        
        fun create() = InterestingObject()
        @JvmStatic 
        override fun getName(): String {
            return "Interesting Object from Companion"
        }
    }
    // metoda instancji ma dostęp do prywatnego pola companion object
    fun getSecret() = secret
}

val obj0 = InterestingObject() // błąd, prywatny konstruktor uniemożliwia utworzenie instancji w ten sposób
val obj1 = InterestingObject.create() // metoda z CompanionFactory czyli companion object
val obj2 = InterestingObject.Factory.create() // metoda z obiektu Factory
val obj3 = InterestingObject.CompanionFactory.create() // znów metoda z companion object, ze zbędnym podaniem jego nazwy

val secret = InterestingObject.CompanionFactory.secret // błąd, brak dostępu do pola w companion object
val secret2 = InterestingObject.create().getSecret() // ale przez instancję już się da do niego dostać
val publicInfo = InterestingObject.publicInfo // to pole jest dostępne w companion object

// wywołanie metody z klasy abstrakcyjnej
val print1 = InterestingObject.print()
val print2 = InterestingObject.Factory.print()
val print3 = InterestingObject.CompanionFactory.print()

// błąd, zagnieżdżone obiekty są niedostępne z poziomu instancji obiektu
InterestingObject.create().Factory.getName()
InterestingObject.create().CompanionFactory.getName()
```

Poniższy przykład przedstawia zwiększenie licznika `counter` za każdym razem, kiedy utworzymy instancję klasy `CountMe`. Jak pisałem wcześniej klasa `ReallyCountMe` nie ma dostępu do `companion object` rodzica. Jednak podczas tworzenia jej instancji licznik i tak jest zwiększany. Dzieje się tak, ponieważ zawsze jest wywoływany konstruktor i blok `init` rodzica, a pole `counter` należy do Singletona - `companion object`, więc za każdym razem jest zwiększana ta sama zmienna.

```kotlin
open class CountMe(val objectName: String) {
    init {
        count++  // zwiększanie licznika w companion object
    }
    companion object {
        var count = 0 // wszystkie instancje CountMe mają dostęp do tego samego pola
        fun printCount() = println("count = $count")
    }   
}

class ReallyCountMe(private val name: String): CountMe(name)

// użycie
CountMe.printCount() // wypisuje 0
CountMe("1")
CountMe("2")
CountMe("3")
CountMe.printCount() // wypisuje 3

ReallyCountMe("1")
ReallyCountMe("2")
ReallyCountMe("3")

CountMe.printCount() // wypisuje 6
```
Każdy obiekt `CountMe` w środku ma referencję do tej samej (jedynej) instancji `companion object`.

Inny przykład z biblioteki `Fuel` (prosty klient HTTP), gdzie konkretne podklasy `Result` tworzone są przez `companion object`:
```kotlin
sealed class Result<out V : Any?, out E : Exception> {
...
companion object {
        // Factory methods
        fun <E : Exception> error(ex: E) = Failure(ex)

        fun <V : Any?> success(v: V) = Success(v)

        fun <V : Any?> of(value: V?, fail: (() -> Exception) = { Exception() }): Result<V, Exception> =
                value?.let { success(it) } ?: error(fail())

        fun <V : Any?, E: Exception> of(f: () -> V): Result<V, E> = try {
            success(f())
        } catch (ex: Exception) {
            error(ex as E)
        }
    }
```

### Companion Object Extensions
Jeśli nie ma możliwości modyfikacji `companion object`, bo obiekt np. pochodzi z obcej biblioteki, można wykorzystać `extension functions` dla `companion object` klasy (lub interfejsu) - o ile został do niej dodany.

```kotlin
// --- zewnętrzny moduł, np. biblioteka ---
class ExternalClass {
    companion object
}

class ExternalClassWithoutCompanion

interface Car{
    companion object
}

// --- wewnętrzny moduł ---
class Ferrari: Car // wewnętrzna klasa implementująca interfejs z biblioteki
fun Car.Companion.getFerrari():Ferrari = Ferrari() // extension method tworząca instancję wewnętrznej klasy
// użycie
val ferrari = Car.getFerrari()

// inne przykłady extension methods
fun ExternalClass.Companion.newMethod() = "Wow! Extending class companion"
fun Car.Companion.newMethod() = "Wow! Extending interface companion"
fun ExternalClassWithoutCompanion.Companion.newMethod() = "Wow!" // błąd, ta klasa nie ma companion object
fun ExternalClassWithoutCompanion.newMethod() = "Wow!" // extension method dla instancji

// użycie
ExternalClass.newMethod() // nowa metoda companion object ExternalClass
ExternalClassWithoutCompanion.newMethod() // błąd, newMethod() rozszerza instancję
ExternalClassWithoutCompanion().newMethod() // OK, extension method dla instancji
```

I znowu korzystając z `Fuel`, którego klasa `Result` posiada `companion object`. Sama w sobie jest `sealed` więc nie można z niej dziedziczyć poza jej modułem.
```kotlin
fun Result.Companion.awersomeNewMethod(){
    println("This wasn't originally here")
}

//użycie
Result.awersomeNewMethod()
```

Warto korzystać z `companion object` i w publicznie udostępnianych interfejsach dodawać nawet jeśli ma być pusty. Ułatwia to późniejszym klientom rozszerzanie ich możliwości bez kombinowania z wrapperami czy dziedziczeniem.

# Alternatywa - Top-level functions

Innym ciekawym sposobem na tworzenie obiektów są `top-level functons`, znane na przykład z:
```kotlin
val intList = listOf(1, 2, 3)
val someSet = setOf("1", "2", "3")
val someMap = mapOf(1 to "1", 2 to "2")
```
Są szczególnie użyteczne do prostych, ale często użwanych obiektów jak lista czy mapa. Jak sama nazwa może sugerować, są to funkcje znajdujące się poza (ponad?) klasami i z tego powodu są dostępne publicznie. Należy więc mieć na uwadze przypadkowe przesłonięcia tych funkcji, np.:
```kotlin
// dodając w projekcie taką funkcję
fun listOf(vararg item: Int) = item.asList()

// przesłaniamy funkcję z kotlin.collections, w zasadzie bez śladu w samym kodzie
val intList = listOf(1, 2, 3)   // nasza funkcja
val intList2 = listOf("1", "2") // funkcja z kotlin.collections
```
Niekoniecznie jest to złe, ale może zaskoczyć. Dlatego lepiej unikać zbyt generycznych nazw i nie nadużywać samych `top-level functions`, żeby też nie zaśmiecać podpowiedzi z IDE.

# Podsumowanie
Kotlin ze swoim cukrem syntaktycznym pozwala na sporo przyjemniejsze pisanie kodu niż Java, ale mimo tego stosowanie metod tworzących instancje zamiast konstruktorów jest często dobrym pomysłem. Statyczne metody wytwórcze są stosowane zarówno w bibliotece standardowej Kotlina, jak i przez twórców bibliotek.

W tym wpisie chciałem też pokazać jak można ciekawie wykorzystać `companion object` który **nie jest tylko kontenerem na stałe**, ani nie symuluje słówka `static` z Javy. Twórcy Kotlina dość szeroko z niego korzystają np. w Coroutines.

## Zalety
- **metody mają opisowe nazwy**
  
  Niektóre języki pozwalają nazywać konstruktory, niestety nie Kotlin i Java. Statyczne Metody Wytwórcze to sposób na zapewnienie sobie takiej funkcjonalności. 
  
- **metody mogą przyjmować takie same parametry**

  Jeśli mają różne nazwy, to lista argumentów może być taka sama. Nie da się tego osiągnąć gołymi konstruktorami. Nazwane argumenty i wartości domyślne trochę ratują jednak sytuację.

- **nie zawsze trzeba tworzyć nowy obiekt**

  Konstruktor zawsze tworzy nową instancję, ale metoda może najpierw zajrzeć do cache.

- **możliwość zwracania dowolnego podtypu**

  Nawet jeśli ten typ nie jest publiczny. Można utworzyć `companion object` wewnątrz interface i zwracać instancje wewnętrznych typów które go implementują, ale dla klienta dostępny będzie tylko publiczny interface.  

## Wady
- **brak możliwości dziedziczenia po klasach, które posiadają tylko prywatny konstruktor**

  Ale zawsze można to traktować jako stanowcze zachęcenie do kompozycji zamiast dziedziczenia :)

- **nie da się od razu odróżnić statycznych metod wytwórczych od innych metod**

  Użycie konstruktora jest wyraźnie widocznie w kodzie, ale metody wytwórcze wyglądają jak wszystkie inne metody. Automatyczne generatory dokumentacji również nie będą jakoś szczególnie oznaczać tych metod jako sposobów na utworzenie instancji. Pomóc może korzystanie z utartego nazewnictwa metod wytwórczych, wymienionego [wyżej](#elementy).