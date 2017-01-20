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

If you haven't read my other [debugging](/tags/#debugging) post about visualizing software, I advice you to start there. You should now have a proper understanding of what software is and how to visualize it. Also the term _bugs_ is completly clear.

Developers spend up to 50% of their time debugging according to [research by Cambridge university](http://download.microsoft.com/documents/rus/visualstudio/03_CambridgeUniversity_study-time_and_cost_saved_using_RDBs-January_20....pdf).
But education typically lacks training on the subject [(McCauley 2008).](http://faculty.salisbury.edu/~xswang/research/papers/debugging/computerscienceeducation/contentserver1.pdf)

In this blogpost I want to give you a usefull collection of abstract ways to think about debugging.
If you write code for a living you probably recognise most (or all) of the techniques, but as starter this can help you structuring your debugging efforts.

> __Disclaimer__
> * There is no golden gun/silver bullet which solves all problems!
> * This post is about strategies, not techniques (_printf's, gdb, etc..._)
> * This post uses a high amount of over-simplification, if you've studied CS, don't shoot me.
> * This post aims at people starting as a developer

# Understand, Find, Fix

We now have the groundwork layed out and can start to think about how to debug.
A lot of sources and books about debugging and problem solving all identify a process somewhere between 4-9 steps/phases, see list below:

| Steps | Title |
|:-----:|:------|
| 4     | [Debugging: An analysis of bug-location strategies - Katz & Anderson](http://act-r.psy.cmu.edu/wordpress/wp-content/uploads/2012/12/216KatzAndersonHCI88.pdf) |
|       | [GROW model](https://en.wikipedia.org/wiki/GROW_model) |
|       | [How to solve it - George Polya](https://en.wikipedia.org/wiki/How_to_Solve_It)
| 5     | [General Debugging framework - Araki et al](https://pdfs.semanticscholar.org/0c39/6691adf89d048fa7475795d738b3e06caf0b.pdf) |
|       | [Expertise in debugging computer programs: A process analysis - Vessey](https://www.researchgate.net/profile/Iris_Vessey2/publication/223474955_Expertise_in_debugging_computer_programs_A_process_analysis/links/5552c4dc08aeaaff3bf0011c.pdf)
| 6     | [ Troubleshooting overview - Microsoft](https://technet.microsoft.com/en-us/library/bb457121.aspx) |
| 7     | [Debugging: 7 steps to fix an error - Making Good; Software](http://www.makinggoodsoftware.com/2009/06/14/7-steps-to-fix-an-error/) |
| 8     | [A3 problem solving](https://en.wikipedia.org/wiki/A3_problem_solving) |
| 9     | [Debugging - David Agans](http://debuggingrules.com/)                  |

When we take a look at all these apporaches they consist of 3 main phases:

  * Understand
  * Find
  * Fix

Debugging is like removing a needle from a haystack.
First you try to __understand__ what the needle and the haystack are.
Once you know what needle you're looking for, you try to __find__ the needle in the haystack.
Finally you try to remove the needle from the haystack and __fix__ the problem.

To make structured debugging a clear and instead of rambling about abstract concepts,
I want to show the process using a buggy `find_max` implementation.
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
You should start with collecting your first clues and checking the obvious.
Your goal of the understanding phase is to have a clear problem description and a consistent reproducement procedure. You should be able to draw a basic diagram as described in earlier #debugging posts that describes the _Given_, _When_ and _Then_. To reach this goal we will start with checking the obvious.

### Checking the obvious
> On a cold winter morning, you walk to your car to go to work.
> You turn the key of the ignition, but all you hear is the motor being cranked, but not starting.
> What is the first thing you do? you check the fuel-level.

Always start with checking the most obvious possible defects.
Don't take too long checking the obvious, and try to timebox it within 10 minutes.
Is the program you're running the one you expect? Are you looking at the haystack you expect?
You can't imagine the amount of problems that are actually solved by noticing you're running the wrong program.
The big benefit of starting with this (apart from solving it directly :), you’re collecting clues to understand failure & system.
See each clue as a constraint, describing the defect/infection/failure involved.
If we think about the visualisation programs we are asserting our assumptions about the first state also known as the _Given_.

> Add diagram.

Checking the obvious is answering the following two questions:
* Is it plugged in?
* Am I doing what I expect I'm doing?

In our example we should assert that we are running the `find_max` script we are expecting.
Double check that the maximum of 3, 5, 1 should be 5.

### Describe the problem
> Your mother yells up the stairs: _"Something is wrong with the computer?!.
> It doesn't do what I expect."_ What is the first question that comes to mind?

We've checked the obvious and found nothing.
Let's move on to the next step.
Before you ever can declare anything fixed, you should be able to reproduce the failure.
But if you want to reproduce _it_, you must know what _it_ is, describe the failure.

You should be able to answer the following 2 questions:
* What did I expect or want to happen?
* What did happen?

In our example we expect `find_max` to return 5, but actually it returns 3.

If we think about the visualisation programs we are describing the final state that we expect and that it actually is, also known as the _Then_.

> Add diagram.

### Reproduce the failure
> Someday you get a message telling you; _"Thanks for your program, but when I open
> a file, the program crashes"_. As a developer you will probably ask, what file did you try to
> open, and describe me the buttons you've pressed.

A reproduction procedure describes the steps needed to show the failure.
Trying to reproduce, gives you more clues/constraints.
Write down (better yet automate) a procedure how to reproduce the failure.
The procedure should be fool-proof, you should absolutly no room for ambiguity.
And what is better suited for any fool than running a simple script?

* What steps do I have to do to reproduce the failure?

Reproducing the failure in our example is easy, just run the script!

If we think about the visualisation programs we are asserting our assumptions about the steps to go from the first state to the final state.
In the _Given_, _When_, _then_ idiom this would represent the _When_, since we already know the _Given_ and _Then_ from our previous steps.

> Add diagram.

### Start keeping notes
> This part seems to be the hardest part for developers.
> Typically a debugging session start out with small checks and as time goes by you keep tumbling down the rabbithole.
> After a while you realize you are 2 days further in time, have tried a lot of things, but the failure is still there.

After checking the obvious (<10 min), keep notes during all steps!
Keeping notes makes it easy to stop and continue the next day.
It also helps in communicating your findings and effort with your colleagues, your team lead, or even to your future self.
Sometimes re-reading some notes of the previous day, provides you with the jolt of insight you need to solve your problem.

You should choose your own format & medium, but a simple hand drawn table can suffice:

> insert example

## Find
We now know (and understand) the first and final state of our software. We also know what we must do to get it from the first state into the faulty final state (reproduction).
Remember the dependency chain introduced in my previous post?
We actually defined the beginning of this chain, being the start or input state.
And we defined the failure being the last link in the dependency chain.
We should now start to find the steps linking these states which makes the output state a failure.
Finding the defect in the cause-effect chain is the main part of debugging.
Some research showed it can take up to 95% of the time ([Myers. - The Art of Software Testing.](http://barbie.uta.edu/~mehra/Book1_The%20Art%20of%20Software%20Testing.pdf))

To make our _haystack_ (i.e. chain) as small as possible we should reduce it through simplification and isolation.
To do this, we will use our main tool, and no it is ot the debugger.
It is __the scientific method__.

### Scientific Method
The scientific method is the heart of every piece of science in the world, also the _science of debugging_.
By following the steps consistently you have a good structured approach for locating the defect.

<figure align="center">
<img src="/images/structured-debugging/ScientificMethod.svg" alt="image">
<figcaption>The scientific method</figcaption>
</figure>

So what are these steps?

1. Make observation(s)
2. Create a hypothesis
3. Predict outcomes of tests / inspections (must be falsifiable)
4. test predictions (try to disprove your hypothesis)
5. adjust/extend or create new hypothesis.
6. repeat from step 1. Ad nauseam

When following these steps, you should do one thing at a time, if the hypothesis is refuted (still useful info), but undo any changes.

So we now have our main tool ready for action, let's start with reducing our haystack using __Simplification__.

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
Isolation is a very powerful technique in debugging.
Isolation means minimizing the difference between the actual situation and the expected/wanted situation.
The critical difference causing the failure is by definition always in the red circle!

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
For pinpointing you should apply one of these three main strategies:

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
