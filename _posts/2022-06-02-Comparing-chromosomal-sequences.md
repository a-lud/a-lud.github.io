---
title: Comparing chromosomal sequences
layout: post
post-image: "https://camo.githubusercontent.com/833e373b369f1c221132d729e1cf0128a8cd638fc04c90782ad7d5e9d30867db/68747470733a2f2f7777772e64726f70626f782e636f6d2f732f39766c337973336e6476696d6734632f67726170652d70656163682d636163616f2e706e673f7261773d31"
description: A guide to generate synteny plots using MCscan
tags:
  - Comparative
  - Synteny
  - Guide
---

Comparing genetic sequences is a common task in genomics. There are a range of methods to evaluate
the similarities and differences within and between organisms. Here, I'll demonstrate how to generate
a ribbon plot between two chromosomes.

---

# Introduction

Often we'll want to comapre genetic sequences to identify structural similarities or differences.
Depending on the sequence data you have, there are any number of ways to do this. For instance,
genetic variants can be used to compare sequences at the base-pair level, multiple sequence alignments
can be used to compare collections of sequences, while whole genome alignments can be used to assess
variations in overall genetic architechture. Increasingly more chromosome scale genome assemblies are
becoming available, making whole genome comparisons a logical starting point in comparative analyses.
Here, I'll outline how to compare chromosome scale sequences between two organisms, generating a range
of informative plots along the way.

## Background and software

[`MCScan`][mcscan] is a tool written in python for enabling the comparison of whole genome sequences with
relative ease. Typically, aligning whole chromosomes is a computationally intensive task. `MCscan` gets
around this problem by using gene sequences as anchors to aid alignment. For example, let's say we
have two chromosomal sequences that we know are homologs.

```text
Seq1    <------------------------------------------------------------------------------------>

Seq2    <--------------------------------------------------------------------------->
```

`MCScan` uses genes as anchors to help alignment. Knowing that certain genes are shared between the
two sequences helps locate alignment seeds (i.e. shared regions that can be aligned). Further, as the
clustering of genes is ususally pretty conserved, aligners can use this information to better identify
larger syntenic regions prior to conducting base-pair alignment, greatly simplifying the overall alignment
problem (see below).

```text
seq1    <------------------------------------------------------------------------------------>
genes1          ... ...     ...      ...    ... ... ...         ... ... ... ...         ...
                 |   |       |               |   |   |           |   |   |   |           |
genes2          ... ...     ...             ... ... ...         ... ... ... ...         ...
seq2    |------------------------|         |-----------|       |-----------------------------|
```

In the following sections, I'll demonstrate how to generate a rough gene annotation (if one doesn't exist)
using [`Liftoff`][liftoff], before moving into the use of `MCScan` to generate a range of useful comparative
figures.

# Step 1: Gene annotation

**NOTE**: You can ignore this step if a gene annotation file (GFF3) already exists for your organism of interest.

As stated above, `MCScan` uses gene annotations to find homologous regions between sequences to aid alignment.
If you have a genome assembly without an annotation file, it's possible to make one that'll do the job for
alignment purposes. The tool for the job is `Liftoff`, which lifts over the gene annotation of an evolutionarily
close organism onto your genome of interest. However, there are a few caveats to its use:

- The organism that you lift from needs to be evolutionarily close, otherwise you'll have few genes lift-over.
- The gene models produced on your genoem will be accurate, but are not to be taken as gospel.
- Ensure your genome is of sufficient completeness. Fragmented genomes will have fewer genes lift over due to missing genetic content.
- The output annotation will only have genes found in the reference file! Novel genes will not be annotated.

## Install the software

`Liftoff` is available through `conda` and can be installed as follows.

```bash
conda create -n liftoff -c bioconda liftoff gffread
```

The command above will create a conda environment `liftoff` and install the software `liftoff` and 
`gffread` within it.

## Prep your input files

`Liftoff` requires three files as input:

1. your.genome.fasta        = the genome you want to annotate
2. reference.genome.fasta   = the genome that you want to lift annotations from
3. reference.genome.gff3    = the gene annotations for the reference genome

## Liftoff script

With these files ready to go, you'll want to create a script like the following

```bash
#!/usr/bin/env bash

liftoff \
    your.genome.fasta \
    reference.genome.fasta \
    -g reference.genome.gff3 \
    -o /out/path/your.genome.gff3 \
    -exclude_partial \
    -p <threads> &> "liftoff.log" || exit 1
```

In the script above, we pass the un-annotated genome (`your.genome.fasta`) as the first positional argument.
The second positional argument is the annotated reference (`reference.genome.fasta`) that we're going to
lift annotations from. The third argument (`-g`) is the actual gene annotations for `reference.genome.fasta`
in `GFF3` format, which is described [here][gff3]. We then specify an output file (`-o`) and specify not to
include partial gene models (`-exclude_partial`). This is because we only want complete genes. Finally we
ask for `-p` threads to speed the process up.

If all goes to plan, you should end up with a `GFF3` file containing the genes found in your genome of interest.

## Extract CDS sequences

Following annotation, we then extract the coding sequence (CDS) using [`gffread`][gffread].


```bash
gffread your.genome.gff3 \
    -g your.genome.fasta \
    -x your.genome.fna
```

In the call above, the output file passed to `-y`  will house the coding sequences for the genes that could
be lifted over to our genome of interest.

## MCScan

[mcscan]: https://github.com/tanghaibao/jcvi/wiki/MCscan-(Python-version)
[liftoff]: https://github.com/agshumate/Liftoff
[gff3]: https://learn.gencore.bio.nyu.edu/ngs-file-formats/gff3-format/
[gffread]: http://ccb.jhu.edu/software/stringtie/gff.shtml#gffread