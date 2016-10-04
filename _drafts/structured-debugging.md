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

If you haven't read my other [debugging](/tags/#debugging) post about visualizing software, I advice you to start there.

Developers spend up to 50% of their time debugging according to [research by Cambridge university](http://download.microsoft.com/documents/rus/visualstudio/03_CambridgeUniversity_study-time_and_cost_saved_using_RDBs-January_20....pdf).
But education typically lacks training on the subject [(McCauley 2008).](http://faculty.salisbury.edu/~xswang/research/papers/debugging/computerscienceeducation/contentserver1.pdf)

In this blogpost I want to give you a usefull collection of abstract ways to think about debugging.
If you write code for a living you probably recognise most (or all) of the techniques, but as starter this can help you structure your debugging efforts.

> __Disclaimer__
> * There is no golden gun/silver bullet which solves all problems!
> * This post is about strategies, not techniques (printf's, gdb, etc...)
> * High amount of over-simplification is used, if you've studied CS, don't shoot me.

## What are bugs?

Let's start with defining what bugs are. The term _Bug_ is actually a very old.
It was already known to Edison in 1878:

> __Bugs__ -- as such little faults and difficulties are called --
> show themselves and months of intense watching, study and labor are
> requisite before commercial success or failure is certainly reached.

The first actual case of a bug in a computer was in 1945, when in the Mark II computer at Harvard a moth crept in a relay.
This actual bug can still be seen in the Smithsonian National Museum of American History.

<figure align="center">
<a title="By Courtesy of the Naval Surface Warfare Center, Dahlgren, VA., 1988. [Public domain], via Wikimedia Commons" href="https://commons.wikimedia.org/wiki/File%3AH96566k.jpg"><img width="512" alt="H96566k" src="https://upload.wikimedia.org/wikipedia/commons/8/8a/H96566k.jpg"/></a>
</figure>

As a developer it is important to know that although _bug_ is the most common used word in the industry, it bugs some people to call bugs bugs.
One of the most influential persons of the Computer Science field, Edgar Dijkstra, said the following:

> We could, for instance, begin with cleaning up our language by no longer calling a bug a bug but by calling it an error.
> It is much more honest because it squarely puts the blame where it belongs, viz. with the programmer who made the error.
>
> ~_Dijkstra_ - [_"On the cruelty of really teaching computer science"_](https://en.wikipedia.org/wiki/On_the_Cruelty_of_Really_Teaching_Computer_Science)

As Dijkstra points out, the problem with the term bugs is that it implies something magic happened.
It implies that something crawled in and messed up the program.
Although the moth proves this is potentially possible, you or your fellow developer is the one to blame.

There is another reason why it is better not to use the term bugs.
Which I find evne more important, namely the ambiguity of the term _bug_.
It can either mean:
* The typo that caused the wrong behavior
* Wrong behavior in some part of the software that was caused by the typo
* The wrong behavior as seen by the user

I prefer the terms Andreas Zeller uses, in his great book _Why Programs Fail_:

* __Failure__: bad/unexpected behavior as seen externally (by the user) and actually the last effect on the cause/effect chain.
* __Infection__: bad/unexpected caused by other bad/unexpected behavior. Each cause in the cause effect chain except first cause.
* __Defect__: bad/unexpected behavior causing problem. First cause in the cause effect chain.

# Understand, Find, Fix

So we now have the groundwork layed out.
We can now start to think about how to debug.
A lot of sources and books about debugging all identify a process somewhere between 3-7 steps, which basically all boil down to:

<figure align="center">
<img src="/images/structured-debugging/UnderstandFindFix.svg" alt="image">
<figcaption>Basic process boils down to: Understand, Find and Fix</figcaption>
</figure>

Finding defects is a search-problem, first you try to __understand__ what the needle and the haystack are.
Once you know what needle you're looking for, you try to __find__ the needle in the haystack.
Finally you try to remove the needle from the haystack and __fix__ the problem.

In order to make structured debugging a little bit clear, instead of rambling about abstract concepts,
I want to show the process using a buggy find_max implementation.
As developer you will start screaming, _what kind of idiot created this example?!_ 
So look for the bug in your typical quick way and, after that continue reading.
Then we have that over with.

{% highlight python linenos %}

def find_max(num1, num2, num3):
   ''' Returns the highest of the three numbers '''
   max_num = 0

   if (num1 > num2) and (num1 > num3):
      max_num = num1
   if (num2 > num1) and (num2 > num3):
      max_num = num1
   if (num3 > num1) and (num3 > num2):
      max_num = num3

   return max_num

my_max = find_max(3, 5, 1)
{% endhighlight %}

## Understand

The first step of the process is to understand the problem (_needle_) and the system (_haystack_).
You should start with collectng your first clues and checking the obvious.
After that you should describe the problem, and reproduce the failure consistently.

### Checking the obvious
> On a cold winter morning, you walk to your car to got to work.
> You turn the key of the ignition, but all you hear is the motor being cranked, but not starting.
> What is the first thing you do? you check the fuel-level.

Always start with checking the most obvious possible defects.
Don't take too long checking the obvious, and try to timebox it within 10 minutes.
Is the program you're running the one you expect? Are you looking at the haystack you expect?
You can't imagine how many problems are actually solved by noticing you're running the wrong program.
The big benefit of starting with this (apart from solving it directly :), you’re collecting clues to understand failure & system.
Each clue can be seen as a constraint, describing the defect/infection/failure involved.

Checking the obvious is answering the following two questions:
* Is it plugged in?
* Am I doing what I expect I'm doing?

In our example we should know that we are running the find_max script we are expecting.
Double check that the maximum of 3, 5, 1 should be 5.

### Describe the problem
> Your mother calls: "Child, something is wrong with the computer".
> It doesn't do what I expect. What are the first questions that come to mind?

We checked the obvious and found nothing. 
Let's move on to the next step.
Before you ever can declare anything fixed, you should be able to reproduce the failure.
But if you want to reproduce _it_, you must know what _it_ is, describe the failure.

You should be able to answer the following 2 questions:
* What did I expect or want to happen?
* What did happen?

In our example we expect 5 to be returned, but actually 3 is returned.

### Reproduce the failure
> Someday you get a message telling you; "Thanks for your program, but when I open
> a file, the program crashes". As a developer you will probably ask, what file did you try to
> open, and describe me the buttons you've pressed.

A reproduction procedure describes the steps to show the failure.
Trying to reproduce, gives you more clues/constraints.
Write down (better yet automate) a procedure how to reproduce the failure.
Think of the _Given_, _When_, _then_ contruction.
Procedure should be fool-proof (no ambiguity), and what is better suited for any fool than just running a simple script.

* What steps do I have to do to reproduce the failure?

Reproducing the failure in our example is easy, just run the script!

### Start keeping notes
> This part seems to be the hardest part for developers. 
> Typically a debugging session start out with small checks and as time goes by you keep tumbling down the rabbithole.
> After a while you realize you are 2 days further in time, have tried a lot of things, but the failure is still there.

After checking the obvious (<10 min), keep notes during all steps!
Keeping notes makes it easy to stop and continue the next day. 
It also helps in communicating your findings / effort with your colleagues, your team lead, or even to your future self.
Sometimes re-reading some notes of the previous day, provides you with the jolt of insight you need to solve it.

You should choose your own format & medium, but a simple hand drawn table can suffice:

> insert example

## Find
We now know (and understand) our goal and domain.
The goal is our needle and the domain our haystack.
Remember the dependency chain introduced in my previous post? 
We actually defined the beginning of this chain, being the start or input state.
And we defined the failure being the last link in the dependency chain.
We should now start to find the step in between these states which makes the output state a failure.
Finding the defect in the cause-effect chain is the main part of debugging.
Some research showed it can take up to 95% of the time ([Myers. - The Art of Software Testing.](http://barbie.uta.edu/~mehra/Book1_The%20Art%20of%20Software%20Testing.pdf))

To make our _haystack_ (i.e. chain) as small as possible we should reduce it through simplification and isolation.
To do this, we will use our main tool, and no it is ot the debugger.
It is _the scientific method_.

### Scientific Method
The scientific method is the heart of every piece of science in the world, also the science of debugging.
By following a few steps consistently you have a good structured approach for locating the defect.

<figure align="center">
<img src="/images/structured-debugging/ScientificMethod.svg" alt="image">
<figcaption>Basic process boils down to: Understand, Find and Fix</figcaption>
</figure>

So what are these steps?

1. Make observation(s)
2. Create a hypothesis
3. Predict outcomes of tests / inspections (must be falsifiable)
4. test predictions (try to disprove your hypothesis)
5. adjust/extend or create new hypothesis.
6. repeat from step 1. Ad nauseam

When following these steps, you should do 1 thing at a time, if hypothesis is refuted (still useful info), but undo any changes. 

So we now have out main tool ready for action, let's start with reducing our haystack using Simplification.

### Simplification
> Add example

First step in locating is to make the reproduction procedure as short as possible to reduce the search domain (haystack).
We decompose our system and/or our procedure in chunks (functions, libraries, steps, etc..) in smaller haystacks
and remove unnecessary steps, but keep reproducing the same (or similar) failure. (Breadth-first search).

After doing this we should update our notes describing our smaller haystack.

* What steps are not neccesarry to reproduce the failure?

We could prune our example by .....extend.....

### Isolation
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

So what just happened? You've __isolated the problem__ to the difference in cables.
Isolation is very powerful technique in debugging.
Isolation means minimizing the difference between the actual situation and the expected/wanted situation. 
 Defect is by definition always in red circle!

<figure class="half">
<img src="/images/structured-debugging/LaptopVennDiagram.svg" alt="image">
<img src="/images/structured-debugging/IsolationVennDiagram.svg" alt="image">
<figcaption>Isolating items that happen only in bad situation</figcaption>
</figure>

Main isolation technique is either:
* move items from red-only to overlapping area (adding items also to good run).
* OR removing them from red-only area.

Isolation can be applied to:
* The inputs causing the failure (user input, files, settings, OS, etc...)
* The transformations involved in the failure (code changes, execution path, order)
* The outputs showing the failure (logs, traces, files, values)


* What happens only in the bad/unwanted situation?

### After isolation

When you isolated the problem to the smallest possible part of your software, it is time for pinpointing the defect.
For pinpointing you should apply on of these three main strategies:

* __Binary search / wolf-fencing__: 
  * split program/procedure in half, see if system (cause-effect chain) is infected, investigate infected half.

* __(Forward) Topographic__
  * Given certain bad-run only inputs, walkthrough the code seeing what path is followed. Follow cause-effect chain from inputs.

* __Backward reasoning__
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

