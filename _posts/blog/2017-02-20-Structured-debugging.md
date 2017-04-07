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
date: 2017-02-21T07:00:01-04:00
---

If you haven't read my other [debugging](/tags/#debugging) post about visualizing software, I advice you to start there. You should now have a proper understanding of what software is and how to visualize it. Also the term _bugs_ is completely clear.

Developers spend up to 50% of their time debugging according to [research by Cambridge university](http://download.microsoft.com/documents/rus/visualstudio/03_CambridgeUniversity_study-time_and_cost_saved_using_RDBs-January_20....pdf).
But education typically lacks training on the subject [(McCauley 2008).](http://faculty.salisbury.edu/~xswang/research/papers/debugging/computerscienceeducation/contentserver1.pdf)

In this blog post I want to give you a useful collection of abstract ways to think about debugging.
If you write code for a living you probably recognize most (or all) of the techniques, but as starter this can help you structuring your debugging efforts.

> __Disclaimer__
> * There is no golden gun/silver bullet which solves all problems!
> * This post is about strategies, not techniques (_printf's, gdb, etc..._)
> * This post uses a high amount of over-simplification, if you've studied CS, don't shoot me.
> * This post aims at people starting as a developer

# Understand, Find, Fix

We now have the groundwork laid out and can start to think about how to debug.
A lot of sources and books about debugging and problem solving all identify a process somewhere between 4-9 steps, see list below:

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

When we take a look at all these approaches they consist of 3 main phases:

  * Understand
  * Find
  * Fix

Debugging is like removing a needle from a haystack.
First you try to __understand__ what the needle and the haystack are.
Once you know what needle you're looking for, you try to __find__ the needle in the haystack.
Finally you try to remove the needle from the haystack and __fix__ the problem.

The best learning is done by example, so we can reuse the example of [the previous post](/blog/What-are-bugs/).
It is a buggy `find_max` implementation.
As mentioned in that post, as developer you will probably start screaming, _what kind of idiot created this example?!_
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

print(find_max(3, 5, 1))

{% endhighlight %}

## Understand

The first step of the process is to understand the problem (_needle_) and the system (_haystack_).
You should start with collecting your first clues and checking the obvious.
Your goal of the understanding phase is to have a clear problem description and a consistent reproduction procedure. You should be able to draw a basic diagram as described in earlier [debugging posts](/tags/#debugging) that describes the _Given_, _When_ and _Then_. To reach this goal we will start with checking the obvious.

### Checking the obvious
> On a cold winter morning, you walk to your car to go to work.
> You turn the key of the ignition, but all you hear is the motor cranking, but not starting.
> What is the first thing you do? You check the fuel-level.

Always start with checking the most obvious possible defects.
Don't take too long checking the obvious, and try to time box it within 10 minutes.
Is the program you're running the one you expect? Are you looking at the haystack you expect?
You can't imagine the amount of problems that are actually solved by noticing you're running the wrong program.
The big benefit of starting with this (apart from solving it directly :)), you’re collecting clues to understand the failure & system.
See each clue as a constraint, describing the defect/infection/failure involved.
If we think about the visualization programs we are asserting our assumptions about the first state also known as the _Given_.

![Checking the obvious](/images/structured-debugging/checking-the-given.svg)

Checking the obvious is answering the following questions:
* Is it plugged in?
* Am I doing what I expect I'm doing?
* Are my assumptions about the first state correct?

In our example we should assert that we are running the `find_max` script we are expecting.
We should also double check that the maximum of `3`, `5`, `1` should be `5`.

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

If we think about the visualization programs we are describing the final state that we expect and that it actually is, also known as the _Then_.

![Describe the problem](/images/structured-debugging/describe-the-problem.png)

### Reproduce the failure
> Someday you get a message telling you; _"Thanks for your program, but when I open
> a file, the program crashes"_. As a developer you will probably ask, what file did you try to
> open, and describe me the buttons you've pressed.

A reproduction procedure describes the steps needed to show the failure.
Trying to reproduce, gives you more clues/constraints.
Write down (better yet automate) a procedure how to reproduce the failure.
The procedure should be fool-proof, you should absolutely no room for ambiguity.
And what is better suited for any fool than running a simple script?

* What steps do I have to do to reproduce the failure?

Reproducing the failure in our example is easy, just run the script!

If we think about the visualization programs we are asserting our assumptions about the steps to go from the first state to the final state.
In the _Given_, _When_, _then_ idiom this would represent the _When_, since we already know the _Given_ and _Then_ from our previous steps.

![Reproduce the failure](/images/structured-debugging/reproduce-the-failure.png)

We can now formulate it more formally with the information we have up until now:

> __Given__ the numbers `3`,`5` and `1`, __when__ calling `print(find_max())`, __then__ `3` is printed, __but expected__ `5` to be printed.

### Start keeping notes
> This part seems to be the hardest part for developers.
> Typically a debugging session start out with small checks and as time goes by you keep tumbling down the rabbit hole.
> After a while you realize you are 2 days further in time, have tried a lot of things, but the failure is still there.

After checking the obvious (<10 min), keep notes during all steps!
Keeping notes makes it easy to stop and continue the next day.
It also helps in communicating your findings and effort with your colleagues, your team lead, or even to your future self.
Sometimes re-reading some notes of the previous day, provides you with the jolt of insight you need to solve your problem.

You should choose your own format & medium, but a simple hand drawn 2-column table can suffice:

| What | Description |
|:-----:|:------|
| _Problem_ |  __Given__ the numbers `3`,`5` and `1`, __when__ calling `print(find_max())`, __then__ `3` is printed, __but expected__  `5` to be printed. |


## Find
We now know (and understand) the first and final state of our software. We also know what we must do to get it from the first state into the faulty final state (reproduction).
Remember the dependency chain introduced in [my previous post](/blog/What-are-bugs/)?
We actually defined the beginning of this chain, being the start or input state.
And we defined the failure being the last link in the dependency chain.
We should now start to find the steps linking these states which makes the output state a failure.
Finding the defect in the cause-effect chain is the main part of debugging.
Some research showed it can take up to 95% of the time ([Myers. - The Art of Software Testing.](http://barbie.uta.edu/~mehra/Book1_The%20Art%20of%20Software%20Testing.pdf))

To make our _haystack_ (i.e. chain) as small as possible we should reduce it through simplification and isolation.
To do this, we will use our main tool, and no it is __not__ the debugger.
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

When following these steps, you should do one thing at a time.
If the hypothesis is refuted remember it can still be useful info, but undo any changes.

So we now have our main tool ready for action, let's start with reducing our haystack using __Simplification__.

### Simplification
> When writing some unit tests you see that a test fails.
> In order to investigate the issue you run the entire test suite of your application, which takes 20 minutes.
> You try a fix and rerun the application, and retest again, waiting another 20 minutes.
> At this point you realize that only one unittest in your enormous test suite is failing so running all other tests is unnecessary.
> You disable the other tests and now you can test within a few seconds if your fix works.

First step in locating is to make the reproduction procedure as short as possible to reduce the search domain (haystack).
We decompose our system and/or our procedure in chunks (functions, libraries, steps, etc..) in smaller haystacks
and remove unnecessary steps, but keep reproducing the same (or similar) failure.
For you graph-lovers out there, you will recognize it as a Breadth-first search.

After doing this we should update our notes describing our smaller haystack.

You should be able to answer the question:

* What steps are not necessary to reproduce the failure?

If we expand the diagram of our example we can see all steps the software took to come up with the bad result.

| What           | Description |
|:--------------:|:------------|
| _Problem_      |  __Given__ the numbers `3`,`5` and `1`, __when__ calling `print(find_max())`, __then__ `3` is printed, __but expected__  `5` to be printed. |
| _Hypothesis-1_ | `print` is not needed for getting `3` instead of `5` |
| _Prediction-1_ | When we replace `print` statement with `assert`, `3` is returned instead of `5` |
| _Test-1_       | _Prediction-1_ is confirmed |

With respect to the original call the `print` call is unnecessary since with a debugger we can see the value of `max_num`

![Dependency chain](/images/structured-debugging/dependency-chain.png)

### Isolation
> You arrive at your make-or-break presentation and you try to connect your laptop to the beamer.
> What! No image?! Your presentation is almost about to start so you ask your colleague
> if his laptop works. You connect the cable to his laptop, but still no cigar.
> You start mumbling: _'Stupid facility management, it is 2016 and you still can't
> even get a proper presentation on the screen'_. Maybe changing the cable works.
> You run out to your desk get a new HDMI cable you have laying around, after running
> back you connect it to the beamer and your laptop and finally!
> Your presentation appears on the screen. That was close.
>
> You survive your presentation and after the presentation your
> colleague asks if he can try if his laptop works, so he knows his laptop will work
> the next time with the new cable. Yes it does, so the problem is definitely in the
> old cable.

So what just happened? You've __isolated the problem__ to the difference in cables.
Isolation is a very powerful technique in debugging.
Isolation means minimizing the difference between the actual situation and the expected/wanted situation.
The critical difference causing the failure is by definition always in the red circle!
You are applying the _law of parsimony_ a.k.a. ["_Occam's razor_"](https://en.wikipedia.org/wiki/Occam's_razor).
Meaning you are looking for the smallest difference to explain the defect.

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

Lets take our example and apply isolation to the inputs of `find_max`.
And let us look at [permutations](https://en.wikipedia.org/wiki/Permutation) of the inputs.
Needless to say we should use our _scientific method_ tool and create a hypothesis and a test to try to disprove our hypothesis.

| What           | Description |
|:--------------:|:------------|
| _Problem_      |  __Given__ the numbers `3`,`5` and `1`, __when__ calling `print(find_max())`, __then__ `3` is printed, __but expected__  `5` to be printed. |
| _Hypothesis-1_ | `print` is not needed for getting `3` instead of `5` |
| _Prediction-1_ | When we replace `print` statement with `assert`, `3` is returned instead of `5` |
| _Test-1_       | _Prediction-1_ is confirmed |
| _Hypothesis-2_ | The failure only occurs when there is a specific permutation of the inputs. |
| _Prediction-2_ | Take all permutations of `3`, `5` and `1` and feed all these values through our `find_max` implementation. Only certain permutations will occur in bad situations. |

Feeding these values to `find_max` indeed results in different outcomes.
If we take an isolation diagram an placing inputs resulting in correct outcomes
in the green circle and wrong outcomes in the red circle, we can make the
following image:

<figure align="center">
<img src="/images/structured-debugging/IsolationVennDiagramPermutations.svg" alt="image">
<figcaption>Isolating inputs of `find_max`</figcaption>
</figure>

It is now clear that somehow when `5` is in the middle the `find_max` function fails.
This confirms our hypothesis and we have now isolated the problem to inputs where the max is in the middle position.
We've answered the question: _What happens only in a bad/unwanted situation?_

In a future post I'm writing, I want to explain more concrete isolation techniques.

### After isolation

When you have isolated the problem to the smallest possible part of your software, it is time for pinpointing the defect.
Pinpointing the defect always involves following the dependency chain.

The literature shows three main strategies to be effective:

* __Binary search / wolf-fencing__:
  * Split the program/procedure in half, see if the system (dependency chain)
    is infected, if so investigate the half before that point, if not inspect
    the half after that point.

* __(Forward) Topographic__
  * Given a set of inputs and transformations resulting in a failure, follow
    the dependency chain from the start step-by-step, until you see the defect.

* __Backward reasoning__
  * Given a set of outputs that are faulty, walk backwards in time and follow
    the infections in the dependency chain up to the defect.

Choosing which strategy to use is something you will learn when doing.
The rate of success of each strategy differs per situation.
The most important part is that you stick with one of the three until you get stuck.
After that, switch to another strategy.
This way your debugging effort becomes structured.

We can take our example and apply __forward__ reasoning to it.
Again it is important to create a hypothesis, this might seem overly complex, but this will keep you focussed on what you are trying to accomplish.
Next to that, this is only a trivial example, used to show the process.

Isolation showed that only in bad situations `5` is in the middle.
If we use one of the inputs in the bad situation: `3`, `5`, `1` we can use that to walk our dependency chain starting from the first state.
We should make a hypothesis as to how this specific permutation would result in a failure.
Maybe the `find_max` function can not determine what number is the maximum.
We can add _Hypothesis-3_ and accompany it with _Prediction-3_:

| What           | Description |
|:--------------:|:------------|
| _Problem_      |  __Given__ the numbers `3`,`5` and `1`, __when__ calling `print(find_max())`, __then__ `3` is printed, __but expected__  `5` to be printed. |
| _Hypothesis-1_ | `print` is not needed for getting `3` instead of `5` |
| _Prediction-1_ | When we replace `print` statement with `assert`, `3` is returned instead of `5` |
| _Test-1_       | _Prediction-1_ is confirmed |
| _Hypothesis-2_ | The failure only occurs when there is a specific permutation of the inputs. |
| _Prediction-2_ | Take all permutations of `3`, `5` and `1` and feed all these values through our `find_max` implementation. Only certain permutations will occur in bad situations. |
| _Test-2_       | Only permutations `3, 5, 1` and `1, 3, 5` result in a failure.
| _Hypothesis-3_ | The correct maximum number is not determined. |
| _Prediction-3_ | Given the inputs `3`, `5`, and `1`, `num2` (`5`) is not seen as maximum |

![Dependency chain](/images/structured-debugging/dependency-chain.png)

If we follow the chain step-by-step trying to disprove our hypothesis, we can see the following:

* __Step 1.__ `max_num = 0` : This does not determine the maximum, ignore.
* __Step 2.__ `if num1 > num2 and num1 > num3` : This checks if `num1` is the maximum, it is correct since it steps over it and has as result `line_number = 7`.
* __Step 3.__ `elif num2 > num1 and num2 > num3` : This checks if `num2` is the maximum,
                                        it is correct since it evaluates to
                                        `True` and steps into the `elif` branch
                                        and has as result `line_number = 8`.

The last step contradicts our hypothesis and that means our hypothesis is _disproven_ and not valid.
With this information we can create a new hypothesis and test and continue our forward reasoning:

> __Hypothesis__: Given the inputs `3`, `5`, and `1`, the correct maximum number is determined, but the incorrect output value is assigned.
>
> __Test__: Follow dependency chain from where the maximum number is determined until the return value is calculated.

Let us continue where we left of on line 8.

* __Step 4.__ `max_num = num1`: This assigns `num1` to `max_num` which is wrong!

Eureka! We have proven our hypothesis and probably found the defect. To be sure
we should continue our forward reasoning to see if this value is also returned.

> __Hypothesis__: Given the incorrect assignment `max_num = num1`, `max_num` is returned from the function.
>
> __Test__: Follow dependency chain from where the maximum number is assigned until the value is returned.

This is a very trivial example and only one step.

* __Step 5.__ `return max_num`: The infected `max_num` value is returned infecting the rest of the program and resulting in the failure.

In our debugging log we now have the following tested hypotheses, giving a complete description of the inputs, defect, infection and failure.
Note that the hypothesis that was disproved is still useful since you put in effort to disprove it.
If somebody in the future would take your debugging log, he can see you have tested and disproved the hypothesis saving that person effort.

* The failure only occurs when there is a specific permutation of the inputs.
* ~~Given the inputs `3`, `5`, and `1`, the correct maximum number is not determined.~~
* Given the inputs `3`, `5`, and `1`, the correct maximum number is determined, but the incorrect output value is assigned.
* Given the incorrect assignment `max_num = num1`, `max_num` is returned from the function.

## Fix

After you pinpointed the defect. You can climb on your chair and shout: _"We found the defect!"_
If you have found the defect you can apply a fix.
Keep being scientific and test your fix using your reproduction scenario.
Usually defects cluster, so also test other values and look around.

> NOTE: Extend with examples of `find_max`

# So, what is structured debugging?

To wrap up and give you the must-knows after reading this post:

Structured debugging is:
* Choosing a strategy and keep applying it (scientifically)
* Keep a notes during the process.
* When a strategy fails, or you’re blocked --> pick another strategy and stick to that.

Don’t get stuck by ‘THE process’, e.g.:
* If reproducing is hard, try locating it first, can give you clues/constraints about how to reproduce.
* If isolation looks promising, skip simplifying.
* Use binary-split for simplifying.

# Further reading
