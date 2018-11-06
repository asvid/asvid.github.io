---
layout: post
title: "Android build hacks #3 - documentation with Dokka"
date: "2018-11-05 20:05:58 +0100"
description: Generating documentation with Dokka and publishing it on GitHub walkthrough
permalink: android-build-hacks-3-documentation
comments: true
crosspost_to_medium: false
tags:
- Android
- Gradle
- Build
- DevOps
- Dokka
- Documentation
categories:
- Tools
- Documentation
image: /assets/posts/android-build-hacks-3/bg.jpg
---

This is third part in series of articles about Android build configuration, all parts will be linked right below.
> **[#1 Build basics]({% post_url 2018-07-23-android-build-hacks %})**  
> **[#2 Build time optimization]({% post_url 2018-09-16-android-build-hacks-2 %})**  
> **[#3 Documentation with Dokka]({% post_url 2018-11-05-android-build-hacks-3 %})**

* TOC
{:toc}

## Homework
Wait what? You've wrote beautiful self-documenting code and someone tells you to create **DOCUMENTATION** for it? It's already there! Well named methods and variables, design patterns used.
All that a person who wants to have basic idea of how it works needs to do, is read through it - *well named method by well named method*...  
I know IDEs are supporting that and you just need to click on method name or class to go there but be a good person, create documentation of at least public methods. Forcing people **(or your future self)** to go through code each time you want to understand (or remind) how it works is cruel.

**Self-documenting code is micro-documentation, it won't ever show the big picture.**

So if you are working with more than yourself - docs can help others understand your intention during code review. If you create library - well I won't use it if it's not documented.
I understand being rebellious about it - I was. Until I had to work with long-term undocumented project.

## Where to start
Start with attitude. It won't be rocket science, you've already wrote a code that works *(and it's unit tested obviously)*, now just describe it in more human manner. It might seem boring, but programmers often lack soft skills like basic ability to communicate intents - take it as an exercise to make yourself a better professional.
