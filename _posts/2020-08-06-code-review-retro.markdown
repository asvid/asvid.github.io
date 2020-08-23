---
layout: post
title: "Code Review Retro"
date:  "2020-08-23 11:43"
description: "This is not a list of 'golden rules' of code review, there are many blogposts about it. Here I just try to perform retrospective my own experiences."
permalink: "code-review-retro"
comments: true
toc: true
tags:
- code review
- self-retro
- pull request
- team work
categories:
- rss
image: /assets/posts/codereview-retro/inspection.jpg
---

# Importance of Code Review

I sure do hope that I don't need to convince anyone how important is the process of code review.
If you need some guidelines on how to do it well I can recommend [this Yelp blog post](https://engineeringblog.yelp.com/2017/11/code-review-guidelines.html)
Here I'd just like to perform a retrospective of my own `code review` experiences.

### A bit of history

When I started my developer career in a rather small company - we were not doing code reviews. One reason was
that we had one developer per technology, so it was hard to find reviewers. But the main reason was that it was the first job for most of us, and nobody had experience with it.
There was nobody to say: **"we should be reviewing pull requests, despite we are not experts in a certain technology because good code always fulfills the same criteria"**.


Luckily, later on, I worked in place, where code reviewing was a standard, and I learned a lot thanks to it.

# Let the hunger games begin

Pull request is always a bit of a fight - [Yegor Bugayenko](https://youtu.be/DUWiy7fvVzk?t=81) explains it well.
As a code author, you want your changes to be merged, but as a reviewer, you protect the project from attaching any mess to it.
> The most important thing is to remember that both parties have the same goal here - keep the highest code quality in the project without pushing a particular solution by any means necessary

Pull request's purpose is to verify that new code is written according to conventions agreed in the organization (code style, tests),
doesn't include any dangerous changes or silly mistakes or trash left after development.

So much for the theory, now practice...

## Reviewers hat

First some observations about reviewing code

### Lol

I got a comment like this once during code review :roll_eyes: `lol`. Ok, cool. But what does it mean? I did something funny? Something silly?
Now I have to answer this comment and ask, hoping that its author will reply (in a different way) and then eventually, if I get lucky, I will know what he meant.

Let's respect each other and our time that our employer pays for, and try to write **as clear as possible** what is going on from the very beginning.
Code review is a great place to discuss the very matter of our profession - the code - but it's hard to discuss with `lol`.

### Memes

I add them myself in comments, but there are some rules:
- reviewer and PR author should know each other well, to be sure that meme will be understandable
- `enough is enough` well-placed meme from time to time can be great, but if you put it in every comment it looks unprofessional and will obfuscate useful information
- there are more and more topics that are better off left for after work, and PRs can often be publicly visible for the whole organization. Not everyone is a fan of non-PC humor

### Personal preferences

"And here you can do it `like this`" - where `like this` often gives the same result, just in reviewer's preferred way.
In extreme cases, it may require refactoring half of the project and changing its principles because reviewer just came back from random conference and is spoiled with hype-driven development :smile:. 

The reviewer's solution was likely taken into account during development, but eventually, it didn't make it to the final solution. 
PR's author may know something the reviewer is not aware of.

Such comment can be useful and start a fruitful conversation for both sides, **as long as the reviewer is not blocking PR** by all means necessary.

### Imperative

"Change `X` to `Y`" - no explanation or reasoning given. Because. The reviewer knows best.

IMO this way of commenting is unacceptable, even when there are huge experience and knowledge gaps between author and reviewer.
When a junior developer is not getting any explanation why `X` is worse than `Y` they will **never learn or will use `Y` blindly** "because senior said so"

Usually, 1-2 sentences of clarification will do the work, StackOverflow, or some blogpost link may also help.

### Esseys

But don't get too verbose, `Ockham's razor` is your friend.
It's an extremely important and useful skill to be able to pack maximum content in a minimum amount of well-picked words.
There are times when I have to crawl through epic in the comment, just to get information that I could use another name for a variable.

What I would expect is to point what variable name may be better, why current is bad and 1..n suggestions of better names.

- I don't need an introduction like "Don't you think that this name is too much/not enough something?" - No, I don't and this is why I picked it.
- Then I don't need to get 10 links to StackOverflow and suggesting 3 books about clean code and software engineering - I won't be able to read all this in any reasonable time.
- In the end, I don't need to read that "this is just a suggestion, and I will do as I want if I think that current name is better, but {here more useless words}" - 
yes I know, every comment in PR is just a suggestion.
- Every reviewer's comment is kind of `call to action` and author reaction is expected, no need to add "what do you think about it?" every single time.

I understand that this kind of verbose language makes comments sound nicer than just dry information, but it's exhausting for long the term.
If we agree that we are all professionals and are respectful towards each other, then I think we don't need such verbose language in comments.

This is also shadowing the main reason for the comment itself.

So please be specific :smile:

### Author is doing the same mistakes all the time

People keen to learn from mistakes and are adjusting to organization standards.
If somehow PR author is doing the same mistakes each time or is not following code style - you should act on another level than just comments.
Often by saying "this guy doesn't get it" you mean "I just can't explain it well".

It is worthwhile to first talk with them about whether it is because of absent-mindedness or deliberate breaking of conventions, because, 
for example, he may think that his are better.
In the case of absent-mindedness, you can try to automate rules and standards - there are tools like code formatter, SonarLint, or rules in IDE that will highlight broken rules in real-time, long before PR.
When the author is fighting with organization standards... well not everyone is good to work with :roll_eyes:

I believe that imperfect rules are better than their absence, and it's worthwhile to work on them rather than fighting against them.
Deliberate breaking organization standards instead of at least trying to fix them is causing chaos and unnecessary tension.

### Appreciate

When you see a cool change in the code - **appreciate this in the comment**, :+1: it's enough.
PR approve means just "good enough", but if the author did something interesting and went the extra mile it's really good to appreciate this.

It creates a nice culture among developers and means that the reviewer actually read the code and not just scroll through it and approved.

However, it is important to not overdo this, or it will lose impact.

### Wider context

The easiest part of reviewing code is catching typos, naming issues, or little implementation improvements.
It's because **these problems are isolated - they are present in a single place in code**.

Much harder but also much more important is to understand the wider context of changes in PR.
It requires a deep understanding of project code, business domain, and task that is being resolved by reviewed changes. 
This comes over time depending on the size of the project and the complexity of the domain.
You also need a good team working together to achieve sprint goals and that is actively involved in all tasks. 

Group of alienated contractors probably won't be able to review code looking in a wider context and will focus on basic issues mentioned at the beginning of this chapter.

### Always leave a comment

I don't like `silent approvals`, even though I sometimes do them :smirk:
When changes in PR are clear, and you don't find any issues it's good to say this in the comment to the whole PR.
Simple "looks good" is enough.
Additionally, it means that, as a reviewer, you spent some time to understand changes, and you don't just want to `approve` quickly because the author is bugging you to finally look at his PR.

When you see a trivial issue in the code - also leave a comment and don't pass over it, but with information that it's a trivial issue.
It's not about being picky, but let know that something can be done better.

------

## Authors hat

Pull request author has a lot of ways to make code review a good or bad experience.

### Description

I've noticed that I like when PR has a description informing about the intention of changes and is linking to the most important changes in code.

Usually, in the PR description or title, you should find a link to Jira ticket (or any other ticket system), but we all know how tickets can look like.
Then you have undocumented discoveries during implementation and encountered problems with solutions.

So to help reviewers to understand the wider context (check the previous chapter) **its good to write in human-understandable language what is going on in pull request**.
Of course, that code should talk for itself... but code diff may not do this well enough. 
Also jumping between code in IDE and pull request is not pleasant.
This makes the review process longer and is causing aversion to reviewing pull requests.

It's worthwhile to link to the most important changes in code, to distinguish them from minor gardening or fixes.
In the description, you can add screens (check below) or diagrams that will help to better understand complex changes, that may be hidden when looking only on diff.

### Screens

When your changes are impacting UI consider adding screens with the effect to pull request description.
Even if there was a designer involved in the whole process.

It's fairly easy to miss something and adding screens will allow reviewers to find errors without running the app.
From an Android developer perspective - sometimes the design is not working well on all devices and if this is discovered during code review, changing it will be faster and cheaper than doing this after users start complaining.

### Self-review

**Before you add reviewers to pull requests - do yourself a review**. You might want to check:
- did you leave any development garbage in the code (logs, comments)
- can you figure out what is going on just looking on diff
- do names for class/variable/method picked up during development still fit
- are tests reasonable
- are organization conventions and standards kept

During development, it's easy to miss or forget one of the above-mentioned things. Before adding reviewers to pull request you have last chance to fix it before anyone can see it :smile:.

Some organizations or developers themselves are using pre-pull request checklists that are saving reviewers time by finding silly errors before they start checking the code.
Surprisingly this is also making development faster because you don't have to wait for feedback from others to fix issues you will have to fix anyway.

### Feedback is a gift

By writing the code, you expose yourself to criticism and have no influence (or just indirectly) on the form in which you receive it. You control only how you react to it.
Even though you disagree with the reviewer's point of view, appreciate fact that they spent time writing a comment with (hopefully) reasoning of their point, preceded by deeply reading your changes.
Unless they wrote just `lol` then just move on or escalate it :no_mouth:.

It's good to respond or react on each comment in some way ("like" comment on BitBucket, or reactions on GitHub).
Leaving hanging comments creates the impression that reviewer work is irrelevant.

### Lower the noise

Nobody likes to review pull requests containing changes in 100 files, resolving many issues at once and touching unrelated places in code.
If you feel that in your organization there is a problem to motivate other developers to review your PR, maybe you can start creating them in a way that would allow fairly easy readthrough and understanding intention of changes.
Tasks can often be split on smaller independent fragments and then you can create few PRs with changes focused on part of the issue.

Unfortunately, it won't be always an option.
Then you can try to **stop fixing the world by the way**, especially when you notice that you are changing places unrelated to your task.
PR can easily explode to enormous size, where hundreds of files are changes in a dozen places.
This phenomenon is making the intent of changes blurry, and it's hiding the most important changes from reviewers. Not to mention that it allows bugs to slip in easily.

It's better to create yourself a ticket for later and try to do it ASAP, or just do a separate PR within the original ticket but just with refactoring you found to be useful.

If for some reason you end up with a huge pull request, please add a good description with links to most important changes in code - this is where you want reviewers to focus.

### Pull Request doesn't exist in a vacuum

Good PR is not alive for a long time. When you have weeks or months passing since it was created, the chance of resolving conflicts grows exponentially.
And this is tiring and frustrating...

This is why **small pull requests are awesome**, because reviewers will check them sooner, the number of potential issues and fixes is lower, and they are merged quicker.

It's becoming even more important the more developers work at the same project in independent teams.

# Summary

We all deeply care about being productive at work, despite on which side (author or reviewer) are we.
Commonly each developer works in each role, so it should be easy to understand how important **empathy and communication** are. 

Let's write in such a way that others can easily understand what you want to convey, regardless of whether it is a code or a comment. 
A well-prepared pull request will be verified faster and more thoroughly. 
Respectfully written comments will work better for project code quality than those mocking the author.

When working with people it's important to remember that it should be pleasant for everyone :smile:
