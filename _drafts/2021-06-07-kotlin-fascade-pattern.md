---
layout: post
title: "Fasada w Kotlinie"
date:  "2021-06-05 11:43"
description: "

"
permalink: "kotlin-fascade-pattern"
comments: true
toc: true
tags:
- design patterns
- Kotlin
- Fascade Pattern
- structural design pattern

categories:
- Design Patterns

image: /assets/posts/strategy.jpg

---

# Przeznaczenie
Fasada jest bardzo prostym wzorcem, którego zadaniem jest przesłonić szczegóły jakiejś grupy klas - modułu odpowiedzialnego za jakąś funkcjonalność. 

Można to porównać do fasady budynku, która sama w sobie nie pełni żadnej funkcji. Budynek to pokoje, korytarze, schody, instalacje. Ale to fasada wskazuje wejście do budynku, a wyglądem może sugerować jego przeznaczenie.

Zadaniem wzorca jest upraszczanie klientom dostępu do funkcjonalności przesłanianego modułu. Tak, żeby nie musieli znać szczegółów implementacyjnych, zależności między klasami, wiedzieć którą wersję obiektu wstrzykiwać itd.

# Implementacja
Fasada to zasadniczo 1 klasa. Na diagramie przedstawia się to tak:

{% plantuml %}
@startuml
skinparam groupComposition 1
hide members
show Facade methods
class ClientA
class ClientB
  
ClientA-down-Facade
ClientB-down-Facade

package module <<Node>> {

class Facade{
	+ operation()
  }

class A
class B
class C
class D
class E
class F

A-down-*B
B-*C
D-*A
A-*F
D-*B
}

F-up->Facade
E-up->Facade
C-up->Facade

@enduml
{% endplantuml %}

Klasy `A,B,C,D,E,F` razem tworzą moduł i dostarczają jakąś funkcjonalność. `Facade` jest punktem dostępu do tej funkcjonalności. Dzięki temu żaden klient nie musi znać szczegółów modułu, wiedzieć jak tworzyć instancje klas i jak ich używac. Klientom wystarczy znajomość fasady, która jest w stanie zbudować drzewko zależności dla wszystkich klas, schować ich implementacje i dostarczyć proste API dla klientów.

Zastanawiałęm się, czy klasa `Facade` nie powinna być wewnątrz modułu. Takie zastosowanie Fasady jest często spotykane w bibliotekach - mają sporo wewnętrznej logiki, ale udostępniają jedynie proste API przez pojedynczą klasę i ew. kilka obiektów konfiguracyjnych. Jednak Fasada może służyć do przykrycia kodu, nad którym nie mamy kontroli, tworząc tzw. `Antycorruption Layer`. Wtedy zmieniający się za nią kod nie psuje klientów.

## Abstrakcyjna

## Repozytorium
Szczególnym przypadkiem Fasady jest `Repozytorium`.
# Nazewnictwo
# Podsumowanie
## Zalety
## Wady