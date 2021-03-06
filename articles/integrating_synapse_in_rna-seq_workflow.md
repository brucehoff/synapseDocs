---
title: "Integrating Synapse in your RNA-Seq workflow"
layout: article
category: inpractice
excerpt: Using Synapse in an RNA-Seq workflow to manage files and track processing steps.  
---

# Goals

The goal is to demonstrate how to use Synapse in an RNA-Seq workflow to manage files and track processing steps.

## Getting Raw Data
The first step is to download the data onto the computer where you will be processing it.

The data used in this example is a small RNA-Seq dataset for adrenal and brain tissue generated by the [https://www.ebi.ac.uk/arrayexpress/experiments/E-MTAB-513/](Illumina Body Map) project. A sub-sampled raw dataset has been stored in a public [Synapse project](https://www.synapse.org/#!Synapse:syn2468548/files/).

Here, we access two `fastq` files and a small region of chr19 (300000-350000 Mb) of the hg19 reference genome directly by their Synapse identifiers and download them to our local computer.

{% highlight bash %}
# Get brain fastq file
synapse get syn2468554

# Get adrenal fastq file
synapse get syn2468552
{% endhighlight %}


## Map the Raw Reads
Now that we have the data, you can then use the alignment tool of choice to map these reads. We will use [STAR](http://bioinformatics.oxfordjournals.org/content/early/2012/10/25/bioinformatics.bts635) to map the reads

## Setting Up the Local Environment

{% highlight bash %}
mkdir demo-rnaseq-workflow

#dir to store the STAR genome index for the reference genome
mkdir ref-genome

{% endhighlight %}

## Creating a Genome Index

{% highlight bash %}

# the reference genome
# downloads as hg19_chr19_subregion.fasta
synapse get --downloadLocation ref-genome/ syn2468557

#create a STAR genome index
STAR --runMode genomeGenerate --genomeDir ref_genome --genomeFastaFiles ref-genome/hg19\_chr19\_subregion.fasta

{% endhighlight %}

## Map Adrenal and Brain Tissue Reads

{% highlight bash %}
star --runThreadN 1 --genomeDir ref-genome/ --outFileNamePrefix brain --outSAMunmapped Within --readFilesIn brain.fastq

star --runThreadN 1 --genomeDir ref-genome/ --outFileNamePrefix adrenal --outSAMunmapped Within --readFilesIn adrenal.fastq
{% endhighlight %}

Let's store the results and provenance in Synapse.

{% highlight bash %}
# create a project
synapse create Project --name demo-rnaseq-workflow

# Use Synapse ID reported from the above command to use as the parent ID
synapse store brain.sam --parentId syn234567890 --used brain.fastq ref-genome/hg19\_chr19\_subregion.fasta

synapse store adrenal.sam --parentId syn234567890 --used adrenal.fastq ref-genome/hg19\_chr19\_subregion.fasta
{% endhighlight %}
