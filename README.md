# Mapping the sex ratio locus in *Formica pressilabris*
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

We then aligned the reads to the reference genome of the M haplotype of *Formica selysi* using `bwa mem` vXXXX in a loop, where `$1` is the sample name and `$2` the path to the genome.
```bash
bwa mem -t 16 $2 ${1}_1P.fq.gz ${1}_2P.fq.gz | samtools view -b - > ${1}.bam
```

We then filtered the bam files and added read group (still in a loop with the sample name as `$1`):
```bash
# fills in mate coordinates and insert size fields
samtools fixmate -m ${1}.bam - | \

# sort by coordinates
samtools sort - | \

# Mark and remove duplicates
# 2500 is the distance recommended for NovaSeq by the samtools manual
samtools markdup -r -S -d 2500 - ${1}_clean.bam

samtools addreplacerg -r "@RG\tID:${1}\tSM:${1}" -o ${1}-RG.bam ${1}_clean.bam
```

We then merged all bam files together:

```bash
samtools merge -r -o merged.bam *-RG.bam
```

Finally, we called variants with GATK version xxxx separately for each chromosome in a loop, where `$1` is the scaffold name:
```bash
gatk HaplotypeCaller -I merged.bam -O ${1}.vcf -R ../genome/FsiM_PB_v5.fasta -L $1 --max-reads-per-alignment-start 0
```
and merged all vcfs together with `bcftools` v XXXX
```bash
bcftools concat -o pressilabris_full_raw.vcf -O v FsiM_PB_v5_scf*.vcf
```

```bash
vcftools --vcf pressilabris_full_raw.vcf \
--remove-indels \
--min-alleles 2 \
--max-alleles 2 \
--minQ 20 \
--mac 3 \
--minDP 8 \
--max-meanDP 40 \
--max-missing 0.75 \
--recode \
--out pressilabris_snps_2alleles_GQ20_DP8_maxmeanDP40_mac3_miss75
```
It retained 531124 out of a possible 5570775 Sites.
