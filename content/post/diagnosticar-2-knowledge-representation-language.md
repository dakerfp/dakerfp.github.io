+++
title = "DiagnostiCar (2) - Knowledge Representation Language"
date = "2010-04-11"
tags = ["abduction", "diagnosticar", "logic", "operator", "prolog", "reasoning", "shell", "swi"]
+++
I'm giving sequence to the last post about the DiagnostiCar expert
system. Today I'm going to specify the predicates to represent some
domain specific knowledge.

Prolog has a helpful and clean [syntax](http://www.csci.csusb.edu/dick/samples/prolog.syntax.html), at least when you're used to, and it's easy to define new [operators](http://www.swi-prolog.org/pldoc/man?section=operators) syntax in SWI.
Such feature improves the readability of the knowledge base or other DSL you want to create.

To make an abductive reasoning we will need at least the logic operators
'and','or' and 'implies' to represent knowledge and chain it.

Our simple language will have the following syntax:

- `known X`: Says that the sentence X must be true;
- `X \<- Y - X`: X is caused by Y;
- `X & Y`: Is true if X and Y are true;
- `X v Y`: Is true if X or Y are true ('|' is already used by
    Prolog);
- `assumable X`: X can be assumed to prove something;
- `askable X`: X can be asked to the user;

Note that I'm not using the negation, because some dangerous
particularities of Prolog negation as failure.

It's interesting to stress the difference if we represented 
the knowledge base with the regular prefixed notation of
Prolog predicades:

```prolog
- known(X)
- <-(X,Y)
- &(X,Y)
- v(X,Y)
- assumable(X)
- askable(X)
```

Defining opertors increases the readabilty of the knowledge
representation significantly. To redefine operators we will use the SWI
predicate `op/3`.
We can define this predicate this way:

```prolog
:- op(910, xfy, &).
```

The `op/3` predicate has three fields `op(+Precedence, +Type, :Name)`.
The first one says the precendence of the operator (a number between 0
and 1200). The second is how the functor will be placed within it's
"children nodes", for example: `xfy` let the functor between the
arguments while `xf` let the functor after it's argument (remembering
that you can only create unary or binary operators). For our
representation language we have:

```prolog
:- discontiguous(     known/1 ). % The discontiguous predicate tells Prolog
:- discontiguous( assumable/1 ). % that the given predicate can be declared
:- discontiguous(   askable/1 ). % unsorted.

:- op(910, xfy,      &    ). % a higher priority with an infix notation
:- op(920, xfy,      v    ). % infix notation and lower priority than &
:- op(930, xfy,     <-    ). % infix notation
:- op(940,  fx,   known   ). % prefixed notation and lowest priority
:- op(940,  fx, assumable ). % prefixed notation and lowest priority
:- op(940,  fx,   askable ). % prefixed notation and lowest priority
```

Now let's model the car problems knowledge into our language (remember
that Upper names are variables):

```prolog
%
% DiagnostiCar - Knowledge Base
%

% Observe that implication is seen as the real sense of consequence <- cause.
known problem(crank_case_damaged) <- crank_case_damaged.
known problem(hydraulic_res_damaged) <- hydraulic_res_damaged.
known problem(brakes_res_damaged) <- brakes_res_damaged.
known problem(old_engine_oil) <- old_engine_oil & leak_color(black).

known leak(engine_oil) <- crank_case_damaged.
known leak(hydraulic_oil) <- hydraulic_res_damaged.
known leak(brakes_oil) <- brakes_res_damaged.

% Implications in false can be seen as constraints.
known oil(Oil) <- exists_leak(true) & leak(Oil).
known false <- oil(brakes_oil) & leak_color(Color) & Color \= green. 
known false <- oil(engine_oil) & leak_color(Color) & Color \= brown & Color \= black.
known false <- oil(hydraulic_oil) & leak_color(Color) & Color \= red.

% The assumable predicate asserts what will be can be suposed 
assumable crank_case_damaged.
assumable hydraulic_res_damaged. % to prove something e else.
assumable brakes_res_damaged.
assumable old_engine_oil.

askable leak_color(Color). % Askable predicate asserts which information will be asked for the user.
askable exists_leak(TrueOrFalse). % The argument of the predicate will be unified with the answer.

known goal(X) <- problem(X). % The goal predicate is what we want to prove with the expert system.
```

That's all for now. On the next post I will explain the question and
answer interface using the dynamic Prolog predicates.
