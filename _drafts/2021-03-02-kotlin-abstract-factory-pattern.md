---
layout: post
title: "Kotlin Abstract Factory"
date:  "2021-03-02"
description: "
Po `Static Factory Method` nadeszła pora na klasyczne `Factory`. Fabryka jest bardzo użytecznym i często stosowanym wzorcem konstrukcyjnym. Kotlin daje nam ciekawe możliwości dzięki klasom `sealed` oraz `internal`, których odpowiedników brakuje w Javie.
"
permalink: "kotlin-abstract-factory"
comments: true
toc: true
tags:
- design patterns
- Kotlin
- Abstract Factory
- construction design pattern
  
categories:
- Design Patterns
      
image: /assets/posts/plyta.jpg

---

# Przeznaczenie

Nazwa tego wzorca nie do końca sugeruje, czym się wyróżnia od innych wzorców konstrukcyjnych, takich jak Builder czy Factory. W Abstract Factory nie chodzi o dostarczenie pojedynczej instancji — a całej `rodziny powiązanych obiektów`. Nadal brzmi to podobnie jak zwykłe Factory, które może produkować np. obiekty kontrolek GUI. Żeby otrzymać `Abstract Factory` musimy właśnie dodać kolejną warstwę abstrakcji, i stworzyć mechanizm produkujący obiekty kontrolek GUI, ale w wariantach dla np. Linuxa, Windowsa i MacOS, które nadal będą miały generyczne API dla klienta wzorca. 

`Abstract Factory` to **fabryka fabryk**, dostarczająca klientom wybraną wersję fabryki spośród tworzących obiekty o takim samym interfejsie. Rodziny obiektów będą powiązane przez konkretną fabrykę, która je buduje. Trzymając się tematu GUI, będziemy mieć rodziny kontrolek dla Windowsa, Linuxa i MacOS — zapobiegnie to sytuacji użycia przycisku z MacOS obok pola tekstowego z Windowsa. Klienta nie będzie jakoś szczególnie interesowało to, z której rodziny (i z którego dokładnie factory) otrzyma obiekt, ponieważ jego API będzie takie samo. I na tym polega `Abstrakcja` tej fabryki, na **udostępnieniu interfejsu fabryk rodzin obiektów**.

Przydatne informacje przed lekturą tego posta:
- [Static Factory Method](https://asvid.github.io/pl/kotlin-static-factory-methods)
- ["Typowe" Factory Method](https://asvid.github.io/pl/kotlin-factory-method)

# Przykłady implementacji
Sam miałem problem to zrozumieć zanim nie zobaczyłem kodu i jakichś diagramów, dlatego nejlepiej od razu przejść do przykładów.

## Podstawowa
Książkowy przykład pochodzący z "Wzorców Programowania"[^design_patterns] "bandy czworga".

Interfejs fabryki dostarczającej rodzinę produktów. Na rodzinę składają się 2 produkty: `ProductA` i `ProductB`.
```kotlin
interface Factory {
    fun createProductA(): ProductA
    fun createProductB(): ProductB
    
    companion object // może się przydać
}
```

Każdy z produktów ma 2 warianty `1` i `2`.
```kotlin
// produkty
interface ProductA
interface ProductB

// wariant 1
class Product1A : ProductA
class Product1B : ProductB

// wariant 2
class Product2A : ProductA
class Product2B : ProductB
```

Każdy wariant ma osobną konkretną fabrykę, dostarczającą oba produkty w odpowiednim dla siebie wariancie. `internal` gwarantuje niewyciekanie klasy poza moduł. Konkretnych fabryki są dobrym kandydatem do Singletona, stąd `object`.
```kotlin
// fabryka produktów w wariancie 1
internal object ConcreteFactory1 : Factory { 
    override fun createProductA() = Product1A()
    override fun createProductB() = Product1B()
}

// fabryka produktów w wariancie 2
internal object ConcreteFactory2 : Factory { // 
    override fun createProductA() = Product2A()
    override fun createProductB() = Product2B()
}
```

Fabryka abstrakcyjna, czyli zwracanie odpowiedniej konkretnej fabryki (odpowiedniego wariantu) ale schowanej za generycznym interfejsem `Factory`. W ten sposób klienty nie muszą nawet znać implementacji konkretnej fabryki. Klienty wiedzą tylko, że dostaną `Factory` i że będzie ono potrafiło zbudować `ProductA` i `ProductB`. Metody `getFactory()` jest tutaj tzw. `top-level function` bo znajduje się poza jakąkolwiek klasą, ale może Być częścią np. `companion object` interfejsu Factory. Jednak sam interfejs Factory niekoniecznie powinien znać swoje implementacje, na szczęście może mieć pusty `companion object`, który dostanie w odpowiednim miejscu `extension function`. Szerzej pisałem o tym [tutaj](wpis o builderze)
```kotlin
// naiwan ale działająca implementacja ze zwykłym Int-em
fun getFactory(whichOne: Int): Factory {
    return when (whichOne) {
        1 -> ConcreteFactory1
        2 -> ConcreteFactory2
        else -> throw IllegalArgumentException("value: $whichOne is not available")
    }
}
// warianty jako enum
enum class FactoryVariant {
    Variant1,
    Variant2
}
// mniej naiwna ale w zasadzie taka sama implementacja z enumem
fun getFactory(whichOne: FactoryVariant): Factory {
    return when (whichOne) {
        FactoryVariant.Variant1 -> ConcreteFactory1
        FactoryVariant.Variant2 -> ConcreteFactory2
        else -> throw IllegalArgumentException("value: $whichOne is not available")
    }
}

// extension function dla companion objectu Factory
fun Factory.Companion.getFactory(whichOne: Int): Factory {
    return when (whichOne) {
        1 -> ConcreteFactory1
        2 -> ConcreteFactory2
        else -> throw IllegalArgumentException("value: $whichOne is not available")
    }
}
```

Użycie:
```kotlin
val factory: Factory = getFactory(2)

factory.createProductA() //  Product2A
factory.createProductB() //  Product2B

val factory2: Factory = getFactory(FactoryVariant.Variant1)
factory2.createProductA() // Product1A
factory2.createProductB() //Product1B

// lub jako metoda companion object interfejsu Factory
Factory.getFactory(1).createProductA()

```

Na diagramie prezentuje się to następująco:

{% plantuml %}
@startuml
interface Factory{
+ createProductA() : ProductA
+ createProductB() : ProductB
}

interface ProductA
interface ProductB

class Product1A<< (C,#FF7700) Variant1 >> implements ProductA
class Product1B<< (C,#FF7700) Variant1 >> implements ProductB

class Product2A<< (C,#FF0077) Variant2 >> implements ProductA
class Product2B<< (C,#FF0077) Variant2 >> implements ProductB

class ConcreteFactory1<< (C,#FF0077) Variant1 >> implements Factory
class ConcreteFactory2<< (C,#FF7700) Variant1 >> implements Factory

Factory::createProductA-right[bold]-|>ProductA
Factory::createProductB-right[bold]-|>ProductB

@enduml
{% endplantuml %}

## Rodzina elementów GUI

## Fabryka z rejestrem

## Metoda Make


# Podsumowanie

## Zalety
- **całkowite oddzielenie implementacji od interfejsu** - z poziomu klienta fabryki interesuje nas wyłącznie interfejs obiektu, co pozwala na łatwe dodawanie nowych implementacji bez konieczności zmian klienta.

## Wady
- **konieczność tworzenia dodatkowych klas** - interfejsów, fabryk, typów wyliczeniowych itd. Niekoniecznie jest to wada, zwłaszcza jeśli chodzi o interfejsy, bo znacząco pomaga to potem testować kod. Należy wykazać się tutaj rozsądkiem, jeśli dostarczamy instancje pojedynczej klasy i nie ma perspektyw na zwiększenie liczby typów, to może nie ma potrzeby tworzyć ten cały boilerplate.

---
[^thinking_in_java]: [Thinking in Java](https://books.google.pl/books?id=bQVvAQAAQBAJ&dq=isbn:9780131872486&hl=pl&sa=X&ved=2ahUKEwjO6uawvojvAhVRtIsKHchPBwUQ6AEwAHoECAAQAg)
[^design_patterns]: [Wzorce projektowe - Elementy oprogramowania obiektowego wielokrotnego użytku](https://books.google.pl/books?id=Mkn6uAEACAAJ&dq=isbn:0201633612&hl=pl&sa=X&ved=2ahUKEwiQgZnevojvAhXeAxAIHerSDCsQ6AEwAHoECAAQAg)