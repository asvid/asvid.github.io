---
layout: post
title: "Would you go for mobile development again, after 10 years of doing it?"
date:  "2023-01-29 16:56"
description: "
I went to a meetup in my city last week and a guy I just met there popped a question after a while of conversation: with my experience, would I go for mobile software development again or pick a different area?
"
permalink: "would-i-do-mobile-again"
comments: true
toc: true
tags:
- career
- mobile
- android
- ios
- flutter
categories:
  - rss

image: /assets/posts/go_mobile.png

---

## Would I?

I went to a meetup in my city last week and a guy I just met there popped a question after a while of conversation: with my experience, **would I go for mobile software development again** or pick a different area?


## And it got me thinking

![My answer in form of meme](assets/posts/mobile-meme.jpg)



**Yes, I would do this again, I'm even more motivated now.**  



I remember how excited I was about the idea that you can write software for a tiny computer that lives in your pocket. The possibilities you can give to people, feed them with information at any time and place, use data from GPS, accelerometer, or camera... it was so much more than web apps that I've been working on previously. And the UI had to be adjusted because screens are tiny, so a whole new design language was created over time.  
It was a wild-west greenfield, nothing was written in stone. 

Some things changed over the 10+ years since I started, but the excitement grew bigger.

## Here is why
> I'm talking from an Android dev perspective, but some points are true for any mobile OS


#### 1. We all use mobile devices
I mean personally, with our own hands. We use backend services as well, but it's not that visible. 

Apps are often the only user-facing entry point of a complex system, that can make or break the whole business. Apps are always waiting in the user's pocket and offer a better experience than browser apps. 

We are paying for groceries with our phones, editing clips for Instagram, scanning invoices for accounting, recording or writing notes, managing todo-lists, listening to music... the number of stuff we do on mobile devices is countless.

There are people that don't own a computer anymore, all they need is a smartphone.


#### 2. It's easy to show
This is kinda true for all front-end tech, but for websites, you need to host it somewhere. For Android, there is no need to publish your app or even have a Google Play account. You can just pop out your phone and show your friends what you are working on and gather feedback and ideas:)


#### 3. Tools got decent over time
Just to mention way better device emulators, Kotlin instead of Java, Jetpack Compose instead of XML views. Error reporting, logging, test running, app deployment, etc.


#### 4. Mobile is not just phones
There is Android Auto, Android TVs, wearables, card terminals, IoT devices… and more. It’s great if you like working with hardware, but not all the way (embedded low-level stuff). I once wrote an app that was talking via Bluetooth to an Arduino controller for cars' pneumatic suspension. Try doing this with Spring Boot.


#### 5. We got fairly good standards
Of quality, architecture, designs, and testing. **I honestly sometimes think we are rediscovering what backend/desktop devs already did** in the past 50 years, but we are getting there way faster. Mobile technology matured a lot. This allows us to focus on what is important (the User experience) without struggling with basics.


#### 6. Cross-platform solutions
As a native Android developer with PTSD from using Cordova/Phonegap, I was very skeptical about any cross-platform solution. But there is Flutter now… and it runs nicely on both platforms (and possibly on the Web, but I didn’t test it). Of course, there is the always-present cross-platform issue with the need of implementing pieces of native code and bridging them to the shared parts, but often you don’t have to do it, depending on the app.


#### 7. It allows me to work in other areas
As mentioned before, I'm Android developer, not backend, but I’m fairly confident about working with Spring services. Modern backend apps use Kotlin so the doors are even more open. I found it valuable to be able to work outside my comfort zone, and it’s highly appreciated in the workplace. On the other hand, I’ve noticed a lot of backend devs are petrified of anything even remotely having UI or not running in a container :)


#### 8. It has multiple faces
There is more UI-oriented work, and there is more backend (on mobile) work. You can develop apps, libraries, or even tools. The whole spectrum of domains is available without prior knowledge. I used to work with IoT, fintech, e-learning, car navigation, and others. My colleagues worked in health, sports apps, tv streaming, and even a toothbrush app - beer conversations are entertaining.


#### 9. It's challenging
Even with tools and patterns, there are still a lot of unknown areas when you start developing an app. The UI or navigation may (and will) change to serve users in a better way, reacting to their feedback. Running your code on a device you have no control over is causing hard-to-track bugs. 
There are thousands of Android devices that your app should run without issues, with different screen densities and sizes. Android is known for OS version fragmentation and targeting only the latest one will cut out a lot of potential users. Some devices don’t use Google Services like Huawei, so you have to do workarounds to provide functionalities.

The OS is changing each year more-less, new APIs, deprecated ones, new requirements and guidelines to follow. Be prepared.


#### 10. Accessibility
The right setup of apps can allow sight-challenged people to see, read and write, or navigate to destination, amplifying their independence. With virtual assistants you can write emails, call people and perform other tasks without using hands. Thanks to mobile devices and accessibility supported apps disabled people can now have better life quality than before.

And while you most likely won't develop a next live changing app, following guidelines for accessibility will also make a change.


#### 11. The job market is good
This might be true for almost all sorts of software development. Because of the reason #1 mobile developers are always needed.

## There are some caveats though
### Mobile is considered front-end
The focus is on UX and not really on the scalability or maintainability of the whole system. And usually, you need the backend to even justify having an app. Depending on the project and organization, it may leave you as a mere consumer of the API without any impact on the whole system - I don’t enjoy that. I remember when the app I was working on was called _“**just a skin**”_ for _**the real app**_ running on the server. It didn’t feel good.

### Users will always blame the app
Not the backend services, not the infrastructure, not the security team neglecting dangers, not the DevOps messing something up, not the managers, product people, or designers for wrong decisions. It will be your baby app getting dreadful reviews in the app store. There will be also kind reviews, don’t worry :)

### Career limiting?
At some point, I’d like to design systems. But I don’t have much experience with the backend, I’m a bit scared of the infrastructure setup, binding everything together, handling AWS, containers, orchestrators, load-balancers, deployment schemes, etc. I understand the concepts though. I can learn that stuff but most likely I won’t get much experience. Most system architects (I guess) are former backend devs. Rarely there are mobile ones that step outside their comfort zone.

Of course, there is also a need for mobile architects in bigger organizations or complex apps, but it's a bit boring for me, I like playing with bigger boxes.

I believe doing mobile may give me some advantages that hardcore backend devs won’t have. Understanding the user perspective, UX, connectivity issues, and backward compatibility for APIs are just a few to mention. But still, there are huge gaps to fill.


## Would I recommend mobile development to others?
**It depends on you, but generally, I would.** 

I saw skeptical backend devs getting excited after a while because they could develop the button that was calling their API and see the result :)
We have UI, UX, and App architecture patterns, no need to reinvent the wheel. Multiple paradigms were announced, tested, hyped, and forgotten. It’s a stable and mature environment now. Just like the tools we are using.

If you already know Java or Kotlin it may be a safe and fun way to develop additional “leg” in your skill tree and get a feeling of how the whole system is working.

If you are just starting **don’t you want to run your code on your phone?** :) Play with a camera or GPS? Design a smartwatch face or complication? A widget for your home screen?

**Then go for it!**


------------


Thanks [Mariia Saveleva](https://lyumotech.com) for reviewing and tip about accessibility :)  


