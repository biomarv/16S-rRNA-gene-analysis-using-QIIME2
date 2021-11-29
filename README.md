# Qiime2 pipeline MetaBac Lab

## Introduction

Since we are using Qiime2 in the command line interface, basic knowledge about the shell is required. There are several tutorials out there to get into it (i.e. <http://linuxcommand.org/lc3_learning_the_shell.php>). When working with bioinformatical data, command line tools are very powerful and makes data processing a lot more effective than using a graphical interface. 

## Rawdata

In general, data from the sequencing facility comes already demultiplexed, barcodes are already removed. Beside our files of interest, `.fastq.gz` (compressed fasta files with Quality information), sometimes already fastqc-processed files a delivered, too. Thus, a fastqc-report stored as n `.html` file (with its corresponding `.zip` folders) is provided. For paired-end data, this is how your files should look like:

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

