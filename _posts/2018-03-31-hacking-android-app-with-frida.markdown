---
layout: post
title: "Hacking Android app with Frida"
date: "2018-03-31 17:43:37 +0200"
description: Quick walkthrough of using Frida to hack Android app behaviour
permalink: android-frida-hacking
comments: true
crosspost_to_medium: false
tags:
- Android
- Kotlin
- Frida
- hacking
categories:
- Hacking
- Tutorial
image: /assets/posts/hacking-android-with-frida/frida.jpg
---

*Side image is of course [Frida Kahlo](https://en.wikipedia.org/wiki/Frida_Kahlo) auto portrait, besides her name she has no connection with topic*

* TOC
{:toc}

## Motivation
Lately I attended to `Sekurak hacking party` - it's event organized by [Sekurak](https://sekurak.pl/) where they show how easy is to hack stuff like IP cameras, routers, phones. I guess Sekurak is known mainly in Poland, but they are real professionals in area of security. During this event Michał Bentkowski was showing how easy it is to spy on Android app communication and also change app behavior using tool named Frida. It was cool but scary at the same time from developers perspective.
Gladly in company I work for, we have `Community of Practise` so with my colleague we've decided to make similar show about hacking Android apps for others during next CoP meeting. I've extended a bit things that Michał showed at Sekurak meeting and it took me some time to find them on various blog posts or Youtube videos, so I'd like to share.

## Preparation
Even if Frida is pretty easy to use, there are some steps you will need to take to make it work.

### Getting ready

#### PC
- python installed - I've got 2.7 and it works just fine, not sure if there are any issues with Python 3.x
- pip installed (its now bundled with Python distributions)
- reasonable console - if you are on Linux or Mac you already have one, for Windows...I use ConEmu with git bash commands installed
- ADB working from console - you already have ADB installed with Android Studio (well Android SDK to be accurate), but if you haven't been using separate console app it's possible that it's not added to PATH

#### phone
- Frida works on Android OS between 4.2 and 6
- it needs to use Dalvik, not ART
- it should be rooted - well... there is a way to avoid this but I did not check it.

To achieve it all pretty easly and cheap, I just used emulator :) with following details:

```
Name: Pixel_2_API_22
CPU/ABI: Google APIs Intel Atom (x86)
Target: google_apis [Google APIs] (API level 22)
```

#### you
- you need to know what architecture your Android device, for my emulator it's x86
- if you already have neat console app, it would be super cool to know how to use it
- also basics of Python and JavaScript will help - if you know ANY other language it will be just enough, we wont be making enterprise scale banking app, just simple hacking scripts

### Installing stuff
To install Frida on your PC just go to console and type
`pip install frida`

Now we'd like to send `frida-server` to our device and run it, so Frida on our PC can communicate with it. On [GitHub release page](https://github.com/frida/frida/releases) are versions for all possible uses (also Windows or OSX), but we are hacking Android so we need to find `frida-server-10.7.7-android-x86.xz` or newer, but always exactly for our device architecture.
Now unpack the archive and send `frida-server` file to your device using:
`adb push {frida-server-file-name} /data/local/tmp`
So Frida is on device, but not running. Yet. We have to go to device shell, change file permissions, and run it like any other executable type on linux machine.
- `adb shell` will get us to device shell
- `cd /data/local/tmp` will take us to where we've send frida-server
- (optional) `mv {frida-server-file-name} frida-server` to change file name for easer to use, without version and architecture name
- `chmod 755 frida-server` to change permissions
- `./frida-server` to finally run it

Notice that you wont get any info in console, it will just start running. Every other command should be run from separate terminal. It's also a good idea to open logcat in separate terminal `adb logcat`.

### Hello Frida
We are all good to go :) Let's check if Frida on PC is getting along with Frida-server on device:
`frida -U asvid.github.io.fridaapp` - It's necessary to know full app package name, and to have app running on device. Flag `-U` tells Frida to check the USB connected device, or in my case, Android emulator.

Console should now look like that:
```
frida -U asvid.github.io.fridaapp
     ____
    / _  |   Frida 10.7.7 - A world-class dynamic instrumentation toolkit
   | (_| |
    > _  |   Commands:
   /_/ |_|       help      -> Displays the help system
   . . . .       object?   -> Display information about 'object'
   . . . .       exit/quit -> Exit
   . . . .
   . . . .   More info at http://www.frida.re/docs/home/

[Android Emulator 5554::asvid.github.io.fridaapp]->
```
We are officially in. Now we can type commands here and our app should obey. Lets just do something simple:
`Java.androidVersion` will just return in console which Android is our device running. Pretty...lame. Lets run something better:
`Java.perform(function(){Java.enumerateLoadedClasses({"onMatch":function(className){ console.log(className) },"onComplete":function(){}})})` - now we are printing name of every object that Dalivik has created. And if we have object instance, maybe we can run its methods? We'll get to that shortly.

## Action
Running frida commands in console is a bit annoying, it's extremely easy to mistype or forget to close some braces.
Other way is to write script to file and run it by Frida.
`frida -U -l script.js asvid.github.io.fridaapp`
It can be even better with Python script that will run JavaScript script.
Why we need Python and JavaScript if we want to hack app written in Java or Kotlin compiled to bytecode and that runs on Dalvik Virtual Machine? Well Python is just to make it easer to run JavaScript files, that will fool Dalvik to run them instead of app instructions.

### Repo
I've made very simple app for hacking with Frida demonstration that is available [here](https://github.com/asvid/FridaApp). There is signing key provided so you can build yourself a signed release APK, but since it's not obfuscaded in anyway by ProGuard hacking scripts will work the same as on debug version.
In `tools` directory we have `dex2jar` and `jd-gui` that we wont be using this time, but they are worth checking if you like to hack APK without having it's code. For now all we are interested about is in `frida-server` directory.

If we look into `script.py` we can see that it does similar thing as we did in console, it looks for `pid` for our app package and attatches JavaScript file to run with it. It also have all JavaScript files already listed, we will go through all of them. To run this script just type `python script.py`.

### Change method implementation
First script shows how we can change method implementation. Sample app is running method `sum()` every second and loggs output. We can change it's implementation without app even noticing it :) In `change_method.js` file in `Java.perform()` block we first find class that contains `sum()` method - we have whole app project so it's not hard to find methods and classes, but if we want to hack someone's APK, even after obfuscating with Proguard it works the same way. APK can be extracted with `dex2jar` tool.
`Java.use()` returns us class object with access to it's variables and methods (note that it is NOT an instance of this class). To change method implementation we just need to overwrite it with new method.
```
activity.sum.implementation = function (x, y) {
        //print the original arguments
        console.log("original call: sum(" + x + ", " + y + ")");
        //call the original implementation of `fun` with args (2,5)
        var ret_value = this.sum(2, 5);
        return ret_value;
    }
```
Interesting thing is that we can still use `this.sum()` - it's because we haven't change original method or class, we just told Dalvik to run our JavaScript instead. In this case we are printing in console original method parameters and then run it with completely others, and return value.
At this point I felt absolute power!

### Run method
We've changed method implementation, but we still needed app to run method, it's because we didn't have access to actual object. But we can have it, why not, we are world class hackers at this point. In file `instance.js` we use a bit different method `Java.choose()` that looks for loaded objects and select one with fitting name.
```
Java.choose("asvid.github.io.fridaapp.MainActivity" , {
      onMatch : function(instance){ //This function will be called for every instance found by frida
        console.log("Found instance: "+instance);
        instance.showToast();
      },
      onComplete:function(){}
    });
```
Method takes two callbacks: called when instance is found and other called when method is completed. In first callback we get access to instance of class we were looking for, and we can call it's methods. Remember that it means method will be called when instance is found, not at start of your app, because looking for instances takes a while.
So YAY! we've showed a toast.

### Bruteforce PIN breaking
In some apps like `Evernote` you can set internal PIN to protect your data from unwanted access. It means PIN has to be stored locally on your device, encrypted of course and hashed. When user provides PIN you just compare hashes, so no plain text PIN is available at any point.
In FridaApp I've provided just simple method that checks if PIN is "1234", but image that its reading hash from SharedPreferences and compares it with hashed user input. No matter what security precautions you take, at the end of a day you want method that takes a `String` and returns a `Boolean`. Lets hack it!
In `brutal.js` we have our PIN destroyer. At first we need instance of class that contains PIN checking method. Than we can run this method for diferent PIN numbers and check which one returns `true`. I've done it in a loop that iterates from 0 to 100000, so it covers all 5 digit PIN numbers. Loop stopes when method returns `true` and prints number in console along with time it took.

### Listing loaded objects
We've already done this in console, but there we've go ALL of objects, it was impossible to find what we can hack. Gladly I've wrote `class_list.js` script that can list only ones in my app.
```
Java.enumerateLoadedClasses(
      {
      "onMatch": function(className){
            if(className.includes("asvid")){
                console.log(className);
            }            
        },
      "onComplete":function(){}
      }
    );
```
Yes in console you can use `| grep asvid` but its usefull to do it inside script in case you would need to use it further.

### Imaginary Security
All previous scripts were working arount `MainActivity` class or instance, it's safe to assume 99% of apps has such class. But now since we've listed loaded objects we can see there is mysterious `Security` object loaded. If we check app code, we can see its a field in `MainActivity` and its used to encode text from input and stores it in `SharedPreferences`. It's also used to read and decode this text that is next printed below input. If you check `SharedPreferences` of sample app it might contain something like that:
```
<string name="password">cNeUD7lWCcd01X4aQfSsgi+YvbP2m9/I5f1zeLmDNmIzPV9A4oUYf68HsCa8r/st7zi8qZnpZLf8
+dPwTWuNZZSsKnFRHlDK7G8f23kq5vf6luIzrjob4ZzzVs/wi2iAEyDD482uQBxTxq9UlE0C2+Wb
Nw3tza5OTFIDoqf9HFBLJmqhKPXE7vmGp7XXJTdcHQhPfNeg/g0LzUXOXhFROUOOc8kzekwaBgmZ
aWf5g0prOONetqr7xRdR4VprNlBhZA0DmhbCXP4vLEj8FWadvkwvtfL7uLQviW0iUVAuGXBfLdCG
Qb7jUeX/jbWxSRLnyXZaUkG0VhuRQt7jXT2H7A==
    </string>
```
Looks serious, but why even bother cracking it when we can read it like it wasn't even encoded? In script `security.js` I'm using some old tricks: `Java.choose()` to find instance of `Security` class, and running method `getPassword()` on found instance.

### Let there be instance
In sample app there is one class left - `SomeClass`. But it wasn't listed on loaded classes, in Android Studio its name is gray so it's not being used anywhere. Is there a way to use it somehow? Yeap, we just need to create instance of it. There is a log of code in `some_class.js` script but most important is this:
```
var someClass = Java.use("asvid.github.io.fridaapp.SomeClass");
var someClassInstance = someClass.$new()
```
So here I'm getting a class that I want to create instance of, and just... create an instance. And now I can do whatever I want with this instance.
At first script is just printing in console output from methods with same name but different signatures - easy. Then it's changing both implementations with help of `overload()` method, because we need to specify which ones implementation we are changing.
Finally, script is printing values in public and private field of our `SomeClass` instance. `Private` doesn't really mean anything for us now, script will change its value (which was also `final`) to anything we order it.

### Debug release
Last script is inspired by silly idea, that developers can hide things from users simply checking if build is `debug` or `release`. Like additional buttons used for testing, app logic changes or logging. Remember that in `MainActivity` there is a method that runs every second and prints in console result of adding 30 and 50, we were changing this method implementation before. This method uses two types of logging, standard `Log.d()` and `Logger.log()`. This second method is checking build type, and prints only for `debug`. So if you build release app you will see only first log.
Script `debug.js` is at first looking for `Logger` instance in memory, and then changes its `showLogs` flag to `true`, from its original value of kinda `isDebug()`. And now we get both logs in logcat.

## Summarization
I've tried to show some basic functions of `Frida` with easy to use scripts. Hope you had fun and will experiment on your own apps, I surely will. Real world app would have obfuscated code, so it will be much harder to know which class and method you need to use to achieve what you want, but it will still work the same way. I'm just an Android developer - not really a hacker, or security guru, but learning about `Frida` pushed me to think way more about my apps security. It's always better to break (and fix) your own app before someone else does it.
