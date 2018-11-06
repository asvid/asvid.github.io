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

### What's worth documenting?
Not everything of course. What is the point of documenting methods like `fun add(a: Int, b: Int): Int`? And private methods, as they are not exposed to a class or library client? It actually depends on your organization standards. I'm in favor of documenting **ALL** public methods, interfaces, classes, variables etc. but the same time keeping as much as possible as internal or private implementations. This way you'll need to document as little as possible and keep implementation nicely separated from exposed interfaces.

What if there's something *hacky* in your code that should be documented (for those who'll wander into it), but it's deeply in the implementation layer? Make a good old comment explaining it. Try to avoid it, give descriptive names or talk with colleagues if you have gut feeling something fishy is going on in your code, but making comment is not a sin if everything fails. Just remember: **commenting code is not documenting it**.

### Basic Dokka syntax
Finaly some code! This post is targeting Android developers, so people who use, or should (or would love to) use `Kotlin` in their projects, so there will be no `JavaDoc` here, no no. Say hi to [Dokka](https://github.com/Kotlin/dokka) and [KDoc syntax](https://kotlinlang.org/docs/reference/kotlin-doc.html).  
`Dokka` is documentation engine that uses `KDoc` syntaxed comments to generate documents in one of many formats, there is also `JavaDoc` but I see no reason to use it. What I like most about `KDoc` is that linking to other methods, classes or code samples is really easy and interactive. But besides that, it's just like `JavaDoc`.

```
/**
 * A group of *members*.
 *
 * This class has no useful logic; it's just a documentation example.
 *
 * @param T the type of a member in this group.
 * @property name the name of this group.
 * @constructor Creates an empty group.
 */
class Group<T>(val name: String) {
    /**
     * Adds a [member] to this group.
     * @return the new size of the group.
     */
    fun add(member: T): Int { ... }
}
```

There are basically two things here: *block tags* like `@param` and *inline markup* like `[member]`.
Available *block tags* are:
- `@param` - method parameter description
- `@return` - documents returned value (who would have expected)
- `@constructor` - documents primary constructor
- `@receiver` - receiver for extension functions
- `@property` - class property description
- `@throws`, `@exception` - describes exceptions thrown by method, no need to put all of them here
- `@sample` - link to code sample with documented element used
- `@see` - link to another element
- `@author` - when you feel especially proud of following code
- `@since` - version name where this element was introduced
- `@suppress` - excludes element from documentation

And *inline markup* is used to create links to other parts of code like methods, classes etc.

### Extras

### Linking

#### Modules

#### Library

#### Repository

## Publish it

### when

### CI automatization

### GitHub Pages
