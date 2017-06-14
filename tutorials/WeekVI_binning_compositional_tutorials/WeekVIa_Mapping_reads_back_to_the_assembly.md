# Week VI. Mapping reads back to the assembly
*SIOB278 Ocean 'Omics: Metagenome Data Processing & Analysis workshop tutorial*

* Last edit: 5/7/17
* Jessica Blanton

**Tutorial Goals**:

1. Map reads to the assembly contigs
1. calculate mean depth of coverage

**Software used**:

- [**Bowtie2**](http://bowtie-bio.sourceforge.net/bowtie2/manual.shtml)
- [**SAMtools**](http://samtools.sourceforge.net/)

## Overview 
<img  style="float:right" src="https://readtiger.com/img/wkp/en/Mapping_Reads.png" width="260" />

###**Read Mapping**

Read mapping is a particularly useful technique which can inform you about how well your assembly went (looking at % total reads map back, looking to see if the reads map in sensical orientations and distances). It also gives us the depth of read coverage is for each contig, which, for a metagenome, is correlated with abundance of the different organisms [genomes]. *image from https://readtiger.com/wkp/en/DNA_sequencing*


###**On using Bowtie2+**

Bowtie2 (and it's predecessor Bowtie) is a widely-used program for mapping reads to genomes.  It is highly paramaterizable, but we'll keep it simple here. The output of the program will provide you with some statistics, and from the user manual, this is what they mean:

* read pairs aligned concordantly: "A pair that aligns with the **expected relative mate orientation** and with the **expected range of distances between mates** is said to align 'concordantly'."


* read pairs aligned *either* 0 times concordantly (not seen) *or* aligned discordantly- are violoate the conditions above. But they are still informative, and may even be correct, especially as our the insert size of the sequenced DNA library is not uniform or standard.

Check out [this image](https://www.researchgate.net/profile/Satoru_Takeda/publication/266680708/figure/fig2/AS:271956289257501@1441850840879/Figure-1-Schematic-presentation-of-the-analytical-flow-and-graphical-presentation-of.png) from Suzuki et al 2014 (DOI: 10.1038/jhg.2014.88) for an a reasonable visual,


###**On using SAMtools**

SAMtools is another very well known and widely used program, here we're using it to convert our mapping files and sort them as needed.
	

## Installing programs and dependencies
Install Bowtie2 and SAMtools

```
sudo apt install bowtie2 samtools
```
Check your install by seeing if the help instructions are output with:

```
bowtie2 -h 
```
```
samtools view -h
```

## Map reads to your assembly
Setup directories

```
mkdir ~/projects/GUM007/mapping/
mkdir ~/projects/GUM007/mapping/mega5/
cd ~/projects/GUM007/mapping/mega5/
```
Build indexed DB of your contigs

```
bowtie2-build \
~/projects/GUM007/assemblies/megahit/G7_subst5_megahit_041617/final.contigs.fa contigs;
```
Map reads to indexed contigs. This will output a large human-readable .sam file.  We are using the "-t" flag to see how long each step takes.

Takes ~10 min

```	
cd ~/projects/GUM007/trimmed_reads/
rm GUM7_forward_paired.fq GUM7_reverse_paired.fq

bowtie2 \
--threads 7 \
--no-unal \
--no-discordant \
--fast-local \
-x contigs \
-1 ../../trimmed_reads/GUM7_F5.fq \
-2 ../../trimmed_reads/GUM7_R5.fq \
-S G7mega5.sam -t > mapfile.log
```
what is your output? What percent of your reads map?

	Time loading reference: 00:00:00
	Time loading forward index: 00:00:00
	Time loading mirror index: 00:00:00
	Multiseed full-index search: 00:09:47
	4485715 reads; of these:
	4485715 (100.00%) were paired; of these:
	3220978 (71.81%) aligned concordantly 0 times
	1117413 (24.91%) aligned concordantly exactly 1 time
	147324 (3.28%) aligned concordantly >1 times
	----
	3220978 pairs aligned 0 times concordantly or discordantly; of these:
	  6441956 mates make up the pairs; of these:
	    3469724 (53.86%) aligned 0 times
	    2469529 (38.34%) aligned exactly 1 time
	    502703 (7.80%) aligned >1 times
	61.32% overall alignment rate
	Time searching: 00:09:47
	Overall time: 00:09:47

Check out the sam file format

```
head G7mega5.sam

```

Convert the .sam file to .bam file.
Takes ~2min

	samtools view \
	-F 4 -bS G7mega5.sam \
	-o G7mega5_unsorted.bam

Sort and index the final .bam file
Takes ~3min 1036

	samtools sort G7mega5_unsorted.bam G7mega5;
	samtools index G7mega5.bam 

Cleanup intermediate output files
	
	rm G7mega5.sam G7mega5_unsorted.bam 



Now you have mapping files that we can use later, and in the next tutorial : "Week VI Simple binning with GC vs coverage"

--

**Please email me the overall alignment rate from this tutorial** (there is some randomness involved in the dataset downsampling and assembly, so it is unlikely to be identical to mine)
