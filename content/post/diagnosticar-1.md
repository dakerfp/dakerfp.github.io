+++
title = "DiagnostiCar (1) - Introduction"
date = "2010-04-07"
tags = ["abduction", "diagnosticar", "logic", "prolog", "reasoning", "swi"]
+++
I'm here to tell my expirience with logic programming in a grad
project: DiagnostiCar, an web expert system for car diagnosis.

![DiagnostiCar](/img/diagnosticar.jpg)

The DiagnostiCar core feature was an expert system for giving hints to
laypeople about their cars symptoms and their causes and consequences to
avoid being deceived by mechanics (an habit of the job). Communication
between the system and the users was conceived to happen with multiple
choice questions forms and at the end the system gives the possible
causes for the observed car characteristics.

This is a video of the DiagnostiCar expert system interface working:

<iframe width="640" height="360" src="https://www.youtube.com/embed/OxsAWwmGL88?feature=player_embedded" frameborder="0" allowfullscreen></iframe>

Although there are many other interesting aspects about the
DiagnostiCar, let's focus on the shell system. The expert system
reasoning system used was abductive reasoning which is a kind of
reasoning that tries to find a plausible cause (or a set of them) for a
given fact occurrence [1][2]. Find the causes for a given observation is
exactly what we need for diagnose car problems (the cause of the
observable symptoms).

I choose to use Prolog not only because it's default support to
first-order logic (although some logic limitations on negation [3]), but
it has a nice support for pattern matching and syntax operator
definition (most of the Prolog implementations). From the Prolog
implementations I've picked SWI Prolog due to it's multi-thread support
(necessary for a system running on a multiple user environment),
stability and speed (not as fast as the YAP) [4].

To illustrate the process of development of the expert system we will
restrict our diagnosis to the problems consisting of some oil leaks
problem. Here are some of the knowledge about car diagnosis I will use
as example in the next sections:

-   A crank case, hydraulic oil reservoir and brakes fluid reservoir
    damaged are problems;
-   The crank case damaged provokes leak of engine oil;
-   The hydraulic reservoir damaged provokes leak of hydraulic system
    oil;
-   The brakes fluid reservoir damaged provokes leak of engine oil; 
-   If an oil is leaking it dirts the garage's floor.
-   The brakes fluid is green.
-   The hydraulic oil is red.
-   The engine oil is black or brown.
-   If the engine oil is black, then it needs to be changed.

For the next parts, it's good to know the basic about Prolog. A VERY
good material about Prolog, it's a the book "Building Expert Systems in
Prolog"[5] which guided through the realms of logic programming and the
nasty api's documentations. It is also very pratical and full of
examples to follow. It's also good see the SWI reference to check some
SWI particularities [6].


[1] [Abductive Logic Programming](http://en.wikipedia.org/wiki/Abductive_logic_programming)
[2] [Abducive Reasoning](http://en.wikipedia.org/wiki/Abductive_reasoning)
[3] [Prolog Negation as Failure](http://en.wikipedia.org/wiki/Negation_as_failure)
[4] [Prologs Implementation Comparisons](http://www.david-reitter.com/compling/prolog/compare.html)
[5] ["Building Expert Systems in Prolog"](http://www.amzi.com/ExpertSystemsInProlog/xsipfrtop.htm)
[6] [SWI Manual](http://www.swi-prolog.org/pldoc/refman/)

