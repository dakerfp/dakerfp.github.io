+++
title = "Protein Profile with HMM"
date = "2011-07-07"
tags = ["machine learning", "bio", "classification", "ghmm", "hmm", "markov", "protein", "profile", "likelihood", "python"]
+++

Profinling is the action of summarizing a set of data in a smaller mathematical
model. One of the practical usages of profiling techniques is the
classification of sequences. With a data set profile, you may calculate
the distance of an instance to the model, and classify the instance
trough this value.

Profiling proteins is a more specific case of profiling sequences. As we
know from the previous post about [Hidden Markov
Models](http://codecereal.blogspot.com/2011/05/hidden-markov-models.html)
(HMMs) is a very robust mathematical model to represent
probabilistically sequences. This post will detail how to profile sets
of proteins with a HMM model.

The HMM model for this problem must be a HMM which when we calculate a
probability of generation of a protein by the model, must give higher
probabilities for proteins similar for the ones used to create the
profile against the others.

To build the model, I've started with a [multiple sequence
alignment](https://secure.wikimedia.org/wikipedia/en/wiki/Multiple_sequence_alignment)
to see how is the structure of the sequences. That means that our HMM
will be a probabilistic representation of a multiple alignment. With the
alignment we see that some columns are complete and some are almost, and
the others have few data. These most common matches can be used as a
common matches for our model and the deletions and insertions can be
modelled as other states. Here is an example of a few sequences aligned
and the core columns of the alignment (marked with an \*):

`AC----ATGTCAACTATCACAC--AGCAGA---ATCACCG--ATC`

With this sample of alignment we have examples of insertions and
deletions between the core alignments, which may have a state on the HMM
to represent them. Note that insertions may occur on arbitrary times
between the matching states. And the deletion states always replaces
some matching. One possible HMM template for building the model is
presented in the following picture:

![HMM protein profile model](/img/profile_hmm.png)

Each matching state (M~j~) is related to the matching on the j^th^ core
alignment. The same applies for deletion state (D~j~). The insertion is
slightly different, because it represents the insertions between the
core alignments, that's why it has one extra state, and this enable to
represent states before the first and after the last alignment.

Our model will be like the one in the picture with the same length as
the core alignments of a multiple alignment for a given set of
sequences. However we should use [maximum likelihood](https://secure.wikimedia.org/wikipedia/en/wiki/Maximum_likelihood)
to estimate the transitions and emission for each state. The easiest way
to count the total emissions/transitions, is to thread each sequence to
be profiled in the model. For each symbol in the aligned sequence you
must check if the symbol is in the core alignment. If it is, then
increment the count of that state to the next match state, otherwise,
you go to the insertion state. If it is a deletion, go to the deletion
state and increment the transition. Finally, to calculate the
probability for the transitions, just divide the count of each
transition and divide it by all the states leaving the same state. It's
important to notice that we have a stopping state, and this is a special
state that has no transitions.

Note that is important to initialize the model with pseudo counts at
each possible transitions. Adding this pseudo count, we let our model
less rigid and avoid overfitting to the train set. Otherwise we could
have some zero probabilities for some sequences.

To create the emissions probabilities, at each match state, you also
have to count which symbol was emitted and then increment them. To
calculate the probability, just divide it by the total symbols matched
in the threading process. 

The similar process could be done with the insertion. However, the
insertion states are characterized by having a low occurrence rate and
this may lead us directly to a overfitting problem because the number of
emissions must be too small. To avoid this we should use the background
distribution as the emissions probabilities of each insertion state. The
background distribution is the probability of the occurrence of a given
amino acid in the entire protein set. To calculate this, count each
amino acid type for all the train set sequence and then divided by the
total count.

For the deletion states, it's important to notice that it is a silent
state. It doesn't emit any symbol at all. To signalize it in ghmm, just
let all the emissions probabilities with 0. Note that the end state is
also a silent state once we have no emission associated to it.

**ALERT**: the loglikelihood is the only function in the ghmm library which
handles silent states correctly.

At this point, the model is ready for use. However we have a problem of
how to classify a new protein sequence. A threshold could be used to
divide the two classes. However, the most common alternative is to use a
null model to compare with it. The null model is a model which aims
represents any protein with similar probability as any other. With this
two models we could take a sequence and compare if it is more similar to
a general definition of a protein or to a specific family. This model
should model a sequence with average size equal to the aligned sequences
being handled, and should emit any kind of symbol at each position. A
common alternative for creating the null model, is using a single
insertion state, which goes to a stopping state with probability of 1
divided by the average length of  sequences in the train set. For the
emissions probability, we should use the background distribution,
because this is related to the general amino acid distribution. At the
end, the model should be like this:

![Null Model](/img/null_model.png)
  
For testing the proposed model, I used a set of 100
[globin](https://secure.wikimedia.org/wikipedia/en/wiki/Globin) proteins
from the [NCBI](http://www.ncbi.nlm.nih.gov/) protein repository as a
train set to build a profile model, and used
[ghmm](http://ghmm.sourceforge.net/) to build and test the model.

To test if our model corresponds to our expectations, 100 globins
different from the ones in the trains set were used with 1800 other
random proteins with similar length. The loglikelihood function from the
ghmm library to calculate the similarity index. The classification of
the globins versus non globins, was a comparison between the likelihood
of the protein with the globins profile hmm, and the null model. This
test gave us 100% of accuracy! To display this result graph, each
sequence was pointed out in a graph where the horizontal axis displays
the length of the sequence and the vertical the log of globins / null
models likelihood (or globins - null model loglikelihood). The globins
are plotted in red an the others in blue. Proteins over the zero line
are classified as globins, and below means they aren't globins. The
graphs show us a clear distinction between the classes, and that our
model is very precise for this problem.

![Length x log(Globin's Profile Model Likelihood/Null Model Likelihood)
  
