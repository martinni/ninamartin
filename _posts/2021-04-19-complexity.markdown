---
layout: post
title:  "The O was a θ all along"
date:   2021-04-19 21:34:00 +0100
categories: algorithms
---

Today is April 19th 2021. Exactly one year ago, I wrote my first post and brought this website to life. Happy birthday, blog! :birthday:

The idea of doing some writing and putting it online had been sitting at the back of my head for a while. Lockdown boredom was just the extra nudge I needed for the idea to materialise. I wasn't entirely sure what I would blog about and so far it turns out to have been mainly algorithms. But stay tuned, because other topics are coming!

But not today, because today I would like to talk about big O notation.

# What we think it means

Big O notation.

Every CS student and aspiring software engineer has learnt to understand it and use it to describe the efficiency of their code. It's an obligatory step of the tech interview, the question everyone expects after solving the coding test: "What is the runtime complexity of your implementation?". Just to make sure that we have the fundamentals covered, that we understand what we're doing.

So what does it mean when an algorithm runs in, say, O(n<sup>2</sup>)? Well easy, that means that the runtime of the algorithm grows quadratically with the input size... 

Well, not really.

# What it actually means

The big O notation defines an asymptotic upper bound, but it doesn't have to be a tight bound. The formal definition for big O notation is the following:

<pre>
O(g(n)) = {f(n) : there exist positive constants c and n<sub>0</sub> such that 0 ≤ f(n) ≤ cg(n) for all n ≥ n<sub>0</sub>}
</pre>

*Any* function that meets the above criteria qualifies for big O notation. So if your program takes input of size N and just prints "Hello World", it sure runs in O(1) but it also runs in O(n<sup>2</sup>) and in O(n<sup>n<sup>n</sup></sup>) for that matter, because all these functions meet the above definition for any positive value of *c* or *n<sub>0</sub>*.

Symmetrically you can define an asymptotic lower bound, typically written with big Ω notation:

<pre>
Ω(g(n)) = {f(n) : there exist positive constants c and n<sub>0</sub> such that 0 ≤ cg(n) ≤ f(n) for all n ≥ n<sub>0</sub>}
</pre>

Similarly to the reasoning above, you can say that an insertion sort runs in Ω(1) and you would be right, because there are many values of *c* and *n<sub>0</sub>* you can pick so that *g(n)=1* meets the above definition. The lower bound doesn't have to be tight.

So now if you combine these two concepts you get the big θ notation, defined as follow:

<pre>
θ(g(n)) = {f(n) : there exist positive constants c<sub>1</sub>, c<sub>2</sub>, and n<sub>0</sub> such that 0 ≤ c<sub>1</sub>g(n) ≤ f(n) ≤ c<sub>2</sub>g(n) for all n ≥ n<sub>0</sub>}
</pre>

Now you can't say that an insertion sort runs in θ(1) because there is no value of *c<sub>2</sub>* that fits the bill for the above definition. And to take our previous example, our hello world program also doesn't run in θ(n<sup>2</sup>) because there is no possible constant c<sub>1</sub> that would make the above inequality hold asymptotically. However if you choose the appropriate values for c<sub>1</sub> and c<sub>2</sub> (with c<sub>1</sub> ≤ c<sub>2</sub>), you can say indeed that the hello world program runs in θ(1) and that insertion sort runs in θ(n<sup>2</sup>).

So there we have it, the defintion of an asymptotic tight bound. The one that is actually useful to describe the efficiency of an algorithm. All those times when we said big-O what we actually meant was big theta! 

The software industry has abused the notation so much now that I'm not sure it's worth going back unless you talk about complexity in an academic context. But it's worth keeping in mind next time you hear somone claiming that a linear search runs in O(n<sup>n</sup>).