---
layout: post
title: "Adapter w Kotlinie"
date:  "2021-06-15 11:43"
description: "
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

# Implementacja

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

## Abstrakcyjna

### Prawo Demeter

## Repozytorium

# Nazewnictwo

# Podsumowanie

## Zalety

## Wady