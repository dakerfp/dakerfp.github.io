+++
title = "CpG Islands (3) - Model Evaluation"
date = "2011-06-05"
tags = ["ai", "baum wlch", "bio", "cpg", "dna", "ghmm", "hmm", "markov", "matplotlib", "maximum likelyhood", "python"]
+++
Following the [post](http://codecereal.blogspot.com/2011/06/cpg-islands-2.html)
we've built a Hidden Markov Model (HMM) for the [CpG islands problem](http://codecereal.blogspot.com/2011/05/cpg-islands-1.html), using the training set.
Now we should evaluate if our model is adequate to predict things about CpG islands.
For evaluate we may use a tagged
sequence and see if the HMM we built can predict the islands and what is
it's accuracy.

Using the viterbi path and a tagged sequence (out ouf the training set),
enable us to compare if the estimative given by the model is coherent
with the real data. Using the HMM and the training and test sets of the
last post, we can compare the results using a [confusion
matrix](http://en.wikipedia.org/wiki/Confusion_matrix). For example, at
the following snippet of the viterbi path paired with the tagged
sequence:

```
[ ... 0,0,0,0,0,1,1,1,1,1, ... ]
[ ... a t t g t c C G A C  ... ]
```
 
 We may calculate the following confusion matrix:

```
+--------------------------------------+ 
|            |  Island    | Non Island |
+------------+------------+------------+
| Island     |     4      |      0     |
+------------+------------+------------+
| Non Island |     1      |      5     |
+------------+------------+------------+
```

Note that the bigger are the main diagonal values, the more accurate our
model is. Making this matrix with the experiment made (using
[HMM with maximum likelihood](http://codecereal.blogspot.com/2011/06/cpg-islands-2.html))
I got the following matrix:

```
+--------------------------------------+ 
|            |  Island    | Non Island |
+------------+------------+------------+
| Island     |   328815   |    61411   |
+------------+------------+------------+
| Non Island |  1381515   |   1889819  |
+------------+------------+------------+
```

We may see that a good amount of islands was correctly predicted (84,26%
of the islands), but 57,76% of the non islands regions was classified as
islands, which is a bad indicator. A experiment with the HMM after
running the Baum Welch was also evaluated. The results was better from
the obtained by the maximum likelihood when predicting islands (96,56%
of accuracy). But also obtained a higher miss rate (70,70%). Here is the
confusion matrix for this HMM:

```
+--------------------------------------+ 
|            |  Island    | Non Island |
+------------+------------+------------+
| Island     |   376839   |    13387   |
+------------+------------+------------+
| Non Island |  2313138   |   958196   |
+------------+------------+------------+
```

![Matching of Real Sequence, Viterbi and Posterior results](/img/cpgs_labels.png)

The result obtained was not very accurate. It may happened because we
had few data to train and to evaluate our model. We could also build a
HMM with other indicators that identify the CpG islands for model it
better. Remember we simplified the CpG problem to the frequencies of
occurring Cs and Gs, but a better model could also use the CG pairs.
Using a more complicate HMM we could have more than 2 states and then
associate a set of states to the CpG and non CpG islands sites. This
would allow to use better the posterior result to classify the sequence.
But I keep this as future work.
