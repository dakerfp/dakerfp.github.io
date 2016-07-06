+++
title = "DiagnostiCar (3) - Question & Answer Interface"
date = "2010-04-11"
+++
This post aims to explain how to create an interactive shell so that
users may answer questions when the reasoning system need user
information. The reasoning engine will access the ask engine by an
predicate (I will call it `ask`). An naive implementation of the
`ask` predicate could be:

```prolog
ask(X) :-
    write(X), write(' is true?'), nl,
    read(yes).
```

It works, but askable predicates will be asked every time the reasoner
will need an information, even if it was previously answered. Another
specific problem from DiagnostiCar constraints, was how to separate
which information is from which user, considering we're at an web
application environment. The simultaneous users access to the reasoning
engine couldn't simply be locked while an user was giving an answer
back, so we must find a smarter way to ask.

To remember info in Prolog, we will need to use dynamic relations to
store the asked questions, for those who don't know about dynamic
relations I will give a short explanation. If you want to know more,
read the ["Building Expert Systems in Prolog"](http://www.amzi.com/ExpertSystemsInProlog/xsipfrtop.htm)

Dynamic relations works similarly as a relational table in a
[relational database](http://www.swi-prolog.org/pldoc/man?section=db),
but without a type scheme (although it's strongly recommended to make a
convention). The informations required to create a dynamic relation are
just it's name and arity. To declare a dynamic predicate you just need
to run dynamic(Predicate). Once the relation is a dynamic one, you can
store information (`assert/1` or `asserta/1`) or delete information
(use `retract/1` and `retractall/1`) eg.: `assert(foo(bar))`. To
query a dynamic predicate you just have to call it (it allows
unification!) eg.: `?- foo(bar). -> true`. I made the following
basic but illustrative example about Prolog DB:
 
```prolog
create_account(Id, Ammount) :-
    assert(account(Id, Ammount)).
         
deposit(Id, Ammount) :-
    retract(account(Id, PrevAmmount)),
    NewAmmount is Ammount + PrevAmmount,
    assert(account(Id, NewAmmount)).
         
withdraw(Id, Ammount) :-
    account(Id, PrevAmmount),
    PrevAmmount >= Ammount,
    retract(account(Id, PrevAmmount)),
    NewAmmount is PrevAmmount - Ammount,
    assert(account(Id, NewAmmount)).
```

To test the predicates, do some queries:

```prolog
?- create_account(1, 1000).
true.

?- deposit(1, 200).
true.

?- account(Id, Ammount).
Id = 1,
Ammount = 1200.

?- withdraw(1, 400).
true.

?- withdraw(1, 20000).
false.

?- account(Id, Ammount).
Id = 1,
Ammount = 800.
```

The best solution found, to solve the concurrence and the question
tracking, was to create predicates to read and write, so the webapp
could obtain such information from the expert system. To avoid repeating
the same questions, we will have to maintain a table with questions
made, one with the answered questions and another to keep the answers
from the users. Note that each relation needs to keep an user id to know
from which user each information belongs to. The expert system question
interface is the following:

```prolog
% asked: UserId | Question
:- dynamic asked/2.
% asked: UserId | Question
:- dynamic askfor/2.
% known: UserId | Question | Fact
:- dynamic known/3.

% Predicate used by the expert system to ask.
ask(Uid, Question, Answer) :-
    known(Uid, Question, Answer).
ask(Uid, Question, _ ) :-
    not(asked(Uid,Question)),
    not(askfor(Uid,Question)),
    assert(askfor(Uid, Question)),
    fail.

% Predicate used by the web app to input users answers for a given qeustion.
answer(Uid, Question, [Answer | Answers]) :-
    assert(known(Uid, Question, Answer)),
    answer(Uid, Question, Answers).
answer(Uid, Question, []) :-
    assert(asked(Uid, Question)),
    retractall(askfor(Uid,Question)).

% Routine called by the webapp to clean the data from a user whose 
% session was expired.
cleanup(Uid) :-
    retractall(askfor(Uid,_)),
    retractall(known(Uid,_,_)),
    retractall(asked(Uid,_)).
```

Not that hard! I will try to tell all about the abductive reasoning on
the next post.
