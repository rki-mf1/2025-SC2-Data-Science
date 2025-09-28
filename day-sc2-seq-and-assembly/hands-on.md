# Workshop: SARS-CoV-2 genome reconstruction from Illumina and ONT data

## In general

Here we describe basic tools and commands to process either Illumina or Nanopore sequencing data using SARS-CoV-2 amplification data as an example. The goal is to construct a genome sequence using a reference-based approach. **Although the methodological steps are quite similar, both Illumina and Nanopore need different tools and parameters.**

**Example data can be found here (click on "Files"): [https://osf.io/9qkz5/](https://osf.io/9qkz5/)** and is also referenced in the used commands again. 

## Create a folder for the hands-on work (maybe you already have one from yesterday)

Below are just example paths, you can also adjust them and use other folder names! Assuming you are on a Linux system on a local machine (laptop, workstation):

```sh
# Switch to a path on your system where you want to store your data and results
# Check that you have enough space!
cd ~
# Create new folder
mkdir workshop
cd workshop
```

## Basic setup

In this part, we will go through the different steps to generate from a FASTQ raw sequencing data file a SARS-CoV-2 consensus genome and lineage annotation. We will do this using the Linux system and command line interface. 

The tools we will use are listed here and will be discussed in detail below:

__QC__
* `FastQC` (Illumina)
* `fastp` (Illumina)
* `NanoPlot` (Nanopore)
* `Filtlong` (Nanopore)
* `BAMclipper` (Illumina & Nanopore)

__Mapping__
* `minimap2` (Illumina & Nanopore)
* `SAMtools` (Illumina & Nanopore)
* `IGV` (Illumina & Nanopore)

__Variant calling & Consensus__
* `Medaka` (Nanopore)
* `freebayes` (Illumina)
* `BCFtools` (Illumina & Nanopore)

__Lineage annotation__
* `Pangolin` (Illumina & Nanopore)

In addition, we install but will not directly use:

* `PycoQC`
* `SNPeff`
* `president`

These tools are handy if you want to perform quality-control directly on the raw signal FAST5 files (`PycoQC`), of you want to annotate variant calls for example to know which amino acid is changed (`SNPeff`), and to perform a basic quality-control of your final consensus sequence (`president`).

In general, you can Google all of these tools and find further information in publications, GitHub code repositories and on (Bio)conda. 

__Install all tools **IF NOT ALREADY DONE**__
Because this is a quite heavy environment we will use `mamba` as an updated version of `conda` to faster resolve the environment and install all tools.

```bash
# config some channels, this might be already done.
# basically helps to not explicitly type the channels
conda config --add channels default
conda config --add channels bioconda
conda config --add channels conda-forge

# now we create a new environment and install all tools
mamba create -y -p envs/workshop fastqc fastp nanoplot filtlong minimap2 samtools bcftools igv pangolin president snpeff bamclipper freebayes
conda activate envs/workshop
```
_Note_: We skip `medaka` here because the tools has some conflicting dependencies with other tools. Thus, we will use `medaka` later in a separate `mamba` environment. 

_Note2_: It might be also more convenient to have separate environments for each tool or to summarize tools per task (e.g. mapping) in one environment. Like it would be also best practice using a Workflow Management System. You can also do that if you want and then switch between environments. 


## Example input data

You can obtain example read files for Illumina and Nanopore [here](https://osf.io/9qkz5) or use the commands below:

```bash
# Illumina
wget --no-check-certificate https://osf.io/yxep6/download -O SARSCoV2-illumina.R1.fastq.gz
wget --no-check-certificate https://osf.io/qvzsh/download -O SARSCoV2-illumina.R2.fastq.gz

# Nanopore (R9 data) - we will use this as example below
wget --no-check-certificate https://osf.io/k9px6/download -O SARSCoV2-nanopore.fastq.gz

# Nanopore (R10 data) - newer data, less errors? You could adjust the commands below and also analyze this sample! 
wget --no-check-certificate https://osf.io/ge38p/download -O SARSCoV2-nanopore-R10.fastq.gz
```

**ATTENTION**: here we set now variables to point to the example data sets so we can just use that in the following commands. Adjust accordingly for your system and folder structure. 
```bash
ILLUMINA_SAMPLE1='SARSCoV2-illumina.R1.fastq.gz'
ILLUMINA_SAMPLE2='SARSCoV2-illumina.R2.fastq.gz'
NANOPORE_SAMPLE='SARSCoV2-nanopore.fastq.gz'
```

## FASTQ quality control

### Illumina

* `FastQC`

__Quality assessment__
```bash
# activate the conda environment, assuming that your envs folder is located here (relative path)
conda activate envs/workshop

# always remember that almost all tools have a help page
fastqc --help

fastqc -t 4 $ILLUMINA_SAMPLE1 $ILLUMINA_SAMPLE2
```

Check the output HTML file. How does the quality look like? Anything strange? 

* `fastp`

Now, we want to quality-trim the Illumina reads to get rid of low-quality base calls especially at the end of reads. Then we check again the quality via `fastqc`. 

__Quality trimming__
```bash
fastp -i $ILLUMINA_SAMPLE1 -I $ILLUMINA_SAMPLE2 -o clean_reads.R1.fastq.gz -O clean_reads.R2.fastq.gz --thread 4 --qualified_quality_phred 20 --length_required 50

fastqc -t 2 clean_reads.R{1,2}.fastq.gz
```

How did the quality change? 

**ATTENTION:** From now on we work with the `clean_reads.R{1,2}.fastq.gz` data files! 

[Publication](https://academic.oup.com/bioinformatics/article/34/17/i884/5093234) | [Code](https://github.com/OpenGene/fastp)


### Nanopore

* `NanoPlot`

__Quality assessment__
```bash
# always remember that almost all tools have a help page
NanoPlot --help

# run NanoPlot on your FASTQ file
NanoPlot -t 4 --fastq $NANOPORE_SAMPLE -o nanoplot/raw 
    
# run NanoPlot on your FASTQ file with some more parameters
NanoPlot -t 4 --fastq $NANOPORE_SAMPLE --title "Raw reads" --color darkslategrey --N50 --loglength -o nanoplot/raw 
```
[Publication](https://academic.oup.com/bioinformatics/advance-article/doi/10.1093/bioinformatics/bty149/4934939) | [Code](https://github.com/wdecoster/NanoPlot)

__Length filtering__

* `filtlong`

```bash
# Attention! min and max length of course depend on your sequencing protocol! 
# the example nanopore reads were sequenced with the ARTIC V1200 kit and thus
# yield ~1.2kbp reads
filtlong --min_length 800 --max_length 1400 $NANOPORE_SAMPLE | gzip - > clean_reads_nanopore.fastq.gz

NanoPlot -t 4 --fastq clean_reads_nanopore.fastq.gz --title "Filtered reads" --color darkslategrey --N50 --loglength -o nanoplot/clean 
```
[Code](https://github.com/rrwick/Filtlong)

_Note_: `PycoQC` is another neat tool for QC and can be also used on raw signal FAST5 data, thus, providing even more detailed insights in the quality of your data. Please feel free to install and run the tool on your own using raw FAST5 data.


## Mapping & visualization

### Illumina & Nanopore

* `minimap2`
* `SAMtools`
* `IGV`

First, we need a reference genome (in FASTA format) to map against. We will use the _index reference sequence_ for SARS-CoV-2. You can download it manually from

* [NCBI, SARS-CoV-2 Index genome, MN908947.3](https://www.ncbi.nlm.nih.gov/nuccore/MN908947.3)

or for example via

```bash
wget --no-check-certificate https://osf.io/kt4dq/download -O nCoV-2019.reference.fasta
# we also download an index for later usage
wget --no-check-certificate https://osf.io/5eunp/download -O nCoV-2019.reference.fasta.fai
```

**ATTENTION**: In our example we will use `minimap2` for both Illumina and Nanopore data. However, `minimap2` was initially developed with long-read alignment in mind. There is a short-read mode, but it might be beneficial to also use specialized short-read-mapper such as [BWA-MEM](https://github.com/lh3/bwa).

__Mapping__
```bash
# map the filtered reads to the reference genome

# Illumina
minimap2 -x sr -t 4 -a -o minimap2-illumina.sam nCoV-2019.reference.fasta clean_reads.R1.fastq.gz clean_reads.R2.fastq.gz

# Nanopore
minimap2 -x map-ont -t 4 -a -o minimap2-nanopore.sam nCoV-2019.reference.fasta clean_reads_nanopore.fastq.gz
```
[Publication](https://doi.org/10.1093/bioinformatics/bty191) | [Code](https://github.com/lh3/minimap2) 

Please notice the different parameter settings used for Illumina and Nanopore data! Check the GitHub page of `minimap2` for more information. 

__Process mapping results__
```bash
# convert mapping results (SAM format) into a binary format 
# the binary format can be faster read by a machine but not by a human
# we will use the binary format (called BAM) for visualization of the mapping

# we will use a FOR loop to process both, Illumina and Nanopore data

for SAM in minimap2-illumina.sam minimap2-nanopore.sam; do

    # get the basename of the input SAM
    BN=$(basename $SAM .sam)

    # 1) convert SAM to BAM --> check the file size after convert!
    samtools view -bS $SAM > $BN.bam

    # 2) sort the BAM
    samtools sort $BN.bam > $BN.sorted.bam

    # 3) index the BAM
    samtools index $BN.sorted.bam
done
```
[Publication](https://academic.oup.com/bioinformatics/article/25/16/2078/204688) | [Code](https://github.com/samtools/samtools) 

**Hint:** You can also delete the `*.sam` files which can be huge. The `*.bam` files can be also always converted again into SAM. 

__Look at the mapped reads__
```bash
# start the Integrative Genomics Viewer (IGV)
igv &
# hint: if you still want to use your terminal although a tool with a 
# graphical interface is running, you can add a '&' to your command
# to move the running process in the background. By that, you can still
# use your terminal.
```

Now, we will load the reference FASTA (that we used for the mapping) and the sorted BAM file (with the mapped reads) into IGV to look at the results. The `samtools index` command is important so that IGV can actually load the sorted BAM file! As so often, a certain index structure is needed.  

* First, load the `nCoV-2019.reference.fasta` reference FASTA via "Genomes" > "Load Genome from File"
* Second, load the sorted BAM file via "File" > "Load from File"

**Hint**: You can load both the Illumina and Nanopore BAM file and compare them! How do they differ? 

__Additional resources__

* [SAM format specifications](https://samtools.github.io/hts-specs/SAMv1.pdf)


## Primer clipping

* `BAMclipper`

It is important to remove primer sequences from your amplicon reads. The primer sequences are used to amplify parts of the SARS-CoV-2 genome. But that also means that they are not actually part of your sequenced SARS-CoV-2 sample and thus might mask important mutations located in the primer binding region. Therefore, they should be removed, which you can for example do via `BAMclipper`. 

To accurately clip the primer sequences, we need to know where they are located. This is usually stored in a so-called BED file and can be found for various primer schemes [online](https://github.com/replikation/poreCov/tree/master/data/external_primer_schemes). 

### Illumina

```bash
# First, we download the primer BED scheme for Cleanplex scheme that was used
# Change to another BED file if needed!
wget --no-check-certificate https://osf.io/4nztj/download -O cleanplex.amplicons.bedpe

# It's important that the FASTA header of the reference genome 
# and the IDs in the BED file match, let's check:
head nCoV-2019.reference.fasta
head cleanplex.amplicons.bedpe

# we can see: they dont match! 
# In the reference FASTA: 'MN908947.3'
# In the BED file: 'NC_045512.2'
# So we need to replace the ID in the BED file, e.g. via
sed 's/NC_045512.2/MN908947.3/g' cleanplex.amplicons.bedpe > cleanplex-corrected.amplicons.bedpe

# check again
head nCoV-2019.reference.fasta
head cleanplex-corrected.amplicons.bedpe

bamclipper.sh -b minimap2-illumina.sorted.bam -p cleanplex-corrected.amplicons.bedpe -n 4
```

### Nanopore

Keep in mind to chose the correct primer file for your sample! 

```bash
# First, we download the primer BED scheme for the ARTIC V1200 scheme
# Change to another BED file if needed!
wget --no-check-certificate https://osf.io/3ks9b/download -O nCoV-2019.bed

# It's important that the FASTA header of the reference genome 
# and the IDs in the BED file match, let's check:
head nCoV-2019.reference.fasta
head nCoV-2019.bed

# now we convert this BED file into a BEDPE file needed by BAMclipper.
# The Illumina BED file we used above was already in the correct BEDPE format.
# we download a python script to do so:
wget --no-check-certificate https://osf.io/3295h/download -O primerbed2bedpe.py

# and run it
python primerbed2bedpe.py nCoV-2019.bed --forward_identifier _LEFT --reverse_identifier _RIGHT -o nCoV-2019.bedpe

# now we can use BAMclipper - finally
bamclipper.sh -b minimap2-nanopore.sorted.bam -p nCoV-2019.bedpe -n 4
```
[Publication](http://www.nature.com/articles/s41598-017-01703-6) | [Code](https://github.com/tommyau/bamclipper) 

__Task:__ Compare the unclipped BAM file with the clipped BAM file in IGV. Do you see differences? Hint: you can load multiple BAM files into IGV. 

* [Code snippets how the official ARTIC pipeline removes primers with a custom script](https://github.com/artic-network/fieldbioinformatics/blob/master/artic/minion.py#L191)

## Variant calling

### Illumina

For Illumina data we use `freebayes` to call variants. We installed the tool already in the Conda workshop environment. We also use some default parameter.

__Call variants with freebayes__

```bash
# first re-calulate the index for the reference FASTA (sometimes issues occur bc/ of different samtool versions used)
samtools faidx nCoV-2019.reference.fasta

# now variant calling
freebayes -f nCoV-2019.reference.fasta --min-alternate-count 10 \
--min-alternate-fraction 0.1 --min-coverage 20 --pooled-continuous \
--haplotype-length -1 minimap2-illumina.sorted.primerclipped.bam > freebayes-illumina.vcf
```
[Code](https://github.com/freebayes/freebayes)

The result is a **VCF** file, short for variant call format. A tab-separated file with information on the detected variants/ mutations. 

### Nanopore

We want to use `Medaka` for variant calling. `Medaka` is not in your current `workshop` environment because it was conflicting with the other tools. That's why we need a separate Mamba environment for `Medaka`:

* Make a new environment for `medaka` 
    * `medaka` might have many dependencies that conflict 
* an alternative to `conda` is `mamba`
    * `mamba` can be much faster in solving your environment, e.g. here for the tool `medaka`
    * thus, let us install `mamba` via `conda` and then install `medaka`

```bash
# check where you are currently located and then create the medaka env in the correct path! E.g. next to the `envs/workshop` env 
mamba create -y -p envs/medaka medaka
conda activate envs/medaka
```
[Code](https://github.com/nanoporetech/medaka) 

__Call variants with Medaka__

```bash
# first generate a file with information about potential variants
# considering the used basecalling model. You should use the matching
# model from your Dorado basecalling settings! Here we exemplarily use 
# older R9 data. For R10 data and different accuracies, you should change the model! 
medaka consensus --model r941_min_hac_g507 --threads 4 --chunk_len 800 --chunk_ovlp 400 minimap2-nanopore.sorted.primerclipped.bam medaka-nanopore.consensus.hdf

# actually call the variants
medaka variant nCoV-2019.reference.fasta medaka-nanopore.consensus.hdf medaka-nanopore.vcf

# annotate VCF with read depth info etc. so we can filter it
medaka tools annotate medaka-nanopore.vcf nCoV-2019.reference.fasta minimap2-nanopore.sorted.primerclipped.bam medaka-nanopore.annotate.vcf
```

__Important__: Always use the matching `medaka` model based on how you or others did the `Dorado` basecalling! You can check which `medaka` models are available via:
```bash
medaka tools list_models | grep -v Default
```

__Task__: Compare the results from the VCF file with what you can observe via the IGV browser. Can you find the variants `medaka` called also in IGV? 


## Consensus generation

### Illumina & Nanopore

__Check and filter the VCF file__

**ATTENTION**: We just do this here as an example using the Nanopore VCF. Just exchange the Nanopore annotated VCF input file with the Freebayes VCF file.

```bash
# switch to the workshop env if you are not already on it (assuimg a relative path to the envs/workshop folder)
conda activate envs/workshop

# compress the annotated VCF file (needed for the next steps)
bgzip -f medaka-nanopore.annotate.vcf
 
# index a TAB-delimited genome position file in bgz format 
# and create an index file
tabix -f -p vcf medaka-nanopore.annotate.vcf.gz

# generate the consensus
bcftools consensus -f nCoV-2019.reference.fasta medaka-nanopore.annotate.vcf.gz -o consensus-nanopore.fasta

# rename the consensus FASTA, right now the FASTA ID is still the reference
sed -i 's/MN908947.3/Consensus-Nanopore/g' consensus-nanopore.fasta
```
[Code](https://samtools.github.io/bcftools/) 

__Attention:__ The above commands yield a simple consensus sequence without all necessary quality checks! What did we not do so far?

* we did not properly filter the VCF file, e.g. for low-coverage variants
* we should also _mask_ low coverage regions in the consensus genome
  * for example the 5' and 3' end of the genome usually has low coverage 
  * but amplicon drop outs can also lead to low (or even no) coverage in the genome
* we did also not distinguish the two primer pools. It improves your accuracy to perform variant calling for each primer pool separately and then merge the resulting VCF files

## FURTHER READING BEYOND CONSENSUS

### Lineage annotation

#### Illumina & Nanopore

* `pangolin`

```bash
pangolin -t 4 consensus-nanopore.fasta
```
[Publication](https://academic.oup.com/ve/article/7/2/veab064/6315289) | [Code](https://github.com/cov-lineages/pangolin) 

__Task:__ Check the output. Which lineage was annotated? Which version of `pangolin` did you run? You can check this via `pangolin -v`. It is important to use the newest version of `pangolin` to also get results for newly defined lineages. You can update `pangolin` via `pangolin --update`. If you used an older version that was installed into the 'workshop' environment, try to create a new separate conda environment for `pangolin` and install the [newest version](https://anaconda.org/bioconda/pangolin). Rerun the lineage annotation. Are there differences? 

**Attention** If the conda/mamba installation of `pangolin` does not work, you can try using a conatiner:

```bash
# this will download the container if not already available
# start one interactive instance from it
# and mount the current directory into the container (using the same path)
docker run --rm -it -v $PWD:$PWD -w $PWD nanozoo/pangolin-v4:4.3--1.30 /bin/bash
# inside the container you can then run:
pangolin -t 4 consensus-nanopore.fasta
```

If you can not use `docker` (for example, bc you are on a HPC), then try `singularity`:

```bash
# this will convert the docker image to singularity and store it as a local file
singularity pull --name nanozoo-pangolin-v4-4.3--1.30.img docker://nanozoo/pangolin-v4:4.3--1.30
singularity run nanozoo-pangolin-v4-4.3--1.30.img
pangolin -t 4 consensus-nanopore.fasta
```

### Consensus QC

#### Illumina & Nanopore

* `president`

__Task:__ Read the manual page of [`president`](https://github.com/rki-mf1/president) and try to run the tool on your own to check the quality of your reconstructed consensus sequence. 
