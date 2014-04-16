---
layout: page
title: "The Secret Sauce Behind A/B Testing"
comments: true
---

[A/B testing](http://en.wikipedia.org/wiki/A/B_testing) has been
ubiquitous at tech companies to run controled experiments on their
user base in the continual prcoess of feature development.  Companies
like [Twitter](https://blog.twitter.com/2013/experiments-twitter)
are constantly running these tests.

In this post, I will describe the statistical tests required to run
these Controlled Experiments.  By the end of this post, you should
be able design and run an A/B test on your website as well as
understand the statistical principles underlying these the tests.

## Proportions, the Simplest A/B Test

The simplest A/B test tries to optimize the fraction or proportion
of users who perform a given action on a website. For example, we
want to get more people click on a button so we come up with a
hypothesis.

In order to perform this test, we take a certain portion of users
and show them one version of the website and another
portion of users and show them the ohter version.

It is very important that we pick which users to show which version
in a totally random way.  That way, you know that a change in
behavior is due to the change in the website and not due to other
subtle biases caused by giving showing differnet kinds of users the
different websites.

By convention, we tend to say that the group of users who
sees the original experiment is in a control bucket
and the users who see the new website are in the
experiment bucket. 

For larger websites with more traffic,
it is common to put 1% of users in the control bucket and 1% of
users in the experiment bucket.  But for websites with less traffic,
you can go as far as putting half of users in the control bucket
and half of users in the experiment bucket.  It is generally a good
idea to have the same number of users in the control and the
experiment to avoid issues like [Simpson's
paradox](http://en.wikipedia.org/wiki/Simpson's_paradox).

As a simple example, suppose in our experiment we make our button
larger. Suppose we run our experiment on 10,000 users in our control
bucket and 10,000 users in our experiment bucket.

|            | clicks | total views |
| ---------- | ------ | ----------- |
| control    |  4,730 |     10,000  |
| experiment |  4,912 |     10,000  |

In this example we estimate 47% of the users in the control experiment
click on the button and 49% of the users in the experiment. Therefore,
this experiment increased the percent of users clicking on the
button by 2%.

In order to properly understand this experiment, we would like to
test if this change is statistically significant. This is to say
that we are interested in learning if the change is unlikey to be
attributable to random chance.

## Measuring an Increase in Amounts

The other common thing to test during A/B testing is if
the amount of something changes. As an example, we might
be interested in learning if users visit more pages on
a website after a redesign.

Want to A/B test number of pages viewed.

* Using the KS Test...

## Power Calculations 


## Other common cotchas about AB Testing

* Statistical Significance is different from magnitude of effect
