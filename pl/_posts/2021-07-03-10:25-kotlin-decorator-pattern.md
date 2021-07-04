---
layout: post
title: "Dekorator w Kotlinie"
date:  "2021-07-03 10:25"
description: "
`Dekorator` stosuje się tam, gdzie tworzenie osobnych klas będących kombinacją wszystkich możliwości skończyłoby się ich eksplozją. Wzorzec ten skupia się na stworzeniu warstw obiektów w celu transparentnego i dynamicznego uzupełniania obiektów o kolejne zadania. Dekorator dostarcza obiekt, o takim samym interfejsie co obiekt dekorowany.
"
permalink: "kotlin-decorator-pattern"
comments: true
toc: true
tags:
- kotlin
- Decorator Pattern
- design patterns
- extension function
- Wrapper

categories:
- Design Patterns

image: /assets/posts/decorator.jpg

---

# Przeznaczenie
Dekorator pozwala na dynamiczne dodanie lub zmianę zachowania konkretnego obiektu danej klasy, bez wpływu na inne obiekty tej samej klasy. W pewnych przypadkach pozwala to znacząco ograniczyć liczbę klas, przez wyciągnięcie współdzielonego zachowania do klasy dekorującej, zamiast rozbudowy struktury dziedziczenia. 

Zadaniem dekoratora jest "owinąć" (stąd inna nazwa: `Wrapper`) oryginalny obiekt i zmodyfikować lub nadpisać jego zachowanie. Klasa dekoratora ma ten sam interfejs co obiekt dekorowany, dlatego dla klienta nie ma znaczenia czy obiekt został udekorowany, czy nie. Co do zasady, Dekorator raczej nie dodaje nowych publicznych metod, bo klient korzystający z interfejsu oryginalnego obiektu i tak nie miałby do nich dostępu. Posiadanie tego samego interfejsu co oryginalny obiekt ma tę zaletę, że dekoratory mogą się swobodnie zagnieżdżać i owijać jeden w drugi.

Dekorowanie odbywa się dynamicznie, w trakcie działania programu, a nie w czasie kompilacji. Obiekt dekorowany jest zwykle przekazywany w konstruktorze Dekoratora, ponieważ instancja `Dekoratora` nie ma sensu sam z siebie, bez obiektu, który owija.

# Implementacja
We wzorcu `Dekorator` wyróżniamy kilka elementów:

{% plantuml %}
@startuml
skinparam linetype ortho
hide members
show Decorator members

class Client
interface Component
class ConcreteComponent implements Component
abstract class Decorator implements Component{
# component: Component
}
class ConcreteDecorator extends Decorator

Client -right- Component : using

Decorator o-- Component

@enduml
{% endplantuml %}

Elementy:
- **Client** - klasa korzystająca z obiektu `Component`, zna tylko ten interfejs, a nie konkretne klasy, tym bardziej nie musi wiedzieć o istnieniu dekoratora
- **Component** - interfejs obiektu, który `Decorator` dekoruje
- **ConcreteComponent** - konkretna implementacja `Component`, która potem trafi w konstruktorze do dekoratora
- **Decorator** - abstrakcyjna klasa implementująca interfejs `Component` i przyjmująca w konstruktorze obiekt z tym interfejsem. Użycie klasy abstrakcyjnej, zamiast tylko interfejsu pozwala wymusić konstruktor na dziedziczących klasach, a jednocześnie nie pozwala stworzyć instancji. Jeśli `Decorator` przyjmuje obiekty z ogólnym interfejsem `Component` a nie konkretne klasy, może dekorować całe rodziny obiektów.
- **ConcreteDecorator** - konkretna implementacja dekoratora

## Abstrakcyjna
W kodzie prezentuje się to mniej więcej tak:
```kotlin
interface Component {
	// przykładowe metody dekorowanego interfejsu
    fun methodA()
    fun methodB()
}
// konkretna implementacja, może ich być wiele
class ConcreteComponent : Component {
    override fun methodA() {}
    override fun methodB() {}
}
// Dekorator `jest obiektem` Component i `posiada obiekt` Component
// pole `component` jest `protected` i dzięki temu dostępne dla konkretnych Dekoratorów
abstract class Decorator(protected val component: Component) : Component

// Konkretna implementacja dekoratora, wymuszony konstruktor przyjmujący obiekt Component
class ConcreteDecorator1(component: Component) : Decorator(component) {
    // metody muszą być nadpisane
	// w tym przypadku Dekorator wywołuje metody owijanego obiektu bez żadnych zmian
	// czyli jest w zasadzie proxy
    override fun methodA() = component.methodA()
    override fun methodB() = component.methodB()
}
// kolejna implementacja Dekoratora
class ConcreteDecorator2(component: Component) : Decorator(component) {
    override fun methodA(){
		// w tej wersji obiektu nie można wykonać methodA()
		// może to być połączone ze sprawdzaniem parametrów komponentu
        throw Exception("you can't do this")
    }
    override fun methodB(){
        println("running methodB")
        component.methodB()
    }
}

fun main(){
	// "goły" obiekt `Component``
    val component: Component = ConcreteComponent()
	// pierwszy Dekorator owijający obiekt
    val dec1: Component = ConcreteDecorator1(component)
	// drugi Dekorator, owijający obiekt owinięty już w pierwszy
    val dec2: Component = ConcreteDecorator2(dec1)
}
```

Widać tutaj jak łatwo można zagnieżdżać Dekoratory. W tej implementacji `ConcreteDecorator1` to w zasadzie `Proxy` bo wywołuje bez zmian metody z przekazanego obiektu `Component`. Zwróć uwagę na `ConcreteDecorator2` - `methodA()` rzuca wyjątek. Dekorator może też zwracać stałą i w ogóle nie wykorzystywać przekazanego obiektu `Component`. `Dekorator` sam decyduje, jak się zachować. Na podstawie własności owijanego obiektu może zdecydować o rzuceniu wyjątku zamiast zwróceniu jakiejś wartości.

{% plantuml %}
@startuml
title Zagnieżdżone dekoratory
node Decorator1{
node Decorator2{
node Component{

		}
	}
}
@enduml
{% endplantuml %}


## Delegaty 

Kotlin ma wbudowane wsparcie dla [wzorca delegacji](https://kotlinlang.org/docs/delegation.html), czyli kolejnego wzorca stawiającego kompozycję ponad dziedziczeniem. Ogólnie chodzi o to, żeby przekazywać (**delegować**) wykonanie zadania wynikającego z interfejsu klasy do jakiegoś obiektu będącego jej składnikiem, przekazanym w konstruktorze, lub wstrzykniętym przez DI. W ten sposób można sobie skomponować klasę z reużywalnych delegatów, zamiast powielać implementację lub tworzyć dziwne struktury dziedziczenia. Trochę taka metoda szablonowa tylko, że dla klasy — klasa szablonowa :)

Delegaty można wykorzystać do implementacji `Dekoratora`. Delegaty w Kotlinie są tworzone przy użyciu słowa kluczowego `by`:
```kotlin
// podstawowy interfejs do użycia z Dekoratorem
interface Component {
    fun sayHello(): String
}
class ConcreteComponent1 : Component {
    override fun sayHello() = "hello from ${javaClass.simpleName}"
}

// delegujemy zachowanie z interfejsu `Component` do obiektu `component` z konstruktora
abstract class Decorator(protected val component: Component): Component by component

class Decorator1(component: Component) : Decorator(component) {
	// nadpisujemy metodę z komponentu, dodając informację z dekoratora
    override fun sayHello() = "${component.sayHello()} and from ${javaClass.simpleName}"
}

// nie ma potrzeby za każdym razem nadpisywać metody `sayHello()`
// została "oddelegowana" do obiektu `component` z konstruktora
class Decorator2(component: Component) : Decorator(component)

fun main() {
    val first: Component = ConcreteComponent1()
    val decOne: Component = Decorator1(first)
    val decTwo: Component = Decorator2(decOne)
    println(first.sayHello())
    println(decOne.sayHello())
    println(decTwo.sayHello())
}
```
Alternatywnie, `component` może nie być nawet polem w abstrakcyjnym Dekoratorze. Wtedy dziedziczące konkretne dekoratory mogą użyć `super`, które oddeleguje wywołanie metody do obiektu `comm` z konstruktora.
```kotlin
abstract class AltDecorator(component: Component): Component by component

class Decorator1(component: Component) : Decorator(component) {
    override fun sayHello() = "${super.sayHello()} and from ${javaClass.simpleName}"
}
```

Delegować można dowolną liczbę interfejsów, **ale tylko interfejsów**, a nie klas:
```kotlin
interface Component {
    fun sayHello(): String
}
interface Component2{
    fun sayBye(): String
}

abstract class Decorator(
        protected val component: Component,
        protected val component2: Component2
) :
        Component by component,
        Component2 by component2
```
Dzięki użyciu delegatów, konkretny `Dekorator` może zawierać wyłącznie metody, które faktycznie chce zmienić. Abstrakcyjna klasa bazowa `Dekorator` mogłaby też zawierać wywołania wszystkich metod na obiekcie dekorowanym, wtedy dziedziczące po niej konkretne klasy nadpisywałyby tylko wymagane dla siebie metody. Ale jeśli można to samo osiągnąć pojedynczym słówkiem `by` to po co przepłacać? :)

## Interfejsy komunikacyjne
Weźmy system wysyłający komunikaty tekstowe. Komunikaty mogą być wysłane przez Bluetooth lub TCP. Chcemy mieć możliwość wysłania samego tekstu, JSON-a oraz JSON-a z wiadomością zakodowaną Base64. Każdy rodzaj wiadomości powinien dać się przesyłać każdym medium. A z poziomu klienta nie powinno mieć znaczenia, jaki typ wiadomości i medium zostało wybrane.

Ok zacznijmy od sposobów komunikacji:
```kotlin
// ogólny interfejs komunikacyjny
interface Comm {
    fun sendMessage(text: String): Result
}

// klasa obsługująca Bluetooth przy pomocy przekazanego BtBodule
class BtComm(val bt: BtModule) : Comm {
    override fun sendMessage(text: String): Result {
        return bt.send(text)
    }
}
// klasa obsługująca TCP przy pomocy przekazanego TcpModule
class TcpComm(val tcpModule: TcpModule) : Comm {
    override fun sendMessage(text: String): Result {
        return tcpModule.send(text)
    }
}
```
Klienty będą znały wyłącznie ogólny interfejs `Comm`, bez wiedzy, jaki to konkretnie sposób przesyłania wiadomości. Szczegółami przesyłania przez Bluetooth i TCP zajmą się osobne moduły. Z punktu widzenia interfejsu komunikacyjnego mamy tutaj fasadę na potencjalnie skomplikowane przesyłanie wiadomości, dzielenie na ramki, parowanie urządzeń, ustalenie adresu itd.

`BtComm` i `TcpComm` to konkretne klasy implementujące dekorowany interfejs, a nie dekoratory.

Użycie np. Bluetooth może wyglądać tak:
```kotlin
val btModule = BtModule()
val message = "hello"
val btComm: Comm = BtComm(btModule)
btComm.sendMessage(message) // sending message via BT: hello
```
Co spowoduje wysłanie gołego tekstu przez Bluetooth. Analogicznie wyglądałoby to dla TCP.

### JsonDecorator
Chcemy móc wysłać wiadomość w formacie JSON zarówno przez BT, jak i TCP. Można do tego wykorzystać dekorator:
```kotlin
// w tym dekoratorze delegowany `comm` nie jest polem a dziedziczące klasy nie mają do niego dostępu
abstract class CommDecorator(comm: Comm) : Comm by comm

class JsonDecorator(comm: Comm) : CommDecorator(comm) {
    override fun sendMessage(text: String): Result {
		// przez `super` wywoływana jest metoda z abstrakcyjnego dekoratora, 
		// która deleguje wywołanie do przekazanego `comm`
        return super.sendMessage("{\"message\":\"$text\"}")
    }
}
```
I użycie:
```kotlin
val tcpModule = TcpModule()
val btModule = BtModule()

val message = "hello"
// owijamy komunikację TCP w dekorator JSON
val tcpJsonComm: Comm = JsonDecorator(TcpComm(tcpModule))
tcpJsonComm.sendMessage(message) // sending message via TCP: {"message":"hello"}
// dla BT analogicznie:
val btJsonComm: Comm = JsonDecorator(BtComm(btModule))
btJsonComm.sendMessage(message) // sending message via BT: {"message":"hello"}
```
Jeśli teraz doszedłby kolejny sposób komunikacji (Websockety, Firebase, gołąb pocztowy) i wiadomości powinny mieć taki sam format, to nie ma problemu. Po prostu udekoruje się nową implementację `Comm` już gotowym dekoratorem. Podobnie, jeśli powstanie nowy format wiadomości, przy pomocy dekoratora można użyć znanych już sposobów przesyłania.

### Base64Decorator
Ostatni dekorator posłuży do kodowania wiadomości przy użyciu Base64:
```kotlin
class Base46Decorator(comm: Comm) : CommDecorator(comm) {
    override fun sendMessage(text: String): Result {
        return super.sendMessage(prepareMessage(text))
    }

    private fun prepareMessage(text: String): String {
        return Base64.getEncoder().encodeToString(text.toByteArray())
    }
}
```
Ma nową prywatną metodę do przygotowania wiadomości. Nie rozszerza w ten sposób oryginalnego interfejsu, więc nie ma różnicy czy klient zna `Comm` czy `Base46Decorator` - ma dostęp do tych samych metod, więc raczej zdecyduje się na `Comm` ze względu na łatwą podmienialność implementacji. Takie podprogowe zachęcanie do dobrych zachowań.

### Kolejność dekoratorów
Użycie wszystkiego razem:
```kotlin
val tcpBase64JsonComm: Comm = JsonDecorator( // zapakuj wiadomość w strukturę JSON
        Base46Decorator( // zakoduj całość Base64
                TcpComm( // wyślij całość przez TCP
                        tcpModule
                )
        )
)
tcpBase64JsonComm.sendMessage(message) // sending message via TCP: eyJtZXNzYWdlIjoiaGVsbG8ifQ==

// inna kolejność dekoratorów
val tcpJsonBase64Comm: Comm = Base46Decorator( // zakoduj samą wiadomość Base64
        JsonDecorator( // zapakuj całość w strukturę JSON
                TcpComm( // wyślij całość przez TCP
                        tcpModule
                )
        )
)
tcpJsonBase64Comm.sendMessage(message) // sending message via TCP: {"message":"aGVsbG8="}
```
Jak widać, kolejność dekoratorów ma kolosalne znaczenie. Co się właściwie stało?
- **tcpBase64JsonComm**
	1. zapakuj wiadomość w strukturę JSON
	2. zakoduj całość Base64
	3. wyślij całość przez TCP
- **tcpJsonBase64Comm**
	1. zakoduj samą wiadomość Base64
	2. zapakuj całość w strukturę JSON
	3. wyślij całość przez TCP
	
Im głębiej znajduje się dekorator, tym później zostanie wywołany z wynikiem przetwarzania danych z wcześniejszych dekoratorów.

{% plantuml %}
@startuml

Client -> JsonDecorator : sendMessage("hello")
JsonDecorator -> Base46Decorator : sendMessage({"message": "hello"})
Base46Decorator -> TcpComm : sendMessage("eyJtZXNzYWdlIjoiaGVsbG8ifQ==")
TcpComm -> TcpModule : send(body: "eyJtZXNzYW...)
TcpModule -> TcpComm : Success/Fail
TcpComm -> Base46Decorator : Success/Fail
Base46Decorator -> JsonDecorator : Success/Fail
JsonDecorator -> Client : Success/Fail

@enduml
{% endplantuml %}

Jak już pisałem wcześniej, któryś z dekoratorów mógłby też rzucić wyjątkiem albo w ogóle nie użyć metody z dekorowanego obiektu i zwrócić np. stałą. Bywa to pomocne np. kiedy jakieś dane chcemy zaciemnić w zależności od tego, gdzie je wysyłamy. Przykładowo logi z aplikacji z użyciem klasy `data class LogEntry(val timestamp: Timestamp, val tag: String, val message: String)`:
- jeśli są przechowywane lokalnie niech mają wszystkie informacje
- jeśli są wysyłane do naszego serwisu przez HTTPS niech tylko wrażliwe dane będą zaciemnione
- jeśli na jakiś zewnętrzny monitoring to wysyłamy minimum informacji, może nawet samego TAG-a i timestamp. 
  
Nie ma sensu rozszerzać implementacji logowania lub wysyłki logów o te funkcjonalności. Można stworzyć dekoratory na `LogEntry`, które korzystając z wewnętrznej logiki, oczyszczałyby przed wysłaniem wiadomość. Zmiana polityki logowania lub dodanie nowych agregatorów nie spowoduje konieczności zmian interfejsów klas odpowiedzialnych za zbieranie czy wysyłkę logów, a jedynie podmianę dekoratorów.

### Extension Methods
Wydaje się, że zamiast dodawać kolejne klasy i bawić się w delegaty można dopisać `extension methods` do interfejsu i w ten sposób przygotować wiadomość do wysłania jako JSON i/lub Base64.
```kotlin
fun String.toJsonMessage(): String = "{\"message\":\"$this\"}"
fun String.toBase64(): String = Base64.getEncoder().encodeToString(this.toByteArray())

btComm.sendMessage(message.toBase64().toJsonMessage()) // sending message via BT: {"message":"aGVsbG8="}
```
Jednak teraz to klienty muszą wiedzieć, w jakiej formie wysyłać wiadomości, zamiast użyć udekorowanego obiektu z ogólnym interfejsem, który można im wstrzyknąć. Dodatkowo `extension method` może zostać niejawnie nadpisane w innym miejscu projektu, więc nigdy nie ma pewności, że wywołanie `toJsonMessage()` da taki sam rezultat. Pisałem już o tym w kontekście [Wzorca Adapter](https://asvid.github.io/pl/kotlin-adapter-pattern#duck-typing). W tym przypadku korzystam z klasy `String`, ale jeśli `toJsonMessage()` zwracałoby obiekt JSON, to metoda `toBase64()` musiałaby to uwzględnić — przy korzystaniu z dekoratorów interfejs jest zawsze ten sam i dekoratory nie muszą o sobie nic wiedzieć nawzajem.

Kod przy wykorzystaniu `extension methods` jest znacznie prostszy niż z użyciem serii dekoratorów. Jednak jeśli metody rozszerzające zaczną być stosowane w wielu miejscach i z wieloma interfejsami, to ich utrzymanie może być kłopotliwe, podobnie jak testowanie. A jeśli nie będą stosowane nigdzie indziej, to może zwykłe metody w klasie wystarczą zamiast `extension methods`. Chociaż przyznam, że czasami preferuję składnię `instance.extMethod()` zamiast `extMethod(instance)`.

# Nazewnictwo
Zwykle nie lubie w nazwach klas sufixów pochodzących ze wzorców projektowych, ale w przypadku `Dekoratora` dobrze mieć jasną informację o celu klasy. Mimo że interfejs klasy jest taki sam jak dekorowanego obiektu, to jednak sama instancja `Dekoratora` nie ma sensu, w przeciwieństwie do obiektu dekorowanego.

# Podsumowanie
Wzorzec `Dekorator` stosuje się tam, gdzie tworzenie osobnych klas będących kombinacją wszystkich możliwości skończyłoby się ich eksplozją. Wzorzec ten skupia się na stworzeniu warstw obiektów w celu transparentnego i dynamicznego uzupełniania obiektów o kolejne zadania. Dekorator dostarcza obiekt, o takim samym interfejsie co obiekt dekorowany. 

Dodawanie nowych metod publicznych w dekoratorze jest możliwe, jednak może zachęcać do rzutowania w górę lub używania interfejsu konkretnego dekoratora zamiast ogólnego komponentu. Główną zaletą Dekoratora jest jego przeźroczystość dla klienta, którą osiąga się przez korzystanie z generycznego interfejsu owijanego komponentu. 

W przykładzie z interfejsami komunikacyjnymi, gdyby nie użycie dekoratora, pewnie trzebaby rozszerzyć klienty o nowe funkcjonalności, lub wprowadzić wszystkie wariacje przetwarzania wiadomości i sposobu przesłania jej w osobnych klasach.

Kotlin pozwala na eleganckie tworzenie dekoratorów za pomocą delegatów. `Extension methods` w pewnym sensie również dekodują oryginalny obiekt, jednak nie są zamiennikiem czy alternatywą dla tego wzorca. Są za to alternatywą do stosowania prostych `Wrapperów` znanych z Javy, które dodają nowe publiczne metody do obiektów, których nie chcemy bądź nie możemy rozszerzać.

# Konsekwencje

- **zmiana zachowania obiektu bez dziedziczenia** - dziedziczenie jest właściwe tylko w przypadkach, gdzie klasa pochodna jest podtypem klasy bazowej (relacja na zasadzie generalizacja->specjalizacja) - [^effective_java]. Pewnie dałoby się znaleźć takie połączenie pomiędzy `String` a pakietem `TCP`, jednak byłoby to trochę naciągane i mało elastyczne. Szczególnie dodając po drodze formatowanie do JSONa i kodowanie Base64. Trudno byłoby też użyć kod ponownie w przypadku wiadomości przesyłanej przez Bluetooth. Dekorator przez kompozycję warstw pozwala elastycznie zmieniać zachowanie obiektu, bez zmiany tego, czym obiekt jest.
- **dynamiczne zmiany zachowania obiektu** - obiekt można udekorować w trakcie działania programu, ponieważ nowe zachowania nie zmieniają interfejsu, a jedynie wpisują się w niego. Ten sam obiekt może w jednym miejscu być dekorowany jedną klasą, a w kolejnym drugą.
- **SRP** - duża monolityczna klasa z wieloma odpowiedzialnościami, może zostać przekształcona w zbiór dekoratorów używanych tylko tam, gdzie są potrzebne. Może to niestety prowadzić również do rzutowania w górę do interfejsu dekoratora lub odwoływaniu się do wewnętrznego dekorowanego obiektu z poziomu klienta (naruszenie prawa Demeter).
- **kolejność stosowania dekoratorów jest ważna** - przykład z wysłaniem wiadomości. Kolejność jest ważna, a jednocześnie same dekoratory nie wiedzą nic o sobie. Odpowiedzialność tworzenia poprawnego zestawu dekoratorów powinna spoczywać na jakiejś np. `Fabryce` lub `Builderze` — o ile mamy do czynienia z rozbudowaną hierarchią.

---
[^effective_java]:["Effective Java"](https://books.google.pl/books/about/Effective_Java.html?id=ka2VUBqHiWkC&redir_esc=y) 