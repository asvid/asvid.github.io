---
layout: post
title: "Code Review Retro"
date:  "2020-08-06 11:43"
description: "To nie jest lista 'złotych zasad' code review, jest sporo bardzo dobrych postów na ten temat. Tutaj po prostu zrobiłem retro na podstawie własnych doświadczeń."
permalink: "code-review-retro"
comments: true
toc: true
tags:
- code review
- self-retro
- pull request
- team work
categories:
image: /assets/posts/codereview-retro/inspection.jpg
---

# Waga code review

Mam szczerą nadzieję, że nie muszę nikogo o tym przekonywać, jak ważne jest robienie codereview.
Jeśli potrzebujesz porad jak robić to sensownie polecam [wpis na blogu Yelpa](https://engineeringblog.yelp.com/2017/11/code-review-guidelines.html)
W tym wpisie chciałbym zrobić retro ze swoich doświadczeń z `codereview`.

Kiedy zaczynałem pracę jako programista w dość małej firmie — nie robiliśmy review. Jednym z powodów był fakt, 
że nie miał kto komu sprawdzać kodu, bo było po 1 programiście na technologię (backend, frontend, mobile).
Kolejnym powodem (i raczej głównym) było to, że dla większości z nas była to pierwsza praca i nie mieliśmy doświadczenia.
Nikt nie powiedział **"musimy robić codereview, nawet jeśli nie znamy danej technologii za dobrze, bo dobry kod zawsze spełnia te same kryteria"**.

Później na szczęście trafiłem w miejsce, gdzie codereview było standardem, dzięki temu sporo się nauczyłem.

# Zaczęły się kwiatki

Pull request to zawsze troche przepychanki — fajnie to podsumowuje [Yegor Bugayenko](https://youtu.be/DUWiy7fvVzk?t=81) - 
jako producent kodu chcesz go dołączyć do projektu, ale jako recenzent pull requesta pilnujesz, żeby nie dostał się tam żaden bałagan.
> Najważniejsze to pamiętać, że obie strony mają ten sam cel — utrzymać jak najwyższą jakość kodu w projekcie, a nie przepychać własne rozwiązanie, bo jest "mojsze".

PR służy do sprawdzenia, czy nowy kod spełnia przyjęte w organizacji konwencje (codestyle, testy), 
nie ma niebezpiecznych zmian czy jakichś głupot lub śmieci, które zostały po developmencie.

Tyle teorii, a praktyka...

## Kapelusz recenzenta

Najpierw kilka obserwacji dotyczących sprawdzania czyjegoś kodu.

### Lol

Taki komentarz dostałem kiedyś w trakcie pull requesta :roll_eyes: `lol` no i fajnie, ale co to znaczy? Zrobiłem coś głupiego? Zrobiłem coś śmiesznego?
Teraz muszę coś odpisać, żeby się dowiedzieć, recenzent ew. odpisze (oby nie w tym samym stylu) i dopiero wtedy, przy odrobinie szczęscia, dowiem się co miał na myśli.

Szanujmy siebie nawzajem i swój czas, za który płaci nasz pracodawca i starajmy się od razu napisać **jak najjaśniej** o co chodzi.
Code Review to jak najbardziej czas i miejsce na dyskusje, ale ciężko dyskutować z `lol`.

### Memy

Sam je czasami wrzucam do PR-a, ale jest kilka warunków:
- recenzent i autor PR muszą się dobrze znać i wiedzieć, że mem zostanie prawidłowo odebrany
- `enough is enough`, mem od czasu do czasu i w punkt jest spoko, ale pojawiający się przy każdym komentarzu będzie wyglądać niepoważnie i zaśmiecać PR-a
- jest coraz więcej tematów, które bezpieczniej zostawić na po pracy, a PR-y często są dość publicznie dostępne w ramach organizacji. Nie wszyscy muszą podzielać niepoprawny politycznie humor.

### Osobiste preferencje

"A tutaj możesz to zrobić `tak`" - gdzie `tak` zwykle daje dokładnie taki sam efekt, po prostu zrobione na preferowany przez recenzenta sposób.
Może nawet ten sposób był brany pod uwagę w trakcie developmentu, ale ostatecznie nie trafił do kodu, bo autor wie coś, czego nie wie recenzent.

Taki komentarz może być pomocy i wywołać owocną dla obu stron dyskusję, o ile recenzent nie blokuje PR-a za wszelką cenę, dopóki nie zmienisz kodu na jego rozwiązanie. 

### Tryb rozkazujący

"Zamień X na Y" - najlepiej bez podania jakiegokolwiek uzasadnienia. Bo tak. Bo recenzent wie lepiej.
Taki sposób komentowania jest IMO niedopuszczalny, nawet jeśli między autorem i recenzentem jest spora różnica wiedzy i doświadczenia.
Jeśli junior nie dostanie wytłumaczenia dlaczego `X` jest gorsze od `Y` to **albo się nigdy nie nauczy, albo będzie ślepo używać `Y`** "no bo senior tak powiedział".

Zwykle 1-2 zdania wytłumaczenia lub nawet link do StackOverflow, czy jakiegoś posta w internecie zrobią robotę.

### Eseje

`Brzytwa Ockhama` to Twój kumpel. To jest niesamowicie ważna i przydatna umiejętność, żeby przekazać maksimum treści w jak najmniejszej ilości dobrze dobranych słów.
Czasami muszę się przebić przez epos w komentarzu, z którego wynika jedynie, że mógłbym nazwać zmienną inaczej.

To czego bym oczekiwał, to wskazania o którą zmienną chodzi, dlaczego ta nazwa jest słaba i 1..n propozycji nazw.

Nie potrzebuję wstępu typu "Czy nie uważasz, że ta nazwa jest zbyt/za mało jakaś-tam" - nie, nie uważam i dlatego właśnie taką nadałem.
W środku nie potrzebuję też dostać 10 linków do StackOverflow i polecenia 3 pozycji książkowych nt. inżynierii oprogramowania - i tak tego wszystkiego nie przeczytam.
Na koniec nie muszę czytać o tym, że "to tylko sugestia, nie musisz tak robić, jeśli uważasz, że obecna nazwa lepiej pasuje, ale {c.d. słowotoku}" - wiem, to dotyczy każdego komentarza w czasie code review.
Komentarz recenzenta jest swego rodzaju `call to action` i reakcja autora jest spodziewana, nie trzeba dodawać "co o tym myślisz? / jak uważasz?" pod każdym komentarzem.

Rozumiem, że takie dodatkowe językowe upiększenia sprawiają milsze wrażenie niż suche informacje, ale mnie na dłuższą męczą i uważam, że niepotrzebnie zaciemniają właściwą treść informacji.

Także konkrety, błagam :smile:

### Autor ciągle powtarza te same błędy

Ludzie raczej uczą się na błędach i dostosowują do panujących standardów w organizacji.
Jeśli jednak autor kodu za każdym razem robi te same błędy, czy nie stosuje się do stylu pisania kodu — należy działać.
Często za każdym "do niego nie dociera" kryje się "nie umiem mu tego przekazać właściwie". 

Warto najpierw pogadać czy to przez roztargnienie, czy celowe łamanie konwencji, bo np. uważa, że jego są lepsze.
W przypadku roztargnienia można próbować różne reguły automatyzować — są narzędzia jak formater kodu, SonarLint czy reguły w IDE które zaznaczają łamanie zasad w trakcie pisania kodu, na długo przed PR.
Jeśli ktoś celowo walczy z systemem... no cóż, nie z każdym się dobrze pracuje :roll_eyes:

Jestem zdania, że niedoskonałe standardy są lepsze od ich braku i zawsze warto nad nimi pracować, zamiast z nimi walczyć. 
Świadome łamanie zasad przyjętych w organizacji zamiast prób ich zmiany prowadzi tylko do chaosu w kodzie i niepotrzebnych napięć.

### Doceniaj

Kiedy widzisz w kodzie fajną zmianę — doceń to w komentarzu, wystarczy łapka w górę :+1:. 
Sam approve PRa znaczy tyle co "wystarczająco dobrze", ale jeśli autor zrobił coś ciekawego i się wykazał, to dobrze jest to docenić.
Buduje to fajną kulturę wśród programistów i przy okazji znaczy, że recenzent się wczytał w kod, a nie tylko przeleciał po łebkach i dał approve.

Nie warto z tym jednak przesadzać, bo straci to na znaczeniu.

### Szerszy kontekst

Najłatwiej w trakcie recenzowania kodu wyłapać literówki, problemy z nazewnictwem czy drobne usprawnienia implementacyjne.
Wynika to z faktu, że **te problemy są wyizolowane — znajdują się tylko w 1 pliku czy nawet linii kodu**.

Znacznie trudniejsze, ale i ważniejsze jest zrozumienie szerszego kontekstu zmian.
Wymaga to znajomości kodu projektu, domeny biznesowej i rozumienia zadania, które realizuje zmiana w kodzie.
To przychodzi z czasem, w zależności od rozmiaru projektu i poziomu skomplikowania domeny.

Potrzebny też jest zgrany zespół, który wspólnie pracuje nad wyznaczonymi celami sprintu i aktywnie angażuje się we wszystkie zadania.
Grupa wyalienowanych kontraktorów prawdopodobnie nie będzie w stanie sobie nawzajem udzielić takiego feedbacku i skupi się jedynie na wymienionych na początku podstawowych problemach.

### Zawsze komentuj

Nie lubię cichych approvów, chociaż sam często to robię :smirk:. Jeśli zmiany w kodzie są jasne i nie znajdujesz żadnych problemów, warto to napisać w komentarzu do całego PRa.
Takie proste "wygląda dobrze" wystarczy. Dodatkowo to sygnał, że poświęciłeś/aś chwilę na przeczytanie zmian, zamiast odruchowo kliknąć `approve` bo Cię ktoś męczy od tygodnia, żeby zrobić ten PR.

Jeśli widzisz w kodzie błahostkę — też zostaw komentarz, ale z informacją że to pierdoła. Nie chodzi o to, żeby się czepiać, ale dać info, że coś można zrobić lepiej.

------

## Czapka autora

### Opis

Zauważyłem, że lubię kiedy PR który mam recenzować ma opis, informujący o intencji zmian i który linkuje do najważniejszych zmian w kodzie.

Standardowo w opisie lub tytule PRa powinien znaleźć się link do ticketa w Jirze (lub do innego systemu zarządzania zadaniami), ale wszyscy wiemy jak potrafią wyglądać opisy zadań.
Do tego dochodzą nieudokumentowane odkrycia w trakcie implementacji, znalezione problemy i rozwiązania.

Dlatego, żeby pomóc recenzentowi w zrozumieniu szerszego kontekstu zmian (patrz wyżej) **dobrze jest opisać słownie co się dzieje w pull requeście**.
Jasne, że kod powinien opisywać się sam... ale diff niekoniecznie zrobi to dobrze, a skakanie między kodem w IDE a PR-em do przyjemnych nie należy.
Dodatkowo wydłuża proces sprawdzania kodu, co powoduje niechęć do robienia PR-ów.

Warto podlinkować najważniejsze zmiany w kodzie, żeby oddzielić je od typowych prac ogrodniczych lub drobnych poprawek.
W opisie można zawrzeć screeny (patrz niżej) lub diagramy pozwalające lepiej zrozumieć bardziej skomplikowane zmiany, które mogą umknąć przy czytaniu wyłącznie diffa.

### Screeny

Jeśli zmiany w kodzie powodują jakiś update UI, dobrym pomysłem jest załączenie screenów z efektem.
Nawet jeśli UI jest wcześniej przygotowany przez grafika. 

Czasami się coś przeoczy, a zrobienie screenów pozwala wyłapać błąd bez potrzeby uruchamiania aplikacji przez recenzenta.

### Self-review

**Przed dodaniem recenzentów do pull requesta — zrób sobie review**. Sprawdź:
- czy nie zostały w kodzie jakieś śmieci
- czy patrząc tylko na diffa zmian można zrozumieć co się dzieje
- czy jakaś nazwa zmiennej/metody/klasy nadana na początku nadal pasuje do całości zmian
- czy testy są sensowne
- czy przyjęte w organizacji konwencje są przestrzegane

W trakcie szukania rozwiązania problemu, łatwo można pominąć czy zapomnieć o którejś z w/w rzeczy i to właśnie zaraz przed dodaniem recenzentów masz ostatnią szansę naprawić potencjalne problemy, zanim ktoś inny je zauważy.
Niektóre organizacje czy nawet sami programiści mają checklisty przed PR-em, które oszczędzają czas recenzentów przez znajdowanie błahych problemów, zanim zostaną przypisani do PR-a.

### Feedback to prezent

Pisząc kod, wystawiasz się na krytykę i nie masz wpływu (ewentualnie pośredni) na to, w jakiej formie ją otrzymasz. Masz jedynie wpływ na to, jak na nią zareagujesz.
Nawet jeśli nie zgadzasz się z recenzentem, doceń fakt, że poświęcił czas na napisanie komentarza i (oby) uzasadnienie swojego punktu widzenia, poprzedzone wczytaniem się w Twoje zmiany.
Chyba że napisał tylko `lol` to wtedy olej albo eskaluj :no_mouth:.

Dobrze jest na każdy komentarz w kodzie odpowiedzieć lub jakoś zareagować (np. BitBucket daje opcję "polubienia" komentarza).
Zostawianie takich wiszących uwag sprawia wrażenie, że recenzent produkuje się na nic.

### Minimalizuj szum

Nikomu nie chce się robić pull requestów ze zmianami w 100 klasach, rozwiązującego jednocześnie wiele problemów i zmieniającego niezależne od siebie miejsca w kodzie.
Jeśli w Twojej organizacji jest problem z wybłaganiem od innych programistów sprawdzenia Twojego PRa, warto robić je tak, żeby sprawnie i szybko dało się zrozumieć intencję zmian.
Często zadania udaje się rozbić na niezależne fragmenty i wtedy można zrobić kilka PR-ów z mniejszymi zmianami zamiast jednego dużego.

Niestety nie zawsze. Wtedy staraj się przy okazji nie naprawiać świata, zwłaszcza kiedy zauważysz, że zaczynasz zmieniać miejsca zupełnie niepowiązane z przyczyną wprowadzania zmian.
PR wtedy puchnie do ogromnych rozmiarów, gdzie setki plików są zmienione w kilkunastu miejscach.
Powoduje to rozmycie intencji zmian w kodzie i schowanie najważniejszych zmian przed recenzentami.

Lepiej zrobić sobie ticket na później i starać się go wciągnąć do sprintu, albo zrobić to w osobnym pull requeście w ramach tego samego zadania.
Jeśli zmian jest dużo i nie da się tego ograniczyć, dodaj dobry opis z linkami do najważniejszych zmian w kodzie, to tam powinna skupić się uwaga recenzentów.

### PR nie leży w próżni

Dobry PR żyje krótko. Jeśli od otwarcia PRa do merga mijają tygodnie lub miesiące, to szansa na konieczność rozwiązywania konfliktów rośnie wykładniczo.
A to jest męczące i irytujące... 

Dlatego **małe pull requesty są super**, bo recenzenci chętniej na nie spojrzą, statystycznie problemów i poprawek też będzie mniej i szybciej się je merguje.
Jest to tym ważniejsze, im więcej programistów lub całych niezależnych od siebie zespołów wprowadza zmiany w projekcie.

# Podsumowanie

Zależy nam na produktywności w pracy, niezależnie od tego po której stronie (autora lub recenzenta) się znajdujemy.
Raczej każdy programista występuje naprzemiennie w każdej z ról, dlatego łatwe powinno być zrozumienie jak ważna jest **komunikacja i empatia**.

Piszmy tak, żeby bezproblemowo można było zrozumieć, co chcemy przekazać, niezależnie czy to kod czy komentarz.
Dobrze przygotowany PR będzie szybciej i lepiej sprawdzony, a komentarze pisane z szacunkiem i bez szydery mają lepszy efekt na jakość kodu w projekcie.

W pracy z ludźmi chodzi o to, żeby się dobrze pracowało, wszystkim :smile:
