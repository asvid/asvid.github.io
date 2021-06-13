---
layout: post
title: "Fasada w Kotlinie"
date:  "2021-06-05 11:43"
description: "
Fasada pozwala a ukrycie szczegółów obiektów modułu przed klientami. Zapewnia przestrzeganie `Prawa Demeter`, a użycie ogólnego interfejsu i różnych implementacji znacząco ułatwia testowanie. Dobrze łączy się z innymi wzorcami takimi jak `Strategia`, `Metoda Szablonowa` czy konstrukcyjnymi pozwalającymi na konfigurację obiektu udostępnianego klientom. Fasada dobrze nadaje się na punkt wejścia dla bibliotek, dając klientom dostęp do wysokopoziomowych funkcjonalności i chowając całą wewnętrzną logikę i klasy.
"
permalink: "kotlin-facade-pattern"
comments: true
toc: true
tags:
- design patterns
- Kotlin
- Facade Pattern
- structural design pattern

categories:
- Design Patterns

image: /assets/posts/facade.jpg

---

# Przeznaczenie
Fasada jest bardzo prostym wzorcem, którego zadaniem jest przesłonić szczegóły jakiejś grupy klas — modułu odpowiedzialnego za jakąś funkcjonalność. 

Można to porównać do fasady budynku, która sama w sobie nie pełni żadnej funkcji. Budynek to pokoje, korytarze, schody, instalacje. To fasada wskazuje wejście do budynku, a wyglądem może sugerować jego przeznaczenie.

Zadaniem wzorca jest upraszczanie klientom dostępu do funkcjonalności przesłanianego modułu. Tak, żeby nie musieli znać szczegółów implementacyjnych, zależności między klasami, wiedzieć którą wersję obiektu wstrzykiwać itd.

Przez `moduł` będę tutaj rozumiał jakiś spójny i względnie samodzielny wycinek systemu. Niekoniecznie faktycznie wydzielony moduł w projekcie czy bibliotekę — może to być np. osobny pakiet.

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

Zastanawiałem się, czy klasa `Facade` nie powinna być wewnątrz modułu. Takie zastosowanie Fasady jest często spotykane w bibliotekach — mają sporo wewnętrznej logiki, ale udostępniają jedynie proste API przez pojedynczą klasę i ew. kilka obiektów konfiguracyjnych. Jednak Fasada może służyć do przykrycia kodu, nad którym nie mamy kontroli, tworząc tzw. `Antycorruption Layer`. Wtedy zmieniający się za nią kod nie psuje klientów.

## Abstrakcyjna
Przyjmijmy następującą sytuację: klasa kliencka potrzebuje wyniku działania metody z jednej klasy, jednak ta metoda wymaga przesłania wyniku z innej klasy, która z kolei ma kilka zależności.

```kotlin
val result = E().finishTheWork( // klient potrzebuje wynik tej metody
        D().doAnotherPart( // ale do tego potrzebuje obiekt zwracany przez metodę w klasie D
            A( // która wymaga instancji A
                B( // która zależy od instancji B
                    C() // która potrzebuje instancji C
                )
            )
                .doPartOfWork() // żeby zwrócić obiekt potrzebny w D.doAnotherPart()
        )
    )

// klasy prezentują się następująco:
class A(val b: B) {
    fun doPartOfWork() = "result"
}

class B(val c: C)
class C
class D {
    fun doAnotherPart(input: Any) {}
}

class E {
	// dla uproszczenia w przykładzie użyłem `Any` i zwracam String
	// ale co do zasady może to być dowolny obiekt, np. domenowy
    fun finishTheWork(input: Any) = "finish"
}
```

Wynik metody z klasy `E` jest dostępny dopiero po zbudowaniu całego drzewka zależności klasy `A`, wywołaniu odpowiedniej metody, której wynik jest przekazany do `D.doAnotherPart()`, której z kolei wynik jest potrzebny w metodzie `E.finishTheWork()`. Aby używać tego kodu, trzeba znać sporo szczegółów modułu, co prawdopodobnie wymusza utworzenie i utrzymanie (!!!) dokumentacji.

W jaki sposób Fasada może nam pomóc?
```kotlin
// wywołanie tej samej logiki z fasadą
val result2 = Facade().complexWork() // przyjemniej prawda?

class Facade() {

    // tworzy wymagane instancje wewnątrz
    // frameworki do DI mogą to robić automagicznie
    // ale klienty modułu niekoniecznie powinny być zmuszane do używania tego samego frameworka
    private val c = C()
    private val b = B(c)
    private val a: A = A(b)
    private val d = D()
    private val e = E()

    fun complexWork(): Any { 
		// trochę przypomina to Metodę Szablonową
		// lub nawet Strategię, jeśli instancje mogłyby być wstrzykiwane
		// nic nie stoi na przeszkodzie żeby Fasada wewnątrz korzystała z innych wzorców
		// dla klienta nie będzie to miało znaczenia
        val firstResult = a.doPartOfWork()
        val secondResult = d.doAnotherPart(firstResult)
        return e.finishTheWork(secondResult)
    }
}
```

### Prawo Demeter
Inaczej "zasada minimalnej wiedzy" lub "reguła ograniczania interakcji". Chodzi o to, żeby klasy używały jedynie swoich metod i pól, oraz obiektów stworzonych przez siebie, lub przekazanych bezpośrednio w parametrach metod. Fasada doskonale pomaga w zachowaniu tej dobrej praktyki. 
```kotlin
val a = A(B(C(D())))
// jeśli klient potrzebuje użyć metody z klasy D
// powinien mieć bezpośredni dostęp do instancji D
// a nie przez kolejkę wywołań dostępu do pól innych obiektów
// prywatne pola zapobiegłyby temu, ale publiczne gettery dałyby ten sam efekt
val result = a.b.c.d.theMethod()
// utworzenie metody delegującej wywołanie `theMethod()` w każdej klasie też nie jest dobrym pomysłem
// prawdopodobnie oznacza że `A` ma zbyt wiele odpowiedzialności
// i sprawi że klasy będą ze sobą ściślej związane
```
Patrząc na poprzedni przykład, bez Fasady klient musiał sporo "wiedzieć" o zależnościach. które musiał spełnić, żeby otrzymać oczekiwany wynik. Wszystkie te szczegóły zostały następnie schowane za Fasadą. W pewnym sensie klient może teraz powiedzieć "potrzebuje wyniku, nie obchodzi mnie, jak go uzyskasz".

Wadą ścisłego stosowania tej zasady może być tworzenie wielu metod, które tylko delegują wywołania do innych klas. Jednak jeśli faktycznie klasa posiada wiele delegujących metod, to może robi za dużo i należałoby ją podzielić na mniejsze, bardziej szczegółowe klasy?

## Repozytorium
Szczególnym przypadkiem Fasady jest `Repozytorium`. Pozwala ono na dostęp do obiektów domenowych, ukrywając szczegóły ich przechowywania. Za `Repozytorium` może ukrywać się baza danych, cache, albo nawet komunikacja ze zdalnym serwerem. Albo wszystkie te rzeczy jednocześnie jak w poniższym przykładzie:

```kotlin
// klienty obchodzi jedynie to:
data class User(val id: UUID, val name: String)
interface UserRepository {
    fun getUser(id: UUID): User
}

// `Fake` bo baza jest w pamięci a API zwraca zawsze takie same wyniki
// wersja repozytorium np. pod testy jednostkowe
class FakeUserRepo(
    private val userDb: UserDb = InMemoryUserDb(),
    private val userApi: UserApi = FakeUserApi(),
    private val userCache: UserCache = SimpleCache()
) : UserRepository {

    override fun getUser(id: UUID): User {
        // sprawdź czy obiekt jest w cache
        val cashedUser = userCache.get(id)
        if (cashedUser == null) {
            // jeśli nie, sprawdź w bazie danych
            val dbUser = userDb.get(id)
            if (dbUser == null) {
                // jeśli nie, sprawdź na zdalnym serwerze
                val userDto = userApi.get(id)
                return userDto.toUser().also {
                    // po otrzymaniu obiektu, wrzuć do bazy i do cache
                    userDb.add(it.toEntity())
                    userCache.add(it)
                }
            } else {
                // jeśli obiekt jest w bazie, wrzuć go do cache i zwróć
                return dbUser.toUser().also {
                    userCache.add(it)
                }
            }
        } else {
            return cashedUser
        }
    }

	// przydatne funkcje rozszerzeń, mapujące obiekty z bazy czy API do obiektu domenowego
    private fun UserDto.toUser(): User {
        return User(this.id, this.name)
    }

    private fun UserEntity.toUser(): User {
        return User(this.id, this.name)
    }

    private fun User.toEntity(): UserEntity {
        return UserEntity(this.id, this.name)
    }
}
```
Zwróć uwagę na interfejsy obiektów. Zarówno `UserRepository` jak i `UserDb`, `UserApi` czy `UserCache` mogą mieć różne implementacje, ale dla klienta to bez różnicy.
```kotlin
// instancja UserRepository mogłaby być wstrzykiwana a nie tworzona tutaj
// wtedy pod testy można użyć instancji bez konieczności dostępu do prawdziwej bazy czy klienta HTTP
val userRepo: UserRepository = FakeUserRepo()
// Repozytorium to Fasada na ogólny dostęp do danych
// klienty nie muszą zajmować się cache, zapytaniami HTTP czy bazą danych
val user = userRepo.getUser(UUID.randomUUID())
```
Repozytorium bierze na siebie dbanie o przechowywanie obiektów w cache czy uzupełnianie braków w lokalnej bazie informacjami ze zdalnego serwera. Klient jedynie dostaje dane w najszybszy możliwy sposób. Takie podejście niesamowicie ułatwia testowanie, zwłaszcza jeśli użyje się interfejsów i wstrzykiwania zależności. Pozwala też na zrównoleglenie pracy kilku osób, gdzie jedna zajmuje się np. warstwą UI wyświetlającą dane, a druga rzeźbi ich przechowywanie i dostęp do nich przez HTTP, lub politykę cache. Wypada do szczególnie ciekawie w interdyscyplinarnych zespołach, gdzie programista mobilny może zająć się UI, a backendowy ogarnąć warstwę dostępu do danych w aplikacji, bez znajomości np. Androida.

# Nazewnictwo
Podobnie jak w [poprzednim wpisie](https://asvid.github.io/pl/kotlin-strategy-pattern#nazewnictwo) przychylam się raczej do niedodawania `Facade` do nazwy klasy, która jest fasadą. Doskonale widać to na przykładzie `Repozytorium`, nie ma potrzeby informować klientów, że mają do czynienia jedynie z fasadą. Klienty chcą on wykonać jakąś czynność, na którą pozwala im obiekt, nie muszą wiedzieć, czy jest to Fasada. Inaczej będzie w przypadku Budowniczego, który z definicji powinien mieć metodę `build()`, ale Fasada nie ma wymuszonego przez schemat wzorca API.
# Podsumowanie
Fasada pozwala a ukrycie szczegółów obiektów modułu przed klientami. Zapewnia przestrzeganie `Prawa Demeter`, a użycie ogólnego interfejsu i różnych implementacji znacząco ułatwia testowanie. Dobrze łączy się z innymi wzorcami takimi jak `Strategia`, `Metoda Szablonowa` czy konstrukcyjnymi pozwalającymi na konfigurację obiektu udostępnianego klientom. Fasada dobrze nadaje się na punkt wejścia dla bibliotek, dając klientom dostęp do wysokopoziomowych funkcjonalności i chowając całą wewnętrzną logikę i klasy.

## Zalety
- **prosty interfejs dla klientów** - udostępnia minimalny interfejs, zamiast wymagać od klientów znajomości szczegółów implementacyjnych, które trzeba dokumentować i utrzymywać.
- **zgodność z Prawem Demeter** - klienty "rozmawiają" jedynie z Fasadą, a nie z klasami z wnętrza modułu, nie muszą ich nawet znać a tym bardziej ich szczegółów.
- **ułatwia testowanie** - wymieniając Fasadę z wersji produkcyjnej na testową, można symulować działanie całego modułu i testować klienty w izolowany sposób.
- **refaktoryzacja** - klienty są luźno związane z modułem, bo znają jedynie Fasadę, więc wszelkie zmiany refaktoryzacyjne nie mają na nie wpływu.
- **kontrola nad tym, co wiedzą klienty** - przydatne w przypadku bibliotek, gdzie nie chcemy udostępniać klientom wszystkich klas, a jedynie świadomie wybraną część.
## Wady
- **ograniczenie możliwości klienta** - jeśli klient potrzebuje dostępu do obiektów z wnętrza modułu, Fasada może tego nie umożliwiać. Zmiany wymagań klienta mogą prowadzić do dodawania kolejnych metod w Fasadzie, do momentu aż przestanie być jedynie "prostym interfejsem". Nie jest to duży problem, jeśli mamy kontrolę nad Fasadą i modułem, który przesłania. Jeśli jest to część zewnętrznej biblioteki, ograniczenie może odebrać sens korzystania z biblioteki, lub wymusić kombinowane wyciąganie obiektów z wnętrza modułu.