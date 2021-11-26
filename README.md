# Qiime2 pipeline MetaBac Lab


## Rawdata

In general, data from the sequencing facility comes demultiplexed, barcodes are already removed.
Beside our files of interest, `.fastq.gz` (compressed fasta files with Quality information), there
are also already fastqc reports for each file stored as `.html` (with its corresponding `.zip` folders).
For paired-end data, this is how your files should look like:

J34142_S41_L001_R2_001.fastq.gz
J34142_S41_L001_R2_001.fastq.gz

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

Assign your metadata file to a variable

```{bash}
METADATA="/home/user/PROJECT_NAME/metadata-file.tsv"
```
<<<<<<< HEAD


=======
>>>>>>> 2ab24e02c439067cf309186c79a373fbb03d5981
