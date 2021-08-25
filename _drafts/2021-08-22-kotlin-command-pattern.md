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

image: /assets/posts/adapter.jpg

---
# Przeznaczenie
Wzorzec `Command` polega na opakowaniu żądania w konkretny obiekt, posiadający wszystkie informacje niezbędne do wykonania swojego zadania. Można o tym pomyśleć jak o kolejnym etapie refaktoryzacji, gdzie najpierw wydzielamy kod do osobnej metody, a następnie do osobnego obiektu, przyjmującego w konstruktorze argumenty potrzebne do wykonania żądania.

Dzięki temu, że żądanie jest obiektem, może zostać przekazane do wykonania do osobnego obiektu (`Handler`), co pozwala na ich kolejkowanie i ułatwia logowanie.

Obiekt żądania, oprócz standardowej metody `execute()` może zawierać metodę w stylu `undo()` czyli cofnięcie wprowadzanych przez żądanie zmian. Praktycznie wszystkie programy graficzne czy edytory tekstu mają taką opcję. Przechowują każdą zmianę w formie `Polecenia` w jakimś buforze o ograniczonej pojemności (stąd możliwość np. tylko 3 cofnięć), i jeśli użytkownik chce cofnąć ostatnią wprowadzoną zmianę, `Polecenie` jest ściągane z bufora i wprowadzane przez nie zmiany są cofane.

# Implementacja

[comment]: <> (są w sumie 2 warianty, z handlerem lub bez i je powinienem tu opisać)
Jak już wspomniałem, standardowo obiekt `Command` ma metodę `execute()`. Oprócz samego obiektu żądania występuje w tym wzorcu kilka innych klas:
- **Receiver** - to na nim `Command` wykonuje żądania. Jeśli `Polecenie` polega na ściągnięciu pogody z internetu, `Receiver` będzie np. klientem HTTP.
- **Invoker** - klasa, która żąda obsłużenia polecenia
- **Client** - tworzy obiekt polecenia i wiąże go z odbiorcą
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
note "creates command instance\npassing receiver\nand params" as N1
Client .. N1
N1 ..> ConcreteCommand

note "sets command instance" as N2
Client .. N2
N2 ..> Invoker

note right of Invoker::executeCommand
	executes command 
	when needed
end note

note right of ConcreteCommand::execute
	calls Receiver
end note
@enduml
{% endplantuml %}

## Abstrakcyjna

## Result

## Lambdas

## CommandProcessor

# Nazewnictwo

# Podsumowanie