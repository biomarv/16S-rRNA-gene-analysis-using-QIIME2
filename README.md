# Qiime2 pipeline MetaBac Lab

## Introduction

Since we are using Qiime2 in the command line interface, basic knowledge about the shell is required. There are several tutorials out there to get into it (i.e. <http://linuxcommand.org/lc3_learning_the_shell.php>). When working with bioinformatical data, command line tools are very powerful and makes data processing a lot more effective than using a graphical interface. 

### Set up your working directory

Create project folder in your home directory. In your project folder create a folder named i.e. `raw_data` for the .fq files. Then change into the project folder and check with the `ls` command the content.

```{bash}
mkdir ~/PROJECT_NAME
mkdir ~/PROJECT_NAME/raw_data 
cd PROJECT_NAME/
ls
```

### Metadata

Assign your metadata file to a variable. This file will be used more often, so using a variable comes in handy. The content of variable is accessed with a `$` prefix. 

```{bash}
METADATA="/home/user/PROJECT_NAME/metadata-file.tsv"
echo $METADATA
`ls`
```

### Rawdata

In general, data from the sequencing facility comes already demultiplexed, barcodes are already removed. Beside our files of interest, `.fastq.gz` (compressed fasta files with Quality information), sometimes already fastqc-processed files a delivered, too. Thus, a fastqc-report stored as n `.html` file (with its corresponding `.zip` folders) is provided. For paired-end data, we should find two files differing only in the read no.:

```{bash}
J34142_S41_L001_R1_001.fastq.gz
J34142_S41_L001_R2_001.fastq.gz
```

The underscore separated filename is composed of:

| Element | Description |
| :------ | :---------- |
| J34142 | Samplename provided by the samplesheet. This is mostly given by the sequencing facility |
| S41 | Sample number is based on the +  order in the samplesheet |
| L001 | Lane number; always the same + | 
| R1 | Read 1. If paired-end reads, th+ ere will be a read 2. 
| 001 | Last segment is set number: al+ ways 001 + + + + + |

### FastQC

When checking the quality reports of FastQC, one must be aware of the origin and processing of the data. The evaluation of the quality is somewhat biased to a purpose. Looking into an `.html` report, FastQC comes with 11 checkpoints, hightling if failed or passed not considering the input data. Starting with `basic statistics` Encoding: Illumina 1.9 tells us, quality format is encoded in `Phred+33`. Total Sequence count should be congruent in forward and reverse reads. 

+ Sequence length distribution: Should show one peak.
+ Per sequence GC content content: Sharp shoulder may indicate existing adapters, primer or rRNA. 
+ Per base sequence content: Nucleotide frequency for each base at each position. As each sequence, in 16S amplicon sequencing.  
+ Sequence duplication level and  Overrepresented sequences: For 16S amplicon sequencing those two criteria should not be considered as valuable quality parameters since we are sequencing one gene. Thus, observing similar sequences is expected. 

### MultiQC

Additionally, you can check all fastQC-reports with multiQC, which incorporates all reports into a single report. By moving into the report-folder multiqc only needs an input folder as an argument. For further options use `multiqc -h`

```{bash}
multiqc .
```

### Set the environment 

To activate the QIIME 2 environment use the following code:

```{bash}
conda activate qiime2-2021.2
```
To deactivate the environment use `conda deactivate`.
Working in this conda environment makes sure, that the correct version of Python is used running QIIME2 commands.

## QIIME2

To find out more about QIIME2 use the well documented source <https://docs.qiime2.org/2021.8/> with several useful tutorials. Clearly, we are using QIIME2 in the command-line interface (CLI), although there is a graphical use available. 
Indispensible, is knowledge about the difference between `.qza` and `.qzv`, the qiime artifacts. The ladder describes any file content that can be visualized. The `.qza` ending is a file were data is stored, processed in and QIIME knows about type and format. Those are the files we are dealing with when processing our data. While in bioinformatics lots of different file types exist which one needs to handle, this is one advantage of QIIME, so the user can focus on data processing. Another advantage of QIIME2 is, that every file keeps track of its provenance, so the workflow of the pipeline is always tracked. 
There are plenty tools to manipulate data and the first to begin with is importan our data into a QIIME2 artifact:

### Import data

``` {bash}
mkdir qiime_files
qiime tools import \
    --type SampleData[PairedEndSequencesWithQuality] \
    --input-format CasavaOneEightSingleLanePerSampleDirFmt \
    --input-path $HOME/PROJECT_NAME/raw_data \
    --output-path /qiime_files/demux_paired_end.qza
```
The tools package has plenty functions to manipulate QIIME2 files. To find out more about its function you can use
``` {bash}
qiime tools --help
```
The tool needs three required inputs: the `--input-path` to tell the program were the data is located, the `--output-path` were to output the QIIME2 artifact and the `--type` of data. In this case, we are dealing with paired-end sequencing data which also has qulity information. There are several data types which we can look up using:
``` {bash}
qiime tools import \
    --show-importable-types
```
Additionally, the `--input-format` option is provided, which tells QIIME2 the files are in the format explained in the *rawdata* section.

Check out the file type which has been imported:
```{bash}
cd qiime_files
qiime tools peek \
    demux_paired_end.qza
```

### Summarize data and visualization

The next command summarizes the data and generates a visualisation file which we can inspect now:
```{bash}
qiime demux summarize \
    --i-data demux_paired_end.qza \
    --o-visualization demux_paired_end.qzv

qiime tools view \
    demux_paired_end.qzv
```
This file gives you information about general sequence distribution and also an interactive quality plot.

### Primer trimming

To remove adapter sequences, primers and any unwanted sequences from the reads, `cutadapt` is used. 
``` {bash}
qiime cutadapt trim-paired \
    --i-demultiplexed-sequences demux-paired-end.qza \
    --p-adapter-f ATTAGAWACCCBDGTAGTCC \
    --p-front-f CCTACGGGAGGCAGCAG \
    --p-adapter-r CTGCTGCCTCCCGTAGG \
    --p-front-r GGACTACHVGGGTWTCTAAT \
    --p-discard-untrimmed \
    --p-times 2 \
    --o-trimmed-sequences demux_cutadapt.qza \
    --verbose > trimming_report.log
```
To find out more about the primer options of this command have look at `qiime cutadapt trim-primer --help`. In this example data set the V3/V4 region of the 16S rRNA was used. The additional two primers are the reversed complements of the V3/V4 primer pair. The option `--p-discard-untrimmed` excludes reads in which no primers were found. The last option redirects the standard output (stdout) into a logfile. The output can also be visualized with the `demux summarize` command. 

## Denoising

To denoise the reads qiime offers two packages, deblur and DADA2. Both pipelines end up with amplicon sequence variants (ASV). In this tutorial the DADA2 package will be used. Your can find out more about the package in this [tutorial](https://benjjneb.github.io/dada2/) or have a look at the [publication](https://pubmed.ncbi.nlm.nih.gov/27214047/). The general purpose of both is to correct for errors, filter phiX reads and chimeric sequences. To set the `trunc` option one has to inspect the quality plot from the `demux_cutadapt.qza` file. 

```{bash}
qiime dada2 denoise-paired \
     --i-demultiplexed-seqs demux_cutadapt.qza \
     --p-trunc-len-f 270 \
     --p-trunc-len-r 247 \
     --p-trunc-q 5 \
     --o-table dada2_table.qza \
     --o-representative-sequences rep-seqs.qza \
     --o-denoising-stats denoising-stats.qza \
     --verbose &> dada2.log
```
Except for `--p-trunc-q 5` all options are essential for DADA2. The truncation options for the reversed reads will be more a lower bases, because the quality drops mostly before the foreward reads. 
The command generates three output files.

 First lets have a look at the `denoising-stats.qza` by running the following command:

``` {bash}
qiime metadata tabulate \
    --m-input-file denoising-stats.qza \
    --o-visualization denoising-stats.qzv
```
There is also an export function in the tool package for exporting data in i.e. `.tsv` format. 
``` {bash}
qiime tools export \
    --input-path denoising-stats.qzv \
    --output-path denoising-stats
```
For each sample-id the following stats are available:

+ input: absoulute no. of input reads
+ filtered: absoulute reads filtered vie --p-options
+ percentage of input passed filter: in %
+ denoised: absoulute no. denoised reads
+ merged: absoulute no. reads left for merging
+ percentage of input merged: in %
+ non-chimeric: absoulute no. reads left from previous filter which are non-chimeric 
+ percentage of input non-chimeric: in %

Based on certain criteria, DADA2 declares representative sequences for which a `Feature ID`, the `Sequence length` and the actual `Sequence` is denotetd. To visualize the representative sequences output:
```{bash}
qiime feature-table tabulate-seqs \
    --i-data rep-seqs.qza \
    --o-visualization rep-seqs.qzv
```
Beside some general statistics, by clicking on the sequence, the sequence will be aligned using BLAST algorithm vie NCBI.

The last output file is a feature count table, which is a very important table for further analysis. It is a two dimensional table with samples as rows and features as columns.  

| Sample  | Feature_1 | Feature_2 | [...] |
| :--- | :--- | :--- | :--- |
| Sample_1 | 120 | 50 | [...] |
| Sample_2 | 20 | 200 | [...] |

The features are represented by the rep-seqs.qza were all sequences are stored in, which incorporate the *unique* bacteria (feature). 

## Taxonomy assignment

Now a part of the preprocessing is done. The `dada2_table` is somehow our key file to work with. One of the next steps is to taxonomacally classify the ASVs. By doing that, we use the [SILVA](https://www.arb-silva.de/) databasa for classifying our ASVs.  

The database is downloaded via the following command from QIIME2 website. This downloads the reference sequences for our reps-seqs `silva-138-99-seqs.qza` to classify and the corresponding taxonomy information `silva-138-99-tax.qza`.

``` {bash}
wget https://data.qiime2.org/2020.8/common/silva-138-99-seqs.qza
wget https://data.qiime2.org/2020.8/common/silva-138-99-tax.qza
``` 
If you work on our remote maschine your can find those files in the following directory:

``` {bash}
ls /db/silva/
```
or using the `$SILVA_DB` variable.
``` {bash}
ls $SILVA_DB
```
### Extract reference sequences

The following function `extract reads`from the `feature-classifier`package trims the silva-seq around the givin primer sequences (note that these are the same oligos used in cutadapt) for 16S rRNA V3/V4 region. Additionally, max and min lenght are given. We output the trimmed reference sequences in our qiime_files directory.

``` {bash}
qiime feature-classifier extract-reads \
    --i-sequences $SILVA_DB/silva-138-99-seqs.qza \
    --p-f-primer CCTACGGGAGGCAGCAG \
    --p-r-primer GGACTACHVGGGTWTCTAAT \
    --p-max-length 505 \
    --p-min-length 250 \
    --o-reads ref-seqs.qza
```