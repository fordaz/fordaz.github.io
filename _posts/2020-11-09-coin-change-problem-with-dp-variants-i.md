---
layout: post
title: "Coin change problem variants (I)"
description: "Reviewing Dynamic Programming by comparing and contrasting different variantes of the Coin change problem"
tags: [dynamic programming, coin change problem, algorithm design, programming interviews]
comments: true
usemathjax: true
---

# Dynamic Programming

This is not a tutorial on Dynamic Programming, but more of a practice session where we take few variants of the Coin Change problem, and step by step, break down the analysis into the fundamental insights that will lead us to put together a solution. If you are looking to brush up on DP or simply get started on it, these are two of my favorite resources [Tim Roughgarden's lectures](https://www.youtube.com/watch?v=0awkct8SkxA&list=PLXFMmlk03Dt5EMI2s2WQBsLsZl7A5HEK6&index=39&ab_channel=StanfordAlgorithms), as well as [Eric Vigoda's lectures](https://www.udacity.com/course/introduction-to-graduate-algorithms--ud401).

The way we'll break down the analysis, is based on the following questions or steps:

* How can we define the sub-problems?
>DP is all about solving smaller sub-problems, so this is where we start.
* What preliminary results need to be computed?
> Sub-problem solutions need to be calculated and stored so we can use them later to build higher level solutions.
* What are the base cases and how do we initialize them ?
> Our base cases give us the starting point to incrementally build our solutions.
* How to define the solution of a problem given the solutions of the sub-problems?
>This is where the magic happens, here we define how a given solution is expressed on top of smaller problem solutions.

NOTE: Even though some of these problems can be solved using recursion and other techniques, I've tried to use a consistent solution structure, so we can compare the different variants.

# The Coin change Problem

## Variant 1: 
> Given unlimited coins of each denomination, determine if a coin change is possible for a given amount. Implement a function that returns True or False accordingly.

### How can we define the sub-problems?

* We can have sub-problems for either smaller amounts or smaller number of coins.
* Since we have unlimited coins per denomination, let's define our sub-problem by subtracting a given coin denomination and getting a smaller amount.
* We can change a given amount $$a$$, if the smaller amount $$a- d_i$$ (after subtracting denomination $$d_i$$) can be changed. 
* For example: given initial amount 3, and denominations **[1,2]**, the question: is there a coin change for the amount of 3?, can be translated into, can we change **3-1=2**?, which then translates into, can we change **2-2=0**?, which turns out to be possible.

### What preliminary results need to be computed?

* In DP problems, solutions to problems are computed by combining solutions to sub-problems. This usually requires the storage of some preliminary results.
* In many cases, the final answer comes directly from the stored results once we've computed all the sub-problems.
* This suggests the idea of storing **true/false** values for our sub-problems.
* Let's define **results[a]** as the **true/false** value, indicating if it is possible to do the coin change for the amount $$a$$.
* Since we have unlimited coins, the **results[a]** does not depend on which coins or how many coins we have used so far. 

### What are the base cases and how do we initialize them ?

* If we keep on subtracting coin denominations to the current amount, we eventually reach **results[0]**. Of course, we must avoid endless subtractions of coins, getting into negative amounts.
* We can see this as the indication that we were able to effectively do the coin change, otherwise we would have ended with an amount different from **0** for which there are no coins. For example: the remaining amount is 2, and the available denominations are **[3,5]**.
* We then get to our base case **results[0] = True**.
* Since not all the amounts can be changed, given any set of denominations, let's use **False** as our default value for all our **results**.

### How to define the solution of a problem given the solutions of the sub-problems?

* We can say, that a given amount $$a$$ can be changed, if the amount after subtracting any of the given denominations can also be changed.
* Let be $$a$$ a given amount, and $$d_i$$ the $$i-th$$ denomination, then the recurrence expression is:

$$results[a] = results[a-d_0] \vee results[a-d_1] ... \vee results[a-d_n], \quad for \quad d_i <= a$$
    
### Solution

One way to implement this solution in Python is

```python
def coin_change(denominations, amount):
    results = [False]*(amount+1)
    results[0] = True
    for a in range(1, amount+1):
        for d in denominations:
            if d <= a:
                results[a] = results[a] or results[a - d]
    return results[amount]
```

Comments:

* It is common in DP problems, that the length of the results array is padded with an extra slot for the base case, in this problem we use **amount+1** array elements to include **amount = 0**.
* The final result comes nicely as **results[n]**. This is not always the case, but it helps to assume this as the starting point.

---

## Variant 2

> Using at most one of each coin denomination, determine if coin change is possible. Implement a function that returns True or False accordingly.

### How can we define the sub-problems?

* Since coin denominations can be used at most once, the sub-problems need to be expressed as a function of the available coins and a smaller amount.
* One smaller sub-problem can be defined as changing the same amount, with fewer coins.
* For example: Initial amount 3, and denominations **[1,2,5]**, the question: can we change 3 using the coins **[1,2,5]**?, translates into, can we change 3 using only coins **[1,2]**, which we already know can be done. The reason is we cannot use the coin **5** to change a smaller amount.
* Another sub-problem can be defined by subtracting one of the coin denominations, but this time, using the set of coins except the one we just used.
* For example: Initial amount 3, and denominations **[1,2]**, the question: can we change 3 using the coins **[1,2]**?, translates into, can we change 3-2=1 using only **[1]**, which then translates into, can we change 1-1=0? Indicating it is possible.

### What preliminary results need to be computed?

* Since we need to keep track of both the smaller amount, as well as the available coins, we need a 2D array to represent our sub-problems.
* Let's define **results[a][j]** as the true or false value, indicating if it is possible to do the coin change for the amount $$a$$, using the first **j** coin denominations (including **j**).

### What are the base cases and how do we initialize them ?

* The first case is, a zero amount with some remaining coins **results[0][j]** which is **True**, as a zero amount can be changed with anything.
* The second case is, a non zero amount with no coins **results[a][0]** which is **False**, as it is not possible to change any amount with no coins.
* We can initialize everything as **False**, and only the first row of **results** as **True**.

### How to define the solution of a problem given the solutions of the sub-problems?

* Let be $$a$$ a given amount, and $$j$$ the index of the **j-th** denomination, then the recurrence expression is:
> **results[a][j] = results[a][j-1] if d_j > a** or \
> **results[a][j] = results[a-d_j][j-1] or results[a][j-1] if d_j <= a**
* The first case is clear, as we cannot substract **d_j** from **a**, we just ignore the **j-th** coin.
* The first part of the second case (**results[a-d_j][j-1]**) is clear, we subtract the **j-th** coin, and continue with the smaller amount and the remaining coins.
* The second part of the second case (**results[a][j-1]**), begs the question: do we really to consider it again?
* The answer is yes, consider the case **a=2** and denominations **[2,1]**, we can certainly discount either 1 or 2 from 2, but 2-1=1 gives us no answer, as we cannot use the denomination 1 a second time. Forcing us to explore the sub-problem of trying the same amount 2 with the single denomination of **[2]**.

### Solution

One way to implement this solution in Python is

```python
def coin_change(denominations, amount):
    num_denom = len(denominations)
    results = [[False]*(num_denom+1) for x in range(amount+1)]
    results[0] = [True]*(num_denom+1)
    for a in range(1, amount+1):
        for j in range(1, num_denom+1):
            d = denominations[j-1]
            if d > a:
                results[a][j] = results[a][j-1]
            else:
                results[a][j] = results[a-d][j-1] or results[a][j-1]
    return results[amount][num_denom]
```

Comments:

* Compare how the **results** turned out to be a 2D array in this variant, compared to 1D in the first variant. 
* Suggestion, think about how many things you need to keep track of, for example, in the variant #1, keeping track of the smaller amount was enough, but in the second one it was not. These things to track, can be seen as the "degrees of freedom" of the problem, and usually influences the number of dimensions of the data structure to solve the problem.
* Notice how the recurrence expression was broken into different cases (**d_j > a vs d_j <= a**). Also, notice how there two cases, in which the **j-th** is either part of the solution or not.
* Again, the final result comes nicely as **results[amount][num_denom]**.

### Take away

* Keep an eye on 1D vs 2D arrays for your **results**, think about what needs to be remembered for your sub-problems, and start simple.
* Solutions are built bottom up, starting with the base cases.
* Break the recurrence expression into cases as needed.
* Start by defining your cached results such that they give you the final answer directly.
* The interplay of the base cases, default values, the recurrence expression is where the magic of DP happens. Minor changes on any of these, make the solution to work for a different problem.

---

## References
* Leetcode's [coin change](https://leetcode.com/problems/coin-change/)
* Leetcode's [coin change II](https://leetcode.com/problems/coin-change-2/)