---
layout: post 
title: "Why should you use Tech proposals"
date:  "2022-04-04 11:20"
description: "
Tech Proposal are a less formal way for internal documentation created before actual implementation. They also facilitate making technical decisions.
"
permalink: "tech_proposals"
comments: true 
toc: true 
tags:
- documentation
- tech proposal

categories:
- rss

image: /assets/posts/archive.jpg

---

# "Post-design documentation"
As the lecturers maliciously called it when they checked university-grade projects, the idea of which was to plan a project, document it, and then implement it according to plan. In practice, I usually was implementing it first so it just worked, and then documented the result. And let anyone who hasn't done that throw a stone first :)

Lecturers (in Polish universities) are often older people with no commercial experience, used to water-fall project planning. Now we are all Agile, we have continuous delivery, self-documenting code, and other aids.

Is there still a need to write a plan today before the actual implementation, or is it a relic of the past, art for art, and a university fad? Honestly, I would often be grateful even for quickly written post-design documentation...

# Tech Proposal
`Tech Proposals` are a less formal way for internal documentation created before actual implementation. They also facilitate making technical decisions.

Creating such documents is useful when:
- we already have a 'system' and not just an 'application', i.e. many services and customers cooperating at different levels
- bigger changes must go through the architect or the security department (best regards, fintech)
- the new feature requires a lot of competence and cooperation between teams is needed
- we work in a diffuse and asynchronous environment

Seemingly, a lot may depend on the way work is organized, the number of teams and their composition, and the approach to the ownership of system fragments or procedures in the company.

But not completely. I worked in interdisciplinary (mobile, backend) and uniform (Android only) teams, in startups without procedures, and in more Byzantine structures. In fact, in every single place (with appropriate assumptions), the creation of a document that several people could edit, comment on, or simply review, significantly reduced the number of errors and misunderstandings. Even if nobody wanted to get involved in my document, at least I had my ass covered, because I shared it with everyone and waited a bit for feedback.

There are places where an analyst (and/or PO, TL) prepares tickets and programmers simply execute them without going into the whole solution. And it might work. However, it is unlikely that anyone will grasp the nuances of all the technologies used with an extensive system. It is not difficult for me to imagine a situation wherein the middle of the implementation of some part of the solution an unforeseen problem comes out, and you have to go back to planning.
Engaging developers sooner would have avoided such ping-pong.

I am aware that I promote a break away from programming in favor of additional bureaucracy and meetings. From my experience, at a certain scale and level of complexity of the project, it significantly **saves time and work**.

# Preconditions
To even start thinking about documenting the implementation plan or consulting it with other teams, you need to know well in advance what should be delivered. Planning work with the appropriate level of detail for a year, quarter, or month ahead helps here. Transparency within the organization is also very important.
I like to check in on other teams' demos to find out what they are working on and what they are planning to do. I can immediately offer help in an area that I know better, instead of hoping that everything will go well or that they will find me themselves. And **karma does come back**, other programmers themselves offer help when my team comes into their area of expertise. Don't get me wrong, there is no goodwill or empathy here, we just **prefer to be in control when someone rummages in our code**, instead of fixing someone else's errors ASAP or being blocked until they fix it. The whole organization benefits from it.

If there is a deadline for introducing a complex change in 3 months, then you can slowly start to estimate the time-consumption, break the problem down into smaller parts, maybe you will have to do some spike, talk to people or teams with competence in the subject, consult the 3rd party providing e.g. hardware - documenting each stage will end with a 'tech proposal' for the solution with the biggest problems already solved.

Procedures in the organization may require approval of changes by the architect or the security department. It is much easier to discuss changes when having a document, maybe with a diagram or API proposal. An architect can look at the 'tech proposal' at any time convenient for them, for example, coming down from an ivory tower. **It's easier to sell your ideas when they're presented in a digestible way**. The security department will also quickly point to potential problems or places that require attention, having some outline of the solution, not bare code that they may not understand.

# When is it worth creating a tech proposal
I've noticed that a good indicator is the emergence of doubts as to how to solve the problem or implement the feature. We don't have a full image before writing the code, but sometimes we manage to identify the biggest gaps well in advance.
Major changes may be beyond the competence of specific individuals or a team. Saved and shared information reduces the risk of time slips caused by staff shortages, the so-called bus factor. With well-written documentation and a committed team, each member can undertake the implementation of at least part of the solution, without having to wait until "that homie who knows what to do returns from vacation".

Self-documenting code is awesome, but if your solution extends beyond a single repository, it will never contain the complete image. Then it is better to create a document and (if necessary) add a link in the comment. The need can appear during a code review when someone asks "why so?". Instead of sending the same link to every potential questioner, it is better to put it in the code.

It is an unnecessary burden to create bloated documentation right at the start of a project. Then it is better to focus on using the `Decision Log` to store information about why a particular technology or solution was chosen. Getting into the legacy project without such information may introduce reluctance towards your predecessors :)

# What does a tech proposal consist of?
It depends on the problem, system, stakeholders... there is no single pattern.

I like it when it starts with the problem summary and the table of contents. This allows the reader to quickly find out if they are in the right place.

Then you can put in the introductory information that you have at the time of writing the document, links to other documentation or spike results. This section may grow with the time, don't be focus on getting all the info at first. Just write down what you already know and add new info as it appears.

The most important part is an adequately detailed description of the solution. There are a few schools of thought: you can literally put pieces of code or SQL there, limit yourself to specifying an API, or a verbal/musical description of the implementation.
Sequence or communication diagrams can be helpful and you don't have to do them in a perfect UML, everyone will understand rectangles and arrows.
In my opinion, the tech proposal should not focus on the implantation details, but rather on designing the logic and interaction between the various elements of the system. Implementation details will be a result of the adopted solution, organization or project standards, and the programmer's preferences.

It's also nice to have a few options for solving the problem and mark your choice in the summary, leaving the others in the document. This gives you a better picture of the thought process as you work on the right solution. Proposals for solutions may come from different people, or they can be a result of conversations with others, which helps you not to fall into the anchoring bias field. Sometimes it's worth putting a ridiculous solution in there, just to see how others stand out. But it also happened to me that the best solution was a mix between the first and the ridiculous one because it allowed me to think outside the box.
The description of each option may end with a small summary of advantages and disadvantages, even in the form of a table. Not all flaws will be discovered, but this document will help you historically understand the motivation behind your decision.

A 'tech proposal' written in this way naturally answers the questions: 
- What will be implemented?
- Why?
- What other solutions were considered?
- Why they were not chosen?

Example document may look like this:
```
Abstract
Table of Contents
Info
Options
 1.
 2.
 3.
Selected solution, summary, justification
```

Of course, in the future, it may turn out that the rejected solution would be better, but this is probably due to undiscovered conditions at the time of making the decision.

## Updating the document
`Tech Proposal` is a living document, it is constantly changing and developing. It's good if everyone can add a comment, but the document should have a single owner who keeps order and composition.

After implementation, the document does not die, but I don't see the need to update with every little change to the original solution. If the changes are large, they will probably require a new document, and you can link the previous one there. This document is for internal project documentation only and is valid only at the closure date. Documentation made available externally should be **always** kept up to date.

# Summary
A properly developed `tech proposal` allows not only to choose the best solution for the time being and gathered knowledge but also is a source of historical information on the development of the system. If you ever wondered "why exactly was something resolved this way" during your career, there would be an answer in the "tech proposal" (or "decision log"). The creation of such a document and teamwork on it is an additional effort that is justified only in certain cases. There is no reason to document every little change done to a single part of the system. However, it will be useful when the changes reach many services, exceed the team's competencies, and maybe need to be coordinated over time.