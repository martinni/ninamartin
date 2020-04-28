---
layout: post
title:  "Amortised analysis on a disk stored stack"
date:   2020-04-28 21:35:00 +0100
categories: algorithms
---

Following up my previous post about [amortised analysis]({% post_url 2020-04-19-amortised-analysis %}), I am going to apply it to a problem drawn from the B-Trees chapter in [Introduction to Algorithms](https://en.wikipedia.org/wiki/Introduction_to_Algorithms).

Here we are considering a stack too large to fit in memory, and the cost of maintaining suck a stack through various paging strategies. Amortised analysis will be used in the last section to evaluate this cost.

# A few words about memory and paging

We don't need to go into great details about memory management to solve this problem but let's just recap the fundamentals to make sure everyone is on the same (pun unintended) page.

Computer memory is divided between main memory (the RAM) and secondary memory (the hard drive).

Main memory consists of silicon chips made of lots of transistors, and by lots I mean *billions*. Each transistor is paired with a "memory cell" that contains a bit of information. The memory cell is a capacitator (you can think of it as a tiny battery) wich represent a 1 when filled with electrons and a 0 when empty. The transistor acts as a switch on the capacitator. Any read or write operation is based on electricity flowing through an electronic circuit. Therefore, RAM access is very fast, it's *nanoseconds* fast. RAM is also expensive. 

That is why computers also have a hard drive. For traditonal hard drives (SDD storage is different), this secondary storage is based on magnetic platters rotating around a spindle. These platters are divided into billions of tiny areas that contain either a 0 (unmagnetised cell) or a 1 (magnetised cell). Each platter is read and written with a magnetic head at the end of an arm. The arms can move their heads towards or away from the spindle. To read or write a cell, we need the arm's head to reach that cell which involves two mechanical movements: the platter rotation and the movement of the arm itself. To reach a particular cell, you might have to wait for a full platter rotation which is in the order of milliseconds. Because disk access is based on mechanical movements, disks are slow. But disks are cheap. This is why computers have much more capacity on their hard drive than their main memory. 

{:refdef: style="text-align: center;"}
![hard drive](/img/hard-drive.png)
<br/><br/>
*A typical hard drive*
{: refdef}

In order to optimise the time spent on mechanical movements, disks access not just one element at a time but several. Information on a disk is divided into equal size blocks of bits, called *pages*. Once the arm's head is positioned correctly and the platter has rotated to the desired page, reading and writing is entirely electronic and the disk can quickly read or write large amounts of data.

For the rest of this article, we will refer to "disk access" as the number of pages that need to be read and written to the disk.


# A stack entirely stored on disk

Back to our stack.

Let's first consider a stack entirely kept on disk. We maintain in memory a stack pointer, which contains the disk address of the top element on the stack. If the pointer has value *p*, the top element is the (*p* mod *m*)th word on page <code>⌊p/m⌋</code> of the disk, where *m* is the number of words per page.

To implement the PUSH operation, we increment the stack pointer, read the appropriate page into memory from disk, copy the element to be pushed on the appropriate word on the page, and write the page back to the disk. For the POP operation, we decrement the stack pointer, read in the appropriate page from disk, and return the top of the stack . Note that we don't need to "clean" the value previously held in memory as it will get overwritten anyway if we use that cell again, but for clarity I will blank the "popped" cell in the following illustrations.

{:refdef: style="text-align: center;"}
*push and pop on a disk stored stack*
<br/>
![stack push pop](/img/stack-push-pop.png)
{: refdef}


Any disk access to a page of *m* words has a CPU time of O(m), since *m* words must be read into memory.

A push operation includes 2 disk access and a pop operation has 1 disk access. Any combination of *n* stack operations has therefore a CPU time of O(n*m). 

# One page in memory

We now consider stack implementation in which we keep one page of the stack in memory. We can perform a stack operation only if the relevant disk page resides in memory. When necessary, we can write the page currently in memory to the disk and read in the new page from the disk to memory. If the relevant disk page is already in memory, then no disk access is required.

What is the CPU time for a series of *n* PUSH operations? One PUSH operation has a cost of either 1 disk access (when the current page is full and we need to load the next one) or no disk access at all when we can write into the currently loaded page. The disk access frequencyis of <code>n / m</code> where *m* is the number of words per page. A series of *n* PUSH incurs therefore a cost of <code>n / m</code> disk access costing O(m) each, which brings us to a worst case cost of O(n).

{:refdef: style="text-align: center;"}
*n push on a stack with the one-page implementation*
<br/>
![one page push series](/img/one-page-n-push.png)
{: refdef}

We can apply a symmetric reasoning for a series of *n* POP.

But what about a series of *n* PUSH **or** POP operations. You might be tempted to think, based on the above, that we will also end up with a cost of O(n). There is actually a worst case scenario where we are back to a cost of O(n*m). Consider a series of PUSH-POP-PUSH-POP-... on the boundary of a page. Essentially the two operations shown below repeated over and over. 

{:refdef: style="text-align: center;"}
<br/>
*push and pop at the page boundary*
![one page push series](/img/one-page-push-pop.png)
{: refdef}

Now it is clear that each operation introduces at least 1 disk access and therefore the sequence has a cost of O(n*m). That means that the amortised cost of a push or a pop is not constant.

# Two pages in memory

Let's now suppose that we can implement the stack by keeping two pages in memory. Can we find a way to achieve a constant amortised cost for any stack operation?

If you are not familiar with potential functions in amortised analysis, now is the time to read [this post]({% post_url 2020-04-19-amortised-analysis %}). For this problem, we are going to define Φ as <code>Φ(S) = |S.size - M|</code> where *M* is the last visited multiple of *m* (in other words, the index for the beginning of the last loaded page). Visually you can imagine the potential value to be the "distance" between the current pointer *p* and the index of the beginning of the last loaded page. When we are at a distance of *m* from that index, we are able to "pay the price" for a disk access.

First of all, let's check that this Φ works as a potential function. We have the requirement that <code>Φ(S<sub>n</sub>) >= Φ(S<sub>0</sub>)</code> for any *n*. For the empty stack S<sub>0</sub>, the most recently visited multiple of m is 0 so we have trivially <code>Φ(S<sub>0</sub>) = 0</code>. Since Φ is an absolute value, any value of Φ will be greater or equal to 0, which is all we need to prove the suitability of Φ as a potential function.

Which amortised cost does it give us for a PUSH or POP operation?

We have:
<pre>ĉ<sub>i</sub> = c<sub>i</sub> + Φ(S<sub>i</sub>) - Φ(S<sub>i-1</sub>)
   = 1 + |S.size<sub>i</sub> - M<sub>i</sub>| - |S.size<sub>i-1</sub> - M<sub>i-1</sub>|
</pre>

To compute the <code>|S.size<sub>i</sub> - M<sub>i</sub>| - |S.size<sub>i-1</sub> - M<sub>i-1</sub>|</code> term, we need to consider two cases, the case when we are pushing/popping inside one of the two currently loaded pages in memory and the case when we access the disk to load a new page.

When there is no disk access, the last loaded page is the same before and after the operation and we have therefore M<sub>i</sub> equal to M<sub>i-1</sub>. <code>S.size<sub>i</sub></code> has either incremented by one (PUSH) or decremented by one (POP), therefore the quantity <code>|S.size<sub>i</sub> - M<sub>i</sub>| - |S.size<sub>i-1</sub> - M<sub>i-1</sub>|</code> is equal to either 1 (PUSH) or -1 (POP). That gives us an amortised cost of <code>ĉ<sub>i</sub> = 1 + 1 = 2</code> for PUSH and <code>ĉ<sub>i</sub> = 1 - 1 = 0</code> for POP.

When there is a disk access, let's consider PUSH and POP separately.

For PUSH, we are adding an element at the beginning of a new page which needs to be loaded into memory. M<sub>i</sub> is the index for the beginning of the newly loaded page and we have <code>S.size<sub>i</sub> = M<sub>i</sub> + 1</code>, and therefore the quantity <code>|S.size<sub>i</sub> - M<sub>i</sub>|</code> is equal to 1. <code>S.size<sub>i-1</sub></code> is equal to <code>S.size<sub>i</sub>-1</code> and M<sub>i-1</sub> is the beginning of the last previously loaded page, which is saying <code>M<sub>i-1</sub> + m = M<sub>i</sub></code>. We have then <code>|S.size<sub>i-1</sub> - M<sub>i-1</sub>| = m</code>. That yields us an amortised cost of <code>ĉ<sub>i</sub> = 1 - m</code> which is constant.

{:refdef: style="text-align: center;"}
*push on a stack introducing a disk access with the two-pages implementation*
<br/>
![one page push series](/img/two-pages-push.png)
{: refdef}

For POP, we are removing an element at the beginning of one of the two currently loaded pages, and loading the page that comes before it. M<sub>i</sub> is the page we are popping from and M<sub>i-1</sub> is the newly loaded page, which means <code> M<sub>i</sub> + m = M<sub>i-1</sub></code>. <code>S.size<sub>i-1</sub></code> is the size of the stack before popping the last remaining element which means <code>S.size<sub>i-1</sub> = M<sub>i-1</sub> + 1</code>, which gives <code>|S.size<sub>i-1</sub> - M<sub>i-1</sub>| = 1</code>. After popping, <code>S.size<sub>i</sub> = S.size<sub>i-1</sub> - 1 = M<sub>i</sub> + m</code>, which gives <code>|S.size<sub>i</sub> - M<sub>i</sub>| = m</code>. That yields us an amortised cost of <code>ĉ<sub>i</sub> = 1 + m</code> which is constant.

{:refdef: style="text-align: center;"}
*pop on a stack introducing a disk access with the two-pages implementation*
<br/>
![one page push series](/img/two-pages-pop.png)
{: refdef}

That's it!

With the potential function of <code>Φ(S) = |S.size - M|</code> we have proved that the amortised cost of both PUSH and POP was constant with the two pages implementation. I think it's really cool that a small adjustment in the stack implementation allows us to change the worst case complexity in such a drastic way. This is an application of amortised analysis where you have to "visualise" the operations to make sense of the maths.