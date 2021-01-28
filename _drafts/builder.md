---
layout: post
title: "Kotlin Builder Pattern"
date:  "2021-01-20 11:43"
description: "Wzorzec projektowy Builder (budowniczy) jest jednym z popularniejszych i bardziej przydatnych wzorców 
konstrukcyjnych. W tym wpisie postaram się go przybliżyć i pokazać jak ciekawie można go użyć w Kotlinie. Niestety 
często widzę implementacje, które wyglądają jak żywcem przetłumaczone z Javy bez wykorzystania dobrodziejstw języka."
permalink: "kotlin-builder-pattern"
comments: true
toc: true
tags:
- design patterns
- Kotlin
- wzorce projektowe
- Builder Pattern
- wzorce konstrukcyjne
  
image: /assets/posts/codereview-retro/inspection.jpg

---

## Przeznaczenie
Wzorzec Builder służy do uproszczenia tworzenia złożonych obiektów ze skomplikowaną logiką budowania lub z wieloma 
argumentami w konstruktorze. Dzięki zastosowaniu tego wzorca obiekt łatwo może być niemutowalny, bo dostajemy taki 
obiekt, jaki ustawiliśmy przez Buildera, więc nie ma potrzeby na nim już wołać żadnych setterów :)

Builder niejako zdejmuje z użytkownika konieczność zrozumienia wewnętrznej implementacji sposobu tworzenia obiektu i 
gwarantuje jego poprawne utworzenie lub zwrócenie błędu.

Zaletą korzystania z Buildera jest możliwość użycia zmiennej liczby argumentów (`vararg`) w wielu miejscach, bo każda 
metoda Buildera może przyjmować taki argument, a w konstruktorze klasy może być tylko 1 argument tego typu.

Builder rozwiązuje problem tzw. konstruktorów teleskopowych, czyli wariantów konstruktora z rosnącą liczbą argumentów.
```kotlin
constructor(firstName: String): this(firstName, "", 0)
constructor(firstName: String, lastName: String): this(firstName, lastName, 0)
constructor(firstName: String, lastName: String, age: Int): this(firstName, lastName, age)
```
W teorii pozwalają one użyć konstruktora z taką ilością argumentów jaką potrzebujemy, ale w praktyce dodanie nowego 
pola do klasy powoduje konieczność edycji wszystkich konstruktorów. Na szczęście w Kotlinie mamy nazwane argumenty i 
nie trzeba naśladować Javy.

Z mojego doświadczenia wynika, że tego wzorca znacznie częściej się używa, niż pisze samemu, ale i tak warto go znać 
i stosować.

### Przykładowe użycie
Z racji mojego zawodowego skrzywienia przykłady pochodzą ze świata Androida.

#### NotificationBuilder
```kotlin
val notificationBuilder = Notification.Builder(this, "channelId")
notificationBuilder.setContentTitle("Title")
notificationBuilder.setContentText("Content")
notificationBuilder.setSmallIcon(R.mipmap.ic_launcher)
val notification = notificationBuilder.build()
```
Chyba najbardziej tradycyjne użycie wzorca. Konstruktor Buildera przyjmuje 2 argumenty niezbędne do poprawnego 
utworzenia obiektu powiadomienia, reszta pól jest ustawiana przez settery wywołane na instancji Buildera. Na koniec 
wywoływana jest metoda `build()` która zwraca obiekt powiadomienia.

#### Dexter
```kotlin
DialogOnAnyDeniedMultiplePermissionsListener.Builder
		.withContext(context)
		.withTitle("Camera permission")
		.withMessage("Camera permission is needed to take pictures of your cat")
		.withButtonText(android.R.string.ok)
		.withIcon(R.mipmap.my_icon)
		.build()
```
W tym przykładzie metody Buildera są połączone w łańcuch, dlatego że każda z nich zwraca instancję Buildera, czyli 
`this`. Przy odpowiednim nazywaniu metod czyta się takie wywołanie prawie jak zdanie.

#### AlertDialog
```kotlin
val dialog = AlertDialog.Builder(this)
    .apply {
        setTitle("Title")
        setIcon(R.mipmap.ic_launcher)
    }.show()
```
Bardzo Kotlinowe podejście z użyciem `apply`. Co ciekawe, nie ma metody `build()` tylko `show()` która nie tylko 
zwraca obiekt, ale też wyświetla dialog na ekranie. Teoretycznie metoda powinna robić tylko jedną rzecz, ale tutaj 
jej odpowiedzialność została rozszerzona. Prawdopodobnie z powodu częstego błędu utworzenia dialogu i 
późniejszego braku wywołania metody wyświetlenia go.

## Elementy
Builder to w 1 zasadzie klasa pomocnicza. 

### Konstruktor
Wbrew pozorom bardzo ważny jest w niej konstruktor, mimo że często nie 
przyjmuje żadnych argumentów. Powinny się w nim znaleźć wszystkie parametry, bez których nie da się utworzyć 
poprawnego obiektu tworzonego przez Builder.
> Wydaje się to oczywiste, ale na Androidzie swego czasu można było utworzyć poprawne powiadomienie, które nie 
> miało szans zostać wyświetlone przez system. 
> Wystarczyło zapomnieć ustawić tytuł, treść lub ikonę powiadomienia — żadnej z tych rzeczy nie wymagał konstruktor 
> Buildera. 
> System nie rzucał żadnym wyjątkiem przy próbie wyświetlenia niepoprawnie utworzonego powiadomienia...

### Metody
Poza konstruktorem Builder dostarcza metody, którymi możemy ustawić budowany obiekt.
W przypadku braku ustawienia jakiejś właściwości obiektu zastosowana będzie wartość domyślna. Może nią być `null`. 
Dzięki świetnej obsłudze nullability w Kotlinie znacznie lepiej jest wykorzystać domyślny `null` niż np. pusty 
`String` czy magiczną wartość, jak `-1` w miejscu, gdzie spodziewamy się wartości dodatniej. Żeby sprawdzić, czy pole 
zostało w ogóle ustawione wystarczy potem sprawdzić `!= null` lub lepiej z użyciem `?`, zamiast porównywać do jakiejś 
domyślnej wartości nieustawienia konkretnego typu w danym projekcie/klasie.

Metody powinny być możliwe do wywołania w dowolnej kolejności.

No i obowiązkowa metoda `build()` (lub inna sensownie nazwana dla domeny, patrz `show()` dla dialogu) zwracająca 
budowany obiekt.

Jeśli już decydujemy się na użycie tego wzorca, to dobrze ograniczyć możliwość użycia konstruktora wynikowego obiektu tylko przez Builder.

### Weryfikacja argumentów
Jeśli podamy w Builderze niepoprawne dane to kiedy i jak powinniśmy zostać o tym poinformowani?
Podejść jest co najmniej kilka, np.:
1. Jak najszybciej, czyli metoda Buildera przyjmująca argument powinna go weryfikować, ale:
   - co jeśli poprawna wartość jednego argumentu zależy od innego? Jeśli kolejność wywoływania setterów Buildera ma znaczenie to co nam daje ten wzorzec ponad tradycyjne tworzenie obiektu?
   - walidacja w samych setterach może nie być wystarczająca, więc i tak trzeba sprawdzić wartości w metodzie `build()`
2. Metoda `build()` powinna sprawdzić wszystkie argumenty, bo mogą być między nimi zależności i dlatego w ogóle 
   został użyty ten wzorzec, więc:
   - poprawnie stworzony Builder pozwala na wywołanie metod w łańcuchu, więc walidacja w metodzie `build()` wcale nie jest wywoływana jakoś szczególnie później od setterów
   - jesteśmy w stanie zweryfikować zależności między argumentami jeśli istnieją
   - ale jeśli dany Builder nie jest jedyną możliwością tworzenia obiektu, to walidacje trzeba skopiować w każde 
     miejsce, które tworzy dany obiekt
3. To nie Builder powinien weryfikować poprawność, tylko sam tworzony obiekt
   - mamy tutaj zachowane SRP (Single-Responsibility Principle), bo Builder buduje obiekt, a obiekt sam potrafi 
     zwalidować swoje pola
   - nie mamy potrzeby kopiowania walidacji, każdy sposób tworzenia obiektu będzie przechodził przez tą w obiekcie

[Ciekawy wątek w temacie na StackExchange](https://softwareengineering.stackexchange.com/questions/241309/builder-pattern-when-to-fail) , 
gdzie niektórzy sugerują połączenie podejść 2 i 3.
Chodzi o to, żeby Builder veryfikował własne kontrakty, a obiekt swoje. Podany tam jest fajny przykład Buildera, 
który zwraca `String` zawierający kolejne przedziały liczbowe, np "1-2,3-4,5-6". Klasa `String` sama w sobie nie ma jak 
zweryfikować poprawności dodanych przedziałów — to nie jest jej odpowiedzialność, ona tylko przechowuje ciąg znaków.
Za to Builder jest w stanie i powinien w tym przypadku sprawdzić, czy przedziały mają sens i np. nie nachodzą na 
siebie (jeśli jest taka potrzeba). Wtedy metoda np. `addRange(min Int,max Int)` powinna rzucić wyjątek 
`IllegalArgumentException` kiedy `min` jest większe od `max` a metoda `build()` kiedy dodane zostaną przedziały jak 
`1-4` i `2-6`. Chociaż tutaj też pewnie można dyskutować, czy już `addRange()` nie powinno tego sprawdzać.

W każdym razie trzymałbym się zasady, że to obiekt sprawdza swoje kontrakty, a Builder swoje.
[Joshua Bloch](https://pl.wikipedia.org/wiki/Joshua_Bloch) w książce "Java - efektywne programowanie" 
(rozdział 2, temat 2) również sugeruje weryfikację poprawności wartości argumentów dopiero w tworzonym obiekcie, a 
nie w samym Builderze.

## Implementacja

### W stylu Javy
Goły przykład najprostszego możliwego Buildera. W zasadzie Java przekonwertowana na Kotlin, bez użycia żadnych fajerwerków.
Mamy tutaj:
- prywatny konstruktor w `Product` żeby tylko wewnętrzny Builder mógł utworzyć obiekt
- wymagane pole `requiredProperty` w konstruktorze Buildera
- opcjonalne pole ustawiane przez metodę `optionalProperty()`, domyślnie `null` co jest akceptowane przez 
  obiekt `Product`
- metodę `optionalProperty()` zwraca instancję Buildera po to, żeby można było łączyć metody w łańcuch
- metodę `build()` tworzącą `Product` z ustawionymi w Builderze argumentami

```kotlin
class Product private constructor(
    val property: Any,
    val optionalProperty: Any?
) {

    class Builder(private val requiredProperty: Any) {
        private var optionalProperty: Any? = null

        fun optionalProperty(value: Any?): Builder {
            this.optionalProperty = value
            return this
        }

        fun build(): Product {
            return Product(requiredProperty, optionalProperty)
        }
    }
}
```
I wywołanie
```kotlin
val product = Product.Builder("required")
        .optionalProperty("optional")
        .build()
```

### Bardziej Kotlin
Builder w konstruktorze określa wszystkie pola i ich ewentualne domyślne wartości.
Mógłby to być `data class` ale w tym przypadku nie daje to żadnych korzyści. 
Brak domyślnej wartości w konstruktorze Buildera powoduje, że argument `requiredProperty` staje się wymagany.

Użycie `apply` sprawia, że z `optionalProperty()` jest zwracany obiekt Buildera. 
Jest tu też użyty tzw [single-expression function](https://kotlinlang.org/docs/reference/functions.html#single-expression-functions),
czyli brak jawnej deklaracji zwracanego przez metodę typu.

```kotlin
class FancyProduct private constructor(
    val property: Any,
    val optionalProperty: Any?
) {

    class Builder(
        private var requiredProperty: String,
        private var optionalProperty: Any? = null,
    ) {

        fun optionalProperty(value: Any) = apply { this.optionalProperty = value }

        fun build(): FancyProduct {
            return FancyProduct(requiredProperty, optionalProperty)
        }
    }
}
```
Takiego Buildera możemy użyć na kilka sposobów:
- identycznie jak w Javie lub w pierwszym przykładzie w Kotlinie wzorowanym na Javie
- podając oba argumenty (wymagany i opcjonalny) od razu w konstruktorze Buildera
- również podając oba, ale w dowolnej kolejności korzystając z nazwanych argumentów

```kotlin
val fancyProduct = FancyProduct.Builder("required")
    .optionalProperty("optional")
    .build()

val fancyProduct2 = FancyProduct.Builder(
    "required",
    "optional"
).build()

val fancyProduct3 = FancyProduct.Builder(
    optionalProperty = "optional",
    requiredProperty = "required"
).build()
```

### Kotlin DSL
[DSL (Domain Specific Language)](https://en.wikipedia.org/wiki/Domain-specific_language) czyli język programowania utworzony pod konkretną domenę.
Kotlin pozwala na dość proste i przyjemne tworzenie funkcji, które potem umożliwiają w czysto domenowy sposób opisać nasz obiekt.

Builder wygląda prawie tak samo, jak w poprzednim przykładzie, ale pole `optionalProperty` nie jest już prywatne.
Jest to wymagane, jeśli chcemy móc je ustawiać przez DSL.

Doszedł `companion object` który posiada metodę `dslProduct()` za pomocą której będziemy tworzyć nasz obiekt.

```kotlin
class DslProduct private constructor(
        val requiredProperty: Any,
        val optionalProperty: Any?
) {
    companion object {
        inline fun dslProduct(requiredProperty: Any, block: Builder.() -> Unit) =
                Builder(requiredProperty)
                        .apply(block)
                        .build()
    }

    class Builder(
            private val requiredProperty: Any,
            var optionalProperty: Any? = null
    ) {
        fun build() = DslProduct(requiredProperty, optionalProperty)
    }
}
```

Wywołanie wygląda trochę jak JSON albo w najlepszym przypadku JavaScript
```kotlin
val dslProduct = dslProduct("required") {
        optionalProperty = "optional"
    }
```
Dla tak prostego obiektu nie wygląda to szczególnie zachęcająco, ale jeśli nasz Builder jest kompozycją innych 
Builderów, to zaczyna robić się ciekawie.
Przykład poniżej pochodzi z mojej aplikacji, który zrealizowałem przy pomocy [DslMaker](https://kotlinlang.org/docs/reference/type-safe-builders.html). 
To część standardowej biblioteki Kotlina i dodatkowo zadba o zasięgi wewnętrznych Builderów.
Adres, położenie i godziny otwarcia również wykorzystują Buildery i DSL do stworzenia swoich obiektów.

```kotlin
val shop = shop("ID") {
    address = address {
        cityName = "Poznań"
        streetName = "ul. Półwiejska"
        streetNumber = "123/2"
    }
    location = location {
        lat = 53.12
        lng = 23.4
    }
    openHours = openHours {
        weekDay = "6:00-22:00"
        saturday = "7:00-23:00"
        sunday = "closed"
    }
    features(
            Feature.Bakery,
            Feature.Atm
    )
}
```
Pełny przykład z użyciem DslMaker jest -> [tutaj](https://gist.github.com/asvid/5fed8dd3c831c2a72744e8ffe8f7dc0f) <-

Podejście DSL wymaga jednak napisania pewnego boilerplate, który na pierwszy rzut oka nie wygląda zbyt przyjaźnie.

## Alternatywa
Nazwane parametry w konstruktorze i domyślne wartości parametrów w tworzonym obiekcie mogą w pewnym sensie dawać podobny efekt co zastosowanie Buildera.
Sprawdzi się to raczej do mniej rozbudowanych obiektów. Warto też zadbać, żeby domyślne wartości tworzyły sensowny obiekt z punktu widzenia domeny, 
a nie tylko "żeby się kompilowało".
```kotlin
val person = Person(
        firstName = "Adam",
        lastName = "Świderski",
        address = Address(
                cityName = "Poznań",
                streetName = "ul. Półwiejska",
                streetNumber = "123/1",
                country = "Poland",
                postalCode = "60-000"
        ),
        contact = Contact(
                workEmail = "adam@work.email",
                workPhoneNumber = "+48 123112312",
                privateEmail = "adam@private.email"
        ),
)
```

Wygląda nawet podobnie do DSL, ale bez konieczności używania adnotacji, pisania dodatkowych metod etc.
Warto zwrócić uwagę na pola takie jak `val height: Float? = null` - uznałem, że nie jest to niezbędna informacja do utworzenia sensownego obiektu `Person`,
dlatego domyślna wartość tego pola to `null`. Tak `null` a nie np. `0.0` czy `-1`, albo jakaś magiczna stała `DEFAULT_HEIGHT`.

W blokach `init` są sprawdzane wartości podanych argumentów (patrz wyżej `Weryfikacja argumentów`). Wrzucenie złego 
typu argumentu zostanie wychwycone już na etapie kompilacji, ale obiekt może mieć jakieś domenowe wymagania, jak np. 
że wzrost jest zawsze dodatni, a rozmiar buta, o ile został podany, nie może być mniejszy niż 4.
```kotlin
data class Person(
    val firstName: String,
    val lastName: String,
    val address: Address,
    val contact: Contact,
    val height: Float? = null,
    val shoeSize: Float? = null,
) {
    init { 
        height?.let { require(0f < it) { "height is always greater than 0" } }
        shoeSize?.let { require(4f <= it) { "smallest standard shoe size is 4" } }
    }
}

data class Address(
    val country: String,
    val cityName: String,
    val streetName: String,
    val streetNumber: String,
    val postalCode: String,
    val district: String? = null,
)

data class Contact(
    val workPhoneNumber: String,
    val workEmail: String,
    val privatePhoneNumber: String? = null,
    val privateEmail: String? = null,
) {
    init {
        require(workPhoneNumber.isValidPhoneNumber())
        require(workEmail.isValidEmail())
        require(privatePhoneNumber?.isValidPhoneNumber() ?: true)
        require(privateEmail?.isValidEmail() ?: true)
    }
}
```

## Podsumowanie
Builder jest całkiem przydatnym wzorcem i na pewno w trakcie pracy się z nim spotkasz. Dobrze wiedzieć z czego się 
składa i w jakich przypadkach warto go użyć - bo nie będzie to zawsze. Kotlin pozwala na znaczne uproszczenie 
boilerplate Buildera i wykorzystanie sposobów takich jak DSL i nazwane argumenty i wartości domyślne na osiągnięcie 
podobnego rezultatu bez pisania dodatkowego kodu.

W literaturze nt. wzorców projektowych można spotkać implementację Buildera z wykorzystaniem elementów takich jak: 
Director i ConcreteBuilder. Dopóki nie trzeba przekazywać jako argumenty do jakiegoś Buildera instancji innych 
Builderów aby w generyczny sposób tworzyć rozbudowane obiekty - powyższe przykłady i opis będą w zupełności 
wystarczające. A jeśli najdzie Cię potrzeba Builder-cepcji - może warto skorzystać z innego wzorca zamiast komplikować 
sobie życie :)