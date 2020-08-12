---
layout: post
title: "Ultimate Github Page Deployment"
date:  "2020-08-04 22:00"
description: "Deploy strony w Jekyllu (takiej jak ta) na GitHuba jest bajecznie prosty. No chyba, że korzysta z pluginów spoza whitelisty... ale nadal da się to łatwo zrobić."
permalink: "github-page-deployment"
comments: true
toc: true
tags:
- Github Pages
- Jekyll
- Github Actions
categories:
image: /assets/posts/ghpages/cover.jpg
---

## Pan Hyde

Tekst który czytasz, znajduje się na stronie postawionej na Jekyll'u i hostowanej na GitHubie, za pomocą darmowych [Github Pages](https://pages.github.com/). 
Tak w skrócie: [Jekyll](https://jekyllrb.com/) to silnik szablonów, zamieniający `markdown` na statyczne strony `HTML`, które potem można sobie hostować gdziekolwiek,
bo nie ma potrzeby podpinania bazy danych, czy obsługi PHP czy innego Pythona po stronie serwera.

## Standardowy proces

Standardowy proces wrzucenia nowego posta wygląda następująco: 
- piszę post w `markdown` ustawiając parametry takie jak tytuł, data czy tagi.
- kiedy jestem zadowolony z tego co napisałem (nigdy), commituję zmiany i wrzucam na repozytorium na GitHubie, które nazywa się od mojego nicku `asvid.github.io`
- Github po pushu na branch `master` buduje stronę HTML ze źródeł, korzystając z silnika Jekyll - czyli pewnie odpala `jekyll build`
- to co, Jekyll wypluje, nie jest widoczne w repozytorium, ale po wejściu pod [adres bloga](https://asvid.github.io/pl/) jest widoczne jako statyczna strona WWW

Powyższy proces działa zupełnie automatycznie, nie trzeba nic konfigurować po stronie repozytorium czy GitHuba, wystarczy, że repozytorium nazywa się wg szablonu: `{nazwa_użytkownika}.github.io`.
I zwykle to wystarcza. Ale...

## Zachciało się pluginów

Czasami chce się dodać coś fajnego na stronę. Mi zachciało się mieć ją po polsku i po angielsku. Jekyll natywnie tego nie wspiera... ale też nie uniemożliwia :smile:
Znalazłem więc plugin [Polyglot](https://polyglot.untra.io/), który daje mi możliwość mieć wiele języków na jednej stronie i robi to bez większego bólu ze zmianą struktury projektu czy konfiguracją.
Dodałem go do strony, po chwili zabawy zaczął troche działać. Po dłuższej chwili działał, jak chciałem. **Lokalnie**.

Po wrzuceniu na Githuba niestety strona się posypała. Okazuje się, że Github nie buduje tak po prostu wszystkiego jak leci, 
ale ma [whitelistę](https://github.com/github/pages-gem/blob/master/lib/github-pages/plugins.rb#L20) (pewnie niedługo zmienią tą nazwę...) pluginów, które wspiera. 
Polyglot nie ma na tej liście mimo starań autora. Sama idea whitelist'y jest zrozumiała ze względów bezpieczeństwa, nie chcemy, żeby GitHub padł, bo ktoś odpalił złośliwy plugin albo kopał bitcoiny na ich serwerach.

## Ale ja chce ten plugin

Na szczęście da się ten problem rozwiązać. I to nawet na kilka sposobów. Najprościej jest zbudować stronę lokalnie i wrzucić zawartość na branch `master` a samą stronę rozwijać np. na branchu `develop`.
Jest też [paczka do NodeJS](https://www.npmjs.com/package/gh-pages), która publikuje aplikacje w NodeJS jako strony na GitHubie.

Ale że jestem leniwy i nie chce za każdym razem budować ręcznie i wrzucać na 2 branche źródeł i strony, i nie mam aplikacji na NodeJS, korzystam z 3 sposobu. 

### GitHub Actions

[GitHub Actions](https://github.com/features/actions) to taki bardzo podstawowy CI dostępny za darmo na dla każdego repozytorium. 
W ramach tego CI tworzymy sobie `Workflows` które w pliku `yaml` określają co i kiedy ma się zadziać. Dostępnych akcji jest [cały katalog](https://github.com/marketplace?type=actions),
a jeśli czegoś brakuje zawsze można stworzyć samemu, lub połączyć w jeden workflow kilka akcji.
Dodać workflow do repozytorium można przez repository->actions->New workflow i kliknięcie w link `set up a workflow yourself`.
![new workflow](assets/posts/ghpages/workflow.png)
![setup](assets/posts/ghpages/setup.png)
Doda to plik `yaml` z konfitugracją w repozytorium w katalogu `.github/workflows`

Do publikacji bloga z nieobsługiwanymi pluginami użyłem akcji [Jekyll-Actions](https://github.com/marketplace/actions/jekyll-actions) skonfigurowanej w Workflow:
```yaml
name: GitHub Pages publication

on:
  push
    
jobs:
  jekyll:
    runs-on: ubuntu-16.04
    steps:
    - uses: actions/checkout@v2

    # Use GitHub Actions' cache to shorten build times and decrease load on servers
    - uses: actions/cache@v1
      with:
        path: vendor/bundle
        key: ${{ runner.os }}-gems-${{ hashFiles('**/Gemfile.lock') }}
        restore-keys: |
          ${{ runner.os }}-gems-
    # Standard usage
    - uses:  helaili/jekyll-action@2.0.3
      env:
        JEKYLL_PAT: ${{ secrets.JEKYLL_PAT }}
    
    # Specify the Jekyll source location as a parameter
    - uses: helaili/jekyll-action@2.0.3
      env:
        JEKYLL_PAT: ${{ secrets.JEKYLL_PAT }}
```
Widać, że akcja odpala się w kontenerze z `ubuntu-16.04`, a następnie:
- pobiera repozytorium po każdym pushu (na dowolny branch co generuje pewien problem)
- korzysta z cache żeby nie pobierać tych samych `Gemów` za każdym razem
- odpala akcję, czyli buduje stronę i publikuje na branchu `master`, za pomocą pusha z wykorzystaniem `secrets.JEKYLL_PAT`

I publikacja na branchu `master` sprawia, że nie możemy na niego wypychać zmian na stronie. 
Jeśli chcemy korzystać z tej akcji, musimy zmiany wrzucać np. na `develop` a `master` zostawić tylko na potrzeby plików wygenerowanych przez skrypt.

### Jak stworzyć secret.JEKYLL_PAT

Skrypt sam z siebie działa w odizolowanym kontenerze i nie ma dostępu do zapisu w naszym repozytorium. Odczytać może, bo to publiczne repozytorium.
Żeby to umożliwić, potrzebujemy wygenerować token dostępu do `public_repo`, a następnie podać go w repozytorium jako sekret pod nazwą podaną w konfiguracji workflow.

Token generujemy, przechodząc do ustawień konta Github: Settings->Developer Settings->Personal Access Tokens. 
Po kliknięciu na `Generate new token` uzupełniamy nazwę i zaznaczamy checkbox `public_repo`.

![nowy token](assets/posts/ghpages/token.png)

Po kliknięciu zielonego przycisku na dole strony `Generate token` będziemy mieć jedyną szansę na skopiowanie go, polecam skorzystać :smile:

Skopiowany token wklejamy do sekretów repozytorium: Settings->Secrets->New secret. Nazwa jak w konfiguracji `secrets.JEKYLL_PAT` a wartość to nowy token.

![nowy secret](assets/posts/ghpages/secret.png)

I powinno działać. Przynajmniej u mnie działa, bo czytasz ten tekst :smile: