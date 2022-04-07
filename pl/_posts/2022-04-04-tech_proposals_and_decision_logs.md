---
layout: post 
title: "Dlaczego warto korzystać z Tech proposals"
date:  "2022-04-04 11:20"
description: "
Tech Proposals są mniej formalnym sposobem na wewnętrzną dokumentację przed implementacją i usprawniają podejmowanie decyzji technicznych.
"
permalink: "tech_proposals"
comments: true 
toc: true 
tags:
- documentation
- tech proposal

categories:
- programming

image: /assets/posts/archive.jpg

---

# "Dokumentacja po-projektowa"
Jak to złośliwie nazywali wykładowcy, sprawdzając uniwersyteckie projekty na zaliczenie, których założeniem było zaplanować projekt i udokumentować plan, a następnie go zaimplementować. W praktyce pisało się, byleby działało, a następnie dokumentowało twór. I niech pierwszy rzuci kamieniem kto tak nie robił :)

Wykładowcy to często starsi ludzie bez doświadczenia komercyjnego, przyzwyczajeni do watter-fall'owego planowania projektów. Teraz mamy Agile, ciągłe dowożenie feature'ów, samodokumentujący się kod i inne pomoce.

Czy dzisiaj jest jeszcze potrzeba pisania planu przed faktyczną implementacją, czy jest to relikt przeszłości, sztuka dla sztuki i uniwersytecka fanaberia? Szczerze mówiąc, często byłbym wdzięczny nawet za napisaną na szybko dokumentację po-projektową...

# Tech Proposal
`Tech Proposals` są mniej formalnym sposobem na wewnętrzną dokumentację przed implementacją. Usprawniają też podejmowanie decyzji technicznych.

Tworzenie takich dokumentów jest przydatne gdy:
- mamy już `system` a nie tylko `aplikację`, czyli wiele współpracujących na różnych poziomach serwisów i klientów
- większe zmiany muszą przejść przez architekta lub dział security (pozdrawiam fintech)
- nowy feature wymaga wielu kompetencji i potrzebna jest kooperacja między zespołami
- pracujemy w rozsianym i asynchronicznym środowisku

Pozornie wiele może zależeć od sposobu organizacji pracy, ilości zespołów i ich składu, podejścia do ownershipu fragmentów systemu, czy procedur w firmie.

Ale nie do końca. Pracowałem w zespołach interdyscyplinarnych (mobile, backend) oraz jednolitych (sam Android), w startup'ach bez procedur i w bardziej bizantyjskich strukturach.
W zasadzie w każdym miejscu (przy odpowiednich założeniach) stworzenie dokumentu, który kilka zainteresowanych osób mogło edytować, komentować lub po prostu przejrzeć, znacząco obniżało ilość błędów i nieporozumień. Nawet jeśli nikomu nie chciało się nanosić na mój dokument, to przyjamniej miałem dupochron, bo udostępniłem go wszystkim i czekałem na feedback.

Są miejsca, gdzie analityk (lub/oraz PO, TL) przygotowuje tickety, a programiści po prostu je wykonują bez wnikania w całość rozwiązania. I to może działać. Jest jednak mało prawdopodobne, żeby ktokolwiek ogarniał niuanse wszystkich zastosowanych technologii przy rozbudowanym systemie. Nietrudno jest mi sobie wyobrazić sytuację, gdzie w połowie implementacji jakiejś części rozwiązania wychodzi nieprzewidziany problem i trzeba wracać do planowania. Angażowanie programistów wcześniej pozwoliłoby uniknąć takich zwrotek.

Zdaję sobie sprawę, że propaguje tutaj oderwanie się od programistycznego mięska na rzecz dodatkowej biurokracji i spotkań. Z mojego doświadczenia przy pewnej skali i poziomie rozbudowania projektu to znacznie **oszczędza czas i pracę**.

# Warunki wstępne
Żeby w ogóle zacząć myśleć o dokumentowaniu planu na implementację czy konsultacjach z innymi zespołami, trzeba wiedzieć odpowiednio wcześniej, co powinno zostać dowiezione. Pomaga tutaj planowanie zadań z odpowiednią szczegółowością na rok, kwartał, miesiąc do przodu. Bardzo ważna też jest transparentność wewnątrz organizacji. 
Lubię wpadać na demo różnych zespołów, żeby dowiedzieć się, nad czym pracują i co planują. Mogę od razu zaproponować pomoc w obszarze, który znam lepiej, zamiast liczyć, że wszystko pójdzie dobrze albo sami do mnie trafią. I ta **karma wraca**, inni programiści sami proponują pomoc, kiedy mój zespół wchodzi w ich obszar kompetencji. Wbrew pozorom nie ma tu żadnej dobrej woli czy empatii, po prostu **wolimy mieć pod kontrolą, kiedy ktoś grzebie w naszym kodzie**, zamiast naprawiać cudze błędy na ASAP. Cała organizacja na tym korzysta.

Jeśli jest deadline na wprowadzenie jakiejś zmiany za 3 miesiące, to można powoli zacząć szacować czasochłonność, rozbierać problem na mniejsze części, może jakiegoś spike'a trzeba będzie zrobić, pogadać z osobami czy zespołami mającymi kompetencje w temacie, skonsultować się z 3rd party dostarczającymi np. hardware — dokumentując każdy etap na koniec mamy `tech proposal` rozwiązania z już rozwiązanymi największymi problemami.

Procedury w organizacji mogą wymagać akceptacji zmian przez architekta lub dział security. Znacznie łatwiej omawiać zmiany mając dokument, może z jakimś diagramem czy propozycją API. Architekt może spojrzeć na `tech proposal` w wygodnej dla niego chwili, np. schodząc z wieży z kości słoniowej. **Łatwiej przepychać swoje pomysły, kiedy są strawnie podane**. Dział security też szybciej nakieruje na potencjalne problemy lub miejsca, które wymagają uwagi, mając jakiś zarys rozwiązania, a nie goły kod, którego mogą nie rozumieć.

# Kiedy warto tworzyć tech proposal
Zauważyłem, że dobrym wyznacznikiem jest pojawienie się wątpliwości co do sposobu rozwiązania problemu. Nigdy nie posiadamy wszystkich informacji przed pisaniem kodu, ale czasami udaje się zidentyfikować największe braki sporo wcześniej. Duże zmiany mogą wykraczać poza kompetencje konkretnych osób lub zespołu. Zapisane i udostępnione informacje zmniejszają ryzyko obsuw czasowych spowodowanych brakami kadrowymi, tzw. bus factor. Przy dobrze napisanej dokumentacji i zaangażowanym zespołem każdy członek może podjąć się realizacji choćby części rozwiązania, bez konieczności czekania aż "ten ziomek co ma wiedzę, wróci z urlopu".

Kod, którego nie trzeba dokumentować, jest super, ale jeśli rozwiązanie wykracza poza jedno repozytorium, nigdy nie będzie zawierać pełnego obrazu. Wtedy lepiej zrobić dokument i (o ile jest potrzeba) podlinkować w komentarzu kodzie. Potrzeba wyjdzie najszybciej w trakcie code review, kiedy ktoś zapyta "dlaczego tak?", zamiast podsyłać linka każdemu potencjalnemu pytającemu, lepiej wrzucić to w kod.

Tworzenie opasłej dokumentacji na samym początku projektu może okazać się niepotrzebnym obciążeniem. Wtedy lepiej skupić się na prowadzeniu `Decision Log`, do przechowania informacji, dlaczego wybrana została konkretna technologia, czy rozwiązanie. Trafienie do projektu legacy bez takich informacji może wprowadzać niechęć do naszych poprzedników :)

# Z czego składa się tech proposal
Co powinien zawierać Tech Proposal? Zależy od problemu, systemu, interesariuszy... nie ma jednego wzorca.

Osobiście lubię, kiedy zaczyna się od streszczenia problemu do rozwiązania i spisu treści. Pozwala to szybko zorientować się czytelnikowi czy jest we właściwym miejscu.

Potem można umieścić informacje wstępne, które mamy na moment pisania dokumentu, linki do innych dokumentacji czy wyników spike'ów. Ta sekcja będzie się rozwijać w czasie, nie skupiaj się na znalezieniu wszystkich informacji od razu. Po prostu zapisz, co wiesz na teraz, a nowe informacje dopisuj, w miarę jak będą się pojawiały.

Najważniejszą częścią jest odpowiednio szczegółowy opis rozwiązania. Tutaj jest kilka szkół: można dosłownie wklejać kawałki kodu czy SQL-ki, ograniczyć się do ustalenia API, lub słownomuzycznego opisu implementacji. Diagramy sekwencji czy komunikacji bywają pomocne i wcale nie trzeba ich robić w perfekcyjnym UMLu, każdy zrozumie prostokąty i strzałki.
Według mnie tech proposal nie powinien skupiać się na szczegółach implelentacyjnych, a raczej na zaprojektowaniu logiki i interakcji między różnymi elementami systemu. Szczegóły implementacyjne będą wynikać z przyjętego rozwiązania, standardów organizacji czy projektu oraz preferencji programisty.

Fajnie też mieć kilka opcji rozwiązania problemu i oznaczyć wybraną w podsumowaniu, zostawiając pozostałe w dokumencie. Daje to lepszy obraz procesu myślowego w trakcie tworzenia rozwiązania. Propozycje rozwiązania mogą pochodzić od różnych osób albo są jakąś wypadkową rozmów z innymi, co pomaga nie wpaść w pułapkę jedynego słusznego rozwiązania. Czasami warto wrzucić do dokumentu niedorzeczne rozwiązanie, od razu skazane na odrzucenie. Zdarzyło mi się, że wybrana opcja była hybrydą pierwszej propozycji oraz tej głupiej, bo samo dodanie jej pozwoliło mi to wyjść poza myślenie szablonowe.
Opis każdej opcji może kończyć się małym podsumowaniem wad i zalet, choćby w formie tabelki. Nie wszystkie wady zostaną odkryte, ale ten dokument pomoże historycznie zrozumieć motywację do podjętej decyzji. 

Tak napisany `tech proposal` naturalnie odpowiada na pytania: 
- co zostanie zaimplementowane, 
- dlaczego, 
- jakie inne rozwiązania były brane pod uwagę 
- i dlaczego nie zostały wybrane

Przykładowy dokument może wyglądać tak:
```
Abstract
Table of Contents
Info
Options
 1.
 2.
 3.
Selected solution, summary, justification
```

Oczywiście, w przyszłości może się okazać, że odrzucone rozwiązanie byłoby lepsze, ale to pewnie ze względu na nieodkryte w czasie podejmowania decyzji uwarunkowania.

## Aktualizowanie dokumentu
`Tech Proposal` jest żywym dokumentem, ulega ciągłym zmianom, rozwija się. Dobrze, jeśli każdy może dodać komentarz, ale dokument powinien mieć jednego osobowego właściciela, który pilnuje porządku i składu.

Po implementacji dokument nie umiera, ale nie widzę potrzeby uaktualniania po każdej małej zmianie pierwotnego rozwiązania. Jeśli zmiany są duże, pewnie będą wymagać nowego dokumentu i tam można podlinkowac poprzedni. Taki dokument służy jedynie do wewnętrznej dokumentacji projektu i jest aktualny wyłącznie na dzień zamknięcia. Dokumentacja udostępniana na zewnątrz powinna być **zawsze** aktualizowana.

# Podsumowanie
Odpowiednio opracowany `tech proposal` pozwala nie tylko wybrać najlepsze rozwiązanie na obecną chwilę i wiedzę, ale stanowi źródło historycznych informacji na temat rozwoju systemu. Jeśli kiedykolwiek w trakcie pracy zastanawialiście się "dlaczego akurat tak coś rozwiązano" to w `tech proposal'u` (lub `decision log'u`) byłaby odpowiedź. Stworzenie takiego dokumentu i zespołowa praca nad nim to dodatkowy wysiłek, który ma uzasadnienie tylko w pewnych przypadkach. Nie ma powodu tworzenia dokumentu na każdą małą zmianę w jednej części systemu. Przyda się natomiast gdy zmiany sięgają wielu serwisów, wykraczają poza kompetencje zespołu i być może, trzeba je skoordynować w czasie.