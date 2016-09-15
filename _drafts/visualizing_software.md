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
  feature: printstate.jpg
date: 2016-09-07T22:33:01-04:00
---

# Visualizing software

As first post in my debugging series I want to introduce a way of visualizing software execution.
I looked around in what helped me in visualizing program execution and I wanted something that I could
apply to any language.
If you have a clear mental picture of how a program executes, then it is much easier to find the root of your problem.

## What is software? #

But first it is essential to get some facts straight and accept that:
* Software isn’t magic.
* (In SW) things happen for a ‘deterministic’ reason, because a program consists of:
    * __finite set of inputs__: Such as input by users, files, hardware effects, system load, etc..
    * __finite set of transformations__: Methods, functions, calls, etc.. 
    * __finite set of outputs__: files, print to screen/console, action on port, etc...

In software (actually in hardware to) you always start with an initial state.
Then some stuff happens to transforms all the inputs and after a while your system is in a new state.

    Input state --> Transformation --> Output State

If we draw a set of blocks in a row, each representing an input element, we can show the input state.
Each block can represent any kind of input, such as files, enviroment variables, port, memory locations, etc.

> image showing row of inputs

Now we have some inputs we want to do something with them, lets add a transformation which changes them
into a next state.

> image showing transformation + outputs

This graphic can represent simple linear programs,
but what if there are statements that influence the flow of a program?
We just add 'line_number' as input. Just actually the program counter.

> image with line_number

We can now represent any function or method.
But off couse any proper program has more than just the main function or module.
So we should add a way to represent this.
Lets add a dotted line to represent switching a context frame.
Each frame represents a different scope, so local variables are not visible anymore when you leave this.

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

# Creating these diagrams

OK, but I'm going to draw these diagrams by hand. 
Off course not! I created a little python script that can generate these images from JSON files with trace information.

> But then I still have to create the JSON file?

Yes, but I also created a python decorator that stores all tracing data into JSON.
So you can generate these diagrams automagically for python code.

If anyone wishes to write other tracers (gdb would be great :) ), then please do!

# Furter reading

