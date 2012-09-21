---
layout: documentation
title: Fast and simple word alignment
---
**This document describes how to use the `training/fast_align` tool to create a word alignment for a parallel corpus**.

Internally, `fast_align` implements several simple lexical translation models (slightly improved variants of [IBM Models 1 and 2](http://acl.ldc.upenn.edu/J/J93/J93-2003.pdf)). Under these models, the (conditional) likelihood of a parallel corpus has the general form <span>\\( \prod\_{\mathbf{e},\mathbf{f}} \sum\_{\mathbf{a}} \prod\_i p(a_i \mid \mathbf{f}) \times p(e\_i \mid f\_{a\_i}) \\)</span>, which is very efficient to evaluate. The EM algorithm is used to find model parameters that maximize this likelihood. Then, the single most probable alignment according to the learned parameters is inferred for each sentence pair.

# Format of the parallel corpus
Most `cdec` tools, including `fast_align`, use a simple text format to represent parallel corpora. In this format, each parallel sentence is a single line of text with the two parts separated by a triple pipe (`|||`). Here is an example parallel corpus consisting of three sentences:

    doch jetzt ist der Held gefallen . ||| but now the hero has fallen .
    neue Modelle werden erprobt . ||| new models are being tested .
    doch fehlen uns neue Ressourcen . ||| but we lack new resources .

# Running the aligner

The aligner can be run with the following command:

    ./training/fast_align -i corpus.de-en -d -v > corpus.de-en.fwd_align

The following options are used:

- The `-i corpus.de-en` specifies the input file.
- The `-d` indicates to use a prior on the alignment points that favors alignments that are close to "diagonal".
- The `-v` indicates to infer parameters assuming a symmetric Dirichlet prior on the lexical translation distributions.

The output is written to a file with as many lines as the input file as a sequence of *source-target* pairs, identical to the format used for word alignment files by tools like Moses and Joshua. Here is an example alignment of the above parallel corpus:

    0-0 1-1 2-4 3-2 4-3 5-5 6-6
    0-0 1-1 2-2 2-3 3-4 4-5
    0-0 1-2 2-1 3-3 4-4 5-5

# Running the aligner in reverse mode

You can also run the aligner in *reverse mode* with the `-r` option, which indicates to the aligner to use the *right* sentence as the *source* and the *left* as the *target*.

    ./training/fast_align -r -d -v -i corpus.de-en > corpus.de-en.rev_align

# Symmetrizing forward and reverse alignments

The "forward" and "reverse" alignments can be symmetrized using the `utils/atools` command:

    ./utils/atools -i corpus.de-en.fwd_align -j corpus.de-en.rev_align -c grow-diag-final-and > corpus.de-en.gdfa

Other symmetrization options for use with the `-c` option are `intersect`, `union`, `grow-diag`, and `grow-diag-final`.

# Visualizing alignments

Alignment files can also be visualized with the `utils/atools` command:

    ./utils/atools -i corpus.de-en.gdfa -c display
     0123456
    0*......0
    1.*.....1
    2....*..2
    3..*....3
    4...*...4
    5.....*.5
    6......*6
     0123456
    
    ...

# Other alignment toolkits
The alignment models implemented by `fast_align` are extremely simple and may be inadequate, especially for language pairs that have substantially different word orders. The following tools may be more appropriate in these cases:

- [Giza++](http://code.google.com/p/giza-pp/)
- [Berkeley Aligner](http://code.google.com/p/berkeleyaligner/)
- [PostCAT](http://www.seas.upenn.edu/~strctlrn/CAT/CAT.html)
- [Nile](http://www.isi.edu/~riesa/software/nile/) (requires manual alignments for training)
