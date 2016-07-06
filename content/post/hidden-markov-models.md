+++
title = "Hidden Markov Models"
date = "2011-05-30"
tags = ["ai", "ghmm", "hmm", "markov", "python"]
+++

Nowadays, many applications use
[Hidden Markov Models](http://en.wikipedia.org/wiki/Hidden_Markov_model)
(HMMs) to solve crucial issues such as bioinformatics, speech recognition,
musical analysis, digital signal processing, data mining, financial
applications, time series analysis and many others. HMMs are
probabilistic models which are very useful to model sequence behaviours
or discrete time series events. Formally it models
[Markov processes](http://en.wikipedia.org/wiki/Markov_process)
with hidden states, like an extension for
[Markov Chains](http://en.wikipedia.org/wiki/Markov_chain).
For computer scientists, is a state machine with probabilistic transitions
where each state can emit a value with a given probability.

For better understanding HMMs, I will illustrate how it works with "The
Fair Bet Casino" problem. Imagine you are in a casino where you can bet
on coins tosses, tossed by a dealer. A coin toss can have two outcomes:
head (H) or tail (T). Now suppose that the coin dealer has two coins, a
fair (F) which outputs both H and T with `1/2` probabilities and a biased
coin (B) which outputs H with probability `3/4` and T with `1/4`. Using
probability language we say:

-   `P(H~i+1~|F~i~) = 1/2`
-   `P(T~i+1~|F~i~) = 1/2`
-   `P(H~i+1~|B~i~) = 3/4`
-   `P(T~i+1~|B~i~) = 1/4`

Now imagine that the dealer changes the coin in a way you can't see, but
you know that he does it with a `1/10` probability. So thinking the coin
tosses as a sequence of events we can say:

-   `P(F~i+1~|F~i~) = 9/10`
-   `P(B~i+1~|F~i~) = 1/10`
-   `P(B~i+1~|B~i~) = 9/10`
-   `P(F~i+1~|B~i~) = 1/10`

We can model it using a graph to illustrate the process:

![HMM for "The Fair Bet Casino" problem](/img/fairbet.png)
  
That's a HMM! It isn't any rocket science. Is just important to add a
few remarks. We call the set of all possible emissions of the Markov
process as the alphabet Σ ({H, T} in our problem). For many of
computational method involving HMMs you will also need a initial state
distribution π. For our problem we may assume that the we have equal
probability for each coin.
 
Now comes in our mind what we can do with the model in our hands. There
are lot's of stuff to do with it, such as: given a sequence of results,
when the dealer used the biased coin or even generate a random sequence
with a coherent behaviour when compared to the model.
 
There is a nice library called [ghmm](http://ghmm.org/)
(available for C and Python) which handles HMMs and already gives us
the most famous and important HMM algorithms.
Unfortunately the python wrapper is not pythonic.
Let's model our problem in python to have some fun:

```
import ghmm 

# setting 0 for Heads and 1 for Tails as our Alphabet
sigma = ghmm.IntegerRange(0, 2)

# transition matrix: rows and columns means origin and destiny states
transitions_probabilities = [
    [0.9, 0.1], # 0: fair state
     [0.1, 0.9], # 1: biased state
]  

# emission matrix: rows and columns means states and symbols respectively 
emissions_probabilities = [
    [0.5, 0.5], # 0: fair state emissions probabilities
    [0.75, 0.25], # 1: biased state emissions probabilities
]

# probability of initial states
pi = [0.5, 0.5]

# equal probabilities for 0 and 1
hmm = ghmm.HMMFromMatrices(
    sigma,
    # you can model HMMs with others emission probability distributions
    ghmm.DiscreteDistribution(sigma),
    transitions_probabilities,
    emissions_probabilities,
    pi )


>>> print(hmm)`\
  DiscreteEmissionHMM(N=2, M=2)
      state 0 (initial=0.50)
          Emissions: 0.50, 0.50
          Transitions: ->0 (0.90), ->1 (0.10)
      state 1 (initial=0.50)
          Emissions: 0.75, 0.25
          Transitions: ->0 (0.10), ->1 (0.90)
```

Now that we have our HMM object on the hand we can play with it. Suppose
you have the given sequence of coin tosses and you would like to
distinguish which coin was being used at a given state:

```
tosses = [1, 1, 1, 0, 1, 0, 0, 1,
          1, 0, 0, 1, 0, 1, 0, 0,
          0, 0, 1, 1, 0, 1, 0, 1,
          0, 0, 0, 1, 0, 0, 0, 0,
          0, 1, 0, 0, 1, 0, 0, 0,
          1, 0, 1]
```
 

The [viterbi algorithm](http://en.wikipedia.org/wiki/Viterbi_algorithm)
can be used to trace the most probable states at each coin toss
according to the HMM distribution:

```
# not as pythonic is could be :-/
sequence = ghmm.EmissionSequence(sigma, tosses)
viterbi_path, _ = hmm.viterbi(sequence)
>>> print(viterbi_path)
  [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, , 1, 1, 1, 1, 1]
```

Nice! But sometimes is interesting to have the probability of each state
on the point instead of only the most probable one. To have that, you
must use the [posterior](http://en.wikipedia.org/wiki/Posterior_mode) or
[forward](http://en.wikipedia.org/wiki/Forward_algorithm) algorithms to
have more detailed information.

```
states_probabilities = hmm.posterior(sequence)
>>> print(states_probabilities)
 [[0.8407944139086141, 0.1592055860913865], [0.860787703168127, 0.13921229683187356], ... ]
```

The posterior method result, returns the list of probabilities at each
state, for example, in the first index we have `[0.8407944139086141, 0.1592055860913865]`.
That means that we have ~0.84 probability of chance that the dealer is
using the fair coin and ~0.16 for the biased coin.
We also can plot a graph to show the behaviour of the curve of
the probability of the dealer being using the fair coin
(I used matplotlib for the graphs).

![Probability of being a fair coin over time](/img/viterbi.png)

This is only a superficial example of what can HMMs do. It's worthy give
a look at it if you want do some sequence or time series analysis in any
domain. I hope this post presented and cleared what are HMM and how they
can be used to analyse data.
