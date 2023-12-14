# 2022-pangenome-graphs-intro

Practical session for the Introduction to Pangenomics class given at Sorbonne University in Paris, December 1st 2022.

From the lecture given at the CGSI, UCLA, July 22nd 2022 by Rayan Chiki. 

Rayan's Slides: https://docs.google.com/presentation/d/1KBckpDnKlDZpvRktt_RSAxUCTcOA9n83ERkDqNo3JKs/edit?usp=sharing

## Rationale

We'll see how some common pangenome graphs can be constructed in practice, on a simple example. 

## Data

Data is two E. coli genomes, present in the `data/` folder.

    mkdir ouput_graphs && cd output_graphs


## de Bruijn graph

Create compacted dBG with k=31 using [bcalm2](https://github.com/GATB/bcalm):
Read bcalm2 instructions and try to run it. Which parameters are needed? 

    ../tools/bcalm ...

Convert bcalm2's FASTA (with edge information) to GFA:

    ../tools/convertToGFA.py two_ecolis.unitigs.fa two_ecolis.unitigs.gfa [k-mer length]

Simplify graph by removing small bubbles using [gfatools](https://github.com/lh3/gfatools):

    ../tools/gfatools asm -b [length of the bubble] -u [graph.gfa] > [graph.simplified.gfa]

Trying a larger k value (k=127, k=319) [the binary is different for such large k-mers]:

    ../tools/bcalm-k320 ...

Get the simplified GFA as before.

Now visualize the three (simplified) graphs. 
What kind of information can we obtain by just visualizing the pangenome of these 2 strains? 
As the k-mer lenght grows, what does it change in these graphs?

## Variation graph

(This is not the pggb standard pipeline, but for our goal this is good enough).
Create a raw pangenome graph using minimap2 + [seqwish](https://github.com/ekg/seqwish):

    minimap2 ... > two_ecolis.paf
    ../tools/seqwish  ...

Simplify using [smoothxg](https://github.com/pangenome/smoothxg) (Takes a while!):

    ./tools/smoothxg ...

Further simplify by removing bubbles:

    gfatools asm -b 1000 -u [graph.gfa] > [graph.bu.gfa]
    gfatools asm -b 1000 -u [graph.smooth.gfa] > [graph.smooth.bu.gfa]

As specified at the beginning, it would be better to just run `pggb` instead of `seqwish`+`smoothxg`, to automatically tweaks the parameters of `smoothxg`.

Now visualize the graph. Is it different from dbgs?


## Minigraph

Construct by aligning o157 on the K12 reference using [minigraph](https://github.com/lh3/minigraph):

    ../tools/minigraph ...

Now visualize the graph. Is it different from dbgs and seqwish vg?

## Minimizer-space de Bruijn graphs

Construct with k=10, d=0.001:

    ../tools/rust-mdbg ...

Compact the (minimizer-space) de Bruijn graph:

    ../tools/gfatools asm -u [graph.gfa] > [graph.u.gfa]

Reincorporate bases in mdBG:

    ../tools/to_basespace -g [graph.u.gfa] -s [graph]

Let's have a look at the graph. Can you distinguish the 2 genomes?

## DISCUSSION

Let's discuss about what these graph show.
Did you expect these results from the theory class?
Does pangenomics seems usefult to you?

## Commands for demo

Change prompt:

    export PS1="\[\e[0;36m\]pangenomics:\W\[\e[0m\]$ "

BCALM 2

    ../tools/bcalm -in ../data/two_ecolis.fasta -kmer-size 31 -abundance-min 1

    ../tools/bcalm-k320 -in ../data/two_ecolis.fasta -kmer-size 319 -abundance-min 1

SEQWISH

    minimap2 -c -X ../data/two_ecolis.fasta ../data/two_ecolis.fasta > two_ecolis.paf
    ../tools/seqwish  -s ../data/two_ecolis.fasta -p two_ecolis.paf -g two_ecolis.gfa

MINIGRAPH

    ../tools/minigraph -cxggs -t8 ../data/k12.fasta ../data/o157.fasta > o157_on_k12.gfa

MDBG 
../tools/rust-mdbg -k 10 -d 0.001 ../data/two_ecolis.fasta --reference --minabund 1
