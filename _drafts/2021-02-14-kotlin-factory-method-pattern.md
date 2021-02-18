---
layout: post
title: "Kotlin Factory Method"
date:  "2021-01-20 11:43"
description: "
TBD
"
permalink: "kotlin-factory-method"
comments: true
toc: true
tags:
- design patterns
- Kotlin
- Factory Method
- construction design pattern
  
categories:
- Design Patterns
      
image: /assets/posts/kotlin-builder-pattern/pkin.jpg

---

## Przeznaczenie

Podobnie jak Builder wzorzec Factory Method służy do oddzielenia procesu tworzenia obiektów od ich reprezentacji. W odróżnieniu od Buildera, zazwyczaj nie będzie nas interesowało podanie wszystkich wymaganych przez obiekt argumentów i zależności — to będzie należeć do zadań Factory. Fabryka może dostarczać obiekty różnych typów implementujących ten sam interfejs, wyłącznie na podstawie dostarczonych argumentów. Inaczej mówiąc — pozwala nam to podmieniać implementację w runtime, a nie w czasie kompilacji.

Korzystając z Fabryki, mówimy sobie mniej więcej: "wiem tylko `to` i `tamto`, daj mi poprawny obiekt danego typu". Przykład: `Locale.forLanguageTag("pl-PL")` - dostarczy nam obiekt `Locale`, z którego wyciągniemy sobie pełną nazwę kraju lub języka. W przypadku buildera, powiemy raczej  "daj mi obiekt, który będzie miał ustawione `to` i `tamto` a resztę zostaw domyślne". Fabryki często mają wstrzykiwane zależności, które pozwalają im na tworzenie rozbudowanych obiektów z minimalną ilością informacji podaną przez klienta.

Jest kilka wariantów tego wzorca:
- Static Factory Method (już opisany [tutaj](#))
- "Typowe" Factory Method
- Abstract Factory

W tym poście chciałbym opisać Factory Method, zostawiając Abstract Factory na osobny wpis.
