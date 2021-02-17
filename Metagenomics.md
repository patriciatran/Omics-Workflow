## Bacterial/Archaeal Metagenome-assembled genomes Metagenomic Workflow

The following document shows the general steps of a metagenomics project. In this tutorial, we will go fom raw reads that we receive from the sequencer, to curated high-quality metagenome-assembled genomes (MAGs). I like to organize my projects by analyses for example, I'll have a folder with my raw data, then subsequent folders named after the analyses.

**Prerequisites**: You should be familiar with the terminal and basic command line scripting. For example: moving between directories `cd`, copying files `cp`, moving files or renaming them `mv`, listing the contents of a directory `ls`, creating new directories `mkdir`. Simple understanding of how to create a bash file `.sh`, running it `bash [filename].sh`, and using a text editor (e.g. `nano`) is recommended.

## Input files

The starting input files will look something like this: `[samplename].R1.fastq`, `[samplename].R2.fastq`

You can explore more information about your starting files using [samtools](http://www.htslib.org/).
To view the options that samtools have type:

```{}
$ samtools
```

### File Formats:

`.fasta` : This is a human-readable file format. It contains your nucleotide or amino acid sequences. It contains headers that start with `>` followed by the "header", and the next line being your sequences. 

`.fastq` : The fastq file is similar to the fasta file, except that is is binary (if you open the file you will see a bunch of symbols that are not human-understandable). It contains much more information encoded in it such as sequence quality, etc. 

`.fna` : A variation on the file extension fasta, and refer to fasta for nucleotide sequences.

`.faa` :A variation on the file extension fasta, and refer to fasta for amino acids

Notes: You can convert from a fastq file to a fasta file, but not from a fasta file to the raw fastq files.

## Quality Filtering of Reads

## Metagenomic Assembly
The step of metagenomic assembly means from going from the paired short reads into longer, "assembled" contigs.
Here, we will perform assembly using [MetaSpades](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5411777/) 

It is currently located here on the server: 
`/slowdata/archive/SPAdes-3.12.0-Linux/bin/spades.py`

Therefore, to view all the options type:
```{}
$ /slowdata/archive/SPAdes-3.12.0-Linux/bin/spades.py -help
```

To run **metaSPADEs** there is a flag called `--meta` to use.

To run metaSPADEs on your dataset do:
```{}
$ /slowdata/archive/SPAdes-3.12.0-Linux/bin/spades.py --meta -1 <[samplename].R1.fastq> -2 <[samplename].R2.fastq> -o <output directory>
```

### How do I assess the quality of my assembly?
Having a good assembly is important because it will influence all the downstream steps. However, which criteria to use to determine whether an assembly is "good" or could be improve?
One way to do it is through **Read mapping back to the assembly**.

#### Read Mapping to the assembly
The idea here is that if the assembly is "good" then most of our original (quality-filtered) reads will map back to it in some ways. For example, a low % of reads mapped back, is not as good as a high percentage such as >90%. 

We will use [bowtie2](http://bowtie-bio.sourceforge.net/bowtie2/) to map the reads to the assembly.
```{}
$ bowtie2 -h
```

#### What can I do if my read mapping is low?
There are solution to improve your assembly, such as running your assembler with different settings. 


## Metagenomic Binning
We will use 3 binning program to bin the metagenomes. The rationale for using several binning programs are to retrieve a maximum number of MAGs, as each binning program has pros and cons and differences in their algorithms. Obtaining the highest number of MAGs is useful because in the next step, we will dereplicate them (which means getting  the  best quality MAG among a group of closely related MAGs)

We will use [MaxBin](https://microbiomejournal.biomedcentral.com/articles/10.1186/2049-2618-2-26) , [MetaBat1](https://peerj.com/articles/1165/), [MetaBat2](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC6662567/).

We have a program (wrapper) called [Metawrap](https://github.com/bxlab/metaWRAP) installed. From there, we can run all three programs.

```{}
$ metawrap
$ mkdir Binning
```

### Running MaxBin
```{}
$ cd Binning
$ mkdir MaxBin
$ cd MaxBin
$ metawrap binning --maxbin2 -a <assemblyfile> -t <nb of threads> -m <amount of ram>
$ cd ../..
```

### Running MetaBat1
```{}
$ cd Binning
$ mkdir MetaBat1
$ cd MetaBat1
$ metawrap binning --metabat1 -a <assemblyfile> -t <nb of threads> -m <amount of ram>
$ cd ../..
```

### Running MetaBat2
```{}
$ cd Binning
$ mkdir MetaBat2
$ cd MetaBat2
$ metawrap binning --metabat2 -a <assemblyfile> -t <nb of threads> -m <amount of ram>
$ cd ../..
```

## Preliminary Quality Statistics

## First Pass-Through Dereplication
To dereplicate your MAGs, you can use [dRep](https://www.nature.com/articles/ismej2017126)

```{}
$ dRep -h
$ dRep dereplicate outout_directory -g path/to/genomes/*.fasta
```


## Taxonomic Classification with GTDB
[GTDB](https://github.com/Ecogenomics/GTDBTk) is a toolkit to assign taxonomy to MAG.
The setup for running GTDB can be a little complicated on our server. However, these instructions might be helpful.
First create a conda environment for GTDB. See this [link](https://ecogenomics.github.io/GTDBTk/installing/bioconda.html?highlight=export) as well

```{}
# latest version
$ conda create -n gtdbtk -c conda-forge -c bioconda gtdbtk
```

Now you have a conda environment named "gtdbtk"

You don't need to redownload the GTDB-tk database since we last updated in November 2020. It is located here on the server: `/slowdata/databases/GTDBTK_DB/release95`

Now we need to put this file path in our conda environment. To do so:
```{}
(base) patricia@sulfur:~$ conda activate gtdbtk
(gtdbtk) patricia@sulfur:~$ which gtdbtk
/home/patricia/.conda/envs/gtdbtk/bin/gtdbtk
```

This path here is important because if you `ls` into that folder, you will see a file named "gtdbtk.sh" under `/etc/conda/activate.d`. For example, mine is here: `/home/patricia/.conda/envs/gtdbtk/etc/conda/activate.d`. 

We will edit this file:
```{}
$ nano gtdbtk.sh
```
Type this in the file:
```{}
export GTDBTK_DATA_PATH="/slowdata/databases/GTDBTK_DB/release95"
```

Exit, save file.

Activate it:
```{}
$ source gtdbtk.sh
```

Now GTDB should be working: type:
```{}
$ gtdbtk
```

We are now ready to run the [classify_wf (Classify Workflow)](https://ecogenomics.github.io/GTDBTk/commands/classify_wf.html) on your MAGs. This will give you a table with the taxonomic classification of your MAGs, among other analyses.

```{}
$ gtdbtk classify_wf --genome_dir <my_genomes> --out_dir GTDB-output
```

Our results will be in the folder "GTDB-output".


Once you are done with running GTDB, you can return to your regular base environment by typing:
```{}
$ conda deactivate
```

## Manual Classification of MAGS using RP16
Another way to manually classify your MAGs is by doing a concatenated gene phylogeny using RP16 proteins. You can download the HMM here on my [GitHub page](https://github.com/patriciatran/LakeTanganyika/tree/master/rp16_HMM). 
You will need:
- [Prodigal](https://github.com/hyattpd/Prodigal) to convert the .fasta into .faa
- [hmmsearch](https://www.ebi.ac.uk/Tools/hmmer/search/hmmsearch).

### Prodigal
The first step is to run prodigal because hmm search will search against amino acid files, not nucleotide.
To run prodigal on your bins/MAGs:
```{}
# To view all options:
$ prodigal -h
# To run prodigal on one MAG:
$ prodigal -i <MAG_name.fasta> -a <MAG_name.faa>
```

You will have to run the prodigal command on _all_ your MAGs. You should end up with one faa file and for each MAG.

Now in anticipation for the hmmsearch, please take a look at the `.faa` file to ensure the MAG identifier is found in the fasta header. To do so type:
```{}
# which will print all the fasta headers from all the files with the file extension `.faa` in your dataset.
$ grep ">" *.faa 
# Once you are certain that each fasta header is unique and can relate to your MAG name, concatenate the .faa files:
$ cat *.faa >> concatenated_MAGs.faa
```

The two `>>` are important as opposed too `>` because `>>` means "append", which means things get added to the file, not overwrites it.


### HMMsearch
We will use HMMsearch, which is a program that uses Hidden Markhov Models on a `.hmm` file that is generated from curated and high quality sequences from your protein of interest.

** TIP: Here we made the rp16 .hmm from a curated dataset. There are many databases that make their hmm available online, such as Pfam, KEGG, etc. All you need to do is download their .hmm files in the same way and run hmmsearch with them. If you are studying a specific protein of interest, you can also make you own hmm from a folder of .fasta sequences you gathered yourself. You would use `hmm-build`**

If you are using our lab's server they are already available at the following path: `/slowdata/databases/rp16_HMM`
You will notice how some have the ending `arc.hmm` for Archea ,and `bact.hmm` for Bacteria.


```{}
## to view all the options
$ hmmsearch -h 
```

**important** : Please ensure that the >fasta headers contain information about the MAGs. This is the file `concatenated_MAGs.faa` you created in the previous section

The overall command to run the hmmsearch is:
```{}
$ hmmsearch --cut_tc -A <outputname_vs_RP#.HMM.sto> <path/to/hmm/file> concatenated_MAGs.faa
```
I find the most efficient way to run the search is to create a `.sh` bash script that I can run for all the hmm. Otherwise, the process is a bit repetitive.

You now have several files with the extension `.sto`. An `.sto` file can be reformated using the script `/slowdata/scripts/scripts/hmmer-3.2.1/easel/miniapps/esl-reformat` that is comes already with hmmsearch.

For **each of your .sto files**, convert from .sto to fasta using:
```{}
$ /slowdata/scripts/scripts/hmmer-3.2.1/easel/miniapps/esl-reformat fasta <outputname_vs_RP#.HMM.sto> > <outputname_vs_RP#.HMM.fasta>
```

You will now have several .fasta files. I prefer to organize my sequences in a Bacterial HMM folder output, and a Archaeal HMM folder output, which will simplify the organization of the tree later.

```{}
mkdir Arch
mkdir Bact
mv *arc.HMM.fasta > Arch/.
mv *bact.HMM.fasta > Bact/.
```

## Repeat for your reference genomes
When making a phylogenetic tree, you need your genomes in the context of more genomes. You can download references genomes as `.fasta` files from NCBI for example, or the primary litterature's data (usually found in a section called Data Availability). Follow the instruction of converting between fasta/fna to .faa using prodigal, and hmmsearch to identify the RP16. The final results should be a folder of RP16.fasta sequences (1 file per RP16) for your reference genomes.

## Exporting the data
You can use Cyberduck transfer the hmm.fasta files from the server on to your computer. I like to export the sequences out of the server at this point because I like to visualize the allignment. However, you can very well do the allignment from the server too.

**Time to Check**: It is very important that there is only 1 copy of each RP16 gene in **each** MAG, and not more. If you have less that one (e.g. a certain RP16 isn't found) that is ok, because this may due to genome completeness. However, if you have more than 1 copy, this will create problems during concatenation. Please see note in the Allignment section below on how to deal with these situation.

## Allignment
We will use the program MAFFT to allign the sequences.
