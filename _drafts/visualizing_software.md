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
Well, although a sequence diagram shows me the sequence of steps that is executed, it misses information about the values.
You cannot see the internal state of the program at a certain point, you still have to read it from the source code, debugger or some kind of tracing.

Well what about flame graphs they look cool (sorry for that small pun).
I must admit they look nice, but still they don't show me the values.

So let's go back to the drawing board.
What do I want? I want a diagram that shows me the state of my program at all times during execution.
Also please note that I said _mental_ picture at the beginning.
The diagrams created here are not for real-world use.
Showing program state from beginning to end of any real program, would let the diagram explode to infinity.
I will use these diagrams just as a vehicle for you to think about software and debugging strategies in later posts.

## What is software? #

In software you always start with an initial state.
For instance when you start your computer and freeze time at that moment, you could see all kinds of information.
You could look at the values of all variables, settings, temperature, system load, free disk space and the contents of all files.

If you then unfreeze the time, and start a program, then some stuff happens.
Your computer transforms this initial input state into in a new state.
This stuff also known as software, is usually the program you've typed with great effort.
Or the statements you copy-pasted from StackOverflow.
In the new state, most inputs still exist but have a different value.

If you would accuratly record all the values at the initial state and would apply the exact same transformation, then the new state will be exactly the same.

So in software things happen for a 'deterministic' reason, because a program consists of:
* __finite set of inputs__: Such as input by users, files, hardware effects, system load, etc..
* __finite set of transformations__: Methods, functions, calls, etc...
* __finite set of outputs__: files, print to screen/console, action on port, etc...

So we should accept that:

> Software isn't magic.

# Creating the diagram

So lets put this information to work for us and start constructing our visualisation using these _inputs_, _transformations_ and _outputs_.
If we draw a set of blocks in a row, each representing an input element, we can show the initial state.
Each block can represent any kind of input, such as files, enviroment variables, port, memory locations, etc.
Say we have some python code in which num1,num2 and num3 are already set.

{% highlight python linenos %}
num1, num2 = 3, 5
{% endhighlight %}
![row of inputs representing initial state](/images/visualizing-software/inputs_only.svg)

Now we have some inputs, having some initial state is nice, but off course worth nothing if we don't do anything with it.
Lets add a transformation which changes this state into a next state.

{% highlight python linenos %}
num1, num2 = 3, 5
max_num = 0
{% endhighlight %}

![First transformation](/images/visualizing-software/first_transform.svg)

So, now we are done, aren't we? We have the _inputs_, _transformations_ and _outputs_ you mentioned.
Nope, not quite, although this graphic can represent simple linear programs.
How about statements that influence the _flow_ of a program?
If you have an if statement the outcome of the if statement should somehow be represented.
The state of the software will be different when it evaluates to TRUE or FALSE.
What if we just add 'line_number' as input.
Think of it as the program counter keeping track of where we are in the program.

{% highlight python linenos %}
num1, num2 = 3, 5
max_num = 0

if num1 > num2:
   max_num = num1
else
   max_num = num2
{% endhighlight %}

![First flow_condition](/images/visualizing-software/first_flow_condition.svg)

So, now we are really done. Aren't we? Nope, we can now represent any function or method.
But off couse any proper program has more than just the main function or module.
So we should add a way to represent this.
Lets add a dotted line to represent switching a context frame.
Each frame represents a different scope, so local variables are not visible anymore when you cross this.


{% highlight python linenos %}
def find_max(num1, num2):
   max_num = 0

   if num1 > num2:
      max_num = num1
   else
      max_num = num2

   return max_num

my_max = find_max(3, 5)
{% endhighlight %}
![first context switch](/images/visualizing-software/first_context_switch.svg)

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
These are very similar to program slices. 

# Creating these diagrams

OK, but I'm not going to draw these diagrams by hand. I have better things to do!
Off course not! I created a little python script that can generate these images from JSON files with trace information.

But then I still have to create the JSON file?
Yes, but I also created a python decorator that stores all tracing data into JSON.
So you can generate these diagrams automagically for python code.

If anyone wishes to write other tracers (gdb would be great :) ), then please do!
Head over to [the PrintState repo](https://www.github.com/spoorcc/PrintState) and check it out.



# Furter reading

