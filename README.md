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
### Quality control
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

Now, we want to trim our fastq files by quality as well as checking/removing Illumina adapter sequences, so type:

```
mkdir trimmed

for f in *_1.fastq.gz; do trim_galore -j 8 -q 30 --paired --fastqc --illumina -o trimmed $f ${f%_1.fastq.gz}_2.fastq.gz; done

```

Once trim_galore has finished, check the outputs. You should see that the new files are named something like this:

T_VC1_1_val_1.fq.gz

T_VC1_2_val_2.fq.gz

Let's change the names of these files, type:

```
cd trimmed

for f in *_1_val_1.fq.gz; do mv $f ${f%_1_val_1.fq.gz}_R1.fastq.gz; done

for f in *_2_val_2.fq.gz; do mv $f ${f%_2_val_2.fq.gz}_R2.fastq.gz; done
```

Now, we'll run a software named Kraken2 which matches each read to the lowest common ancestor (LCA) of all genomes, asigning specific taxa. Being in the "trimmed" directory, type: 

```
mkdir kraken2

for f in *_R1.fastq.gz; do kraken2 -db /mnt/Netapp/KRKDB/KRKDB_st8 --threads 8 --gzip-compressed --paired --report kraken2/${f%_R1.fastq.gz}_kraken2.txt --use-names $f ${f%_R1.fastq.gz}_R2.fastq.gz; done
```

### Assembly





