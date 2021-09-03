---
layout: post 
title: "Android service binding fix for API 30"
date:  "2021-09-03 16:15"
description: "
Android 11 (API 30) changes the way of using external app services. Using `compileSdk 30` and above, without additional Manifest entry the `bindService()` method will always return `False`, even if with `compileSdk 29` the app will work perfectly. I want to share solution of this problem after WAY TOO LONG time I spent on searching it...
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
- rss

image: /assets/posts/api30fix.jpg

---
# TL;DR
> If you want to use `bindService()` with external application services built with API 30 and above, add the <queries> attribute with the service package name to the client app manifest. More info [in the documentation](https://developer.android.com/training/package-visibility/declaring). Without it, `bindService()` will return `False` and the logs will tell you that an Intent-compatible service has not been found.
>
> [My repo with a working example](https://github.com/asvid/Android-Services-Sandbox)

# Problem
I am recently working on communication between applications in Android. Overall, there are 3 ways to achieve this: Broadcasts, AIDL, and Messenger. Without going into details, Messenger seems to be the most suitable for my case. It is quite simple to use, on the basis of the `Bound Service`, more info [here](https://developer.android.com/guide/components/bound-services#Messenger).

I already had a Proof of Concept project ready that I wanted to show to someone at work, so I cleaned it up a bit, bumped `target` and` compileSdk` version to the **API 30** and... it stopped working.

The call to the `bindService()` method returned `False`, which usually means the Service is not present in the Manifest - but it was. Back to **API 29** - and everything works again, so it wasn't a typo in service package or something.

#The reason
After a while of googling, digging through StackOverflow, ritual cleaning of the project and Android Studio cache, I found the cause. It is described in more detail [in the documentation](https://developer.android.com/training/package-visibility), but in general it is about improving the security of the application. Before Android 11 (API 30), any application could check all other applications installed on the system using the `queryIntentActivities()` method. It could be a violation of the user's privacy... or something, more info [here](https://medium.com/androiddevelopers/package-visibility-in-android-11-cc857f221cd9). Anyway, Android decided to do something about it, and from API 30, applications only have access to the 3rd party packages listed in their own Manifest inside `<queries>` attribute.

The absence of this entry in the Manifest make 3rd party application services invisible and `bindService()` returns `False`. This can be seen in the Logcat logs, without filtering for only selected application:
```
I/AppFilter: Interaction: PackageSetting {<client.app.package/PID> -> <server.app.package/PID>} Blocked
W/ActivityManager: Unable to start service Intent { act=<action> pkg=<server.app.package>} U=0: not found
```

To make it funnier, the Service can be easily started with from `adb` and the same intent - there is no error :)

> This change in Android does not affect the launch of internal Services, it is only about 3rd party Services, outside the application.

# The solution
All you need is to add `<queries>` in client app Manifest, where you want tu bind 3rd party app Service. With the same package name you set in the Intent:
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

[My repo with a working example](https://github.com/asvid/Android-Services-Sandbox)

Alternatively you can stay with `compileSdk 29` but I don't recommend that :)

### Rant over Android
It took me far too long to find the cause of this problem... The `bindService()` method only returns `True / False` with no specific error message, although it can throw a `SecurityException` in the absence of permissions. I had to engrave in logs without an application filter. Even this does not help much, because the information "service not found", and which can be launched with the same Intent with `adb`, only intensifies the confusion. [Blogpost](https://medium.com/androiddevelopers/package-visibility-in-android-11-cc857f221cd9) with an explanation of the changes, does not mention `bindService()` at all, and I haven't found a clear answer to my problem anywhere.

Only painstaking work through the documentation of the changes introduced by API 30 led me to the solution. My guess is that there are some security reasons to prevent malicious applications from trying to launch other application Services and not to make it easier to use them... but come on. I just bumped the `compileSdk`, and without any warning or explanation my app stopped working as expected.

There is so many API changes from version to version in Android that a single change can easily escape, even despite following the news. It is also difficult to remember the impact of each change on elements of the application that are not currently in use. And finding a solution can take a long time if the problem is unusual and the development tools are not helping.