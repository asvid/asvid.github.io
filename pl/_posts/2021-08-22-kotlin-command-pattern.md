---
layout: post 
title: "Wzorzec Command (Polecenie) w Kotlinie"
date:  "2021-08-22 10:53"
description: "
Wzorzec Command polega na opakowaniu żądania w konkretny obiekt, posiadający wszystkie informacje niezbędne do wykonania swojego zadania. Można o tym pomyśleć jak o kolejnym etapie refaktoryzacji, gdzie najpierw wydzielamy kod do osobnej metody, a następnie do osobnego obiektu, przyjmującego w konstruktorze argumenty potrzebne do wykonania żądania.
"
permalink: "kotlin-command-pattern"
comments: true 
toc: true 
tags:
- kotlin
- Command Pattern
- design patterns

categories:
- Design Patterns

image: /assets/posts/command.jpg

---
# Przeznaczenie
Wzorzec `Command` polega na opakowaniu żądania w konkretny obiekt, posiadający wszystkie informacje niezbędne do wykonania swojego zadania. Można o tym pomyśleć jak o kolejnym etapie refaktoryzacji, gdzie najpierw wydzielamy kod do osobnej metody, a następnie do osobnego obiektu, przyjmującego w konstruktorze argumenty potrzebne do wykonania żądania.

Dzięki temu, że żądanie jest obiektem, może zostać przekazane do wykonania do osobnego obiektu (`CommandProcessor`), co pozwala na ich kolejkowanie i ułatwia logowanie zdarzeń. To samo polecenie może być wykorzystywane w różnych miejscach systemu, enkapsulując całą logikę wykonania żądania, co zapobiega duplikacji kodu. Nie musi to być ta sama instancja obiektu, ale klasa polecenia.

Obiekt żądania, oprócz standardowej metody `execute()` może zawierać metodę w stylu `undo()` czyli cofnięcie wprowadzanych przez żądanie zmian. Praktycznie wszystkie programy graficzne czy edytory tekstu mają taką opcję. Przechowują każdą zmianę w formie `Polecenia` w jakimś buforze o ograniczonej pojemności (stąd możliwość np. tylko 3 cofnięć), i jeśli użytkownik chce cofnąć ostatnią wprowadzoną zmianę, `Polecenie` jest ściągane z bufora i wprowadzane przez nie zmiany są cofane.

# Implementacja

## Abstrakcyjna
Jak już wspomniałem, standardowo obiekt `Command` ma metodę `execute()` (lub analogiczną). Oprócz samego obiektu żądania występuje w tym wzorcu kilka innych klas:
- **Receiver** - to na nim `Command` wykonuje żądania. Jeśli `Polecenie` polega na ściągnięciu pogody z internetu, `Receiver` będzie np. klientem HTTP. Może to być dowolna klasa.
- **Invoker** - klasa, która żąda obsłużenia polecenia
- **Client** - tworzy obiekt polecenia `ConcreteCommand` i wiąże go z odbiorcą `Receiver` oraz `Invokerem`
- **ConcreteCommand** - konkretne polecenie. Posiada wszystkie inne obiekty potrzebne do wykonania żądania.

{% plantuml %}
@startuml
hide Client members
  skinparam linetype ortho

interface Command{
	+ execute()
}

class ConcreteCommand implements Command{
	- receiver: Receiver
	- params
+ execute()
}

class Receiver{
	+ action()
}

class Client
class Invoker{
- command: Command
+ setCommand(command)
+ executeCommand()
}

Invoker *-->Command
Client --> Receiver
ConcreteCommand::execute --> Receiver::action
note "tworzy instancję polecenia\nprzekazuje Receiver\ni parametry polecenia" as N1
Client .. N1
N1 ..> ConcreteCommand

note "ustawia instancję polecenia" as N2
Client .. N2
N2 ..> Invoker

note right of Invoker::executeCommand
	wykonuje polecenie
	kiedy jest to wymagane
end note

note right of ConcreteCommand::execute
	wywołuje metodę Receivera
end note
@enduml
{% endplantuml %}

Kolejność interakcji przedstawia się następująco:
{% plantuml %}
@startuml

participant aReceiver
aClient --> aReceiver : posiada
aClient --> aCommand: Command(aReceiver)
activate aClient
aClient -> aInvoker : setCommand(aCommand)
deactivate aClient
aInvoker -> aCommand : execute()
aCommand -> aReceiver : action()

@enduml
{% endplantuml %}

1. `Client` tworzy obiekt konkretnego polecenia `Command`, przekazując odbiorcę `Receiver`.
2. `Invoker` dostaje konkretną instancję polecenia.
3. `Invoker` wykorzystuje metodę `execute()` przekazanego polecenia do wykonania własnych zadań. Może to być np. wykonanie akcji po kliknięciu, jeśli `Invoker` jest przyciskiem UI.
4. `Command` wywołuje odpowiednie metody swojej instancji obiektu `Receiver`.

W takiej najbardziej ogólnej formie można to zaimplementować następująco:
```kotlin
// ogólny interfejs polecenia
// związanie już w tym momencie z klasą Receiver pozwala w pewnym sensie grupować polecenia
// związane np. z edytorem tekstu, grafiki czy zapytaniami HTTP
// ale nie jest to wymagane przez sam wzorzec
abstract class Command(val receiver: Receiver) {
    abstract fun execute()
}
// konketne polecenie przyjmujące Receiver w konstruktorze
class ConcreteCommand(receiver: Receiver) : Command(receiver) {
    override fun execute() {
        println("executing ConcreteCommand")
        // wywołanie odpowiedniej akcji na odbiorcy
        receiver.action()
    }
}
// rzeczywisty "wykonawca" akcji, znający szczegóły implementacji
// np. edytor tekstu lub klient HTTP
class Receiver {
    fun action() {
        println("performing action in Receiver")
    }
}
// obiekt wykonujący polecenie, które jest ustawiane setterem
// np. przycisk wywołujący polecenie po kliknięciu
class Invoker {
    private lateinit var aCommand: Command

    fun setCommand(command: Command) {
        aCommand = command
    }

    fun performAction() {
        aCommand.execute()
    }
}
// klasa łącząca odbiorcę, polecenie i obiekt wywołujący polecenie
class Client(invoker: Invoker) {
    private val receiver = Receiver()

    init {
		// ustawienie polecenia dla Invokera
        val concreteCommand = ConcreteCommand(receiver)
        invoker.setCommand(concreteCommand)
    }
}

fun main() {
    val invoker = Invoker()
	
    Client(invoker)
    invoker.performAction()
}
```

### Result
Polecenie może zwracać jakąś wartość. Nie zawsze będzie to jednak dobra praktyka. O ile zwracanie `Success/Failure` zwykle będzie OK, żeby poinformować klasę wywołującą o statusie wykonania polecenia, o tyle zwracanie jakichś konkretnych danych będzie kłócić się z podejściem `CQRS` - command query responsibility segregation. Ogólnie chodzi o to, że polecenie albo coś zmienia, albo coś zwraca. 

Weźmy taką sytuację: wyświetlamy listę imion, każdy obiekt listy można edytować i zmienić imię. Zmiana imienia jest poleceniem, które wykonuje operację na jakimś repozytorium danych. Czy takie polecenie powinno cokolwiek zwracać, a jeśli tak, to co dokładnie? Całą listę imion wraz ze zmienionym przez polecenie? A jeśli lista jest stronicowana, zwracać tylko stronę ze zmianą czy wszystkie elementy aż do strony ze zmianą? Czy może zwracać samo zmienione imię, które potem `Invoker` musi wrzucić do wyświetlanej listy. Wtedy mamy 2 reprezentacje listy imion: w repozytorium i w widoku, nawet jeśli mają te same wartości, nic nie daje takiej gwarancji. Najbezpieczniej będzie, jeśli polecenie zwróci `Success` co spowoduje, że widok pobierze sobie aktualną listę imion z repozytorium. Lub jeszcze lepiej, widok zawsze ma aktualną listę z repozytorium przy pomocy jakiegoś `data-bindingu` czy `Obserwatora`. Po wykonaniu polecenia lista sama się zaktualizuje i nie ma nawet potrzeby zwracania `success/failure` z polecenia, o ile nie ma potrzeby obsłużenia błędu.

Kolejnym pomysłem może być przesłanie jakiegoś callbacka w `execute()`, ale może nie idźmy tą drogą :)

```kotlin
// wynik polecenia
// `sealed` pasuje tutaj idealnie
sealed class CommandResult {
	// zwracane wartości mogą być `object` jeśli nie mają własnych parametrów jak np. powód błędu 
    class Success : CommandResult()
    class Fail : CommandResult()
}
// tym razem `interface` a nie `abstract class` bo nie mamy tutaj wymuszonego argumentu konstruktora
interface Command {
    fun execute(): CommandResult
}
// `receiver` pojawia się dopiero tutaj
class ConcreteCommand(private val receiver: Receiver) : Command {
	// wykonanie polecenia zwraca rezultat
    override fun execute(): CommandResult {
        println("executing ConcreteCommand")
		// w zależności od wyniku z `receivera`
        return if (receiver.action()) {
            CommandResult.Success()
        } else {
            CommandResult.Fail()
        }
    }
}
// `receiver` zwraca losowo True lub False
class Receiver {
    fun action(): Boolean {
        println("performing action in Receiver")
        return Random.nextBoolean()
    }
}

fun main() {
    val receiver = Receiver()
    // pominąłem `Invoker` w tym przykładzie, wywołanie polecenia mamy po prostu w main()
    val result = ConcreteCommand(receiver).execute()
    println("Command result is: $result")
}
```

### CommandProcessor
Zamknięcie całości polecenia w obiekcie umożliwia przesłanie go do `Processora` zamiast natychmiastowego wykonania. Procesor może zgodnie ze swoją wewnętrzną logiką kolejkować i wykonywać polecenia, ale z punktu widzenia `Invokera` nie ma to znaczenia. `Receiver` może zostać przeniesiony z polecenia do `Processora`, dzięki temu polecenia będą zawierać wyłącznie parametry, a resztę dostarczy `Processor` podczas wywoływania `execute()`. Należy jednak uważać, żeby metoda `execute()` miała zawsze taką samą sygnaturę i nie trzeba było w `Processorze` robić osobnego wywoływania ze względu na klasę polecenia. O ile `Command` i `Processor` znajdują się w tej samej domenie (np. zapytania HTTP do konkretnego API, wysłanie wiadomości przez Bluetooth), nie powinno być z tym problemów. Jeśli dany `Processor` musi przekazywać różne obiekty `Receiver` w trakcie wykonywania poleceń, może to oznaczać, że powinny one trafić do różnych `Processorów`.

```kotlin
// pomocna funkcja do losowego opóźnienia
fun randomDelay() = Random.nextLong(1000, 3000)

interface Command {
	// Receiver jest przekazywany dopiero w momencie wykonania polecenia
    suspend fun execute(receiver: Receiver)
}
// polecenia przyjmują tylko własne parametry, bez Receivera
class FirstCommand(private val param: Int) : Command {
    override suspend fun execute(receiver: Receiver) {
        println("executing FirstCommand $param")
        receiver.action()
    }
}

class SecondCommand(private val param: String) : Command {
    override suspend fun execute(receiver: Receiver) {
        println("executing SecondCommand $param")
        receiver.action()
    }
}

class Receiver {
	// wykonanie akcji Receivera może chwilę zająć, stąd `suspend` i `delay()`
    suspend fun action() {
        println("performing action in Receiver")
        delay(randomDelay())
        println("action finished!")
    }
}
// najważniejsza klasa w tym przykładzie
// zawiera obiekt Receivera używany przy wykonaniu poleceń
// same polecenia są odkładane na kolejkę FIFO w postaci `Channel`
object CommandProcessor {
    private val commands = Channel<Command>()
	// użycie osobnych Scope na dodawanie i wykonywanie poleceń powoduje brak blokowania
    private val processScope = CoroutineScope(Executors.newSingleThreadExecutor().asCoroutineDispatcher())
    private val executeScope = CoroutineScope(Executors.newSingleThreadExecutor().asCoroutineDispatcher())
    private val receiver = Receiver()

    fun process(command: Command) {
        processScope.launch {
			// dodanie opóźnienia nie powoduje blokowania dodawania kolejnych poleceń
            delay(randomDelay())
            println("adding $command to the queue")
            commands.send(command)
        }
    }

    init {
		// nasłuchiwanie na nowe polecenia w kolejce i wykonywanie ich jak tylko się pojawią
        executeScope.launch {
            for (command in commands) {
                command.execute(receiver)
            }
        }
    }
}
// Invoker bez zmian
class Invoker {
    private lateinit var aCommand: Command

    fun setCommand(command: Command) {
        aCommand = command
    }

    fun performAction() {
        CommandProcessor.process(aCommand)
    }
}

fun main() {
    val firstInvoker = Invoker()
    val secondInvoker = Invoker()

	// polecenia przyjmują tylko własne parametry, bez Receivera
    firstInvoker.setCommand(FirstCommand(1))
    secondInvoker.setCommand(SecondCommand("2"))

	// wykonanie poleceń po 10x
    repeat(10) {
        firstInvoker.performAction()
        secondInvoker.performAction()
    }
}
```
`CommandProcessor` to obiekt, który przetwarza wysłane do niego polecenia. Dodawanie poleceń nie blokuje ich wykonywania lub dodawania kolejnych. W takim przypadku najlepiej sprawdzą się polecenia, które nic nie zwracają. Po prostu wrzucamy je na kolejkę, a efekt polecenia (o ile jest potrzebny) powinien przyjść innym kanałem, jak na prawilne `CQRS` przystało. Zarządzanie kolejką i wątkami znajduje się po stronie `Processora`, `Invoker` nie musi się tym zajmować, ani nawet wiedzieć, czy są tam `coroutines`, Javowe wątki czy jakiś `RX`. Polecenia są wykonywane sekwencyjnie, jedno po drugim. W połączeniu z `delay()` na wykonanie daje to efekt natychmiastowego dodania poleceń na kolejkę, i mozolnego wykonywania ich.

Wykonywanie poleceń z kolejki może zostać zrównoleglone w łatwy sposób, przy użyciu `launchProcessor` przyjmującego `Channel`:
```kotlin
// najprawdopodobniej byłby dostarczany przez DI, ale na potrzeby przykładu `object` wystarczy
object CommandProcessor {
	// tutaj wszystko bez zmian
    private val commands = Channel<Command>()
    private val processScope = CoroutineScope(Executors.newSingleThreadExecutor().asCoroutineDispatcher())
    private val executeScope = CoroutineScope(Executors.newSingleThreadExecutor().asCoroutineDispatcher())
    private val receiver = Receiver()

    fun process(command: Command) {
        processScope.launch {
            delay(randomDelay())
            println("adding $command to the queue")
            commands.send(command)
        }
    }

    init {
        executeScope.launch {
			// uruchamiamy 5 wewnętrznych procesorów operujących na tej samej kolejce
            // jednocześnie może być wykonywanych 5 poleceń, tyle ile wewnętrznych processorów
            repeat(5) {
                launchProcessor(commands)
            }
        }
    }
	// prywatna `extension function` pozwalająca wielu konsumentom ściągać polecenia z kolejki
	// polecenia są wykonywane tylko raz
	// po zakończeniu wykonania polecenia, wew. procesor ściąga z kolejki następne
    private fun CoroutineScope.launchProcessor(channel: ReceiveChannel<Command>) = launch {
        for (command in channel) {
            command.execute(receiver)
        }
    }
}
```
Ale będzie to również oznaczać, że np. jeśli wykonanie „Polecenia 1” trwa dłużej niż „Polecenie 2”, to drugie zostanie ukończone wcześniej, co nie zawsze będzie poprawne. To zależy w dużej mierze od konkretnego przypadku.

### Undo
Polecenia mogą posiadać metodę umożliwiającą cofnięcie wprowadzonych przez siebie zmian, czyli `undo()` znane z edytorów graficznych lub tekstowych. Sposób jej implementacji będzie mocno zależał od sytuacji, ale można przyjąć, że polecenie zapamiętuje stan sprzed zmiany i w razie potrzeby go przywraca. Trzymanie wielu stanów w historii pochłania pamięć i z tego powodu bufor historii jest często ograniczony do ostatnich 3 poleceń.
Cofnięcie wykonania polecenia może również być osiągnięte przez wykonanie polecenia z przeciwnymi parametrami, nie trzeba wtedy trzymać stanu, a samo wykonanie `undo()` może być zapisane w historii. Cofnięte polecenia mogą trafiać na osobny bufor, co umożliwi ich ponowne wykonanie `redo()`.

Poniżej mocno uproszczony przykład `undo()` z zachowaniem stanu sprzed wykonania polecenia:
```kotlin
// generyczne polecenie rysowania "czegoś" na obiekcie `Canvas`
abstract class DrawCommand(private val canvas: Canvas) {
	// stan sprzed wykonania polecenia - lista już narysowanych elementów
    private var preCommandState = listOf<Shape>()

    abstract fun execute()
    // zachowanie stanu
    fun saveState() {
        preCommandState = canvas.shapes.toList()
    }

	// cofnięcie polecenia, czyli przywrócenie stanu sprzed jego wykonania
    fun undo() {
        println("undo ${this}")
        canvas.shapes = preCommandState.toMutableList()
    }
}
// interfejs i klasy kształtów
interface Shape
data class Line(val length: Int) : Shape
data class Circle(val diameter: Int) : Shape
// polecenia rysujące kształty
class DrawLine(private val length: Int, private val canvas: Canvas) : DrawCommand(canvas) {
    override fun execute() {
        saveState()
        canvas.draw(Line(length))
    }
}
class DrawCircle(private val diameter: Int, private val canvas: Canvas) : DrawCommand(canvas) {
    override fun execute() {
        saveState()
        canvas.draw(Circle(diameter))
    }
}

// Receiver
class Canvas {
	// stan rysunku
    var shapes: MutableList<Shape> = mutableListOf()
	// rysowanie elementu polega na dodaniu go do listy narysowanych
    fun draw(shape: Shape) {
        println("drawing a $shape")
        shapes.add(shape)
    }
}

fun main() {
    val canvas = Canvas()
    val commandsHistory = mutableListOf<DrawCommand>()

	// po wykonaniu polecenia jest ono dodawane do historii
    val drawLine = DrawLine(2, canvas)
    drawLine.execute()
    commandsHistory.add(drawLine)

    val drawCircle = DrawCircle(1, canvas)
    drawCircle.execute()
    commandsHistory.add(drawCircle)

    val drawLongLine = DrawLine(10, canvas)
    drawLongLine.execute()
    commandsHistory.add(drawLongLine)

    val drawBigCircle = DrawCircle(12, canvas)
    drawBigCircle.execute()
    commandsHistory.add(drawBigCircle)

    println("current shapes: ${canvas.shapes}")
    println("--- undo last 2 ---")
	// cofnięcie 2 ostatnich poleceń
    commandsHistory.removeLast().undo()
    commandsHistory.removeLast().undo()
    println("current shapes: ${canvas.shapes}")
}
```

### A może lambdy wystarczą?
Może się wydawać, że tworzenie całej klasy i potem obiektu do wykonania jakiejś akcji to zbytek oraz że taką samą funkcjonalność i czytelność uda się uzyskać korzystając z lambd.

Przykład z wykorzystaniem podobnego processora z poprzedniego przykładu:
```kotlin
fun main() {
	// polecenie przyjmuje Receiver w momencie wykonania
    val command = { receiver: Receiver ->
        println("this is a command to do stuff")
        receiver.action()
    }
    repeat(10) {
        CommandProcessor.process(command)
		
		// polecenia nie muszą być przypisane do konkretnych zmiennych
        CommandProcessor.process { receiver: Receiver ->
            println("this is a another command")
            receiver.action()
        }
    }
}
```
W zasadzie mówimy do `CommandProcessor`: wykonaj ten blok korzystając ze swojej instancji `Receivera`. Jednak tracimy możliwość zwinnego parametryzowania obiektów poleceń. Lambda może przyjmować parametry, ale będą użyte w momencie wykonania, czyli w `CommandProcessor`. Można je oczywiście przekazać w metodzie `process()`, ale trudno wtedy mówić o enkapsulacji poleceń, jeśli blok kodu mamy w jednym miejscu a w innym trzeba przekazać parametry. 
Gdyby metoda `action()` była `suspend` to należałoby to obsłużyć w Lambdzie, owijając wywołanie w jakiś `CoroutineScope`. Mógłby on pochodzić z `CommandProcessor` podobnie jak `Receiver` ale jest to kolejna komplikacja, która dochodzi w momencie tworzenia Lambdy.
Podsumowując: jeśli chcesz mieć parametryzowaną Lambdę, przekazywaną do procesora i wykorzystywać ją w wielu miejscach systemu - **stwórz klasę**.

## Przykład z Home Automation
Może to moje skrzywienie zawodowe, ale sterowanie zdalnymi urządzeniami idealnie nadaje się na życiowy przykład użycia wzorca `Command`.

Mamy urządzenia, które można włączyć lub wyłączyć oraz pilota do sterowania tymi urządzeniami. Pilot nie jest na sztywno związany z żadnym konkretnym urządzeniem, może sterować każdym lub wieloma jednocześnie. Nie ma też świadomości, którym urządzeniem steruje, po prostu obsługuje wciśnięcia swoich przycisków.

```kotlin
// Invoker, przujmuje polecenia w konstruktorze, ale mógłby też mieć settery
class RemoteController(
    private val firstButtonAction: Command,
    private val secondButtonAction: Command,
) {
    fun firstButton() {
        firstButtonAction.execute()
    }

    fun secondButton() {
        secondButtonAction.execute()
    }
}

interface Command {
    fun execute()
}

// Receiver
class Device(private val name: String) {
    // action
    fun switch(on: Boolean) {
        println("turning device $name ${if (on) "ON" else "OFF"}")
    }
}
// polecenie przyjmuje Receiver w konstruktorze
class TurnOnCommand(private val device: Device) : Command {
    override fun execute() {
		// i wywołuje na nim akcję zgodną ze swoim przeznaczeniem
        device.switch(true)
    }
}

class TurnOffCommand(private val device: Device) : Command {
    override fun execute() {
        device.switch(false)
    }
}

fun main() {

    // instancja Receivera
    val lightBulb = Device("living room light")

    // przekazana do poleceń
    val turnOn: Command = TurnOnCommand(lightBulb)
    val turnOff: Command = TurnOffCommand(lightBulb)

    // invoker (pilot) otrzymuje polecenia pod konkretne przyciski
    val remote = RemoteController(turnOn, turnOff)

    // ale sam invoker tylko wywołuje polecenia, nie wie co dokładnie się dzieje i z jakim urządzeniem
    remote.firstButton()

    remote.secondButton()
}
```

# Nazewnictwo
Dodawanie `Command` do nazw konkretnych poleceń wydaje się mieć sens, bo jednoznacznie określa, do czego służy dana klasa. W przypadku `Receivera` czy `Invokera`, które z natury mają już swoje określone zadania i znalazły się w tym wzorcu trochę "przy okazji" tylko wprowadzałoby zamieszanie. Dobrze widać to w ostatnim przykładzie *Home Automation*. 

# Podsumowanie
`Command` jest jednym z moich ulubionych wzorców, najczęściej stosowałem go z użyciem jakiejś formy `CommandProcessora`. Świetnie enkapsuluje żądanie i pozwala na jego przekazywanie i wielokrotne używanie. Ułatwia refaktoryzację, bo łatwo zamienić jedno polecenie na inne, lub zmienić implementację wewnętrzną bez wpływu na klienty klasy.

Obiekty poleceń mogą zawierać metodę do cofania wprowadzanych przez nie zmian. Dzieje się to przez przechowanie stanu `Receivera` sprzed wykonania polecenia, lub wykonanie polecenia z przeciwnymi parametrami. Cofnięte polecenia moga być odkładane na osobny bufor co pozwala na ich ponowne wykonanie w razie potrzeby.

## Zalety
- **enkapsulacja** - zawarcie całej logiki potrzebnej do wykonania zadania w obiekcie z ogólnym interfejsem ma wiele zalet. Pozwala odkładać w czasie wykonanie zadania, ułatwia ponowne użycie kodu, testowanie i refaktoryzację.
- **dynamiczna zmiana zachowania** - przekazywanie obiektów polecenia pozwala w czasie wykonywania programu zmieniać zachowanie `Invokerów`, np. po zmianie konfiguracji przesłanej zdalnie
- **zamiana wielu wywołań na jedno polecenie** - zamiast wywoływać w odpowiedniej kolejności kilka metod `Receivera` można stworzyć polecenie, które to zrobi.
- **undo/redo** - w naturalny sposób pozwala cofnąć i wykonać ponownie zestaw instrukcji
- **łatwe rozszerzanie możliwości** - dodanie nowego polecenia nie wpływa na poprzednie, ani na wywołanie w `Invokerze`

## Wady
- **wiele podobnych klas** - w zależności od sytuacji, użycie wzorca `Command` może spowodować powstanie wielu klas różniących się 1 linią kodu.
- **przedwczesna komplikacja** - zastosowanie tego wzorca zbyt wcześnie, może skończyć się pojedynczą klasą polecenia użytą w jednym miejscu, ale obwarowaną dodatkowymi interfejsami, `CommandProcessorem` itd.