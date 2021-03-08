---
layout: post
title: "Kotlin Abstract Factory"
date:  "2021-03-08 11:43"
description: "
The factory of factories, 'Abstract Factory' makes creating objects that are part of some 'family' easy. This is basically another layer over concrete factories and delivering to the client factory instance creating objects of a certain variant.
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

image: /assets/posts/abstract_factory.jpg

---

# Purpose

Nazwa tego wzorca nie do końca sugeruje, czym się wyróżnia od innych wzorców konstrukcyjnych, takich jak Builder czy
Factory. W Abstract Factory nie chodzi o dostarczenie pojedynczej instancji — a całej `rodziny powiązanych obiektów`.
Nadal brzmi to podobnie jak zwykłe Factory, które może produkować np. obiekty kontrolek GUI. Żeby
otrzymać `Abstract Factory` musimy właśnie dodać kolejną warstwę abstrakcji, i stworzyć mechanizm produkujący obiekty
kontrolek GUI, ale w wariantach dla np. Linuxa, Windowsa i MacOS, które nadal będą miały generyczne API dla klienta
wzorca.

`Abstract Factory` to **fabryka fabryk**, dostarczająca klientom wybraną wersję fabryki spośród tworzących obiekty o
takim samym interfejsie. Rodziny obiektów będą powiązane przez konkretną fabrykę, która je buduje. Trzymając się tematu
GUI, będziemy mieć rodziny kontrolek dla Windowsa, Linuxa i MacOS — zapobiegnie to sytuacji użycia przycisku z MacOS
obok pola tekstowego z Windowsa. Klienta nie będzie jakoś szczególnie interesowało to, z której rodziny (i z którego
dokładnie factory) otrzyma obiekt, ponieważ jego API będzie takie samo. I na tym polega `Abstrakcja` tej fabryki, na **
udostępnieniu interfejsu fabryk rodzin obiektów**.

Przydatne informacje przed lekturą tego posta:

- [Static Factory Method](https://asvid.github.io/pl/kotlin-static-factory-methods)
- ["Typowe" Factory Method](https://asvid.github.io/pl/kotlin-factory-method)

# Implementation examples

Sam miałem problem to zrozumieć zanim nie zobaczyłem kodu i jakichś diagramów, dlatego nejlepiej od razu przejść do
przykładów.

## Basic

Książkowy przykład pochodzący z "Wzorców Programowania"[^design_patterns] "bandy czworga". Jest to dość nomen omen
abstrakcyjny przykład, jednak dobrze pokazuje ideę wzorca `Abstract Factory`.

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

Każdy wariant ma osobną konkretną fabrykę, dostarczającą oba produkty w odpowiednim dla siebie wariancie. `internal`
gwarantuje niewyciekanie klasy poza moduł. Konkretnych fabryki są dobrym kandydatem do Singletona, stąd `object`.

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

Fabryka abstrakcyjna, czyli zwracanie odpowiedniej konkretnej fabryki (odpowiedniego wariantu) ale schowanej za
generycznym interfejsem `Factory`. W ten sposób klienty nie muszą nawet znać implementacji konkretnej fabryki. Klienty
wiedzą tylko, że dostaną `Factory` i że będzie ono potrafiło zbudować `ProductA` i `ProductB`. Metody `getFactory()`
jest tutaj tzw. `top-level function` bo znajduje się poza jakąkolwiek klasą, ale może Być częścią np. `companion object`
interfejsu Factory. Jednak sam interfejs Factory niekoniecznie powinien znać swoje implementacje, na szczęście może mieć
pusty `companion object`, który dostanie w odpowiednim miejscu `extension function`. Szerzej pisałem o
tym [tutaj](https://asvid.github.io/pl/kotlin-builder-pattern)

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

class ConcreteFactory1<< (O,#FF0077) Variant1 >> implements Factory
class ConcreteFactory2<< (O,#FF7700) Variant2 >> implements Factory

Factory::createProductA-right[bold]-|>ProductA 
Factory::createProductB-right[bold]-|>ProductB

class AbstractFactory{ 
+getFactory(variant) : Factory 
}

AbstractFactory::getFactory-->Factory

@enduml 
{% endplantuml %}

## GUI elements family

Konkretny przykład wykorzystujący wspomniane już rodziny kontrolek GUI. Nadal wydaje mi się nie do końca "życiowy" ale
mi osobiście najbardziej pomógł zrozumieć do czego `Abstract Factory` właściwie może posłużyć.

Mamy więc kilka kontrolek GUI

```kotlin
interface View
abstract class Button : View
abstract class Image : View
abstract class GridView : View
```

Oraz interfejs fabryki, który je dostarcza

```kotlin
interface ViewFactory {
    // typy znane Factory
    enum class OS {
        Linux, Windows
    }

    // metody dostarczające kontroli, niezależne od OS
    fun createButton(): Button
    fun createImage(): Image

    companion object {
        // "statyczna" metoda zwracająca konkretne factory
        fun createFactory(os: OS): ViewFactory {
            return when (os) {
                OS.Linux -> LinuxViewViewFactory
                OS.Windows -> WindowsViewViewFactory
            }
        }
    }
}
```

Interfejs `ViewFactory` jest implementowany przez konkretne fabryki dla Windowsa i Linuxa

```kotlin
object LinuxViewViewFactory : ViewFactory {
    // konkretne typy zwracane dla konkretnego OSa
    // internal zapobiega wyciekaniu ich
    internal class LinuxButton : Button()
    internal class LinuxImage : Image()

    // klienty widzą tylko generyczny interfejs kontrolki
    override fun createButton(): Button = LinuxButton()
    override fun createImage(): Image = LinuxImage()
}

object WindowsViewViewFactory : ViewFactory {
    internal class WindowsButton : Button()
    internal class WindowsImage : Image()

    override fun createButton(): Button = WindowsButton()
    override fun createImage(): Image = WindowsImage()
}
```

Powyższy kod można użyć tak:

```kotlin
val winViewFactory = ViewFactory.createFactory(ViewFactory.OS.Windows)
val button = winViewFactory.createButton()
```

Ale w ten sposób, interfejs fabryki abstrakcyjnej musi znać swoje implementacje — czyli konkretne fabryki kontrolek.
Można do tego podejść inaczej i nie tworzyć w `ViewFactory` `companion object` który zwraca fabryki, tylko stworzyć nową
klasę, która zajmie się dostarczaniem konkretnych kontrolek przy pomocy każdej dostępnej fabryki kontrolek.

```kotlin
// klasa przesłaniająca używane fabryki ale implementująca ten sam interfejs
class ViewCreator(os: ViewFactory.OS) : ViewFactory {

    // instancja ViewCreatora dostarcza kontrolki tylko dla konkretnego OS-a
    private val factory = when (os) {
        ViewFactory.OS.Linux -> LinuxViewViewFactory
        ViewFactory.OS.Windows -> WindowsViewViewFactory
    }

    // metody zwracająca generyczne kontrolki, 
    // z factory na podstawie argumentu konstruktora
    override fun createButton(): Button = factory.createButton()
    override fun createImage(): Image = factory.createImage()
}

// użycie
val linuxButton = ViewCreator(ViewFactory.OS.Linux).createButton()
```

Klienty nadal nie znają konkretnej fabryki kontrolek, ale teraz również interfejs `ViewFactory` nie musi ich znać. Nie
ma też potrzeby rozszerzania `companion object`. Po prostu tworzymy instancję fabryki pod konkretny OS, która pod spodem
używa konkretnych fabryk zgodnych z przekazanym OS-em.

Łatwo zauważyć, że wraz ze wzrostem liczby kontrolek sporo kodu będzie wyglądać niemal identycznie, w zasadzie proxując
wywołania metod do konkretnych fabryk. Ciekawym i mało ortodoksyjnym pomysłem jest użycie jednej metody `make<Typ>()`
zamiast osobnych metod dla każdego typu kontrolki. Wtedy `ViewCreator` mógłby wyglądać tak:

```kotlin
// nie implementuje już interfejsu `ViewFactory`
class ViewCreator(os: ViewFactory.OS) {
    // pole musi być publiczne, bo jest wykorzystywane w metodzie `inline`
    val factory = when (os) {
        ViewFactory.OS.Linux -> LinuxViewViewFactory
        ViewFactory.OS.Windows -> WindowsViewViewFactory
    }

    // `inline` i `reified` pozwalają na użycie typu nieznanego w czasie kompilacji
    inline fun <reified T : View> make(): T = when (T::class) {
        Button::class -> factory.createButton() as T
        Image::class -> factory.createImage() as T
        else -> throw IllegalArgumentException()
    }
}

// użycie
val windowsButton = ViewCreator(ViewFactory.OS.Windows).make<GridView>()
```

Słówka kluczowe `inline` i `reified` to spokojnie temat na osobny post. Użycie ich tutaj pozwala na przekazanie typu
rozszerzającego `View` i sprawia, że metoda `make()` zwróci właśnie taki typ. Dzięki temu nie trzeba dla każdej nowej
kontrolki dodawać nowej metody typu `createButton()`. Wystarczy, że nowa kontrolka rozszerza `View` i nowy przypadek
w `when` zostanie obsłużony. Co dokładnie daje `inline`? Kopiuje kod w miejsce wywołania, dosłownie :) i właśnie dlatego
wszystkie wykorzystywane przez metody `inline` pola muszą być publicznie dostępne. Przy takiej implementacji
metody `make()` potrzebujemy użyć `reified`, żeby móc wykorzystać **klasę przekazanego typu <T>** do znalezienia
właściwej metody w konkretnych `ViewFactory`. A `reified` można użyć tylko w metodach `inline`, bo "wklejenie" kodu
metody w miejsce wywołania pozwala w trakcie kompilacji poznać dokładny typ. Po wywołaniu odpowiedniej metody,
np. `createButton()` należy wynik jeszcze zrzutować na `<T>`, ponieważ to `T` jest spodziewanym typem zwracanego obiektu
a nie `Button`, nawet jeśli `T::class == Button::class`.

Kopiowanie kodu powoduje puchnięcie plików, np. wywołanie metody `make()` 2x:

```kotlin
val windowsButton = ViewCreator(ViewFactory.OS.Windows).make<Button>()
// w ramach jednego moduły można niestety przekazać też typ klasy `internal`
// więc typem zwracanym nie będzie generyczny Button a konkretny WindowsButton
val windowsButton2 = ViewCreator(ViewFactory.OS.Windows).make<WindowsViewViewFactory.WindowsButton>()
```

Spowoduje wygenerowanie bytecode odpowiadającemu Javie:

```java
// początek dla pierwszego Buttona
ViewCreator this_$iv=new ViewCreator(OS.Windows);
int $i$f$make=false;
KClass var4=Reflection.getOrCreateKotlinClass(Button.class);
Button var10000;
View var9;
Image var10;
if(Intrinsics.areEqual(var4,Reflection.getOrCreateKotlinClass(Button.class))){
var10000=this_$iv.getFactory().createButton();
if(var10000==null){
throw new NullPointerException("null cannot be cast to non-null type abstract_factory.family.Button");
}

var9=(View)var10000;
}else{
if(!Intrinsics.areEqual(var4,Reflection.getOrCreateKotlinClass(Image.class))){
throw(Throwable)(new IllegalArgumentException());
}

var10=this_$iv.getFactory().createImage();
if(var10==null){
throw new NullPointerException("null cannot be cast to non-null type abstract_factory.family.Button");
}

var9=(View)((Button)var10);
}
// koniec dla pierwszego Buttona
Button windowsButton=(Button)var9;
// ------------------------------------------------------------------------------------------

// początek dla drugiego Buttona     
ViewCreator this_$iv=new ViewCreator(OS.Windows);
int $i$f$make=false;
KClass var5=Reflection.getOrCreateKotlinClass(WindowsButton.class);
if(Intrinsics.areEqual(var5,Reflection.getOrCreateKotlinClass(Button.class))){
var10000=this_$iv.getFactory().createButton();
if(var10000==null){
throw new NullPointerException("null cannot be cast to non-null type abstract_factory.family.WindowsViewViewFactory.WindowsButton");
}

var9=(View)((WindowsButton)var10000);
}else{
if(!Intrinsics.areEqual(var5,Reflection.getOrCreateKotlinClass(Image.class))){
throw(Throwable)(new IllegalArgumentException());
}

var10=this_$iv.getFactory().createImage();
if(var10==null){
throw new NullPointerException("null cannot be cast to non-null type abstract_factory.family.WindowsViewViewFactory.WindowsButton");
}

var9=(View)((WindowsButton)var10);
}
// koniec dla drugiego Buttona
WindowsButton windowsButton2=(WindowsButton)var9;
```

I tak w każdym miejscu, gdzie użyta jest metoda `inline`. Więc może nie jest to najlepsze rozwiązanie dla często
używanych metod, a tworzenie kontrolek GUI w aplikacji, która potrzebuje do tego aż `Abstract Factory` raczej do takich
przypadków się zalicza.

To podejście również pochodzi z książki "Wzorce projektowe"[^design_patterns] i raczej zostało pomyślane dla języków
Objective C czy Smalltalk, ale jeszcze do niego wrócę.

## Factory with registry
W poprzednim wpisie o [Factory Method](https://asvid.github.io/pl/kotlin-factory-method#factories-with-registry) wspomniałem o fabryce z rejestrem, pozwalającą na elastyczne dodawanie nowych fabryk. Podobną rzecz można zrobić z `Abstract Factory`. Posłużę się takim samym przykładem jak poprzednio, czyli factory budujące kontrolki GUI pod konkretny system operacyjny.

```kotlin
// interfejs Factory które będzie rejestrowane
interface ViewFactory {
    fun createButton(): Button
    fun createImage(): Image
}
// typy dostarczane przez Factory
interface View
abstract class Button : View
abstract class Image : View
```
Konkretne Factory są takie same jak w poprzednim przykładzie.
```kotlin
object LinuxViewViewFactory : ViewFactory {
    internal class LinuxButton : Button()
    internal class LinuxImage : Image()

    override fun createButton(): Button = LinuxButton()
    override fun createImage(): Image = LinuxImage()
}

object WindowsViewViewFactory : ViewFactory {
    internal class WindowsButton : Button()
    internal class WindowsImage : Image()

    override fun createButton(): Button = WindowsButton()
    override fun createImage(): Image = WindowsImage()
}
```
I samo rejestrowane Factory.
```kotlin
object ViewFactoryMaker {
    // rejestr konkretnych factory, klucz jest zwykłym Stringiem
    private val registry = mutableMapOf<String, ViewFactory>()

    fun register(name: String, factory: ViewFactory) {
        registry[name] = factory
    }

    // zwracanie konkretnego factory na podstawie klucza
    // w przypadku braku klucza zwrócony zostanie `null`
    fun getFactory(name: String): ViewFactory? {
        return registry[name]
    }
}
```
Użycie wygląda następująco:
```kotlin
// w tym wypadku xpButton1 będzie `null` bo nie ma fabryki pod kluczem "XP"
val xpButton1 = ViewFactoryMaker.getFactory("XP")?.createButton()

// ale można ją stworzyć np. jako obiekt anonimowy i zarejestrować
ViewFactoryMaker.register("XP", object: ViewFactory {
  override fun createButton(): Button {
    // znów anonimowy obiekt rozszerzający klasę Button
    return object: Button() {}
  }

  override fun createImage(): Image {
    return object: Image() {}
  }
})

// xpButton teraz już nie będzie null-em
val xpButton = ViewFactoryMaker.getFactory("XP")?.createButton()
```
Rejestr umożliwia łatwe dodanie nowych fabryk, jeśli zajedzie taka potrzeba. Fabryka Abstrakcyjna dostarcza w zasadzie tylko interfejs konkretnych fabryk i obiektów przez nie tworzonych.

## The `make` method once again
Używając podejścia z metodą `make()` i rejestru fabryk, można osiągnąć ciekawy efekt.
```kotlin
val linuxButton = ViewFactory.getFactory(ViewFactory.OS.Linux).make<Button>()
```
Pozwala to na łatwe rozszerzanie możliwości fabryk, bez konieczności pisania proxy-metod. Aby to osiągnąć, należy rozbudować `ViewFactory` o companion object i metodę `inline`. Każda konkretna fabryka będzie rozszerzać abstrakcyjne `ViewFactory` i nadpisywać metody tworzące kontrolki GUI.
```kotlin
abstract class ViewFactory {
    // typy znane Factory
    enum class OS {
        Linux, Windows
    }

    // metody do nadpisania przez konkretne factory
    protected abstract fun createButton(): Button
    protected abstract fun createImage(): Image

    // rejestr teraz trzyma referencje do metod, a nie konkretnych factory
    // kluczem jest generyczny typ zwracany przez metodę
    protected val registry = mutableMapOf<KClass<out View>, () -> View>(
        Button::class to ::createButton,
        Image::class to ::createImage
    )

    // tej metody nie da się przesłonić
    inline fun <reified T : View> make(): T? {
        // pole `registry` jest protected, 
        // więc metoda `inline` nie ma do niego bezpośredniego dostępu
        return `access$registry`[T::class]?.invoke() as T?
    }

    companion object {
        fun getFactory(os: OS): ViewFactory {
            return when (os) {
                OS.Linux -> LinuxViewFactory // Singleton
                OS.Windows -> WindowsViewFactory() // normalny obiekt
            }
        }
    }

    // sposób na udostępnienie rejestru metodzie `make()`
    // metoda jest internal więc po skompilowaniu nie będzie dostępna
    @PublishedApi
    internal val `access$registry`: MutableMap<KClass<out View>, () -> View>
        get() = registry
}
```
Czyli teraz mając już konkretne factory, dostarczone przez metodę `getFactory()` z companion object, zamiast wywoływać na nim konkretną metodę, np. `createButton()` używamy `make<Button>()` a rejestr znajdzie odpowiednią metodę na podstaie zwracanego typu i ją wywoła przez użycie `invoke()`.

Konkretne factory może wyglądać tak:
```kotlin
// normalna klasa zamiast Singletona pozwala na wewnętrzne klasy
class WindowsViewViewFactory : ViewFactory() {

    // private inner zapewnia że klasa jest niewidoczna poza Factory
    private inner class WindowsButton : Button()
    private inner class WindowsImage : Image()

    // nadpisanie metod tworzących kontrolki
    // referencje do nich trafiają do rejestru
    override fun createButton(): Button = WindowsButton()
    override fun createImage(): Image = WindowsImage()
}
```
Lub tak, z dodaną nową kontrolką `GridView` którą tylko `LinuxViewFactory` potrafi stworzyć
```kotlin
object LinuxViewViewFactory : ViewFactory() {
    internal class LinuxButton : Button()
    internal class LinuxImage : Image()

    // implementację GridView istnieje tylko dla Linuxa
    internal class LinuxGridView : GridView()

    // a factory ma w interfejsie tylko takie metody
    override fun createButton(): Button = LinuxButton()
    override fun createImage(): Image = LinuxImage()

    // dodanie nowej kontrolki specyficznej dla tylko dla tej fabryki
    init {
        // wywołanie metody z parametrem LinuxGridView zwróci NULL-a
        registry[GridView::class] = {
            LinuxGridView()
        }
    }
}
```
Kilka przykładów użycia:
```kotlin
val linuxFactory = ViewFactory.getFactory(ViewFactory.OS.Linux)
val windowsFactory = ViewFactory.getFactory(ViewFactory.OS.Windows)

val linuxButton = linuxFactory.make<Button>()
val winButton = windowsFactory.make<Button>()

val gridView = linuxFactory.make<GridView>() // OK
val winGridView = windowsFactory.make<GridView>() // null

// niestety `access$registry` jest publicznie dostępne w ramach modułu
val registry = linuxFactory.`access$registry`
// i nic nas nie powstrzymuje przed dodaniem obsługi nowej kontrolki
// tak długo jak rozszerza `View` z fabryki abstrakcyjnej
class NewView: View
registry[NewView::class] = { NewView() }

// legalne użycie metody `make()` ale zwróci null
// `View` nie ma w rejestrze metody tworzącej
val view = linuxFactory.make<View>()
```
Dodanie nowej kontrolki dla wszystkich konkretnych factory oznacza jedynie dodanie abstrakcyjnej metody do `ViewFactory` i rejestracja jej. Wszystkie klasy pochodne będą musiały ją zaimplementować. Dodanie kontrolki wyłącznie dla konkretnej fabryki również jest stosunkowo proste i implementacja zamyka się w kodzie konkretnej fabryki. W tym wypadku metoda `inline` nie spowoduje wygenerowania ogromnej ilości kodu w każdym miejscu, gdzie zostanie użyta, ponieważ jedyne co robi, to wywołuje metodę z rejestru. 

Kod wygenerowany dla użycia metody `make<Button>()`.
```kotlin
// `var10000` to referencja do metody zwracającej `Button`
Function0 var10000 = (Function0)linuxFactory.getAccess$registry().get(Reflection.getOrCreateKotlinClass(Button.class));
Button linuxButton = (Button)((View)((Button)(var10000 != null ? (View)var10000.invoke() : null)));
```

Jest więc to ciekawa i całkiem użyteczna hybryda fabryki z rejestrem oraz z pojedynczą metodą tworzącą obiekty. Niewątpliwą zaletą jest elastyczność tej implementacji, ale publiczny dostęp do rejestru metod, oraz brak pewności czy wybrana fabryka potrafi stworzyć żądany obiekt, mogą budzić wątpliwości.

# Summary
Wzorzec `Abstract Factory` przydaje się do tworzenia obiektów, które można połączyć w sensowne rodziny. Stanowi warstwę ponad standardowym `Factory Method`, serwując konkretne factory w zależności od potrzeb klientów. Może występować w kilku wariantach, np. w wersji z metodą `make()` lub rejestrem konkretnych fabryk, ale w każdym przypadku określa interfejs implementowany przez dostarczane fabryki. Dla klienta nie ma różnicy, z której konkretnej implementacji korzysta. Wady i zalety są bardzo zbliżone do tych z `Factory Method`

## Pros
- **łatwe zarządzanie "rodzinami" obiektów** - polegając wyłącznie na interfejsie fabryki, klient ma pewność, że dostarczone obiekty pochodzą z tej samej "rodziny" i będą poprawnie ze sobą współpracować
- **całkowite oddzielenie implementacji od interfejsu** - z poziomu klienta fabryki interesuje nas wyłącznie interfejs
  obiektu, co pozwala na łatwe dodawanie nowych implementacji bez konieczności zmian klienta.

## Cons
- **konieczność tworzenia dodatkowych klas** - interfejsów, fabryk, typów wyliczeniowych itd. Niekoniecznie jest to  wada, zwłaszcza jeśli chodzi o interfejsy, bo znacząco pomaga to potem testować kod. Należy wykazać się tutaj  rozsądkiem, jeśli dostarczamy instancje pojedynczej klasy i nie ma perspektyw na zwiększenie liczby typów, to może nie  ma potrzeby tworzyć ten cały boilerplate.  
- **metody `inline`** - należy z nimi uważać, ponieważ potrafią generować sporo kodu który jest wielokrotnie kopiowany w projekcie
- **dodatkowa praca podczas dodawania nowego typu** - dodanie nowego typu tworzonego przez fabryki powoduje konieczność implementacji w każdej konkretnej fabryce. Dodanie nowej konkretnej fabryki również wymaga utworzenia lub nadpisania wszystkich metod. Niekoniecznie jest to wada, bo wzorzec wymusza dostarczenie klientowi w pełni sprawnej fabryki, ale jest to dodatkowa praca poza dodaniem samych klas obiektów z nowej rodziny.

---
[^design_patterns]: [Wzorce projektowe - Elementy oprogramowania obiektowego wielokrotnego użytku](https://books.google.pl/books?id=Mkn6uAEACAAJ&dq=isbn:0201633612&hl=pl&sa=X&ved=2ahUKEwiQgZnevojvAhXeAxAIHerSDCsQ6AEwAHoECAAQAg)