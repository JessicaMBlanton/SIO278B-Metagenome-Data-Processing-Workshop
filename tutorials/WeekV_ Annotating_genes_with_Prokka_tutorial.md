# Annotating genes with Prokka
*SIOB278 Ocean 'Omics: Metagenome Data Processing & Analysis workshop tutorial*

* Last edit: 4/28/17
* Jessica Blanton

**Tutorial Goals**:

1. Understand the complexity of gene prediction and gene annotation
2. Run basic prediction and annotation pipeline on metagenome

**Software used**:

* [Prokka](https://github.com/tseemann/prokka/blob/master/README.md#installation)
	* [Barrnap](https://github.com/tseemann/barrnap) (Prokka optional add-on for rRNA search)


## Overview


### On the many softwares of Prokka

Now that we are past some of the basic elements of data processing, we are starting to use software that conveniently "wrappers" other established and respected open-source programs. Prokka (itself open-source) is one such wrapper script providing a quick pipeline to annotate genomes and metagenomes. It calls the following commonly-seen bioinformatics programs & packages:

* BLAST+ (Camacho C et al. 2009) similarity search against protein sequences
* HMMER (Finn RD et al. 2011) similarity searching against protein family profiles
* Prodigal ( Hyatt 2010 )  	Finds Coding sequence (CDS) 
* RNAmmer ( Lagesen et al. 2007 )  	Finds Ribosomal RNA genes (rRNA) 
* Aragorn ( Laslett and Canback 2004 )  	Finds Transfer RNA genes 
* SignalP ( Petersen et al. 2011 )  	Finds Signal leader peptides 
* Infernal ( Kolbe and Eddy 2011 )  	Finds Non-coding RNA 
* BioPerl (Stajich et al. 2002) "Perl modules for the life sciences"
* TBL2ASN Prepare sequence records for Genbank submission

The short pub about its release contains all the details: 

> Seemann T. Prokka: rapid prokaryotic genome annotation. Bioinformatics. 2014 Jul 15;30(14):2068-9. PMID:24642063


### Gene Prediction - searches for signal that a sequence is fuctional

As you read about this in our assigned reading, consider these nice structures from [Shafee and Lowe 2017](https://en.wikiversity.org/wiki/WikiJournal_of_Medicine/Eukaryotic_and_prokaryotic_gene_structure), characterizing the elements we look for when we're searching for genes:

*Eukaryotic gene structure*

![Eukaryote](https://upload.wikimedia.org/wikipedia/commons/thumb/5/54/Gene_structure_eukaryote_2_annotated.svg/640px-Gene_structure_eukaryote_2_annotated.svg.png)

*Bacterial/Archaeal gene structure*

![Prokaryote](https://upload.wikimedia.org/wikipedia/commons/thumb/f/fd/Gene_structure_prokaryote_2_annotated.svg/640px-Gene_structure_prokaryote_2_annotated.svg.png)

### Gene *Annotation* - assigning likely function of a gene sequence

The Prokka pipeline includes programs that recognize genes with distinct structures, such as tRNAs and rRNAs. It then searches for similarity in UniProt, RefSeq, and even custom user-provided databases using the BLAST+ suit of tools. It will also use HMMER3 to search curated models of protein families, aka, the hidden Markov model profile databases Pfam and TIGRFAMs. 
	
## Installing programs and dependencies

Sign into your AWS instance

	ssh -i SIO278B_SI17_jmblanto.pem ubuntu@ec2-52-42-111-66.us-west-2.compute.amazonaws.com

Install a bunch of dependencies

	sudo apt-get install libdatetime-perl libxml-simple-perl libdigest-md5-perl git default-jre bioperl

Download Prokka via Git

	cd /opt/
	sudo git clone https://github.com/tseemann/prokka.git

Index the sequence databases
	
	sudo prokka/bin/prokka --setupdb
	
check your prokka installation

	/opt/prokka/bin/prokka --version       
		# prokka 1.12

Install an additional program Barrnap for finding rRNAs
	
	cd /opt/
	sudo git clone "https://github.com/tseemann/barrnap.git"
	echo "PATH=$PATH:/opt/barrnap/bin" >> /home/ubuntu/.bashrc
	source /home/ubuntu/.bashrc


Check that Barrnap is in your path. You should get the commented ("#") output shown with this command:
	
	which barrnap
	# /opt/barrnap/bin/barrnap
	
## Running Prokka on your assembly

	mkdir ~/projects/GUM007/annotations/
	mkdir ~/projects/GUM007/annotations/prokka_annot/
	cd ~/projects/GUM007/annotations/prokka_annot/
	
	/opt/prokka/bin/prokka \
	~/projects/GUM007/assemblies/megahit/G7_subst5_megahit_041617/final.contigs.fa \
	--outdir G7_sub5mega_prka_042617/ \
	--prefix G7sub5mega --metagenome \
	--cpus 7
	
	# Takes ~6 minutes

Lots of goodies have been generated in your directory!  You can get a detail on what each of the files contain from the [Prokka github documentation](https://github.com/tseemann/prokka/blob/master/README.md#output-files)

### Gene prediction 

Check the summary file

	cat G7_sub5mega_prka_042617/G7sub5mega.txt

**For the 50% subset, I have run prokka with the following output:

> cat G7sub50mega.txt

>		organism: Genus species strain
		contigs: 6540
		bases: 14764024
		repeat_region: 4
		CDS: 11134
		rRNA: 4
		tRNA: 45
		tmRNA: 1

*Q. Now check your the 5% subset summary- when compared to an assembly of 10x more data, how many fewer coding regions were predicted in the smaller assembly?* 

### Gene annotation 
	
*Q. What role will does gene prediction and annotation play for your own analysis?*
 Be as specific as you can, this is different for everyone!