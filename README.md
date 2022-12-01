# 2022-pangenome-graphs-intro

Practical session for the Introduction to Pangenomics class given at Sorbonne University in Paris, December 1st 2022.

Modification by Francesco Andreace of the lecture given at the CGSI, UCLA, July 22nd 2022 by Rayan Chiki. 

Francesco's slides: .

Rayan's Slides: https://docs.google.com/presentation/d/1KBckpDnKlDZpvRktt_RSAxUCTcOA9n83ERkDqNo3JKs/edit?usp=sharing

## Rationale

We'll see how some common pangenome graphs can be constructed in practice, on a simple example. 

## Data

Data is two E. coli genomes, present in the `data/` folder.

## de Bruijn graph

Create compacted dBG with k=31 using [bcalm2](https://github.com/GATB/bcalm):

    ../tools/bcalm -in ../data/two_ecolis.fasta -kmer-size 31 -abundance-min 1

Convert bcalm2's FASTA (with edge information) to GFA:

    ../tools/convertToGFA.py two_ecolis.unitigs.fa two_ecolis.unitigs.gfa 31

Simplify graph by removing small bubbles using [gfatools](https://github.com/lh3/gfatools):

    ../tools/gfatools asm -b 100 -u two_ecolis.unitigs.gfa > two_ecolis.unitigs.bu.gfa

Trying a larger k value (k=127, k=319):

    ../tools/bcalm-k320 -in ../data/two_ecolis.fasta -kmer-size SIZE -abundance-min 1

Now visualize the three graphs. 
What kind of information can we obtain by just visualizing the pangenome of these 2 strains? 
As the k-mer lenght grows, what does it change in these graphs?

## Variation graph

(This is not the pggb standard pipeline, but for our goal this is good enough).
Create a raw pangenome graph using minimap2 + [seqwish](https://github.com/ekg/seqwish):

    minimap2 -c -X ../data/two_ecolis.fasta ../data/two_ecolis.fasta > two_ecolis.paf
    ../tools/seqwish  -s ../data/two_ecolis.fasta -p two_ecolis.paf -g two_ecolis.gfa

Simplify using [smoothxg](https://github.com/pangenome/smoothxg) (Takes a while!):

    ./tools/smoothxg -g two_ecolis.gfa -o two_ecolis.smooth.gfa

Further simplify by removing bubbles:

    gfatools asm -b 1000 -u two_ecolis.gfa > two_ecolis.bu.gfa
    gfatools asm -b 1000 -u two_ecolis.smooth.gfa > two_ecolis.smooth.bu.gfa

As specified at the beginninh, it would be better to just run `pggb` instead of `seqwish`+`smoothxg`, to automatically tweaks the parameters of `smoothxg`.

Now visualize the graph. Is it different from dbgs?


## Minigraph

Construct by aligning o157 on the K12 reference using [minigraph](https://github.com/lh3/minigraph):

    ../tools/minigraph -cxggs -t8 ../data/k12.fasta ../data/o157.fasta > o157_on_k12.gfa

Now visualize the graph. Is it different from dbgs and seqwish vg?

## Minimizer-space de Bruijn graphs

Construct with k=10, d=0.001:

    ../tools/rust-mdbg -k 10 -d 0.001 ../data/two_ecolis.fasta --reference --minabund 1

Compact the (minimizer-space) de Bruijn graph:

    ../tools/gfatools asm -u graph-k10-d0.001-l12.gfa > graph-k10-d0.001-l12.u.gfa

Reincorporate bases in mdBG:

    ../tools/to_basespace -g graph-k10-d0.001-l12.u.gfa -s graph-k10-d0.001-l12

Let's have a look at the graph. Can you distinguish the 2 genomes?

## DISCUSSION

Let's discuss about what these graph show.
Did you expect these results from the theory class?
Does pangenomics seems usefult to you?

## Commands for demo

Change prompt:

    export PS1="\[\e[0;36m\]pangenomics:\W\[\e[0m\]$ "
