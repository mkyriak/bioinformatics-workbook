---
title: "Creating a NextFlow workflow"
layout: single
author: Andrew Severin
author_profile: true
header:
  overlay_color: "444444"
  overlay_image: /assets/images/dna.jpg
---

## Learning Objectives

1. Introduction
2. nextflow setup
3. nextflow params
4. nextflow config file
5. nextflow process
6. nextflow channels


## Introduction

Presumably, you are here because you want to take your bash scripts and put them into a functioning workflow and have all the execution taken care of for you.  You have come to the right place.  Nextflow is great but if you are a biologist turned informatician, the groovy language might trip you up a little in many of the great documentation materials that can be found here:

* [Nextflow getting started](https://www.nextflow.io/docs/latest/getstarted.html)

This is a great resource, but it assumes you have had some experience with object oriented programming or even some background in groovy/Java. You will run across methods that aren't part of nextflow but part of the groovy language like [`.trim, .flatten, and the word "it"`].  At first this was hard to separate out when we just wanted to know how to do X.  While they do have examples in their [nextflow patterns section](https://github.com/nextflow-io/patterns), it isn't quite sufficient yet to quickly learn to create a nextflow workflow.  This tutorial is aimed to bridge this gap.


## A practical example

The goal of this tutorial is to introduce you to the concepts of nextflow by building a practical example.  We will journey through the process of making a simple blast workflow that can take in a fasta file as a query and run blast on it. We will then keep extending this example to showcase different features in nextflow that are useful in building a dynamic workflow.

## Prerequisites

This tutorial assumes that you are familiar with bash scripting and [how to run blast locally](https://bioinformaticsworkbook.org/dataAnalysis/blast/blastExample.html#gsc.tab=0).


## Nextflow setup


### Create a github repo

Let's start by setting up our folder.  I usually do this by first creating a github repo and then cloning it so that I can use version control.  I am going to create this in the `isugifNF` organization that I am apart of and name it `tutorial`. You should make a repo on your own account.

![Create New Github Repo](assets/CreateGithubRepo.png#1)

Then pull it to my local machine. Do not pull this repo as it will download the entire finished tutorial.

```
git clone git@github.com:isugifNF/tutorial.git
```

### Files

Every nextflow workflow requires two main files.

* main.nf
  * The main.nf file contains the main nextflow script that calls the processes.
  * It doesn't have to be named main.nf but that is standard practice.
* nextflow.config
  * The config file contains default parameters to use in the nextflow pipeline

```
touch main.nf nextflow.config
```

Your folder should now contain the following

```
ls
README.md       main.nf         nextflow.config
```

## Lesson 1: Nextflow params

Let's create a simple program that has a parameter to set our query input file that we will use for the BLAST program.

```
#! /usr/bin/env nextflow

blastdb="myBlastDatabase"
params.query="file.fasta"

println "I will BLAST $params.query against $blastdb"
```

#### Let's dissect it line by line.

The first line is required for all nextflow programs  

```
#! /usr/bin/env nextflow
```

The second line of code sets a variable inside the nextflow script.

```
blastdb="myBlastDatabase"
```

The third line of code sets a pipeline parameter that can be set at the command line, which I will show you in just a minute. If you want to make a variable a pipeline parameter just prepend the variable with `params.`

```
params.query="file.fasta"
```

The last line is a simple print statement that uses both a nextflow variable and a pipeline parameter.

```
println "I will BLAST $params.query against $blastdb"
```

Go ahead and run it.

```
nextflow run main.nf
```

Output should look like this

```
N E X T F L O W  ~  version 20.07.1
Launching `main.nf` [confident_williams] - revision: f407a6b0e1
I will BLAST file.fasta against myBlastDatabase
```

You will also have the default `work/` folder that will appear but will be empty as we didn't do anything but print something to standard out.

#### Exercises for comprehension

1. Pipeline parameters can be set on the command line.

  ```
  nextflow run main.nf --query "newQuery.fasta"
  ```

  Output:

  ```
  N E X T F L O W  ~  version 20.07.1
  Launching `main.nf` [extravagant_pasteur] - revision: f407a6b0e1
  I will BLAST newQuery.fasta against myBlastDatabase
  ```
2. Nextflow script variables cannot be set at the command line

  ```
  nextflow run main.nf --blastdb "NewBlastDB"
  ```

  Output:
  ```
  N E X T F L O W  ~  version 20.07.1
  Launching `main.nf` [loving_borg] - revision: f407a6b0e1
  I will BLAST file.fasta against myBlastDatabase
  ```
3. Try other --param.query inputs on your own.

  ```
  nextflow run main.nf --params.query "changethistext"
  ```

#### Github saving and ignoring

```
git add main.nf nextflow.config
git commit -c "started a new nextflow project!"
git push origin master
```

Add the following to your .gitignore file

```
.nextflow*
work
out_dir
```

Then save it to your repo

```
git add .gitignore
git commit -c "added .gitignore"
git push origin master
```

## Lesson 2: Nextflow Config file

The nextflow config file is `nextflow.config`.  In here, we can set default global parameters for pipeline parameters (params), process, manifest, executor, profiles, docker, singularity, timeline, report and more.

For now, we are going to just add additional pipeline parameters and move the `params.query` out of the `main.nf` file and into the `nextflow.config` file.


Inside the `nextflow.config` file add the following parameters.

**nextflow.config**

  ```
  params.query = "myquery.fasta"
  params.dbDir = "/path/to/my/blastDB/"
  params.dbName = "myBlastDB"
  params.threads = 16
  params.outdir = "out_dir"
  ```

Remove these lines from `main.nf`

```
blastdb="myBlastDatabase"
params.query="file.fasta"
```

and let's modify the last print statement to include all the parameters.

```
println "I want to BLAST $params.query to $params.dbDir/$params.dbName using $params.threads CPUs and output it to $params.outdir"
```

Your `main.nf` file should look like this.

```
/usr/bin/env nextflow

println "I want to BLAST $params.query to $params.dbDir/$params.dbName using $params.threads CPUs and output it to $params.outdir"

```

**output:**

  ```
  nextflow run main.nf
  N E X T F L O W  ~  version 20.07.1
  Launching `main.nf` [fervent_swanson] - revision: 418bbdfbef
  I want to BLAST myquery.fasta to /path/to/my/blastDB//myBlastDB using 16 CPUs and output it to out_dir
  ```

  Let's add a `\n` to the beginning and end of the print statement so it reports a little more cleanly

  ```
  /usr/bin/env nextflow

  println "\nI want to BLAST $params.query to $params.dbDir/$params.dbName using $params.threads CPUs and output it to $params.outdir\n"

  ```

**output: This looks better:**

  ```
  nextflow run main.nf
  N E X T F L O W  ~  version 20.07.1
  Launching `main.nf` [modest_crick] - revision: 87c6232474

  I want to BLAST myquery.fasta to /path/to/my/blastDB//myBlastDB using 16 CPUs and output it to out_dir

  ```

#### Alternate params config

We can also write the pipeline parameters in a different format that is more similar to what we will be using for the rest of the config definitions.

Instead of

```
params.query = "myquery.fasta"
params.dbDir = "/path/to/my/blastDB/"
params.dbName = "myBlastDB"
params.threads = 16
params.outdir = "out_dir"
```

We can write it as follows.  Go ahead and change the nextflow.config file to look like this and rerun it to verify to yourself that it works identically.

```
params {
  query = "myquery.fasta"
  dbDir = "/path/to/my/blastDB/"
  dbName = "myBlastDB"
  threads = 16
  outdir = "out_dir"
}
```

```
nextflow run main.nf
```





## What is **it**?