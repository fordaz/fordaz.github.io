---
layout: post
title: "Review of Consistency Models - Part I"
description: "Highlighting definitions, main properties and examples of some of the Consistency models"
tags: [distributed systems, consistency models]
---

A consistency model is a contract offered by a data store to a client application. It defines the behavior when multiple clients or processes perform concurrent read and write operations on the data store, particularly when these operations are in conflict.

Consistency models are important for the following reasons:

* It allows us to know what to expect from a data store and design correct programs.
* It helps us to understand the increasing number and variety of data stores available today.

In the following descriptions and examples we'll assume the following model and conventions:

* Different processes `P1`...`Pn` run simultaneously on different nodes `N1`...`Nn` with local copies of individual data items `x` and `y`, which are replicated to other nodes in the network.
* Time progresses from left to right
* Variables `x` and `y` are initialized with `0`
* `P1:W(x)1` : process `P1` performs a write operation of value `1` on variable `x`
* `P2:R(x)2` : process `P2` performs a read of value `2` from variable `x`.

## Quiescent Consistency

> Overlapping read and write operations can be seem in some sequential order (arbitrary interleaving), 
> while the non overlapping operations are perceived in the same way by all the processes as if 
> they occurred in real-time order.

 This means that during a period of quiescence (no pending operations) operations seems to occur in real-time, and during periods of concurrent operations they seem to occur in an arbitrary order.

For example, the following history is Quiescent Consistent:
![example 1]({{ site.url }}/images/2016-01-23-consistency-models-I-post/history-1.png)

Since `P1:W(x)1` and `P2:W(x)2` are non overlapping, they seem to take place in real-time, and both `P3` and `P4` read the same value `2` from variable `x`. Also, the set of overlapping operations `P2:R(x)2`, `P3:W(x)3` and `P4:R(x)3` allow a read of two different values from `x`.

On the other hand, the following history is not Quiescent Consistent:
![example 2]({{ site.url }}/images/2016-01-23-consistency-models-I-post/history-2.png)

As it violates the property that during non overlapping operations like `P1:W(x)1` and `P2:W(x)2` should appear to occur in real-time order.

**Remember**: Quiescent Consistency allows arbitrary interleaving of overlapping single operations on single single objects, while non overlapping operations must appear to occur in real-time ordering.

---

## Strict Consistency
> Strict consistency guarantees that a given read operation always returns the **most recent** written value of a given data element. 

The **most recent** aspect of the definition implies that we have to have a real-time ordering to be able to unambiguously identify which one was the most recent operation.

If we consider the following two processes `P1` and `P2` on different nodes `N1` and `N2` respectively. 
![example 2]({{ site.url }}/images/2016-01-23-consistency-models-I-post/history-3.png)

Where `P1` writes the value of `1` to `x` and Process `P2` immediately reading it from its local copy. Strict consistency requires that the write operation takes no latency to propagate from `N1` to `N2`, regardless of how far the two nodes are, and how close in time the read operation is to the write operation. This makes this consistency model intuitively easy to understand, but virtually impossible to implement in distributed systems.

**Remember**: Writes are instantaneously visible to all processes and real-time ordering is maintained. 

---

## Sequential Consistency
> Any execution is perceived in the same way by all the processes, as if all the read and write operations were executed in some 
> sequential order (arbitrary interleaving), respecting the program order of each one of them.

For example, The following history is Sequentially Consistent:
![example 2]({{ site.url }}/images/2016-01-23-consistency-models-I-post/history-4.png)

In this case, even when `P1:W(x)1` was invoked first, it really took effect after `P2:W(x)2` (arbitrary interleaving). Both processes `P2` and `P3` see the same sequence of values `2,1` from variable `x`.

On the other hand, the following is not a Sequentially Consistent history:
![example 2]({{ site.url }}/images/2016-01-23-consistency-models-I-post/history-5.png)

Because process `P2` is seeing the sequence `2,1` while `P3` is seeing `1,2` from the same variable `x`. This violates the property that all processes see the same interleaving of the operations.

Similarly, the following is not a Sequentially Consistent history:
![example 2]({{ site.url }}/images/2016-01-23-consistency-models-I-post/history-6.png)

As it does not respect the program order of the process `P3`, since the first read operation `P3:R(y)1` should have had returned `0` instead of `1` because the operation `P3:W(y)1` comes as the second operation.

Sequential Consistency applies to single objects (ex: only `x` or `y` but not both) and single operations (ex: read or write, but not grouping of multiple reads and/or writes).

**Remember**: Sequential Consistency allows arbitrary interleaving of single operations on single single objects.

---

## References
* Distributed Systems, Principles and Paradigms by Andres S. Tanenbaum, Maarten van Steen.
* The Art of Multiprocessor Programming by Maurice Herlihy and Nir Shavit.
* [Aphyr's post about Strong Consistency Models](https://aphyr.com/posts/313-strong-consistency-models/)
* [Peter Bailis' post about Linearizability vs Serializability](http://www.bailis.org/blog/linearizability-versus-serializability/)