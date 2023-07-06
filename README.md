# Vc_2023training <!-- omit in toc -->
Here you'll find all the commands necessary to solve the *Vibrio cholerae* command line exercise. Let's start!

## Objectives <!-- omit in toc -->
1) Determine the quality of the sequenced reads
2) Assemble and annotate your genomes
3) Look for AMR and virulence determinants
4) Determine serogroup and serotype
5) Determine Vc lineage and sublineage

```
Each one of you will be working in a user of the server named "alu0X", so whenever you find "alu0X" in a command, change it to the correct name of the user you'll be working with.
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

We now want to obtain de novo assemblies for our genomes. As this process is computationally demanding, we will do it for only one of the samples. Being in the "trimmed" directory, type: 

```
unicycler -1 T_VC4_R1.fastq.gz -2 T_VC4_R2.fastq.gz -o assemblies/T_VC4_uni.out --verbosity 2 -t 8
```

Once it is over, type:

```
cd assemblies

for f in *.out/assembly.fasta; do cp $f ${f%_uni.out/assembly.fasta}.fasta; done
```

Let's check the quality of our newly obtained assembly, being in the assembly folder type:

```
quast.py -t 4 MUESTRA.fasta
```

### Annotation

We will annotate, the de novo assembly we created previously. Being in the "assemblies" folder, type:

```
prokka --prefix T_VC4 --outdir prokka/T_VC4.annotation --addgenes --mincontiglen 300 --cpus 4 T_VC4.fasta
```

To explore the files produced, enter 'cd' the "prokka" folder.

### Looking for AMR and virulence determinants

There are lots of softwares for this purpose but we will be using ariba (with resfinder and vfdb databases) and AMRFinderPlus. To start, being in the "trimmed" folder, type:

```
mkdir ariba

for f in *_R1.fastq.gz; do ariba run --threads 6 /home/inei/secuencias/prepareref/resfinder_db.out/ $f ${f%_R1.fastq.gz}_R2.fastq.gz ariba/${f%_R1.fastq.gz}.res.out.dir; done

for f in *_R1.fastq.gz; do ariba run --threads 6 /home/inei/secuencias/prepareref/vfdb_core_db.out/ $f ${f%_R1.fastq.gz}_R2.fastq.gz ariba/${f%_R1.fastq.gz}.vfdb.out.dir; done

ariba summary ariba/out_res ariba/*res.out.dir/report.tsv

ariba summary ariba/out_vfdb ariba/*vfdb.out.dir/report.tsv
```

AMRFinderPlus requires as input a fasta file, so we will try this software using the de novo assembly from T_VC4. Being in the "trimmed" folder, type:

```
amrfinder -O Vibrio_cholerae --plus -n assemblies/T_VC4.fasta -o T_VC4_amrfinderplus.tsv
```

### Determining serogroup and serotype

To determine the serotype we will check the reports obtained with ariba and vfdb database:

> We assume that an Inaba phenotype would be conferred on isolates in which ariba was unable to detect or assemble *wbeT* in its
totality, and if a mutation in *wbeT* was detected that was predicted to frameshift or truncate translated wbeT (N62fs, N165fs, F244fs, Q274trunc), was associated with Inaba phenotypes (I206K), or was otherwise known to confer an Inaba phenotype (S158P). [(Dorman et al 2020 PMID: 33004800)](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC7530988/)

To determine the serogroup, being in the "assemblies" folder, type:

```
mkdir serogroup/

cd serogroup/

cp /home/sh12/Analisis/VC/AnalisisVC/serogroupDB/DB_Vc_Oserogroup.fasta .

makeblastdb -in DB_Vc_Oserogroup.fasta -dbtype nucl -out serogroup_db

cd ..

blastn -query T_VC4.fasta -db serogroup_db/serogroup_db -out T_VC4_blast_O.txt -outfmt 6 -max_target_seqs 3
```

### Bonus! <!-- omit in toc -->

We would like to determine the *ctxB* variant, for that we will use ariba with a custome database. Being in the trimmed folder, type:

```
for f in *_R1.fastq.gz; do ariba run --threads 4 /home/sh12/Analisis/VC/AnalisisVC/ctxB_out_prepareref/ $f ${f%_R1.fastq.gz}_R2.fastq.gz ariba/${f%_R1.fastq.gz}.ctx.out.dir; done 
```








