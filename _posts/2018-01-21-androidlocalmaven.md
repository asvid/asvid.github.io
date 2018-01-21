---
layout: post
title: "Android local libraries with Maven"
date: 2018-01-21
description: Local Maven repository guide for working on private libraries along with projects that use them
permalink: android-local-maven
comments: true
crosspost_to_medium: true
tags:
- Maven
- Gradle
- Android
- Kotlin
categories:
- Tutorial
---

* TOC
{:toc}

## Intro
Have you ever created Android library? You know, when you are working on specific functionality in some project and get enlightenment "hey, I could use this in some other project!". No? Well... you should - at least sometimes :) I don't mean creating new ultimate architecture framework every week (we are not JavaScript developers after all), but writing simple tools that you know how to use and that will make your work easer on future projects. I recommend trying this, getting stars on GitHub and showing friends your library at [AndroidArsenal][AndroidArsenal] is cool.

Here is some random tutorial on how to put your lib on [JitPack][JitPack]: [How to JitPack your lib][How to JitPack your lib]. When it's there, you can use it like every other dependency in project.

### Why even
Anyway, this text is not about just creating libraries and putting them online. You may not want to make your libs opensource or store them externally, but still benefit from having great tools you've made. Solution is local ```Maven``` repository. It also gives you a **very** easy way to work on your library and test it immediately on project you need this library for. Without this, every time you want to fix or add something to your lib, you need to update it's version, make release on Github, send it to [JitPack][JitPack], wait until it builds, hope it won't fail, update version of lib in your project, and then if everything goes well check if it does it's work. Of course your lib should contain sample application that shows usage of all library features, so you can test if it's working without this whole release process, but sometimes it's not enough and you need to test on real project.

## Entry info
- you dont need to install anything in your system, it's all in ```Gradle``` plugin
- no need to use command line
- no need to modify your project, just ```build.gradle``` file
- you don't have to know where are your local libs stored, but its good to know:
  - Unix/Mac OS X – ~/.m2
  - Windows – C:\Documents and Settings\{username}\.m2
- each local release **overrides previous one** with same version number, so there is no need to update version each time (like for [JitPack][JitPack] or other remote ```Maven``` repository)
- you may have many local versions of your library, versioning works in the same way. You can even develop separate versions of your lib that are used in different projects, because of different ```minSdkVersion``` etc.

Sample code in ```Kotlin``` is available on [GitHub][github]. There are 2 projects in repo: ```app``` that contains application using locally published libraries from project ```library```. Library project has sample application and 2 library modules: ```firstlib``` and ```secondlib```. They are dead simple and contain just custom button class that overrides initial background color.

## How to

Sounds too good to be true? Don't worry, there is no catch.

### Library

In your library module ```build.gradle``` you have to add (usually on top of file):

```
apply plugin: 'maven-publish'
```

If you publishing your lib to JitPack, there is a chance you have below variables set. If not, just add below lines before ```android``` block.

```
def artifactId = 'your-library-name}'
def groupId = 'com.your.package.name'
```


Now is the only tricky part. Following code should be added at the bottom of lib module ```build.gradle```, after any blocks you already have there.

#### Typical case
If your lib is single module, or you have many library modules in single project but without mutual dependencies:
```
publishing {
    publications {
      library(MavenPublication) {
        setGroupId(groupId)
        setArtifactId(artifactId)
        version android.defaultConfig.versionName

        artifact(bundleRelease)
      }
    }
}
```

#### Not-so-typical case
If your library project contains many modules that are independent libs and one that is collection of them - it might sound stupid but sometimes it's useful. You can use it also for single module, but it's just an overkill. :
```
publishing {
    publications {
      library(MavenPublication) {
        setGroupId(groupId)
        setArtifactId(artifactId)
        version android.defaultConfig.versionName
        artifact(bundleRelease)

        pom.withXml {
          def dependenciesNode = asNode().appendNode('dependencies')
          configurations.compile.allDependencies.each {
            if (it.group != null && (it.name != null || "unspecified".equals(it.name)) && it.version != null) {
            def dependencyNode = dependenciesNode.appendNode('dependency')
            dependencyNode.appendNode('groupId', it.group)
            dependencyNode.appendNode('artifactId', it.name)
            dependencyNode.appendNode('version', it.version)
            }
          }
        }
      }
    }
}
```

#### Protip

You can create file with publishing code and apply it in each module instead of copy-paste whole thing. You can see it used in [Github project][github] in module ```secondLib``` of library project. There is a file ```publish_local.gradle``` in library project root folder that contains publishing code. In library module you can now just add
```
apply from: '../publish_local.gradle'
```
Just remember to set variables ```artifactId``` and ```groupId```, they are individual for each module.

#### Publish your lib
All you need to do now is run task: ```publishToMavenLocal```.

![Gradle task]({{ "/assets/posts/local-maven-repo/gradle_task.png" | absolute_url }})

So what is it for actually? You are telling ```maven-publish``` to release your library module to local ```Maven``` repository with name you've set in ```artifactId``` and package set in ```groupId```. For multi-module project, it just iterates over modules and if they have ```publishing``` code they are also published to local repository.

### Project

All you have to do in project you want to use locally published library, is add in project root ```build.gradle``` file ```mavenLocal()``` in repository list, so it looks like that:
```
allprojects {
  repositories {
    mavenLocal()
    jcenter()
    ...
```
How it works: ```Gradle``` while building your project will look for dependencies first in local ```Maven``` repository, then if it won't find requested dependency it will try ```JCenter``` etc. So if you are using CI, your library should be released on source that CI have access to, like remote repository (private Nexus server, JitPack).
I highly recommend using local repository only for development and testing your lib, and then publishing it somewhere when you are sure your work is done.

## Outro

This approach to library development saved me **tons** of useless work and waiting for publishing on remote repository. I hope with this tutorial I can save some of your time. Or maybe you've figure it out in better way? :)

[AndroidArsenal]: https://android-arsenal.com/
[JitPack]: [https://jitpack.io/]
[How to JitPack your lib]: https://medium.com/@ome450901/publish-an-android-library-by-jitpack-a0342684cbd0
[github]: https://github.com/asvid/local_maven_repo
