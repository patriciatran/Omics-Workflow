# Viral Metagenomics Workflow

Patricia Tran, with advice from Kris Kieft
Notes and Instructions on how to go analyze Viral Metagenomes

## Read trimming & assembly

Start with trimmed and assembled reads . For each set of reads (.fasta) format run vibrant:
For this tutorial, I'm using a MG from Lake Mendota's time series:
`/slowdata/data2/Lake_Mendota_Patricia/Virus_workflow/Assembly/3300020480.a.fna`

## Phage annotations

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

One we have the VIBRANT results, we want to run CheckV to assess quality on that file : 3300020480.a.phages_combined.fna

To run CheckV:
`/slowdata/data2/Lake_Mendota_Patricia/Virus_workflow$ /home/kieft/anaconda3/bin/checkv end_to_end /slowdata/data2/Lake_Mendota_Patricia/Virus_workflow/VIBRANT/VIBRANT_3300020480.a/VIBRANT_phages_3300020480.a/3300020480.a.phages_combined.fna CheckV/ -t 10`

## Dereplication


# Additional Analyses
## Taxonomy

## Coverage across time

