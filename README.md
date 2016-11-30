# SPANDx - a comparative genomics pipeline for microbes

## As of SPANDx version 3.2 we've upgraded to BWA-mem alignment (BWA v0.7+). This algorithm provides improved intraspecies SNP/indel identification and will be used as default for SPANDx versions post v3.2. If you would like to use a pre 0.7 version of bwa with the aln/sampe algorithm, please use a version of SPANDx pre v3.2. All version are available on sourceforge https://sourceforge.net/projects/spandx/

###N.B. We haven't yet upgraded the pipeline for really old Illumina data (pre 1.3 quality encoding). This will be implemented soon.


<i>What is SPANDx?</i>

SPANDx (Synergised Pipeline for Analysis of NGS Data in Linux) is a genomics pipeline for comparative analysis of haploid whole genome re-sequencing datasets. 

<i>Why use SPANDx?</i>

SPANDx is your one-stop tool for identifying SNP and indel variants in haploid genomes using NGS data. SPANDx performs alignment of raw NGS reads against your chosen reference genome or pan-genome, followed by accurate genome-wide variant calling and annotation, and locus presence/absence determination. SPANDx produces handy SNP and indel matrices for downstream phylogenetic analyses. Annotated SNPs and indels are identified and output in human-readable format. A presence/absence matrix is generated to allow you to identify the core/accessory genome content across all your genomes. The outputs generated by SPANDx can also be imported into PLINK for microbial genome-wide association study (mGWAS) analyses.

<i>How to install SPANDx from github</i>

1) Download the latest installation with git clone

git clone https://github.com/dsarov/SPANDx.git

2) Change into the SPANDx directory and alter file permissions

cd SPANDx/

chmod +x ./*

3) Download and install the GATK. Copy or link GATK into the SPANDx directory. Note that due licensing restrictions GATK is not included in the SPANDx bundle

4) Change the install location in the SPANDx.config file to direct SPANDx to its dependencies. Make sure that you have selected the correct resource manager for your system in the scheduler.config file (see below).

5) SPANDx should now be good to go!

<i>What resource managers can SPANDx use?</i>

From v2.7 onwards, SPANDx  works with SLURM, SGE and PBS (Torque) resource managers. SPANDx can also be run directly without a resource handler (set SCHEDULER=NONE in scheduler.config), although this is not recommended, particularly for modest to large datasets.

<i>SPANDx workflow</i>

To achieve high-quality variant calls, SPANDx incorporates the following programs into its workflow:

- <b>Burrows Wheeler Aligner (BWA)</b>
- <b>SAMTools</b>
- <b>Picard</b>
- <b>Genome Analysis Toolkit (GATK)</b>
- <b>BEDTools</b>
- <b>SNPEff</b>
- <b>VCFtools</b>

<i>How do I run SPANDx?</i>

USAGE: SPANDx.sh 
<parameters, required> 
-r <reference, without .fasta extension> 
[parameters, optional] 
-o [organism] 
-m [generate SNP matrix yes/no] 
-i [generate indel matrix yes/no] 
-a [include annotation yes/no] 
-v [Variant genome file. Name must match the SnpEff database] 
-s [Specify read prefix to run single strain] 
-t [Sequencing technology used Illumina/Illumina_old/454/PGM] 
-p [Pairing of reads PE/SE] 
-w [Window size in base pairs for BEDcoverage module]
-z [include tri- and tetra-allelic SNPs in the SNP matrix yes/no]

<i>What are the important things I need to know before running SPANDx?</i>

SPANDx, by default, expects reads to be paired-end, Illumina data in the following format: STRAIN_1_sequence.fastq.gz (first pair) and STRAIN_2_sequence.fastq.gz (second pair). 
Reads not in this format will be ignored.
If your data are not paired, you must set the -p parameter to SE to denote unpaired reads. By default -p is set to PE.

SPANDx requires a reference file in FASTA format. 
For compatibility with all steps in SPANDx, FASTA files should conform to the specifications listed here: http://www.ncbi.nlm.nih.gov/BLAST/blastcgihelp.shtml.
Note that the use of nucleotides other than A, C, G, or T is not supported by certain programs in SPANDx so should not be used in reference FASTA files. 
In addition, Picard, GATK and SAMtools handle spaces within contig names differently. Therefore, please do not use spaces or special characters (e.g. $/*) in contig names.

By default, all reads in SPANDx format (i.e. strain_1_sequence.fastq.gz) in the present working directory are processed. 
Sequence files are aligned against the reference using BWA. Alignments are subsequently filtered and converted using SAMtools and Picard Tools.
SNPs and indels are identified with GATK and coverage assessed with BEDtools. 

All variants identified in the single genome analysis are merged and re-verified across the entire dataset to minimise incorrect variant calls. This error-correction step is an attempt at establishing a "Best Practice" guideline for datasets where variant recalibration is not yet possible due to a lack of comprehensive quality-assessed variants (i.e. most haploid datasets!). Such datasets are available for certain eukaryotic species (e.g. human, mouse), as detailed in GATK "Best Practice" guidelines.

Finally, SPANDx merges high-quality, re-verified SNP and indel variants into user-friendly .nex matrices, which can be used for phylogenetic resconstruction using various phylogenetics tools (e.g. PAUP, PHYLIP, RAxML).

<i>Using SPANDx for microbial GWAS (mGWAS)</i>

The main comparative outputs of SPANDx (SNP matrix, indel matrix, presence/absence matrix, annotated SNP matrix and annotated indel matrix in $PWD/Outputs/Comparative/) can be used as input files for mGWAS. From version 2.6 onwards, SPANDx is distributed with GeneratePlink.sh. The GeneratePlink.sh script requires two input files: an ingroup.txt file and an outgroup.txt file. The ingroup.txt file should contain a list of the taxa of interest (e.g. antibiotic-resistant strains) and the outgroup.txt file should contain a list of all taxa lacking the genotype or phenotype of interest (e.g. antibiotic-sensitive strains). The ingroup.txt and outgroup.txt files must include only one strain per line. Although larger taxon numbers in the ingroup and outgroup files will increase the statistical power of mGWAS, it is better to only include relevant taxa i.e. do not include taxa that have not yet been characterised, or that have equivocal data. The GeneratePlink.sh script will generate .ped and .map files for SNPs, and presence/absence loci and indels if these were identified in the initial analyses. The .ped and .map files can be directly imported into PLINK. For more information on mGWAS and how to run PLINK, please refer to the PLINK website: http://pngu.mgh.harvard.edu/~purcell/plink/

GeneratePLINK usage:

GeneratePLINK.sh -i ingroup.txt -o outgroup.txt -r reference (without .fasta extension)

Comparing microbial genomes with the above methods will test for associations with orthologous and non-orthologous SNPs, indels and a presence/absence analysis. For more thorough mGWAS the accessory genome also needs to be taken into account and is a non-trivial matter in microbes due to the presence of multiple paralogs. To accurately characterise the accessory genome an accurate pan-genome is required. To construct a pan-genome we recommend the excellent pan-genome software, Roary (https://sanger-pathogens.github.io/Roary/). Once a pan-genome has been created the script GeneratePLINK_Roary.sh can be used to analyse associations between the groups of interest. GeneratePLINK_Roary.sh is run similarly to GeneratePLINK.sh and requires both an ingroup.txt file and an outgroup.txt file. Once this script has been run the .ped and .map files should be directly importable into PLINK.

GeneratePLINK_Roary.sh usage:

GeneratePLINK_Roary.sh -i ingroup.txt -o outgroup.txt -r Roary.csv output (if different than the default gene_presence_absence.csv)

<b>Note that this script has an additional requirement for R and Rscript with the dplyr package installed. If this script can’t find these programs in your path then it will fail. </b>

<i>Who created SPANDx?</i>

SPANDx was written by Derek Sarovich ([@DerekSarovich](https://twitter.com/DerekSarovich)) and Erin Price ([@Dr_ErinPrice](https://twitter.com/Dr_ErinPrice)) at Menzies School of Health Research, Darwin, NT, Australia.

<i>What to do if I run into issues with SPANDx?</i>

Please send bug reports to mshr.bioinformatics@gmail.com or derek.sarovich@gmail.com.

<i>How do I cite SPANDx?</i>

Sarovich DS & Price EP. 2014. SPANDx: a genomics pipeline for comparative analysis of large haploid whole genome re-sequencing datasets. <i>BMC Res Notes</i> 7:618.
