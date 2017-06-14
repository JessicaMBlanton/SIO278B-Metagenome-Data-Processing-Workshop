# Week II. MEGAHIT assembly- install and run

*SIOB278 Ocean 'Omics: Metagenome Data Processing & Analysis workshop tutorial*

* Last edit: 4/17/17
* Jessica Blanton

**Tutorial Goals**:

1. Downsample your data as needed to run-able size, maintaining paired structure.
1. Run MEGAHIT assembler

**Software used**:

- [**MEGAHIT v1.0.6.1**](https://github.com/voutcn/megahit)
- [**seqtk**](https://github.com/lh3/seqtk)


## Overview

###**On "Randomly Downsampling" your readfiles**

The first thing we need to do is make sure your data files are small enough to be useful.  There are two common reasons to use a smaller subset of your data: 

**1.** You want to testing operations or a full pipeline.  Trying things out on a smaller dataset saves time and resources during development (as is the case for this course).

**2.** Sometimes less is more when it comes to assemblers! Recall Table 2 of [Ghurye 2016 review](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5045144/).

The first reason is the primary one for the course- a small subset of data will let you build your pipeline “at the speed of learning”, and then when you have it polished it’s really easy to re-run a larger dataset! All the better the files we are using for the course must be relatively small to run within our resources on the Amazon Cloud.  

###**On trying Megahit as our first assembler**

We have chosen the MEGAHIT assembler for a number of reasons, but here's some of the important lowdown:

- It is a **Debruijn Graph assembler** (as opposed to Overlap Consensus or Greedy)
- It is **easy** to implement, yet has a number of **customizable** parameters
- It **only outputs contigs**, not scaffolds (there is a workaround for the brave as needed)
- It is perfect for some datasets, but can have a high **missasembly** rate horrible for others
- It is one of the **fastest** out there in it's "class" with regards to quality

That last reason is why we've chosen it for this course.  (ツ)



## Installing programs and dependencies

Sign into your AWS instance

	ssh -i SIO278B_SI17_jmblanto.pem ubuntu@ec2-52-42-111-66.us-west-2.compute.amazonaws.com

Install seqtk and a bunch of dependencies

	sudo apt install seqtk libz-dev g++ make

Make python sccessible in your path

	sudo ln -s /usr/bin/python3.5 /usr/bin/python
		# check that it is find-able:
	which python
		# /usr/bin/python

Download megahit and install (you’ll have lots of lines pass by for a while)
	
	cd /opt/
	sudo git clone https://github.com/voutcn/megahit.git
	cd megahit
	sudo make

Check that seqtk is working by calling it from the command line:
	
	seqtk
	
		# Usage:   seqtk <command> <arguments>
		# Version: 1.0-r31
	
		# Command: seq       common transformation of FASTA/Q
		         # comp      get the nucleotide composition of FASTA/Q
		         # sample    subsample sequences
		         # subseq    extract subsequences from FASTA/Q
		         # trimfq    trim FASTQ using the Phred algorithm
		
		         # hety      regional heterozygosity
		         # mutfa     point mutate FASTA at specified positions
		         # mergefa   merge two FASTA/Q files
		         # randbase  choose a random base from hets
		         # cutN      cut sequence at long N
		         # listhet   extract the position of each het


Check that MEGAHIT is working by pulling up the “help” screen.

While you're there, why not give it a bit of a read?
	
	/opt/megahit/megahit -h


-
### Downsample the data:

**Check out your data**

	cd ~/projects/GUM007/trimmed_reads/
	ll *_paired.fq.gz
	#-rw-r--r-- 1 root   root   7774770903 Apr 17 11:27 GUM7_forward_paired.fq.gz
	#-rw-r--r-- 1 root   root   8335851966 Apr 17 11:27 GUM7_reverse_paired.fq.gz

		
7-8G files are pretty big for our machine (and my patience)!  We definitly need to subset this. For this data, let's downsample to 1/20 the total dataset. Our fastQC analysis tells us this should be about 4484243 reads.

**Subset reads with seqtk**

* Indicate the 1/20 selection proportion as 0.05. 
* Make sure to set the same seed "-s" for each so the reads stay paired.

```
seqtk sample -s100 GUM7_forward_paired.fq.gz 0.05 > GUM7_F5.fq&
seqtk sample -s100 GUM7_reverse_paired.fq.gz 0.05 > GUM7_R5.fq&
# Takes 2min

```
-

### Run the assembler:

- filter to contigs ≥1kb
- use most of your CPUs (you have 8, but use only 7, leaving one for system processes)

```
mkdir ~/projects/GUM007/assemblies/
mkdir ~/projects/GUM007/assemblies/megahit
cd ~/projects/GUM007/assemblies/megahit


/opt/megahit/megahit --verbose --min-contig-len 1000 --num-cpu-threads 7 -1 ../../trimmed_reads/GUM7_F5.fq -2 ../../trimmed_reads/GUM7_R5.fq -o G7_subst5_megahit_041617&

/opt/megahit/megahit \
--verbose \
--min-contig-len 1000 \
--num-cpu-threads 7 \
-1 ../../trimmed_reads/GUM7_F5.fq \
-2 ../../trimmed_reads/GUM7_R5.fq \
-o G7_subst5_megahit_041617

# Takes 1h

```
Let’s look at the starting info for the assembly, at the top of the log file

```
head -8 log
	# MEGAHIT v1.1.1-2-g02102e1
	# --- [Mon Apr 17 12:28:59 2017] Start assembly. Number of CPU threads 7 ---
	# --- [Mon Apr 17 12:28:59 2017] Available memory: 33737007104, used: 30363306393
	# --- [Mon Apr 17 12:28:59 2017] Converting reads to binaries ---
	# /opt/megahit/megahit_asm_core buildlib G7_subst5_megahit_041617/tmp/reads.lib G7_subst5_megahit_041617/tmp/reads.lib
	#     [read_lib_functions-inl.h  : 209]     Lib 0 (../../trimmed_reads/GUM7_F5.fq,../../trimmed_reads/GUM7_R5.fq): pe, 8971430 reads, 150 max length
	#     [utils.h                   : 126]     Real: 11.2522 user: 9.7720    sys: 1.4560     maxrss: 167472
	# --- [Mon Apr 17 12:29:10 2017] k list: 21,29,39,59,79,99,119,141 ---
```	

Take a look at the final assembly summary- found at the tail end of the log

```
tail -2 log
	#--- [STAT] 7965 contigs, total 18594537 bp, min 1000 bp, max 91430 bp, avg 2335 bp, N50 2541 bp
	#--- [Mon Apr 17 13:27:03 2017] ALL DONE. Time elapsed: 3484.391373 seconds ---
```
	
OK! We have an assembly, and this is very exciting.  Our next step will be to evaluate how good it actually is.

In the meantime, **repeat this process with your independent dataset**, noting that you should try to cut your data to something reasonable (e.g. 1-2Gb per paired readfile, or around 4Gb total)
	
		
