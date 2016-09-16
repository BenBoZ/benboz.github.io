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
If you have a clear mental picture of how a program executes, then it is much easier to find the root of your problem.

I looked around in what helped me in visualizing program execution and I wanted something that I could
apply to any language. Your first reaction is probably, _UML sequence diagram!_
Although a sequence diagram shows me the sequence of steps that is executed, it misses information about the values.
You cannot see the internal state of the program at a certain point, you have to infer this.

Well what about flame graphs they look cool (sorry for that small pun).
I must admit they look nice, but still they don't show me the values. 

So let's go back to the drawing board.

## What is software? #

In software (actually in hardware to) you always start with an initial state.
Then some stuff happens to transforms all the inputs and after a while your system is in a new state.

So we should accept that:

> Software isn't magic.

In software things happen for a 'deterministic' reason, because a program consists of:
* __finite set of inputs__: Such as input by users, files, hardware effects, system load, etc..
* __finite set of transformations__: Methods, functions, calls, etc...
* __finite set of outputs__: files, print to screen/console, action on port, etc...

So lets put this information to work for us and start constructing our visualisation.
If we draw a set of blocks in a row, each representing an input element, we can show the input state.
Each block can represent any kind of input, such as files, enviroment variables, port, memory locations, etc.

> image showing row of inputs

Now we have some inputs, having some initial state is nice, but worth nothing if we don't do anything with it.
Lets add a transformation which changes them into a next state.

> image showing transformation + outputs

So, now we are done, aren't we? We have the _inputs_, _transformations_ and _outputs_ you mentioned.
Nope, off course not, although this graphic can represent simple linear programs, how about _flow_?
What if there are statements that influence the flow of a program?
If you have an if statement the outcome of the if statement should somehow be represented.
What if we just add 'line_number' as input.
It is kind of the program counter keeping track of where we are in the program.

> image with line_number

So, now we are really done. Aren't we? Nope, we can now represent any function or method.
But off couse any proper program has more than just the main function or module.
So we should add a way to represent this.
Lets add a dotted line to represent switching a context frame.
Each frame represents a different scope, so local variables are not visible anymore when you cross this.

> image which goes in/out function

Now for the final step. If you're really serious about programming, you don't stop at a single thread or program.
You separate the processing over multiple threads or processes.
To represent this we can just create a diagram next to the other.

> image with multithreading

# Dependency chain

So we now have a way of representing the state of a program during each step.
But what if we want to know what influences what?
If we want to get any usefull information out of this we should somehow visualize parts that influence eachother.
Lets link up the input values of a transformation to the output values of a tansformation using an arrow.

> example with single arrow

If we now apply this across the entire figure, we get a _chain of dependencies_.
This is appropriatly called a __dependency chain__.
A dependency chain is great for visualizing what your software does and how each step interacts.

# Creating these diagrams

OK, but I'm not going to draw these diagrams by hand.
Off course not! I created a little python script that can generate these images from JSON files with trace information.

> But then I still have to create the JSON file?

Yes, but I also created a python decorator that stores all tracing data into JSON.
So you can generate these diagrams automagically for python code.

If anyone wishes to write other tracers (gdb would be great :) ), then please do!

# Furter reading

