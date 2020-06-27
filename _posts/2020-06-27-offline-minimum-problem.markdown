---
layout: post
title:  "The off-line mininum problem with disjoint sets"
date:   2020-06-27 16:43:00 +0100
categories: algorithms
---

Continuing my journey through [Algorithms](https://www.amazon.co.uk/Introduction-Algorithms-Thomas-H-Cormen/dp/0262033844/ref=sr_1_1?adgrpid=52755502465&dchild=1&gclid=EAIaIQobChMIp4-WpbSi6gIV6YBQBh33twKZEAAYASAAEgJcbvD_BwE&hvadid=259080196986&hvdev=c&hvlocphy=9045997&hvnetw=g&hvqmt=e&hvrand=17787531869004437962&hvtargid=kwd-300139095800&hydadcr=17612_1775484&keywords=introduction+to+algorithm&qid=1593275108&sr=8-1&tag=googhydr-21), I just finished the "Advanced data structures" section which walked me through the wonderful world of B-Trees, Fibonacci heaps, van Emde Boas Trees and Disjoint sets.

B-Trees are fairly simple to grasp (tl;dr: high branching factor, optimised disk access). Fibonacci heaps are a bit fiddly for certain operations but ok if you have a visual representation of the heap to follow what's going on. Van Emde Boas trees.. I don't really want to talk about them (my head still hurts from that chapter). And finally, we've got lovely disjoint sets, which are simple yet incredibly efficient thanks to a couple of well-chosen heuristics.

After reading the disjoint sets chapter, I decided to give a go to the first problem exposed in the book referred to as the "off-line" minimum.

If you are not familiar with disjoint sets, [here](https://jackmorris.xyz/2020/disjoint-sets-why-i-have-a-favourite-data-structure/) is an excellent introduction.

## Problem statement

 The **off-line minimum** problem asks us to maintain a dynamic set *T* of elements from the domain `{1,2,...,n}` under the operations INSERT and EXTRACT-MIN. We are given a sequence *S* of *n* INSERT and *m* EXTRACT-MIN calls, where each key in `{1,2,...,n}` is inserted exactly once. We wish to determine which key is returned by each EXTRACT-MIN call. Specifically, we wish to fill in an array `extracted[1..m]`, where for `i = 1,2,...`, *m* is the key returned by the *i*th EXTRACT-MIN call. The problem is "off-line" in the sense that we are allowed to process the entire sequence *S* before determining any of the returned keys.

1. In the following instance of the off-line minimum problem, each INSERT is represented by a number and each EXTRACT-MIN is represented by the letter *E*:
    
    [4, 8, E, 3, E, 9, 2, 6, E, E, E, 1, 7, E, 5]

    Fill in the correct values in the extracted array.


To develop an algorithm for this problem, we break the sequence *S* into homogeneous subsequences. That is, we represent *S* by:

[I<sub>1</sub>, E, I<sub>2</sub>, E, I<sub>3</sub>, ..., I<sub>m</sub>, E, I<sub>m+1</sub>]

where each *E* represents a single EXTRACT-MIN call and each *I<sub>j</sub>* represents a (possibly empty) sequence of INSERT calls. For each subsequence *I<sub>j</sub>*, we initially place the keys inserted by these operations into a set *K<sub>j</sub>*, which is empty if *I<sub>j</sub>* is empty. We then do the following:

```
OFF-LINE-MINIMUM(m, n)
    for i = 1 to n
        determine j such that i ∈ K[j]
        if j != m + 1
            extracted[j] = i
            let l be the smallest value greater than j for which set K[l] exists
            K[l] = K[j] ∪ K[l], destroying K[j]
    return extracted
```

2. Argue that the array *extracted* returned by OFF-LINE-MINIMUM is correct.


3.  Describe how to implement OFF-LINE-MINIMUM efficiently with a disjoint-set data structure. Give a tight bound on the worst-case running time of your implementation.

## Answers

1. [4, 3, 2, 6, 8, 1]

2. So here's my less than rigorous explanation of the proposed algorithm:

    Elements in the domain are processed incrementally. If the current element *i* is in one of the sets, we know that it will be extracted by the "next" extraction (which is extraction number *j* if the element is in set *K<sub>j</sub>*), if there is one. How do we know this? We know that *i* couldn't have been extracted by an extraction prior to extraction *j*, as *i* wasn't part of the sequence at that point. We also know that when we reach extraction *j*, *i* is the smallest element in the sequence, as smaller elements have already been extracted in previous iterations of the loop, and the union operation at the end ensured that only "subsequent" extractions are considered for the remaining element inside the sequence.  

3. The general idea builds upon the algorithm proposed in section 2, where you represent the *I<sub>1</sub>...I<sub>m+1</sub>* subsequences through a disjoint set forest. You can easily find an element and its corresponding tree-set through the FIND operation. In order to determine the index *j* of the extraction, you need to somehow label the trees with their appropriate index. For efficiently finding the next tree-set *K<sub>l</sub>* you can use a linked list, but you'll need to keep a mapping from elements to their corresponding linked list node. The union at the end is done trivially through the disjoint set UNION operation.

## Implementation with disjoint sets

I decided to go one step further and try my own implementation of the solution in Python. I am using an implementation of [disjoint sets](https://pypi.org/project/disjoint-set/_) and [linked lists](https://pypi.org/project/pyllist/) found in PyPI. I find the chosen interface for the disjoint sets a bit confusing as there is no INSERT or MAKE-SET operation, instead `find` has the side effect of adding the element to the forest as its own tree if it wasn't there (in other words, `find` is used to both insert and find elements). But hey, at least I didn't have to write my own!

To keep track of the indices and linked list nodes, I am using a dictionary to map each elements to a tuple consisting of its index (what we referred to so far as *j*), and the corresponding linked list node for that element.

We can't have empty sets in the disjoint sets so we have to handle them a bit differently. We keep track of them as `None` in the linked list, and when the "next" tree-set *K<sub>l</sub>* is found to be `None` we increment the index of the current set *K<sub>j</sub>* instead of performing the union operation.

The implementation is two-fold. First we build the disjoint sets forest and the satellite data structures based on the given sequence of elements and extractions. Then we have the implementation of the algorithm described in question 2 with the necessary tweaks.

Here's the code:

```python
#!/usr/bin/env python3

import collections
from disjoint_set import DisjointSet
from pyllist import sllist


def build_forest(sequence):
    forest = DisjointSet()
    root_ll = sllist()  # linked list to easily find next tree-set K(l)
    root_data = {}  # dict mapping elements to their index and linked list node

    if not sequence:
        return forest, root_ll, root_data

    current_tree_root = None
    j = 0
    for key in sequence:
        if key == 'E':
            root_node = root_ll.append(current_tree_root)
            if current_tree_root:
                root_data[current_tree_root] = (j, root_node)
            current_tree_root = None
            j = j + 1
            continue

        if current_tree_root == None:
            forest.find(key)
            current_tree_root = key
        else:
            forest.union(current_tree_root, key)
            # the root for the current set might have changed
            # due to the union by rank heuristic
            current_tree_root = forest.find(key)

    if current_tree_root:
        # put last tree root inside the linked list
        root_node = root_ll.append(current_tree_root)
        root_data[current_tree_root] = (j, root_node)

    return forest, root_ll, root_data


def off_line_minimum(domain, sequence):
    forest, root_ll, root_data = build_forest(sequence)
    
    counter=collections.Counter(sequence)
    extractions_count = counter['E'] 
    extracted = [None for i in range(extractions_count)]

    for key in domain:
        if key in forest:
            key_root = forest.find(key)
            key_root_index, key_root_node = root_data[key_root]
            
            if key_root_index == extractions_count:
                # key is in the tree that was formed after the last extraction
                # it can't be the in extracted array
                continue
            
            extracted[key_root_index] = key
            next_root_node = key_root_node.next
            
            if next_root_node.value == None:
                # union with an empty set:
                # update index of current tree and remove next node
                root_data[key_root] = (key_root_index + 1, key_root_node)
                root_ll.remove(next_root_node)
                continue

            # find next tree-set K(l) and union K(j) with K(l)
            next_root_index, next_root_node = root_data[next_root_node.value] 
            forest.union(key_root, next_root_node.value)

            # the root could be either key_root or next_root
            # due to the union by rank heuristic
            merged_trees_root = forest.find(key_root)
            if merged_trees_root == key_root:
                root_ll.remove(key_root_node.next)
                root_data[key_root] = next_root_index
            else:
                root_ll.remove(key_root_node)


    return extracted


def main():
    sequence = [4, 8, 'E', 3, 'E', 9, 2, 6, 'E', 'E', 'E', 1, 7, 'E', 5]
    extracted = off_line_minimum(set(range(10)), sequence)
    
    assert extracted == [4, 3, 2, 6, 8, 1]


if __name__ == "__main__":
    main()
```

## Final Thoughts

I have to admit, I'm not entirely sure that there are any practical applications for the off-line minimum problem and a quick internet search seemed to mostly confirm that.

But at least I had fun!