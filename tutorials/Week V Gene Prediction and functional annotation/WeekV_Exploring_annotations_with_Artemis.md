# Exploring annotations with Artemis

*SIOB278 Ocean 'Omics: Metagenome Data Processing & Analysis workshop tutorial*

* Last edit: 5/03/17
* Jessica Blanton

**Tutorial Goals**:

1. Importing annotation output for a metagenome to a commonly used viewer

**Software used**:

* [Artemis viewer](http://www.sanger.ac.uk/science/tools/artemis)

## Overview


###


## Set up your laptop

1. Download and install appropriate version of the [Artemis viewer](http://www.sanger.ac.uk/science/tools/artemis).

2. Copy the Prokka output from your Amazon server to your laptop

```
cd <path/to/class/output>
scp -r -i ../SIO278B_SI17_jmblanto.pem ubuntu@ec2-35-161-80-132.us-west-2.compute.amazonaws.com:/home/ubuntu/projects/GUM007/annotations/prokka_annot/G7_sub5mega_prka_042617 .
```

## Load annotated metagenome to Artemis

1. Open Artemis 
	* accept default working directory options
1. Open **.gff** file from your Prokka output within the program 
	* If you get warnings, check them out, but don't worry about them
1. The default view should include the "Base View" and "Feature List". In the viewer you can see the predicted genes and which contig they are on (megahit assembly contigs are named as "kxx_xxxx")

	[view 1- open in new tab](https://www.dropbox.com/s/09wrs5kmqtlehce/artemis_1.png)
	
1. Right click in the Feature list and select "Show Gene Names" then repeat and select "Show Products". Now you can see the predicted function of the annotations.

	[view 2- open in new tab](https://www.dropbox.com/s/menbruyx0avnlnu/artemis_2.png) 

#### Search for specific features

1. In the "GoTo" menu, select "Navigator..."
2. In the Navigator Dialog box, you can search for various gene names or "qualifiers". Here we highlight a recA gene annotation

	[view 3- open in new tab](https://www.dropbox.com/s/58arn8oq0dqtscv/artemis_3.png)




# Extra: Selecting a specific contig of interest

Sometimes it's not realistict to load your entire metagenomic assembly into Artemis (often the case), but you know you'd like to look at certain contigs/scaffolds


Install additional useful software:

* [Bioawk](https://github.com/lh3/bioawk)


```
sudo apt-get install bison
```
```
cd /opt/
sudo git clone https://github.com/lh3/bioawk
cd bioawk
sudo make
```

Identify the longest contig:

```
# remember to change this to your specific prokka directory

cd ~/projects/GUM007/annotations/prokka_annot/G7_sub5mega_prka_042617/

cat ~/projects/GUM007/assemblies/megahit/G7_subst5_megahit_041617/final.contigs.fa | /opt/bioawk/bioawk -c fastx '{ print $name, length($seq) }' | sort -k2 -nr | head -1 | cut -f1 > ctg_longest.txt
		
grep -f ctg_longest.txt final.contigs.fa -A1 > ctg_longest.fa
```

Identify the contig containing recA

```
cd ~/projects/GUM007/annotations/prokka_annot
# you can look for this inside the prokka output
grep recA G7sub5mega.tsv 

grep "k99_79374"  ~/projects/GUM007/assemblies/megahit/G7_subst5_megahit_041617/final.contigs.fa -A1 > recA_ctg.fa

/opt/prokka/bin/prokka \
recA_ctg.fa \
--outdir G7_sub5mega_recA_prka/ \
--prefix recActg --metagenome \
--cpus 7
```	

Now you have output for a single contig that can easily load in Artimis! 

