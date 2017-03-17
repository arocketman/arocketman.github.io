---
layout: post
title:  "Using a paired t-test to compare different algorithms."
date:   2017-02-27 14:52:07 +0200
categories: main
icons: 
---
In this post we'll talk a little about statistics, suppose we have a same algorithm implemented in two different languages. We'll use the [sieve algorithm][sieve]
Suppose we have two different implementations of the same algorithm. A Java one and a c++ one, the implementation of such algorithm is pretty straight forward so I'll go straight to the statistics side of the comparision.

First above all, we'll be doing a paired T-Test, where we have two samples with equal sample size so we can compute the difference between each observation. I will fix the amount of numbers in the algoritm to 1 milion.

Let's check out the data (in ms):

C++ with O3 optimization:

294894
295242
292113
287117
285912
298225
283971...

Java with JIT enabled: 

418015
405551
412712
409556
406612
408800
407396...

on a visual analysis, they seem to be different, c++O3 seems faster. But, from a statistical point of view, can we actually say that they are statistically different? To do that, we could use a T-test for normally distributed sample means or a Wilcoxon test (non parametric) for non normally distributed sample means.
Since the Central Limit theorem applies for sample size greater than ~30, and since our sample size is 250, we'll be using a t-test.

First, we will computer the difference between the observations then we'll apply a t-test on such differences. The t-test assumes as null hypotesis H0 that the mean of the observed distribution is 0. So we want to know if the mean of the differences distribution is actually 0. 

I'll be using JMP by SAS to verify this.


![Exec1](/assets/images/t-test.PNG){: .center-image}

Looking at the image we can see that the p-value is < 0.0001, meaning the null hypotesis is rejected, thus confirming that the difference of the two samples is non-zero. So they are statistically different.
Also, notice we have a Wilcoxon test as well that also confirms the non-null of the sample.

We can confirm, even from a statistically point of view, that the speeds of the two algorithms are different, and c++ is actually faster than java in this case.

[sieve]: https://en.wikipedia.org/wiki/Sieve_of_Eratosthenes
