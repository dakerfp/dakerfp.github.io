+++
title = "DiagnostiCar (4) - Abductive Reasoning"
date = "2010-07-29"
tags = ["abduction", "diagnosticar", "logic", "prolog", "reasoning", "swi"]
+++

Finally I had some time to write about abductive reasoning, the core of
the DiagnostiCar expert system. As you may know from my [previous posts](http://codecereal.blogspot.com/2010/04/diagnosticar-introduction.html)
(means you're tough and reached this point), the abductive reasoning
tries to infer what facts implies a given situation.

To build such a reasoning we must start with the conclusion and try to
prove it. Let's say that we have the following scenario:

```prolog
known mortal(X) <- human(X).
known human(socrates).
```

How abductive reasoning behaves when tries to find out what things are
mortal?

The first step is to check if **mortal(X)** is a tautology such as:\

```prolog
known mortal(xerxes).
```
As in our example there isn't such tautology the reasoning agent must
try to find a rule which implies that something is mortal. In our case
proceeds that `mortal(X) <- human(X)` so to prove that something is
mortal we have to prove that it is human. The first step is then
repeated and we reach the tautology `human(socrates)` which asserts
that Socrates is mortal.

**Note** that we have no difference with we try to use Prolog directly:

```prolog
mortal(X) :-
    human(X).

human(socrates).
```

Now consider a non trivial example. Imagine that we want to prove that
`mortal(aristoteles)` but we may let the reasoner conclude that such
affirmation is only possible if Aristoteles is a human. Pretty smart
huh? That's what abductive reasoning is capable of. To build such a
predicate we will need to know not only what are proving but also the
user id (to ask questions when possible) and a list to yield what is
being assumed.

Lets try to prove a goal using the following rules (together with
the Prolog code):

1 - Check if the goal is a tautology:

```prolog
abduction(_, Goal, _) :-
    known(Goal).
```

2 - Check equalities:

```prolog
abduction(_, Value == Value, _) :-
    Value = Value.

abduction(_, Value \= Value, _) :-
    Value \= Value.
```

3 - Check if the goal is implied by something else:

```prolog
abduction(Uid, Goal, Assuming) :-
    known(Goal <- Cause),
    abduction(Uid, Cause, Assuming).
```

4 - If the goal is an or or an and conjunction:

```prolog
abduction(Uid, Left & Right, Assuming) :-
    abduction(Uid, Left, Assuming),
    abduction(Uid, Right, Assuming).

abduction(Uid, Left v _, Assuming) :-
    abduction(Uid, Left, Assuming).

abduction(Uid,_ v Right,Assuming) :-
    abduction(Uid, Right, Assuming).
```

5 - Ask the user:

```prolog
abduction(Uid, Goal, _) :-
    functor(Goal,Question,1), 
    askable(Question),
    arg(1,Goal,Answer),
    ask(Uid,Question,Answer). % Defined in a previous post
```

6 - Make an assumption:
 
```prolog
abduction(_, Goal, Assuming) :-
    assumable(Goal),
    member(Goal, Assuming).
    % Says that the goal must be assumed to reach the goal.
```

This does it's task but we still have some problems. The first one is
the lack of constraints. This reasoning doesn't consider if you assume
something that implies in a contradiction. e.g.: `known false <-
mortal(X) & immortal(X)`. Another problem we can observe is that the
system doesn't follow the Occam's razor principle, which affirms that
between two valid explanation the simplest tends to be true. Finally we
have a crucial problem is a infinite loop occurence when you try to
prove the negation of something and the goal is a tautology.

Let's solve those problems by parts. The first can be solved with a
predicate which proves the goal and doesn't reach a contradiction at the
same time:

```prolog
reason(Uid, Goal, Assuming) :-
    abductive_reasoning(Uid, Goal, Assuming),
    not(abductive_reasoning(Uid, false, Assuming)).
```

But that predicate falls directly to the third problem. To solve this
problem, the must intuitive solution is do a search starting with a
empty list of assumptions and try incrementing the list size until it
reachs a cutoff that you may assume that there is no explanation bigger
than that. A simple and clear implementation of that would be:

```prolog
max_assumptions(5).
abductive_reasoning(Uid, Goal, Assuming) :-
    max_assumptions(Max),
    abduction(Uid, Goal, Assuming, Max).

abduction(Uid, Goal, Assuming, Depth) :-
    Depth >= 0,
    max_assumptions(Max),
    Len is Max - Depth,
    length(Assuming, Len),
    abduction(Uid, Goal, Assuming).

abduction(Uid, Goal, Assuming, Depth) :-
    NextStep is Depth - 1,
    NextStep >= 0,
    abduction(Uid, Goal, Assuming, NextStep).
```

It is not the most efficient code to handle code but works fine. You
can avoid process a search path again by creating a dynamic predicate
and asserting when a branch of the search is truth or false. It may use
a lot of memory but it helps to execute faster.

