+++
title = "Regex like operators for DCG"
date = "2014-01-07"
tags = ["bnf", "dcg", "ebnf", "latin", "logic", "meta", "nlp", "operator", "prolog", "regex", "natural language"]
+++

Today I was trying to create a simple parser to count syllables in latin
words with Prolog. I usually use DCGs in Prolog for parsing. Their
semantic is very similar to
[BNF](https://en.wikipedia.org/wiki/Backus%E2%80%93Naur_Form). I love
DCGs, but sometimes the verbosity in some cases annoys me. Take the
following example:

```prolog
consonant -->
    "b"; "c"; "d"; "f"; "g"; "h"; "l"; "j"; "k"; "m";
    "n"; "p"; "q"; "r"; "s"; "t"; "v"; "x"; "z".
consonants -->
    [].
consonants -->
    consonant, consonants.

vowel -->
    "a"; "e"; "i"; "o"; "u".
vowels -->
    vowel.
vowels -->
    vowel, vowels.

syllable -->
    vowels.
syllable -->
    consonants, vowels.

syllables(0) -->
    [].
syllables(N) -->
    syllable, syllables(N_1),
    { N is N_1 + 1 }.
```

The vowels and consonant rules were created merely as helpers for the
syllable predicate. That could be reduced if I had regex operators like
+, \* or ?. Although there are modules for using regex in Prolog (
swi-regex ), it is not suitable when using within in DCGs. So I wrote
these regex like operators, with meta DCG predicates, for DCG (like EBNF
operators):

```prolog
% op statements let me use them without parenthesis
:- op(100, xf, *).
:- op(100, xf, +).
:- op(100, xf, ?).

*(_) -->
    [].
*(EXPR) -->
    EXPR, *(EXPR).

+(EXPR) -->
    EXPR.
+(EXPR) -->
    EXPR, +(EXPR).

?(EXPR) -->
    [].
?(EXPR) -->
    EXPR.
```

They allow me to modify the times a given rule will be matched. So, I
can replace this:

```prolog
consonants -->
    [].
consonants -->
    consonant, consonants.

vowels -->
    vowel.
vowels -->
    vowel, vowels.

syllable -->
    vowels.
syllable -->
    consonants, vowels.
```

with a simpler version without intermediate rules (using the operators
definition through a library):

```prolog
syllable -->
    *consonant, +vowel.
```
