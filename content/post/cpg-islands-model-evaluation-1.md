+++
title = "CpG Islands (1) - Problem Motivation & Definitions"
date = "2011-05-29"
+++
This semester I'm attending the course
[Processing of Sequences and Methods and Algorithm in Computational Biology](http://www.cin.ufpe.br/%7Eigcf/wiki/pmwiki.php?n=Disciplinas.ProcessamentoDeCadeias2011)
(basically DNA and proteins). One of the focus of it is the use of the
Hidden Markov Models to solve many of it's problems. One well studied
problems is how to find codifying sequences (snippets that can be
translated into proteins) in a given sequence of DNA, which has both
codifying and non codifying regions. One of the most used techniques is
called CpG islands.

In a random DNA sequence we have a lower frequency of two ((G)uanine and
(C)ytosine) than the other two nucleotide types ((A)denine and
(T)hymine) due to a phenomenon called
[DNA methylation](https://secure.wikimedia.org/wikipedia/en/wiki/Dna_methylation).
Due to this methylation, it's more probable to occur a deletion of a C
and G nucleotides than with A or T

![DNA double helix structure](/img/dna_model.jpg)

In the species evolution, occurs that genes which are essential for it's
survival cannot mutate much, otherwise the organism can't live and
reproduce to transmit it's gene to their descendants. Because of this,
is very unlikely that even with lots of generations, those genes doesn't
suffer much alteration. It's important to notice that non codifying
regions can mutate without affecting the organism because it doesn't
affect a vital part of the genetic code.

With these two facts in hand we can see that mutation of the DNA can
occur more frequently on non codifying regions. As the methylation of C
is a very common mutation cause, we can observe that non codifying
regions will have less C and G due to the methylation which it doesn't
interfere (much) in the organism. The opposite occurs with codifying
regions because it has less frequent mutations.

![CpG island around an important human DNA regulation site](/img/exon1_CpG.gif)

With this information in our hands we can use it to discover regions
which are likely to be a codifying region. The sites which contains
greater concentrations of C-phosphate-G (called CpG islands) are a
strong indicator that it hasn't suffered much mutations through the
generations. So it's very probable to be an area of interest for
biologists. I'll describe in the next posts how can we identify these
CpG using HMM and the ghmm python library.

