---
layout: post
title: "Review of Consistency Models - Part II"
description: "Highlighting definitions, main properties and examples of more Consistency models"
tags: [distributed systems, consistency models]
comments: true
---

In the [Review of Consistency Models - Part I](http://fordaz.github.io/consistency-models-I-post/) we looked at:

* Quiescent Consistency.

* Strict Consistency.

* Sequential Consistency. 

This post covers Linearizability, Serializability, Strict Serializability and Causal Consistency.

## Linearizability
> Any execution is perceived in the same way by all the processes, as if all the read and write operations were executed in real-time 
> order and respecting the program order of each one of them.

In a Linearizable data store:

1. The write operations appear to occur instantaneously.
1. Once the most recent value has been read, any subsequent read returns the same or an even more recent value.

The following history is Linearizable:
![example 2]({{ site.url }}/images/2016-01-23-consistency-models-II-post/history-7.png)

As expected, once `P2: W(x)2` takes place, any process sees the most recent value in any subsequent read operation.

On the other hand, The following history is not Linearizable:
![example 2]({{ site.url }}/images/2016-01-23-consistency-models-II-post/history-8.png)

Because, in real-time ordering the operation `P2:W(x)2` takes places after `P1:W(x)1`, and any subsequent read operation must return the value of `2`.

Linearizability is composable, which means that the combination of individually Linearizable data items, is also Linearizable. 

In order to develop some intuition about this property, let's consider these two independently Linearizable histories for two different data items.
![example 2]({{ site.url }}/images/2016-01-23-consistency-models-II-post/history-9.png)

![example 2]({{ site.url }}/images/2016-01-23-consistency-models-II-post/history-10.png)

If we combine them, then the composed history is also Linearizable, and there is no way of making it non Linearizable and at the same time maintain the individual ones Linearizables.
![example 2]({{ site.url }}/images/2016-01-23-consistency-models-II-post/history-11.png)

**Remember**: Linearizability allows single operations on single objects ordered in real-time.

---

## Serializability
> Any execution is perceived in the same way by all the processes, as if groups of read and write operations were executed in some 
> sequential order (arbitrary interleaving), respecting the program order of each one of them.

As you can see, Serializability is similar to Sequential Consistency, but at the level of group of operations (think transactions). In this case, the group of operations take place atomically in arbitrary interleaving.

Assuming that operations within parenthesis occur as a group, the following history is Serializable:
![example 2]({{ site.url }}/images/2016-01-23-consistency-models-II-post/history-12.png)

As it can be rearranged in the following sequence:
![example 2]({{ site.url }}/images/2016-01-23-consistency-models-II-post/history-13.png)

Which is consistent with the fact that `P3:W(y)1` and `P3:R(x)1` happened before `P2:R(y)1` and `P2:W(x)2`. Notice how the group of operations were ordered arbitrarily and not respecting the real-time order of their invocations.

On the other hand, the following history would not be Serializable:
![example 2]({{ site.url }}/images/2016-01-23-consistency-models-II-post/history-14.png)

As `P2:R(y)1` implies that the `P3` group of operations happened before `P2`'s group, but `P3:R(x)2` implies the opposite.

**Remember**: Serializability allows group of operations on multiple objects ordered in arbitrary interleaving.

---

## Strict Serializability
> Any execution is perceived in the same way by all the processes, as if groups of read and write operations were executed in 
> real-time order and respecting the program order of each one of them.

In other words, Strict Serializability borrows the real-time ordering from Linearizability and applies it to groups of operations as indicated by the Serializability definition.

The first example from Serializabilty is not Strict Serializable
![example 2]({{ site.url }}/images/2016-01-23-consistency-models-II-post/history-15.png)

As it does not respect real-time ordering, but just by making a little adjustment, it becomes Strict Serializable:
![example 2]({{ site.url }}/images/2016-01-23-consistency-models-II-post/history-16.png)

As this respects the fact that `P2` group of operations happened before `P3`.

**Remember**: Strict Serializability allows group of operations on multiple objects ordered in real-time.

---

## Causal Consistency
> Write operations that are potentially causally related are perceived in the same way by all the processes, as if they were executed 
> in real-time order and respecting the program order of all the processes.

In other words, Causal Consistency is similar to Linearizability, but instead of requiring that **all** operations must be ordered in real-time, it is limited only to **causally related** operations.

Two operations are causally related if the execution of one could affect the result of the other one. Consider the following examples:

* A read operation on `x`, followed by a write operation on `y` are potentially causally related as the write value of `y` might be derived by the value of `x`.

* A write operation on `x`, followed by a read operation on `x` are potentially causally related as the read value of `x` is returning the previously written value.

* Two simultaneous write operations on different or same variables are not causally related.

* Two operations not causally related are called concurrent.

The following history is Causally Consistent:
![example 2]({{ site.url }}/images/2016-01-23-consistency-models-II-post/history-17.png)

As `P1:W(x)1` and `P2:W(x)2` are considered concurrent and other processes can see their writes in different order.

Just by introducing a small variation, the following history is not Causally Consistency:
![example 2]({{ site.url }}/images/2016-01-23-consistency-models-II-post/history-18.png)

Since `P2:R(x)1` was generated by `P1:W(x)1` and might affect the result of `P2:W(x)2`, therefore `P1:W(x)1` and `P2:W(x)2` are causally related and must be seen by all the processes in the same order.

Yet, another small variation makes this history Causally Consistent:
![example 2]({{ site.url }}/images/2016-01-23-consistency-models-II-post/history-19.png)

According to a real-time ordering `P1:W(x)1` occurred after `P2:R(x)0` and cannot affect it. Therefore, `P1:W(x)1` and `P2:W(x)2` are not causally related.

**Remember**: Causal Consistency allows single operations on single objects, but only causally related operations must be ordered in real-time.

---

## References
* Distributed Systems, Principles and Paradigms by Andres S. Tanenbaum, Maarten van Steen.
* The Art of Multiprocessor Programming by Maurice Herlihy and Nir Shavit.
* [Aphyr's post about Strong Consistency Models](https://aphyr.com/posts/313-strong-consistency-models/)
* [Peter Bailis' post about Linearizability vs Serializability](http://www.bailis.org/blog/linearizability-versus-serializability/)
