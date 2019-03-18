# Discovery of bioactive micropeptides in the immune system

This repository holds the pipeline for prediction of actively translated small open reading frames (smORFs) in the immune system.

## Obtaining

To download the source code, please use git to download the most recent development
tree.  Currently, the tree is hosted on github, and can be obtained via:

    git clone git://github.com/boboppie/orf-discovery.git
    
## Usage

### Dependencies

* [Samtools](http://www.htslib.org/)
* [bedtools](https://bedtools.readthedocs.io/en/latest/)
* [BEDOPS](https://bedops.readthedocs.io/en/latest/)
* [Bowtie](http://bowtie-bio.sourceforge.net/index.shtml)
* [STAR](https://github.com/alexdobin/STAR)
* [FastQC](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/)
* [Trim Galore](https://www.bioinformatics.babraham.ac.uk/projects/trim_galore/)
* [plastid](https://plastid.readthedocs.io/en/latest/index.html)
* [StringTie](https://ccb.jhu.edu/software/stringtie/)
* [EMBOSS](http://emboss.sourceforge.net/)
* [GNU Parallel](https://www.gnu.org/software/parallel/)(recommended)
* [R](https://www.r-project.org/)

R/Bioconductor packages:
* [riboSeqR](http://bioconductor.org/packages/release/bioc/html/riboSeqR.html)
* [GenomicFeatures](http://bioconductor.org/packages/release/bioc/html/GenomicFeatures.html)
* [rtracklayer](http://bioconductor.org/packages/release/bioc/html/rtracklayer.html)

### Tutorial to run the pipeline

#### Annotation

We recommend to use [GENCODE](https://www.gencodegenes.org/) annotation (GTF/GFF3 files) and genome and transcript sequences (Fasta files) for Human and Mouse. For other species, we have used [Ensembl](https://www.ensembl.org/info/data/ftp/index.html) annotation for Zebrafish. tRNA sequences can be downloaded from UCSC Table Browser.

For example, we have downloaded the following public sequences and annotation files for Mouse:

File | Type | Region | Source
---- | ---- | ------ | ------
Genome sequence, primary assembly | Nucleotide sequence of the GRCm38 primary genome assembly (Fasta format) | PRI (reference chromosomes and scaffolds) | GENCODE
Transcript sequences | Nucleotide sequences of all transcripts (Fasta format) | CHR (reference chromosomes only) | GENCODE
Protein-coding transcript sequences | Nucleotide sequences of coding transcripts (Fasta format) | CHR | GENCODE
LncRNA transcript sequences | Nucleotide sequences of lncRNA transcripts (Fasta format) | CHR | GENCODE
Comprehensive gene annotation | The main annotation file (GTF and GFF3 format) | CHR | GENCODE
LncRNA gene annotation | comprehensive gene annotation of lncRNA genes (GTF and GFF3 format) | CHR | GENCODE
tRNA sequences | Nucleotide sequences of tRNA genes predicted by UCSC using tRNAscan-SE (Fasta format) | CHR | UCSC Table Browser

#### Defining transcript sets

We combined GENCODE protein-coding transcripts and LncRNA transcripts to form reference transcriptome. Users can potentially assemble the transcriptome using RNA-seq data (e.g. [Cufflinks](http://cole-trapnell-lab.github.io/cufflinks/)), however, Studies have shown that computational approaches produce a large number of artefacts (false positives), which absorbed a substantial proportion of the reads from truly expressed transcripts and were assigned large expression estimates.

We do not consider the following [biotypes](https://www.gencodegenes.org/pages/biotypes.html):
* IG_* and TR_* (Immunoglobulin variable chain and T-cell receptor genes)
* miRNA
* misc_RNA
* Mt_rRNA and Mt_tRNA
* rRNA and ribozyme
* scaRNA, scRNA, snoRNA, snRNA and sRNA
* nonsense_mediated_decay
* non_stop_decay


For example, transcriptome can be generated by:

```bash
# Assume the source code is in ~/code/github/orf-discovery/
# Mouse GENCODE M13 reference files are in ~/ref/GENCODE/mouse/M13/
# Transcript sequence file downloaded from GENCODE is named gencode.vM13.transcripts.fa.gz, and placed in ~/ref/GENCODE/mouse/M13/fasta/transcriptome/
# The newly generated transcript sequence file name is transcripts_biotype_filtered.fa

cd ~/ref/GENCODE/mouse/M13/fasta/transcriptome/

~/code/github/orf-discovery/script/fastagrep.sh -v 'IG_*|TR_*|miRNA|misc_RNA|Mt_*|rRNA|scaRNA|scRNA|snoRNA|snRNA|sRNA|ribozyme|nonsense_mediated_decay|non_stop_decay' <(gzip -dc gencode.vM13.transcripts.fa.gz) >transcripts_biotype_filtered.fa
```

#### Defining ORFs

Given transcriptome sequences, we exhaustively searched for putative ORFs beginning with a start codon (“ATG”, “TTG”, “CTG”, “GTG”) and ending with a stop codon ("TAG", "TAA", "TGA") without an intervening stop codon in between in each of the three reading frames.

```bash
# Arguments:
# START_CODON - start codon, can be “ATG”, “TTG”, “CTG”, “GTG”  
# TRANSCRIPTOME_FASTA - transcriptome file location, assume is it in ~/ref/GENCODE/mouse/M13/fasta/transcriptome
# GTF - gene annotation file, assume it is in ~ref/GENCODE/mouse/M13/annotation/CHR/comprehensive/, and named gencode.vM13.annotation.gtf
# ORGANISM - scientific name, e.g. Mus musculus
# NCORE - number of computing cores if you want the process to be multi-threading, default 1
#
# for example, they can be assigned as: 
# START_CODON=ATG
# TRANSCRIPTOME_FASTA=~/ref/GENCODE/mouse/M13/fasta/transcriptome/transcripts_biotype_filtered.fa
# GTF=~ref/GENCODE/mouse/M13/annotation/CHR/comprehensive/gencode.vM13.annotation.gtf
# ORGANISM="Mus musculus"
# NCORE=8 
#
# Output:
# The output file is in BED12 format (https://genome.ucsc.edu/FAQ/FAQformat#format1) and named orf_$START_CODON.bed 

Rscript ~/code/github/orf-discovery/script/ORFPredict.R $START_CODON $TRANSCRIPTOME_FASTA $GTF $ORGANISM $NCORE 
```

In the output file, a unique ID is given for each ORF, for example:

    ENSMUST00000000010.8:Hoxb9:protein_coding:117:134:2:ATG

the fields separated by colons are **transcript ID**, **gene symbol**, **transcript biotype**, **transcript start**, **transcript stop**, **reading frame**, **start codon**. 

## Support

### email

Please report any issues or questions by creating a ticket, or by email to 
<fengyuan.hu@babraham.ac.uk>.
