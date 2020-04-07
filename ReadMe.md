<!-- [![AAFC](https://avatars1.githubusercontent.com/u/4284691?v=3&s=200)](http://www.agr.gc.ca/eng/home/)-->

# Paired End Genotype By Sequencing (GBS) Pipeline

> A bioinformatics package that includes worflows to generate bams and a vcf file from multiplexed GBS paired end fastq files.
  
> This tool utilizes the workflow management system [Snakemake](https://snakemake.readthedocs.io/en/stable/) and allows you to accomplish a complete pipeline process with just a few command line calls, even on a computing cluster.

An Example Workflow:

![workflow](workflow/resources/images/BAM_workflow.jpg?raw=true "Workflow")

## Outline

- Generate GBS data from Paired-End Fastq's generated by Illumina sequencing technologies
- Split multiplexed libraries into separate fastq's for each sample (read 1 and read 2) for in-line barcoded libraries.
- Process GBS libraries that are made of different species, or samples that require different reference genomes.
- End to end processing with only 2 command line calls to GridEngine. One for producing BAMs, one for producing VCF's
- Designed to run on high performance clusters
- Custom iGenomics Riptide GBS library processing
- Ease of use - you need to only provide the tool with paths to a sample sheet, barcode file and a reference genome


## Table of Contents
- [Quick Use Guide](#quick-use-guide)
- [Installation](#installation)
- [Usage](#Usage)
- [Running the pipeline](#Executing)
- [Features](#features)
- [Team](#team)
- [License](#license)

## Quick Use Guide
> This is assuming you have followed the installation.  If not, proceed first to [installations](#installation).

> Once your environment is set up you can start running the pipeline!
>Start at the beginning with Generating BAMs, or if you have BAMs you want to process, move on to generating VCFs!

#### 1. Generating BAMs
Take paired end Fastq files (R1 and R2) and produce sorted BAM files and BAM index files
- input required:
	- `sample_R1.fastq.gz`
	- `sample_R2.fastq.gz` 
	- `samplesheet.txt` for format [see samplesheet](#Samplesheet) below
	- `barcode_file.txt` for format [see barcode file](#Barcode-file) below 
	- `reference_genome.fasta`

- update the config/config.yaml file.  Use absolute paths, path needs to be in quotations.
- sample_R1.fastq and sample_R2.fastq need to be the same name, only difference is R1 vs R2 (must be uppercase R1 and R2)!
```shell script
$  nano ~/gbs/GBS_snakemake_pipeline/config/config.txt

sample_read1: "/absolute/path/to/sample_R1.fastq.gz"
samplesheet: "/absolute/path/to/samplesheet.txt or samplesheet.csv"
barcodefile:  "workflow/resources/barcodes/barcodes_192.txt"
```
`cntrl-o` to save [enter]

`cntrl-x` to exit

- run the pipeline by submitting the process via shell script to the GridEngine queue

```shell script
$ qsub ~/gbs/GBS_snakemake_pipeline/workflow/resources/gbs.sh
```

- this will give you a job number if submitted properly.  The job will produce snakemake stats (which rule is running) in your current working directory and be named `gbs.sh.e(jobnumber)`
- you can check to make sure the program is running by typing `qstat -f`.  If not, check the `gbs.sh.e(jobnumber)` for why.
- runtime ~ 1 to 2 days for a standard output from Illumina HiSeq.  
- output of the tool will be in the directory where your fastq files were located. 

#### 2. Generating VCF
Take a list of BAM directories, and produce a single VCF file from all the BAM files included in those directories.
- input required:
    - directories of bam files (multiple directories allowed)
    - `reference_genome.fasta`

- update the config/config_vcf.yaml file for the 3 fields: bam files, outfile, and reference file.  Use absolute paths, paths need to be in quotations.
```shell script
$  nano ~/gbs/GBS_snakemake_pipeline/config/config_vcf.txt

sample_directories:
  - "/absolute/path/to/some/sorted_bams"
  - "/absolute/path/to/some/more/sorted_bams"
  - "/absolute/path/to/even/more/sorted_bams"

outfile: "/absolute/path/to/output/vcf/directory"

reference: "/absolute/path/to/reference_genome.fasta"
```
`cntrl-o` to save [enter]

`cntrl-x` to exit

- run the pipeline by submitting the process via shell script to the gridengine queue

```shell script
$ qsub ~/gbs/GBS_snakemake_pipeline/workflow/resources/vcf.sh
```

- This will give you a job number if submitted properly.
- You can check to make sure the program is running by typing `qstat`.  Look for `vcf.sh` in the list.  If not, check the `vcf.sh.e(jobnumber)` for why.
- Final output will be in the folder you designated in the config_vcf.yaml file

## Installation
> download the repository using `git clone` or by clicking `clone or download` on the main page of the repository
and dowloading the zip file.  
- in your home directory on the cluster/local computer create a directory for the pipeline to live called `gbs` 
```shell script
$ mkdir ~/gbs & cd ~/gbs/
```

#### Cloning the repository to your home directory in cluster 
> you will need git installed on your cluster account to accomplish this.  Can be done using `conda install -c anaconda git` on your cluster account.

- Use the `git clone` process to make an exact copy of the current repository
- clone this repository to your local machine using the following command:

```shell script
$ git clone https://github.com/elderberry-smells/GBS_snakemake_pipeline.git
```
- press `enter`, your local clone should be created with the following pathways for the pipeline:

    - `~/gbs/GBS_snakemake_pipeline/snakefiles`
    - `~/gbs/GBS_snakemake_pipeline/config/`
    - `~/gbs/GBS_snakemake_pipeline/workflow/envs/`    
    - `~/gbs/GBS_snakemake_pipeline/workflow/rules/`
    - `~/gbs/GBS_snakemake_pipeline/workflow/scripts/`
    - `~/gbs/GBS_snakemake_pipeline/workflow/resources/`
 
#### Installing the GBS snakemake environment on your computer
- Create an environment for snakemake using the `workflow/envs/gbs.yaml` file from the repo you downloaded, then activate it.
```shell script
$ conda env create -f ~/gbs/GBS_snakemake_pipeline/workflow/envs/gbs.yaml
$ conda activate gbs
```
when that is done running, you should see `(gbs)` on the left side of your command line


#### bioinformatics dependencies
- all of these dependencies should be installed from the environment being created except for novosort

`python=3.6`
`snakemake=4.0`
`trimmomatic=0.39`
`bwa=0.7.17`
`samtools=1.9`
`bcftools=1.8`
`Novosort=2.00.00`

- move the novosort folder to your newly created gbs environment bin
    
    ```shell script
    $ mv GBS_snakemake_pipeline/workflow/resources/novocraft/ ~/miniconda3/envs/gbs/bin/
    ```
      
    - add that folder to your profile so novosort is a callable command
    
    ```shell script
    $ nano ~/.bashrc 
    ```
    - add the path to the novocraft folder to the bottom of the file, save, and exit.  Change the path below to the correct one on your machine
    ```shell script
    export PATH="$PATH:~/miniconda3/envs/gbs/bin/novocraft"
    ``` 

## Usage  
A more detailed guide to running the program

### BAM Generation

> The snakemake tool is structured in a way that you need only activate the environment and run the snakefile  after you update the `config/config.yaml` to direct the program.

### Config file
The config file is 3 lines, all required inputs for the snakefile to work properly

```
sample_read1: "/absolute/path/to/sample_R1.fastq.gz"
samplesheet: "/absolute/path/to/samplesheet.txt"
barcodefile:  "workflow/resources/barcodes/barcodes_192.txt"
```

### Samplesheet
The sample sheet is a tab delimited txt file or a csv file with 4 columns.  Your sample sheet must be formatted in same way for this to work.

The samples can have any number of different references, each sample can even have a different genome if required.  An example of a samplesheet with 2 references is shown below:

```
Sample_number	Index_name	Sample_ID   Reference_path
1	gbsx001	Sample1	    /absolute/path/to/reference1/reference_genome1.fasta
2	gbsx002	Sample2     /absolute/path/to/reference1/reference_genome1.fasta
3	gbsx003	Sample3	    /absolute/path/to/reference1/reference_genome1.fasta
4	gbsx004	Sample4	    /absolute/path/to/reference1/reference_genome1.fasta
5	gbsx005	Sample5	    /absolute/path/to/reference1/reference_genome1.fasta
6	gbsx006	Sample6	    /absolute/path/to/reference2/reference_genome2.fasta
7	gbsx007	Sample7	    /absolute/path/to/reference2/reference_genome2.fasta
8	gbsx008	Sample8	    /absolute/path/to/reference2/reference_genome2.fasta
9	gbsx009	Sample9     /absolute/path/to/reference2/reference_genome2.fasta
10	gbsx010	Sample10    /absolute/path/to/reference2/reference_genome2.fasta
```
### Barcode file
The barcode file, as seen in the barcode folder `workflow/resources/barcodes/` are 2 columns, tab delimited txt files.  This file does not have any headers

`col1: index_name` `col2: barcode`

example barcode file:

```
gbsx001	TGACGCCATGCA
gbsx002	CAGATATGCA
gbsx003	GAAGTGTGCA
gbsx004	TAGCGGATTGCA
gbsx005	TATTCGCATTGCA
gbsx006	ATAGATTGCA
gbsx007	CCGAACATGCA
gbsx008	GGAAGACATTGCA
gbsx009	GGCTTATGCA
gbsx010	AACGCACATTTGCA
```

## Executing

### Testing to see if the pipeline is working

Update the `config/config.yaml` file to direct the pipeline to the resources it needs.


Activate the GBS pipeline and change directory to where the snakefile is located  
```shell script
$ conda activate gbs
$ cd ~/gbs/GBS_snakemake_pipeline/
```
Invoke the snakemake process in test mode to see if outputs are being called correctly.
```shell script
$ snakemake -np
```
the snakemake process should show you all it intends to do with the samples located in the samplesheet, and how many process for each module/rule will run.  

example:

```
Job counts:
	count	jobs
	1	all
	184	bwa_map
	1	demultiplex
	184	novosort
	184	trimmomatic
        1       fastqc
        1       multiqc
	555
```

- you *can* run the program with requested cores on local machine or interactive node on cluster, although this isn't recommended on large data sets (use nohup in case you need to close terminal or interactive sessions have a time limit) 
```shell script
$ nohup snakemake -j 16
``` 
would run the program using 
###  Running the program on the cluster with qsub using shell scripts 

`workflow/resources/gbs.sh` for BAM generation
`workflow/resources/vcf.sh` for VCF generation

- go back to the [quick use guide](#quick-use-guide) for instructions on running this on gridengine


#### Features
Brief documentation on each step in this pipeline can be found in the [Features ReadMe](workflow/resources/Features.md)

# Team
- Author:  Brian James (brian.james4@canada.ca)
- Testing and Improvements:  Dr. Jana Ebersbach (jana.ebersbach@canada.ca)
- AAFC Scientist: Dr. Isobel Parkin (isobel.parkin@canada.ca)


## License
[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)
