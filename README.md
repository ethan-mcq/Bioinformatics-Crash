# Bioinformatics - Crash Course
## Learning Objectives
By the end of today's course, you will understand the following:
 - What is nanopore sequencing?
 - What does raw read data look like?
 - What are important file formats as a bioinformatician?
 - How can I basecall, align, and interpret sequencing results from raw read data?

## Introduction
Bioinformatics is often a 'black box' of information where there are many things to learn and unpack, even as a bioinformatician! There are so many facets to the field of bioinformatics, everything from protein analytics, to spatial transcripomics, to metagenomic analysis, to pipeline engineering, there is no end to what you can learn. However, having at least some understanding of the field and how to learn field-specific tools can hopefully provide a foundation for continued learning. Let's get started!

## What is nanopore sequencing?
Nanopore seqencing relies on passing a single strand of DNA through a nanometer-scale pore, typically a biological protein pore, and measuring the changes in electrical current as the nucleotides move through the pore. The following steps and graphic outline how seqeucnign works at a surface level:
1. Preparation of the sample: The DNA or RNA sample is typically fragmented into smaller pieces before sequencing.
2. Passage through nanopores: A nanopore is embedded in a membrane, and an electrical potential is applied across this membrane. As the DNA or RNA molecule passes through the nanopore, the changes in electrical current are recorded.
3. Current modulation: Each nucleotide has a distinct size and shape, leading to different current modulations as they pass through the pore. By analyzing these changes in current, the sequence of the DNA or RNA molecule can be determined.

![alt text](https://github.com/ethan-mcq/Bioinformatics-Crash/blob/main/images/41587_2021_1108_Fig1_HTML.png?raw=true)

Nanopore sequencing has several advantages:
1. Long reads: It can produce long sequence reads, which is beneficial for assembling complex genomes and resolving repetitive regions.
2. Real-time sequencing: The sequencing process is real-time, meaning that data can be generated as the DNA or RNA passes through the nanopore.
3. Single-molecule sequencing: Each molecule is sequenced individually, which can provide more accurate information compared to some other sequencing methods.

## Important file formats for today
### POD5 / FAST5
POD5 files store raw read data that is generated by the sequencing machine. This file is not super useful until after basecalling, or interpreting the signal data into bases. 

POD5 = Newer

FAST5 = Older

### SAM / BAM
A SAM file, which stands for Sequence Alignment/Map format, is a standard plain-text format in bioinformatics used to store nucleotide sequence alignment information. SAM files are often generated by DNA or RNA sequencing technologies and are used to represent the alignment of sequence reads to a reference genome. BAM files are the compressed, or smaller version of SAM files. See the link for more information

[SAM Formatting](https://samtools.github.io/hts-specs/SAMv1.pdf)

![alt text](https://github.com/ethan-mcq/Bioinformatics-Crash/blob/main/images/sam.png?raw=true)

### BED 
A BED file in bioinformatics is a widely used file format for representing genomic region data. The name "BED" stands for Browser Extensible Data. This format is simple and flexible, making it easy to use for storing and sharing genomic annotations.

A BED file typically contains information about genomic features such as genes, transcripts, regulatory elements, or other functional elements. Each line in the BED file represents a specific genomic region and includes several columns of information. The first three columns are mandatory, and additional columns can be included for additional details.

[BED Formatting](https://useast.ensembl.org/info/website/upload/bed.html)

![alt text](https://github.com/ethan-mcq/Bioinformatics-Crash/blob/main/images/BED.png?raw=true)

### FASTA
The FASTA format is a simple and widely used text-based file format in bioinformatics for representing nucleotide or protein sequences. It is named after the software package "FASTA," which introduced this format. FASTA files typically have a .fasta or .fa file extension.

A FASTA file consists of one or more sequence entries, where each entry has two parts: Header Line (starting with '>') and Sequence Data

![alt text](https://github.com/ethan-mcq/Bioinformatics-Crash/blob/main/images/fasta.png?raw=true)

### VCF
A VCF (Variant Call Format) file is a standard file format in bioinformatics used to store information about genetic variants discovered during DNA sequencing. Genetic variants can include single nucleotide polymorphisms (SNPs), insertions, deletions, and structural variations. VCF files are commonly used in the analysis and sharing of genomic variation data.

[VCF Formatting](https://samtools.github.io/hts-specs/VCFv4.2.pdf)

![alt text](https://github.com/ethan-mcq/Bioinformatics-Crash/blob/main/images/VCF.png?raw=true)

## Basecalling
[Unix Learning Tool](https://swcarpentry.github.io/shell-novice/)

Here we will install all of the necessary hardware to basecall on your own machine. We will be using Oxford Nanopore's open source basecalling software, Dorado. Continue on the link below and click on your operating system to download Dorado. This link can also be used later to access Dorado's in-depth documentation. 

[Dorado](https://github.com/nanoporetech/dorado/tree/master#installation)

Once that has been downloaded, we will get it set up to easily use in your terminal. Open the terminal and enter in these commands.
```
export PATH="/path/to/dorado/bin:$PATH"
dorado
```
You should see this pop up: 
```
Usage: dorado [options] subcommand

Positional arguments:
aligner
basecaller
demux
download
duplex
summary

Optional arguments:
-h --help               shows help message and exits
-v --version            prints version information and exits
-vv                     prints verbose version information and exits
```
Now we want to download a basecalling model. These models are ML models that are pre-trained to detect certain patterns in the raw read data to determine what base each signal is. We can download all possible models by entering the following: 
```
dorado download --model all
```
However to cut down storage, we will just download the following model, which works with methylation and should fit all needs in the lab. 
```
dorado download --model dna_r10.4.1_e8.2_400bps_hac@v4.2.0
```
Now we are ready to begin basecalling! We can do this by entering this command
```
dorado basecaller dna_r10.4.1_e8.2_400bps_hac@v4.1.0 pod5s/ > calls.bam
```
pod5s/ will be replaced with the directory of your POD5 files and calls.bam will need to be pointing to the correct output directory. 

### Alignment
Alignment can also be done with Dorado, as the Nanopore aligner [minimap2](https://github.com/lh3/minimap2) is integrated. 
First, we need to get our reference genome. You will need to find whatever suits your needs best, but we will use HG38, as that is the current human genome standard. [Here](https://drive.google.com/file/d/12pGDaFliSzbo37eag6slsnrK6HcqaOv2/view?usp=sharing) is a download link for the HG38 reference genome I've saved for us. 
Second, align!
```
dorado aligner <index> <reads> > calls.aln.bam
```
index is the location of your reference genome and reads will be your unalinged BAM file we created while basecalling. Now you should have a basecalled, aligned BAM file filled with your sequence data!

### samtools
[Samtools Documentation](http://www.htslib.org/doc/samtools.html)

Samtools is a suite of programs for interacting with high-throughput sequencing data in the SAM (Sequence Alignment/Map) and BAM (Binary Alignment/Map) formats. These formats are commonly used in bioinformatics for representing sequence alignment information, especially in the context of DNA and RNA sequencing data.

Samtools provides a set of command-line utilities that enable users to perform various operations on SAM/BAM files, such as:

1. Converting between SAM and BAM formats: Samtools allows users to convert files between the human-readable SAM format and the more compact and efficient binary BAM format. This conversion is useful for both storage and processing efficiency.
2. Sorting and indexing: Samtools can sort the alignment records in a BAM file by genomic coordinates and create an index file, which facilitates faster retrieval of specific genomic regions. This is essential for many downstream analyses.
3. Viewing and filtering: Samtools provides commands to view the contents of SAM/BAM files, filter records based on criteria like mapping quality or read flags, and extract specific genomic regions.
4. Merging and comparing: Samtools can merge multiple BAM files into a single file, compare multiple sorted BAM files, and perform other operations that involve multiple samples or datasets.
5. Calculating statistics: Samtools includes tools for calculating various statistics, such as coverage, depth, and other metrics related to the alignment data.

Lets get a [package manager](https://en.wikipedia.org/wiki/Package_manager) that we can install samtools into. The most versatile one is [condas](https://docs.conda.io/en/latest/).

[Download miniconda](https://docs.conda.io/projects/miniconda/en/latest/)
```
# This filename must be edited to match that of your downloaded file
bash Miniconda3-latest.sh
```
Reload your terminal

Now condas should be downloaded to your system and you should notice '(base)' at the beginning of your terminal line:
```
(base) Ethans-MacBook-Air-124:~
```
run the command ```conda``` and you should see the following output:
```
usage: conda [-h] [--no-plugins] [-V] COMMAND ...

conda is a tool for managing and deploying applications, environments and packages.

options:
  -h, --help          Show this help message and exit.
  --no-plugins        Disable all plugins that are not built into conda.
  -V, --version       Show the conda version number and exit.

commands:
  The following built-in and plugins subcommands are available.

  COMMAND
    clean             Remove unused packages and caches.
    compare           Compare packages between conda environments.
    config            Modify configuration values in .condarc.
    content-trust     See `conda content-trust --help`.
    create            Create a new conda environment from a list of specified
                      packages.
    doctor            Display a health report for your environment.
    env               See `conda env --help`.
    info              Display information about current conda install.
    init              Initialize conda for shell interaction.
    install           Install a list of packages into a specified conda
                      environment.
    list              List installed packages in a conda environment.
    notices           Retrieve latest channel notifications.
    package           Create low-level conda packages. (EXPERIMENTAL)
    remove (uninstall)
                      Remove a list of packages from a specified conda
                      environment.
    rename            Rename an existing environment.
    run               Run an executable in a conda environment.
    search            Search for packages and display associated information
                      using the MatchSpec format.
    update (upgrade)  Update conda packages to the latest compatible version.
```
Now that we have verified that condas is working, lets make a samtools environment. 
```
conda create --name samtools -c bioconda samtools
```
Select ```y``` to all options and voila! Activate the samtools environment with the command:
```
conda activate samtools
```
Now your terminal line should look like this, notice how it says samtools instead of '(base)': 
```
(samtools) Ethans-MacBook-Air-124:~
```
Verify samtools is installed by running ```samtools``` where you should see something like: 
```

Program: samtools (Tools for alignments in the SAM format)
Version: 1.17 (using htslib 1.17)

Usage:   samtools <command> [options]

Commands:
  -- Indexing
     dict           create a sequence dictionary file
     faidx          index/extract FASTA
     fqidx          index/extract FASTQ
     index          index alignment

  -- Editing
     calmd          recalculate MD/NM tags and '=' bases
     fixmate        fix mate information
     reheader       replace BAM header
     targetcut      cut fosmid regions (for fosmid pool only)
     addreplacerg   adds or replaces RG tags
     markdup        mark duplicates
     ampliconclip   clip oligos from the end of reads

  -- File operations
     collate        shuffle and group alignments by name
     cat            concatenate BAMs
     consensus      produce a consensus Pileup/FASTA/FASTQ
     merge          merge sorted alignments
     mpileup        multi-way pileup
     sort           sort alignment file
     split          splits a file by read group
     quickcheck     quickly check if SAM/BAM/CRAM file appears intact
     fastq          converts a BAM to a FASTQ
     fasta          converts a BAM to a FASTA
     import         Converts FASTA or FASTQ files to SAM/BAM/CRAM
     reference      Generates a reference from aligned data
     reset          Reverts aligner changes in reads

  -- Statistics
     bedcov         read depth per BED region
     coverage       alignment depth and percent coverage
     depth          compute the depth
     flagstat       simple stats
     idxstats       BAM index stats
     cram-size      list CRAM Content-ID and Data-Series sizes
     phase          phase heterozygotes
     stats          generate stats (former bamcheck)
     ampliconstats  generate amplicon specific stats

  -- Viewing
     flags          explain BAM flags
     head           header viewer
     tview          text alignment viewer
     view           SAM<->BAM<->CRAM conversion
     depad          convert padded BAM to unpadded BAM
     samples        list the samples in a set of SAM/BAM/CRAM files

  -- Misc
     help [cmd]     display this help message or help for [cmd]
     version        detailed version information
```
#### Basic samtools commands
Convert BAM to SAM 
```
samtools view -h -o file.sam file.bam
```
Convert SAM to BAM
```
samtools view -bS file.sam > file.bam
```
Extract number of reads
```
samtools view -c
```
Sort aligned BAM file
```
samtools sort file.bam > file.sorted.bam
```
Index sorted BAM file
```
samtools index file.sorted.bam
```
Extract average read length
```
samtools view -h your_file.bam | awk '$1 !~ /^@/ {print length($10)}' | awk '{sum += $1} END {if (NR > 0) print sum / NR}'
```
