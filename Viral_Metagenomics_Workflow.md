# Viral Metagenomics Workflow

Patricia Tran, with advice from Kris Kieft
Notes and Instructions on how to go analyze Viral Metagenomes

## Read trimming & assembly

Start with trimmed and assembled reads . For each set of reads (.fasta) format run vibrant:
For this tutorial, I'm using a MG from Lake Mendota's time series:
`/slowdata/data2/Lake_Mendota_Patricia/Virus_workflow/Assembly/3300020480.a.fna`

## Phage annotations
Run [VIBRANT](https://microbiomejournal.biomedcentral.com/articles/10.1186/s40168-020-00867-0)

General command:
`/slowdata/data4/VIBRANT/VIBRANT_v1.2.1/VIBRANT_run.py -i assembly.fasta -t threads -folder output_folder`

If you want to run proteins, use -f , you will get less data back, but you can have more .
Command:
`/slowdata/data4/VIBRANT/VIBRANT_v1.2.1/VIBRANT_run.py -i Assembly/3300020480.a.fna -t 10 -folder VIBRANT/`

Output folder:
`/slowdata/data2/Lake_Mendota_Patricia/Virus_workflow/VIBRANT/VIBRANT_3300020480.a`

For scale, this took less than 5 minutes to run.

the identified phages are here: `3300020480.a.phages_combined.fna` which should be under the folder called VIBRANT_phages_... and the file ending with combined.fna

## Quality Checking

One we have the VIBRANT results, we want to run [CheckV](https://bitbucket.org/berkeleylab/CheckV) to assess quality on that file : 3300020480.a.phages_combined.fna
-d /slowdata/data4/checkv-db-v0.6/
To run CheckV:
`/home/kieft/anaconda3/bin/checkv end_to_end  /slowdata/data2/Lake_Mendota_Patricia/Virus_workflow/VIBRANT/VIBRANT_3300020480.a/VIBRANT_phages_3300020480.a/3300020480.a.phages_combined.fna CheckV/ -t 10 -d /slowdata/data4/checkv-db-v0.6/`

(make sure you have the permissions set up to access the `checkv-db-v0.6` folder

## Dereplication
We can dereplicate scaffolds using different programs, such as [dRep](https://github.com/MrOlm/drep) or [CD-HIT](http://weizhongli-lab.org/cd-hit/). The reason why we want to replicate is that if we have many of the same scaffolds identified, this will make the mapping less accurate. A read to match to either scaffolds. To get true coverage of each scaffold, it can be recommended to replicated first.

### dereplication using dRep
outout_directory is the name of the output folder, -g is the path of the folder containing the viral fasta sequencines (that is `/slowdata/data2/Lake_Mendota_Patricia/Virus_workflow/VIBRANT/VIBRANT_3300020480.a/VIBRANT_phages_3300020480.a/3300020480.a.phages_combined.fna` in the example above), and 
the --meta flag.

`dRep dereplicate outout_directory -g path/to/genomes/*.fasta --meta`

### dereplication using CD-HIT
To cluster nucleotide sequencing, use `cd-hit-est`. Instructions [here](https://github.com/weizhongli/cdhit/wiki/3.-User's-Guide#CDHITEST)
Since we are not working with paired reads, and that we are only working with one fasta file (rather than multiple), the general format will be:

`cd-hit-est -i est_human -o est_human95 -c 0.95 -n 10 -d 20 -M 16000 -T 8`

There -i is the input fasta sequence, the -o is the clustered output, -c is the sequence identity threshold (we suggest 0.97), -n is given the values 10 because -c is between 0.95 and 1.0, -d is the length of description in .clstr file (default 20, if 0 it cuts after the first space in the fasta file), -M is the memory, and -T is the number of threads. 

## Mapping (getting coverage values)
There are several methods to get abundance values (i.e. coverage, mapped reads, etc.), and several programs exist. Overall, the idea is to map the set of reads (trimmed, quality checked) against the fasta file (referred to as "database") of the scaffolds you want to get values for.




# Additional Analyses
## Taxonomy

## Coverage across time

