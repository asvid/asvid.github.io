---
layout: post
title: "Android Build Hacks #1 - build types and product flavors"
date: "2018-07-23 22:24:27 +0200"
description: Android build file tricks and hacks I've found at blogposts, YT videos etc.
permalink: android-build-hacks
comments: true
crosspost_to_medium: false
tags:
- Android
- Gradle
- Build
- DevOps
categories:
- Tools
image: /assets/posts/android-build-hacks/build-hacks.jpg
---

*Disclaimer - I did not invent anything here, I've just collected some good practices and tricks from other articles (mentioned at the bottom) or official Android documentation or Google IO presentations*

This is first part in series of articles about Android build configuration, next parts will be linked right below.

* TOC
{:toc}
## Build configuration!
This is not the most exciting part of software engineering. Each technology, language, framework has it's own rules so there are no universal patterns, [Uncle Bob](https://en.wikipedia.org/wiki/Robert_C._Martin) will not help you here. But just like this worker on left, tighten the screw of [Empire State Building](https://en.wikipedia.org/wiki/Empire_State_Building) skeleton, developers should polish their build tools - build config itself is not the application (like skeleton is not the building), but application is useless if you cannot build release version. It also sucks if you have to wait minutes until build is finished, or manually change config for releases.

In Android Studio you start new project, type app name and select minimum SDK and Android Studio generates some files that you don't event want to touch.
But you should. The more complex app you are working on, bigger benefits you may get. Starting from faster builds, through work automation, to easer development and releasing process.

## From the beginning
After we create new project we already have 2 build variants to select: `debug` and `release`. Even if only second one is mentioned in `build.gradle` file in `buildTypes` section.
We can easily edit or add more `build types` and `product flavors`, creating way too many build variants we need. Each build type is by default combined with each product flavor, and product flavors are combined if they are set for different dimensions.

Above build types and product flavors your config should also have `defaultConfig` - just to avoid repeating yourself in build types or flavors.
```
android {
  ...
    defaultConfig {
        applicationId "io.github.asvid.flavorhell"
        minSdkVersion 16
        targetSdkVersion 27
        versionCode buildDateTime
        versionName "4.0.12"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
  ...
```

But what are build types and product flavors even for?

### Build Types
**The same app, different builds**

If you want to build exactly the same app (same look, same features etc.) but sometimes you need `debug` it, sometimes you want to build `release`, sometimes you want different signing configuration, switched on/of Proguard and Crashlytics  - separate those variants with `build types`.

Build types can kinda *inherit* from each other, for example: you have `release` build type and want to have `alpha` build that is exactly the same BUT with different app name. No need to copy-paste whole config:
```
buildTypes {
  alpha {
    initWith release
    ...
```

### Product Flavors
**The same build, different app** - like bubble gum, same process of creating product, but you can have many flavors (mint, fruit etc.)

If your business model requires *free, demo, premium* app variants, you need apps in different color schemes, you want to build `release` app but talking with staging server (bit debatable - it could be also a build type but IMO if it talks with different server, its a different app), you want to build the same app but with different IAP (in app purchases) for different app shops, or you build your app with different features for specific client - use `product flavors` for separating those variants.

Product flavors are scoped with `flavorDimensions`. Each flavor needs to be in one dimension, flavors from same dimension are excluding each other from build variant. For example:

```
flavorDimensions "stage", "other"
productFlavors {
    dev {
        dimension "stage"
    }

    prod {
        dimension "stage"
    }

    other {
        dimension "other"
    }

    other2 {
        dimension "other"
    }
}
```
Above configuration will give us variants: devOtherDebug, prodOtherDebug, devOther2Debug, prodOther2Debug - and the same for `release` build type, but you wont see **devProdDebug** or **devOtherOther2Debug**.

Please notice that `build type` name comes always on the end - order of names is important. It is similar to CSS, each dimension added to variant overrides previous settings with it's own, and then build type sets them on the end. Just remember this mechanism only overrides settings, not cleans them.

 If you have no idea what is `build variant`, its this thing you select here:
![build variants window in Android Studio](assets/posts/android-build-hacks/buildVariants.png)

### Dealing with Flavor Hell

So if you enthusiastically started adding flavors and build types to your config you should notice one thing: number of build variants grows at a geometric rate. Each new flavor in existing dimension adds number of build types to build variants, each new dimension doubles number of build variants. And it's really hard to find this one variant you need to build fast in this jungle.

Reducing number of dimensions or flavors is not really a solution if they were created according to business needs - if not you may consider rethinking this whole division.
What you should do is ask yourself a question: do I really need ALL of those build variants? If you are releasing Alpha build from your CI server, you don't really need `dev` flavor. Also `dev` flavor (will be explained later) makes sense only with `debug` build.

There are many cases that cause certain build variants to be pointless, so lets filter them out!
```
variantFilter { variant ->
        // 'dev' flavor is only available for debug build
        if (!variant.buildType.name.equals('debug') && variant.getFlavors().get(0).name.equals("dev")) {
            variant.setIgnore(true)
        }
        // 'prod' flavor is only available for release build
        if (!(variant.buildType.name.equals('release') || variant.buildType.name.equals('alpha')) &&
            variant.getFlavors().get(0).name.equals("prod")) {
            variant.setIgnore(true)
        }
    }
```
Using simple `if` statement that checks variant build type and flavors we can set them to be ignored.

## Config Settings
Ok we have build types and product flavors, we've filtered out pointless variants but what can we actually set?

## Useful links
