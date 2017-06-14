# Week VI Simple binning with abundance vs. GC
*SIOB278 Ocean 'Omics: Metagenome Data Processing & Analysis workshop tutorial*

* Last edit: 5/8/17
* Jessica Blanton
* This tutorial follows the tutorial "WeekVIa_Mapping_reads_back_to_the_assembly

**Tutorial Goals**:

1. Calculate %GC statistic for all contigs
1. Graph %GC vs read coverage

**Software used**:

* **get_percent_GC.pl** - Sheila Podells's custom script
* [**R**](https://cran.rstudio.com/)
* [**R studio**](https://www.rstudio.com/)


## Overview


###**Compositional metrics for distinguishing genomes**

We have been discussing how different organism's genomes have unique siganatures from each other in metagenomic datasets.  There are two major aspects we are looking at today, which can be measured by looking at simple statistics about each contig.

1. **Abundance of organism : Number of reads that map to assembled contigs**. (eg. relatively more genome copies means relatively more reads will "map" to a contig assemble from this genome)

2. **Unique nucleotide composition : Distinct %GC or K-mer content**. Remember when we first looked at the %GC distribution in the FastQC tutotial, this metric has biological relevance, because organisms evolve different distributions of As, Gs, Ts, and Cs. 
		<img src="https://www.researchgate.net/profile/Petr_Bures/publication/270817364/figure/fig6/AS:295035158450193@1447353271892/Fig-146.png" width="390"/> 
	
	*Plant genomes GC Distribution from Šmarda & Bureš, Plant Genome Diversity 2012 1;209-235* 




###**On graphing metrics**

There are much more sophisticated ways to dive into the unique signatures of genomes, but it's surprisingly informative to start simply. In this tutorial we illustrate how two metrics can start to distinguish different genome "bins". Simply put, let's plot the **%GC** against **depth of coverage** for all the contigs.

## Setup/installations
	
##### ON YOUR AWS INSTANCE
**1. Download custom scripts** (thanks Sheila Podell!), and make them executable. 

```
cd /opt/

sudo wget https://www.dropbox.com/s/hz3o5obucgw5biy/get_percent_GC.pl

# This one you should already have, but just in case...
sudo wget https://www.dropbox.com/s/33ys9vd9fo442vt/outer_join_tabfiles.pl

sudo chmod +x /opt/*.pl 

```
##### ON YOUR LAPTOP:
RStudio installation stuff. Duh skip this if you already have it!

**2. Download and install R** (Required for Rstudio)

- [https://cran.rstudio.com/](https://cran.rstudio.com/)

**3. Download and install RStudio**

- Scroll to bottom for appropriate download: [https://www.rstudio.com/products/rstudio/download2/](https://www.rstudio.com/products/rstudio/download2/)

**4. install the ggplot2 package**

*Open RStudio and type the following in your RStudio console:*

	install.packages("ggplot2")
Check it by loading the package w/o errors:

	library("ggplot2")

## Collect compositional statistics on the assembled contigs

**Setup directories**

	mkdir ~/projects/GUM007/binning
	mkdir ~/projects/GUM007/binning/mega5/
	mkdir ~/projects/GUM007/binning/mega5/gc_cov
	cd ~/projects/GUM007/binning/mega5/gc_cov

**Get coverage stats and lengths, sorted by seq length.**

	echo "contig;length;depth_mapped;unmapped_reads" > header_cov.txt 
	samtools idxstats \
	~/projects/GUM007/mapping/mega5/G7mega5.bam \
	| sort -k2 -nr | grep -v "*" | cat header_cov.txt - | sed 's_;_\t_g' > G7mega5_ln_cov.txt

Check the structure of the resulting tab-delimited file
	
	head -5 G7mega5_ln_cov.txt 
	
		# contig  length  depth_mapped    unmapped_reads
		# k99_13825       78981   65757   0
		# k99_113023      76852   62267   0
		# k99_92301       67350   55021   0
		# k99_43914       56576   44319   0
**Get mean GC content for each sequence**

```
echo "contig;gc" > header_gc.txt 

/opt/get_percent_GC.pl \
~/projects/GUM007/assemblies/megahit/G7_subst5_megahit_041617/final.contigs.fa \
| cat header_gc.txt - | sed 's_;_\t_g' > G7mega5_gc.txt

```
Check the structure of the resulting tab-delimited file
	
	head -5 G7mega5_gc.txt 

		# contig  gc
		# k99_60  34
		# k99_67  41
		# k99_90  48
		# k99_138 33
**Combine length, mapping, and %GC statistics, still sorted by seq length**

```
/opt/outer_join_tabfiles.pl G7mega5_ln_cov.txt G7mega5_gc.txt | cut -f1,2,3,6 > G7mega5_ln_cov_gc.txt

```
Check out the file structure
	
	head G7mega5_ln_cov_gc.txt 
			
		# contig  length  depth_mapped    gc
		# k99_13825       78981   65757   47
		# k99_113023      76852   62267   45
		# k99_92301       67350   55021   47
		# k99_43914       56576   44319   48
		# k99_48378       54879   43209   47
		# k99_37866       41482   33073   48
		# k99_27428       40033   31974   48
		# k99_92801       40028   32571   48
		# k99_10749       36228   27848   48
Aren't you itching to graph this?

## Plotting %GC vs Coverage depth

While you can do plots any program that takes a table, there are so many rows in our file that it's best handled by something like matlab or R. And that way you end up with simple code that is easy to reproduce and tweak.

Do this however you like, but below is a simple way to visualize this in R. Hopefully you're all comfortable with RStudio? Call me if not!

-
####**Download your data to your local computer**

This is done on your laptop, changing the directory path to whatever suits you:

```
mkdir /Users/jess/Dropbox/R_directory/Metagenomic_analyses/GC_v_Coverage
cd /Users/jess/Dropbox/R_directory/Metagenomic_analyses/GC_v_Coverage 
scp -i SIO278B_SI17_jmblanto.pem ubuntu@ec2-52-42-111-66.us-west-2.compute.amazonaws.com:/home/ubuntu/projects/GUM007/binning/mega5/gc_cov/G7mega5_ln_cov_gc.txt .
```
-
####*The following to be done inside RStudio*

Set the working directory to the location containing your downloaded file

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
G7gvc <- read.table("G7mega5_ln_cov_gc.txt",
                    header=T, sep="\t", na.strings="NA", dec=".", strip.white=TRUE)
# check column headers
colnames(G7gvc) 

		# [1] "contig"       "length"       "depth_mapped" "gc"          
```
**Calculate estimated DEPTH from read coverage of ~120bp reads over the length of each sequence**

	depth_coverage<- ((G7gvc$depth_mapped)*120)/(G7gvc$length)
	G7gvc_cov <- cbind(G7gvc,depth_coverage)

**Explore the bounds of the data frame's variables:**

```
# length:
max(G7gvc$length)
min(G7gvc$length)

# depth_coverage:
max(G7gvc$depth_coverage)
min(G7gvc$depth_coverage)

# %GC:
max(G7gvc$gc)
min(G7gvc$gc)
```
**Visualize by making plots of %GC vs Depth of coverage**

** Note: For non-ggplot2 users, you can replace the "qplot" funtions below with this structure: `plot(G7gvc$gc, G7gvc$depth_coverage)`

 1. Plot at all the contigs 
 
		G7gvc.plot <- qplot(gc, depth_coverage, data=G7gvc_cov, main ="Composition of all contigs")
		G7gvc.plot

	Do you see somthing like [this image](https://www.dropbox.com/s/naunndg42g2htsb/G7gvc.jpeg)?
 2. Limit to larger contigs (range is 1kb-79kb) and replot
 
		G7gvc_3k <- subset(G7gvc_cov, length >= 3000)
		G7gvc_3k.plot <- qplot(gc, depth_coverage, data=G7gvc_3k, main ="Composition of large contigs")
		G7gvc_3k.plot
		

 3. Eliminate outlier high coverage contigs (range is 1.3-1.3k x coverage) and replot
 
		G7gvc_under400x <- subset(G7gvc_cov, depth_coverage <= 400)
		G7gvc_under400x.plot <- qplot(gc, depth_coverage, data=G7gvc_under400x, main ="Composition of contigs, no cov. outliers")
		G7gvc_under400x.plot

 4. Eliminate outlier high coverage AND limit to larger contigs and replot
 
		G7gvc_3ku400x <- subset(G7gvc_cov, depth_coverage <= 400 & length >= 3000)
		G7gvc_3ku400x.plot <- qplot(gc, depth_coverage, data=G7gvc_3ku400x, main ="Composition of large contigs, cov outliers")
		G7gvc_3ku400x.plot	
	
	Do you see something that might be the *Hormoscilla spongeliae* cyano-symbiont "bin" that appears in [Figure 4 of the published analysis](https://www.dropbox.com/s/m7irk6d9k7kcn47/Fig4_GCvCov.jpeg)?
	
5. You can save any of the plots to your working directory as a .jpeg image using:
	
		ggsave("G7gvc_3ku400x.jpeg",plot = G7gvc_3ku400x.plot)

		
**Please email me one of your plots.**  I like the last graph especially, but whatever you'd like to show, playing either with this data or your own!

