---
layout: post
title: "Adapter w Kotlinie"
date:  "2021-06-15 11:43"
description: "
Adapter lub Wrapper pozwala "przetłumaczyć" jeden interfejs na inny, oczekiwany przez klienta. Jest to szczególnie przydatne, gdy adaptowany obiekt pochodzi z niezależnej biblioteki i nie chcemy uzależniać naszego systemu od jego interfejsu, tworząc tzw. `anticorruption layer`. Zmiany interfejsu obiektu wpłyną tylko na `Adapter` a nie na resztę kodu.
"
permalink: "kotlin-adapter-pattern"
comments: true
toc: true
tags:
- design patterns
- Kotlin
- Adapter Pattern
- structural design pattern

categories:
- Design Patterns

image: /assets/posts/facade.jpg

---

# Przeznaczenie
Jak sama nazwa sugeruje, wzorzec `Adapter` przekształca interfejs klasy na inny, wymagany przez klienta. Zastosowanie `Adaptera` pozwala na współdziałanie klas, które przez niezgodne interfejsy nie mogłyby tego robić. Innym określeniem na ten wzorzec jest `Wrapper`.

Adapter umożliwia "zmapowanie" obiektu adaptowanego (`Adaptee`) na oczekiwany (`Target`) przez klasę kliencką bez dokładania kolejnego poziomu dziedziczenia. Takie dziedziczenie nawet nie zawsze byłoby możliwe, jeśli `Target` byłby klasą, a nie interfejsem. Nowa klasa musiałaby być jednocześnie i `Target`, i `Adaptee`.

Zamiast tworzyć nową klasę `Adapter` można również dodać metodę w kliencie, która przyjmie obiekt z interfejsem `Adaptee`, ale to powoduje rozrost klasy o metody robiące to samo, tylko dla różnych argumentów. Klasa kliencka może też pochodzić z zewnątrz i wtedy nie będzie możliwości zmiany.
# Implementacja
`Adapter` to w zasadzie pojedyncza klasa, ale ważne jest zrozumienie jej otoczenia.

{% plantuml %}
@startuml
skinparam groupComposition 1
class Client{
- useTarget()
}
interface Target{
+ method()
}
class Adapter extends Target{
+ method()
- adaptee: Adaptee
}
interface Adaptee{
 + otherMethod() 
}

Adapter::adaptee -right-* Adaptee
Client-right->Target
Adapter::method..>Adaptee::otherMethod

@enduml
{% endplantuml %}

- **Client** - klasa korzystająca z obiektu `Target`
- **Target** - interfejs wymagany przez klasę `Client`
- **Adapter** - klasa z interfejsem `Target` adaptująca obiekt `Adaptee`, najczęściej obiekt adaptowany będzie przypisany do pola w tej klasie
- **Adaptee** - klasa adaptowana z niepasującym interfejsem, którą chcemy użyć razem z `Client`

## Abstrakcyjna
W kodzie powyższy diagram może wyglądać następująco:
```kotlin
class Client(private val target: Target) {
    private val argument = BigDecimal(10.0)

    fun doWork() {
        target.method(argument)
    }
}
// klient akceptuje tylko ten interfejs
interface Target {
    fun method(argument: BigDecimal): Double
}
// interfejs obiektu który chcemy użyć z klientem
interface Adaptee {
    fun originalMethod(argument: CustomArgument): CustomResult
}
// adapter implementujący Target i przyjmujący Adaptee w konstruktorze
class Adapter(private val adaptee: Adaptee) : Target {
    override fun method(argument: BigDecimal): Double {
        val stringArgument = argument.toCustomArgument()
		// wywołanie metody adaptowanej klasy
        return adaptee.originalMethod(stringArgument).toDouble()
    }
}

// użycie
fun main() {
    val target1: Target = TargetImpl()
    val adaptee: Adaptee = AdapteeImpl()
    val adapter: Target = Adapter(adaptee)

    val client1: Client = Client(target1)
    val client2: Client = Client(adaptee) // błąd! zły interfejs
    val client3: Client = Client(adapter) // użycie adaptera z obiektem Adaptee
}
```
Widać tutaj jak klasa `Adapter` używa obiektu `Adaptee` w ramach interfejsu `Target`. `Adapter` enkapsuluje logikę mapowania jednego interfejsu na drugi. Pozwala to uniknąć niepotrzebnego rozszerzania czy modyfikacji klienta lub `Adaptee` tylko pod konkretnego klienta.

Różne klienty mogą korzystać z różnych adapterów tej samej klasy `Adaptee`. Dzięki temu nie ma potrzeby dostosowywania `Adaptee` pod konkretnego klienta lub klika klientów. Typo spójrz na tego potworka:
```kotlin
// Adaptee implementuje interfejsy wymagane przez klienty
class Adaptee: Target1, Target2, Target3{
    fun originalMethod(argument: CustomArgument): CustomResult
    // metody z interfejsów klientów, gdzie Adaptee jest używany
    override fun target1Method()
    override fun target2Method()
    override fun target3Method()
}
```
A z Adapterami:
```kotlin
// niezmieniona klasa
class Adaptee{
    fun originalMethod(argument: CustomArgument): CustomResult
}
// Adaptery mogą znajdować się pakietowo bliżej Klienta niż adaptowanej klasy
class Adapter1(val adaptee: Adaptee) : Target1{
    override fun target1Method()
}
class Adapter2(val adaptee: Adaptee) : Target2{
    override fun target2Method()
}
class Adapter3(val adaptee: Adaptee) : Target3{
    override fun target3Method()
}
```

## Listy
Adapter dobrze sprawdzi się w przypadku listy elementów, na których chcemy wykonać operację, do której nie zostały stworzone.
```kotlin
// załóżmy że ta klasa pochodzi z zewnętrznej biblioteki
// `data class` nie może być rozszerzone
data class Item(val name: String)

// w naszym kodzie mamy interface
interface PrettyPrintableItem {
    fun prettyPrint(): String
}

// Adapter/Wrapper zapewniający funkcjonalność `PrettyPrintableItem` dla obiektu `Item`
class ItemAdapter(private val item: Item) : PrettyPrintableItem {
    override fun prettyPrint(): String {
        return "hello, my name is: ${item.name}"
    }
}
// użycie
fun main() {
	// lista elementów zwrócona przez bibliotekę, którą chcemy "ładnie wydrukować"
    val list: List<Item> = listOf(
            Item("Adam"),
            Item("Not Adam"),
            Item("Adam Maybe"),
            Item("Yes"),
    )

    list.forEach {
        val adapted = ItemAdapter(it)
        println(adapted.prettyPrint())
    }
}
```
Zastosowanie `Adaptera` w czysty sposób pozwala nam łączyć klasy, nad którymi nie mamy kontroli (pochodzą z zewnętrznych bibliotek), z naszym kodem o innych interfejsach. Dobrą praktyką jest nieużywanie, o ile to możliwe, interfejsów niezależnych zewnętrznych bibliotek (3rd party libraries) w całym systemie, a jedynie w punkcie styku biblioteki z naszym kodem. Zapewnia to wymienialność bibliotek, łatwiejszy update wersji oraz pozwala uchronić nasz system przed koniecznością zmian podyktowanych zmianami w niezależnym od nas oprogramowaniu.

Zdaję sobie sprawę, że nie zawsze się tak da, a czasami wręcz nie warto tworzyć pośrednie interface. Jednak głupio byłoby w długożyjącym projekcie, nad którym pracuje wiele zespołów, mieć problem — bo biblioteka od konwersji daty zmieniła interface z wersji na wersję.

## Duck Typing
Kaczko-typowanie, czyli jeśli coś kwacze jak kaczka, pływa jak kaczka i lata jak kaczka, to musi być kaczką. W przeciwieństwie do silnego typowania gdzie wiemy na 100%, że obiekt jest kaczką, bo dziedziczy po klasie `Duck`, tutaj interesuje nas faktyczne zachowanie obiektu. Kotlin jest silnie typowany i nie pozwala na wielodziedziczenie, ale extension functions umożliwiają "dopisanie" funkcjonalności do obiektu. Możemy więc mieć gatunkowo-płynnego psa:
```kotlin
class York: Dog{
    fun bark(){}
}

fun York.quack(){}
//
York().bark()
York().quack()
```
![York duck](assets/posts/york_duck.jpg){: .center-image }

Ten York nie wygląda na szczęśliwego. I zdecydowanie nie jest kaczką. Ale jeśli potrafi kwaczeć, a tylko tego potrzebujemy, to chyba wystarczy?

Wracając do przykładu z listą, zamiast tworzyć `Adapter` wystarczyłoby dodać extension function dla `Item`:
```kotlin
fun Item.prettyPrint(): String {
    return "hello, my name is: ${this.name}"
}

// Item potrafi się "ładnie wydrukować" bez dodatkowego Adaptera	
list.forEach {
	println(it.prettyPrint())
}
// wersja z Adapterem
list.forEach {
    val adapted = ItemAdapter(it)
    println(adapted.prettyPrint())
}
```
Extension functions ogólnie są super i w takim prostym przykładzie chyba sprawdzą się lepiej niż dodatkowa klasa `Adapter`. Można je dodawać także do klas, nad którymi nie mamy kontroli, np. pochodzących z bibliotek.

Problem pojawia się, kiedy zaczynamy tych metod do konkretnej klasy dodawać coraz więcej, w dodatku w różnych miejscach w kodzie. IDE świetnie podpowiada możliwe metody, łatwo można przejść do implementacji, ale w trakcie code review może być ciężko się połapać, skąd klasa ma daną metodę — bo nie ma jej w implementacji klasy. Może to też prowadzić do zaspamowania interfejsu klasy lub niejawnego przesłaniania metod.

```kotlin
interface Item

class FirstItem : Item
class AnotherItem : Item

// funkcja rozszerzająca ogólny interface `Item`
fun Item.prettyPrint(): String = "hello, I'm: $this"
// funkcja rozszerzająca konkretną klasę `AnotherItem`
fun AnotherItem.betterPrint(): String = "greetings, I'm: $this"
// funkcja przesłaniająca `prettyPrint` dodaną do interejsu `Item`
fun AnotherItem.prettyPrint(): String = "yo, I'm: $this"

fun main() {

    val item1: Item = FirstItem()
    val item2: Item = AnotherItem()
    val item3: AnotherItem = AnotherItem()

    item1.prettyPrint()
    item2.prettyPrint()
    item3.prettyPrint() // która metoda tutaj się wywoła?
    item3.betterPrint()
}
```
W tym przykładzie dodałem 3 funkcje rozszerzeń. Szczególnie ciekawy jest przypadek z `prettyPrint`, gdzie implementacja jest inna dla ogólnego interfejsu `Item` i dla konkretnej klasy `AnotherItem`. IDE ani kompilator nie widzą żadnego problemu, po prostu z typem `AnotherItem` zostanie użyta druga metoda. Ale `extension functions` można napisać gdziekolwiek w kodzie, nawet w bardzo odległych miejscach od miejsca zadeklarowania klasy. 

Wyobraźcie sobie sytuację, gdzie napisaliście do ogólnego interfejsu `extension functions` i korzystacie z niej przez długi czas bez problemu z różnymi typami implementującymi ten interfejs. W którymś momencie zupełnie inny zespół potrzebował `extension functions` dla konkretnego typu, więc ktoś napisał w dogodnym dla siebie miejscu metodę o takiej samej nazwie i sygnaturze, nadpisując metodę dla ogólnego interfejsu. Bez żadnego `override` :) testy mogą to wyłapać, ale nie muszą. Code Review robił pewnie ktoś z drugiego zespołu, więc dopóki się coś nie zacznie się zachowywać w niespodziewany sposób, prawdopodobnie nie dowiesz się o całej operacji. A potem szukanie przyczyny błędu może nie być najprzyjemniejsze...

Mamy tutaj kilka problemów:
- uzależnienie od `extension functions` dla interfejsu, którą można łatwo można nadpisać
- rozsiewanie `extension functions` po całym systemie
- code review ograniczone do członków 1 zespołu (to jest w ogóle temat na osobnego posta)

Zamknięcie całej "rozszerzonej" funkcjonalności w np. `ItemPrinter` pozwoliłoby uniknąć nieporozumień i ułatwić code review. W Git można łatwo sprawdzić, kto jest autorem lub modyfikował ostatnio tę klasę i również dodać do pull requesta. W przypadku `extension functions` taka opcja też istnieje, ale metody mogą być rozsiane po systemie, co utrudnia znalezienie autorów, a jeśli coś jest trudniejsze niż bezproblemowe, to nikt tego nie będzie robił. 

## Shapes
We wpisie o [Factory Method](https://asvid.github.io/pl/kotlin-factory-method) korzystałem z przykładu z figurami geometrycznymi, który tutaj też się fajnie nadaje.

Dla przypomnienia:
```kotlin
interface Shape {
    fun draw()
    fun createManipulator(): ShapeManipulator<out Shape>
}

interface ShapeManipulator<T : Shape> {
    fun drag()
    fun resize(scale: Float)
}

internal class Circle : Shape {
    override fun draw() {}
    override fun createManipulator() = CircleManipulator(this)
}

internal class CircleManipulator<T>(private val shape: T) : ShapeManipulator<Circle> {
    override fun drag() = println("CircleManipulator is manipulating circle $shape")
    override fun resize(scale: Float) = println("CircleManipulator is resizing circle $shape")
}
```
Mamy interfejs `Shape` który implementuje np. `Circle`. Do tego, każda figura ma swój `ShapeManipulator` czyli obiejkt, który wie jak modyfikować rozmiar, kształt itd. konkretnej figury.
Figury na ekranie wyświetla klasa `Window`
```kotlin
class Window() {
    fun drawShape(shape: Shape) {
		// magia
    }
}
```
I teraz chcemy móc wyświetlić tekst na ekranie, obok figur geometrycznych. Jednak klasa `TextView` nie jest figurą, ma inny interfejs i pochodzi np. z jakiejś biblioteki. Jest też na tyle skomplikowaną klasą, że nie ma szans na przepisanie jej z wykorzystaniem interfejsu `Shape`, ani na zmianę działania klasy `Window`.
```kotlin
class TextView {
    fun displayText() {}
    fun changeSize() {}
    fun changePosition() {}
}
```
Z pomocą przychodzi `Adapter`
```kotlin
class TextViewAdapter(val textView: TextView) : Shape {
    override fun draw() {
        textView.displayText()
    }
	
	// anonimowy obiekt zamiast osobnej klasy
    override fun createManipulator(): ShapeManipulator<out Shape> {
        return object : ShapeManipulator<Shape> {
            override fun drag() {
                textView.changePosition()
            }

            override fun resize(scale: Float) {
                textView.changeSize()
            }

        }
    }
}
```
Który dla `Window` zachowuje się jak każda inna figura
```kotlin
fun main() {
    val window = Window()
    val circle = Circle()
    window.drawShape(circle)

    val textView = TextView()
    window.drawShape(textView) // błąd! zły interfejs
    window.drawShape(TextViewAdapter(textView)) // użycie adaptera
}
```
Specjalnie pominąłem tutaj zwracane typy czy logikę wyświetlania elementów. Taki `Adapter` w prawdziwym projekcie byłby dużo bardziej skomplikowany.

# Nazewnictwo
Tutaj mam mieszane uczucia, czy dodawanie w nazwie `Adapter` jest konieczne. Z jednej strony to wyraźna informacja, że obiekt tylko mapuje jeden interfejs na inny. Z drugiej — ta informacja może nie być wcale potrzebna, klienty interesuje tylko interfejs. Czy `ItemAdapter` lub `ItemWrapper` mówi więcej od `ItemWithPrettyPrint`? Co więcej, jeśli dla `Item` stworzymy więcej adapterów dla różnych klientów, nazywanie ich `ItemForClient1Adapter` nie wygląda najlepiej. 

Dlatego skłaniam się raczej do nazywania adapterów od obiektu, który adaptują i implementowanego interfejsu.

# Podsumowanie
Adapter lub Wrapper pozwala "przetłumaczyć" jeden interfejs na inny, oczekiwany przez klienta. Jest to szczególnie przydatne, gdy adaptowany obiekt pochodzi z niezależnej biblioteki i nie chcemy uzależniać naszego systemu od jego interfejsu, tworząc tzw. `anticorruption layer`. Zmiany interfejsu obiektu wpłyną tylko na `Adapter` a nie na resztę kodu.

Kotlin pozwala w pewnym sensie, przez `extension functions`, zapewnić podobną do `Adaptera` funkcjonalność bez konieczności tworzenia całej klasy. Będzie to sensownie działać w przypadku, kiedy nie interesuje nas typ obiektu, a jego możliwości co jest często określane jako `Duck Typing`. Jednak `extension functions` mogą zaciemniać faktyczny interfejs klasy, przesłaniać się nawzajem i ogólnie wprowadzać chaos. Ograniczając ich zasięg, można sobie z tym poradzić, ale jeśli zaczyna ich przybywać dla konkretnej klasy, może warto wydzielić osobną klasę.

## Konsekwencje
- **pojedyncza odpowiedzialność klas** - nie trzeba zmieniać klasy adaptowanej pod konkretnego klienta, a jedynie dodać `Adapter` pod wymagany interfejs. Klasa adaptowana może się zmieniać niezależnie od klientów, a zadaniem `Adaptera` jest pogodzić te zmiany z interfejsem klienta.
- **anticorruption layer** - oddziela interfejs, nad którym nie mamy kontroli i "tłumaczy" go na nasz własny. Zmiany w interfejsie obiektu pochodzącego np. z biblioteki nie mają wpływu na działanie systemu, a jedynie na `Adapter`.
- **dostępność dla podklas** - jak w przykładzie abstrakcyjnym, stworzenie adaptera dla ogólnego interfejsu umożliwia stosowanie każdej klasy implementującej ten interfejs.
- **ostrożnie z extension functions** - czasami może się wydawać, że `extension function` zapewni wystarczającą funkcjonalność, ale w dużych projektach z wieloma zespołami pracującymi na tym samym kodzie może to powodować nieprzewidziane konsekwencje. Ograniczając zasięg `extension functions` można ten problem częściowo rozwiązać.