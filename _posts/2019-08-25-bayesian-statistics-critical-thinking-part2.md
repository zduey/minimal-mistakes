---
title: Critical Thinking, Bayesian Statistics, and Effective Political Discourse -- Part 2

categories:
    - miscellaneous
tags:
    - bayesian
    - personal
image:
    thumb: /assets/images/Thomas_Bayes.gif
    credit: wikimedia
    creditlink: https://commons.wikimedia.org/wiki/Thomas_Bayes
---

# Background

In this part, I’ll introduce some concepts that will be important for understanding the crux of the analogy in part 3. Even if some of the details remain murky, my hope is that everyone can take away the key ideas. In his excellent book, *Statistical Rethinking*, Richard McElreath provides a great introduction to these concepts, so rather than reinvent the wheel, the remainder of this post leverages the same example, but in less detail. For anyone interested in diving deeper into Bayesian statistics, I highly recommend picking up a copy of [Statistical Rethinking](https://xcelab.net/rm/statistical-rethinking/).

Even if you have never taken a statistics course, you have almost certainly encountered word problems either in school or on standardized tests that start with something like: “There is a bag with 1 blue marble and 3 white marbles. Jill reaches into the bag and pulls out a marble. What is the probability that Jill selects a blue marble?” In this example, the **event** of interest is: Jill selects a blue marble. Since we know the contents of the bag, we can say that the **probability** of this event occurring is 1/5 or 20%. But what if we did not know the exact contents of the bag, and only that there were a total of 4 marbles that are either blue or white? How could we make a judgement about the contents of the bag? How could we make an educated guess if all we were allowed to do was select a marble, observe its color, and put then put the marble back in the bag?

As the example above makes clear, probability is all about counting the ways that some event could occur. Bayesian inference directly builds upon this idea: if you want to know how likely some event is relative to another, count the different ways that each could occur and the one with more ways is more plausible.

In our example of the bag containing 4 marbles, there are only 5 possibilities for the contents of the bag: all white, one blue and four white, two blue and three white, four blue and one white, and all blue. Let’s call each of these possibilities a **conjecture** (to use McElreath’s term): our goal is to determine which conjecture is the most plausible. To do so, we need some **evidence** about the bag’s contents. We follow the procedure alluded to earlier: we pick out a marble, observe its color, and put it back. We do this three times. For the sake of argument, let’s assume that we chose a blue marble twice in a row and then a white marble. For each of the 5 possible bag configurations, our goal is to count how many times this particular sequence (Blue, Blue, White) could have occurred. Once we have this information, we can rank the possible bag configurations from least plausible to most plausible.

We can immediately rule out bags with only one color marble, so we only need to consider the remaining three. To keep things from getting too tedious, consider the conjecture that there is one blue and three white marbles in the bag. Since there are 4 possible marbles to be drawn each time and 4 rounds, there are 64 different possible sequences of marbles. Can you think of a way to enumerate the possible ways to produce the observed sequence? How many were there? Figure 1 shows one way that the possibilities can be enumerated. Only a *subset* of possible branches are shown. The solid lines indicate possibilities that are consisent with the observed sequence. We can see that there are three ways for the sequence Blue, Blue, White to have emerged if the bag contained three white marbles and one blue marble.

<figure class="full">
    <a href="/assets/images/marble_selector.png"><img src="/assets/images/marble_selector.png"></a>
</figure>

The following table shows the number of ways that the observed sequence could have been achieved for each conjecture.

| Bag (Conjecture)           | Count | Plausibility |
|----------------------------|:-----:|:------------:|
| Blue, Blue, Blue, Blue     |   0   |       0      |
| Blue, Blue, Blue, White    |   27  |  27/38 = 71% |
| Blue, Blue, White, White   |   8   | 8/38 = 21%   |
| Blue, White, White, White  |   3   | 3/28 = 8%    |
| White, White, White, White |   0   | 0            |

These counts represent the **likelihood** of each conjecture. Noting that there are a total of 38 ways to produce the observed sequence, we can turn these into probabilities. These are the *posterior probabilities* of each conjecture given the evidence.

So far, we have implicitly assumed that each bag configuration is equally likely. How would these posterior probabilities change if we were told by the bag manufacturer that for every bag with all white or all blue marbles, they make three bags with an equal number of colored marbles and two bags of all other types? This information provides a **prior probability** for each conjecture. We can incorporate this information by multiplying the counts we have already computed by the prior counts given to use by the manufacturer. Table 2 shows the updated counts for our example. Although a bag containing three blue marbles is still the most plausible conjecture, it is more plausible that the bag has an equal number of blue and white marbles compared to what we had previously computed in Table 1.

| Bag (Conjecture)           | Prior Counts | Count | Updated Count | Plausibility |
|----------------------------|--------------|:-----:|:-------------:|--------------|
| Blue, Blue, Blue, Blue     |       1      |   0   |   1 x 0 = 0   |       0      |
| Blue, Blue, Blue, White    |       2      |   27  |  2 x 27 = 54  |  54/84 = 64% |
| Blue, Blue, White, White   |       3      |   8   |   3 x 8 = 24  |  24/84 = 29% |
| Blue, White, White, White  |       2      |   3   |   2 x 3 = 6   |   6/84 = 7%  |
| White, White, White, White |       1      |   0   |   1 x 0 = 0   |       0      |

Although *Statistical Rethinking* goes into much greater detail, we have now introduced the key concepts that will come up in part 3 of this series. As a warning, we will be taking a much more abstracted view of these concepts in the following post, especially when it comes to the likelihood. These concepts were introduced in a very methodical manner: counting the possible ways that some event could have occurred. Unfortunately, this sort of enumeration often breaks down when the conjectures are no longer about marbles in a bag. That caveat aside, I hope the extended analogy will prove to be instructive and useful.
