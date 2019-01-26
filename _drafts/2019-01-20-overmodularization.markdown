---
layout: post
title: "Overmodularization"
date: "2019-01-20 18:57:02 +0100"
description: Creating your own modules and libraries is generaly a good thing, but it comes with a cost. I'd like to share some of my experience about it.
permalink: overmodularization
comments: true
crosspost_to_medium: false
tags:
- Android
- Gradle
- Architecture
categories:
- Architecture
image: /assets/posts/android-build-hacks-3/bg.jpg
---

* TOC
{:toc}
<!-- 
* main post parts
    * what they tell us
        * creating modules is great for encapsulating pieces of software
        * easer testing
        * single responsibility code
        * modules vs artefacts - just discuss the diference
    * how it may be working 
        * sharing code between projects is great
        * commons module
        * example with logging flow
    * why it may fail to work
        * example with 2-3 projects using same library with changing requirements
        * slow development
        * long PR 
        * what should go to module?
        * upside-down dependency when module demands app to use certain navigation for ex.
        * various ideas how to maintain module
        * versioning
        * splits like in Linux distros
    * things to consider
        * each artifact should have owner
        * ALWAYS document code
        * make example apps
        * show usage
        * create README
        * to create library is easer than to maintain it
* not mention how its exacly done @ current job
* dont pretend to be God, just share experience
* "too much love will kill you" - Queen
 -->
## Modularization is great
I've read this many times one way or another in books and blog posts, heard in podcasts etc. Creating smaller pieces of code that can be reused in more than one project in general saves time and forces developers to make more encapsulated and single responsibility software, that is easer to test. Modules can be published as a versioned artifacts - private library, how cool is that! That's the theory.

## But why? 
So some people said it's good to create modules/libraries, but why exactly? 

If you are (like me) an Android developer it's highly probable you've handled input fields, like email, password, IP address etc. Handling such fields usually means creating some `helper` class to verify if typed text is in correct form. Yes, there are libs for that, but loading whole library into project full of validators and other stuff you will never use just to check few simple fields seems a bit overkill. Also checking input fields may be very usecase specific - I remember checking if typed IP is available in local network or outside it, for me it was faster to create and unit test it than look for library providing such validator. After a while there were few `helpers` like that, and new project appeared were they could be used. So what now, copy and paste code from one project to another? Hell no. I've created library for internal usage that I've shared between both projects.
<!-- some graph of shared components -->
![Simple module dependency graph](assets/posts/overmodularity/simple_module.png)


## Story time
That was simple example, now something more complex. Imagine whole user flow shared between projects, like login or registration to cloud service. Basically it's few screens with inputs, some HTTP requests, maybe data persistence. On the end you want to know if user is properly logged in and get auth data for future HTTP requests. If the flow itself is not that straightforward, has some weird branches only Product Owner seems to understand it appears really worthy to write it once and use every next time new project needs it.
<!-- complicated flow example -->
![Complex module dependency graph](assets/posts/overmodularity/complex_mdule.png)

### Here fun begins
But what to put into this login module? It needs to talk with cloud, so maybe lets put HTTP client there. User email should be saved so some simple key-value persistence will come handy. Layouts, fragments, presenters and whole navigation - after all we just need to get logged user data from this module. Flow is being developed at first in project that uses `Dagger2` for DI, so for easier assembly flow library exposes itself through `Dagger2` module. Library is using other shared modules like `common helpers module` from above, also some custom views like input fields with fancy error showing. Everything versioned and kept on servers, ready to use in future projects.

### More fun
The second project is launched some time after development of `Login module` was finished and it has been released with first project. New project is also developed by other team than first. There is the same requirement of logging to cloud service in new project, so developers decided to use `Login module` from first project. But how to use it if there is no `README` or example in library project? Just check the first project... oh it's exposed only as Dagger2 module, and in second project they used `Kodein`. No problem at all, it can be changed to expose public classes in standard way, without any DI-ready modules.
Second project team updated `Login module` so they could use it (for faster development they've used **[local Maven library]({% post_url 2018-01-21-androidlocalmaven %})**), and created pull request with changes, and they added team who developed `Login module` as reviewers. But the reviewers were busy with their own project so it took them a week to check the PR, it got accepted, new version of library was released and used in second project.
Then product owner has great idea to change look of login screens in second app to be more up to date. Another change in `Login module`, another pull request, another week.
Developers of first project have found a bug in `Login module`, they created fix and wanted to release it fast... but now they need to change their application because of changes made by second team.
And now product owner has another great idea to change flow for second project only

### Please stahp
You see where I'm going with it? And list of problems can be easily extended: http client headers may be different for each project, how about analytics of login flow for each project, diferent ideas how to maintain library, forking library instead of changing it, etc.

At the end creating such shared module didn't solve any problems, but generated new ones. It didn't speed up development but made it slower and annoying, possibly creating useless tension between teams and people.

## Too much love will kill you
[Queen - Too much love will kill you](https://www.youtube.com/watch?v=ivbO3s1udic)
Somehow this song fits to above story: great idea of sharing code, so much passion in developing library module that would solve so many problems, team cooperation... so much love. But yet it sucks. But it doesn't have to, here is how:

`Too much love will kill you
If you can't make up your mind`

- each shared module should have an owner, person who knows why and how it works and decides about direction of module development. And I mean 1 person, not team - any project will die with bit enough decision committee. Shared ownership means shared responsibility - will work as great as communism.
- creating shared library is no different from adding new internal service or class and the same rules apply. 
    - reduce dependencies
    - consciously design your API
    - think about various configuration variants rather than current use cases
    - hide everything behind interfaces
    - shared code is to be used by application so do not force clients of your lib to change their projects in way they don't want just to use your shared code.
    - what you want to achieve is both projects to share behavior (logging user) not exact implementation - it's similar to OOP inheritance problem
- think few times do you really need to create shared library, talk with other teams what they think about it. Something that seems a great idea at first may be overkill or not be used at all on the end. 
- create README, and make it a good one. Examples, explanations, generated documentation
- create example application that uses your shared code, so going through big old project won't be necessary to understand it
- always add tests (unit, functional) in your library

So for my example with `Login module`:
- using any 3rd party library as only entry point to module is pretty naive, it should be as pure as possible
- putting views and whole navigation is asking for problems: UI can change often and each change has to be released as new version of library and updated dependency in app, it's also limiting for application to control navigation and alterate flows
- putting HTTP client and persistence directly into library is easy, but also may cause problems. Such dependencies should be injected by application that uses library.
- select owner - faster pull requests
- add README, code examples etc