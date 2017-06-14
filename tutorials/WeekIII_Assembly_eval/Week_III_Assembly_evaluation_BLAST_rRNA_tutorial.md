# Week III. Assembly evaluation: BLAST rRNA hits
*SIOB278 Ocean 'Omics: Metagenome Data Processing & Analysis workshop tutorial*

* Last edit: 4/21/17
* Jessica Blanton

**Tutorial Goals**:

1. Running a BLAST search from the command line against a "custom" database 
1. Evaluate the number of ribosomal markers are found in your dataset

**Software used**:

- [**BLAST+ v2.2.31**](https://www.ncbi.nlm.nih.gov/books/NBK279690/)
- [**seqtk**](https://github.com/lh3/seqtk)
- [**Tabview**](https://github.com/TabViewer/tabview)

## Overview

The 16S rRNA gene is currently the standard for taxonomic assignment for bacteria & archaea. It's eukaryotic counterpart, the 18S rRNA gene is no slouch either. In this tutorial you will look to see if these ribosomal markers are present in your assembly, and what taxa they are linked to.  

###**The GUM007 16S rRNA amplicon profile**
Now's a good time to look at the amplicon profile that was run on GUM007 DNA before the shotgun sequencing too place. 

![Agarwal,Blanton2017](/Users/jess/Dropbox/SCOHH_sponges/pbde\ paper/poster\ figs/Figure_3_bkgrnd.pdf) 

To explore in more detail [download this zip to your local laptop](https://www.dropbox.com/s/27arcojnswn46x3/GUM_007_102_096_barcharts.zip?dl=1) and check out the **bar_charts.html** file. It includes two other sponge types for comparison, and all three were processed to enrich for the cyanobacterial symbiont shown here:

 ![*Hormoscilla (Oscillatoria) spongeliae*](/Users/jess/Dropbox/SCOHH_sponges/Oscillatoria\ microscopy/Top_picks/Hormoscilla_spongelieae_63x.pdf).

###**On using BLAST+**

**B**asic **L**ocal **A**lignment **S**earch **T**ool (BLAST) searches are a staple of bioinformatic analysis. The **BLAST+** suite of tools allows us to do these alignment searches from the command line, in this case against a databse of our choosing. A classic move.


###**On the SILVA rRNA Database**

This is a wonderfully expansive collection of 16S/18S rRNA gene sequences, with taxonomy.  The team at the [SILVA rRNA database project](https://www.arb-silva.de/) work hard to create highly curated reference databases and alignments for all to use.  You’ll see that the sequences included have Accession numbers, because they are sourced from the National Center for Biotechnology Information (NCBI), and more info about each sequence can be found there.

###**On using Tabview to look at delimited txt documents**
Have you noticed that looking at tab-dilimited, comma-delimited, or other documents in the terminal emulator is just terrible to read?  Tabview solves this by displaying your text spaced in an nice row-by-column format.

	
## Installing programs and dependencies, download database

**Install BLAST+**

```
sudo apt install ncbi-blast+

```

Download Custom scripts (thanks Sheila Podell!), and make them executable. These are handy manipulators of text documents.

```
cd /opt/
sudo wget https://www.dropbox.com/s/33ys9vd9fo442vt/outer_join_tabfiles.pl
sudo chmod +x /opt/outer_join_tabfiles.pl 

sudo wget https://www.dropbox.com/s/5fun5m624uiosyd/select_tab_first.pl
sudo chmod +x /opt/select_tab_first.pl 
```
Install Tabview for easy viewing of csv/tsv documents.

```
cd /opt/
sudo git clone https://github.com/TabViewer/tabview.git
cd tabview
sudo python setup.py install

#Check that Tabview it is in your $PATH by calling the help screen
cd ~
tabview -h
```

Download SILVA ribosomal database v119.

```
mkdir ~/databases/
cd ~/databases/
wget https://www.arb-silva.de/fileadmin/silva_databases/qiime/Silva_119_release.zip
unzip Silva_119_release.zip
```
Copy files with representatives from clusters of 99% and 97% identity and remove the rest. You’ll want both the sequence file and it’s maxing taxonomy.  

```
cd ~/databases/
mkdir silva_blastdb
cd silva_blastdb

# info about the collection:
cp ../Silva119_release/Silva_119_notes.txt .

cp ../Silva119_release/rep_set/97/Silva_119_rep_set97.fna .
cp ../Silva119_release/taxonomy/97/taxonomy_97_7_levels.txt .

cp ../Silva119_release/rep_set/99/Silva_119_rep_set99.fna .
cp ../Silva119_release/taxonomy/99/taxonomy_99_7_levels.txt .
```
Check that your files make sense

```
cd ~/databases/silva_blastdb

head ~/databases/silva_blastdb/Silva_119_notes.txt
head ~/databases/silva_blastdb/Silva_119_rep_set97.fna
head ~/databases/silva_blastdb/taxonomy_97_7_levels.txt
```

Do a little cleanup of unneeded database files for the sake of disk space
	
	sudo rm -rf ~/databases/Silva_119_release.zip ~/databases/Silva119_release/ ~/databases/__MACOSX/	

## Run a BLAST search for Ribosomal proteins

The first thing is that we need to get the SILVA ribosomal databases in a "BLAST-able" format. Takes a few seconds…

```
cd ~/databases/silva_blastdb
makeblastdb -in Silva_119_rep_set97.fna -dbtype nucl;
makeblastdb -in Silva_119_rep_set99.fna -dbtype nucl

```

*Q. How many sequnces are in each set?*

Set up a results directory in your project folder
	
	mkdir ~/projects/GUM007/assemblies/megahit/G7_subst5_megahit_041617/rRNA_blast/

Run the your assembly with the BLASTn tool for nucleotide alignments, retrieving the top 5 hits

```
cd /home/ubuntu/databases/silva_blastdb

blastn -db Silva_119_rep_set97.fna \
-query ~/projects/GUM007/assemblies/megahit/G7_subst5_megahit_041617/final.contigs.fa \
-outfmt 6 -evalue 1e-10 -max_target_seqs 5 -out ~/projects/GUM007/assemblies/megahit/G7_subst5_megahit_041617/rRNA_blast/ribosomal_contig.prelim.m8 -num_threads 7

# Took 1min
```

### Viewing your BLAST results

 	cd ~/projects/GUM007/assemblies/megahit/G7_subst5_megahit_041617/rRNA_blast/

Look at the top of your results document using "head" 
	
	head ribosomal_contig.prelim.m8

now Look at your results using tabview 

	tabview ribosomal_contig.prelim.m8
	# (type "q" to exit viewer)

### Combine BLAST results with taxonomy

Our blast results contain a lot of good info, but not the actual taxonomic assignments. We will add these using a nice perl script written by Sheila Podell: */opt/outer_join_tabfiles.pl* 

This script combines the entries of two tab-delimited files. The rows in one document are kept in order, and the rows of the second document are matched and pasted to it. Let’s call **doc1 the "frame"** and **doc2 the "follow"**.

We will use our blast results as the frame document, and follow this with the SILVA-described taxonomy.

Get a list of hit subject IDs to retrieve from taxonomy file

	cut -f2 ribosomal_contig.prelim.m8 > ribosomal_contigs.hitID

Get the taxonomic assignments for each subject ID
	
	grep -f ribosomal_contigs.hitID /home/ubuntu/databases/silva_blastdb/taxonomy_97_7_levels.txt > ribosomal_tax_follow.txt

The category used for joining must be the first column in each document. So re-paste "subject IDs" as first column in the blast results file
	
	paste ribosomal_contigs.hitID ribosomal_contig.prelim.m8 > ribosomal_contig_frame.m8


Join the hits with their taxonomic string

	/opt/outer_join_tabfiles.pl ribosomal_contig_frame.m8 ribosomal_tax_follow.txt | cut -f2-13,15 > ribosomal_ctg_blasttax.txt;


Make a header row describing each column This includes the [standard columns for .m8 format](http://www.pangloss.com/wiki/Blast), but modified to fit the column addition made above


	echo '#query_id;subject_id;%_identity;alignment_length;mismatches;gap_opens;q._start;q._end;s._start;s._end;evalue;bit_score;taxonomy' | sed 's/;/\t/g' > header.txt

Add a header row describing each column

	cat header.txt ribosomal_ctg_blasttax.txt > megahit_041617_ribosomal_ctg.txt

Now it should be easier to scan your results:

	tabview megahit_041617_ribosomal_ctg.txt

### Rough tally of relevant hits

E.G.

```
grep Cyanobacteria megahit_041617_ribosomal_ctg.txt | cut -f1-4,13
```
	
Try piping this same output to tabview
	
	grep Cyanobacteria megahit_041617_ribosomal_ctg.txt | cut -f1-4,13 | tabview -

Get the first hit for each contig with another custom script from sheila.

** Note: 
This is often, but not always the "top" hit. That is best decided by examining manually.

	/opt/select_tab_first.pl megahit_041617_ribosomal_ctg.txt > megahit_041617_ribosomal_ctg_first.txt 
	
Count Bacterial hits 
	
	grep "D_0__Bacteria" megahit_041617_ribosomal_ctg_first.txt -c

Count Eukaryotic hits 

	grep "D_0__Eukaryota" megahit_041617_ribosomal_ctg_first.txt -c

-
Summary of this assembly:
	

|Dataset|# of 16S|# of 18S|length of candidate cyano symbiont|
|-------|--------|--------|----------------------------------|
|Total|1|3| 1460bp|

*Q. Are all of these hits reasonable? Take a look at the statistics reported for each top hit!*

*Q. Given what you know about GUM007, including the amplicon-based ribosomal profile, do you think this subset of the data + assembly went well?*

-

