---
layout: post
title: "What are bugs?"
modified:
categories: blog
excerpt: "In this blogpost I want to provide you with a clear sense of how to think of bugs...."
tags: [debugging]
comments: true
share: true
image:
  feature: bug.jpg
date: 2016-09-07T22:33:01-04:00
---

If you haven't read my other [debugging](/tags/#debugging) post about visualizing software, I advice you to start there.

In this blogpost I want to provide you with a clear sense of how to think of bugs.
If your mental picture is clear, thinking about debugging becomes easier.

## What are bugs?

Let's start with defining what bugs are. The term _Bug_ is actually not from recent years.
The term was already known to Edison in 1878:

> __Bugs__ -- as such little faults and difficulties are called --
> show themselves and months of intense watching, study and labor are
> requisite before commercial success or failure is certainly reached.

The first actual case of a bug in a computer was in 1945, when in the Mark II computer at Harvard a moth crept in a relay.
This actual bug is for all to see at the Smithsonian National Museum of American History.

<figure align="center">
<a title="By Courtesy of the Naval Surface Warfare Center, Dahlgren, VA., 1988. [Public domain], via Wikimedia Commons" href="https://commons.wikimedia.org/wiki/File%3AH96566k.jpg"><img width="512" alt="H96566k" src="https://upload.wikimedia.org/wikipedia/commons/8/8a/H96566k.jpg"/></a>
</figure>

As a developer you should know that although _bug_ is the most common used word in the industry, it bugs some people to call bugs bugs.
One of the most influential persons of the Computer Science field, Edgar Dijkstra, said the following:

> We could, for instance, begin with cleaning up our language by no longer calling a bug a bug but by calling it an error.
> It is much more honest because it squarely puts the blame where it belongs, viz. with the programmer who made the error.
>
> ~_Dijkstra_ - [_"On the cruelty of really teaching computer science"_](https://en.wikipedia.org/wiki/On_the_Cruelty_of_Really_Teaching_Computer_Science)

As Dijkstra points out, the problem with the term bugs is that it implies something magic happened.
It implies that something crawled in and messed up the program.
Although the moth proves this is potentially possible, typically you or your fellow developer is the one to blame.

Another reason why not to use the term bugs is its ambiguity.
It can either mean:
* The typo that caused the wrong behavior
* Wrong behavior in some part of the software caused by the typo
* The wrong behavior as seen by the user

I prefer the terms Andreas Zeller uses, in his great book _Why Programs Fail_:

* __Failure__: bad/unexpected behavior as seen externally (by the user) and actually the last effect on the cause/effect chain.
* __Infection__: bad/unexpected caused by other bad/unexpected behavior. Each cause in the cause effect chain except first cause.
* __Defect__: bad/unexpected behavior causing problem. First cause in the cause effect chain.

Remember the diagrams from my blogpost ['Visualizing Software'](/blog/Visualizing-software/).
We can visualize these 3 items using those diagrams.
