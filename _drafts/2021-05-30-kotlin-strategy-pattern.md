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
- Template Method
- behavioral design pattern

categories:
- Design Patterns

image: /assets/posts/pizza.jpg

---

# Przeznaczenie
Wzorzec projektowy Strategia określa rodzinę algorytmów i umożliwia zamienne ich stosowanie. Jest to w pewnym sensie rozwinięcie wzorca [Template Method]. Strategie nie dziedziczą po żadnej konkretnej klasie, a jedynie implementują wspólny interfejs. Pozwala to na łatwe kapsułkowanie kodu i elastyczne wymienianie algorytmów bez narzutu dziedziczenia.

Strategia przedkłada kompozycję nad dziedziczenie, odwrotnie do [Template Method].

Przez `algorytm` będę tutaj rozumiał jakąkolwiek logikę, czy to sortowanie, czy wyszukiwanie, czy obliczanie jakiejś wartości na podstawie jakichś danych. Nie ma to znaczenia.

## Problem
Przykładowym problemem, który pomaga rozwiązać Strategia, może być sposób obliczania ceny z uwzględnieniem promocji:
```kotlin
data class Item(val name: String, val price: Double) // przedmiot na rachunku

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
Mamy 3 promocje z czego jedna `NoPromotion` nic nie daje. Implementacja sposobu obliczania promocji jest w klasie `Bill` - czyli klasa oprócz zbierania produktów oblicza też cenę, nie mamy tutaj zachowanego `Single Responsibility Principle`.

Jeśli pojawi się nowa promocja wystarczy dodać `enum` i jego implementację. IDE będzie krzyczeć, jeśli tego nie zrobimy.

Nie wygląda to źle, poza brakiem SRP. Ale załóżmym, że oprócz standardowego paragonu chcemy mieć możliwość wystawienia faktury, uwzględniającej te same promocje.
```kotlin
class Invoice {
	...
}
```
Można skopiować kod obliczający wartość promocji, albo sztucznie wyciągnąć jakąś abstrakcyjną klasę z implementacją promocji, po której dziedziczyły by `Invoice` i `Bill`. A wtedy pojawi się jeszcze jakaś klasa zupełnie niepasująca do tej hierarchii, która będzie potrzebować implementacji naliczania promocji... 

# Implementacja
Na ratunek przychodzi Strategia. Pozwala łatwo zawrzeć sposób obliczania promocji w osobnych klasach ze wspólnym interfejsem, w taki sposób, że klienty nie muszą nawet wiedzieć`jaka konkretnie promocja została użyta.

## Abstrakcyjna
Na początek ogólna implementacja wzorca:
```kotlin
class Context(private val strategy: Strategy) { // klient strategii
    fun useStrategy() = strategy.use() // użycie ogólnego interfejsu wstrzykniętej strategii
}

interface Strategy { // użycie interfejsu zamiast klasy jest bardzo istotne,
    // klasa abstrakcyjna lub nawet konkretna ograniczyłaby możliwość stosowania strategii tylko do klas pochodnych
    fun use() // Strategie mają zwykle 1 metodę publiczną
}

class StrategyA : Strategy { // pierwsza strategia
    override fun use() { // klasy z 1 metodą enkapsulujące jakąś logikę nazywa się 'obiektami funkcyjnymi'
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
Mamy tu `Context` czyli klienta korzystającego ze Strategii. Zna tylko interfejs, a nie konkretne klasy. Pozwala to łatwo rozszerzać rodzinę strategii o nowe, bez konieczności zmian w klientach. Z racji użycia interfejsu a nie klasy `Strategy`, konkretne Strategie są ze sobą luźno powiązane, jednocześnie gwarantując klientom wspólne API.

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

object ChristmasPromotion : Promotion { // singleton, bo ta konkretna strategia nie potrzebuje przechowywać stanu, choć może
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
    fun calculateFinalPrice(promotion: Promotion): Double {
        println("applying ${promotion.name}")
        val initialSum = items.sumOf { it.price }
        return promotion.calculate(initialSum) // to promocja oblicza cenę
    }
}
```
Może nie jest to najlepszy przykład, bo nadal nie mamy SRP — Bill nadal oblicza końcową kwotę, ale teraz przynajmniej deleguje to do obiektu `Promotion`.
Dodanie nowego rodzaju promocji nie ma najmniejszego znaczenia dla klasy `Bill`. Tych samych klas promocji można użyć w klasie `Invoice` albo dowolnej innej, która musi je uwzględniać.

Znacznie łatwiej testuje się logikę zawartą w osobnych klasach, niż w początkowym przykładzie z instrukcją `when`. Dodanie kolejnej promocji nie spowoduje konieczności poprawienia już istniejących testów, a jedynie dodania nowych dla nowej klasy.
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
Strategie mogą być przekazane jako parametr w metodzie, gdzie mają być użyte, ale mogą też zostać wstrzyknięte w konstruktorze obiektu. Poniżej przykład z wartościami domyślnymi (chyba najprostszy rodzaj [Buildera] w Kotlinie), co pozwala na nadpisanie tylko tych strategii, które faktycznie mają być inne niż domyślne.
```kotlin
data class Bill(
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
Równie dobrze można pokusić się o wykorzystanie [Factory]
```kotlin
object BillFactory {
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
```
Naliczanie poszczególnych składników ostatecznej kwoty musi odbywać się w ustalonej kolejności, nie możemy nalicząć promocji od podatku itd.
```kotlin
fun calculateFinalPrice(): Double {
	val initialSum = items.sumOf { it.price }
	return initialSum.run {
		promotion.applyPromotion(this)
	}.run {
		clientPolicy.applyPolicy(this)
	}.run {
		tax.applyTaxes(this)
	}
}
```
W rzeczywistości różne podatki dotyczyłyby raczej konkretnych produktów niż całego rachunku. Zakładając, że prawo podatkowe też się zmienia, użycie strategii `Tax` pozwala na szybkie reagowanie na nowe przepisy bez konieczności poprawiania klientów strategii.

# Podsumowanie
Strategia tworzy rodzinę algorytmów, zamykając różniącą się logikę w osobnych klasach, jednocześnie ukrywając ją przed klientami za wspólnym interfejsem. Umożliwia wymienne stosowanie implementacji. Użycie strategii znacząco upraszcza kod klientów, pozwala uniknąć duplikacji kodu i instrukcji warunkowych. Znacząco ułatwia testowanie — oddzielając testowanie klienta od algorytmów strategii.
Wzorzec ten powinien być stosowany dość często, nawet jeśli początkowo cała "rodzina" strategii będzie składać się z 1 klasy. Zalety płynące z enkapsulacji przewyższają wady dodania interfejsu i nowej klasy. Często prędzej niż później okazuje się, że dany algorytm trzeba wykorzystać gdzieś jeszcze, albo pojawia się potrzeba dodania kolejnego.

## Zalety
- *enkapsulacja algorytmów* - cały algorytm zamknięty w osobnej klasie, gotowej do stosowania w dowolnym miejscy systemu.
- *kompozycja zamiast dziedziczenia* — brak ścisłego powiązania między algorytmem a klientem.
- *polityka anty-IF* - to strategia wie, jak ma przetwarzać dane, klienty jej tylko używają, więc znikają instrukcje warunkowe.
- *wymienność implementacji* - różne implementacje np. sortowania mogą lepiej sprawdzać się w określonych przypadkach. Strategia pozwala na szybkie dostarczenie "akceptowalnego" algorytmu, a następnie poprawienie go, bez potrzeby zmiany klienta. 
- *łatwość testowania* - niezależne testowanie klienta i strategii. Zmiany strategii nie wymuszają poprawiania testów klienta.

## Wady
- *zwiększenie liczby obiektów* - użycie `object` czyli Singletona pozwala ominąć ten problem
- *może być użyta nadmiarowo* - trochę na siłę, ale jeśli **absolutnie** nie ma szans na to, że dany algorytm będzie użyty gdziekolwiek, albo pojawi się potrzeba alternatywnej wersji, to dodawanie strategii może być niepotrzebne... ale i tak ułatwi testowanie.
---
[^effective_java]:["Java - efektywne programowanie"](https://books.google.pl/books/about/Effective_Java.html?id=ka2VUBqHiWkC&redir_esc=y) 
[^fowler]:[Architektura systemów zarządzania przedsiębiorstwem. Wzorce projektowe](https://books.google.pl/books?id=FyWZt5DdvFkC&q=active+record&pg=PT187&redir_esc=y)
[^active_record]:[Active Record](https://en.wikipedia.org/wiki/Active_record_pattern)
[^ruby_active_record]:[Active Record w Ruby](https://guides.rubyonrails.org/active_record_callbacks.html)
[^active_record_antipatern]:[antywzorzec](https://www.mehdi-khalili.com/orm-anti-patterns-part-1-active-record)