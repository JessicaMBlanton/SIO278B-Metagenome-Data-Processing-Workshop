# Week II. Explore and QC your data
*SIOB278 Ocean 'Omics: Metagenome Data Processing & Analysis workshop tutorial*

* Last edit: 4/15/17
* Jessica Blanton

**Tutorial Goals**:

1. Basic best practice for organizing data files and installed programs
1. Learn to evaluate the quality of your sequence data
1. Clean (trim) your data prior to any further processing

**Software used**:

* [**Fastqc v0.11.5**](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/)
* [**Trimmomatic v.0.36**](http://www.usadellab.org/cms/?page=trimmomatic)


## Overview
####What is the type of data you should have recieved?

Understanding the sequencing process lets you know if your data is ok, better, or worse than expected! With the long wait times at sequenceing cores, you'll want to be able to let them know if there is an issue as soon as you recieve your files. 

Consider the steps used to generate your data:

1. Sample DNA extraction, attempting to minimize degradation
2. Shearing?
3. Size selection of DNA to control insert size?
1. Adapter/sequence site/barcode ligation
1. Size selection of library
1. Pooling
1. Sequence by synthesis  - SE, PE, MP

####  What is the format of the input data?
A large part of seqeunce processing is constantly converting between formats. fasta vs fastq sequence formats:

Fasta structure:

1. 	Header (defline)
1. 	Sequence

```
head -2 seqfileR1.fa
	>HEADER Description
	AGTCAGTCAGTCAGCTAGTCAGTCAGTCAGCTAGTCAGTCAGTCAGCT
```
Fastq structure:

1. 	Header
1. 	Sequence
1. 	Quality score identifier
1. 	Quality score
	 
```
head -4 seqfileR1.fa
	@HEADER Description
	AGTCAGTCAGTCAGCTAGTCAGTCAGTCAGCTAGTCAGTCAGTCA
	+
	AAGGGIGGGGIIG<AGGGAGAG.AAGGAGGIGG.<GGGIIGA.<A
```
*Q. What is the major difference between these formats, what are the pros/cons?*
*A. There is more data in the fastq file. More information, but bigger files!*

---

## Getting your computer set up
#### ssh into a running instance
	ssh -i /Users/jess/Documents/Archived_Docs/SIO278B_SI17_jmblanto.pem ubuntu@ec2-34-208-144-51.us-west-2.compute.amazonaws.com



#### Installing programs and dependencies

**Note: we are putting things in a somewhat deliberate location (/opt/) where we will have to specify the path for each executable (command). 

	cd /opt/
	sudo apt-get install unzip
	sudo apt-get install default-jre
	
Fastqc v0.11.5
	
	cd /opt/
	sudo wget http://www.bioinformatics.babraham.ac.uk/projects/fastqc/fastqc_v0.11.5.zip
	sudo unzip fastqc_v0.11.5.zip
	sudo chmod 755 FastQC/fastqc 
	
Trimmomatic v.0.36
	
	cd /opt/
	sudo wget 'http://www.usadellab.org/cms/uploads/supplementary/Trimmomatic/Trimmomatic-0.36.zip'
	sudo unzip Trimmomatic-0.36.zip
	sudo chmod 755 Trimmomatic-0.36/trimmomatic-0.36.jar 

####Setup archival directory to store your raw data files

	mkdir ~/raw_reads/
	mkdir ~/raw_reads/GUM007
	cd ~/raw_reads/GUM007

####Download the data
	
	wget 'https://www.dropbox.com/s/zyz3heoe8xi1oil/GUM7E1_S4_L001_R1_001.fastq.gz'&
	wget 'https://www.dropbox.com/s/te1q942aejyhey7/GUM7E1_S4_L001_R2_001.fastq.gz'&

	# each takes 5min
	
Check to see if this file uncorrupted by seeing if the digital fingerprint (the md5sum) is the same for your downloaded file as the one expected

```
md5sum GUM7E1_S4_L001_R1_001.fastq.gz
# 2503a55d3266376e52dddf20bfb0f7d2  GUM7E1_S4_L001_R1_001.fastq.gz
	
md5sum GUM7E1_S4_L001_R2_001.fastq.gz
# 73f08be2a3ef2bbc32eb9ecbc3077a23 GUM7E1_S4_L001_R2_001.fastq.gz
```
####Tag your data with an informational document

```
echo 'Shotgun metagenome data of 1 samples of dysideidae sponge GUM007 
enriched for trichomes by spinning through RNALater. 
This same DNA was used to generate PacBio sequence data. 
	
Sequenced at IGM core on Hiseq, PE150.
Data returned 10/16/2015
Downloaded for SIOB 278 on 4/12/17' > seqinfo.txt

```

Check the files. 

*Q. How big are they and why is there more than 1 file for this 1 sample?*
	
```
ll 
total 20646800
drwxrwxr-x 2 ubuntu ubuntu        4096 Apr 15 11:05 ./
drwxrwxr-x 3 ubuntu ubuntu        4096 Apr 15 11:04 ../
-rw-rw-r-- 1 ubuntu ubuntu 10244742805 Apr 15 11:08 GUM7E1_S4_L001_R1_001.fastq.gz
-rw-rw-r-- 1 ubuntu ubuntu 10897556802 Apr 15 11:09 GUM7E1_S4_L001_R2_001.fastq.gz
-rw-rw-r-- 1 ubuntu ubuntu         294 Apr 15 11:05 seqinfo.txt
```

---

## Use FastQC to confirm the awesome sequence data you expected… is what you got
Now that we are going to fuss with the data, we should do this in a working directory, away from our archived raw data.

	mkdir ~/projects/
	mkdir ~/projects/GUM007
	cd ~/projects/GUM007
	
	# Create “softlinks” to easily refer to your data files. Using the full path in your commands would also work.
	ln -s ~/raw_reads/GUM007/GUM7E1_S4_L001_R1_001.fastq.gz .
	ln -s ~/raw_reads/GUM007/GUM7E1_S4_L001_R2_001.fastq.gz .

Run FastQC:

	mkdir fastqc
	/opt/FastQC/fastqc GUM7E1_S4_L001_R1_001.fastq.gz -o fastqc &
	/opt/FastQC/fastqc GUM7E1_S4_L001_R2_001.fastq.gz -o fastqc &
	# each take about 12 min

[Best practices presentation, University	of	Cambridge](http://bioinformatics-core-shared-training.github.io/cruk-bioinf-sschool/Day1/NGS_QC_inesdesantiago.pdf)

### Use the data qc evaluation to choose parameters for clean up
look at .html output in a browser- *on your own laptop*

```
cd /Users/jess/Downloads
scp -i /Users/jess/Documents/Archived_Docs/SIO278B_SI17_jmblanto.pem ubuntu@ec2-35-167-250-241.us-west-2.compute.amazonaws.com:/projects/GUM007/fastqc/*fastqc.zip .
```
Since this will take a moment to run, here are the pre-reun files, to look at now:

[R1 fastqc](https://www.dropbox.com/s/soqn0td9ax4a4zu/GUM7E1_S4_L001_R1_001_fastqc.html?dl=0)

[R2 fastqc](https://www.dropbox.com/s/wkzjw9qpfjtanwv/GUM7E1_S4_L001_R2_001_fastqc.html?dl=0)

Parameters that are useful for general QC of any data, including shotgun metagenomes

* Basic statistics
* Per base sequence quality
* Per tile sequence quality
* Per sequence quality scores
* Per base sequence content
* Per base N content
* Sequence Length Distribution
* Overrepresented sequences
* Adapter Content
* Kmer Content

Parameters that are NOT especially useful when assessing metagenomic data

* Per sequence GC content
* Sequence Duplication Levels

*GC distribution assumes a biological relevance of this metric. What is the assumption?*

- [Bacterial Genomes](https://www.ncbi.nlm.nih.gov/pubmed/15568993) Bentley & Parkhill 2004
- [Plant genomes](https://www.researchgate.net/profile/Petr_Bures/publication/270817364/figure/fig6/AS:295035158450193@1447353271892/Fig-146.png) Šmarda & Bureš 2012
- [Metagenomes of increasing salinity](http://www.nature.com/articles/srep00135/figures/1) Ghai et al 2011


#### Record the results for each raw sequence file:

|Filename|State of Data|Total Sequences|Sequence length|Per base sequence quality trim 5’?|Per base sequence quality trim 3’?|Per base sequence content|GC patterns|Percent of seas remaining if reduplicated|Overrepresented sequences|Adapter Content|Kmer Content|
|--------|-------------|---------------|---------------|----------------------------------|----------------------------------|-------------------------|-----------|-----------------------------------------|-------------------------|---------------|------------|
|GUM7E1_S4_L001_R1_001.fastq.gz|raw data|105,947,360|150|No|Qscore sliding window 4:15|OK|sloping peak|52.19|NONE|NONE|NONE|
|GUM7E1_S4_L001_R2_001.fastq.gz|raw data|105,947,360|150|No|Qscore sliding window 4:15|OK|sloping peak|57.9|NONE|NONE|high in position 1-9|

___

##Trim reads for quality and to remove artifical sequence

Apply the fastqc evaluation to cleanup of our data

[trimmomatic program manual](http://www.usadellab.org/cms/?page=trimmomatic)

Make sure to use the current adapter file found here:

	ls /opt/Trimmomatic-0.36/adapters/TruSeq3-PE-2.fa

```
cd ~/projects/GUM007
mkdir trimmed_reads
cd trimmed_reads
		
sudo java -jar /opt/Trimmomatic-0.36/trimmomatic-0.36.jar PE \
-threads 7 \
-phred33 \
~/raw_reads/GUM007/GUM7E1_S4_L001_R1_001.fastq.gz \
~/raw_reads/GUM007/GUM7E1_S4_L001_R2_001.fastq.gz \
GUM7_forward_paired.fq.gz GUM7_forward_unpaired.fq.gz GUM7_reverse_paired.fq.gz GUM7_reverse_unpaired.fq.gz \
ILLUMINACLIP:/opt/Trimmomatic-0.36/adapters/TruSeq3-PE-2.fa:2:30:10 \
LEADING:3 \
TRAILING:10 \
SLIDINGWINDOW:4:15 \
MINLEN:100

# Takes 1h

# Record the informative goodies from the standard output:

echo ’Input Read Pairs: 105947360 Both Surviving: 89684875 (84.65%) Forward Only Surviving: 12891346 (12.17%) Reverse Only Surviving: 1183140 (1.12%) Dropped: 2187999 (2.07%)
TrimmomaticPE: Completed successfully
’ > trim.log
```
Check quality of trimmed data

	cd ../
	mkdir fastqc_trim
	/opt/FastQC/fastqc trimmed_reads/GUM7_forward_paired.fq.gz -o fastqc_trim &
	/opt/FastQC/fastqc trimmed_reads/GUM7_reverse_paired.fq.gz -o fastqc_trim &
 	
 # Takes 10 min 
 
Look at the .html output in a browser *on your laptop*:

[R1 trimmed fastqc]
(https://www.dropbox.com/s/8oj89wox40xo3iq/GUM7_forward_paired_fastqc.html?dl=0)

[R2 trimmed fastqc](https://www.dropbox.com/s/1ymjbqez4kdb7tv/GUM7_reverse_paired_fastqc.html?dl=0)

```
cd /Users/jess/Downloads

scp -i /Users/jess/Documents/Archived_Docs/SIO278B_SI17_jmblanto.pem ubuntu@ec2-34-208-144-51.us-west-2.compute.amazonaws.com:/projects/GUM007/fastqc_trim/*html .
```

Now we see that the strange k-mer blips in the start of R2 reads are gone. These were likely unremoved adapters or barcodes.

#### Record results for trimmed sequence files:

|Filename|State of Data|Total Sequences|Sequence length|Per base sequence quality trim 5’?|Per base sequence quality trim 3’?|Per base sequence content|GC patterns|Percent of seas remaining if reduplicated|Overrepresented sequences|Adapter Content|Kmer Content|
|--------|-------------|---------------|---------------|----------------------------------|----------------------------------|-------------------------|-----------|-----------------------------------------|-------------------------|---------------|------------|
|GUM7\_forward\_paired.fq.gz|raw, quality trimmed|89,684,875|100-150|na|na|OK|sloping peak|54.05|NONE|NONE|NONE|
|GUM7\_reverse\_paired.fq.gz|raw, quality trimmed|89,684,875|100-150|na|na|OK|sloping peak|54.86|NONE|NONE|NONE|
_

### Your data is ready to process. Next stop- assembly!
