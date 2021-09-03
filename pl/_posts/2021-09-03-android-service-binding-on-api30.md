---
layout: post 
title: "Android service binding fix for API 30"
date:  "2021-09-03 16:15"
description: "
Android 11 (API 30) zmienia korzystanie z zewnętrznych serwisów aplikacji. Korzystając z `compileSdk 30`, bez odpowiedniego wpisu w Manifeście aplikacji `bindService()` zwróci `False`, mimo że z `compileSdk 29` będzie działać poprawnie. Chcę się podzielić rozwiązaniem, po zdecydowanie zbyt długiej walce z tym problemem.
"
permalink: "2021-09-03-android-service-binding-on-api30"
comments: true 
toc: true 
tags:
- Android
- rant
- fix

categories:
- Android

image: /assets/posts/api30fix.jpg

---
# TL;DR
> Jeśli chcesz korzystać z `bindService()` zewnętrznych aplikacji budowanych pod API 30 i wyżej, dodaj w manifeście klienta serwisu atrybut <queries> z nazwą pakietu serwisu. Więcej info [w dokumentacji](https://developer.android.com/training/package-visibility/declaring). Bez tego `bindService()` zwróci `False` a w logach będzie informacja o nieznalezieniu serwisu zgodnego z Intentem.
> 
> [Moje repo z działającym przykładem](https://github.com/asvid/Android-Services-Sandbox)

# Problem
Pracuję ostatnio nad komunikacją między aplikacjami w Androidzie. Ogólnie istnieją 3 sposoby na osiągnięcie tego: Broadcasty, AIDL i Messenger. Nie wchodząc w szczegóły, Messenger wydaje mi się najbardziej odpowiedni do mojego zastosowania. Korzysta się z tego dość prosto, na zasadzie `Bound Service`, więcej info [tutaj](https://developer.android.com/guide/components/bound-services#Messenger).

Miałem już gotowy projekt testowy, który chciałem pokazać komuś w pracy, więc trochę posprzątałem, podbiłem `target` i `compileSdk` w buildzie do **API 30** i... przestało działać. 

Wywołanie metody `bindService()` zwracało `False`, co zwykle oznacza, że serwisu nie ma w Manifeście — ale był. Powrót do **API 29** — wszystko znów działa.

#Przyczyna
Po jakimś czasie googlowania, przekopywania StackOverflow, rytualnego czyszczenia projektu i cache Android Studio znalazłem przyczynę. Jest ona opisana dokładniej [w dokumentacji](https://developer.android.com/training/package-visibility), ale ogólnie chodzi o poprawienie bezpieczeństwa aplikacji. Przed Androidem 11 (czyli API 30) każda aplikacja mogła sobie sprawdzić wszystkie zainstalowane w systemie aplikacje metodą `queryIntentActivities()`. Może to być naruszeniem prywatności użytkownika...czy coś, więcej info [tutaj](https://medium.com/androiddevelopers/package-visibility-in-android-11-cc857f221cd9). W każdym razie Android postanowił coś z tym zrobić i od wersji API 30, aplikacje mają dostęp wyłącznie do pakietów innych aplikacji, które mają wymienione we własnym Manifeście, w atrybucie `<queries>`.

Brak tego wpisu w Manifeście sprawia, że serwisy aplikacji są niewidoczne i `bindService()` zwraca `False`. Widać to w logach Logcata, bez filtrowania per aplikacja:
```
I/AppFilter: Interaction: PackageSetting {<client.app.package/PID> -> <server.app.package/PID>} Blocked
W/ActivityManager: Unable to start service Intent { act=<action> pkg=<server.app.package>} U=0: not found
```

Żeby było śmieszniej, serwis można bez problemu odpalić z `adb` i takiego samego intentu — wtedy nie ma żadnego błędu :)

> Ta zmiana w Androidzie nie ma wpływu na startowanie wewnętrznych serwisów, chodzi wyłącznie o serwisy spoza aplikacji.

# Rozwiązanie
Wystarczy dodać `<queries>` w manifeście z nazwą pakietu serwisu drugiej aplikacji, którego chcemy użyć. Czyli to samo, co podajemy tworząc Intent:
```kotlin
val intent = Intent("example_action")
intent.`package` = "io.github.asvid.services.server"
bindService(intent, connection, Context.BIND_AUTO_CREATE)
```
Manifest:
```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="io.github.asvid.services.client">
    <queries>
        <package android:name="io.github.asvid.services.server" />
    </queries>
	...
</manifest>
```

[Moje repo z działającym przykładem](https://github.com/asvid/Android-Services-Sandbox)

Alternatywnie można utknąć z `compileSdk 29` :) ale nie polecam.

### Rant na Androida
Znalezienie przyczyny tego problemu zajęło mi zdecydowanie zbyt wiele czasu... Metoda `bindService()` zwraca jedynie `True/False` bez konkretnej informacji o błędzie, mimo że potrafi rzucić wyjątek `SecurityException` przy braku permissions. Trzeba ryć w logach bez filtra aplikacji. Nawet to niewiele pomaga, bo informacja "serwisu nie znaleziono", a który udaje się takim samym Intentem odpalić z `adb` tylko potęguje zmieszanie. [Blogpost](https://medium.com/androiddevelopers/package-visibility-in-android-11-cc857f221cd9) z wyjaśnieniem zmian, w ogóle nie wspomina o `bindService()` i w zasadzie nigdzie nie znalazłem jasnej odpowiedzi na mój problem. 

Dopiero mozolne przebijanie się przez dokumentację wprowadzanych przez API 30 zmian naprowadziło mnie na rozwiązanie. Domyślam się, że chodzi o jakieś względy bezpieczeństwa, żeby złośliwe aplikacje nie mogły próbować odpalać serwisów innych aplikacji i żeby nie ułatwiać dostawania się do tych serwisów... ale bez przesady. Podbiłem `compileSdk` i moja aplikacja bez ostrzeżenia ani wyjaśnienia przestała działać.

Z wersji na wersję w Androidzie zmienia się tyle API, że pojedyncza zmiana może łatwo umknąć, nawet mimo śledzenia newsów. Trudno też pamiętać wpływ każdej zmiany na elementy aplikacji, z których nie korzysta się aktualnie. A znalezienie rozwiązania potrafi zająć bardzo dużo czasu, jeżeli problem jest nietypowy, a narzędzia deweloperskie nie pomagają.