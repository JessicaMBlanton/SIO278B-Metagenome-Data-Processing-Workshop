# Week VI using fully automated binning tools - MetaBAT
*SIOB278 Ocean 'Omics: Metagenome Data Processing & Analysis workshop tutorial*

* Last edit: 5/8/17
* Jessica Blanton
* This tutorial follows the tutorials 
	* "WeekVIa_Mapping_reads_back_to_the_assembly"
	* "WeekVIb_Simple_binning_with_GC_vs_Coverage"

**Tutorial Goals**:

* Run binning programs on assembled metagenome
* Compare binning outputs

**Software used**:

* [**MetaBAT v0.32.5**](https://bitbucket.org/berkeleylab/metabat)

## Overview
This tutorials covers a common fully-automated binning tool that gruoups metagenomic data based on abundance and tetranucleotide frequency. While these components were calculated "by hand" in our last tutorial, this program does it in a more sophisticated way using [maths](https://vimeo.com/13497928). Here's a little more information on how the two major compositional characteristics are represented in this program, and other elements that make it worth checking out:

- **Abundance - as distance metric.** We used mean read abundance mapped to each contig. MetaBAT also integrates the variance of coverage over the contigs to calculate distance between sequences.
- **Tetranucleotide frequency - as distance metric.** We used a simple statistic- the %GC for each contig.  MetaBAT uses full tetranucleotide frequency (TNF) to calculate empirical posterior probability of distance between contigs.
- **Uncertainty with small contigs.** We haven't explored the reliability of assigning small contigs to bins. But MetaBAT has, and the authors conclude that contigs <2.5kb are "unstable" with respect to the normal variance of a genome: it's hard to assign small contigs with reasonable certainty.
- **Iterative binning.** Ok, we haven't refined our binning with multiple rounds of anything. Metabat takes certain contigs (e.g. with highest coverage), and looks to recruit contigs which are closely related (within a distance cutoff) to it by abundance & tetranucleotide frequency. From this group it looks for a new "best" seed from the recruited group, and re-iterates this process until there are no more contigs to recruit within the given cutoffs.
Simply put, iterative binning is the process of taking the output of your maths- and doing more maths on it to refine your binning. Check [the methods paper](https://peerj.com/articles/1165/).

Importantly for us, the developers of MetaBAT have attempted make calls on where the bin boundaries are. Of note, the program comes with [lots of tweakable parameters](https://bitbucket.org/berkeleylab/metabat/wiki/Best%20Binning%20Practices). Let's see how this automated method perform!

## Setup/installations
	
Sign into your AWS instance

	ssh -i SIO278B_SI17_jmblanto.pem ubuntu@ec2-52-42-111-66.us-west-2.compute.amazonaws.com

**Install MetaBAT**

```
cd /opt/
sudo wget https://bitbucket.org/berkeleylab/metabat/downloads/metabat-static-binary-linux-x64_v0.32.4.tar.gz
sudo tar xvf metabat-static-binary-linux-x64_v0.32.4.tar.gz
sudo chown root:root metabat/
sudo chmod +rx metabat
sudo chmod +rx metabat/*
```
Check MetaBAT installation

```
/opt/metabat/metabat -h
/opt/metabat/runMetaBat.sh -h

```
## Use MetaBAT to bin GUM007 assembly

Setup directories

```
mkdir ~/projects/GUM007/binning/mega5/metabat
cd ~/projects/GUM007/binning/mega5/metabat

```
MetaBAT uses **read coverage** and **coverage variance** for each contig. Get a table of these statistics for each contig from the .bam file generated in the *WeekVIa_Mapping_reads_back_to_the_assembly* tutorial
 
```
# coverage file format is:
# contig_name		coverage		cov_variance

/opt/metabat/jgi_summarize_bam_contig_depths --outputDepth depth.txt ~/projects/GUM007/mapping/mega5/G7mega5.bam
```
Run MetaBAT script

```
/opt/metabat/metabat \
-i ~/projects/GUM007/assemblies/megahit/G7_subst5_megahit_041617/final.contigs.fa \
-a depth.txt \
--verysensitive \
-o metabat -l -v --unbinned > log.txt
# Note that we are outputting info to a logfile
```
The following series of commands collects contig names and adds group number *for each bin*, and appends the group lists together

```
sed 's/$/\tmetabat_1/g' metabat.1 > metabat.txt
sed 's/$/\tmetabat_2/g' metabat.2 >> metabat.txt
sed 's/$/\tmetabat_3/g' metabat.3 >> metabat.txt
sed 's/$/\tmetabat_unbinned/g' metabat.unbinned >> metabat.txt
# MetaBAT does not work on reads of <1500b, so we label them as "unbinned" with this eyebleed-inducing workaround:
/opt/outer_join_tabfiles.pl ../gc_cov/G7mega5_ln_cov_gc.txt metabat.txt | grep "metabat" -v | grep "contig" -v| cut -f1 | sed 's/$/\tmetabat_unbinned/g' >> metabat.txt
```
Add a header to the collected bin list

```
echo "contig;metabat" | cat - metabat.txt| sed 's/;/\t/g' > metabat_bin.txt
```
## Make master results file to visualize all binning methods

Combine results from both binning tools with the compositional stats you calculated in the *WeekVIb_Simple_binning_with_GC_vs_Coverage* tutorial

```
cd ~/projects/GUM007/binning/mega5/

/opt/outer_join_tabfiles.pl gc_cov/G7mega5_ln_cov_gc.txt metabat/metabat_bin.txt | cut -f1-4,6 >  G7mega5_ln_cov_gc_metabat.txt
```
Check the document by looking at the data structure:

```
head G7mega5_ln_cov_gc_metabat.txt
	# contig	length	depth_mapped	gc	metabat
	# k99_13825	78981	65757	47	metabat_1
	# k99_113023	76852	62267	45	metabat_1
	# k99_92301	67350	55021	47	metabat_1
	# k99_43914	56576	44319	48	metabat_1
	# k99_48378	54879	43209	47	metabat_1
	# k99_37866	41482	33073	48	metabat_1
	# k99_27428	40033	31974	48	metabat_1
	# k99_92801	40028	32571	48	metabat_1
	# k99_10749	36228	27848	48	metabat_1
```	
---
## Visualize on your local computer (laptop)

```
cd /Users/jess/Dropbox/R_directory/Metagenomic_analyses/GC_v_Coverage 
scp -i SIO278B_SI17_jmblanto.pem ubuntu@ec2-52-42-111-66.us-west-2.compute.amazonaws.com:/home/ubuntu/projects/GUM007/binning/mega5/G7mega5_ln_cov_gc_metabat.txt .

# If you are skipping the past tutorials, download this file here:
# wget https://www.dropbox.com/s/k3xa0nabfoqfzli/G7mega5_ln_cov_gc_metabat.txt
```
####*The following to be done inside RStudio*

**Set the working directory** to the location containing your downloaded file

```
setwd("~/Dropbox/R_directory/Metagenomic_analyses/GC_v_Coverage")
```
Load the ggplot2 package

```
library("ggplot2"); packageVersion("ggplot2")
```

**Load and explore the data frame**

```
# read in your data table (a.k.a data frame)
G7gvc_bin <- read.table("G7mega5_ln_cov_gc_metabat.txt",
                    header=T, sep="\t", na.strings="NA", dec=".", strip.white=TRUE)
# check column headers
colnames(G7gvc_bin) 

		# [1] "contig"	"length"	"depth_mapped"	"gc"	"metabat"     
```
**Calculate depth from read coverage over the length of each sequence**

```
depth_coverage<- ((G7gvc_bin$depth_mapped)*120)/(G7gvc_bin$length)
G7gvc_bin_cov <- cbind(G7gvc_bin,depth_coverage)
```

**Visualize by making plots of %GC vs Depth of coverage**

Plot all contigs

```
qplot(gc, depth_coverage, data=G7gvc_bin_cov, main ="Composition of all contigs", color=metabat)
```

Eliminate outlier high coverage AND limit to larger contigs and replot

```
G7gvc_bin_cov_3ku400x <- subset(G7gvc_bin_cov, depth_coverage <= 300 & length >= 3000)
qplot(gc, depth_coverage, data=G7gvc_bin_cov_3ku300x, main ="Composition of large contigs, cov outliers", color=metabat)
```
*Do you see something like [this image](https://www.dropbox.com/s/zits7rwjgtcr6yi/G7gvc_metabat_3ku300x.jpeg)?*
	
Save any of the plots to your working directory as a .jpeg image, e.g.:

```
G7gvc_metabat_3ku300x.plot <- qplot(gc, depth_coverage, data=G7gvc_bin_cov_3ku300x, main ="Composition of large contigs, cov outliers",  color=metabat)
ggsave("G7gvc_metabat_3ku300x.jpeg",plot = G7gvc_metabat_3ku300x.plot)
```		
**Q.** Now that we've added some results from an "official" binning tool, which "bin" do you think might be the *Hormoscilla spongeliae* cyano-symbiont "bin" that appears in [Figure 4 of the published analysis](https://www.dropbox.com/s/9ft73he6ml61nq8/G7gvc_metabat_3ku300x.jpeg)?

**Q.** Do you feel comfortable with the binning results as is? If there's anything you'd like to "refine" in the assignments shown in your plot, what would you describe it as?