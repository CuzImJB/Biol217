# Day 6 Workflow

## Quality control
Before assembling the reads, we check the quality of both the long and the short reads.

### Short reads
For the short reads, fastqc and fastp are used.

#### Fastqc
```
for i in $WORK/genomics/0_raw_reads/short_reads/*.gz
do 
    fastqc $i -o $WORK/genomics/1_short_reads_qc/1_fastqc_raw -t 8
done
```

#### Fastp
``` 
fastp -i 0_raw_reads/short_reads/241155E_R1.fastq.gz -o 0_raw_reads/short_reads/241155E_R1_clean.fastq.gz -I 0_raw_reads/short_reads/241155E_R2.fastq.gz -O 0_raw_reads/short_reads/241155E_R2.fastq_clean.gz -t 6 -q 25 -h 0_raw_reads/short_reads/report.html -R 0_raw_reads/short_reads/fastp_report
```

#### Check quality again with fastqc
```
for i in $WORK/genomics/0_raw_reads/short_reads/*_clean.gz
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