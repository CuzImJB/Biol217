# Day 6 Workflow

## Quality control
Before assembling the reads, we check the quality of both the long and the short reads.

### Short reads
For the short reads, ```fastqc``` and ```fastp``` are used.
<!--
```micromamba activate .micromamba/envs/01_short_reads_qc```
-->
#### Fastqc
```
for i in $WORK/genomics/0_raw_reads/short_reads/*.gz
do 
    fastqc $i -o $WORK/genomics/1_short_reads_qc/1_fastqc_raw -t 8
done
```

#### Fastp
``` 
fastp -i 0_raw_reads/short_reads/241155E_R1.fastq.gz -o 1_short_reads_qc/1_fastqc_raw_clean/241155E_R1_clean.fastq.gz -I 0_raw_reads/short_reads/241155E_R2.fastq.gz -O 1_short_reads_qc/1_fastqc_raw_clean/241155E_R2_clean.fastq.gz -t 6 -q 25 -h 0_raw_reads/short_reads/report.html -R 0_raw_reads/short_reads/fastp_report

```

#### Check quality again with fastqc
```
for i in $WORK/genomics/1_short_reads_qc/1_fastqc_raw_clean/*_clean.fastq.gz
do 
    fastqc $i -o $WORK/genomics/1_short_reads_qc/1_fastqc_raw_clean -t 16
done
```
#### Questions
* How good is the read quality?
    * Average Phred score of 35 --> Good quality
* How many reads before trimming and how many do you have now?
    * Before trimming 1.639.549 and after trimming 1.613.392
* Did the quality of the reads improve after trimming?
    * The quality improved slightly --> After trimming the average Phred score is 36

### Long reads
For the long reads, ```Nanoplot``` and ```Filtlong``` are used.
<!--
```micromamba activate .micromamba/envs/02_long_reads_qc```
-->

#### Nanoplot
```
NanoPlot --fastq 0_raw_reads/long_reads/241155E.fastq.gz -o 0_raw_reads/long_reads/nanoplot -t 6 --maxlength 40000 --minlength 1000 --plots kde --format png --N50 --dpi 300 --store --raw --tsv_stats --info_in_report
```
[Nanoplot](../Images/Nanoplot_day6.png)

#### Filtlong 
```
filtlong --min_length 1000 --keep_percent 90 0_raw_reads/long_reads/241155E.fastq.gz | gzip > 241155E_cleaned_filtlong.fastq.gz
mv 241155E_cleaned_filtlong.fastq.gz 2_long_reads_qc/filtlong
```

#### Check the quality with nanoplot again
```
NanoPlot --fastq 2_long_reads_qc/filtlong/241155E_cleaned_filtlong.fastq.gz -o 2_long_reads_qc/nanoplot_clean -t 6 --maxlength 40000 --minlength 1000 --plots kde --format png --N50 --dpi 300 --store --raw --tsv_stats --info_in_report
```
[Nanoplot_clean](../Images/Nanoplot_clean_day6.png)

#### Questions
* How good is the long reads quality
    * low to moderate quality, but expected and acceptable for long-read assembly.
* How many reads before trimming and how many do you have now?
    * Before trimming 15.963 and after trimming 12.446

## Assembly of the genomce using Unicycler
<!--
```micromamba activate .micromamba/envs/03_unicycler```
-->
```
unicycler -1 1_short_reads_qc/1_fastqc_raw_clean/241155E_R1_clean.fastq.gz -2 1_short_reads_qc/1_fastqc_raw_clean/241155E_R2_clean.fastq.gz -l 2_long_reads_qc/filtlong/241155E_cleaned_filtlong.fastq.gz -o 3_assembly -t 24
``` 
