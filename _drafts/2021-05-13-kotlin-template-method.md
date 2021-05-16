---
layout: post
title: "Metoda szablonowa w Kotlinie"
date:  "2021-05-13 11:43"
description: ""
permalink: "kotlin-template-method"
comments: true
toc: true
tags:
- design patterns
- Kotlin
- Template Method
- behavioral design pattern

categories:
- Design Patterns

image: /assets/posts/abstract_factory.jpg

---

# Przeznaczenie
Metoda szablonowa to bardzo prosty wzorzec, pozwalający oddzielić to, co stałe od tego, co zmienne w rodzinie klas.  Polega na utworzeniu abstrakcyjnej klasy nadrzędnej, zawierającej kolejne kroki jakiegoś algorytmu i pozwoleniu klasom dziedziczącym z niej nadpisywać poszczególne kroki, ale nie sam algorytm, który je wykorzystuje.

Pomyśl o tym jak o pizzy — kroki zą z grubsza takie same, najpierw ciasto, potem sos, dodatki i do pieca. Różne rodzaje pizzy mogą mieć różne ciasto, sos, dodatki lub czas wypiekania (chyba nie wiem :) ), ale kolejność kroków i same kroki są dla każdego rodzaju takie same. Możemy więc uznać, że istnieje abstrakcyjna `Pizza` z metodą `zrób()`, ale dziedzicząca z niej klasa `Hawajska` czy `Pepperoni` nadpisze metodę dodającą dodatki bez zmiany samego algorytmu robienia pizzy.

Mamy tutaj do czynienia z odwróceniem sterowania, bo to klasa nadrzędna `Pizza` wywołuje metody klasy podrzędnej, a nie odwrotnie jak to zwykle bywa.

Ze wzorcem tym można się spotkać w bibliotekach, gdzie twórcy pozwalają nam rozszerzać klasy biblioteki i nadpisywać niektóre metody, jednak mają kontrolę nad kolejnością ich wykonywania.  

# Przykłady implementacji
## Podstawowy wzorzec
Zacznę od generycznego przykłądu, pokazującego na czym polega wzorzec Metody Szablonowej:
```kotlin
abstract class AbstractClass { // klasa nadrzędna

	// tej metody nie da się nadpisać, bo nie jest `open`
    fun templateMethod() { // tytułowa Metoda Szablonowa, czyli algorytm z listą kroków
        println("running template method")
        primitiveOperation1() // wywołania kolejnych kroków algorytmu
        primitiveOperation2()
        primitiveOperation3()
    }

    abstract fun primitiveOperation1() // krok algorytmu do nadpisania przez konkretną klasę
    
    private fun primitiveOperation2() { // krok algorytmu którego nie chcemy nadpisywać
        println("doing abstract operation 2")
    }

    abstract fun primitiveOperation3() // kolejny krok algorytmu
}

class ConcreteClass : AbstractClass() { // konkretna klasa
    override fun primitiveOperation1() { // implementacja dla konkretnej klasy
        println("doing concrete operation 1")
    }

    override fun primitiveOperation3() {
        println("doing concrete operation 3")
    }
}

class AnotherConcreteClass : AbstractClass() { // kolejna konkretna klasa
    override fun primitiveOperation1() {
        println("doing concrete operation 1") // implementacja taka sama jak w `ConcreteClass`, niefajne powielanie kodu
    }

    override fun primitiveOperation3() {
        println("doing another concrete operation 3")
    }
}

```
Mamy tutaj `templateMethod()` w klasie abstrakcyjnej, czyli nasz algorytm albo inaczej listę kroków, czyli wywołań metod zawierających poszczególne operacje prymitywne. Te operacje mogą posiadać domyślną implementację już w klasie abstrakcyjnej lub wymuszać dodanie implementacji w klasach dziedziczących.
Metoda Szablonowa nie może być nadpisana przez klasy dziedziczące. Byłoby to możliwe, gdyby ta metoda była oznaczona jako `open` ale domyślnie Kotlin stawia na enkapsulację, więc metody i klasy są `final`. Nadpisanie `templateMethod()` oznaczałoby zmianę algorytmu, a jest on powodem zastosowania tego wzorca. Jeśli istnieje potrzeba zmiany algorytmu, może lepiej zastosować wzorzec `Strategia`.

## Pizza
Przejdźmy do bardziej obrazowego przykładu ze wstępu: Pizzy
```kotlin
abstract class Pizza { // klasa bazowa dla wszystkich rodzajów Pizzy

    fun make() { // kroki dla każdej pizzy są takie same
        makeDough()
        applySauce()
        applyAddons()
        bake()
    }

    open fun bake() { 
		// domyślne implementacje kroków, pozwalają nadpisywać tylko te wyróżniające się
        println("baking for 20 minutes")
    }

    open fun applyAddons() {
        println("adding cheese")
    }

    open fun applySauce() {
        println("applying tomato sauce")
    }

    open fun makeDough() {
        println("making 30cm dough")
    }
}

class Pepperoni : Pizza() { // konkretny rodzaj pizzy
    override fun applyAddons() { // nadpisuje nakładanie dodatków zgodnie z przepisem
        println("adding salami")
        println("adding onion")
        println("adding cheese")
    }
	// ale nie kontroluje procesu przygotowywania samej pizzy
	
	// pozostałe metody mają zostawioną domyslną implementację
}
class BigPepperoni : Pizza() { 

    override fun applyAddons() { // taka sama implementacja jak dla poprzedniej klasy
        println("adding salami")
        println("adding onion")
        println("adding cheese")
    }

    override fun makeDough() { // jedyna różnica do zwykłej Pepperoni
        println("making 50cm dough")
    }
}

```
Mamy więc klasę `Pizza` która zawiera wspólne dla wszystkich rodzajów pizzy kroki przygotowania. Poszczególne rodzaje pizzy nadpisują wyłącznie te kroki, które różnią się od domyślnych, np. inny rozmiar lub dodatki czy sos. Powstaje tutaj jednak problem ze współdzieleniem implementacji między klasami `Pepperoni` i `BigPepperoni` - mają takie same składniki, ale `BigPepperoni` zmienia także rozmiar ciasta. Wydaje się, że `BigPepperoni` powinna po prostu dziedziczyć po `Pepperoni` i nadpisywać tylko metodę `makeDough()`, ale wtedy klasa `Pepperoni` musiałaby być `open` i wprowadzałoby to kolejny poziom dziedziczenia. Łatwo sobie wyobrazić postępującą eksplozję klas, jeśli każdy rodzaj pizzy występowałby w 3 rozmiarach i z 2 sosami do wyboru. Zaczyna to przypominać powód istnienia wzorca `Factory`.

### Pizza Lambdy
Zamiast nadpisywać metody interfejsu klasy bazowej `Pizza` możemy spróbować wykorzystać lambdy przekazywane w konstruktorze:
```kotlin
abstract class Pizza( // konstruktor klasy bazowej przyjmuje lambdy, ale domyślnie mają już implementację
    private val makeDough: () -> Unit = {
        println("making 30cm dough")
    },
    private val applySauce: () -> Unit = {
        println("applying tomato sauce")
    },
    private val applyAddons: () -> Unit = {
        println("adding cheese")
    },
    private val bake: () -> Unit = {
        println("baking for 20 minutes")
    }
) {

    fun make() { // metoda szablonowa bez zmian
        makeDough() // wywołanie lambdy z konstruktora zamiast nadpisywalnej metody
        applySauce()
        applyAddons()
        bake()
    }
}

class Pepperoni : Pizza( // konkretna klasa
    applyAddons = { // przekazuje lambdę w konstruktorze zmieniając domyślną implementację
        println("adding salami")
        println("adding onion")
        println("adding cheese")
    }
)

class BigPepperoni : Pizza(
    applyAddons = { // znów powtórzona implementacja
        println("adding salami")
        println("adding onion")
        println("adding cheese")
    },
    makeDough = { // nadpisana lambda rozmiaru ciasta
        println("making 50cm dough")
    }
)
```
Wygląda już badziej kotlinowo :) W książce [^effective_java] Joshua Bloch wspomina, że po wprowadzeniu lambd do Javy wzorzec `Metody Szablonowej` stracił na znaczeniu, bo wygodniej jest wstrzykiwać zachowanie jako lambdy w konstruktorze niż tworzyć konkretne klasy nadpisujące wybrane metody.

Nadal występuje jednak problem z powielaniem implementacji. Można taką implementację zamknąć do jakiejś zmiennej i przekazywać klasom w konstruktorze.
```kotlin
val pepperoniAddons: () -> Unit = {
    println("adding salami")
    println("adding onion")
    println("adding cheese")
}

class Pepperoni : Pizza(
    applyAddons = pepperoniAddons
)

class BigPepperoni : Pizza(
    applyAddons = pepperoniAddons,
    makeDough = {
        println("making 50cm dough")
    }
)
```
Działa. Jednak przekazywanie wszędzie `() -> Unit` jest średnio bezpieczne i ułatwia powstawanie błędów, chociażby przez pomylenie kolejności argumentów.

### Pizza interfejsy
Zakładając, że nasze metody nie mają nic zwracać, a tylko wykonywać operacje na obiekcie, bezpieczniej będzie stworzyć konkretne interfejsy odpowiadające krokom algorytmu.
```kotlin
interface DoughMaker { // interfejs kroku algorytmu wytwarzania pizzy
	operator fun invoke() { // pozwala wywołać obiekt implementujący interfejs jak lambdę
		println("making dough") // domyślna implementacja, Kotlin pozwala na to w interfejsach
	}
}

interface SauceApplier {
	operator fun invoke() {
		println("applying sauce")
	}
}

interface AddonsApplier {
	operator fun invoke() {
		println("applying addons")
	}
}

interface Baker {
	operator fun invoke() {
		println("baking")
	}
}
```
Mając takie interfejsy, abstrakcyjna klasa `Pizza` będzie wyglądać następująco:
```kotlin
abstract class Pizza(
    private val makeDough: DoughMaker = object : DoughMaker { // dzięki różnym typom nie da się pomylić kolejności argumentów
        override fun invoke() {
            println("making 30cm dough") // nadpisana domyślna implementacja z interfejsu
        }
    },
    private val applySauce: SauceApplier = object : SauceApplier {
        override fun invoke() {
            println("applying tomato sauce")
        }
    },
    private val applyAddons: AddonsApplier = object : AddonsApplier {
        override fun invoke() {
            println("adding cheese")
        }
    },
    private val bake: Baker = object : Baker {
        override fun invoke() {
            println("baking for 20 minutes")
        }
    }
) {

    fun make() { // metoda szablonowa wygląda identycznie jak poprzednio
        makeDough() // jednak nie wywołujemy tutaj lambdy, a metodę `invoke()` konkretnego obiektu
        applySauce()
        applyAddons()
        bake()
    }
}

```
Konkretne klasy mogą mieć wstrzykiwane te same obiekty implementujące interfejs, więc znika problem powielania kodu w wielu miejscach:
```kotlin
object PepperoniAddonsAplier : Pizza.AddonsApplier { // wspólny dla wszystkich rozmiarów Pepperoni
    override fun invoke() {
        println("adding salami")
        println("adding onion")
        println("adding cheese")
    }
}

object BigPizzaDoughMaker : Pizza.DoughMaker { // może być użyty z innymi rodzajami pizzy
    override fun invoke() {
        println("making 50cm dough")
    }
}

class Pepperoni : Pizza(
    applyAddons = PepperoniAddonsAplier
)

class BigPepperoni : Pizza(
    applyAddons = PepperoniAddonsAplier,
    makeDough = BigPizzaDoughMaker
)
```

Pozwala to też na łatwe stworzenie zupeniłe customowej Pizzy, bez potrzeby tworzenia nowej klasy:
```kotlin
val customPizza = object : Pizza(
	applySauce = object : SauceApplier {
		override fun invoke() {
			println("adding super sauce")
		}
	},
	makeDough = object : DoughMaker {
		override fun invoke() {
			println("making super dough 48cm")
		}
	},
	applyAddons = object : AddonsApplier {
		override fun invoke() {
			println("no addons, its super by itself")
		}
	},
	bake = object : Baker {
		override fun invoke() {
			println("I like it raw")
		}
	}
) {}
customPizza.make()
```

Ale czy to jeszcze `Metoda Szablonowa` czy już `Strategia`?

## Active Record
Przykład zaczerpnięty ze świata Ruby (którego w ogóle nie znam), gdzie wziął się od wzorca zaproponowanego przez Martina Fowlera w książce [^fowler]. Ogólnie chodzi o to, że mamy klasę - model w rozumieniu MVC, która oprócz pól posiada metody pozwalające ten model zapisać czy usunąć, np. w bazie danych. Więcej o [^active_record].

W `Ruby on Rails` taki model ma dodatkowo sporo callbacków [^ruby_active_record], tzw hooków, które są wywoływane np. przed zapisem lub w przypadku błędu. I własnie do takiego zastosowania wzorzec `Metoda Szablonowa` nadaje się idealnie. W Kotlinie możnaby to zaimplementować tak:
```kotlin
abstract class ActiveRecord { // bazowa klasa ActiveRecord
	
	// metoda szablonowa, wywołująca hooki w odpowiedniej kolejności
    fun save() { // generyczna metoda zapisu obiektu w bazie danych
        println("saving record $this")
        this.beforeSave() // wywołanie hooka
        val isSuccess = DB.save(this) // sam zapis do bazy może się udać lub nie
        if (isSuccess) {
            this.afterSave() // kolejny hook na sukces zapisu
        } else {
            this.failedSave() // i hook w razie błędu
        }
    }

    open fun beforeSave() { // generyczny hook z domyślną implementacją, otwarty do nadpisania
        // NOOP
        println("NOOP beforeSave()")
    }

    open fun afterSave() {
        // NOOP
        println("NOOP afterSave()")
    }

    open fun failedSave() {
        // NOOP
        println("NOOP failedSave()")
    }
}

object DB { // jakiś mock bazy danych
    fun save(record: ActiveRecord): Boolean {
        println("DB is saving record: $record")
        // tutaj byłby zapis i zwrócenie true jeśli się powiodło, false jeśli nie
        return true
    }
}

class User(private var username: String) : ActiveRecord() { // konkretny model dziedziczący po ActiveRecord
	
    override fun beforeSave() { // nadpisanie hooka
        println("$this beforeSave()")
        sanitizeRecord() // wywołanie ewentualnego poprawienia danych przed zapisem
    }

    private fun sanitizeRecord() { // poprawienie danych w modelu przed zapisem
        println("$this sanitizeRecord()")
        username.trim()
        username = username.filter { it.isLetter() }
    }
}

class Post(val text: String) : ActiveRecord() // inny model ActiveRecord, ale bez nadpisania hooków
```
Mając tak skonstruowany `ActiveRecord` z hookami można łatwo wpiąć np. logowanie modyfikacji modelu, albo obsługę błędów zapisu czy właśnie poprawiania danych w modelu przed samym zapisaniem. Na podobnej zasadzie działa `JUnit`, gdzie oznaczamy metody-hooki adnotacjami `@BeforeEach` czy `@AfterAll`, żeby wywołać je przed każdym lub po wszystkich testach w klasie.

Sam `ActiveRecord` jest jednak czasami określany jako [^active_record_antipatern] i są ku temu całkiem dobre powody: naruszenie SRP (jedna klasa do operacji na bazie danych, walidacji modelu, przechowywania danych etc), bezpośrednie mapowanie struktury bazy na obiekt, problemy z testowaniem. Ale jak już wspominałem - Ruby to nie mój świat, a ten post nie jest o `ActiveRecord`, po prostu wzorzec `Metody Szablonowej` dobrze pasuje do takiego zastosowania. 

# Podsumowanie
Wzorzec `Metody Szablonowej` jest dosyć prosty w swojej konstrukcji i chyba sam go stosowałem nawet nie zdając sobie sprawy, że tak się właśnie nazywa. Pozwala, zgodnie z nazwą, stworzyć szablon działania metody, jednocześnie wymuszając (lub tylko zezwalając na) nadpisanie poszczególnych kroków algorytmu w konkretnej implementacji (patrz Pizza).
Mimo swojej prostoty i archaicznej konstrukcji nadal znajduje zastosowanie we współczesnych projektach, szczególnie bibliotekach. Ma jednak trochę ograniczeń, których rozwiązanie przez wstrzykiwanie lambd czy całych obiektów zaczyna zacierać różnicę między `Metodą Szablonową` a `Strategią`. Niekoniecznie jest to złe, ale należy sobie zdawać sprawę, że może nie jest to najlepszy wzorzec w każdym przypadku gdzie mamy rodzinę klas z pewnymi podobnymi zachowaniami. 

## Zalety
- stosunkowo prosta implementacja rodziny klas różniących się tylko częścią zachowań
- ciekawe wykorzystanie w `ActiveRecord`
- może być użyteczny do implementacji hooków
- w przypadku tworzenia biblioteki gdzie klienty rozszerzają jej elementy, zapewnia kontrolę nad kolejnością wywyoływania metod nadpisanych przez klienta

## Wady
- potencjalna eksplozja klas
- klasa pochodna musi trochę wiedzieć o rodzicu, znać algorytm i wiedzieć które metody nadpisać, gdzie użyć `super()` a gdzie nie
- dziedziczenie zamiast kompozycji, gdzie zmiana na wstrzykiwanie lambd w konstruktorze w zasadzie zmienia wzorzec w `Strategię`

---
[^effective_java]:["Java - efektywne programowanie"](https://books.google.pl/books/about/Effective_Java.html?id=ka2VUBqHiWkC&redir_esc=y) 
[^fowler]:[Architektura systemów zarządzania przedsiębiorstwem. Wzorce projektowe](https://books.google.pl/books?id=FyWZt5DdvFkC&q=active+record&pg=PT187&redir_esc=y)
[^active_record]:[Active Record](https://en.wikipedia.org/wiki/Active_record_pattern)
[^ruby_active_record]:[Active Record w Ruby](https://guides.rubyonrails.org/active_record_callbacks.html)
[^active_record_antipatern]:[antywzorzec](https://www.mehdi-khalili.com/orm-anti-patterns-part-1-active-record)