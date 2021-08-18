#!/bin/bash

# 20210621_GSF2848
Practicing RNA-seq analysis using GSF2848

# Activate RNA-seq environment

conda activate rnaseq 


# Unload Perl

module unload perl


# Variables

ASSEMBLY='ftp://ftp.wormbase.org/pub/wormbase/releases/WS275/species/c_elegans/PRJNA13758/c_elegans.PRJNA13758.WS275.genomic.fa.gz'
ANNOTATION='ftp://ftp.wormbase.org/pub/wormbase/releases/WS275/species/c_elegans/PRJNA13758/c_elegans.PRJNA13758.WS275.canonical_geneset.gtf.gz'


# Go to your slate account

cd /N/slate/by6


# Make a new folder 

mkdir GSF2848


# Change directory to project space: 

cd /N/project/HundleyLab/GSF2848


# Secure copy and paste files from HundleyLab project space 

scp GSF2848-EE* by6@carbonate.uits.iu.edu:/N/slate/by6/GSF2848


# Unzip all files pasted 

gunzip GSF2848*	


################################
## Generate STAR Genome Index ##
################################
	
# Make a directory to store the genome files in the GSF2848 folder 
	
mkdir -p genome


# Then go back to /N/slate/by6/GSF2848 Always go back when you run any alignment etc. 

	
# Download and unpack the genome assembly.
	
curl $ASSEMBLY | gunzip > ./genome/assembly.fasta
	

# Download and unpack the genome annotation.
	
curl $ANNOTATION | gunzip > ./genome/annotation.gtf
	

# Create a directory to store the index.
	
mkdir -p genome/index
	

# Create the STAR genome index.
# --genomeSAindexNbases 12 was recommended by software.
	
	STAR \
	  --runThreadN 4 \
	  --runMode genomeGenerate \
	  --genomeDir genome/index \
	  --genomeFastaFiles genome/assembly.fasta \
	  --sjdbGTFfile genome/annotation.gtf \
	  --genomeSAindexNbases 12
	

###########################
## Align Reads to Genome ##
###########################
	
# Create an output directory for aligned reads.
	
mkdir -p results/aligned
	
  
# Align the reads.
	
FASTQ=$GSF2848*
	
  
	for FASTQ in ${FASTQ[@]}; do
	  PREFIX=results/aligned/$(basename $FASTQ .fastq)_
	  STAR \
	    --runThreadN 8 \
	    --outFilterMultimapNmax 1 \
	    --outFilterScoreMinOverLread .66 \
	    --outFilterMismatchNmax 10 \
	    --outFilterMismatchNoverLmax .3 \
	    --runMode alignReads \
	    --genomeDir genome/index \
	    --readFilesIn $FASTQ \
	    --outFileNamePrefix $PREFIX \
	    --outSAMattributes All \
	    --outSAMtype BAM SortedByCoordinate
	done
	








