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

## Modularization is great
I've read this many times one way or another in books and blog posts, heard in podcasts etc. Creating smaller pieces of code that can be reused in more than one project in general saves time and forces developers to make more encapsulated and single responsibility software, that is easer to test. Modules can be published as a versioned artifacts - private library, how cool is that! That's the theory.

## But why? 
So some people said it's good to create modules/libraries, but why exactly? 

If you are (like me) an Android developer it's highly probable you've handled input fields, like email, password, IP address etc. Handling such fields usually means creating some `helper` class to verify if typed text is in correct form. Yes, there are libs for that, but loading whole library full of validators and other stuff you will never use just to check few simple fields seems a bit overkill. Also checking input fields may be very usecase specific - I remember checking if typed IP is available in local network or outside of it. For me it was faster to create and unit test my own solution than look for library providing such validator. After a while there were few `helpers` like that, and new project appeared were they could be used. So what now, copy and paste code from one project to another? Hell no. I've created library for internal usage that I've shared between both projects.
<!-- some graph of shared components -->
![Simple module dependency graph](assets/posts/overmodularity/simple_module.png)


## Story time
*Disclaimer: This is pure fictional story, based on my own experience and other developer stories I've heard, oversimplified and overcolorized*

Imagine whole complex user flow shared between projects, like login or registration to cloud service. Basically it's few screens with inputs, some HTTP requests, maybe data persistence. On the end you want to know if user is properly logged in and get auth data for future HTTP requests. If the flow itself is not that straightforward, has some weird branches only business seems to understand, it appears really worthy to write it once and use every next time new project needs it.
<!-- complicated flow example -->
![Complex module dependency graph](assets/posts/overmodularity/complex_module.png)

### Here fun begins
But what to put into this login module? It needs to talk with cloud, so maybe lets add HTTP client there. User email should be saved so some simple key-value persistence will come handy. Layouts, fragments, presenters and whole navigation - after all we just need to get logged user data from this module. Library is using other shared modules like `Commons Library` from first example, also some custom views like input fields with fancy error showing. Everything versioned and kept on servers, ready to use in future projects.

### More fun
The second project is launched some time after development of `Login Library` was finished and it has been released with first project. New project is also developed by other team than first. There is the same requirement of logging to cloud service in new project, so developers decided to use `Login Library` from first project. But how to use it if there is no `README` or example in library project? Just check the first project... Second project business requirement was to have all persisted data encoded but `Login Library` was using internaly simple key-value storage without any encoding possibilities. It can be easly fixed to 

Second project team updated `Login Library` so they could use it (for faster development they've used **[local Maven library]({% post_url 2018-01-21-androidlocalmaven %})**), and created pull request with changes, and they added team who developed `Login Library` as reviewers. But the reviewers were busy with their own project so it took them a week to check the PR, it got accepted, new version of library was released and used in second project.

Then business has great idea to change look of login screens in second app to be more up to date. Another change in `Login Library`, another pull request, another week.
Meanwhile developers of first project have found a bug in `Login Library`, they created fix and wanted to release it fast... but now they need to change their application because of changes made by second team, or fork it and begin linux-distro-like hell.

And now business has another great idea to change flow for second project only...

### Please stahp
You see where I'm going with it? And list of problems can be easily extended: http client headers may be different for each project, how about analytics of login flow for each project, diferent ideas how to maintain library, etc.

At the end creating such shared module didn't solve any problems, but generated new ones. It didn't speed up development but made it slower and annoying, possibly creating useless tension between teams and people.

## Too much love will kill you
<iframe width="560" height="315" src="https://www.youtube.com/embed/ivbO3s1udic" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
Somehow this song fits to above story: great idea of sharing code, so much passion in developing library module that would solve so many problems, team cooperation... so much love. But yet it sucks. But it doesn't have to, here is how:

- never believe when business tells you "we won't be changing that"
- each shared module should have an owner, person who knows why and how it works and decides about direction of module development. And I mean 1 person, not team - any project will die with big enough decision committee. Shared ownership means shared responsibility - will work as great as communism.
- creating shared library is no different from adding new internal service or class and the same rules apply. 
    - reduce outside dependencies
    - consciously design your API
    - think about various configuration variants rather than current use cases
    - hide everything behind interfaces
    - shared code is to be used by application so do not force clients of your lib to change their projects in way they don't want just to use your shared code.
    - what you want to achieve is both projects to share behavior (logging user) not exact implementation - it's similar to OOP inheritance problem
    - SOLID, DRY bla bla bla...
- think few times do you really need to create shared library, talk with other teams what they think about it. Something that seems a great idea at first may be overkill or not be used at all on the end. 
- create README, and make it a good one. Examples, explanations, generated documentation
- create example application that uses your shared code, so going through big old project won't be necessary to understand it
- always add tests (unit, functional) in your library

So for my example with `Login module`:
- putting views and whole navigation is asking for problems: UI will change for different reasons than business logic and each change has to be released as new version of library and updated dependency in app, it may also limit navigation control and changing user flows
- putting HTTP client and persistence directly into library is easy, but also may cause problems. Such dependencies should be injected by application that uses library.
- select owner - faster pull requests
- add README, code examples, documentation