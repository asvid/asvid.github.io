---
layout: post
title: "IntelliJ FileWatcher + Latex"
date:  "2021-06-21 11:43"
description: "
IntelliJ IDEA całkiem sprawnie radzi sobie z obsługą Latex. Śmiem twierdzić, że jest to nawet lepsze doświadczenie niż TexStudio czy Texmaker, które są dedykowane do tego typu projektów. Jednak siła IntelliJ nie polega na możliwościach `out of the box` ale na pluginach i ręcznej konfiguracji procesu budowania.
"
permalink: "kotlin-adapter-pattern"
comments: true
toc: true
tags:
- IDE
- plugins
- IntelliJ
- Latex

categories:
- Today I learned

image: /assets/posts/facade.jpg

---

> TL;DR: IntelliJ Idea świetnie nadaje się do pracy z Latex. Podstawową funkcjonalność zapewniają pluginy, a sporo dodatkowych automatyzacji można sobie ustawić za pomocą File Watcher-ów. Brakuje mi tylko wyświetlania rozdziałów i sekcji jak np. w TexStudio.

# IntelliJ IDEA — prawdopodobnie jedyne IDE, jakiego potrzebujesz.
Nie pracuję z Latex-em zbyt często, ale ten format ma sporo zalet, zwłaszcza przy pracy nad dłuższymi formami pisanymi, zawierającymi sporo grafik, diagramów, formuł matematycznych itp. Zwykle korzystałem z TexStudio, ale postanowiłem sprawdzić, czy i jak poradzi sobie IntelliJ IDEA, z którego korzystam na co dzień.

I radzi sobie świetnie. Do TexStudio raczej już nie wrócę :) Bo i tak korzystałbym z IntelliJ do pisania np. kawałków kodu, które potem wklejałbym do dedykowanego edytora Latex, a tak mam (prawie) wszystko w 1 IDE.

# Moje założenia
Założyłem sobie, że chcę mieć framework do pisania dłuższych tekstów, docelowo do publikacji jako ebook lub do druku. Latex wydaje się całkiem dobrym rozwiązaniem:
- wszystko jest w kodzie, tekst, style formatowania, fragmenty kodu w dedykowanych plikach, diagramy
- organizacja projektu w wielu folderach i plikach, np osobne pliki na rozdziały i jeden główny plik zbierający całość dokumentu
- świetne wsparcie dla bibliografii, przypisów i spisu treści
- łatwe generowanie PDF-ów
- ...i pewnie sporo innych zalet, których jeszcze nie poznałem

Odrzuciłem Google Docs czy MS Office, bo wolę mieć całość projektu w kodzie i wersjonować go sobie w GIT-cie. Czyli tak jak pracuję na co dzień. 

Jednak są pewne problemy:
- podgląd dokumentu - domyślnie nie jest dostępny od razu, dopiero po kompilacji. To może być nawet zaleta, bo nie skupiam się na wyglądzie, tylko na treści. A sam wygląd powinien podlegać z góry ustalonym regułom dla całego dokumentu. Ale da się ustawić automatyczną kompilację i podgląd wewnątrz IDE.
- kolorowanie składni kodu - póki co brak wsparcia dla `Kotlina`, ale znalazłem szablon na [GitHubie](https://github.com/cansik/kotlin-latex-listing)
- wsparcie dla diagramów - chyba są jakieś pakiety dla Latex-a ale nie chce mi się ich uczyć. Lubię PlantUML i najchętniej bym wykorzystał go bezpośrednio.

# Pluginy
IDE nie ma domyślnie wsparcia dla Latex-a, ale jest kilka przydatnych pluginów, które wykorzystałem:
- [Texify](https://plugins.jetbrains.com/plugin/9473-texify-idea) - wsparcie dla Latex w IntelliJ IDEA
- [PDF Viewer](ttps://plugins.jetbrains.com/plugin/14494-pdf-viewer) - podgląd wygenerowanych PDF-ów bezpośrednio w IDE
- [PlantUML Integration](https://plugins.jetbrains.com/plugin/7017-plantuml-integration) - Podgląd diagramów z PlantUML bezpośrednio w IDE
- [File Watchers](https://plugins.jetbrains.com/plugin/7017-plantuml-integration) - obserwatory plików, chyba jest już domyślnie instalowany w IDE, ale to nadal plugin. Znacząco usprawnia pracę, przynajmniej lokalnie.

Do tego oczywiście w systemie trzeba mieć zainstalowany Latex, np. MiKTeX, którego IDE użyje do kompilacji plików do PDF-a.

## Texify
Najważniejszym pluginem jest `Texify`, który zapewnia tony funkcjonalności związanych z Latex. Od podświetlania i podpowiadania składni do kompilacji plików do PDF-a.

Dodatkowo można tak ustawić plugin, żeby automatycznie po każdej zmianie generował podgląd pliku. 

[comment]: <wstawić screen z ustawień pluginu> (TODO)

Dokument można ręcznie skompilować klikając w ikonę obok bloku `\begin`, lub tworząc customową akcję, jeśli jest potrzeba bardziej rozbudowanego procesu.

## PDF Viewer
W zasadzie bezobsługowy plugin pozwalający na wyświetlanie PDF-ów bezpośrednio w IDE, jak normalny plik z kodem. Texify dobrze z nim współpracuje automatycznie po kompilacji wyświetlając wynikowy plik PDF.

## PlantUML Integration
Korzystam z PlantUML na blogu do tworzenia diagramów, głównie klas. Samo narzędzie jest dosyć potężne, a jednocześnie przyjazne. Ten plugin zapewnia wsparcie IDE do generowania podglądu i eksportowania diagramów do plików `.svg` lub `.png`.

## File Watchers
Chyba domyślnie instalowany plugin do "wyklikania" automatyzacji wewnątrz IDE. Pozwala ustawić typ i lokalizację obserwowanych plików oraz co ma się wydarzyć, jeśli zostaną zmienione.

[comment]: <wstawić screen z ustawień pluginu> (TODO)

Zwykle będzie to np. polecenie z linii komend z określonymi parametrami.

# Mój workflow
Docelowo chciałbym, żeby mój workflow wyglądał tak:
1. Piszę tekst w dokumencie Latex, odwzorowując strukturę dokumentu w strukturze folderów
2. Docieram do momentu gdzie chcę wrzucić fragment kodu
	1. Tworzę nowy plik w odpowiednim folderze, z rozszerzeniem dla danego języka
	2. IDE zapewnia mi wsparcie składni dla wybranej technologii, formatowanie kodu itd.
	3. Po napisaniu kodu, linkuję plik w dokumencie Latex
	4. Latex poprawnie generuje kod w PDF, zachowując reguły kolorowania składni
3. Chcę dodać diagram, np. klas
	1. Dodaję nowy plik `.puml` i w nim toworzę diagram korzystając z PlantUML
	2. IDE wyświetla podgląd diagramu
	3. Linkuję diagram w dokumencie Latex
	4. Latex generuje PDF-a z poprawnie wyświetlanym diagramem, narysowanym wektorowo (brak rozmycia pixeli, zaznaczanie tekstów z diagramu itd.)
4. Podgląd PDF-a generuje się po każdej zmianie w dokumencie

## Problemy/Wyzwania
IntelliJ z pluginem `Texify` daje podobne możliwości co dedykowane edytory Latex, ale nie chce mi się ręcznie kopiować fragmentów kodu, lub generować `.svg` z diagramu i wrzucać do dokumentu. Do tego dochodzi brak wsparcia dla `Kotlina` w pakiecie `listings` do wyświetlania bloków kodu w Latex, oraz brak obsługi `.svg`.

### Kotlin
Pakiet `listings` jest dość prosty w obsłudze i ma spore wsparcie dla wielu języków. Niestety nie dla `Kotlina`, ale na szczęście pozwala na dodanie własnych reguł kolorowania składni. Kolorowanie składni w wygenerowanym PDF-ie nie jest co prawda obowiązkowe, ale za każdym razem, gdy otwieram fizyczną książkę z przykładami kodu, które są w całości tym samym kolorem, mam ochotę to przepisać w IDE. Po prostu źle mi się to czyta i zrozumienie co się dzieje, zajmuje mi znacznie więcej czasu.

Gotowy schemat kolorowania składni dla `Kotlina` znalazłem [na GitHubie](https://github.com/cansik/kotlin-latex-listing). Wystarczy dorzucić plik do projektu i podlinkować w dokumencie Latex, np:
```latex
% pakiet Latex do wyświetlania bloków kodu
\usepackage{listings} 
% linkowanie pliku z definicją stylu kolorowania dla Kotlina
\input{kotlin_def.tex}
% dodanie bloku kodu z pliku `Simple.kt`
\lstinputlisting[caption={Simple code listing.}, label={lst:example1}, language=Kotlin]{Simple.kt}
```
Podoba mi się zwłaszcza trzymanie kodu POZA dokumentem Latex. Dzięki temu, mogę pracować z kodem tak jak to zwykle robię, a potem tylko podlinkować plik w dokumencie. Kolorowanie i podpowiadanie składni rozleniwia, ale pozwala też sprawniej pracować.

### Diagramy PlantUML
Tutaj sprawa się komplikuje. Nie ma gotowego wsparcia dla diagramów z PlantUML do Latex (a przynajmniej nie znalazłem). Można generować ręcznie plik graficzny z diagramu w osobnym pliku i potem podlinkować go w dokumencie. Ale Latex nie wspiera też `.svg` a nie chcę porozciąganych `.png` w moim ślicznym PDF-ie. Więc musiałbym generować `.svg` z diagramu, a następnie zamienić plik na `.pdf` który linkuję w dokumencie Latex. 

I tutaj wchodzą `File Watcher`-y, całe na biało. Z niewielką pomocą pakietu `svg` dla Latex.

[comment]: <Inkscape i PlantUML dostępne w konsoli> (TODO)

Ustawiłem sobie 2 `File Watchery`:
1. Po każdej zmianie pliku `.puml` (którego wsparcie mam z pluginu `PlantUML Integration`) uruchamia polecenie `plantuml $FileName$ -tsvg` - czyli generuje plik `.svg` z diagramu.
2. Po każdej zmianie pliku `.svg` (czyli np. wygenerowaniu z poprzedniego kroku) wykonaj polecenie `inkscape --file=$FileName$  --export-pdf=$FileName$.pdf` - generuje plik `.pdf` z diagramem za pomocą aplikacji `Inkscape`

I taki przygotowany automagicznie PDF z diagramem mogę podlinkować i umiejscowić w Latex:
```latex
\begin{figure}[htbp]
    \centering
    \includesvg{test.svg}
    \caption{svg image}
    \label{fig:figure}
\end{figure}
```
Zwróć uwagę, że podaję nazwę pliku `.svg` a nie `.pdf` - pakiet `svg` dla Latex sam poszuka PDFa pasującego nazwą do podanego pliku.

# Braki
Najbardziej brakuje mi chyba ładnego podziału na rozdziały i sekcje, który miałem w TexStudio. Póki co IntelliJ nie potrafi tego wyświetlić. Organizując strukturę folderów w projekcie, pewnie da się osiągnąć podobną czytelność, ale nie sprawdziłem tego jeszcze w praktyce.

# Podsumowanie
IntelliJ IDEA z zestawem pluginów i File Watcherów skutecznie zastąpił mi dedykowane edytory Latex. Wydaje mi się, że zapewnia nawet większe możliwości, dzięki łatwej automatyzacji i wykorzystywaniu zewnętrznych narzędzi. Brakuje trochę ładnego podziału plików `.tex` na sekcje i rozdziały - choćby takiego jak wyświetlanie pól i klas w pliku z kodem. Niewątpliwą zaletą korzystania z jednego IDE, jest dobre wsparcie dla wielu technologii. Edytowanie fragmentów kodu w edytorach Latex albo przeklejanie z innego IDE nie jest zbyt wygodne, może powodować błędy lub nieczytelne formatowanie.

Gdyby nie praca z kodem i diagramami (które lubię mieć w kodzie), pewnie wystarczyłoby mi GoogleDocs. Ale praca w Latex i trzymanie wszystkiego w kodzie i w osobnych plikach pozwala na łatwe wersjonowanie zmian w Gicie. Mam więc bardzo znajomy dla siebie tryb pracy, mimo że wynikiem jest PDF a nie oprogramowanie :)