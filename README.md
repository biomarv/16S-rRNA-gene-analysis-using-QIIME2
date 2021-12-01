# Qiime2 pipeline MetaBac Lab

## Introduction

Since we are using Qiime2 in the command line interface, basic knowledge about the shell is required. There are several tutorials out there to get into it (i.e. <http://linuxcommand.org/lc3_learning_the_shell.php>). When working with bioinformatical data, command line tools are very powerful and makes data processing a lot more effective than using a graphical interface. 

## Rawdata

In general, data from the sequencing facility comes already demultiplexed, barcodes are already removed. Beside our files of interest, `.fastq.gz` (compressed fasta files with Quality information), sometimes already fastqc-processed files a delivered, too. Thus, a fastqc-report stored as n `.html` file (with its corresponding `.zip` folders) is provided. For paired-end data, we should find two files differing only in the read no.:

```{bash}
J34142_S41_L001_R1_001.fastq.gz
J34142_S41_L001_R2_001.fastq.gz
```
The underscore separated filename is composed of:

| Element | Description |
| :------ | :---------- |
| J34142 | Samplename provided by the samplesheet. This is mostly given by the sequencing facility |
| S41 | Sample number is based on the sample order in the samplesheet |
| L001 | Lane number; always the same | 
| R1 | Read 1. If paired-end reads, there will be a read 2. 
| 001 | Last segment: alway 001 |

## FastQC reports

When checking the quality reports of FastQC, one must be aware of the origin and processing of the data. The evaluation of the quality is somewhat biased to a purpose. Looking into an `.html` report, FastQC comes with 11 checkpoints, hightling if failed or passed not considering the input data. Starting with `basic statistics` Encoding: Illumina 1.9 tells us, quality format is encoded in `Phred+33`. Total Sequence count should be congruent in forward and reverse reads. 

+ Sequence length distribution
Should show one peak.
+ Per sequence GC content content
Sharp shoulder may indicate existing adapters, primer or rRNA. 
+ Per base sequence content
Nucleotide frequency for each base at each position. As each sequence, in 16S amplicon sequencing,  
+ Sequence duplication level and  Overrepresented sequences
For 16S amplicon sequencing those two criteria should not be considered as valuable quality parameters since we are sequencing one gene. Thus, observing similar sequences is expected. 

## MultiQC

Additionally, you can check all fastq-reports with multiQC, which incorporates all reports into a single report. By moving into the report-folder multiqc only needs an input folder as an argument. For further options use `multiqc -h`

```{bash}
multiqc .
```

## Set the environment 

```{bash}
conda activate qiime2-2021.2
```

## Prepare your files

Create project folder and change into it

```{bash}
mkdir PROJECT_NAME
cd PROJECT_NAME
```

Assign your metadata file to a variable. This file will be used more often, so using a variable comes in handy. The content of variable is accessed with a `$` prefix. 

```{bash}
METADATA="/home/user/PROJECT_NAME/metadata-file.tsv"
echo $METADATA
```

