# Vc_2023training <!-- omit in toc -->
Here you'll find all the commands necessary to solve the *Vibrio cholerae* command line exercise. Let's start!

## Objectives <!-- omit in toc -->
1) Determine the quality of the sequenced reads
2) Assemble and annotate your genomes
3) Look for AMR and virulence determinants
4) Determine serogroup and serotype
5) Determine Vc lineage and sublineage

```
Each one of you will be working in a user of ther server named "alu0X", so whenever you find "alu0X" in a command, change it to the correct name of the user you'll be working with.
```
## Exercise <!-- omit in toc -->
First, enter the working directory where we will be working throughout the whole course

Type:
```
cd Vc_training2023
```

Let's see what's inside the folder, type:

```
ls
```

You can see you have different fastq files. We are interested in checking the quality of these reads, type:

```
mkdir fastqc_raw

fastqc -t 8 -o fastqc_raw *.fastq.gz

mkdir kraken2

for f in *_1.fastq.gz; do kraken2 -db /mnt/Netapp/KRKDB/KRKDB_st8 --threads 8 --gzip-compressed --paired --report kraken2/${f%_1.fastq.gz}_kraken2.txt --use-names $f ${f%_1.fastq.gz}_2.fastq.gz; done
```

To see the results of these commands `cd` to the different folders you've created previously. 

> **Remember**: once you've enter a new directory, you can go to the previous one by typing: `cd ..`

Now, we want to trim our fastq file by quality, so type:



