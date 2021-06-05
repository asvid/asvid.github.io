---
layout: post
title: "Wzorzec Strategia w Kotlinie"
date:  "2021-06-05 11:43"
description: "
Strategia tworzy rodzinę algorytmów, zamykając różniącą się logikę w osobnych klasach, jednocześnie ukrywając ją przed klientami za wspólnym interfejsem. Umożliwia wymienne stosowanie implementacji. Użycie strategii znacząco upraszcza kod klientów, pozwala uniknąć duplikacji kodu i instrukcji warunkowych. Znacząco ułatwia testowanie — oddzielając testowanie klienta od algorytmów strategii.
"
permalink: "kotlin-strategy-pattern"
comments: true
toc: true
tags:
- design patterns
- Kotlin
- Strategy Pattern
- behavioral design pattern

categories:
- Design Patterns

image: /assets/posts/strategy.jpg

---

# Przeznaczenie
Wzorzec projektowy `Strategia` określa rodzinę algorytmów i umożliwia zamienne ich stosowanie. Przez `algorytm` będę tutaj rozumiał jakąkolwiek logikę, czy to sortowanie, czy wyszukiwanie, czy obliczanie jakiejś wartości na podstawie danych. Nie ma to znaczenia. Jest to w pewnym sensie rozwinięcie wzorca [Metoda Szablonowa](https://asvid.github.io/pl/kotlin-template-method), ale odwrotnie do niego `Strategia` przedkłada kompozycję nad dziedziczenie. Strategie nie dziedziczą po żadnej konkretnej klasie, a jedynie implementują wspólny interfejs. Pozwala to na łatwe kapsułkowanie kodu i wymienianie algorytmów bez narzutu dziedziczenia.

## Problem
Przykładowym problemem, który pomaga rozwiązać `Strategia`, może być sposób obliczania ceny z uwzględnieniem promocji:
```kotlin
data class Item(val name: String, val price: Double) // produkt na rachunku

enum class Promotion { // enum z promocjami
    NoPromotion, SpecialPromotion, ChristmasPromotion
}

class Bill { // rachunek
    private val items = mutableListOf<Item>() // lista przedmiotów na rachunku
    fun addItem(item: Item): Bill {
        items.add(item)
        return this
    }

	// metoda obliczająca cenę końcową na podstawie listy produktów i wybranej promocji
    fun calculateFinalPrice(promotion: Promotion): Double {
        val initialSum = items.sumOf { it.price }
        return when (promotion) { // sprawdzenie promocji i implementacja sposobu obliczania ceny
            Promotion.NoPromotion -> initialSum
            Promotion.SpecialPromotion -> when {
                initialSum > 20 -> initialSum * 0.95
                initialSum > 30 -> initialSum * 0.85
                initialSum > 40 -> initialSum * 0.75
                else -> initialSum
            }
            Promotion.ChristmasPromotion -> initialSum * 0.80
        }
    }
}
```
> Wiem, że `Double` nie jest najlepszym typem do operacji na pieniądzach, ale łatwiej się nim operuje na potrzeby przykładów z tego posta.

Mamy 3 promocje, z czego jedna `NoPromotion` nic nie zmienia. Implementacja sposobu obliczania promocji jest w klasie `Bill` - czyli klasa oprócz zbierania produktów oblicza też cenę, nie mamy tutaj zachowanego `Single Responsibility Principle`.

Jeśli pojawi się nowa promocja wystarczy dodać `enum` i jego implementację. IDE będzie informować o braku obsłużenia przypadku, jeśli tego nie zrobimy.

Nie wygląda to jakoś bardzo źle, poza brakiem SRP. Ale załóżmy, że oprócz standardowego paragonu chcemy mieć możliwość wystawienia faktury, uwzględniającej te same promocje.
```kotlin
class Invoice {
	...
}
```
Możnaby skopiować kod obliczający wartość promocji, albo sztucznie wyciągnąć jakąś abstrakcyjną klasę z implementacją promocji, po której dziedziczyłyby `Invoice` i `Bill`. O ile oczywiście nie dziedziczą już po innej klasie. 

A wtedy pojawi się jeszcze jakaś klasa zupełnie niepasująca do tej hierarchii, która będzie potrzebować implementacji naliczania promocji... 

# Implementacja
I wtedy wchodzi `Strategia`, cała na biało. Pozwala łatwo przenieść sposób obliczania poszczególnych promocji do osobnych klas ze wspólnym interfejsem, w taki sposób, że klienty nie muszą nawet wiedzieć`jaka konkretnie promocja została użyta.

## Abstrakcyjna
Na początek ogólna implementacja wzorca:
```kotlin
class Context(private val strategy: Strategy) { // klient strategii, strategia wstrzyknięta w konstruktorze
    fun useStrategy() = strategy.use() // użycie ogólnego interfejsu wstrzykniętej strategii
}

interface Strategy { // użycie interfejsu zamiast klasy jest bardzo istotne,
    // klasa abstrakcyjna lub nawet konkretna ograniczyłaby możliwość stosowania strategii tylko do klas pochodnych
    fun use() // Strategie mają zwykle 1 metodę publiczną
}

class StrategyA : Strategy { // pierwsza strategia
    override fun use() { // konkretna implementacja algorytmu
        println("using strategy A")
    }
}

class StrategyB : Strategy { // druga strategia
    override fun use() {
        println("using strategy B")
    }
}

fun main() {
    // użycie obu strategii wygląda identycznie
	// dla klienta strategie są transparentne, nie ma pojęcia której używa
    val contextA = Context(StrategyA())
    contextA.useStrategy()
    val contextB = Context(StrategyB())
    contextB.useStrategy()
}
```
Mamy tu `Context` czyli klienta korzystającego ze Strategii. Zna tylko interfejs, a nie konkretne klasy. Pozwala to łatwo rozszerzać rodzinę strategii o nowe, bez konieczności zmian w klientach. Z racji użycia interfejsu, a nie klasy `Strategy`, konkretne Strategie są ze sobą luźno powiązane, jednocześnie gwarantując klientom wspólne API.

Symbolicznie można to przedstawić tak:

{% plantuml %}
@startuml

interface Strategy{
+ use()
  }

class StrategyA implements Strategy{
+ use()
  }
class StrategyB implements Strategy{
+ use()
  }

class Context{
	- strategy: Strategy
	+ useStrategy()
}

Context::strategy -> Strategy

@enduml
{% endplantuml %}

## Rozwiązanie problemu
Promocje można zamknąć w osobnych klasach:
```kotlin
interface Promotion {
    fun calculate(sum: Double): Double
    val name: String
}

// singleton, bo ta konkretna strategia nie potrzebuje przechowywać stanu, choć może
object ChristmasPromotion : Promotion {
    override val name = "Christmas Promotion"
    override fun calculate(sum: Double): Double {
        return sum * 0.8
    }
}

// w sumie jest to NullObject, czyli szczególny przypadek Strategii nie wykonujący żadnych działań
object NoPromotion : Promotion { 
    override val name = "No Promotion"
    override fun calculate(sum: Double): Double {
        return sum
    }
}

object SpecialPromotion : Promotion {
    override val name = "Special Promotion"
    override fun calculate(sum: Double): Double {
        return when {
            sum > 20 -> sum * 0.95
            sum > 30 -> sum * 0.85
            sum > 40 -> sum * 0.75
            else -> sum
        }
    }
}
```
Mając tak zaimplementowane promocje, znacząco upraszcza nam się klasa `Bill`:
```kotlin
class Bill {
    ...
	// Strategie mogą być przekazane w konstruktorze Context-u, lub w metodzie która z nich korzysta
    fun calculateFinalPrice(promotion: Promotion): Double {
        println("applying ${promotion.name}")
        val initialSum = items.sumOf { it.price }
        return promotion.calculate(initialSum) // to promocja oblicza cenę
    }
}
```
Może nie jest to najlepszy przykład, bo nadal nie mamy SRP — `Bill` ciągle oblicza końcową kwotę, ale teraz przynajmniej deleguje to do obiektu `Promotion`.
Dodanie nowego rodzaju promocji nie ma najmniejszego znaczenia dla klasy `Bill`. Tych samych klas promocji można użyć w klasie `Invoice` albo dowolnej innej, która musi je uwzględniać.

Znacznie łatwiej testuje się logikę zawartą w osobnych klasach, niż w początkowym przykładzie z instrukcją warunkową `when`. Dodanie kolejnej promocji nie spowoduje konieczności poprawienia już istniejących testów, a jedynie dodania nowych dla nowej klasy.
Aby nie popsuć sobie tego testowania, należy zwracać uwagę, żeby `Strategia` dostawała wszystkie potrzebne argumenty w swoich metodach lub konstruktorze, zamiast magicznie je wyciągać z jakiejś konfiguracji. Często `Strategia` nie potrzebuje też przechowywać swojego stanu, a jedynie wykonuje jakieś działania. Jest to jeden z niewielu przypadków gdzie użycie `Singletona` ma sens.

## Wiele strategii
Ok, z jedną strategią jest fajnie, ale nic nie stoi na przeszkodzie, żeby klasa `Bill` miała ich wiele rodzajów. Rachunek to nie tylko promocje, ale też podatki i może program lojalnościowy.
```kotlin
interface Tax {
    fun applyTaxes(sum: Double): Double
}

interface Promotion {
    fun applyPromotion(sum: Double): Double
}

interface ReturningClientPolicy {
    fun applyPolicy(sum: Double): Double
}
```
Strategie mogą być przekazane jako parametr w metodzie, gdzie mają być użyte, ale mogą też zostać wstrzyknięte w konstruktorze obiektu. Poniżej przykład z wartościami domyślnymi (chyba najprostszy rodzaj [Buildera](https://asvid.github.io/pl/kotlin-builder-pattern) w Kotlinie), co pozwala na nadpisanie tylko tych strategii, które faktycznie mają być inne niż domyślne.
```kotlin
class Bill(
    val tax: Tax = DefaultTax,
    val promotion: Promotion = NoPromotion,
    val clientPolicy: ReturningClientPolicy = NewClient
)

val newClientAnarchist = Bill(
        tax = NoTax,
        clientPolicy = NewClient
)

val returningClientWithSpecialPromotionBill = Bill(
        clientPolicy = ReturningClient,
        promotion = SpecialPromotion
)
```
Równie dobrze można pokusić się o wykorzystanie [Static Factory](https://asvid.github.io/pl/kotlin-static-factory-methods)
```kotlin
object Bill {
    ...
    companion object Factory{
        val defaultTax = DefaultTax
        val defaultPromotion = NoPromotion
        val defaultClientPolicy = NewClient

        fun returningClient(): Bill = Bill(
                defaultTax, defaultPromotion, ReturningClient
        )

        fun returningClientWithSpecialPromotion() = Bill(
                defaultTax, SpecialPromotion, ReturningClient
        )

        fun newClientAnarchist() = Bill(
                NoTax, defaultPromotion, NewClient
        )
    }
}
```
**Jednak wtedy klasa `Bill` staje się świadoma przynajmniej części konkretnych implementacji `Strategy`.**

Naliczanie poszczególnych składników ostatecznej kwoty musi odbywać się w ustalonej kolejności, nie możemy naliczać promocji od podatku itd. Podobnie jak w [Metodzie Szablonowej](https://asvid.github.io/pl/kotlin-template-method) klasa `Bill` jest odpowiedzialna za właściwą kolejność kroktów, ale wzorzec Strategia pozwala implementację tych kroków podmieniać.
```kotlin
fun calculateFinalPrice(): Double {
	val initialSum = items.sumOf { it.price }
	return initialSum.run { 
		promotion.applyPromotion(this) // `this` jest sumą początkową
	}.run {
		clientPolicy.applyPolicy(this) // a tutaj sumą po uwzględnieniu promocji
	}.run {
		tax.applyTaxes(this) // i programu lojalnościowego
	} // zwracana wartość po uwzględnieniu wszystkich modyfikatorów
}
```
W rzeczywistości różne podatki dotyczyłyby raczej konkretnych typów produktów niż całego rachunku. Zakładając jednak, że prawo podatkowe cały czas się zmienia, użycie strategii `Tax` pozwala na szybkie reagowanie na nowe przepisy bez konieczności poprawiania klientów strategii.

Przyznam, że średnio podoba mi się takie kolejkowanie metodami `run()`. Na szczęście można to poprawić korzystając z funkcji rozszerzeń:
```kotlin
fun Double.applyPromotion(promotion: Promotion): Double {
    return promotion.applyPromotion(this)
}

fun Double.applyPolicy(policy: ReturningClientPolicy): Double {
    return policy.applyPolicy(this)
}

fun Double.applyTaxes(tax: Tax): Double {
    return tax.applyTaxes(this)
}
```
I wtedy mamy:
```kotlin
fun calculateFinalPrice(): Double {
    val initialSum = items.sumOf { it.price }
    return initialSum
            .applyPromotion(promotion)
            .applyPolicy(clientPolicy)
            .applyTaxes(tax)
}
```

### Invoke
Z racji, że Strategie mają raczej jedną publiczną metodę można użyć operatora `invoke()`. Do tego obiekty anonimowe zamiast domyślnych implementacji Strategii, żeby mnie mnożyć `NullObject`:
```kotlin
interface Tax {
    // użycie tej metody pozwala użyć obiektu jak funkcji
    operator fun invoke(sum: Double): Double
}
...
class Bill(
        val tax: Tax = object : Tax {
            override fun invoke(sum: Double): Double {
                return sum
            }
        },
		...
) {
    ...
    fun calculateFinalPrice(): Double {
        val initialSum = items.sumOf { it.price }
        return initialSum
                .applyPromotion(promotion)
                .applyPolicy(clientPolicy)
                .applyTaxes(tax)
    }
}

// użycie obiektów anonimowych pozwala tworzyć w locie nowe strategie
// jednak brak przypisania ich do konkretnego obiektu zabija sporą część korzyści wzorca Strategia
val customBill = Bill(promotion = object : Promotion {
    override fun applyPromotion(sum: Double): Double {
        return sum * 0.123512
    }
})
```

# Podsumowanie
`Strategia` tworzy rodzinę algorytmów, zamykając różniącą się logikę w osobnych klasach, jednocześnie ukrywając ją przed klientami za interfejsem. Umożliwia wymienne stosowanie implementacji. Użycie strategii upraszcza kod klientów, pozwala uniknąć duplikacji kodu i instrukcji warunkowych. Znacząco ułatwia testowanie — oddzielając testowanie klienta od algorytmów strategii.

Wzorzec ten powinien być stosowany dość często, nawet jeśli początkowo cała "rodzina" strategii będzie składać się z 1 klasy. Zalety płynące z enkapsulacji przewyższają wady dodania interfejsu i nowej klasy. Często prędzej niż później okazuje się, że dany algorytm trzeba wykorzystać gdzieś jeszcze, albo pojawia się potrzeba dodania kolejnego. Nie należy jednak popadać w skrajność, znajdą się miejsca, gdzie ewidentnie tworzenie Strategii mija się z celem, bo np. algorytm to 1 linia kodu użyta w 1 miejscu bez perspektyw na rozprzestrzenienie się.

`Kotlin` daje ciekawe możliwości wykorzystania Strategii, dzięki nazwanym argumentom, użyciu operatora `invoke()` i `extension functions`. Należy jednak zwracać uwagę na to, czy cukier syntaktyczny nie utrudnia nam testowania lub wykorzystania kodu w innych miejscach.

## Zalety
- **enkapsulacja algorytmów** - cały algorytm zamknięty w osobnej klasie, gotowej do stosowania w dowolnym miejscy systemu.
- **kompozycja zamiast dziedziczenia** — brak ścisłego powiązania między algorytmem a klientem.
- **polityka anty-IF** - to strategia wie, jak ma przetwarzać dane, klienty jej tylko używają, więc znikają instrukcje warunkowe.
- **wymienność implementacji** - różne implementacje np. sortowania mogą lepiej sprawdzać się w określonych przypadkach. Strategia pozwala na szybkie dostarczenie "akceptowalnego" algorytmu, a następnie poprawienie go, bez potrzeby zmiany klienta. 
- **łatwość testowania** - niezależne testowanie klienta i strategii. Zmiany strategii nie wymuszają poprawiania testów klienta.

## Wady
- **zwiększenie liczby obiektów** - użycie `object` czyli Singletona pozwala ominąć ten problem
- **może być użyta niepotrzebnie** - trochę na siłę, ale jeśli **absolutnie** nie ma szans na to, że dany algorytm będzie użyty gdziekolwiek, albo pojawi się potrzeba alternatywnej wersji, to dodawanie strategii może być niepotrzebne... ale i tak ułatwi testowanie.