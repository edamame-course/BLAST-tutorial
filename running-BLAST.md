---
layout: page
title: Running BLAST from the command line to identify environmental sequences
comments: true
date: 2015-06-25 14:00:00
---
# Running BLAST from the command line to identify environmental sequences
Authored by Jin Choi for EDAMAME2015, based on a previous tutorial.
[EDAMAME-2015 wiki](https://github.com/edamame-course/2015-tutorials/wiki)

***
EDAMAME tutorials have a CC-BY [license](https://github.com/edamame-course/2015-tutorials/blob/master/LICENSE.md). _Share, adapt, and attribute please!_
***

## Overarching Goal
* This tutorial will contribute towards an understanding of BLAST running locally

## Learning Objectives
* Build local DB
* Blast query into local DB

***

# Introduction
OK, your first introduction to the use and abuse of command line tools is... BLAST! That's right, the [Basic Local Alignment Search Tool](http://en.wikipedia.org/wiki/BLAST)!

You can use the NCBI BLAST Web page to do individual searches. That's great when you're looking for just a few things. However, you might
want to BLAST a lot of queries, or you might be interested in searching your own data using BLAST. 

Today we'll automate batch searches at the command line on your own computer. This is a technique that works well for small-to-medium sized sequencing data sets. The various database (nr, nt) are getting big enough that it's reasonably time consuming to search them on your own, although of course you can do it if you want – you might just have to wait a while for things to finish.

## Motiviation: Why I want to run BLAST locally?
[Here are the slides](https://github.com/edamame-course/2015-tutorials/blob/master/presentations_resources/BLAST.pdf)
 
1. BLAST on my own database
2. Automation: run on server, downstream analysis, can be part of other program
3. Speed

In this tutorial we'll search a representative-otus-database using my-gene-of-interest query.

Before I forget, let me say that there are a lot of tips and tricks for working at the UNIX command line that I'm going to show you, so even if you've used command line BLAST before, you should skim along.

First, let's check and see if we have BLAST.

```
which blastn
```

If you have BLAST installed and in your path (BLAST may be installed but not in your path), it will give you something like this:

```
/usr/bin/blastn
```

If you don't have BLAST, then you will need to install it.

## Step 1: Installing BLAST
To install the BLAST software, type this:

```
sudo apt-get install ncbi-blast+
```

- `sudo` stands for "super user do". It's giving you the administrative privledges to install things.
- `apt-get install` is how we install things on Ubuntu. Programmers package software so it can be installed this way, and a list of available packages is maintained.
- `ncbi-blast+` is the name of the software we want to install


## Step 2: Download the databases
Now, we can't run BLAST without downloading the databases. For this you'll need the DB such as nt.  This, like a lot of NCBI databases is huge, so I don't suggest putting this on your laptop unless you have a lot of room.  It's best on a larger computer (HPCC, Amazon machine, that you have access to).  I wouldn't install this database unless you know you have room on your computer. Let's download small database for this tutorial.

Use curl to retrieve the database and file that we will use today:

```
curl -O https://s3.amazonaws.com/edamame/Blast_Tutorial.tar.gz
```

unzip file.

```
tar -zxvf Blast_Tutorial.tar.gz
```

Let's navigate into the directory

```
cd Blast_Tutorial
```

Now you've got these files. How big are they?

```
ls -l
```

We can also do 

```
ls -lh
```

This shows us the file sizes in 'human readable' format.

What are each of these files?

MyQuery.txt : This is a set of 16S sequences and will be used for a query

Refsoil16s.fa : This is 16S data from soil

rep_set.fna : This is a file of short 16S sequences that we will use as a reference database

rep_set_sub.fna : These a few sequences from rep_set.fna that we will use for a query

Database files are large files, but let's use something small for this tutorial.

So, now we've got the database files, but BLAST requires that each subject database be preformatted for use; this is a way of speeding up certain types of searches. To do this, we have to format the database.  You should do:

```
makeblastdb -in rep_set.fna -dbtype nucl -out My16sAmplicon
```

The -in parameter gives the name of the database, the -out parameter says "save the results", and the -dbtype parameter says "what type of the database". For DNA, you'd want to use '-dbtype nucl'. FYI, for protein, '-dbtype prot'.

[makeblastdb manual](http://nebc.nerc.ac.uk/bioinformatics/documentation/blast+/user_manual.pdf)

Let's see how a BLAST database looks

```
ls
```

You may notice that there are 3 files that were generated.

My16sAmplicon.nhr, My16sAmplicon.nin, My16sAmplicon.nsq

BLAST uses this 3 files to search efficiently.

Just a reminder:

1. UNIX generally doesn't care what the file is called, so '.fna' and '.fa' are all the same to it. 
2. UNIX utilities work well with text files, and almost everything you'll encounter is a basic text file. This is different from Windows and Mac, where more complicated formats are used that can't be as easily dealt with on UNIX.

## Step 3: Run BLAST
Now try a BLAST: You need a file that have your query in. Here is what your query looks like.

```
less MyQuery.txt
```

We have 4 bacteria and 5 fungi.

Now BLAST. We use blastn because we will search a nucleotide database using a nucleotide query.

```
blastn -db My16sAmplicon -query MyQuery.txt
```

You can do three things at this point.

First, you can scroll up in your terminal window to look at the output.  

Second, you can save the output to a file:

```
blastn -db My16sAmplicon -query MyQuery.txt -out out.txt
```

and then use 'less' to look at it:

```
less out.txt
```

and third, you can pipe it directly to less, by which I mean you can send all of the output to the 'less' command without saving it to a file:

```
blastn -db My16sAmplicon -query MyQuery.txt | less
```

Sometimes tabular form (output format) is useful. To get the result in tabular form,

```
blastn -db My16sAmplicon -query MyQuery.txt -out outtabular.txt -outfmt 6
```

## Excercise

Let's try to search the Reference Soil database using the 16S amplicon query, rep_set_sub.fna

### 1. Make Reference Soil database

Hint: Here is the fasta format of reference soil. : RefSoil16s.fa

### 2. Search using a sample 16s amplicon query

Hint: You can use this sample query: rep_set_sub.fna


## Different BLAST options
BLAST has lots and lots and lots of options. Run 'blastn' by itself to see what they are. Some of the most useful ones are `-evalue`.

If you want to search protein, use 
- 'blastp' - search protein databases using a protein query
- 'blastx' - search protein databases using a translated nucleotide query
- 'tblastn' - search translated nucleotide databases using a protein query
- 'tblastx' - search translated nucleotide databases using a translated nucleotide query

alignment view options:

0 = pairwise,

1 = query-anchored showing identities,

2 = query-anchored no identities,

3 = flat query-anchored, show identities,

4 = flat query-anchored, no identities,

5 = XML Blast output,

6 = tabular,

7 = tabular with comment lines,

8 = Text ASN.1,

9 = Binary ASN.1

10 = Comma-separated values

11 = BLAST archive format (ASN.1)

## Other BLAST uses

In this tutorial we've used 16S sequences, but it's also very useful with shotgun metagenomic databases. You can take a gene, or set of genes, that you're interested in and BLAST the database to see if it's present and in what abundance. In these cases you would want to create a protein database and do a `tblastx` search.

```
makeblastdb -in metagenomic-data.fa -dbtype prot -out metagenomicDB
tblastx -db metagenomicDB -query MyQuery.txt -out out.txt
```

***
##Help and other resources
* [Manual of Blast+](http://www.ncbi.nlm.nih.gov/books/NBK279690/)

* FTP link for RefSeq DB : ftp://ftp.ncbi.nlm.nih.gov/refseq/release/

* [RefSeq site](http://www.ncbi.nlm.nih.gov/refseq/)

* [HMP site](http://www.hmpdacc.org)

* [Stript for get the best hit from BLAST search in tabular form](https://github.com/metajinomics/Get_best_hit_blast_tabular)

-----------------------------------------------
-----------------------------------------------
