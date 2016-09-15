---
layout: post
title: "Visualizing Software"
modified:
categories: blog
excerpt: "An effective way of visualizing software execution"
tags: [debugging]
comments: true
share: true
image:
  feature: bug.jpg
date: 2016-09-07T22:33:01-04:00
---

# Visualizing software

As first post in my debugging series I want to introduce a way of visualizing software execution.
If you have a clear mental picture of how a program executes, then it is much easier to find the root of your problem.

## What is software? #

But first it is essential to accept that:
* Software isn’t magic.
* (In SW) things happen for a ‘deterministic’ reason, because a program consists of:
    * __finite set of inputs__: Such as input by users, files, hardware effects, system load, etc..
    * __finite set of transformations__: Methods, functions, calls, etc.. 
    * __finite set of outputs__: files, print to screen/console, action on port, etc...

In SW you always start with an initial state, then some stuff happens to transforms all the inputs and after a while your system is in a new state.
And when the final state does not match your expectation, then you start finding out why.

    Input state --> Transformation --> Output State ≠ Goal State

If we depict inputs as a set of blocks with 1 element of the input state in a row,
we can show the input state. Note that this state can consist of files, enviroment variables,
memory locations, etc. So anything that you can consider being an input.

> image showing row of inputs

Now we have some inputs we want to do something with them, lets add a transformation which changes them
into a output state.

> image showing transformation + outputs

We now have a trivial example, but what if there are statements that influence the flow of a program.
We just add 'line_number' as input.

> image with line_number

Any proper program has more than just the main function or module. So we should add a way to represent this.
We can do this by adding a dotted line to represent switching a context frame.

> image which goes in/out function

If you're really serious about programming, you don't stop at a single thread or program.
You separate the processing over multiple threads or processes.
To represent this we can just create a diagram next to the other.

> image with multithreading

# Dependency chain

So we now have a way of representing the state of a program during each step.
If we want to get any usefull information out of this we should somehow visualize parts that influence eachother.
If a variable is used as input for a transformation to and written to an output state we can draw an arrow representing this

> example with single arrow

If we now apply this across the entire figure, we get a chain of dependencies.
This is appropriatly called a __dependency chain__.
A dependency chain is great for visualizing what your software does and how each step interacts.





# Furter reading

