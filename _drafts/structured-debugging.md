---
layout: post
title: "Structured debugging"
modified:
categories: blog
excerpt: "Developers spend up to 50% of their time debugging, but don't know ...."
tags: [debugging]
comments: true
share: true
image:
  feature: bug.jpg
date: 2016-09-07T22:33:01-04:00
---


# Structured debugging #

Developers spend up to 50% of their time debugging (according to Britton 2013 – Reversible debugging).
Education (typically) lacks training on the subject (McCauley 2008 – Debugging: a review of the literature from an educational perspective)

In this blogpost I want to give you a usefull collection of abstract ways to think about debugging.
If you write code for a living you probably recognise most or all of the techniques, but as starter this can help you structure your debugging efforts.

> Disclaimer
> * There is no golden gun/silver bullet which solves all problems!
> * This post is about strategies, not techniques (printf’s, gdb, etc...)
> * High amount of over-simplification is used, if you studied CS, don’t shoot me.

## What is software? #

In order to understand debugging (strategies), it is essential to accept that:
* Software isn’t magic.
* (In SW) things happen for a ‘deterministic’ reason, because a program consists of:
    * __finite set of inputs__: Such as input by users, files, hardware effects, system load, etc..
    * __finite set of transformations__: Methods, functions, calls, etc.. 
    * __finite set of outputs__: files, print to screen/console, action on port, etc...

In SW you always start with an initial state, then some stuff happens to transforms all the inputs and after a while your system is in a new state.
And when the final state does not match your expectation, then you start finding out why.

    Input state --> Transformation --> Output State ≠ Goal State

## What are bugs?

First actual case of bug in computer was in 1945, when in the Mark II computer at Harvard there was a moth in a relay.
It can still be found. 
The term bug was already known to Edison in 1878:

> “'Bugs' -- as such little faults and difficulties are called -- 
> show themselves and months of intense watching, study and labor are 
> requisite before commercial success or failure is certainly reached.” 

Problem with the term bugs is taht it implies something magic happened. 
The term bug is very ambiguous, it can either mean:
* The typo that caused the wrong behavior
* Wrong behavior in some part of the SW that was caused by the typo
* The wrong behavior as seen by the user

I prefer the terms Andreas Zeller uses, in his book "Why Programs Fail":

* 'Failure': bad/unexpected behavior as seen externally (by the user) and actually the last effect on the cause/effect chain.
* 'Infection': bad/unexpected caused by other bad/unexpected behavior. Each cause in the cause effect chain except first cause.
* 'Defect': bad/unexpected behavior causing problem. First cause in the cause effect chain.

# Introducing: "THE process"

A lot of sources and books have all identify a three to five phase process, which basically all boil down to:

* Understand
* Find
* Fix

Finding defects is search-problem;
* Understand phase defines the search domain & problem statement (needle & haystack)
* Location phase minimizes search domain, then does exhaustive search.

## Understand
The first step of the process is to understand the problem and the system. 
It actually starts with collect your first clues and checking the obvious.
After that you should describe the problem, and reproduce the failure consistently.

### Checking the obvious
When your car doesn’t start, you check the fuel-level.
When your vacuum cleaner doesn’t work, you check if it is plugged in.

Always start with checking the most obvious possible defects.
Don't take too long checking the obvious, and try to timebox it within 10 minutes.
Is the program you’re running the one you expect? 
You can't imagine how many problems are actually solved by noticing you're running the wrong program.
The big benefit of starting with this (apart from solving it direrctly :), you’re collecting clues to understand failure & system.
Each clues can be seen as a constraint, defining the defect/infection/failure involved. 

### Describe the problem
Before you ever can declare anything fixed, you should be able to reproduce the failure.
But if you want to reproduce “it”, you must know what “it” is, describe the failure.

For yourself answer the following 2 questions:
* What did I expect or want to happen?
* What did happen?

### Reproduce the failure
A reproduction procedure describes the steps to show the failure.
Trying to reproduce, gives you more clues/constraints.
Write down (better yet automate) a procedure how to reproduce the failure. 
Procedure should be fool-proof (no ambiguity), and what it better suited for any fool than just running a simple script.

### Start keeping notes
After checking the obvious (<10 min), keep notes during all steps!
Keeping notes makes it easy to stop and continue next day. 
It also helps in communicating your findings / effort with you colleagues, your team lead aor even to your future self.
Sometimes re-reading some notes of the previous day, provides you with the jolt of insight you need to solve it.

You should choose your own format & medium, but remember that a simple hand drawn table can suffice:

> insert example

## Find
We now know (and understand) our goal and domain. 
The goal is our needle and the domain our haystack.
Our goal is solving the defect that is causing the failure as described.
Finding the defect at the beginning of the cause-effect chain is the main part of debugging.
Some research showed it can take up to 95% of the time (G. J. Myers. The Art of Software Testing. John Wiley & Sons, Inc., New York, 1979)

To make our haystack as small as possible we should reduce it through simplification and isolation.
Our main tool to do this is the scientific method.

### Scientific Method
The scientific method is the heart of every piece of science in the world, also the science of debugging.

* Make observation(s)
* Create a hypothesis
* Predict outcomes of tests / inspections (must be falsifiable)
* test predictions (try to disprove your hypothesis)
* adjust/extend or create new hypothesis.
* repeat from step 1. Ad nauseam

When following these steps, you should do 1 thing at a time, if hypothesis is refuted (still useful info), but undo any changes. 

### Simplification
First step in locating is to make the reproduction procedure as short as possible to reduce the search domain (haystack).
We decompose our system and/or our procedure in chunks (functions, libraries, steps, etc..) in smaller haystacks
and remove unnecessary steps, but keep reproducing the same (or similar) failure. (Breadth-first search).

After doing this we should update our notes describing our smaller haystack.

### Isolation
Let's start with an example we all encounter.

> You arrive at your make-or-break presentation and you try to connect your laptop to the beamer.
> What! No image?! Your presentation is almost about to start so you ask your colleague
> if his laptop works. You connect the cable to his laptop, but still no sigar.
> You start mumbling: 'Stupid facility management, it is 2016 and you still can't
> even get a proper presentation on the screen'. Maybe changing the cable works. 
> You run out to your desk get a new HDMI cable you have laying around, after running
> back you connect it to the beamer and your laptop and finally! 
> Your presentation appears on the screen. That was close.
>
> You survive your presentation and after the presentation your
> colleague asks if he can try if his laptop works, so he knows his laptop will work
> the next time with the new cable. Yes it does, so the problem is definitvly in the
> old cable.

So what just happened? You __isolated the problem__ to the difference in cables.
Isolation is very powerful technique in debugging.
Isolation means minimizing the difference between the actual situation and the expected/wanted situation. 
 Defect is by definition always in red circle!

Main isolation technique is either:
* move items from red-only to overlapping area (adding items also to good run).
* OR removing them from red-only area.

Isolation can be applied to:
* The inputs causing the failure (user input, files, settings, OS, etc...)
* The transformations involved in the failure (code changes, execution path, order)
* The outputs showing the failure (logs, traces, files, values)

### After isolation

3 main strategies:

Binary search / wolf-fencing: 
* split program/procedure in half, see if system (cause-effect chain) is infected, investigate infected half.

Topographic
* Given certain bad-run only inputs, walkthrough the code seeing what path is followed. Follow cause-effect chain from inputs.

Backward reasoning
* Given bad-run only outputs, walk backwards in time and follow the infection up to the defect. Follow cause-effect chain backwards from outputs.

## Fix

We found the defect!
Apply a fix
Keep being scientific and test your fix given your reproduction scenario.
Usually defects cluster, so also test other values and look around.

# So, what is structured debugging?

Structured debugging is:
* Choosing a strategy and keep applying it (scientifically)
* Keep a notes during the process.
* When a strategy fails, or you’re blocked --> pick another strategy and stick to that.

Don’t get stuck by ‘THE process’, e.g.:
* If reproducing is hard, try locating it first, can give you clues/constraints about how to reproduce.
* If isolation looks promising, skip simplifying.
* Use binary-split for simplifying. 

# Furter reading

