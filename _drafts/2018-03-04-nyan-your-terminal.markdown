---
layout: post
title: "Nyan your terminal"
date: "2018-03-02 08:23:03 +0100"
description: Some tools I use with my terminal to make my work more efficient.
permalink: nyan-your-terminal
comments: true
crosspost_to_medium: false
tags:
- Linux
- terminal
- plugins
- tools
- Nyan Cat
categories:
- Tools
- Linux
image: /assets/posts/nyan-your-terminal/nyan.jpg
---

* TOC
{:toc}

*Disclamer #1: I'm an Android developer, not sysadmin. I use terminal to help me with my workflow, not as my main tool, so please keep this in mind :)*

*Disclamer #2: This is based on Linux (Ubuntu), but most stuff should work on OSX also.*

# Terminal?
If you are a developer (and not a Windows peasant), you might had to use terminal for day to day stuff like installing node.js or python packages, or running scripts. But there is a lot more in this simple text user interface. Imagine you could do anything, and I mean **ANYTHING** you can do with GUI software or even more, in single window, just by typing orders. Unlimited power (and responsibility also...) available for those who are not afraid.
Also, using terminal is something that distinguishes casual PC user (those internet == Facebook type) from true super users.

# Anything? I want NyanCat
We'll get to that, don't worry. But you have to eat your meat before you can have pudding [^pink_floyd]. You know already how the terminal window looks like, right? It's pretty boring and not really encouraging. But we can make it better, both looking and functional.

## Guake
First thing I get on fresh Linux installation is Guake terminal. Not only it reminds me one of my favorite games, it's also a productivity monster. This is terminal that overlays any application, provide tabs, lots of customization options. For more info check website [Guake](http://guake-project.org/). If you are using Ubuntu for install you just type `sudo apt-get install guake` in your current terminal window, and then type `guake` to run it and `F12` to show it, should look like this:
![Guake terminal image](assets/posts/nyan-your-terminal/guake.png)
How cool is that? :) It's a good idea to add `guake` to start programs, so it will always be turned on when you need it. Let's customize it a bit. Right click anywhere in guake terminal and then find and open `Preferences`. Lots of interesting and confusing stuff, but first I recommend to change some keyboard shortcuts. `F12` for me at least is not optimal way to open terminal. Some people have it set for tilde key (this one below Esc) because its rarely used but easy to reach. I personally prefer to have `` ctrl + ` `` so I don't loose backtick sign access (that is pretty useful when writing in Markdown... like this post).

### Tabs
Another thing is creating and closing tabs - we can have many terminal windows open in bars and we should use it! I use `` ctrl + t `` to open new tab, and `` ctrl + w `` to close it. Ok so we have tabs, you can switch them by clicking on them... but wait, we use terminal, why even bother with mouse? I've set switching to next tab to ``ctrl + right arrow`` and ``ctrl + left arrow`` for previous one. There are also setting for selecting tab you want i.e. third from left, but I don't really use them.

### Themes
At first Guake may not look any more interesting than standard terminal app, but it has build-in themes that you should check out in preferences. All popular ones are here like `Monokai` or `Solarized`. I used to be a fan of Hombrew (this green Matrix-like style), but recently I use `Spixel`. Choose whatever makes you happy, but don't forget about adjusting transparency of terminal. Also remember main purpose of theme is to make text in terminal be readable in pleasant way, not just look good :)

## Zsh
You are probably familiar with `bash` because it's default shell in most Linux distributions. But it's not the only one, actually you can have many at the same time. This chapter will be focused on shell called `zsh (Z shell)` and tool `Oh-My-Zsh` with its plugins. You can find details on how to install both of them [here](https://github.com/robbyrussell/oh-my-zsh/wiki/Installing-ZSH). After installation, you have to switch your `guake` default shell to `zsh`

### plugins

## Other Tools

### ccat

### htop

### bmon

### Images in terminal

## Fun stuff

### Terminal Slack

### What does the cow say?




[^pink_floyd]: Pink Floyd - [Another Brick In The Wall](https://www.youtube.com/watch?v=YR5ApYxkU-U)
