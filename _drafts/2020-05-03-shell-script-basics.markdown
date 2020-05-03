---
layout: post
title: "Shell script basics"
date:  "2020-05-03 12:14"
description: "If you are lazy and you know it clap your han... write shell scripts"
permalink: "shellscriptbasics"
comments: true
crosspost_to_medium: false
tags:
- shell
- bash
- automatization
- scripts
categories:
- programming
image: TODO - add post image from assets
---

* TOC
{:toc}

## Why
Recently I had to do some tasks that required me fallowing a strict checklist. It wasn't anything complex, no creativity required - just do as list tells you to do.
I was also one of the creators of checklist.

I had to do it few times, almost regularly, sometimes in a short notice but nothing extreme.

So **obviously** I made errors, because I knew the checklist so well **I wasn't paying attention**.

## No more errors
After a few failures that were result of my *(looking for excuses)* being bored with checklist I decided to write some shell scripts that would prevent me from doing almost anything - so prevent me from making an human error.
I'm not an expert but during this process I've learned a few things, here they are:

### Add Help
Even when your script doesn't have any switches or flags, doesn't accept any parameters etc. it's really good to add simple help under ```-h``` flag.
Yes you can keep scripts in repository as part of project they are used with, or separately, and add **READyouME** chapter but trust me, nobody will look there.
If you are using terminal you are already used to use this flag to get help about command or tool.

Here is one of ways to achieve this:

{% gist ffb2105cbe60fe2364ccddd97a7a8291 %}

There is a method ```Help()``` that prints info about your script. You can use ```echo``` for printing line by line, but using ```cat << "EOF"``` makes it way faster to edit and more readable.
Then the switch-case is looking for arguments, for now we are interested in ```-h``` which will run ```Help()``` and exit the script

### Handling arguments
You can already see some of it in previous Gist, here some more cases:

{% gist d2882cdd9cbc9860ef7f327a4e777b99 %}

Note that we have few types of arguments passed here:
- simple flag like ```-a``` can be used to set in-script variable that can be then used in ```IF``` statement
- flag with value like ```-p``` or ```--path``` can take string (and only string) argument
- all other arguments will be sent to ```OTHER_ARGUMENTS``` array

```shift``` after each case is handled removes argument from further processing. For flag with value it needs to be doubled because you want remove flag itself and value from processing.

It's not necessary to have both short ```-p``` and long ```--path``` flag names covered, but it's a good habit.

### Adding some colors
I love colors in terminal :) and your scripts can also make use of them

{% gist d605e2451ec13e544a58296ffa831825 %}

It's important to add ```${NC}``` at the end, otherwise next printed line will be in last used color

### Handle user input
To handle simple **yes/no** from user you can do:

{% gist 4975699f4f7694ebfb9d8fa612283b11 %}

### IF|ELSE

General idea of IF looks like this:
```shell script
if [ <statement> ] # <-- beginning of statement
then # <-- IF something THEN do something
  <do some work> # <-- actuall work
fi # <-- closing IF statement
```
Indenting doensn't matter, but it's worth keeping it consistent - remember you are doing this to **avoid** silly mistakes.

A bit more comple example:

{% gist 0eff5148b4c72710f49525e944c799c5 %}

In this gist there are few examples of IF statement:
- ```; then``` is moved to same line with statement, it looks more like we are used to in "normal" code
- negation with ```!``` inside condition block
- nested IF
- ```elsif```
- combined conditions with AND ```&&``` but could be also with OR ```||```

List of typical operators:

| Operator                  |	Description|
|---------------------------|--------------|
|! EXPRESSION               |	The EXPRESSION is false.|
|-n STRING |	The length of STRING is greater than zero.|
|-z STRING |	The lengh of STRING is zero (ie it is empty).|
|STRING1 = STRING2          |	STRING1 is equal to STRING2|
|STRING1 != STRING2 |	STRING1 is not equal to STRING2|
|INTEGER1 -eq INTEGER2 |	INTEGER1 is numerically equal to INTEGER2|
|INTEGER1 -gt INTEGER2 |	INTEGER1 is numerically greater than INTEGER2|
|INTEGER1 -lt INTEGER2 |	INTEGER1 is numerically less than INTEGER2|
|-d FILE |	FILE exists and is a directory.|
|-e FILE |	FILE exists.|
|-r FILE |	FILE exists and the read permission is granted.|
|-s FILE |	FILE exists and it's size is greater than zero (ie. it is not empty).|
|-w FILE |	FILE exists and the write permission is granted.|
|-x FILE |	FILE exists and the execute permission is granted.|

### Run commands

My favourite part of using bash scripts is how easy it runs commands. If you can run it in terminal - you can as easy run it in script.
```shell script
git pull || exit 1

eval "./gradlew" 'clean :app:assembleDebug --info'

open "app/build/outputs/apk/app/debug/"

(cd $SDK_LOCATION 	&&
	git checkout feature/awersome_feature || exit 1	&&
	./scripts/awersome_release_script.sh)
```

What we have here is:
- using git, ```|| exit 1``` means whole script will fail if there was an non-successful result of command which is pretty useful
- ```eval``` 

### Print Ascii header

http://patorjk.com/software/taag/#p=display&f=Big&t=My%20Shell%20Script

```
 __  __          _____ _          _ _    _____           _       _   
 |  \/  |        / ____| |        | | |  / ____|         (_)     | |  
 | \  / |_   _  | (___ | |__   ___| | | | (___   ___ _ __ _ _ __ | |_ 
 | |\/| | | | |  \___ \| '_ \ / _ \ | |  \___ \ / __| '__| | '_ \| __|
 | |  | | |_| |  ____) | | | |  __/ | |  ____) | (__| |  | | |_) | |_ 
 |_|  |_|\__, | |_____/|_| |_|\___|_|_| |_____/ \___|_|  |_| .__/ \__|
          __/ |                                            | |        
         |___/                                             |_|        
```