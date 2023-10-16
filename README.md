# pressilabris_SR_mapping
Looking for a genomic region associated with the sex ratio of sexual offspring in Formica pressilabris.

The raw data for this study will be archived in SRA upon submission. It consists of whole-genome resequencing (150 bp, PE) of 32 workers, each from a different nest (16 from female-producing and 16 from queen-producing colonies).

We first concatenated the data of the different runs:

```bash
ls | cut -f 2 -d '_' | sort | uniq > individuals.txt
for i in $(cat individuals.txt); do cat *_$i_*R1*.fastq.gz > $i_R1.fq.gz; cat *_$i_*R2*.fastq.gz > $i_R2.fq.gz; done
```

We then trimmed reads using `trimmomatic` v0.39:
```bash
trimmomatic PE -threads 4 -phred33 ${1}_R1.fq.gz ${1}_R2.fq.gz ILLUMINACLIP:NexteraPE-PE:2:25:10 -baseout ${1}.fq.gz LEADING:3 TRAILING:3 SLIDINGWINDOW:4:20 MINLEN:25
```

We then aligned the reads to the reference genome of the P haplotype of *Formica selysi* using `bwa mem` vXXXX:
```bash

```
