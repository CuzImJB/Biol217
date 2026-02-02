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

## Check the quality of the assembly
To check the quality of the assembly, ```Quast```, ```CheckM``` and ```CheckM2``` are used.

### Quast
<!--
```micromamba activate .micromamba/envs/04_quast```
-->
```
quast.py 3_assembly/assembly.fasta --circos -L --conserved-genes-finding --rna-finding --glimmer --use-all-alignments --report-all-metrics -o 4_assembly_qc/quast -t 8
```

### CheckM
<!--
```micromamba activate .micromamba/envs/04_checkm```
-->
```
mkdir -p 4_assembly_qc/checkm

checkm lineage_wf 3_assembly 4_assembly_qc/checkm -x fasta --tab_table --file 4_assembly_qc/checkm/checkm_results -r -t 8 
checkm tree_qa 4_assembly_qc/checkm 
checkm qa 4_assembly_qc/checkm/lineage.ms 4_assembly_qc/checkm/ -o 1 > 4_assembly_qc/checkm/final_table_01.csv
checkm qa 4_assembly_qc/checkm/lineage.ms 4_assembly_qc/checkm/ -o 2 > 4_assembly_qc/checkm/final_table_checkm.csv
```

### CheckM2
<!--
```micromamba activate .micromamba/envs/04_checkm2```
-->
```
checkm2 predict --threads 1 --input 3_assembly/assembly.fasta --output-directory 4_assembly_qc/checkm2
```

### Visualization with Bandage
[Hybrid_Assembly](../Images/Hybrid_Assembly_day6.png)

### Annotate the genomes with Prokka
<!--
```micromamba activate .micromamba/envs/05_prokka```
-->
```
prokka 3_assembly/assembly.fasta --outdir 5_annotated_genome --kingdom Bacteria --addgenes --cpus 32
```

### Classify the genomes with GTDBTK
<!--
```micromamba activate .micromamba/envs/06_gtdbtk```
-->
``` 
gtdbtk classify_wf --cpus 1 --genome_dir 6_gtdb_classification --out_dir 6_gtdb_classification --extension .fna --skip_ani_screen
```

### MultiQC to combine the reports
Run MultiQC to combine all the QC reports at once at the end of the pipeline
<!--
```micromamba activate .micromamba/envs/01_short_reads_qc```
-->
```
multiqc -d $WORK/genomics -o 7_multiqc
```

### Questions 
* How good is the quality of the genome?
    * CheckM: Completeness 99.98% and Contamination 0.29
    * Quast: N50 is 1 and L50 is 4.331.274 (Genome is almost one contig)
    * --> High quality
* Why did we use the hybrid assembler?
    * We used a hybrid assembler to combine the advantages of short and long reads.
    * Short reads are highly accurate, but cannot span repetitive regions. Long reads can bridge repeats and structural regions but have a higher error rate.
* What is the difference between short and long reads?

    | Feature | Short reads | Long reads |  
    | --- | --- | --- |
    | Read lenght | 100-300 bp | 1kb to >100kb |    
    | Accuracy | Very high| lower |         
    | Repeats solution | Poor | Excellent |     
    | Cost per base | Low | Higher |   
    | Assembly | Fragmented | More contigous |     

* Did we use Single or Paired end reads? Why? 
    * We used Paired end reads. They provide information from both end of the DNA fragment and improve assemble accuracy, repeat solutions and scaffolding. They reduce ambiguity compared to single reads.
* Which classification was assigned to the genome? Is it trustworthy and why?
    * Bacteroides muris. It is trustworthy: High completeness, low contamination; high confidence of the assignment
