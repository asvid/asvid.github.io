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
I've read this many times one way or another in books and blogposts, heard in podcasts etc. Creating smaller pieces of code that can be reused in more than one project in general saves time and forces developers to make more encapsulated and single responsibility software, that is easer to test. That's theory.