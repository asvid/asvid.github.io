---
layout: post
title: "Mediator w Kotlinie"
date:  "2021-07-17 16:23"
description: "
Zadaniem Mediatora jest organizować komunikację między bliskimi klasami. Wzorzec `Mediator` uwalnia zależności pomiędzy komponentami. Przejmuje na siebie interakcję między nimi, stając się głównym hubem komunikacyjnym dla grupy klas. Odwracamy sterowanie, ponieważ komponenty zaczynają informować tylko 'co się stało', zamiast nakazywać innym, żeby 'coś zrobiły'. Można go spotkać np. pod postacią `ViewModel` w Androidzie, gdzie oddziela interakcje UI od zmian modelu danych.
"
permalink: "kotlin_mediator_pattern"
comments: true
toc: true

tags:
- kotlin
- Mediator Pattern
- design patterns

categories:
- Design Patterns

image: /assets/posts/mediator.jpg

---

# Przeznaczenie
`Mediatorem` określamy obiekt, kapsułkujący interakcje pomiędzy obiektami (`komponentami`) z danego zbioru. Wzorzec ogranicza lub nawet ucina zupełnie, bezpośrednie zależności między obiektami. Komponenty mogą porozumiewać się ze sobą wyłącznie za pośrednictwem `Mediatora`, który staje się centralnym hubem przekazywania informacji oraz sterowania. `Mediator` steruje przepływem informacji, korzystając ze swojej wewnętrznej logiki.

Można to porównać do wieży kontroli lotów (ATC). Komunikują się z nią piloci wszystkich samolotów z okolicy i to wieża decyduje o kolejności startów i lądowań. Wieża nie kontroluje jednak całego lotu we wszystkich aspektach, a jedynie organizuje samoloty względem siebie. Łatwo wyobrazić sobie chaos bez wieży, gdyby piloci wszystkich maszyn musieli komunikować się bezpośrednio ze sobą, aby ustalić, kto ma następny lądować.

Z tego powodu, wzorzec `Mediator` wydaje mi się **pasować raczej do refaktoryzacji** klas z bliskiego otoczenia, których liczba wzajemnych interakcji zaczęła już przeszkadzać w utrzymaniu, ale mają dobrze ustalone odpowiedzialności. W pewnych przypadkach użycie `Mediatora` będzie miało sens od samego początku tworzenia kodu, ale może też być przerostem formy nad treścią.

Dość powszechnym zastosowaniem tego wzorca jest [View Model](https://developer.android.com/topic/libraries/architecture/viewmodel) znany z `Android Architecture Components`. Przechwytuje interakcje z UI i według własnej logiki informuje model danych oraz w drugą stronę — zmiany w modelu przekazuje do UI.

# Implementacja
Mediator ma bardzo prostą strukturę, wyróżniamy tak naprawdę 2 typy obiektów:
- **Mediator** - główny hub komunikacyjny, znany wszystkim `komponentom`
- **Component** - nie zna innych komponentów, nie zależy od nich (choć może od innych klas). Komunikuje się wyłącznie z `Mediatorem` który przesyła, bądź nie, informację do odpowiednich `komponentów`

{% plantuml %}
@startuml

interface Mediator{
	+ notify(sender, event, context)
}
class ConcreteMediator implements Mediator{
	- componentA: ComponentA
	- componentB: ComponentB
	- componentC: ComponentC
	+ notify(sender, event, context)
}

class ComponentA{
	- mediator: Mediator
}
class ComponentB{
	- mediator: Mediator
}
class ComponentC{
	- mediator: Mediator
}

ComponentA --* ConcreteMediator::componentA
ComponentB --* ConcreteMediator::componentB
ComponentC --* ConcreteMediator::componentC

ComponentA::mediator-left->ConcreteMediator::notify
ComponentB::mediator-right->ConcreteMediator::notify
ComponentC::mediator-up->ConcreteMediator::notify

@enduml
{% endplantuml %}

Warto zauważyć, że jedynym potrzebnym interfejsem jest `Mediator`, który znają wszystkie `komponenty`. Każdy `komponent` jest osobnym i niezależnym obiektem, który pewnie już dziedziczy z innej klasy czy implementuje potrzebne interfejsy. Można się pokusić o wprowadzenie dodatkowego interfejsu `Component` ale niewiele by to dało, bo `Mediator` i tak musi znać konkretne interfejsy swoich `komponentów`, żeby po otrzymaniu wiadomości móc wywołać ich konkretne metody.

Obiekt `mediatora` może otrzymać komponenty w konstruktorze, przez DI lub nawet stworzyć potrzebne instancje i zarządzać ich cyklem życia.

## Abstrakcyjna implementacja
Przejdźmy do podstawowej implementacji w kodzie:
```kotlin
interface Mediator {
    // mediator deklaruje metody komunikacji
    fun method(sender: Any, args: Any? = null)
}

// różne komponenty, nie muszą mieć wspólnego interfejsu, ale muszą mieć referencję do Mediatora
// nie powinny mieć zależności do siebie nawzajem, bo niszczy to sens Mediatora
class ComponentA(private val mediator: Mediator) {
    fun operationA() {
        mediator.method(this, "arg A")
    }
}

class ComponentB(private val mediator: Mediator) {
    fun operationB() {
        mediator.method(this, 10.34)
    }
}

class ComponentC(private val mediator: Mediator) {
    fun operationC() {
        mediator.method(this, println("print me!"))
    }
}

// Mediator hermetyzuje relacje pomiędzy komponentami
// posiada referencje do wszystkich komponentów jakimi zarządza
// czasami zarządza również ich cyklem życia
class Concretemediator : Mediator {

    // komponenty mogą być też wstrzykiwane lub przekazane w konstruktorze
    // ale referencja do Mediatora musi być przekazana do obiektu
    val componentA = ComponentA(this)
    val componentB = ComponentB(this)
    val componentC = ComponentC(this)

    override fun method(sender: Any, args: Any?) {
		// sprawdzenie jaki obiekt jest nadawcą
        when (sender) {
            // Mediator wie co zrobić z otrzymaną wiadomością
            is ComponentA -> println("arg from A: $args")
            is ComponentB -> println("arg from B: ${args as Float * 3}")
            is ComponentC -> (args as () -> Any).invoke()
        }
    }
}
```
Zamiast metody `notify(sender, event, context)` użyłem tutaj `method(sender, args)` ale zasada jest ta sama, to interfejs `Mediatora` określa formę komunikacji. `Event` w pierwszym wypadku może być obiektem dziedziczącym po jakimś ogólnym interfejsie albo klasą `sealed`, co stworzy zamknięty zbiór zdarzeń, po którym łatwiej się poruszać.

`Context` może oznaczać, z jakiego miejsca pochodzi wiadomość, np. czy produkt do koszyka zakupów został włożony z listy polecanych produktów dodatkowych, czy bezpośrednio ze strony produktu. Argumenty metody, czy nawet metod `Mediatora` będą mocno zależeć od konkretnej implementacji i potrzebnych informacji.

## Dialog
Częstym przykładem zastosowania `Mediatora` są okienka dialogowe, zawierające elementy UI takie jak przyciski i pola tekstowe. Często zdarza się, że stan elementów interfejsu użytkownika zależy od siebie wzajemnie. Przycisk "OK" powinien być nieaktywny, jeśli nie wszystkie wymagane pola formularza zostały wypełnione itp. Jeśli elementów widoku jest sporo, liczba takich zależności między nimi może rosnąć i być ciężka do kontrolowania.

Dlatego dobrym pomysłem jest posiadanie centralnego huba komunikacyjnego, który będzie zbierał zdarzenia z elementów UI i odpowiednio dostosowywał stan wyświetlanych elementów. Takim `Mediatorem` może być sam `Dialog`.

```kotlin
// zarządca widoku, kontrolujący elementy UI do niego przypisane
interface UiDirector {
    // `event` jest naiwnie Stringiem dla uproszczenia
    // `sender` musi być elementem UI dziedziczącym po klasie `UiElement`
    fun notify(sender: UiElement, event: String)
}

// klasa `UiElement` jest komponentem UI osadzonym w `UiDirector`
abstract class UiElement(val uiDirector: UiDirector)

// przycisk jest elementem UI
class Button(uiDirector: UiDirector) : UiElement(uiDirector) {
    fun click() {
        // który powiadamia `uiDirector` o fakcie kliknięcia
        // ale sam nie wykonuje żadnej logiki z tym związanej
        uiDirector.notify(this, "click")
    }
}

class TextBox(uiDirector: UiDirector) : UiElement(uiDirector) {
    // TextBox również jest elementem UI
    // przyjmijmy że posiada wewnętrzną logikę odświeżającą zawartość pola `text`
    // która uwzględnia np. wklejanie tekstu, wpisywanie przez użytkownika ręcznie itd.
    // i korzystając z `observable` po każdej zmianie informuje dialog
    // dzięki temu nie trzeba w każdym miejscu zmieniającym `text` pamiętać o wywołaniu `notify`
    val text: String by Delegates.observable("") { property, oldValue, newValue ->
        uiDirector.notify(this, "text_changed")
    }
}

// Okienko dialogowe, zarządza stanem widoku na podstawie zdarzeń z komponentów UI
class FancyDialog : UiDirector {

    private val title = "fancy dialog"
    // dialog sam tworzy instancje swoich `komponentów`
    private val okButton = Button(this)
    private val cancelButton = Button(this)
    private val input = TextBox(this)

    private var inputText = ""

    override fun notify(sender: UiElement, event: String) {
        when (sender) {
            // po każdej zmianie kontrolki dostaje zaktualizowane dane
            input -> if (event == "text_change") inputText = input.text
            // posiada logikę wywoływaną po kliknięciach w przyciski
            okButton -> if (event == "click") submit()
            cancelButton -> if (event == "click") dismiss()
        }
    }
    private fun dismiss() {}
    private fun submit() {}
}
```

Pozwala to używać ponownie implementacji kontrolek UI, ponieważ nie mają żadnej konkretnej logiki poza zarządzaniem swoim własnym stanem. `Delegates.observable` w klasie `TextBox` ułatwia powiadamianie zarządcy widoku o zmianie. Nie trzeba tego robić w każdym miejscu zmieniającym pole `text`.

## Chat
Ciekawym przykładem zastosowania `Mediatora` jest chat. Mamy uczestników którzy wysyłają wiadomości publiczne lub bezpośrednio do innego uczestnika, ale to obiekt `Chatu` decyduje kto dostanie jaką wiadomość.

```kotlin
// pomocniczy alias
typealias ParticipantName = String

// interfejs Mediatora
interface Chatroom {
    fun registerParticipant(participant: Participant)
    // odbiorca wiadomości jest opcjonalny, wtedy wiadomość jest traktowana jako publiczna
    fun send(message: String, from: ParticipantName, to: ParticipantName? = null)
}

// klasa uczestnika ma bardzo proste API, ogranicza się do wysłania i odebrania wiadomości
// nie ma tu logiki sterującej do kogo wiadomość trafia
class Participant(val name: String, private val chatroom: Chatroom) {

    init {
		// uczestnik rejestruje się na chacie
        chatroom.registerParticipant(this)
    }

	// potrafi wysłać wiadomość
    fun send(message: String, to: ParticipantName? = null) {
        chatroom.send(message, this.name, to)
    }

	// i ją odebrać
    fun receive(message: String) {
        println("[$name] gets: $message")
    }
}
// object tylko ze względu na prostotę użycia w `main()`
object SuperChat : Chatroom {
	// tryby pracy chatu
    enum class Mode { 
		PUBLIC, // każda wiadomość trafia do wszystkich
		PRIVATE, // tylko wiadomości z odbiorcą będą dostarczone
		MIXED // wiadomości bezpośrednie trafiają tylko do odbiorcy, pozostałe do wszystkich
    }

    var mode: Mode = Mode.PRIVATE

	// lista uczestników do której wpisywani są po rejestracji
    private val participants = mutableListOf<Participant>()

    override fun registerParticipant(participant: Participant) {
        participants.add(participant)
    }

	// obsługa wiadomości od uczesników chatu
    override fun send(message: String, from: ParticipantName, to: ParticipantName?) {
        when (mode) {
            // w zależności od trybu są wysyłane inaczej
            Mode.PUBLIC -> participants.forEach { it.receive("$from says: $message") }
            Mode.PRIVATE -> participants.find { it.name == to }?.receive("$from says: $message")
            Mode.MIXED -> {
                if (to == null) participants.forEach { it.receive("$from says: $message") }
                else participants.find { it.name == to }?.receive("$from says: $message")
            }
        }
    }
}

fun main() {
    val alice = Participant("Alice", SuperChat)
    val bob = Participant("Bob", SuperChat)
    val charlie = Participant("Charlie", SuperChat)

    alice.send("hi all!")
    bob.send("hi Alice", "Alice") // tylko ta wiadomość zostanie wypisana
    charlie.send("hi Alice")

    SuperChat.mode = SuperChat.Mode.PUBLIC
}
```

Ponieważ tryb chata to początkowo `PRIVATE`, tylko wiadomość z adresatem zostanie wypisana `[Alice] gets: Bob says: hi Alice`. Pozostałe wiadomości trafią do `Mediatora`, ale nie do innych uczestników. Zmiana polityki dostarczania wiadomości spowoduje taki efekt dla `MIXED`:
```
[Alice] gets: Alice says: hi all!
[Bob] gets: Alice says: hi all!
[Charlie] gets: Alice says: hi all!
[Alice] gets: Bob says: hi Alice
[Alice] gets: Charlie says: hi Alice
[Bob] gets: Charlie says: hi Alice
[Charlie] gets: Charlie says: hi Alice
```
> Jest tutaj błąd, ponieważ Alice dostaje wiadomość od samej siebie. Jej `hi all` nie miało adresata, więc trafiło do wszystkich uczestników — w tym jej samej. Podobnie jak Charlie odpowiadając `hi Alice` ale bez adresata.

Uczestnicy nie muszą wiedzieć o sobie nawzajem, żeby wysyłać sobie wiadomości, zajmuje się tym `Mediator` w formie obiektu `SuperChat`. Dodatkowo ma swoją wewnętrzną politykę rozsyłania wiadomości, nie jest tylko proxy pomiędzy obiektami. Chat mógłby mieć listę wyciszonych uczestników, którzy np. czasowo i za karę nie mogą wysyłać wiadomości. Ale zamiast blokować wysyłanie wiadomości, można zablokować ich dostarczanie bez informowania uczestnika o tym `shadow-banie`.

# Nazewnictwo
`Mediator` pod względem struktury jest bardzo prostym wzorcem, to w zasadzie 1 interfejs. Nie ma potrzeby dodawać tego do nazwy obiektów, które faktycznie pełnią funkcję Mediatora. W zależności od implementacji `Mediator` może nawet nie być widoczny na zewnątrz, w odróżnieniu od `Fasady`, która z definicji jest interfejsem wejściowym do jakiejś funkcjonalności.
Z tego względu zdecydowanie unikałbym używania nazwy Mediator w nazwach klas. Nawet metoda `notify()` może i powinna odnosić się do domeny rozwiązywanego problemu, jak w przypadku chatu `send()`. Na dobrą sprawę, wzorzec ten powinien być transparentny na zewnątrz grupy klas i być wyłącznie szczegółem implementacyjnym, usprawnieniem organizacyjnym.

# Podsumowanie
Zadaniem Mediatora jest organizować komunikację między bliskimi klasami. Wzorzec `Mediator` uwalnia zależności pomiędzy komponentami. Przejmuje na siebie interakcję między nimi, stając się głównym hubem komunikacyjnym dla grupy klas. Odwracamy sterowanie, ponieważ komponenty zaczynają informować tylko "co się stało", zamiast nakazywać innym, żeby "coś zrobiły". Można go spotkać np. pod postacią `ViewModel` w Androidzie, gdzie oddziela interakcje UI od zmian modelu danych. 

Mediator, korzystając z wewnętrznej logiki, decyduje co zrobić z informacją przychodzącą od jednego ze swoich komponentów. Może wywołać metodę na jednym lub wielu komponentach. Z tego względu musi znać konkretne interfejsy komponentów, a nie jakiś generyczny pochodzący ze wzorca. Teoretycznie `Komponent` też może posiadać tylko metodę `notify()` i samemu decydować jak zareagować na dany komunikat od `Mediatora` (mediacje w obie strony), ale to znacząco komplikuje sprawę i wchodzi w inne wzorce.

Należy uważać, żeby `Mediator` nie przerodził się w tzw. **boski obiekt**, rozrośnięty i skomplikowany, zawierający znaczną część logiki systemu. Mediator powinien być stosowany raczej lokalnie i nie jako interfejs dostępowy do funkcjonalności, w przeciwieństwie do Fasady. 

Dobrze nadaje się do refaktoryzacji, jeśli mamy grupę powiązanych klas, których ponowne użycie jest uciążliwe przez liczbę zależności. Usprawnia też testowanie, bo nie ma potrzeby tworzenia mocków grupy klas, a jedynie Mediatora. Jeśli wszystkie interakcje między komponentami zawierają się w Mediatorze, użycie tych samych komponentów, ale w innym kontekście będzie wymagało jedynie napisania nowego Mediatora.

## Zalety
- **ucina wzajemne zależności** - Mediator przejmuje komunikację między komponentami, więc nie muszą wiedzieć o sobie nawzajem.
- **testowanie** - wystarczy mock/stub Mediatora do przetestowania klasy, nie trzeba tworzyć całego drzewka zależności.
- **ułatwia ponowne użycie kodu** - brak zależności od innych komponentów pozwala użyć klasy w nowych miejscach w prosty sposób. Można użyć ponownie nawet całej funkcjonalności w nowym kontekście, tworząc nowy obiekt Mediatora bez zmian komponentów.
- **przydatny w refaktoryzacji** - Mediator pozwala rozplątać wzajemne zależności „dojrzałego” kodu, gdzie odpowiedzialności klas są już ustalone, ale ilość wzajemnych interakcji zaczyna przeszkadzać.

## Wady
- **boski obiekt** - może rozwinąć się do ogromnych rozmiarów, "połykając" kolejne komponenty i stać się punktem dostępowym do funkcjonalności, zamiast tylko organizatorem komunikacji.
- **przykrycie problemu zamiast rozwiązania** - przesunięcie całej komunikacji i zależności między obiektami do Mediatora może spowodować, że obiekty nadal będą od siebie zależne, ale niejawnie. Zamiast posiadać referencje do siebie w kodzie, będą nadal oczekiwać określonych wywołań od strony Mediatora, żeby poprawnie spełniać swoje zadanie.